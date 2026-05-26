---
title: "BSP: Twilio"
category: formas-de-uso-bsps
tags: [twilio, bsp, cloud-api, pay-as-you-go]
updated: 2026-05-20
source_official:
  - https://www.twilio.com/docs/whatsapp
related:
  - 03-formas-de-uso/comparativa.md
  - 03-formas-de-uso/cloud-api-oficial.md
audience: [integrador, comercial, dev]
---

# BSP: Twilio

> **TL;DR:** Twilio es un BSP de WhatsApp con modelo **pay-as-you-go** (sin suscripción fija de plataforma, solo se paga lo que se usa). Provee acceso oficial a la Cloud API con su propia abstracción de API. Útil cuando se quiere billing consolidado con otros canales (SMS, voz, email) o cuando el integrador ya usa Twilio para otra cosa.

## Contexto

Twilio es la opción de BSP más conocida globalmente. Diferencias clave vs Cloud API directa: hay un costo de plataforma (markup sobre el de Meta), pero a cambio se obtiene una API uniforme, soporte y la consolidación con otros canales de Twilio.

## Cuándo conviene Twilio

| Caso | Razón |
|---|---|
| El cliente / integrador ya usa Twilio para SMS o voz | Consolidación de cuenta y billing |
| Necesita SLA de plataforma | Twilio ofrece SLAs en planes empresariales |
| Pay-as-you-go sin compromiso mensual | Bueno para volumen variable |
| Setup rápido sin pasar por Meta directo | Twilio simplifica algunos pasos |
| Sandbox de WhatsApp para testing | Twilio tiene un sandbox sencillo |

## Cuándo no

| Caso | Mejor |
|---|---|
| Alto volumen sostenido | Cloud API directa (sin markup) |
| Equipo técnico con tiempo | Cloud API directa |
| Único canal es WhatsApp | Cloud API directa o Kommo/Respond.io |

## Diferencias con Cloud API directa

| Aspecto | Cloud API directa | Twilio |
|---|---|---|
| Auth | Bearer token de Meta | Account SID + Auth Token de Twilio |
| Endpoints | `graph.facebook.com/.../messages` | `api.twilio.com/.../Messages` |
| Webhooks | Configurados en Meta | Configurados en Twilio (que reenvía o no a Meta) |
| Pricing | Meta directo | Meta + markup de Twilio |
| Plantillas | Creadas en Meta o vía Twilio | Idem |
| Sandbox | Test number de Meta | Sandbox de Twilio (más simple) |

## Setup básico

1. Crear cuenta Twilio.
2. Pedir habilitación de WhatsApp Business API.
3. Asociar el número (Twilio guía el proceso con Meta).
4. Obtener Account SID y Auth Token desde la consola.
5. Configurar webhook hacia tu backend / n8n.

## Enviar un mensaje (Twilio API)

```bash
curl -X POST "https://api.twilio.com/2010-04-01/Accounts/{{ACCOUNT_SID}}/Messages.json" \
  --user "{{ACCOUNT_SID}}:{{AUTH_TOKEN}}" \
  --data-urlencode "From=whatsapp:+{{TWILIO_NUMBER}}" \
  --data-urlencode "To=whatsapp:+{{DESTINATION}}" \
  --data-urlencode "Body=Hola desde Twilio"
```

| Detalle | Notas |
|---|---|
| Prefijo | `whatsapp:` en `From` y `To` |
| Auth | Basic Auth con SID + token |
| Formato | `application/x-www-form-urlencoded` |

## Plantillas en Twilio

Twilio expone las plantillas con su propia abstracción (Content API) o se siguen creando en Meta y sincronizan. Para enviar plantilla:

```bash
--data-urlencode "ContentSid={{CONTENT_SID}}"
```

`ContentSid` reemplaza al `template name` de Meta directo.

## Webhooks

Twilio envía POSTs `application/x-www-form-urlencoded` a tu URL configurada con campos como `From`, `To`, `Body`, `MessageSid`. Estructura distinta a la de Meta.

| Campo | Equivalente Cloud API |
|---|---|
| `MessageSid` | `wamid` |
| `From` | `value.messages[].from` (con prefijo `whatsapp:`) |
| `Body` | `value.messages[].text.body` |

n8n: usar un parser específico para Twilio.

## Pricing

| Concepto | Cobro |
|---|---|
| Conversaciones | Twilio cobra: precio de Meta + markup |
| Sin suscripción de plataforma | Pay-as-you-go |
| SMS / voz / email | Cada uno con su pricing |

Twilio tiende a ser más caro que Cloud API directa a escala, pero más simple a baja escala.

## Sandbox

Twilio Sandbox para WhatsApp: número compartido al que los usuarios se "suscriben" enviando una palabra clave. Útil para desarrollo sin verificación. **No usar en producción.**

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| `From` rechazado | Olvidaste el prefijo `whatsapp:` | Agregar |
| Mensaje fuera de ventana de 24h | Igual que Cloud API: requiere plantilla | Usar `ContentSid` |
| Webhook con campos en formato distinto | Es Twilio, no Meta | Parser específico |
| Auth falla | SID y token mal | Verificar credenciales |

## Migrar de Twilio a Cloud API directa

Si en algún momento el volumen justifica eliminar el markup:

1. Crear App de Meta + System User.
2. Migrar el número de la WABA gestionada por Twilio a la del integrador (proceso vía Meta).
3. Reapuntar webhooks y reescribir el cliente API.
4. Validar plantillas (suelen mantenerse).

## Referencias

- [Twilio · WhatsApp](https://www.twilio.com/docs/whatsapp) — Verificado 2026-05-20.
- [Twilio Content API](https://www.twilio.com/docs/content) — Verificado 2026-05-20.
- [`03-formas-de-uso/comparativa.md`](../comparativa.md)
- [`03-formas-de-uso/cloud-api-oficial.md`](../cloud-api-oficial.md)
