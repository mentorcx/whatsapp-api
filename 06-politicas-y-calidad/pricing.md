---
title: Conversation-based pricing
category: politicas-y-calidad
tags: [pricing, conversaciones, costos, marketing, utility, authentication, service]
updated: 2026-05-20
source_official:
  - https://developers.facebook.com/docs/whatsapp/pricing
  - https://business.whatsapp.com/products/business-platform-pricing
related:
  - 05-plantillas-hsm/categorias.md
  - 06-politicas-y-calidad/ventana-24h.md
  - 06-politicas-y-calidad/free-entry-points.md
audience: [integrador, marketing, comercial]
---

# Conversation-based pricing

> **TL;DR:** Meta cobra por **conversación** (ventana rolling de 24h con un usuario), no por mensaje. Cuatro categorías: **Marketing**, **Utility**, **Authentication**, **Service**. La tarifa varía por país. Hay un cupo de conversaciones de servicio gratuitas por mes y por número. **Verificado: 2026-05-20** — el modelo se actualiza, consultar pricing oficial siempre.

## Contexto

El pricing de WhatsApp cambió varias veces: en 2024-2025 Meta migró a "per-conversation" desde "per-message". Conviene entender la unidad de cobro antes de planificar un proyecto, porque la diferencia entre Marketing y Service es de **órdenes de magnitud** según el país.

## Unidad de cobro: la conversación

Una conversación es una **ventana rolling de 24 horas** con un mismo usuario, abierta por una acción que define la categoría.

| Categoría | Quién abre | Cómo |
|---|---|---|
| **Marketing** | Negocio | Plantilla Marketing |
| **Utility** | Negocio | Plantilla Utility |
| **Authentication** | Negocio | Plantilla Authentication |
| **Service** | Usuario | Mensaje del usuario (o respuesta dentro de ventana de 24h) |

Reglas clave:

- Una sola conversación cuenta por las 24h aunque haya 50 mensajes.
- Categorías son **mutuamente excluyentes en simultáneo**: si abrís Marketing y después enviás Utility, se abre **otra** conversación Utility (cobra ambas).
- La conversación Service abre cuando el usuario te escribe; durante esas 24h, lo que envíes en respuesta no abre otra conversación.

## Las cuatro categorías

### Marketing

| Aspecto | Detalle |
|---|---|
| Quién la abre | Negocio, con plantilla Marketing |
| Tarifa | **Más alta** |
| Cuándo usar | Promociones, re-engagement, contenido genérico |
| Opt-in obligatorio | Sí |

### Utility

| Aspecto | Detalle |
|---|---|
| Quién la abre | Negocio, con plantilla Utility |
| Tarifa | Intermedia. **En algunos países / casos: $0** si está dentro de una ventana de servicio abierta |
| Cuándo usar | Transaccional, post-acción del usuario |
| Opt-in obligatorio | Sí (implícito en muchos casos) |

### Authentication

| Aspecto | Detalle |
|---|---|
| Quién la abre | Negocio, con plantilla Authentication |
| Tarifa | Generalmente la **más baja** entre las plantillas; pricing especial por país |
| Cuándo usar | OTP / códigos de verificación |
| Restricciones | Reglas estrictas de redacción y botones |

### Service

| Aspecto | Detalle |
|---|---|
| Quién la abre | Usuario (mensaje entrante o CTWA) |
| Tarifa | **Free hasta cupo mensual**, luego tarifa baja |
| Cuándo aplica | Toda respuesta dentro de ventana 24h post-mensaje del usuario |
| Beneficio | El modelo más barato; conviene incentivar que el usuario inicie |

## Cupo gratuito mensual

Meta históricamente ofreció un cupo de conversaciones Service gratuitas por número y por mes. Detalles a revisar en pricing oficial; han cambiado.

**Reglas básicas (verificar siempre):**

- El cupo se cuenta por número, no por WABA.
- Renueva mensualmente.
- Solo aplica a Service, no a Marketing/Utility/Authentication.
- Conversaciones Free Entry Point (CTWA) pueden tener un trato especial. Ver [`free-entry-points.md`](./free-entry-points.md).

## Cómo se factura

Meta envía factura mensual con:

- Conversaciones por categoría.
- Costo por país de destino.
- Aplicación del cupo gratuito.
- Conversaciones billable (`pricing.billable=true` en webhooks).

Cobra a la tarjeta o método de pago configurado en Business Settings → Pagos. Si falla el pago, los envíos business-initiated se suspenden tras un período de gracia.

