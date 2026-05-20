---
title: Comparativa de formas de uso de WhatsApp Business
category: formas-de-uso
tags: [comparativa, cloud-api, evolution-api, respond-io, kommo, bsp]
updated: 2026-05-20
related:
  - 03-formas-de-uso/cloud-api-oficial.md
  - 03-formas-de-uso/evolution-api.md
  - 03-formas-de-uso/respond-io.md
  - 03-formas-de-uso/kommo-crm.md
---

# Comparativa de formas de uso

> **TL;DR:** Si necesitás HSM oficiales y escalabilidad, Cloud API (directa, vía BSP, Respond.io o Kommo). Si es interno o prototipo, Evolution API (con riesgo). Para agentes humanos con UI lista, Respond.io o Kommo.

## Contexto

Existen muchas formas de conectar un negocio a WhatsApp. La elección impacta en: costo, riesgo regulatorio, velocidad de implementación, escalabilidad, y experiencia para los agentes humanos. Esta tabla cruza las dimensiones más relevantes para que el integrador tome una decisión informada.

## Tabla cruzada

| Solución | Tipo | Oficial Meta | Costo base | HSM real | Multi-número | Webhooks | UI agentes | Curva | Riesgo de ban |
|---|---|---|---|---|---|---|---|---|---|
| **Cloud API directa** | API REST | Sí | Free + conversación | Sí | Sí | Nativo | No (DIY) | Media | Nulo |
| **Evolution API** | API REST self-host | No (Baileys) | Solo hosting | Simulado | Sí (instancias) | Sí | No (DIY) | Baja-Media | Alto |
| **Respond.io** | SaaS omnicanal | Sí (partner) | Suscripción + MAU | Sí | Sí | Sí | Sí | Baja | Nulo |
| **Kommo (Cloud)** | CRM + WA | Sí | Suscripción + conv | Sí | Sí | Sí | Sí (CRM) | Baja-Media | Nulo |
| **Kommo (WA Lite)** | CRM + WA Business App | No | Suscripción | No | No | Limitado | Sí (CRM) | Baja | Medio |
| **Twilio** | BSP | Sí | Pay-as-you-go + conv | Sí | Sí | Sí | No (DIY) | Media | Nulo |
| **Gupshup** | BSP | Sí | Suscripción + conv | Sí | Sí | Sí | Parcial | Media | Nulo |
| **HubSpot / Salesforce / Zoho** | CRM con WA | Sí (vía partner) | Suscripción CRM + WA | Sí | Sí | Sí | Sí (CRM) | Media-Alta | Nulo |

## Dimensiones explicadas

- **Oficial Meta**: la solución usa el protocolo oficial. Las no oficiales se conectan como WhatsApp Web y pueden ser banneadas en cualquier momento.
- **HSM real**: soporta plantillas aprobadas por Meta para mensajes business-initiated. Sin esto no se puede iniciar conversaciones fuera de la ventana de 24h de forma legítima.
- **Multi-número**: capacidad de operar varios números desde la misma plataforma.
- **UI agentes**: trae una interfaz lista para que humanos atiendan, sin desarrollo.
- **Riesgo de ban**: probabilidad de que Meta corte el acceso al número.

## Recomendaciones rápidas

| Caso de uso | Recomendación primaria | Alternativa |
|---|---|---|
| Empresa mediana, agentes humanos, marketing y soporte | Respond.io o Kommo (vía Cloud API) | HubSpot |
| Startup con foco en automatización 100% IA | Cloud API directa + n8n + OpenAI | Twilio + n8n |
| Prototipo interno, equipo chico, sin presupuesto | Evolution API self-host en Railway | Cloud API + n8n free tier |
| E-commerce con catálogo y carrito | Cloud API + n8n + integración con Tienda | Kommo + Salesbot |
| Cliente que ya usa Salesforce | Salesforce con WA nativo o Twilio | Respond.io con sync |
| Alta volumetría transaccional (banca, logística) | Cloud API directa con BSP de backup | Gupshup |

## Cuándo NO usar Evolution API

- Operaciones de marketing masivo.
- Industrias reguladas (salud, finanzas).
- Cuando el cliente final pone la cara con su número personal.
- Cuando se necesita HSM con tracking de delivery confiable.

Ver [`evolution-api.md`](./evolution-api.md) para el análisis completo de riesgos.

## Referencias

- [Cloud API Overview](https://developers.facebook.com/docs/whatsapp/cloud-api) — Meta.
- [Evolution API Docs](https://doc.evolution-api.com/) — comunidad.
- [Respond.io docs](https://docs.respond.io/) — partner oficial.
- [Kommo Help Center](https://www.kommo.com/help/) — modelos de conexión a WA.
- [Pricing de WhatsApp Business](https://developers.facebook.com/docs/whatsapp/pricing) — verificado 2026-05-20.
