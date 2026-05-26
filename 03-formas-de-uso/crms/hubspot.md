---
title: "CRM: HubSpot con WhatsApp"
category: formas-de-uso-crms
tags: [hubspot, crm, cloud-api, marketing-hub]
updated: 2026-05-20
source_official:
  - https://developers.hubspot.com/docs/api/conversations
  - https://www.hubspot.com/products/marketing/whatsapp
related:
  - 03-formas-de-uso/comparativa.md
  - 03-formas-de-uso/kommo-crm.md
audience: [integrador, comercial]
---

# CRM: HubSpot con WhatsApp

> **TL;DR:** HubSpot ofrece integración nativa con WhatsApp (Cloud API) en sus planes superiores. Foco fuerte en marketing automation y CRM corporativo; menos conversacional que Kommo o Respond.io. Útil cuando el cliente ya tiene HubSpot como CRM y quiere sumar WhatsApp sin cambiar de herramienta.

## Cuándo elegir HubSpot

| Caso | Razón |
|---|---|
| Cliente ya usa HubSpot | Continuidad |
| Marketing automation potente | HubSpot brilla aquí |
| Equipo medio-grande con foco corporativo | UI familiar |
| Necesita integrar WhatsApp con email + landing pages + ads | Todo en HubSpot |

## Cuándo no

| Caso | Mejor |
|---|---|
| Foco conversacional (inbox de agentes) | Respond.io o Kommo |
| Volumen alto de mensajes con bots | n8n + Cloud API directa |
| Presupuesto ajustado | Kommo, Cloud API directa |
| LATAM, equipos chicos | Kommo |

## Modelo de integración

HubSpot integra WhatsApp como un canal de **Conversations** (inbox unificado) y para **Marketing Hub** (campañas, automation).

| Componente | Detalle |
|---|---|
| Conversations inbox | Mensajes entrantes/salientes con asignación |
| Marketing Hub | Envíos de plantillas en workflows |
| Contacts | Cada conversación queda atada al contacto/lead de HubSpot |
| Workflows | Automatización HubSpot nativa, alternativa a n8n |

## Setup

1. Plan de HubSpot que incluye WhatsApp (suele ser Marketing Hub Professional+ o Service Hub).
2. Conectar la WABA / número desde el panel de HubSpot (flujo embebido con Meta).
3. Asignar el canal a un inbox y a agentes.
4. Crear/sincronizar plantillas.

## Versus Kommo

| Dimensión | HubSpot | Kommo |
|---|---|---|
| Foco | Corporativo, marketing automation | Conversacional, CRM de ventas LATAM |
| UI | Más densa | Más simple, kanban |
| Precio | Más alto | Más bajo |
| Workflows nativos | Potentes | Salesbot, más limitado |
| Adopción LATAM | Media | Muy alta |

## Integración con n8n

n8n tiene nodos nativos de HubSpot. Patrón híbrido:

- HubSpot como sistema de registro (contacts, deals).
- WhatsApp por la integración nativa de HubSpot, o por Cloud API directa con n8n haciendo el bot y reportando a HubSpot.

## API REST

HubSpot expone API para conversations, contacts, deals, etc. Auth con OAuth o private app token.

```bash
curl -X GET \
  "https://api.hubapi.com/crm/v3/objects/contacts?limit=10" \
  -H "Authorization: Bearer {{HUBSPOT_TOKEN}}"
```

## Pricing (orden de magnitud)

| Plan | WhatsApp incluido | Costo |
|---|---|---|
| Starter | No / limitado | Bajo |
| Professional | Sí (con add-on a veces) | Medio-alto |
| Enterprise | Sí | Alto |

Más: conversaciones de Meta (no incluidas en la suscripción HubSpot).

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| Cliente espera funcionalidad WA y plan no la incluye | Confusión de tiers | Confirmar plan |
| Costo total más alto que Kommo equivalente | HubSpot es premium | Calcular antes |
| Workflows nativos compiten con n8n | Decidir quién orquesta | Una sola fuente de lógica |
| Plantillas no aparecen | Sin sincronizar | Forzar sync |

## Referencias

- [HubSpot · WhatsApp](https://www.hubspot.com/products/marketing/whatsapp) — Verificado 2026-05-20.
- [HubSpot API · Conversations](https://developers.hubspot.com/docs/api/conversations) — Verificado 2026-05-20.
- [`03-formas-de-uso/kommo-crm.md`](../kommo-crm.md)
- [`03-formas-de-uso/comparativa.md`](../comparativa.md)