## Cómo leer el costo en webhooks

Cada `status` de mensaje saliente trae info de pricing:

```json
{
  "id": "wamid.HBgL...",
  "status": "delivered",
  "conversation": {
    "id": "abc...",
    "origin": { "type": "marketing" }
  },
  "pricing": {
    "billable": true,
    "pricing_model": "CBP",
    "category": "marketing"
  }
}
```

| Campo | Significado |
|---|---|
| `conversation.id` | Identificador de la conversación de 24h |
| `conversation.origin.type` | Categoría que la abrió |
| `pricing.billable` | Si esta conversación va a la factura |
| `pricing.category` | Categoría de cobro real (puede diferir de la declarada si Meta recategorizó) |

Loggear estos campos permite armar reports propios de costos por conversación.

## Variación por país

Las tarifas se publican por código de país de destino del mensaje. Algunos rangos referenciales (verificar pricing oficial):

| Mercado | Marketing | Utility | Authentication | Comentario |
|---|---|---|---|---|
| Argentina | Alto | Medio | Bajo | Mercado caro relativo |
| México | Medio | Bajo | Bajo | |
| Brasil | Medio-alto | Bajo | Bajo | |
| España | Alto | Medio | Bajo | |
| Estados Unidos | Alto | Medio | Bajo | |
| India | Bajo | Bajo | Muy bajo | Mercado más barato globalmente |

**Verificado: 2026-05-20** — usar pricing oficial para números exactos.

## Patrones para optimizar costos

| Patrón | Efecto |
|---|---|
| Inviar al usuario a iniciar (CTWA, `wa.me`, QR en local) | Conversaciones Service, mucho más baratas |
| Consolidar mensajes en una sola plantilla por día | 1 conversación en vez de 3 |
| Usar Free Entry Point cuando aplique | Conversación sin costo de plantilla |
| Elegir Utility sobre Marketing cuando el contenido lo permita | Tarifa menor |
| Bajar volumen de Marketing low-engagement | Menos costo + mejor calidad |
| Multi-número segmentado | Cuidar la calidad del número de marketing aparte del de service |
| Evitar reintentos automáticos cuando un envío falla | No multiplicar conversaciones |

## Cálculo de presupuesto típico

Ejemplo para 1.000 clientes/mes con bot conversacional:

| Concepto | Cantidad | Tarifa ref. | Subtotal |
|---|---|---|---|
| Conversaciones Service (usuarios escriben) | 1.000 | ~$0 (cupo) | ~$0 |
| Conversaciones Utility (confirmaciones) | 800 | ~$0.02-0.05 | $16-40 |
| Conversaciones Marketing (re-engagement mensual) | 1.500 | ~$0.05-0.10 | $75-150 |
| Conversaciones Authentication (OTP, opcional) | 0 | $0 | $0 |
| **Total mensual estimado en conversaciones** | | | **$91-190** |

Más el costo del orquestador (n8n, OpenAI) y hosting. Para 1.000 clientes activos: orden de magnitud $200-400/mes total.

## Errores comunes / sorpresas en facturación

| Sorpresa | Causa | Mitigación |
|---|---|---|
| Factura mucho más alta de lo esperado | Plantilla recategorizada a Marketing | Auditar `pricing.category` en webhooks |
| Conversaciones duplicadas | Reintentos sin verificar status | Usar `wamid` y reintentar solo en errores transitorios |
| Pricing distinto entre dos números de la misma WABA | Países de destino distintos | Calcular por país, no global |
| Free tier "agotado" antes de fin de mes | Cupo se cuenta por Service, no por todo | Verificar cómo Meta cuenta en tu país |

## Cómo monitorear

1. Loggear `pricing` y `conversation` de cada webhook de status.
2. Agrupar por `conversation.id` para evitar doble conteo.
3. Cruzar contra factura de Meta al final del mes.
4. Alertar si el ratio Marketing/Service supera un umbral (señal de uso ineficiente).

## Referencias

- [WhatsApp Business Platform Pricing · Meta](https://developers.facebook.com/docs/whatsapp/pricing) — Verificado 2026-05-20.
- [Conversation Categories](https://developers.facebook.com/docs/whatsapp/cloud-api/guides/conversations/conversation-categories) — Verificado 2026-05-20.
- [`05-plantillas-hsm/categorias.md`](../05-plantillas-hsm/categorias.md)
- [`06-politicas-y-calidad/ventana-24h.md`](./ventana-24h.md)
- [`06-politicas-y-calidad/free-entry-points.md`](./free-entry-points.md)
