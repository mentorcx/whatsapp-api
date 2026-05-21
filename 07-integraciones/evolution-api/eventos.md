---
title: "Evolution API: eventos webhook"
category: integraciones-evolution
tags: [evolution-api, eventos, webhook, messages-upsert, n8n]
updated: 2026-05-20
source_official:
  - https://doc.evolution-api.com/
related:
  - 07-integraciones/evolution-api/endpoints.md
  - 07-integraciones/n8n/webhook-entrante.md
audience: [integrador, dev]
---

# Evolution API: eventos webhook

> **TL;DR:** Evolution envía eventos a tu webhook cuando pasa algo: `MESSAGES_UPSERT` (mensaje entrante/saliente), `CONNECTION_UPDATE` (estado de sesión), `QRCODE_UPDATED`, `MESSAGES_UPDATE` (estados), `CONTACTS_UPSERT`, etc. El payload trae el campo `instance` y `event`. No hay firma HMAC: asegurar el endpoint por otros medios.

## Contexto

Los eventos son cómo Evolution le avisa a n8n que llegó un mensaje o cambió algo. El payload difiere del de Cloud API: este documento lo mapea.

## Eventos principales

| Evento | Cuándo se dispara | Importancia |
|---|---|---|
| `MESSAGES_UPSERT` | Llega o se envía un mensaje | Crítica |
| `MESSAGES_UPDATE` | Cambia el estado de un mensaje (entregado/leído) | Alta |
| `CONNECTION_UPDATE` | Cambia el estado de la sesión | Alta |
| `QRCODE_UPDATED` | Se generó un QR nuevo | Media |
| `CONTACTS_UPSERT` | Contacto nuevo o actualizado | Media |
| `CONTACTS_UPDATE` | Cambio en un contacto | Baja |
| `CHATS_UPSERT` | Chat nuevo | Baja |
| `PRESENCE_UPDATE` | Typing / online del otro lado | Baja |
| `GROUPS_UPSERT` | Cambios en grupos | Según uso |

## Estructura general del payload

```json
{
  "event": "messages.upsert",
  "instance": "acme-soporte",
  "data": { ... },
  "destination": "https://n8n.../webhook/evolution",
  "date_time": "2026-05-20T10:00:00.000Z",
  "server_url": "https://evo...",
  "apikey": "..."
}
```

| Campo | Para qué |
|---|---|
| `event` | Tipo de evento (en minúsculas con punto) |
| `instance` | Qué instancia/número/cliente |
| `data` | El contenido específico del evento |

## MESSAGES_UPSERT (el más importante)

Mensaje entrante de texto:

```json
{
  "event": "messages.upsert",
  "instance": "acme-soporte",
  "data": {
    "key": {
      "remoteJid": "5491133334444@s.whatsapp.net",
      "fromMe": false,
      "id": "BAE5..."
    },
    "pushName": "Juan Pérez",
    "message": {
      "conversation": "Hola, quería consultar"
    },
    "messageType": "conversation",
    "messageTimestamp": 1716200000
  }
}
```

Campos clave:

| Campo | Para qué |
|---|---|
| `data.key.remoteJid` | Número del usuario (`...@s.whatsapp.net`) |
| `data.key.fromMe` | `false` = entrante, `true` = lo enviaste vos |
| `data.key.id` | ID del mensaje — **usar para dedup** |
| `data.pushName` | Nombre del contacto |
| `data.messageType` | Tipo de mensaje |
| `data.messageTimestamp` | Unix seconds |

### Extraer el texto según tipo

El texto vive en distintos lugares según `messageType`:

| messageType | Dónde está el texto |
|---|---|
| `conversation` | `message.conversation` |
| `extendedTextMessage` | `message.extendedTextMessage.text` |
| `imageMessage` | `message.imageMessage.caption` (+ media) |
| `videoMessage` | `message.videoMessage.caption` (+ media) |
| `audioMessage` | (sin texto, es audio) |
| `documentMessage` | `message.documentMessage.caption` |
| `buttonsResponseMessage` | `message.buttonsResponseMessage.selectedButtonId` |
| `listResponseMessage` | `message.listResponseMessage.singleSelectReply.selectedRowId` |

### Parser en n8n

```javascript
const body = $input.first().json;

if (body.event === 'messages.upsert') {
  const d = body.data;
  if (d.key?.fromMe) {
    return [{ json: { skip: true, reason: 'outbound' } }];
  }
  const m = d.message || {};
  const text = m.conversation
            || m.extendedTextMessage?.text
            || m.imageMessage?.caption
            || m.videoMessage?.caption
            || m.documentMessage?.caption
            || null;
  return [{
    json: {
      event: 'message',
      instance: body.instance,
      from: d.key?.remoteJid?.split('@')[0],
      wa_message_id: d.key?.id,
      contact_name: d.pushName,
      type: d.messageType,
      timestamp: (d.messageTimestamp || 0) * 1000,
      text,
      raw: d
    }
  }];
}
```

