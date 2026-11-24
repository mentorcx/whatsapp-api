---
title: Pricing por mensaje (PMP)
category: politicas-y-calidad
tags: [pricing, pmp, costos, marketing, utility, authentication, service]
updated: 2026-11-24
source_official:
  - https://developers.facebook.com/docs/whatsapp/pricing
  - https://business.whatsapp.com/products/business-platform-pricing
related:
  - 05-plantillas-hsm/categorias.md
  - 06-politicas-y-calidad/ventana-24h.md
  - 06-politicas-y-calidad/free-entry-points.md
audience: [integrador, marketing, comercial]
---

# Pricing por mensaje (PMP)

> **TL;DR:** Desde el **1 de julio de 2025** Meta cobra **por mensaje de plantilla entregado** (per-message pricing, PMP), ya no por conversación de 24h. Se cobran las plantillas **Marketing**, **Utility** y **Authentication**, con tarifa según categoría y país del destinatario. Los mensajes **free-form (service)** dentro de la ventana de 24h son **gratis**, y las plantillas **Utility** enviadas dentro de esa ventana también. Utility y Authentication tienen descuentos por volumen. Las tarifas cambian: consultar siempre el pricing oficial.

## Contexto

El pricing de WhatsApp cambió varias veces:

| Período | Modelo | Unidad de cobro |
|---|---|---|
| Hasta mayo 2023 | Conversation-based pricing (CBP), por quién iniciaba | Conversación de 24h, "user-initiated" vs "business-initiated" |
| Junio 2023 – junio 2025 | CBP por categoría | Conversación de 24h por categoría (Marketing, Utility, Authentication, Service) |
| Nov 2024 | Cambio dentro de CBP | Las conversaciones Service pasan a ser gratis sin límite |
| **Desde 1 jul 2025** | **Per-message pricing (PMP)** | **Cada mensaje de plantilla entregado** |

Mucha documentación de terceros (blogs, cursos, BSPs) todavía habla de "conversaciones" y del "cupo de 1.000 conversaciones gratis". Eso **ya no aplica**.

## Qué se cobra y qué es gratis

| Mensaje | Dentro de la ventana de servicio (24h) | Fuera de la ventana |
|---|---|---|
| Free-form (texto, media, interactivos) | **Gratis** | No se puede enviar |
| Plantilla **Utility** | **Gratis** | Se cobra tarifa Utility |
| Plantilla **Marketing** | Se cobra tarifa Marketing | Se cobra tarifa Marketing |
| Plantilla **Authentication** | Se cobra tarifa Authentication | Se cobra tarifa Authentication |
| Cualquier mensaje en ventana de **Free Entry Point** (72h) | **Gratis**, incluidas plantillas | — |

Reglas clave:

- Se cobra **cada plantilla entregada** (`delivered`). Diez plantillas Marketing al mismo usuario el mismo día = diez cobros. Ya no hay "una conversación cubre las 24h".
- Las plantillas que **fallan** o no se entregan no se cobran.
- La **ventana de servicio** se abre (o se renueva) cada vez que el usuario te escribe y dura 24h. Ver [`ventana-24h.md`](./ventana-24h.md).
- La categoría que se cobra es la que Meta tiene asignada a la plantilla al momento del envío. Si Meta la recategorizó, se cobra con la nueva. Ver [`05-plantillas-hsm/categorias.md`](../05-plantillas-hsm/categorias.md).

## Las categorías

| Categoría | Tarifa relativa | Gratis cuando | Cuándo usar |
|---|---|---|---|
| **Marketing** | La más alta | Solo dentro de ventana Free Entry Point | Promociones, re-engagement, contenido genérico |
| **Utility** | Intermedia | Dentro de la ventana de servicio de 24h, o Free Entry Point | Transaccional, post-acción del usuario |
| **Authentication** | Generalmente la más baja | Solo dentro de ventana Free Entry Point | OTP / códigos de verificación |
| **Service** (free-form) | **$0** | Siempre (solo se puede enviar dentro de la ventana) | Respuestas a lo que escribe el usuario |

### Authentication internacional

En algunos países Meta aplica una tarifa **Authentication-International**, más alta, cuando el negocio que envía el OTP está registrado en otro país que el destinatario. Si tu cliente manda OTPs a usuarios de otro país, revisar en la tabla oficial si ese mercado tiene tarifa internacional.

## Descuentos por volumen

Con PMP, las plantillas **Utility** y **Authentication** tienen **tiers de volumen**: a partir de ciertos umbrales de mensajes por mes y por país, la tarifa unitaria baja. El volumen se acumula a nivel de Business Portfolio. Marketing no tiene descuento por volumen.

Los umbrales y tarifas por tier se publican en el pricing oficial y se actualizan; no hardcodearlos en presupuestos de largo plazo.

## Cómo se factura

Meta envía factura mensual (o cobra a la línea de crédito del BSP) con:

- Mensajes de plantilla entregados, por categoría.
- País de destino de cada mensaje.
- Tier de volumen aplicado.
- Mensajes no facturables (`pricing.billable=false`) excluidos.

Se cobra a la tarjeta o método de pago configurado en Business Settings → Pagos. Si falla el pago, los envíos de plantillas se suspenden tras un período de gracia.

## Cómo leer el costo en webhooks

Cada `status` de mensaje saliente trae un objeto `pricing`:

```json
{
  "id": "wamid.HBgL...",
  "status": "delivered",
  "pricing": {
    "billable": true,
    "pricing_model": "PMP",
    "type": "regular",
    "category": "marketing"
  }
}
```

