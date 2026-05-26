---
title: "n8n: errores comunes en workflows de WhatsApp"
category: integraciones-n8n
tags: [n8n, errores, debugging, troubleshooting]
updated: 2026-05-20
related:
  - 07-integraciones/n8n/patrones.md
  - 07-integraciones/n8n/webhook-entrante.md
  - 10-troubleshooting/codigos-error.md
audience: [integrador, dev]
---

# n8n: errores comunes en workflows de WhatsApp

> **TL;DR:** Lo que más rompe en producción: webhooks duplicados (sin dedup), respuestas duplicadas del bot (sin lock), timeouts a Meta (sin queue mode), credentials que se pierden (encryption key rotada), contexto que se cruza entre usuarios (conversation_id mal resuelto). Casi todos se previenen con los patrones de [`patrones.md`](./patrones.md).

## Webhooks duplicados

| Síntoma | Causa | Solución |
|---|---|---|
| Bot responde 2-3 veces al mismo mensaje | Meta reintentó por tu lentitud | Dedup por `wamid` en Redis o Postgres UNIQUE |
| Workflow ejecuta 2 veces el mismo evento | Sin dedup + retry de Meta | Idem |

Patrón en [`patrones.md`](./patrones.md#2-dedup-por-wamid).

## Respuestas duplicadas del bot

| Síntoma | Causa | Solución |
|---|---|---|
| Dos mensajes salen del bot por uno entrante | Lógica que envía en dos ramas distintas | Revisar branches |
| Salesbot de Kommo + n8n ambos envían | Sin definir quién envía | Que solo uno envíe |
| Bot procesa su propio outbound | Filtro mal | Descartar `fromMe: true` o status `out` |

## Timeouts a Meta

| Síntoma | Causa | Solución |
|---|---|---|
| Meta reintenta mucho | Tu webhook tarda > 20s | `Respond to Webhook` con 200 inmediato, procesar async |
| Workflow lento | Sin queue mode | Activar |
| OpenAI tarda y bloquea | Llamada sync dentro del webhook | Mover a worker |

## Credentials perdidas

| Síntoma | Causa | Solución |
|---|---|---|
| Tras redeploy faltan credentials | `N8N_ENCRYPTION_KEY` cambió | Fijar la key, restaurar la original |
| Credentials ilegibles tras restore | Restauraste DB con otra key | Restaurar con la key original |

La encryption key es **el activo más crítico** después de los tokens.

## Contexto cruzado entre usuarios

| Síntoma | Causa | Solución |
|---|---|---|
| Bot responde a A con info de B | `conversation_id` no único | UNIQUE (`wa_user_id`, `wa_phone_number_id`) |
| Multi-tenant cruzado | Sin filtro por `phone_number_id` | Filtrar siempre |
| Histórico de otro lead | Query sin scope | Filtrar por conversación |

## Rate limits

| Síntoma | Causa | Solución |
|---|---|---|
| `80007` / `130429` | Demasiados envíos por segundo | Throttle client-side; ver [`10-troubleshooting/rate-limits.md`](../../10-troubleshooting/rate-limits.md) |
| `131056` | Demasiados al mismo usuario | Lock por usuario |
| `4` | App-level rate limit | Cachear lecturas, reducir polling |
| `429` de OpenAI | Tier de OpenAI superado | Retry on fail + backoff |

## Errores de variables/expressions

| Síntoma | Causa | Solución |
|---|---|---|
| `Cannot read property of undefined` | Expression accede a campo que no siempre existe | `?.` y fallbacks |
| Variable no se resuelve | Sintaxis de expression mal | Verificar `{{ $json.x }}` |
| Datos del nodo anterior vacíos | Filtro previo descartó todo | Loggear; manejar caso vacío |

## Postgres / Redis

| Síntoma | Causa | Solución |
|---|---|---|
| `too many connections` Postgres | Pool chico, muchos workers | Aumentar `max_connections` o pgbouncer |
| Lentitud en queries grandes | Sin índices | Agregar índices sobre `wa_user_id`, `conversation_id`, `wa_message_id` |
| Redis dedup no funciona | Key mal armada | Log de la key real |
| TTL expira antes de tiempo | Mal calculado | Verificar `EX` |

## Webhooks no llegan a n8n

Ver [`10-troubleshooting/webhooks-no-llegan.md`](../../10-troubleshooting/webhooks-no-llegan.md). Causas comunes:

- WABA no suscrita a la App (`subscribed_apps`).
- Path incorrecto.
- Body parseado en vez de raw → firma inválida.
- Verify token mal.

## Workflow no se activa

| Síntoma | Causa | Solución |
|---|---|---|
| Workflow no toma webhooks | Está en estado "inactivo" | Activar |
| Estás usando path `/webhook-test/...` | Modo test, solo dispara una vez | Usar `/webhook/...` y activar el workflow |
| Cambios no aplican | No guardaste / desactivaste-activaste | Save + toggle |

## OpenAI

| Síntoma | Causa | Solución |
|---|---|---|
| Respuesta del modelo incompleta | `max_tokens` muy bajo | Subirlo |
| Tool calls no se ejecutan | Parser mal | Switch sobre `tool_calls[0].function.name` |
| Modelo inventa datos | Prompt débil + sin RAG | Reforzar prompt + RAG |
| Costo alto inesperado | Historial sin recortar | Ventana deslizante |
| Loop de tool calls | Bug en handler | Limitar N tool calls por turno |

## Loops

| Síntoma | Causa | Solución |
|---|---|---|
| Kommo webhook dispara n8n que actualiza Kommo que dispara webhook | Loop | Filtrar cambios propios |
| Workflow se llama a sí mismo | Subflow mal diseñado | Diseñar dependencias acíclicas |
| Reintentos infinitos | Sin límite | maxRetries en error handler |

## Debugging

Herramientas:

| Herramienta | Uso |
|---|---|
| Executions UI | Ver runs históricos paso a paso |
| Code node con `console.log` (n8n logs) | Inspeccionar valores |
| `Respond to Webhook` antes del procesamiento | Si responde, llega el webhook |
| `webhook.site` para aislar | Probar que Meta manda algo |
| Re-run de una execution | Testear cambios |

## Error Trigger workflow

Configurar un workflow con **Error Trigger** que se ejecute cuando otros fallan. Enviarse a Slack / email para enterarse en tiempo real.

## Buenas prácticas para reducir errores

| Práctica | Razón |
|---|---|
| Subflows con manejo de errores estandarizado | Consistencia |
| Logging en cada paso crítico | Diagnóstico |
| Tests con payloads reales (de `examples/`) | Confianza |
| Versionado en Git de los JSON | Rollback |
| Error Trigger global | Alertas |
| Monitor de executions fallidas | Dashboard |

## Referencias

- [`07-integraciones/n8n/patrones.md`](./patrones.md)
- [`07-integraciones/n8n/webhook-entrante.md`](./webhook-entrante.md)
- [`10-troubleshooting/codigos-error.md`](../../10-troubleshooting/codigos-error.md)
- [`10-troubleshooting/webhooks-no-llegan.md`](../../10-troubleshooting/webhooks-no-llegan.md)
- [`10-troubleshooting/rate-limits.md`](../../10-troubleshooting/rate-limits.md)
