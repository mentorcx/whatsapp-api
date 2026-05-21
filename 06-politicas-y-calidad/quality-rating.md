---
title: Quality Rating
category: politicas-y-calidad
tags: [quality-rating, calidad, tiers, salud-numero]
updated: 2026-05-20
source_official:
  - https://developers.facebook.com/docs/whatsapp/cloud-api/guides/phone-numbers
  - https://developers.facebook.com/docs/whatsapp/cloud-api/overview
related:
  - 02-numeros-y-conexion/tiers-mensajeria.md
  - 06-politicas-y-calidad/opt-in.md
  - 05-plantillas-hsm/pausing.md
audience: [integrador, marketing, dev]
---

# Quality Rating

> **TL;DR:** El Quality Rating es un semáforo (green / yellow / red) que Meta asigna a cada número en función del comportamiento de los usuarios (bloqueos, reportes, "no me interesa"). Caer a red puede bajar tu tier de mensajería e incluso suspender el número. Se previene con opt-in real, segmentación honesta y bajando volumen cuando aparecen señales.

## Contexto

Meta evalúa la calidad de cada número en una ventana rolling de los últimos días. La métrica no depende del contenido textual: depende de **cómo reaccionan los usuarios** a tus mensajes. Es decir, una plantilla aprobada y bien escrita puede igualmente degradar el rating si la audiencia no la quiere.

Esta métrica es independiente del estado del Business Portfolio o de la verificación. Aplica número por número.

## Estados

| Estado | Semáforo | Qué significa | Impacto |
|---|---|---|---|
| **Green / High** | Verde | Pocos bloqueos y reportes | Tier normal, sin restricciones extra |
| **Yellow / Medium** | Amarillo | Señales tempranas de descontento | Advertencia, monitoreo más estrecho |
| **Red / Low** | Rojo | Tasa alta de bloqueos/reportes | Riesgo de degradación de tier y suspensión |
| **N/A** | Gris | Sin datos suficientes | Número nuevo o bajo volumen |

Después de un período sostenido en Red sin mejora, Meta puede:

- Bajar el tier (de 1K a 250, etc.).
- Marcar el número como restringido.
- Suspender envíos business-initiated.
- En casos extremos, suspender el número.

## Qué afecta el rating

| Señal | Peso | Cómo aparece |
|---|---|---|
| Bloqueo del usuario | Alto | "Bloquear contacto" |
| Reporte de spam | Muy alto | "Reportar y bloquear" |
| Botón "No me interesa" | Medio | En mensajes promocionales |
| Bajo engagement | Bajo (pero acumula) | Usuario no responde nunca |
| Mensajes entregados pero no leídos | Bajo | Si nunca se leen, sospecha |

**No** afecta directamente (aunque puede correlacionar):

- Que la conversación quede sin respuesta del negocio.
- Tiempo de respuesta.
- Que el usuario ignore el mensaje (sin bloquear).

## Cómo consultar el rating actual

Endpoint Graph API:

```bash
curl -X GET \
  "https://graph.facebook.com/v21.0/{{PHONE_NUMBER_ID}}?fields=quality_rating,messaging_limit_tier,name_status,display_phone_number" \
  -H "Authorization: Bearer {{ACCESS_TOKEN}}"
```

Respuesta esperable:

```json
{
  "quality_rating": "GREEN",
  "messaging_limit_tier": "TIER_1K",
  "name_status": "APPROVED",
  "display_phone_number": "+54 9 11 ...",
  "id": "{{PHONE_NUMBER_ID}}"
}
```

| Campo | Valores posibles |
|---|---|
| `quality_rating` | `GREEN`, `YELLOW`, `RED`, `UNKNOWN` |
| `messaging_limit_tier` | `TIER_250`, `TIER_1K`, `TIER_10K`, `TIER_100K`, `TIER_UNLIMITED` |
| `name_status` | `APPROVED`, `PENDING_REVIEW`, `DECLINED`, `EXPIRED`, `NONE` |

También llega como webhook si está suscrito el evento `account_update` / `phone_number_quality_update`.

