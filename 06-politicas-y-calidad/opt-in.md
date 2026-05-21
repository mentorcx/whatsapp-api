---
title: Opt-in obligatorio
category: politicas-y-calidad
tags: [opt-in, consentimiento, legal, gdpr, lgpd, quality]
updated: 2026-05-20
source_official:
  - https://www.whatsapp.com/legal/business-policy
  - https://developers.facebook.com/docs/whatsapp/overview/getting-opt-in
related:
  - 06-politicas-y-calidad/quality-rating.md
  - 05-plantillas-hsm/aprobacion.md
audience: [integrador, marketing, legal]
---

# Opt-in obligatorio

> **TL;DR:** Meta exige **opt-in explícito, verificable y específico** del usuario antes de que un negocio le envíe el primer mensaje business-initiated por WhatsApp. No alcanza con tener el número en la base de clientes. Sin opt-in real, el rating cae y el número se bannea. Aceptable: formularios, checkbox en checkout, opt-in por SMS, página web con consent claro, mensaje del usuario al negocio.

## Contexto

Es el aspecto más subestimado y el que más impacta en la calidad del número. La diferencia entre una operación sana y un número quemado en 30 días es casi siempre la calidad del opt-in.

Además del riesgo de plataforma (ban / quality drop), existe el riesgo legal: GDPR (UE), LGPD (Brasil), Ley 25.326 (Argentina), CCPA (California), etc., todas exigen consentimiento informado y revocable.

## Qué exige Meta

Tres atributos del opt-in:

| Atributo | Detalle |
|---|---|
| **Explícito** | El usuario acciona algo concretamente (no es opt-in implícito por haber comprado) |
| **Específico** | Sabe que va a recibir mensajes **por WhatsApp**, no genérico "comunicaciones" |
| **Verificable** | Tu sistema puede demostrar cuándo, cómo y a qué consintió (log con timestamp, fuente, IP, copy mostrado) |

Y además:

- **Revocable**: el usuario puede pedir baja en cualquier momento, y vos debés respetarla.
- **Granular por tipo**: si vas a mandar Marketing **y** Utility, conviene declarar ambos.

## Formas válidas de obtener opt-in

| Mecanismo | Cómo se ve | Calidad del opt-in |
|---|---|---|
| Checkbox no pre-marcada en formulario | "Quiero recibir mensajes por WhatsApp de {{NEGOCIO}}" | Alta |
| Confirmación post-compra en checkout | Paso final con opción explícita | Alta |
| Link `wa.me/{{NUMERO}}?text=...` que el usuario cliquea | El usuario inicia la conversación | Muy alta (es opt-in implícito + acción explícita) |
| QR code en local físico que abre WA | Idem | Muy alta |
| CTWA (Click-to-WhatsApp Ads) | Click en anuncio abre WA | Muy alta |
| Confirmación por SMS / Email | "Respondé SÍ a este SMS para recibir por WA" | Media-alta |
| Botón en website "Recibir por WhatsApp" | Click consciente | Alta |
| Opt-in dentro de otro mensaje de WA | "¿Querés recibir nuestras novedades? Sí/No" | Alta |

## Formas inválidas (que causan problemas)

| Mecanismo | Por qué no |
|---|---|
| Lista comprada de leads | No hay consentimiento real |
| Base de clientes histórica sin opt-in específico | Aceptaron términos genéricos, no WA puntual |
| Importar contactos de la agenda del teléfono del negocio | No es consentimiento |
| "Aceptás recibir cualquier comunicación" en TyC | Demasiado genérico |
| Checkbox pre-marcada | Ilegal en muchas jurisdicciones, no aceptable para Meta |
| "Si no querés, escribí STOP" sin opt-in previo | Es opt-out implícito, no opt-in |
| Sortear premios pidiendo el número sin explicar uso | Engaño |

## Qué guardar como evidencia

| Campo | Por qué |
|---|---|
| Timestamp UTC | Cuándo consintió |
| Identificador del usuario | Quién (teléfono, email, ID) |
| Fuente | Form web, checkout, CTWA, SMS, etc. |
| Copy exacto que se le mostró | Para demostrar qué se le ofreció |
| IP o dispositivo (opcional) | Defensa adicional |
| Categoría aceptada | Marketing, Utility, ambas, etc. |
| Estado actual | `active`, `revoked`, `bounced` |

Schema mínimo recomendado:

```sql
CREATE TABLE wa_optins (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  wa_user_id TEXT NOT NULL,            -- E.164 sin +
  source TEXT NOT NULL,                -- 'web_form', 'checkout', 'ctwa', 'sms', 'manual'
  categories TEXT[] NOT NULL,          -- {'marketing','utility'}
  copy_shown TEXT NOT NULL,            -- texto exacto presentado al usuario
  consented_at TIMESTAMPTZ NOT NULL,
  revoked_at TIMESTAMPTZ,
  ip TEXT,
  user_agent TEXT,
  metadata JSONB DEFAULT '{}'::jsonb
);

CREATE INDEX idx_optins_user ON wa_optins(wa_user_id) WHERE revoked_at IS NULL;
```