## CONNECTION_UPDATE

```json
{
  "event": "connection.update",
  "instance": "acme-soporte",
  "data": {
    "state": "open",
    "statusReason": 200
  }
}
```

| `state` | Significado |
|---|---|
| `open` | Conectado |
| `connecting` | Conectando |
| `close` | Desconectado |

**Reaccionar:** si `state` es `close` o `connecting`, alertar al equipo. Es la señal temprana de una caída. Ver [`10-troubleshooting/evolution-desconexion.md`](../../10-troubleshooting/evolution-desconexion.md).

## MESSAGES_UPDATE

Cambio de estado de un mensaje que enviaste:

```json
{
  "event": "messages.update",
  "instance": "acme-soporte",
  "data": {
    "keyId": "BAE5...",
    "status": "DELIVERY_ACK"
  }
}
```

| `status` | Equivale a |
|---|---|
| `SERVER_ACK` | Enviado al servidor |
| `DELIVERY_ACK` | Entregado |
| `READ` | Leído |
| `PLAYED` | Audio reproducido |

## QRCODE_UPDATED

```json
{
  "event": "qrcode.updated",
  "instance": "acme-soporte",
  "data": { "qrcode": { "base64": "data:image/png;base64,..." } }
}
```

Útil para mostrar el QR en una UI propia durante el onboarding.

## Configurar qué eventos recibir

Por variable de entorno (global):

```
WEBHOOK_EVENTS_MESSAGES_UPSERT=true
WEBHOOK_EVENTS_CONNECTION_UPDATE=true
WEBHOOK_EVENTS_QRCODE_UPDATED=true
WEBHOOK_EVENTS_MESSAGES_UPDATE=true
```

O por instancia con `POST /webhook/set/{instance}` listando los `events`. Ver [`endpoints.md`](./endpoints.md).

Suscribir solo lo que usás: menos tráfico, menos ruido.

## webhook_by_events

| Valor | Comportamiento |
|---|---|
| `false` | Todos los eventos van a la misma URL |
| `true` | Cada tipo de evento va a `{url}/{evento}` |

Para n8n, `false` con un solo workflow que hace switch por `event` es lo más simple.

## Seguridad (sin firma HMAC)

A diferencia de Cloud API, Evolution **no firma** los webhooks. Asegurar el endpoint:

| Medida | Detalle |
|---|---|
| URL con token secreto en el path | `/webhook/evolution-{{secreto-largo}}` |
| Validar `apikey` del payload | Evolution incluye su apikey en el body; compararla |
| IP allowlist | Si Evolution corre en IP fija |
| Validar `instance` | Que sea una instancia conocida |

## Deduplicación

Evolution puede reenviar eventos. Deduplicar por `data.key.id`:

```
SET evo:dedup:{{key.id}} 1 NX EX 86400
```

Ver el patrón en [`07-integraciones/n8n/webhook-entrante.md`](../n8n/webhook-entrante.md).

## Diferencias con los webhooks de Cloud API

| Aspecto | Cloud API | Evolution API |
|---|---|---|
| Firma | HMAC-SHA256 | Ninguna |
| Estructura | `entry[].changes[].value` | `event` + `instance` + `data` |
| ID de mensaje | `wamid.*` | `data.key.id` |
| Tipos de evento | `messages`, etc. | `messages.upsert`, etc. |
| Identificador de tenant | `phone_number_id` | `instance` |

Un workflow de n8n bien diseñado abstrae ambos a una estructura común. Ver el parser en [`07-integraciones/n8n/webhook-entrante.md`](../n8n/webhook-entrante.md).

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| No llega ningún evento | Webhook no configurado | `POST /webhook/set` o variables globales |
| Llegan eventos pero `fromMe: true` se procesan | No filtrás los salientes | Descartar `key.fromMe === true` |
| Texto siempre `null` | Leés `conversation` para todos los tipos | Switch por `messageType` |
| Eventos duplicados | Evolution reenvía | Dedup por `key.id` |
| n8n no sabe de qué cliente | Multi-tenant sin distinguir | Usar el campo `instance` |
| Caídas no detectadas | No escuchás `CONNECTION_UPDATE` | Suscribir y alertar |

## Referencias

- [Evolution API Docs](https://doc.evolution-api.com/) — Verificado 2026-05-20.
- [`07-integraciones/evolution-api/endpoints.md`](./endpoints.md)
- [`07-integraciones/n8n/webhook-entrante.md`](../n8n/webhook-entrante.md)
- [`10-troubleshooting/evolution-desconexion.md`](../../10-troubleshooting/evolution-desconexion.md)
