---
title: "Respond.io: setup"
category: integraciones-respond-io
tags: [respond-io, setup, cloud-api, workspace]
updated: 2026-05-20
source_official:
  - https://docs.respond.io/
related:
  - 03-formas-de-uso/respond-io.md
  - 07-integraciones/respond-io/channels.md
audience: [integrador, comercial, dev]
---

# Respond.io: setup

> **TL;DR:** Crear workspace en Respond.io, conectar WhatsApp Cloud API vía el flujo embebido (partner oficial), invitar agentes, crear plantillas, configurar workflows. Operativo en horas. Los costos: suscripción + MAU (Monthly Active Contacts) + conversaciones de Meta.

## Pasos de alta

### 1. Crear workspace

Registrarse en [respond.io](https://respond.io/). El workspace es la cuenta de la empresa.

| Configuración inicial | Detalle |
|---|---|
| Nombre del workspace | Empresa |
| Plan | Según tamaño del equipo y MAU esperado |
| Zona horaria | Para reports |
| Idioma | UI |

### 2. Conectar WhatsApp

Settings → Channels → Add Channel → WhatsApp Business Platform.

Dos opciones:

| Opción | Detalle |
|---|---|
| WhatsApp Cloud API (oficial) | Embedded signup vía Meta dentro de Respond.io. **Recomendada.** |
| WhatsApp Business app (QR) | Conexión tipo no oficial, limitada |

Para producción: Cloud API. El flujo embebido pide login de Facebook, crear/elegir Portfolio + WABA + número, y verificar.

### 3. Invitar usuarios

Settings → Users → Invite.

Roles típicos:

| Rol | Permisos |
|---|---|
| Owner | Todo |
| Manager | Casi todo, no billing |
| Agent | Solo bandeja + lo asignado |
| Restricted Agent | Sub-conjunto |

### 4. Crear plantillas

Settings → Channels → WhatsApp → Templates → Create.

Las plantillas se aprueban con Meta (Respond.io es el front). Sincronizadas con la WABA. Categorías: Marketing, Utility, Authentication.

Ver criterios en [`05-plantillas-hsm/aprobacion.md`](../../05-plantillas-hsm/aprobacion.md).

### 5. Configurar workflows básicos

Workflows → New:

| Workflow recomendado al inicio | Detalle |
|---|---|
| Saludo de bienvenida | Trigger: first message |
| Asignación a equipo | Por palabra clave o horario |
| Auto-cierre por inactividad | Cerrar conversaciones sin actividad N días |
| Handoff a humano | Tag o palabra clave dispara asignación |

### 6. Integrar con sistemas externos (opcional)

| Integración | Para qué |
|---|---|
| HTTP request en workflows | Llamar APIs externas (n8n, CRM) |
| Zapier / Make / n8n | Sync con otros sistemas |
| Custom API | Disparar workflows desde tu backend |

## Datos importantes a configurar

### Custom fields

Crear campos custom en contactos para enriquecer:

| Campo sugerido | Tipo |
|---|---|
| `wa_opt_in` | Boolean |
| `intent` | Single select |
| `source` | Text (ad, organic, etc.) |
| `customer_segment` | Single select |

### Tags

Tags para segmentar conversaciones / contactos. Usar lista controlada, no improvisar tag por mensaje.

### Teams

Agrupar agentes por equipo (Ventas, Soporte). Routing de workflows asigna a equipos.

## Costos típicos

| Componente | Cuándo |
|---|---|
| Suscripción mensual | Por tier |
| MAU | Cada contacto único con actividad en el mes |
| Conversaciones de Meta | Aparte, paga el dueño de la WABA |
| Add-ons (AI, integraciones) | Según plan |

Calcular antes de prometer al cliente: el MAU sube rápido con bases grandes.

## Checklist post-setup

| Ítem | ✓ |
|---|---|
| Channel WhatsApp Cloud conectado y activo | |
| Plantillas core aprobadas | |
| Agentes invitados con roles correctos | |
| Workflow de saludo de bienvenida | |
| Workflow de handoff a humano | |
| Reports configurados | |
| Webhook saliente a n8n (si aplica) | |
| Auto-respuestas en horarios fuera de oficina | |

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| Channel WA no se conecta | Embedded signup interrumpido | Reintentar; revisar permisos del usuario Meta |
| Plantillas no aparecen | Sin sincronizar | Forzar sync en el channel |
| Workflow no dispara | Trigger mal configurado | Revisar condiciones del trigger |
| MAU mayor al previsto | Base activa más amplia | Revisar segmentación |

## Referencias

- [Respond.io Docs](https://docs.respond.io/) — Verificado 2026-05-20.
- [`03-formas-de-uso/respond-io.md`](../../03-formas-de-uso/respond-io.md)
- [`07-integraciones/respond-io/channels.md`](./channels.md)
