---
title: "BSP: Gupshup"
category: formas-de-uso-bsps
tags: [gupshup, bsp, cloud-api, suscripcion]
updated: 2026-05-20
source_official:
  - https://www.gupshup.io/developer/docs
related:
  - 03-formas-de-uso/comparativa.md
  - 03-formas-de-uso/bsps/twilio.md
audience: [integrador, comercial, dev]
---

# BSP: Gupshup

> **TL;DR:** Gupshup es un BSP popular en mercados emergentes (India, LATAM, África), con foco en mensajería masiva y bots. Modelo de **suscripción** mensual + conversaciones. Ofrece bot builder propio y API REST. Útil cuando Meta no opera directo en el mercado o cuando el cliente quiere las herramientas adicionales del proveedor.

## Contexto

Gupshup compite con Twilio como BSP global, con mayor presencia en mercados emergentes. Difiere en el modelo de pricing (suscripción) y en las herramientas que ofrece encima de la API.

## Cuándo conviene Gupshup

| Caso | Razón |
|---|---|
| Mercado donde Gupshup tiene presencia fuerte | Soporte local mejor que Twilio |
| Necesitás bot builder visual incluido | Tienen herramientas propias |
| Volumen alto sostenido | Suscripción puede ser más predecible que pay-as-you-go |
| Cliente acepta lock-in moderado | A cambio de soporte |

## Cuándo no

| Caso | Mejor |
|---|---|
| Pay-as-you-go preferido | Twilio o Cloud API directa |
| Cero markup | Cloud API directa |
| Integraciones con SMS/voz globales | Twilio |

## Diferencias con Cloud API directa

| Aspecto | Cloud API directa | Gupshup |
|---|---|---|
| Endpoints | `graph.facebook.com` | API REST de Gupshup |
| Auth | Bearer token de Meta | API key de Gupshup |
| Plantillas | Meta directo | Vía Gupshup o Meta (sincronizan) |
| Bot builder | DIY | Incluido |
| Pricing | Por conversación de Meta | Suscripción + por conversación |

## Setup básico

1. Crear cuenta Gupshup.
2. Solicitar habilitación de WhatsApp Business API.
3. Onboardear el número (Gupshup asiste con Meta).
4. Obtener API key desde el panel.
5. Configurar webhook hacia tu backend / n8n.

## Enviar un mensaje (Gupshup API)

```bash
curl -X POST "https://api.gupshup.io/wa/api/v1/msg" \
  -H "apikey: {{GUPSHUP_API_KEY}}" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  --data-urlencode "channel=whatsapp" \
  --data-urlencode "source={{GUPSHUP_NUMBER}}" \
  --data-urlencode "destination={{DESTINATION}}" \
  --data-urlencode 'message={"type":"text","text":"Hola desde Gupshup"}' \
  --data-urlencode "src.name={{APP_NAME}}"
```

| Campo | Detalle |
|---|---|
| `apikey` | Header con tu API key |
| `source` | Número emisor sin `+` |
| `destination` | Número destino sin `+` |
| `message` | JSON serializado del payload |

## Plantillas en Gupshup

Las plantillas se crean en el panel de Gupshup (que las propaga a Meta) o se sincronizan desde Meta Business Manager.

Enviar plantilla:

```bash
--data-urlencode 'template={"id":"{{TEMPLATE_ID}}","params":["Juan","ARS 1000"]}'
```

## Webhooks

Gupshup envía eventos a la URL configurada con su propia estructura (varía según el evento). Distinta tanto de Cloud API como de Twilio. n8n: parser específico.

| Evento | Detalle |
|---|---|
| `message` | Mensaje entrante |
| `message-event` | Estados de mensaje saliente |
| `user-event` | Eventos del usuario (opt-in, etc.) |

## Bot builder

Gupshup ofrece un constructor visual de bots. Útil para casos no-code, similar a Salesbot de Kommo o Typebot. Para integraciones serias con IA, mejor delegar a n8n.

## Pricing

| Concepto | Cobro |
|---|---|
| Suscripción mensual | Por tier |
| Conversaciones | Tarifa Meta + markup |
| Features extra | Adicionales (chatbot, analítica) |

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| Mensaje no llega | `source` o `destination` mal formateados | Sin `+`, solo dígitos |
| Plantilla no encontrada | `template.id` distinto al `name` | Verificar ID en el panel |
| Webhook con formato extraño | Cada evento tiene shape distinto | Parser flexible |
| API key rechazada | Header mal escrito | Es `apikey` minúsculas |

## Referencias

- [Gupshup Developer Docs](https://www.gupshup.io/developer/docs) — Verificado 2026-05-20.
- [`03-formas-de-uso/bsps/twilio.md`](./twilio.md)
- [`03-formas-de-uso/comparativa.md`](../comparativa.md)
