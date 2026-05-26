---
title: "Respond.io: channels (canales)"
category: integraciones-respond-io
tags: [respond-io, channels, whatsapp, instagram, messenger, omnicanal]
updated: 2026-05-20
source_official:
  - https://docs.respond.io/
related:
  - 07-integraciones/respond-io/setup.md
audience: [integrador, comercial]
---

# Respond.io: channels

> **TL;DR:** Respond.io conecta múltiples canales en un mismo inbox: **WhatsApp Cloud API**, **WhatsApp Business app** (QR, limitado), **Instagram**, **Messenger**, **Telegram**, **WeChat**, **LINE**, **email**, **webchat**, **SMS**. Cada canal tiene su configuración pero comparten contactos y workflows. Es el diferencial principal vs Kommo (más CRM).

## Canales disponibles

| Canal | Comentario |
|---|---|
| WhatsApp Cloud API | Oficial, recomendado |
| WhatsApp Business app | QR, limitado |
| Instagram | Mensajes directos |
| Facebook Messenger | DM y comentarios |
| Telegram | Bot vía token |
| WeChat | Para mercados asiáticos |
| LINE | Idem |
| Email | Cuenta de correo |
| Webchat | Widget para sitio web |
| SMS | Vía Twilio u otros |
| Viber, Google Business Messages, etc. | Según plan |

## Cómo se unifica un contacto

Un mismo contacto puede comunicarse por varios canales. Respond.io intenta unificar (mismo `phone`/`email`) pero a veces queda separado. Patrón:

| Acción | Cómo |
|---|---|
| Merge manual de contactos duplicados | UI |
| Política de "un contacto, todos los canales" | Configurar en settings |
| Identificador propio (`external_id`) | Si tenés CRM con ID maestro |

## Configurar el canal WhatsApp Cloud

Settings → Channels → Add → WhatsApp Business Platform.

| Paso | Detalle |
|---|---|
| Embedded signup | Login Facebook → Portfolio/WABA → número |
| Verificación del número | SMS o llamada |
| Plantillas | Sincronizadas con Meta |
| Display name | Se propone durante el alta |

## Cambiar números

Si necesitás cambiar el número que opera por un canal:

| Opción | Detalle |
|---|---|
| Migrar el número en Meta y re-vincular en Respond.io | Más limpio |
| Agregar un canal nuevo y dejar el viejo | Convivencia mientras se migra |

## Multi-canal en workflows

Los workflows pueden:

| Acción | Detalle |
|---|---|
| Disparar por mensaje entrante en cualquier canal | Trigger genérico |
| Disparar solo en un canal | Filtrar por `channel` |
| Enviar por el canal de origen | Default |
| Enviar por otro canal | Si tenés contacto del usuario allí |

Patrón: si un usuario escribió por WA y vos querés mandarle un email después, podés desde el mismo workflow (si Respond.io tiene su email también).

## Diferencias por canal (importante)

| Canal | Particularidad |
|---|---|
| WhatsApp | Ventana de 24h, plantillas (HSM), opt-in obligatorio |
| Instagram | Ventana de 24h también |
| Messenger | Ventana de 24h + message tags |
| Telegram | Bot puede enviar cuando quiera al usuario que inició |
| Email | Sin ventana |
| SMS | Pricing por mensaje, no por conversación |

WhatsApp es el más restrictivo. El workflow debe respetar sus reglas (ver [`06-politicas-y-calidad/`](../../06-politicas-y-calidad/)).

## Pricing por canal

Respond.io cobra MAU por contacto único en el mes, independiente del canal. Pero cada canal puede tener sus propios costos externos:

| Canal | Costo externo |
|---|---|
| WhatsApp | Conversaciones de Meta |
| SMS | Twilio (u otro proveedor) |
| Email | Provider (a veces incluido) |
| IG/Messenger | Sin costo de Meta para los mensajes |

## Cuándo conviene multi-canal vs sólo WA

| Caso | |
|---|---|
| Cliente solo opera por WhatsApp | Un solo canal alcanza; tal vez Respond.io es exagerado, Kommo o n8n directo basta |
| Cliente atiende por WA + IG + Messenger | Respond.io brilla |
| Cliente quiere consolidar email + WA | Respond.io o un CRM más fuerte |
| Foco en ventas con pipeline | Kommo |

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| Contacto duplicado por canal | No se unificó automáticamente | Merge manual o lógica de matching |
| Bot responde por canal equivocado | Workflow no filtra canal | Filtrar |
| Plantilla WA usada en otro canal | Las HSM son solo de WA | Adaptar el mensaje por canal |
| MAU disparado al sumar canales | Más superficie | Calcular antes |

## Referencias

- [Respond.io Docs · Channels](https://docs.respond.io/) — Verificado 2026-05-20.
- [`03-formas-de-uso/respond-io.md`](../../03-formas-de-uso/respond-io.md)
- [`07-integraciones/respond-io/setup.md`](./setup.md)
