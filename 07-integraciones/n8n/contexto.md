---
title: Almacenar contexto de conversación en n8n
category: integraciones-n8n
tags: [n8n, contexto, postgres, redis, estado, sesion]
updated: 2026-05-20
source_official:
  - https://docs.n8n.io/
related:
  - 07-integraciones/n8n/patrones.md
  - 07-integraciones/openai/contexto-conversacion.md
audience: [integrador, dev]
---

# Almacenar contexto de conversación en n8n

> **TL;DR:** n8n no persiste estado entre ejecuciones: usá una base externa. **Postgres** para el histórico durable de conversaciones y mensajes; **Redis** para estado efímero (dedup, locks, flags con TTL). Una tabla `wa_conversations` + una `wa_messages` cubren casi todo. No guardar contexto en variables de workflow.

## Contexto

Cada ejecución de un workflow de n8n es independiente. Cuando llega el segundo mensaje de un usuario, n8n no "sabe" nada del primero salvo que vos lo hayas guardado. El estado vive en una base externa, no en n8n.

## Qué guardar dónde

| Dato | Dónde | Por qué |
|---|---|---|
| Histórico de mensajes | Postgres | Durable, consultable, base del contexto del LLM |
| Estado de la conversación (modo bot/humano, etapa) | Postgres | Durable |
| Facts del cliente (nombre, interés, presupuesto) | Postgres (`metadata` JSONB) | Durable, estructurado |
| Dedup de wamid | Redis con TTL 24h | Efímero, alta frecuencia |
| Lock de "procesando este usuario" | Redis con TTL corto | Efímero |
| Flags temporales (ej: "esperando que mande la foto") | Redis con TTL | Efímero |
| Rate limit counters | Redis | Efímero, atómico |
| Sesión de Flow en curso | Postgres o Redis según duración | Según TTL esperado |

Regla: **durable → Postgres, efímero → Redis**.

## Schema base

```sql
CREATE TABLE wa_conversations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  wa_user_id TEXT NOT NULL,              -- E.164 sin +
  wa_phone_number_id TEXT NOT NULL,      -- para multi-numero / multi-tenant
  contact_name TEXT,
  mode TEXT NOT NULL DEFAULT 'bot',      -- bot | human
  stage TEXT,                            -- etapa libre del funnel
  last_inbound_at TIMESTAMPTZ NOT NULL,
  last_outbound_at TIMESTAMPTZ,
  assigned_agent_id TEXT,
  metadata JSONB DEFAULT '{}'::jsonb,    -- facts, summary, flags
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now(),
  UNIQUE(wa_user_id, wa_phone_number_id)
);

CREATE TABLE wa_messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  conversation_id UUID REFERENCES wa_conversations(id) ON DELETE CASCADE,
  direction TEXT NOT NULL,               -- in | out
  wa_message_id TEXT UNIQUE,             -- wamid, sirve para dedup tambien
  role TEXT NOT NULL,                    -- user | assistant | system | tool
  type TEXT,                             -- text | image | interactive | ...
  content TEXT,
  tool_call JSONB,
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_wa_messages_conv ON wa_messages(conversation_id, created_at DESC);
CREATE INDEX idx_wa_conversations_user ON wa_conversations(wa_user_id, wa_phone_number_id);
```

## Operaciones n8n ↔ Postgres

### Upsert de conversación

Al recibir un mensaje:

```sql
INSERT INTO wa_conversations (wa_user_id, wa_phone_number_id, contact_name, last_inbound_at)
VALUES ('{{from}}', '{{phone_number_id}}', '{{contact_name}}', now())
ON CONFLICT (wa_user_id, wa_phone_number_id)
DO UPDATE SET
  last_inbound_at = now(),
  contact_name = COALESCE(EXCLUDED.contact_name, wa_conversations.contact_name),
  updated_at = now()
RETURNING *;
```

Devuelve la conversación (creada o existente) con su `id`.

### Insertar mensaje

```sql
INSERT INTO wa_messages (conversation_id, direction, wa_message_id, role, type, content)
VALUES ('{{conversation_id}}', 'in', '{{wamid}}', 'user', '{{type}}', '{{text}}')
ON CONFLICT (wa_message_id) DO NOTHING
RETURNING id;
```

El `ON CONFLICT DO NOTHING` sobre `wa_message_id` te da **dedup gratis**: si el mensaje ya estaba, no devuelve fila.

### Cargar contexto para el LLM

