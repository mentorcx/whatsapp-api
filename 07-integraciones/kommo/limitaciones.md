---
title: "Kommo: limitaciones conocidas"
category: integraciones-kommo
tags: [kommo, limitaciones, rate-limit, salesbot]
updated: 2026-05-20
source_official:
  - https://developers.kommo.com/
related:
  - 07-integraciones/kommo/api-rest.md
  - 07-integraciones/kommo/salesbot.md
audience: [integrador, dev]
---

# Kommo: limitaciones conocidas

> **TL;DR:** Lo que conviene saber antes de prometerle algo a un cliente: rate limit de la API (~7 req/s), Salesbot con esperas de granularidad de minutos, búsqueda de leads limitada, cuentas regionales que no comparten datos, y techo de complejidad del Salesbot. Casi todo se mitiga delegando a n8n.

## Contexto

Conocer los límites evita prometer lo que Kommo no da y diseñar workarounds desde el inicio.

## Rate limits de la API

| Límite | Valor referencial |
|---|---|
| Requests por segundo | ~7 por cuenta |

| Implicancia | Mitigación |
|---|---|
| Operaciones masivas chocan el límite | Batch (`PATCH /leads` con array) |
| Picos de webhooks que disparan API calls | Throttling en n8n |
| Sync de muchos leads | Espaciar, batchear |

Ante `429`, backoff exponencial.

## Salesbot

| Limitación | Detalle | Mitigación |
|---|---|---|
| Esperas de granularidad de minutos | No hay "esperar 10 segundos" | Delegar timing fino a n8n |
| Memoria limitada | Depende de campos del lead | Estado en Postgres vía n8n |
| Condicionales complejos | Se vuelven inmanejables | Lógica en n8n |
| Sin LLM nativo | No hay IA en el Salesbot | OpenAI vía n8n |
| Debugging limitado | Difícil rastrear por qué un bot hizo algo | Loggear en n8n |
| Timeout corto del bloque webhook | El webhook receptor debe responder rápido | 200 inmediato + async |

## Búsqueda y consultas

| Limitación | Detalle |
|---|---|
| Búsqueda full-text limitada | El `query` param es básico |
| Filtros complejos restringidos | No es un motor de queries potente |
| Paginación | Hay que paginar resultados grandes |

Si necesitás búsquedas potentes sobre los datos, mantener un índice propio (Postgres / Elastic) sincronizado.

## Cuentas regionales

| Limitación | Detalle |
|---|---|
| `.kommo.com` y `.amocrm.ru` | Plataformas separadas, no comparten datos |
| Migración entre regiones | No trivial |

Confirmar en qué plataforma está la cuenta del cliente antes de integrar.

## WhatsApp Lite

| Limitación | Detalle |
|---|---|
| No oficial | Riesgo de desconexión/ban |
| Sin HSM reales | No apto para marketing serio |
| Multi-device limitado | Un dispositivo activo |

Para producción seria: Cloud API integrada, no Lite. Ver [`conexion-whatsapp.md`](./conexion-whatsapp.md).

## Plantillas

| Limitación | Detalle |
|---|---|
| Sincronización con Meta | A veces hay delay |
| Edición | Editar en Meta puede requerir re-sync en Kommo |
| Algunos tipos interactivos avanzados | Pueden no estar expuestos en la UI |

## Custom fields

| Limitación | Detalle |
|---|---|
| Tipos de campo | Set acotado (texto, número, select, fecha, etc.) |
| Cantidad | Hay límites según plan |
| Performance | Muchos campos custom pueden enlentecer |

## Webhooks

| Limitación | Detalle |
|---|---|
| Sin firma HMAC | A diferencia de Meta, no firma los webhooks |
| Posibles duplicados | Hay que deduplicar |
| Formato variable por evento | El parser debe ser flexible |
| Riesgo de loops | n8n actualiza → dispara webhook |

Ver [`webhooks.md`](./webhooks.md).

## Reporting

| Limitación | Detalle |
|---|---|
| Reportes nativos | Útiles pero no infinitamente flexibles |
| Analítica avanzada | Para BI serio, exportar a un data warehouse |

## Planes y precios

| Limitación | Detalle |
|---|---|
| Features por plan | Algunas (Salesbot avanzado, más campos) requieren plan superior |
| Usuarios | Costo por agente |
| MAU / contactos | Según plan |

Verificar que el plan del cliente incluye lo que la integración necesita **antes** de desarrollar.

## Qué se mitiga con n8n

La mayoría de las limitaciones del Salesbot y de la lógica:

| Limitación de Kommo | Lo resuelve n8n |
|---|---|
| Sin IA | OpenAI |
| Sin RAG | Vector DB + embeddings |
| Esperas de minutos | Timing fino en n8n |
| Lógica compleja | Workflows |
| Integraciones con muchas APIs | Nodos HTTP |
| Estado / memoria | Postgres |

Lo que **no** se mitiga: rate limit de la API de Kommo, límites de plan, cuentas regionales.

## Checklist antes de prometer

| Pregunta | Verificar |
|---|---|
| ¿El plan del cliente incluye lo necesario? | Sí |
| ¿El volumen respeta el rate limit (~7 req/s)? | Sí, o batchear |
| ¿Necesita IA/RAG? | Entonces n8n obligatorio |
| ¿WhatsApp Lite o Cloud API? | Cloud para producción |
| ¿Cuenta `.kommo.com` o `.amocrm.ru`? | Confirmar |
| ¿Necesita búsquedas potentes? | Índice propio |

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| `429` en operaciones masivas | Rate limit | Batch + throttling |
| Bot lento por esperas del Salesbot | Granularidad de minutos | Timing en n8n |
| Prometiste IA y el Salesbot no la tiene | Desconocer el límite | n8n + OpenAI desde el diseño |
| Feature no disponible en el plan del cliente | No se verificó | Confirmar plan antes |
| Loops de webhooks | Sin filtro de cambios propios | Marcar y filtrar |

## Referencias

- [Kommo Developers](https://developers.kommo.com/) — Verificado 2026-05-20.
- [`07-integraciones/kommo/api-rest.md`](./api-rest.md)
- [`07-integraciones/kommo/salesbot.md`](./salesbot.md)
- [`07-integraciones/kommo/webhooks.md`](./webhooks.md)
