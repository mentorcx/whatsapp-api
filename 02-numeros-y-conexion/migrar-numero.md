---
title: Migrar un número a Cloud API
category: numeros-y-conexion
tags: [migracion, numero, wa-business-app, on-premise, portabilidad]
updated: 2026-05-20
source_official:
  - https://developers.facebook.com/docs/whatsapp/cloud-api/get-started/migrate-existing-whatsapp-number-to-a-business-account
related:
  - 02-numeros-y-conexion/requisitos-numero.md
  - 03-formas-de-uso/cloud-api-oficial.md
audience: [integrador, dev]
---

# Migrar un número a Cloud API

> **TL;DR:** Tres escenarios de migración: desde la **app de WhatsApp Business**, desde la **API On-Premise** (deprecada), o entre **WABAs**. El número no puede estar en dos lados a la vez. La migración implica un downtime corto y, en el caso de la app, perder el historial de chats de la app. Avisar a los usuarios.

## Contexto

Pocos clientes arrancan de cero: muchos ya usan el número en la app de WhatsApp Business. Migrar bien evita perder el número o dejar al cliente sin comunicación.

## Escenarios

| Origen | Destino | Complejidad |
|---|---|---|
| App de WhatsApp Business | Cloud API | Media |
| App de WhatsApp (consumidor) | Cloud API | Media (similar) |
| API On-Premise (deprecada) | Cloud API | Media-alta |
| Una WABA | Otra WABA | Baja |
| Otro BSP | Tu acceso (directo o BSP) | Media |

## Escenario 1: desde la app de WhatsApp Business

El caso más común en LATAM: el cliente ya atiende desde la app.

### Qué se pierde y qué no

| Elemento | ¿Se mantiene? |
|---|---|
| El número | Sí |
| Historial de chats de la app | **No** — no migra a Cloud API |
| Contactos guardados en la app | No relevante (Cloud API no usa "contactos") |
| Display name | Se vuelve a proponer |
| Catálogo de la app | No migra; se rehace si hace falta |

**Avisar al cliente:** el historial de conversaciones de la app NO se transfiere. Si necesita ese historial, exportarlo antes.

### Pasos

1. **Preparar la WABA destino** (creada, con App vinculada).
2. **Tener acceso al número** para recibir el código de verificación.
3. En el panel de Meta, agregar el número a la WABA → inicia el proceso de migración.
4. Meta detecta que el número está en la app y pide verificación.
5. **Verificar** con el código (SMS o llamada).
6. Al completarse, el número **se desvincula de la app** automáticamente.
7. La app de WhatsApp Business en el celular del cliente queda sin ese número.
8. **Registrar** el número en Cloud API:
   ```bash
   curl -X POST \
     "https://graph.facebook.com/v21.0/{{PHONE_NUMBER_ID}}/register" \
     -H "Authorization: Bearer {{ACCESS_TOKEN}}" \
     -H "Content-Type: application/json" \
     -d '{ "messaging_product": "whatsapp", "pin": "{{PIN_6_DIGITOS}}" }'
   ```

### Downtime

Durante la migración (minutos a un par de horas), el número puede no recibir mensajes con normalidad. Hacerlo en horario de bajo tráfico.

### Two-step verification PIN

Si el número tenía verificación en dos pasos activa en la app, hay que conocer ese PIN o desactivarlo antes. El `register` de Cloud API define un PIN nuevo de 6 dígitos.

## Escenario 2: desde On-Premise API

La On-Premise API está deprecada (sunset 2025). Migrar es obligatorio para proyectos legacy.

### Pasos

1. Tener la WABA y App de Cloud API listas.
2. Usar el flujo de migración guiado de Meta (mantiene el número y, en este caso, **las plantillas a nivel WABA**).
3. Reapuntar los webhooks al nuevo endpoint.
4. Adaptar el código: los endpoints de On-Premise y Cloud difieren.
5. Verificar y registrar.

### Qué cambia en el código

| Aspecto | On-Premise | Cloud API |
|---|---|---|
| Host | Tu servidor / el del BSP | `graph.facebook.com` |
| Auth | Login + token propio | Bearer token de Meta |
| Webhooks | Configurados en tu instancia | Configurados en la App de Meta |
| Media | Endpoints distintos | `/media` de Graph API |

Plantillas: viven a nivel WABA, se conservan.

## Escenario 3: entre WABAs

Mover un número de una WABA a otra (mismo o distinto Portfolio).

### Pasos

1. En el panel: WABA origen → Phone number → **Remove** o iniciar transferencia.
2. Esperar el período de cool-down (24-72h según el caso).
3. Agregar el número a la WABA destino.
4. Re-verificar si Meta lo pide.
5. Re-suscribir webhooks, re-registrar.

**Las plantillas NO migran** entre WABAs: hay que recrearlas en la destino.

## Escenario 4: cambiar de BSP / proveedor

Si el cliente venía con otro BSP y querés traerlo a tu acceso:

1. Confirmar que el cliente es dueño de su Portfolio y WABA (si no, hay un problema de lock-in con el BSP anterior).
2. Coordinar con el BSP saliente la liberación.
3. Migrar el número a tu App / acceso.
4. Recrear plantillas si la WABA cambia.

Si el BSP anterior es dueño del Portfolio: el cliente puede no poder llevarse el número fácilmente. Lección para el futuro: cliente siempre dueño del Portfolio.

## Checklist de migración

| Ítem | ✓ |
|---|---|
| WABA y App destino listas | |
| Acceso para recibir el código de verificación | |
| Cliente avisado del downtime | |
| Cliente avisado de pérdida de historial (caso app) | |
| PIN de 2FA conocido o desactivado | |
| Horario de bajo tráfico elegido | |
| Plan B si la migración falla | |
| Webhooks listos para reapuntar | |
| Plantillas listas para recrear (si cambia WABA) | |
| PIN nuevo de 6 dígitos definido para el `register` | |

## Después de migrar

1. Verificar que el número aparece `CONNECTED` en la WABA.
2. Suscribir la WABA a la App (`subscribed_apps`).
3. Registrar el número.
4. Enviar un mensaje de prueba.
5. Proponer/confirmar el display name.
6. Recrear plantillas si la WABA es nueva.
7. Avisar al cliente que la app de WhatsApp Business ya no funciona con ese número.

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| "El número ya está en uso" | Sigue activo en la app | Completar la desvinculación de la app |
| Código de verificación no llega | Operador bloquea SMS | Probar verificación por llamada |
| 2FA bloquea la migración | PIN de la app desconocido | Desactivar 2FA en la app antes, o recuperarlo |
| Tras migrar, webhooks no llegan | WABA no suscrita | `POST /{{WABA_ID}}/subscribed_apps` |
| Número conectado pero no envía | Falta `register` con PIN | Registrar |
| Cliente perdió el historial sin saberlo | No se le avisó | Comunicar siempre antes de migrar |
| Plantillas desaparecieron | Cambió de WABA | Recrearlas en la destino |

## Referencias

- [Migrate Existing Number · Meta](https://developers.facebook.com/docs/whatsapp/cloud-api/get-started/migrate-existing-whatsapp-number-to-a-business-account) — Verificado 2026-05-20.
- [On-Premise to Cloud migration](https://developers.facebook.com/docs/whatsapp/on-premises) — Verificado 2026-05-20.
- [`02-numeros-y-conexion/requisitos-numero.md`](./requisitos-numero.md)
- [`03-formas-de-uso/cloud-api-oficial.md`](../03-formas-de-uso/cloud-api-oficial.md)