## Copy recomendado

### En formulario web

```
[ ] Quiero recibir información de {{NEGOCIO}} por WhatsApp al número que indiqué arriba.
Voy a recibir mensajes sobre {{TIPO_DE_CONTENIDO}}. Podés darte de baja escribiendo "BAJA"
en cualquier momento.
```

### En checkout

```
¿Querés que te avisemos por WhatsApp sobre el estado de tu pedido?
[ ] Sí, quiero recibir notificaciones de mi pedido por WhatsApp.
```

(Notar: pidiendo solo Utility, no Marketing.)

### Post-CTWA o `wa.me` initiation

El usuario ya inició la conversación. Aprovechá ese contacto para confirmar y subir el nivel de opt-in:

```
¡Hola! Para enviarte novedades y promociones por WhatsApp en el futuro,
respondé "SÍ". Si solo querés que respondamos esta consulta, ignorá este mensaje.
```

## Manejo del opt-out

| Trigger | Acción |
|---|---|
| Usuario escribe `BAJA`, `STOP`, `NO`, `CANCELAR` (palabras clave) | Marcar `revoked_at = now()`, dejar de enviar Marketing |
| Usuario bloquea el número | Detectar por bounce de mensajes (no llega `delivered`), marcar inactivo |
| Usuario reporta el negocio | Detectar por quality drop, suspender envíos |
| Pedido por otro canal (email, llamada) | Marcar manualmente |

Tras opt-out, **respetar** y no enviar nuevamente Marketing. Utility/Authentication pueden seguir si hay base legal (ej. seguridad de la cuenta), pero conviene preguntar antes.

## Diferencias por jurisdicción

| Región | Norma | Diferencia clave |
|---|---|---|
| UE | GDPR | Consentimiento explícito, revocable, registrable. Derecho al borrado. |
| Brasil | LGPD | Similar a GDPR, ANPD como autoridad |
| Argentina | Ley 25.326 | Consentimiento expreso, base de datos registrada |
| México | LFPDPPP | Consentimiento informado |
| EEUU | TCPA (federal) + estatales | Pre-existing relationship no es opt-in suficiente para SMS/WA |
| Cualquier país | Política Meta | Aplica además de la ley local |

Recomendación práctica: aplicar el **estándar más estricto** del mercado donde opera el cliente. Sale más barato que litigar / perder número.

## Patrón de doble opt-in (opcional pero recomendado)

1. Usuario marca el checkbox en el form web.
2. Se envía un mensaje de confirmación por WhatsApp (vía plantilla Utility con CTA "Confirmar").
3. Si el usuario cliquea "Confirmar", queda `confirmed = true`.
4. Solo a partir de ahí se le envían mensajes Marketing.

Pros: rating sube, segmentación filtra inactivos.
Contras: una capa más en el funnel.

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| Quality rating cae a Yellow tras la primera campaña | Opt-in pobre (lista comprada o histórica) | Re-confirmar opt-in vía doble opt-in antes de seguir |
| Usuarios reportan "no me suscribí" | Checkbox poco visible o pre-marcada | Rediseñar con consent explícito |
| `BAJA` no se respeta | Lógica de opt-out no implementada | Trigger automático en n8n con palabras clave |
| Plantilla Marketing rechazada por "policy" | Indicios de opt-in débil en el copy | Mejorar el texto del checkbox upstream |
| Cliente trae base histórica y quiere usarla ya | Sin tiempo de re-confirmar | Doble opt-in vía Utility primero |

## Integración con CRMs

| CRM | Cómo manejar opt-in |
|---|---|
| Kommo | Campo custom en el contacto, validado antes de enviar Marketing |
| HubSpot | Subscription type específico para WhatsApp |
| Salesforce | Campo en Contact, junto con consents existentes |
| Sistema propio | Tabla dedicada como la del schema arriba |

En el workflow de envío masivo, **filtrar siempre por opt-in activo** antes de disparar plantillas Marketing.

## Referencias

- [WhatsApp Business Policy · Opt-in](https://www.whatsapp.com/legal/business-policy) — Verificado 2026-05-20.
- [Getting Opt-In · Meta](https://developers.facebook.com/docs/whatsapp/overview/getting-opt-in) — Verificado 2026-05-20.
- [`06-politicas-y-calidad/quality-rating.md`](./quality-rating.md)
- [`05-plantillas-hsm/aprobacion.md`](../05-plantillas-hsm/aprobacion.md)