## Webhook de cambio de calidad

Suscribiendo `phone_number_quality_update` en la WABA, Meta avisa cuando hay degradación:

```json
{
  "entry": [{
    "id": "{{WABA_ID}}",
    "changes": [{
      "field": "phone_number_quality_update",
      "value": {
        "display_phone_number": "+54 9 11 ...",
        "event": "FLAGGED",
        "current_limit": "TIER_1K"
      }
    }]
  }]
}
```

Esto debería disparar una alerta interna (Slack, email) para que el equipo investigue antes de seguir enviando.

## Cómo prevenir caídas de calidad

1. **Opt-in real**: el usuario tiene que haber dicho explícitamente que quiere recibir mensajes. Ver [`opt-in.md`](./opt-in.md).
2. **Segmentación**: no enviar Marketing a quienes no respondieron las dos últimas veces.
3. **Frecuencia razonable**: máximo 1-2 mensajes Marketing por semana por usuario.
4. **Horarios humanos**: enviar entre 9 y 21 hora local del usuario.
5. **Plantillas variadas**: no enviar exactamente la misma plantilla 5 veces seguidas a la misma persona.
6. **Botón de opt-out claro**: incluso si no es obligatorio en tu región, ayuda al rating.
7. **Calentar números nuevos**: empezar con bajo volumen y subir progresivamente.
8. **Monitor diario** del `quality_rating` durante campañas grandes.

## Cómo recuperarse de Yellow / Red

Si el rating cayó:

1. **Pausar inmediatamente** envíos business-initiated no críticos.
2. Analizar qué plantilla / segmento disparó la caída (correlacionar con tu envío más reciente).
3. Limpiar audiencia: remover contactos sin engagement de los últimos 30 días.
4. Revisar opt-in: ¿realmente todos los destinatarios consintieron?
5. Mantener bajo volumen 7-14 días enviando solo a usuarios activos.
6. Reanudar gradualmente.

Yellow suele revertir en pocos días si bajás volumen y mejorás segmentación. Red puede tomar más tiempo y deja marca: conviene cuidarlo activamente.

## Tiers de mensajería

El rating está acoplado con el [tier de mensajería](../02-numeros-y-conexion/tiers-mensajeria.md): cuántas conversaciones business-initiated nuevas se pueden iniciar por día.

| Tier | Conversaciones nuevas / 24h |
|---|---|
| `TIER_250` | 250 |
| `TIER_1K` | 1,000 |
| `TIER_10K` | 10,000 |
| `TIER_100K` | 100,000 |
| `TIER_UNLIMITED` | Sin límite |

Para subir de tier hace falta:

- Estado `GREEN` o `YELLOW` sostenido.
- Volumen sostenido cerca del límite del tier actual durante varios días.

Una caída a Red puede bajar el tier directamente.

## Errores comunes

| Error / Síntoma | Causa | Solución |
|---|---|---|
| Rating cae a Yellow después de una campaña | Audiencia mal segmentada o cold | Limpiar audiencia, mejorar opt-in |
| Número nuevo arranca en Red | Calentamiento agresivo | Reducir volumen, esperar |
| Rating verde pero plantillas pausadas | Pausing por engagement bajo, independiente | Ver [`05-plantillas-hsm/pausing.md`](../05-plantillas-hsm/pausing.md) |
| `quality_rating: UNKNOWN` por días | Volumen muy bajo | Normal; sin datos suficientes |
| Suspensión total de envíos | Red sostenido + reportes graves | Soporte Meta, evaluar nuevo número |

## Referencias

- [Phone Number Quality Rating · Meta](https://developers.facebook.com/docs/whatsapp/cloud-api/guides/phone-numbers) — Verificado 2026-05-20.
- [Messaging Limits · Meta](https://developers.facebook.com/docs/whatsapp/cloud-api/overview#messaging-limits) — Verificado 2026-05-20.
- [`02-numeros-y-conexion/tiers-mensajeria.md`](../02-numeros-y-conexion/tiers-mensajeria.md)
- [`06-politicas-y-calidad/opt-in.md`](./opt-in.md)
