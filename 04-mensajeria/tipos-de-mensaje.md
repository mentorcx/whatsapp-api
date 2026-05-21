---
title: Tipos de mensaje en Cloud API
category: mensajeria
tags: [mensajeria, text, media, location, contacts, reaction, sticker]
updated: 2026-05-20
source_official:
  - https://developers.facebook.com/docs/whatsapp/cloud-api/reference/messages
related:
  - 04-mensajeria/interactivos.md
  - 04-mensajeria/limites-media.md
  - 02-numeros-y-conexion/webhooks.md
audience: [integrador, dev]
---

# Tipos de mensaje en Cloud API

> **TL;DR:** Cloud API soporta **text, image, audio, video, document, sticker, location, contacts, reaction, interactive (buttons/lists), template** y replies. Todos se envían vía `POST /{{PHONE_NUMBER_ID}}/messages` cambiando el campo `type` y el objeto correspondiente. Media puede ir por **URL pública** o **media ID** (subida previa).

## Contexto

Conocer los tipos disponibles evita reinventar UX. Una pregunta sí/no se resuelve mejor con un botón que con texto libre. Una imagen vale por 200 palabras y una location ahorra confusiones. Este documento es el menú.

## Endpoint base

```
POST https://graph.facebook.com/v21.0/{{PHONE_NUMBER_ID}}/messages
Headers:
  Authorization: Bearer {{ACCESS_TOKEN}}
  Content-Type: application/json
```

Body común a todos:

```json
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "{{DESTINATION_E164}}",
  "type": "{{TYPE}}",
  "{{TYPE}}": { ... }
}
```

| Placeholder | Descripción |
|---|---|
| `PHONE_NUMBER_ID` | ID del número emisor |
| `ACCESS_TOKEN` | System User token |
| `DESTINATION_E164` | Número destino sin `+`, ej `5491133334444` |
| `TYPE` | `text`, `image`, `interactive`, etc. |

## Tipos disponibles

### Text

```json
{
  "type": "text",
  "text": {
    "body": "Hola, ¿en qué te puedo ayudar?",
    "preview_url": true
  }
}
```

| Campo | Detalle |
|---|---|
| `body` | Máx 4096 caracteres |
| `preview_url` | Si `true`, renderiza preview del primer link |

### Image

Dos formas: URL pública o `media_id` previo.

**Por URL:**

```json
{
  "type": "image",
  "image": {
    "link": "https://midominio.com/img.jpg",
    "caption": "Mirá este modelo"
  }
}
```

**Por media_id:**

```json
{
  "type": "image",
  "image": {
    "id": "{{MEDIA_ID}}",
    "caption": "Mirá este modelo"
  }
}
```

| Detalle | Valor |
|---|---|
| Formatos | JPEG, PNG |
| Tamaño máx | 5 MB |
| Caption | Opcional, máx 1024 chars |

### Audio

```json
{
  "type": "audio",
  "audio": { "link": "https://midominio.com/clip.mp3" }
}
```

| Detalle | Valor |
|---|---|
| Formatos | AAC, MP3, MP4 audio, OGG (Opus), AMR |
| Tamaño máx | 16 MB |
| Caption | No soportado |

Útil para mensajes de voz pregrabados, confirmaciones audibles.

### Video

```json
{
  "type": "video",
  "video": {
    "link": "https://midominio.com/video.mp4",
    "caption": "Tutorial rápido"
  }
}
```

| Detalle | Valor |
|---|---|
| Formato | MP4 (H.264 + AAC) |
| Tamaño máx | 16 MB |
| Caption | Opcional, máx 1024 chars |

### Document

```json
{
  "type": "document",
  "document": {
    "link": "https://midominio.com/factura.pdf",
    "filename": "Factura-2026-05.pdf",
    "caption": "Tu factura del mes"
  }
}
```

| Detalle | Valor |
|---|---|
| Formato típico | PDF |
| Tamaño máx | 100 MB |
| Filename | Recomendado: define cómo se muestra |

### Sticker

```json
{
  "type": "sticker",
  "sticker": { "id": "{{MEDIA_ID}}" }
}
```

| Detalle | Valor |
|---|---|
| Formato | WebP |
| Estático | Máx 100 KB, 512×512 |
| Animado | Máx 500 KB |

Útil para personalidad de marca; no abusar.

### Location

```json
{
  "type": "location",
  "location": {
    "latitude": -34.603722,
    "longitude": -58.381592,
    "name": "Casa Central",
    "address": "Av. de Mayo 1000, CABA"
  }
}
```

Renderiza mapa interactivo. Útil para enviar dirección de sucursal.

### Contacts

```json
{
  "type": "contacts",
  "contacts": [{
    "name": { "formatted_name": "Atención al cliente", "first_name": "Atención" },
    "phones": [{ "phone": "+5491100000000", "type": "WORK", "wa_id": "5491100000000" }],
    "emails": [{ "email": "soporte@negocio.com", "type": "WORK" }]
  }]
}
```

Sirve para compartir tarjeta de contacto que el usuario puede guardar.

### Reaction

```json
{
  "type": "reaction",
  "reaction": {
    "message_id": "wamid.HBgL...",
    "emoji": "👍"
  }
}
```

Reacciona a un mensaje previo (entrante o saliente). `emoji: ""` quita la reacción.