```sql
SELECT role, content, tool_call
FROM wa_messages
WHERE conversation_id = '{{conversation_id}}'
  AND role IN ('user', 'assistant', 'tool')
ORDER BY created_at DESC
LIMIT 12;
```

Invertir el orden en un Code node para tener cronología ascendente.

### Actualizar facts

```sql
UPDATE wa_conversations
SET metadata = metadata || '{{nuevos_facts_json}}'::jsonb,
    updated_at = now()
WHERE id = '{{conversation_id}}';
```

El operador `||` sobre JSONB hace merge: agrega/sobrescribe claves sin pisar el resto.

### Cambiar modo (handoff)

```sql
UPDATE wa_conversations
SET mode = 'human', assigned_agent_id = '{{agent_id}}', updated_at = now()
WHERE id = '{{conversation_id}}';
```

## Operaciones n8n ↔ Redis

### Dedup

```
SET wa:dedup:{{wamid}} 1 NX EX 86400
```

Si devuelve `OK` → nuevo. Si `null` → duplicado.

Alternativa: confiar en el `UNIQUE(wa_message_id)` de Postgres (ver arriba). Redis es más rápido para alto volumen.

### Lock de procesamiento

Evitar que dos mensajes del mismo usuario se procesen en paralelo y pisen estado:

```
SET wa:lock:{{wa_user_id}} 1 NX EX 30
```

Si no lo obtenés, esperar y reintentar, o encolar. Liberar (`DEL`) al terminar.

### Flag temporal

```
SET wa:awaiting:{{wa_user_id}} "photo" EX 600
```

"Estoy esperando que este usuario mande una foto, durante 10 minutos."

### Rate limit counter

```
INCR wa:rate:{{wa_phone_number_id}}:{{minuto}}
EXPIRE wa:rate:{{wa_phone_number_id}}:{{minuto}} 60
```

Si el contador supera el límite, esperar.

## Nodos de n8n a usar

| Nodo | Uso |
|---|---|
| Postgres | Query / insert / update con SQL directo |
| Redis | SET/GET/INCR/DEL con opciones NX/EX |
| Code | Transformar, construir el array de mensajes, parsear JSONB |
| Set | Mapear campos entre nodos |

Para Postgres, preferir el nodo Postgres con **parámetros** (no concatenar strings) para evitar SQL injection con contenido del usuario.

## Patrón: facts vs histórico

| Enfoque | Pro | Contra |
|---|---|---|
| Solo histórico textual | Simple | El LLM puede "olvidar" datos clave; crece |
| Solo facts estructurados | Compacto, robusto | Pierde matices conversacionales |
| **Ambos** (recomendado) | Robusto + natural | Un poco más de código |

Tras cada respuesta del bot, extraer facts nuevos (con una llamada al LLM o reglas) y mergearlos al `metadata`.

## Limpieza y retención

| Dato | Retención sugerida |
|---|---|
| `wa_messages` | 90-180 días, luego archivar o borrar |
| `wa_conversations` | Mantener, son livianas |
| Redis keys | Auto-expiran por TTL |

Job de limpieza (Cron en n8n):

```sql
DELETE FROM wa_messages
WHERE created_at < now() - interval '180 days';
```

Considerar requisitos legales (GDPR/LGPD): si el usuario pide borrado, eliminar su conversación y mensajes.

## Multi-tenant

Si manejás varios clientes, **siempre** filtrar por `wa_phone_number_id` (o un `tenant_id` derivado). Nunca cargar contexto sin ese filtro: cruzarías datos entre clientes.

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| Estado se pierde entre mensajes | Guardado en variables de workflow | Usar Postgres/Redis |
| Contexto mezclado entre usuarios | `conversation_id` mal resuelto | UNIQUE (user, phone_number_id) y filtrar siempre |
| Mensajes duplicados en el histórico | Sin `ON CONFLICT` sobre `wa_message_id` | Agregar constraint UNIQUE |
| Dos respuestas pisándose | Sin lock | Lock en Redis por usuario |
| `metadata` se sobrescribe entero | `SET metadata = '{...}'` en vez de merge | Usar `metadata || '{...}'::jsonb` |
| SQL injection con texto del usuario | String concatenado en la query | Parámetros del nodo Postgres |

## Referencias

- [n8n Postgres node](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.postgres/) — Verificado 2026-05-20.
- [n8n Redis node](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.redis/) — Verificado 2026-05-20.
- [`07-integraciones/n8n/patrones.md`](./patrones.md)
- [`07-integraciones/openai/contexto-conversacion.md`](../openai/contexto-conversacion.md)