| Campo | Significado |
|---|---|
| `pricing.pricing_model` | `PMP` (per-message). Si ves `CBP`, es un payload viejo o un ejemplo desactualizado |
| `pricing.billable` | Si **este mensaje** va a la factura |
| `pricing.category` | Categoría de cobro real (`marketing`, `utility`, `authentication`, `service`...). Puede diferir de la declarada si Meta recategorizó |
| `pricing.type` | `regular` (se cobra), `free_customer_service` (gratis por estar en ventana de servicio) o `free_entry_point` (gratis por ventana FEP) |

El objeto `conversation` era la base del modelo CBP. Con PMP **no conviene depender de él**: puede no venir o venir solo en casos puntuales. Para costos, usar `pricing`.

Loggear estos campos por `wamid` permite armar reports propios de costos por mensaje.

## Variación por país

Las tarifas se publican por código de país del destinatario. Rangos relativos referenciales (verificar en pricing oficial):

| Mercado | Marketing | Utility | Authentication | Comentario |
|---|---|---|---|---|
| Argentina | Alto | Medio | Bajo | Mercado caro relativo |
| México | Medio | Bajo | Bajo | |
| Brasil | Medio-alto | Bajo | Bajo | |
| España | Alto | Medio | Bajo | |
| Estados Unidos | Alto | Medio | Bajo | Ver nota abajo |
| India | Bajo | Bajo | Muy bajo | Tiene tarifa Authentication-International |

**Estados Unidos:** desde abril de 2025 Meta pausó la entrega de plantillas **Marketing** a números `+1` de EE. UU. Confirmar si sigue vigente antes de planificar campañas a ese mercado.

## Patrones para optimizar costos

| Patrón | Efecto |
|---|---|
| Invitar al usuario a iniciar (CTWA, `wa.me`, QR en local) | Abre ventana de servicio: respuestas free-form y Utility gratis |
| Enviar Utility **dentro** de la ventana de 24h cuando sea posible | Utility a $0 |
| Usar Free Entry Point (CTWA, Page CTA) | 72h con todos los mensajes gratis |
| Consolidar plantillas: una con toda la info en vez de tres | Cada plantilla se cobra por separado |
| Elegir Utility sobre Marketing cuando el contenido realmente lo es | Tarifa menor y posibilidad de $0 en ventana |
| Bajar volumen de Marketing low-engagement | Menos costo + mejor calidad |
| Evitar reintentos ciegos de plantillas | Cada entrega se cobra |
| Concentrar volumen Utility/Authentication en un Portfolio | Llegar antes a tiers con descuento |

## Cálculo de presupuesto típico

Ejemplo para 1.000 clientes/mes con bot conversacional:

| Concepto | Cantidad | Tarifa ref. | Subtotal |
|---|---|---|---|
| Respuestas free-form del bot (usuarios escriben) | ~10.000 mensajes | $0 | $0 |
| Confirmaciones Utility dentro de la ventana | 500 | $0 | $0 |
| Recordatorios Utility fuera de la ventana | 800 | ~$0.02-0.05 | $16-40 |
| Plantillas Marketing (re-engagement mensual) | 1.500 | ~$0.05-0.10 | $75-150 |
| Authentication (OTP, opcional) | 0 | — | $0 |
| **Total mensual estimado en mensajes de Meta** | | | **$91-190** |

Más el costo del orquestador (n8n, OpenAI) y hosting. Para 1.000 clientes activos: orden de magnitud $200-400/mes total. Las tarifas de referencia son ilustrativas: usar la tabla oficial del país.

## Errores comunes / sorpresas en facturación

| Sorpresa | Causa | Mitigación |
|---|---|---|
| Factura más alta que con el modelo viejo | Varias plantillas por usuario por día; antes una conversación las cubría | Consolidar plantillas |
| Factura mucho más alta de lo esperado | Plantilla recategorizada a Marketing | Auditar `pricing.category` en webhooks |
| Utility cobrado aunque "el usuario había escrito" | El envío salió después de vencida la ventana de 24h | Verificar timestamp del último inbound antes de enviar |
| Cobros duplicados | Reintentos sin verificar status | Usar `wamid` y reintentar solo en errores transitorios |
| Pricing distinto entre dos números de la misma WABA | Países de destino distintos | Calcular por país, no global |
| Presupuesto basado en "1.000 conversaciones gratis" | Info del modelo CBP, ya no existe | Rehacer el cálculo por mensaje |

## Cómo monitorear

1. Loggear el objeto `pricing` de cada webhook de status, con `wamid`, país y `phone_number_id`.
2. Contar solo los status con `billable=true` (uno por mensaje; no sumar `sent` + `delivered` + `read`).
3. Cruzar contra la factura de Meta al final del mes, o contra los analytics de pricing de la WABA.
4. Alertar si el ratio Marketing / total supera un umbral (señal de uso ineficiente).

## Referencias

- [WhatsApp Business Platform Pricing · Meta](https://developers.facebook.com/docs/whatsapp/pricing) — fuente de tarifas, tiers y reglas vigentes. Pendiente de re-verificación contra la página oficial (2026-11-24).
- [Business Platform pricing · WhatsApp Business](https://business.whatsapp.com/products/business-platform-pricing)
- [`05-plantillas-hsm/categorias.md`](../05-plantillas-hsm/categorias.md)
- [`06-politicas-y-calidad/ventana-24h.md`](./ventana-24h.md)
- [`06-politicas-y-calidad/free-entry-points.md`](./free-entry-points.md)
