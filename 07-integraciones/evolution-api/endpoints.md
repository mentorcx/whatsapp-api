---
title: "Evolution API: endpoints"
category: integraciones-evolution
tags: [evolution-api, endpoints, api, mensajes]
updated: 2026-05-20
source_official:
  - https://doc.evolution-api.com/
related:
  - 07-integraciones/evolution-api/instancias.md
  - 07-integraciones/evolution-api/eventos.md
audience: [integrador, dev]
---

# Evolution API: endpoints

> **TL;DR:** API REST. Autenticación por header `apikey`. Todos los endpoints de mensajería llevan el `instanceName` en el path: `/message/sendText/{instance}`. Esta es la referencia de los endpoints más usados (v2). El payload de mensajes difiere del de Cloud API.

## Contexto

Referencia técnica de los endpoints de Evolution API v2. Para el ciclo de vida de instancias ver [`instancias.md`](./instancias.md); para eventos webhook ver [`eventos.md`](./eventos.md).

## Autenticación

Header en cada request:

```
apikey: {{API_KEY}}
```

`API_KEY` es el valor de `AUTHENTICATION_API_KEY` del servidor. No es `Authorization`, no es `Bearer`. Es literalmente `apikey`.

## Base URL

```
https://{{EVO_HOST}}/
```

Los endpoints de mensajería incluyen el `instanceName`:

```
/message/sendText/{{INSTANCE}}
```

## Instancias

| Acción | Método + Path |
|---|---|
| Crear instancia | `POST /instance/create` |
| Conectar (QR/pairing) | `GET /instance/connect/{instance}` |
| Estado de conexión | `GET /instance/connectionState/{instance}` |
| Listar instancias | `GET /instance/fetchInstances` |
| Logout | `DELETE /instance/logout/{instance}` |
| Eliminar | `DELETE /instance/delete/{instance}` |
| Reiniciar | `PUT /instance/restart/{instance}` |

Detalle en [`instancias.md`](./instancias.md).

## Mensajes

### Texto

```bash
curl -X POST "https://{{EVO_HOST}}/message/sendText/{{INSTANCE}}" \
  -H "apikey: {{API_KEY}}" \
  -H "Content-Type: application/json" \
  -d '{
    "number": "{{NUMERO_E164}}",
    "text": "Hola desde Evolution API"
  }'
```

| Campo | Detalle |
|---|---|
| `number` | Destino en E.164 sin `+` (ej `5491133334444`) |
| `text` | Cuerpo del mensaje |

### Media (imagen, video, documento, audio)

```bash
curl -X POST "https://{{EVO_HOST}}/message/sendMedia/{{INSTANCE}}" \
  -H "apikey: {{API_KEY}}" \
  -H "Content-Type: application/json" \
  -d '{
    "number": "{{NUMERO_E164}}",
    "mediatype": "image",
    "media": "https://midominio.com/img.jpg",
    "caption": "Mirá esto"
  }'
```

| Campo | Detalle |
|---|---|
| `mediatype` | `image`, `video`, `document`, `audio` |
| `media` | URL pública o base64 |
| `caption` | Opcional |
| `fileName` | Para documentos |

### Audio como nota de voz

```bash
curl -X POST "https://{{EVO_HOST}}/message/sendWhatsAppAudio/{{INSTANCE}}" \
  -H "apikey: {{API_KEY}}" \
  -H "Content-Type: application/json" \
  -d '{ "number": "{{NUMERO_E164}}", "audio": "https://.../audio.mp3" }'
```

### Botones e interactivos

Evolution soporta envío de botones y listas, pero con **soporte limitado e inestable** respecto de la Cloud API (es una emulación, no la API oficial de interactivos). Endpoints como `/message/sendButtons` y `/message/sendList` existen, pero su comportamiento depende de la versión de WhatsApp del lado servidor y puede romperse. Para interactivos confiables: Cloud API.

### Ubicación, contacto, reacción

| Acción | Path |
|---|---|
| Ubicación | `POST /message/sendLocation/{instance}` |
| Contacto | `POST /message/sendContact/{instance}` |
| Reacción | `POST /message/sendReaction/{instance}` |

