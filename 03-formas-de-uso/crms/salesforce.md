---
title: "CRM: Salesforce con WhatsApp"
category: formas-de-uso-crms
tags: [salesforce, crm, service-cloud, messaging]
updated: 2026-05-20
source_official:
  - https://help.salesforce.com/s/articleView?id=sf.messaging_whatsapp.htm
related:
  - 03-formas-de-uso/comparativa.md
  - 03-formas-de-uso/crms/hubspot.md
audience: [integrador, comercial]
---

# CRM: Salesforce con WhatsApp

> **TL;DR:** Salesforce integra WhatsApp principalmente a través de **Service Cloud Messaging** (Digital Engagement). Es la opción para empresas grandes que ya operan sobre Salesforce: alto costo, alta capacidad, requiere equipo Salesforce. No es default para PyMEs LATAM.

## Cuándo elegir Salesforce

| Caso | Razón |
|---|---|
| Empresa ya usa Salesforce como CRM | Continuidad |
| Industria regulada (banca, salud corporativa) | Cumplimiento y gobernanza |
| Operación a gran escala con muchos canales | Service Cloud lo maneja |
| Hay equipo Salesforce (admin / dev) | Necesario |

## Cuándo no

| Caso | Mejor |
|---|---|
| PyME, equipo chico | Kommo o Respond.io |
| Presupuesto ajustado | Cloud API directa |
| Foco solo en WhatsApp | Respond.io |
| No hay capacidad técnica Salesforce | Otra plataforma |

## Componentes Salesforce relevantes

| Componente | Para qué |
|---|---|
| Service Cloud | CRM de soporte / atención |
| Digital Engagement / Messaging | Canales de mensajería (WhatsApp, SMS, FB Messenger, web chat) |
| Marketing Cloud | Campañas multi-canal |
| Flows | Automatizaciones declarativas |
| Apex / APIs | Desarrollo custom |

## Modelo de integración

Salesforce conecta WhatsApp como un **Messaging Channel** dentro de Digital Engagement (Service Cloud) o vía Marketing Cloud. Cada conversación queda asociada a un Contact / Case / Lead.

| Aspecto | Detalle |
|---|---|
| WABA | Vinculada a Salesforce |
| Plantillas | Sincronizadas |
| Routing | Por Service Cloud Routing |
| Agentes | Service Console |
| Analytics | Reports & Dashboards nativos |

## Setup (resumen)

1. Plan / licencia Salesforce que incluye Digital Engagement.
2. Configurar el Messaging Channel para WhatsApp.
3. Onboardear la WABA (proceso guiado).
4. Configurar routing y queues.
5. Crear plantillas.
6. Asignar a agentes.

Requiere admin de Salesforce. No es self-service.

## Integración con n8n

n8n tiene nodos de Salesforce (auth con OAuth). Patrón típico:

- Salesforce como sistema de registro y UI de agentes.
- n8n como orquestador para lógica que Flows/Apex no resuelven o serían más caros de mantener.
- WhatsApp por el canal nativo de Salesforce o por Cloud API directa con sync a Salesforce.

## API

Salesforce REST API:

```bash
curl -X GET \
  "https://{{INSTANCE}}.my.salesforce.com/services/data/v60.0/sobjects/Contact/{{ID}}" \
  -H "Authorization: Bearer {{ACCESS_TOKEN}}"
```

Auth: OAuth 2.0.

## Pricing

Costo elevado (Service Cloud + Digital Engagement). Cotización por usuario por mes; suele ser de los CRM más caros.

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| Cliente sin licencia adecuada | Plan no incluye WA | Confirmar antes |
| Setup demorado | Requiere admin Salesforce | Coordinar con su equipo |
| Costo subestimado | Licencias por usuario | Calcular bien antes |
| Workflows en Flows muy rígidos | A veces n8n es mejor | Patrón híbrido |

## Referencias

- [Salesforce · WhatsApp Messaging](https://help.salesforce.com/s/articleView?id=sf.messaging_whatsapp.htm) — Verificado 2026-05-20.
- [`03-formas-de-uso/comparativa.md`](../comparativa.md)
- [`03-formas-de-uso/crms/hubspot.md`](./hubspot.md)
