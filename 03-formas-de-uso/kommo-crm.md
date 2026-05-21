---
title: Kommo CRM
category: formas-de-uso
tags: [kommo, crm, salesbot, wa-lite, cloud-api]
updated: 2026-05-20
source_official:
  - https://developers.kommo.com/
  - https://www.kommo.com/help/
related:
  - 03-formas-de-uso/comparativa.md
  - 07-integraciones/kommo/README.md
  - 09-recetas/kommo-cloudapi-openai-handoff.md
  - 09-recetas/kommo-n8n-rag.md
audience: [integrador, dev, no-code]
---

# Kommo CRM

> **TL;DR:** Kommo (ex amoCRM) es un CRM con foco en mensajería muy adoptado en LATAM. Permite operar WhatsApp por tres vías distintas (WA Lite, Cloud API oficial o proveedor externo). Su valor diferencial: UI lista para agentes humanos, Salesbot nativo y pipelines visuales. Recomendado cuando el cliente necesita agentes humanos y automatización combinados sin construir UI propia.

## Contexto

Kommo nace como un CRM conversacional: lead = conversación. Esto lo diferencia de Salesforce o HubSpot donde el lead es una ficha y la conversación un add-on. Para integradores que venden a PyMEs LATAM es una de las opciones más solicitadas porque:

- Onboarding rápido para usuarios no técnicos.
- Pipeline visual tipo Kanban donde se arrastran leads.
- Salesbot incluido sin contratar herramienta aparte.
- Multi-canal: WhatsApp, Instagram, Messenger, Telegram, email, llamadas.
- Precio accesible vs HubSpot/Salesforce.

Limitaciones a tener presentes:

- El Salesbot tiene una curva de "no-code" con techo: para lógica compleja conviene delegar a n8n.
- Rate limits de la API REST que pueden morder en escenarios de alta volumetría.
- La UI no expone todas las features de la Cloud API (algunos tipos interactivos quedan afuera).

## Modelos de conexión a WhatsApp

Esta es la decisión más importante al integrar Kommo. Las tres opciones conviven y tienen costos y capacidades distintas.

| Modelo | Oficial | HSM real | Multi-agente | Pricing | Riesgo de ban | Recomendado para |
|---|---|---|---|---|---|---|
| **WhatsApp Lite** | No | No | Limitado | Incluido en plan | Medio | Pruebas, freelancers, 1 número personal |
| **Cloud API (nativo Kommo)** | Sí | Sí | Sí | Plan Kommo + conversación Meta | Nulo | Empresas con marketing y soporte serios |
| **Proveedor externo** (Wazzup, Chat API, etc.) | Depende | Depende | Sí | Suscripción proveedor + Kommo | Variable | Casos legacy o features que Kommo no expone |

### WhatsApp Lite

- Conecta el número escaneando un QR con la app de WhatsApp Business.
- No usa Cloud API: por debajo es similar a Baileys.
- No hay HSM reales; los mensajes business-initiated son simulados como texto plano.
- Solo un dispositivo "activo" a la vez (limitación heredada de WhatsApp Multi-Device).
- Suficiente para emprendedores que ya operan desde su celular y quieren centralizar en CRM.

### Cloud API integrada en Kommo

- Onboarding embebido: Kommo lleva al usuario por el flow de Meta sin salir del CRM.
- Plantillas se crean y aprueban desde Kommo o desde Meta Business Manager (sincroniza).
- Soporta multi-agente real con asignación por pipeline.
- Quality rating y tiers visibles en el CRM.
- Esta es la opción correcta para producción.

### Vía proveedor externo

- Algunos proveedores (Wazzup, Chat API, B2Chat) ofrecen integración con Kommo + WhatsApp.
- Útil cuando ya hay contrato previo con ese proveedor o se necesitan features ausentes en la Cloud nativa.
- Implica una capa más en la cadena de fallos y costos.

## Conceptos clave de Kommo

| Concepto | Descripción |
|---|---|
| **Account** | Instancia de Kommo del cliente. Subdominio único (`{{cliente}}.kommo.com`). |
| **User** | Agente humano del CRM con permisos. |
| **Lead** | Oportunidad / conversación. Vive en un pipeline. |
| **Pipeline** | Conjunto ordenado de stages (Kanban). Cada cuenta puede tener varios. |
| **Stage** | Columna del Kanban. Estados como "Sin contactar", "Calificado", "Ganado", "Perdido". |
| **Contact** / **Company** | Personas y empresas asociadas a leads. |
| **Salesbot** | Motor de bots visual basado en bloques. |
| **Digital Pipeline** | Automatizaciones por stage (enviar mensaje, ejecutar bot, llamar webhook). |
| **Widget** | Integración instalada (WA, Salesbot, integraciones de terceros). |
| **Long-lived token** | Token de larga duración para integraciones server-to-server. |

