---
title: Tiers de mensajería
category: numeros-y-conexion
tags: [tiers, messaging-limits, quality-rating, escalado]
updated: 2026-05-20
source_official:
  - https://developers.facebook.com/docs/whatsapp/cloud-api/overview
related:
  - 06-politicas-y-calidad/quality-rating.md
  - 06-politicas-y-calidad/pricing.md
audience: [integrador, marketing, dev]
---

# Tiers de mensajería

> **TL;DR:** Cada número tiene un **tier** que limita cuántas conversaciones **business-initiated nuevas** podés iniciar por día (250 → 1K → 10K → 100K → ilimitado). Las conversaciones reactivas (usuario inició) no cuentan contra el tier. Subís de tier sosteniendo Quality Rating verde y volumen cerca del límite.

## Contexto

El tier existe para proteger a los usuarios de spam y para que números nuevos no envíen miles de mensajes el día 1. Saberlo bien evita campañas que se cortan a la mitad y "número que no envía más" en plena promoción.

## Tabla de tiers

| Tier | Conversaciones nuevas / 24h | Cuándo aplica |
|---|---|---|
| `TIER_50` | 50 | Números muy nuevos (algunos mercados) |
| `TIER_250` | 250 | Estado inicial post-verificación |
| `TIER_1K` | 1,000 | Tras volumen sostenido y calidad ok |
| `TIER_10K` | 10,000 | Tras volumen sostenido y calidad ok |
| `TIER_100K` | 100,000 | Operación mediana-grande |
| `TIER_UNLIMITED` | Sin límite | Cuentas grandes con quality consolidada |

**Verificado: 2026-05-20** contra docs de Meta. Tiers exactos pueden variar por región / cambios de política.

## Qué cuenta y qué no

| Acción | ¿Cuenta contra el tier? |
|---|---|
| Plantilla Marketing enviada a usuario nuevo en 24h | Sí |
| Plantilla Utility / Authentication a usuario nuevo en 24h | Sí |
| Responder dentro de la ventana de 24h | No |
| Free Entry Point (CTWA) | No |
| Mensaje al mismo usuario en una conversación ya abierta | No |

**Unidad: conversación, no mensaje.** Una conversación = todos los mensajes intercambiados con un usuario dentro de 24h.

## Cómo consultar tu tier

```bash
curl -X GET \
  "https://graph.facebook.com/v21.0/{{PHONE_NUMBER_ID}}?fields=quality_rating,messaging_limit_tier" \
  -H "Authorization: Bearer {{ACCESS_TOKEN}}"
```

Respuesta esperable:

```json
{
  "quality_rating": "GREEN",
  "messaging_limit_tier": "TIER_1K",
  "id": "{{PHONE_NUMBER_ID}}"
}
```

## Cómo subir de tier

Meta evalúa automáticamente. Para subir:

1. **Estado de calidad** en `GREEN` o `YELLOW` sostenido.
2. **Volumen real cerca del límite actual** durante varios días seguidos (heurística: ~80-90% del tier).
3. **Sin reportes graves** de spam.

No hay botón de "pedir subida": es automático y opaco. Tipicamente:

| Salto | Tiempo típico |
|---|---|
| 250 → 1K | 1-3 días con volumen cerca de 250 + green |
| 1K → 10K | 1-2 semanas |
| 10K → 100K | Varias semanas |
| 100K → unlimited | Meses, requiere historial limpio |

## Cómo bajar de tier (lo que querés evitar)

| Disparador | Efecto |
|---|---|
| Quality cae a `RED` | Baja al tier inmediato inferior |
| Pico de bloqueos / reportes | Baja inmediata posible |
| Inactividad sostenida | Puede recortar tier hacia abajo |
| Plantillas pausadas en cadena | Indicador de bajo engagement, riesgo de baja |

Una vez que bajaste, recuperar suele ser más lento que llegar la primera vez.

## Patrones de uso para escalar sano

### Calentamiento (warming) de número nuevo

Días 1-3:

- Enviar < 50 plantillas/día solo a usuarios opt-in confirmados.
- Mantener ratio inbound/outbound balanceado.

Días 4-10:

- Subir progresivamente a 200-300/día si calidad sigue green.
- Variar plantillas, no repetir la misma.

Día 10+:

- Volumen normal según tier disponible.

### Distribuir picos

Si la campaña planificada supera tu tier diario:

- Programar envíos a lo largo del día (no ráfaga de 1K en 10 minutos).
- Priorizar a usuarios con historial de respuesta.
- Spillover al día siguiente para los que no llegaron.

### Multi-número

Si necesitás más capacidad de la que un solo número da:

- Agregar números a la misma WABA (multi-número soportado).
- Cada uno con su tier propio.
- Repartir audiencia por hashing del número del usuario para consistencia.
- Cuidado: cada número tiene **su propio display name y quality rating**.

## Pricing y tier no son lo mismo

| Concepto | Lo que mide | Cambio |
|---|---|---|
| Tier | Cuántas conversaciones nuevas/día | Limita volumen |
| Pricing | Costo por conversación | No limita, solo cobra |

Podés estar en `TIER_UNLIMITED` y aun así pagar por cada conversación Marketing. Ver [`06-politicas-y-calidad/pricing.md`](../06-politicas-y-calidad/pricing.md).

## Webhook de cambios

Suscribiendo el field correspondiente, Meta avisa por webhook cuando hay un cambio de tier:

```json
{
  "entry": [{
    "id": "{{WABA_ID}}",
    "changes": [{
      "field": "business_capability_update",
      "value": {
        "max_daily_conversation_per_phone": 10000,
        "max_phone_numbers_per_business": 25
      }
    }]
  }]
}
```

Disparar alerta interna cuando esto baja.

## Errores comunes

| Error / Síntoma | Causa | Solución |
|---|---|---|
| Error 131045 / messaging limit reached | Llegaste al tope diario del tier | Esperar 24h o reducir volumen |
| Campaña corta a la mitad | Estimación de tier incorrecta | Consultar tier antes de planificar |
| Tier baja sin razón aparente | Caída de quality silenciosa | Suscribir `phone_number_quality_update` y `business_capability_update` |
| No subo de tier aunque envío parejo | Volumen muy por debajo del tier actual | Mantener cerca del tope |
| Número nuevo arranca en `TIER_50` y no sube | Quality `UNKNOWN` o `YELLOW` | Mejorar segmentación, esperar |

## Referencias

- [Messaging Limits · Meta](https://developers.facebook.com/docs/whatsapp/cloud-api/overview#messaging-limits) — Verificado 2026-05-20.
- [Quality Rating Guide](https://developers.facebook.com/docs/whatsapp/cloud-api/guides/phone-numbers) — Verificado 2026-05-20.
- [`06-politicas-y-calidad/quality-rating.md`](../06-politicas-y-calidad/quality-rating.md)
- [`06-politicas-y-calidad/pricing.md`](../06-politicas-y-calidad/pricing.md)
