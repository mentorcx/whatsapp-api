---
title: "n8n: subflows reutilizables"
category: integraciones-n8n
tags: [n8n, subflows, execute-workflow, reutilizable]
updated: 2026-05-20
source_official:
  - https://docs.n8n.io/flow-logic/subworkflows/
related:
  - 07-integraciones/n8n/patrones.md
audience: [integrador, dev]
---

# n8n: subflows reutilizables

> **TL;DR:** Cuando varios workflows hacen lo mismo (enviar texto a WA, descargar media, upsertear conversación), extraer esa lógica a un **subflow** invocable con **Execute Workflow**. Centraliza cambios: si Meta actualiza un payload, lo arreglás en un solo lugar.

## Contexto

Sin subflows, terminás copiando los mismos 8 nodos (enviar texto a WA con manejo de errores, fallback a plantilla, log en DB) en cada workflow. Multiplica el costo de mantenimiento. Los subflows son la solución.

## Subflows que conviene tener

| Subflow | Responsabilidad |
|---|---|
| `wa-send-text` | Enviar texto a WA con error handling |
| `wa-send-media` | Enviar media (URL o media_id) |
| `wa-send-template` | Enviar plantilla con verificación de estado |
| `wa-send-interactive` | Enviar buttons / list |
| `wa-send-or-template` | Decide free-form vs plantilla según ventana 24h |
| `wa-download-media` | Descargar media entrante respetando TTL |
| `wa-mark-as-read` | Marcar como leído |
| `openai-chat` | Llamar OpenAI con histórico estandarizado |
| `openai-embedding` | Generar embedding |
| `kommo-update-lead` | Actualizar lead Kommo con manejo de errores |
| `kommo-send-message` | Enviar mensaje al lead vía Kommo API |
| `db-conversation-upsert` | Upsert estandarizado de conversación |
| `db-message-insert` | Insertar mensaje en histórico |
| `notify-team` | Notificar Slack/email |

## Cómo se invoca

Nodo **Execute Workflow**:

| Campo | Detalle |
|---|---|
| Workflow | El subflow a ejecutar |
| Data | El JSON de input |
| Wait for completion | Sí (default) |

El subflow recibe el input, ejecuta sus nodos y devuelve datos al workflow padre.

## Estructura de un subflow

| Nodo | Función |
|---|---|
| **Execute Workflow Trigger** | Entrada del subflow |
| (lógica) | Lo que el subflow hace |
| **Set / Code (último)** | Output que vuelve al padre |

Documentar input y output:

```javascript
// wa-send-text
// INPUT:  { phoneNumberId, accessToken, to, text, replyTo? }
// OUTPUT: { ok, wamid?, error? }
```

## Ejemplo: subflow `wa-send-text`

1. **Execute Workflow Trigger** recibe `{ phoneNumberId, accessToken, to, text, replyTo }`.
2. **HTTP Request** POST a Graph API.
3. **IF** sobre el response: éxito vs error.
4. **Set** output: `{ ok: true, wamid: response.messages[0].id }` o `{ ok: false, error: ... }`.

Cualquier workflow que necesite enviar texto invoca este subflow y maneja `ok` true/false.

## Versionado

Los subflows también se versionan. Cuando hay cambio breaking:

| Opción | Detalle |
|---|---|
| Crear `wa-send-text-v2` | Conviven; migrás workflows a v2 progresivamente |
| Pisar el existente | Solo si el cambio es retro-compatible |

Recomendado: versiones explícitas cuando hay riesgo de romper consumidores.

## Tagging

Etiquetar subflows con un prefijo (`subflow:wa-*`, `subflow:openai-*`) en n8n. Permite filtrar y entender de un vistazo qué es qué.

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| Cambio en subflow rompe consumidores | Sin versionado | Versionar |
| Subflow tarda y bloquea al padre | Wait for completion sync | Considerar async con queue |
| Datos de input mal estructurados | Sin contrato claro | Documentar y validar |
| Recursión accidental | Subflow A llama a B llama a A | Diseñar dependencias acíclicas |
| Credentials del subflow no se ven | Pertenece a otro usuario / scope | Verificar accesos |

## Buenas prácticas

| Práctica | Razón |
|---|---|
| Contratos explícitos (input/output) | Mantenibilidad |
| Manejo de errores estandarizado (`ok` true/false) | Consistencia |
| Tests dedicados de cada subflow | Confianza al refactorizar |
| Documentar en el primer Code node del subflow | Doc viva |
| No abusar de subflows triviales | Si es 2 nodos, copy/paste OK |

## Referencias

- [Sub-workflows · n8n](https://docs.n8n.io/flow-logic/subworkflows/) — Verificado 2026-05-20.
- [`07-integraciones/n8n/patrones.md`](./patrones.md)