## Salesbot vs n8n

| Capacidad | Salesbot (Kommo) | n8n |
|---|---|---|
| Curva | Muy baja, visual | Media |
| Branching simple | Excelente | Excelente |
| Llamadas HTTP a APIs externas | Limitado | Nativo |
| Manejo de contexto / memoria | Muy limitado | Total |
| Integración con OpenAI | Vía webhook a endpoint propio | Nativo |
| Editar lead / cambiar stage | Nativo | Vía API REST |
| Costos | Incluido | Hosting separado |
| Versionado | Cada cambio queda guardado | Git si se exporta JSON |

**Recomendación práctica:** usar Salesbot para flujos lineales simples (FAQ, calificación inicial, derivación) y delegar a n8n cualquier lógica que requiera estado, function calling, RAG o varias llamadas externas.

## Patrón híbrido recomendado

```
Usuario WhatsApp
    ↓
Cloud API (vía Kommo)              ← Transporte oficial
    ↓
Kommo Account                       ← Lead + UI agentes
    ↓ (Salesbot llama webhook)
n8n                                 ← Orquestación
    ↓ ↑
OpenAI                              ← Cerebro
    ↑
Kommo API REST                      ← n8n actualiza lead, cambia stage, asigna agente
```

Este patrón aparece en las recetas [`09-recetas/kommo-cloudapi-openai-handoff.md`](../09-recetas/kommo-cloudapi-openai-handoff.md) y [`09-recetas/kommo-n8n-rag.md`](../09-recetas/kommo-n8n-rag.md) (pendientes).

## Autenticación de la API REST

Kommo expone OAuth 2.0 y **long-lived tokens** para integraciones server-to-server.

| Tipo | Cuándo usar |
|---|---|
| OAuth 2.0 | Marketplace, integraciones distribuidas con muchos clientes |
| Long-lived token | Integración interna de un solo cliente (más simple) |

Long-lived token se genera desde **Settings → Integrations → Crear integración** y vive en el header:

```
Authorization: Bearer {{LONG_LIVED_TOKEN}}
```

Detalle en [`07-integraciones/kommo/api-rest.md`](../07-integraciones/kommo/api-rest.md) (pendiente).

## Webhooks de Kommo

Eventos que disparan llamadas salientes a n8n u otra URL:

| Evento | Cuándo se dispara | Uso típico |
|---|---|---|
| Lead creado | Llega un mensaje nuevo de número desconocido | Calificación inicial |
| Lead status changed | Lead se mueve a otro stage del pipeline | Trigger de automatización |
| Incoming message | Mensaje entrante de WA / IG / etc. | Bot conversacional |
| Bot ended | Salesbot termina su ejecución | Handoff a humano |
| Note created | Nota interna agregada | Sync con sistemas externos |

## Limitaciones conocidas

- **Rate limits**: ~7 requests/segundo por cuenta en la API REST. Implementar throttling en n8n.
- **Salesbot delays**: bloques de espera tienen granularidad mínima de minutos.
- **Multi-país**: cuentas en `.kommo.com` y `.amocrm.ru` no comparten datos.
- **Búsqueda de leads**: full-text limitado, mejor indexar en sistema propio si hace falta.

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| `401 Unauthorized` con token long-lived | Token revocado o cuenta cambió de plan | Regenerar y actualizar credentials en n8n. |
| Salesbot no dispara | No está vinculado al pipeline o stage correctos | Revisar Digital Pipeline → stage → trigger. |
| Webhook a n8n falla con timeout | n8n tarda más de 30s | Responder rápido y procesar async. |
| Plantillas no aparecen en Kommo | No sincronizadas o categoría rechazada | Forzar sync desde el widget de WA. |
| Lead duplicado por cada mensaje | Webhook mal configurado o lógica de match ausente | Buscar por contacto antes de crear lead. |

## Referencias

- [Kommo Developers Portal](https://developers.kommo.com/) — Verificado 2026-05-20.
- [Help Center · WhatsApp en Kommo](https://www.kommo.com/help/) — Verificado 2026-05-20.
- [`03-formas-de-uso/comparativa.md`](./comparativa.md)
- [`07-integraciones/kommo/`](../07-integraciones/kommo/) — Salesbot, API, webhooks, integración con n8n y OpenAI.
