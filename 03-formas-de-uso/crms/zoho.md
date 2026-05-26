---
title: "CRM: Zoho con WhatsApp"
category: formas-de-uso-crms
tags: [zoho, crm, sales-iq, cliq]
updated: 2026-05-20
source_official:
  - https://www.zoho.com/crm/help/integrations/whatsapp.html
related:
  - 03-formas-de-uso/comparativa.md
  - 03-formas-de-uso/crms/hubspot.md
audience: [integrador, comercial]
---

# CRM: Zoho con WhatsApp

> **TL;DR:** Zoho ofrece integración con WhatsApp vía Zoho CRM, SalesIQ y Cliq. Punto dulce: costo razonable + capacidad amplia. Bueno para PyMEs que ya usan el ecosistema Zoho. Menos foco en WhatsApp puro que Respond.io/Kommo; más como un canal adicional dentro del CRM.

## Cuándo elegir Zoho

| Caso | Razón |
|---|---|
| Cliente ya usa Zoho CRM | Continuidad |
| Necesita CRM + email + calendar + más, todo en una suite | Zoho One |
| Equipo medio, presupuesto medio | Buen balance |
| Quiere features de CRM (deals, forecasting) sin pagar HubSpot/Salesforce | Punto dulce |

## Cuándo no

| Caso | Mejor |
|---|---|
| Foco fuerte en conversacional / omnicanal | Respond.io |
| LATAM PyME pura | Kommo |
| Quiere bot inteligente con IA | n8n + Cloud API |

## Productos Zoho relevantes

| Producto | Rol |
|---|---|
| Zoho CRM | Leads, deals, contactos |
| SalesIQ | Live chat / mensajería en sitio web |
| Cliq | Chat interno + canales externos |
| Marketing Automation | Campañas multicanal |
| Flow | Automatización entre apps de Zoho y externos |

WhatsApp se integra en varios de estos según el caso de uso (CRM para ventas, SalesIQ para soporte).

## Setup

1. Habilitar la integración de WhatsApp Business en Zoho CRM / SalesIQ.
2. Onboardear la WABA (flujo embebido).
3. Vincular el número.
4. Configurar plantillas.
5. Asignar el canal a equipos.

## Integración con n8n

n8n tiene nodos de Zoho CRM. Patrón:

- Zoho CRM como sistema de registro de contactos / deals.
- n8n como orquestador para lógica que Zoho Flow no resuelva.
- WhatsApp por integración nativa de Zoho o por Cloud API directa con sync.

## API

```bash
curl -X GET \
  "https://www.zohoapis.com/crm/v3/Contacts" \
  -H "Authorization: Zoho-oauthtoken {{TOKEN}}"
```

Auth: OAuth 2.0.

## Pricing

Más accesible que HubSpot y Salesforce. Zoho One ofrece toda la suite a un precio por usuario competitivo.

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| Confusión entre productos Zoho | Hay varios que tocan WA | Definir cuál es el principal (CRM vs SalesIQ) |
| Plantillas no sincronizan | Sin permiso o config | Revisar la integración |
| API rate limit | Zoho tiene límites por plan | Throttling |

## Referencias

- [Zoho CRM · WhatsApp](https://www.zoho.com/crm/help/integrations/whatsapp.html) — Verificado 2026-05-20.
- [`03-formas-de-uso/crms/hubspot.md`](./hubspot.md)
- [`03-formas-de-uso/comparativa.md`](../comparativa.md)