## Chat

| Acción | Método + Path |
|---|---|
| Verificar si un número tiene WhatsApp | `POST /chat/whatsappNumbers/{instance}` |
| Marcar como leído | `POST /chat/markMessageAsRead/{instance}` |
| Buscar contactos | `POST /chat/findContacts/{instance}` |
| Buscar mensajes | `POST /chat/findMessages/{instance}` |
| Buscar chats | `POST /chat/findChats/{instance}` |
| Presence (typing) | `POST /chat/sendPresence/{instance}` |

Verificar número antes de enviar evita mensajes a números sin WhatsApp:

```bash
curl -X POST "https://{{EVO_HOST}}/chat/whatsappNumbers/{{INSTANCE}}" \
  -H "apikey: {{API_KEY}}" \
  -H "Content-Type: application/json" \
  -d '{ "numbers": ["{{NUMERO_E164}}"] }'
```

## Webhook

| Acción | Método + Path |
|---|---|
| Configurar webhook de la instancia | `POST /webhook/set/{instance}` |
| Consultar webhook | `GET /webhook/find/{instance}` |

```bash
curl -X POST "https://{{EVO_HOST}}/webhook/set/{{INSTANCE}}" \
  -H "apikey: {{API_KEY}}" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://n8n.../webhook/evolution",
    "events": ["MESSAGES_UPSERT", "CONNECTION_UPDATE"],
    "webhook_by_events": false
  }'
```

## Settings

| Acción | Método + Path |
|---|---|
| Configurar settings de la instancia | `POST /settings/set/{instance}` |

Settings ajustables: rechazar llamadas, marcar mensajes como leídos automáticamente, ignorar grupos, etc.

## Respuesta típica de envío

```json
{
  "key": {
    "remoteJid": "5491133334444@s.whatsapp.net",
    "fromMe": true,
    "id": "BAE5..."
  },
  "message": { "conversation": "Hola desde Evolution API" },
  "messageTimestamp": "1716200000",
  "status": "PENDING"
}
```

| Campo | Para qué |
|---|---|
| `key.id` | ID del mensaje, para tracking |
| `key.remoteJid` | Destino con sufijo `@s.whatsapp.net` |
| `status` | Estado inicial |

## Diferencias con Cloud API

| Aspecto | Cloud API | Evolution API |
|---|---|---|
| Auth | `Authorization: Bearer` | `apikey` header |
| Identificador en el path | `phone_number_id` | `instanceName` |
| Número destino | `to` | `number` |
| JID | No expone JID | Usa `@s.whatsapp.net` |
| Interactivos | Robustos | Limitados / inestables |
| HSM / plantillas | Sí | No reales |
| Verificar número | Endpoint `contacts` | `/chat/whatsappNumbers` |

## El formato JID

WhatsApp internamente usa JIDs:

| Tipo | Formato |
|---|---|
| Usuario | `5491133334444@s.whatsapp.net` |
| Grupo | `...@g.us` |

Evolution los expone en los payloads. Al enviar alcanza con `number` en E.164; Evolution arma el JID.

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| `401` / `apikey` rechazada | Header mal | Usar `apikey` exacto |
| `404` instancia no existe | `instanceName` mal o no creada | Verificar con `fetchInstances` |
| Mensaje no llega | Número sin WhatsApp | Verificar con `/chat/whatsappNumbers` |
| Instancia `close` al enviar | Sesión caída | Reconectar; ver [`10-troubleshooting/evolution-desconexion.md`](../../10-troubleshooting/evolution-desconexion.md) |
| Botones no aparecen | Soporte limitado de interactivos | Usar texto, o Cloud API |
| Versión del endpoint distinta | v1 vs v2 tienen paths distintos | Usar v2; ver docs |

## Referencias

- [Evolution API Docs](https://doc.evolution-api.com/) — Verificado 2026-05-20.
- [`07-integraciones/evolution-api/instancias.md`](./instancias.md)
- [`07-integraciones/evolution-api/eventos.md`](./eventos.md)