### Reply (responder a un mensaje específico)

Cualquier tipo puede ir como reply usando `context`:

```json
{
  "type": "text",
  "context": { "message_id": "wamid.HBgL..." },
  "text": { "body": "Te respondo lo que preguntaste..." }
}
```

En WhatsApp se ve la cita arriba del mensaje, mejorando legibilidad en conversaciones largas.

### Interactive (buttons, lists, CTA URL)

Cubierto en [`interactivos.md`](./interactivos.md).

### Template

```json
{
  "type": "template",
  "template": {
    "name": "recordatorio_turno",
    "language": { "code": "es_AR" },
    "components": [{ "type": "body", "parameters": [{ "type": "text", "text": "Juan" }] }]
  }
}
```

Único tipo válido fuera de la ventana de 24h. Detalle en [`05-plantillas-hsm/`](../05-plantillas-hsm/).

## Subir media previa (recomendado para alta frecuencia)

Para evitar que Meta tenga que descargar la URL cada vez:

```bash
curl -X POST \
  "https://graph.facebook.com/v21.0/{{PHONE_NUMBER_ID}}/media" \
  -H "Authorization: Bearer {{ACCESS_TOKEN}}" \
  -F "file=@/path/to/img.jpg" \
  -F "type=image/jpeg" \
  -F "messaging_product=whatsapp"
```

Respuesta:

```json
{ "id": "1234567890" }
```

Ese `id` se usa como `media_id` en envíos posteriores. Los IDs **expiran a los 30 días** y son privados al `phone_number_id`.

## Descargar media entrante

Cuando el usuario te envía media (imagen, audio, etc.), llega `id` en el webhook. Para descargar:

```bash
# 1. Pedir la URL temporal del media
curl -X GET \
  "https://graph.facebook.com/v21.0/{{MEDIA_ID}}" \
  -H "Authorization: Bearer {{ACCESS_TOKEN}}"
```

Respuesta:

```json
{
  "url": "https://lookaside.fbsbx.com/...",
  "mime_type": "image/jpeg",
  "sha256": "...",
  "file_size": "12345",
  "id": "{{MEDIA_ID}}",
  "messaging_product": "whatsapp"
}
```

```bash
# 2. Descargar el archivo con el token
curl -X GET "{{URL}}" \
  -H "Authorization: Bearer {{ACCESS_TOKEN}}" \
  --output /tmp/file.jpg
```

| Detalle | Valor |
|---|---|
| TTL de la URL | ~5 minutos |
| Auth requerida | Sí, con el token |
| Reuso | Cada GET genera una URL nueva |

## Respuesta del API al enviar

Éxito típico:

```json
{
  "messaging_product": "whatsapp",
  "contacts": [{ "input": "5491133334444", "wa_id": "5491133334444" }],
  "messages": [{ "id": "wamid.HBgL...", "message_status": "accepted" }]
}
```

| Campo | Para qué |
|---|---|
| `messages[0].id` | `wamid` para tracking |
| `contacts[0].wa_id` | Confirmación del número como user de WA |

`message_status: accepted` significa "Meta lo aceptó", no "el usuario lo recibió". Eso llega vía webhook después.

## Marcar mensaje como leído

```json
{
  "messaging_product": "whatsapp",
  "status": "read",
  "message_id": "wamid.HBgL..."
}
```

POST al mismo endpoint `/messages`. Cambia el doble check a azul del lado del usuario.

## Typing indicator

Disponible vía endpoint específico (varía por versión de API). Útil para que el usuario sepa que el bot está procesando antes de responder.

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| `Invalid parameter` al enviar imagen | URL no pública o sin extensión correcta | Verificar accesibilidad sin auth |
| Imagen llega pero rota | Formato no soportado (HEIC, BMP, etc.) | Convertir a JPEG/PNG antes |
| Audio no se reproduce en iPhone | OGG sin Opus | Usar AAC o MP3 |
| Media ID falla con 400 | Subido para otro `phone_number_id` | Cada número tiene sus media IDs |
| Sticker llega como imagen estática | Tamaño excede límite animado | Reducir a < 500 KB |
| Reply no se ve como cita | `context.message_id` mal | Usar el `wamid` exacto del mensaje a citar |

## Buenas prácticas

- Para alta frecuencia con la misma imagen, subir una vez y reusar `media_id`.
- Si tu CDN es lento, mejor subir vía `/media` que pasar URL.
- Caption en imágenes/documentos > enviar imagen + texto suelto (mejor UX).
- Para enviar más de un archivo, mandarlos en mensajes separados, no concatenados.
- Mensajes muy largos: dividir en 2-3 párrafos en mensajes separados; mejora legibilidad.

## Referencias

- [Messages Reference · Cloud API](https://developers.facebook.com/docs/whatsapp/cloud-api/reference/messages) — Verificado 2026-05-20.
- [Media Upload](https://developers.facebook.com/docs/whatsapp/cloud-api/reference/media) — Verificado 2026-05-20.
- [`04-mensajeria/interactivos.md`](./interactivos.md)
- [`04-mensajeria/limites-media.md`](./limites-media.md)
- [`02-numeros-y-conexion/webhooks.md`](../02-numeros-y-conexion/webhooks.md)
