---
title: "n8n: nodo WhatsApp Business Cloud"
category: integraciones-n8n
tags: [n8n, whatsapp-business-cloud, nodo, trigger]
updated: 2026-05-20
source_official:
  - https://docs.n8n.io/integrations/builtin/credentials/whatsapp/
related:
  - 07-integraciones/n8n/webhook-entrante.md
  - 07-integraciones/n8n/setup.md
audience: [integrador, dev, no-code]
---

# n8n: nodo WhatsApp Business Cloud

> **TL;DR:** n8n trae un nodo nativo de WhatsApp Business Cloud (envío + trigger). Útil para casos simples; para producción robusta conviene usar **HTTP Request** directo a Graph API (más control sobre payloads, errores, dedup, firma). Esta wiki usa HTTP Request por flexibilidad.

## Contexto

El nodo nativo simplifica el envío de mensajes para flujos rápidos. Conviene saber qué cubre y dónde tiene techo.

## Nodos disponibles

| Nodo | Función |
|---|---|
| **WhatsApp Business Cloud** (action) | Enviar mensajes (texto, media, plantilla, etc.) |
| **WhatsApp Trigger** | Recibir mensajes (alternativa al Webhook trigger genérico) |

## Credentials

Crear una credential `WhatsApp Business Cloud OAuth2 API` (o equivalente según la versión de n8n) con:

| Campo | Valor |
|---|---|
| Access Token | System User token de Meta |
| Phone Number ID | ID del número |
| Business Account ID | WABA ID (en algunas versiones) |

## Nodo de envío

Operaciones disponibles:

| Operación | Detalle |
|---|---|
| Send Message | Texto, image, audio, video, document, location, contacts |
| Send Template | Plantilla aprobada con parámetros |

Ejemplo enviar texto:

| Campo | Valor |
|---|---|
| Resource | Message |
| Operation | Send |
| Recipient Phone Number | `{{ $json.to }}` |
| Message Type | Text |
| Body | `{{ $json.text }}` |

## Nodo vs HTTP Request

| Aspecto | Nodo nativo | HTTP Request |
|---|---|---|
| Curva | Más simple, formularios | Más manual |
| Tipos cubiertos | Comunes | Todos |
| Interactivos (buttons, list, flow) | Cobertura parcial | Total |
| Errores | Manejo genérico de n8n | Vos ves la respuesta cruda |
| Versión de la API | Atada a la del nodo | Vos elegís (`v21.0`...) |
| Multi-tenant | Una credential | Token por tenant en runtime |

**Recomendación de esta wiki:** HTTP Request para producción seria. El nodo nativo, para prototipos o flujos muy simples.

## WhatsApp Trigger

Trigger que recibe webhooks de Meta. Hace el handshake (GET de verificación) y entrega los eventos. Alternativa al Webhook trigger genérico.

| Aspecto | Trigger nativo | Webhook trigger genérico |
|---|---|---|
| Verificación GET | Automática | Manual |
| Validación de firma HMAC | A veces incluida (verificar versión) | Manual |
| Parser del payload | Opinionado | Manual |
| Flexibilidad | Menor | Mayor |
| Multi-tenant | Más complicado | Mejor (vos parseás `phone_number_id`) |

Para esta wiki: **Webhook trigger genérico** + parser propio (ver [`webhook-entrante.md`](./webhook-entrante.md)). Más control, mejor para multi-tenant.

## Cuándo usar el nodo nativo

| Caso | OK |
|---|---|
| Prototipo rápido para validar | Sí |
| Operación 1 cliente, 1 número, baja complejidad | Sí |
| Aprender la API | Sí |

## Cuándo usar HTTP Request

| Caso | Mejor HTTP Request |
|---|---|
| Multi-tenant (token por cliente) | Sí |
| Necesitás features que el nodo no expone | Sí |
| Pinear versión de la API | Sí |
| Control fino de errores | Sí |
| Producción seria | Sí |

## Ejemplo: enviar texto con HTTP Request

```
Method: POST
URL: https://graph.facebook.com/v21.0/{{phoneNumberId}}/messages
Authentication: Header Auth (Authorization: Bearer ...)
Body: JSON
  {
    "messaging_product": "whatsapp",
    "to": "{{to}}",
    "type": "text",
    "text": { "body": "{{text}}" }
  }
```

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| Nodo nativo no soporta el tipo que necesitás | Cobertura parcial | HTTP Request |
| Credential del nodo expira | User token corto | System User token |
| Multi-tenant complicado con el nodo | Una sola credential | HTTP Request con token dinámico |
| Versión de la API obsoleta en el nodo | Lag con Meta | HTTP Request con versión explícita |

## Referencias

- [n8n WhatsApp credential](https://docs.n8n.io/integrations/builtin/credentials/whatsapp/) — Verificado 2026-05-20.
- [`07-integraciones/n8n/webhook-entrante.md`](./webhook-entrante.md)
- [`07-integraciones/n8n/patrones.md`](./patrones.md)
