---
title: "Troubleshooting: plantillas pausadas"
category: troubleshooting
tags: [plantillas, pausing, engagement, calidad]
updated: 2026-05-20
source_official:
  - https://developers.facebook.com/docs/whatsapp/business-management-api/message-templates
related:
  - 05-plantillas-hsm/pausing.md
  - 05-plantillas-hsm/aprobacion.md
  - 06-politicas-y-calidad/quality-rating.md
audience: [integrador, marketing]
---

# Troubleshooting: plantillas pausadas

> **TL;DR:** Una plantilla se pausa cuando los usuarios reaccionan mal (bloqueos, reportes, "no me interesa", baja respuesta). Estado `PAUSED`: no se puede enviar hasta que **Meta la reactive automáticamente** o el integrador la **edite** (lo que dispara nueva aprobación). Reactivación automática: típicamente 3-24h si los disparadores se resuelven.

## Síntomas

| Síntoma | Frecuencia |
|---|---|
| Error 132015 al intentar enviar | Muy común |
| Plantilla aparece en estado `PAUSED` en el panel | Confirmación |
| Llega webhook `message_template_status_update` con `PAUSED` | Si tenés el field suscrito |
| Quality del número cae junto con la pausa | Correlacionado |

## Cómo se detecta

Vía API:

```bash
curl -X GET \
  "https://graph.facebook.com/v21.0/{{WABA_ID}}/message_templates?fields=name,status,category,quality_score,last_updated_time" \
  -H "Authorization: Bearer {{ACCESS_TOKEN}}"
```

Esperado:

```json
{
  "data": [{
    "name": "promo_lanzamiento",
    "status": "PAUSED",
    "category": "MARKETING",
    "quality_score": {
      "score": "RED",
      "date": 1716200000
    },
    "last_updated_time": "2026-05-20T10:00:00+0000"
  }]
}
```

Vía webhook (si `message_template_status_update` está suscrito):

```json
{
  "field": "message_template_status_update",
  "value": {
    "event": "PAUSED",
    "message_template_id": "...",
    "message_template_name": "promo_lanzamiento",
    "message_template_language": "es_AR",
    "reason": "LOW_QUALITY_RATING"
  }
}
```

## Causas

Meta pausa cuando una o más de estas señales aparecen sostenidas:

| Señal | Peso |
|---|---|
| Tasa alta de bloqueos sobre destinatarios de esta plantilla | Muy alto |
| Tasa alta de "Reportar como spam" | Muy alto |
| Tasa baja de respuesta del destinatario | Medio |
| Tasa baja de lectura | Bajo |
| Click rate bajo en CTAs (si la plantilla tiene botones URL) | Bajo |

`quality_score` viene a nivel plantilla con `GREEN` / `YELLOW` / `RED`. `RED` sostenido → pausing.

## Diagnóstico rápido

### 1. ¿Cuándo se pausó?

`last_updated_time` en el panel y en webhook. Suele coincidir con la última campaña.

### 2. ¿Cuántos destinatarios y cuál era el segmento?

Si fueron > 1.000 con segmento amplio o lista comprada, la causa es probable. Si fueron 50 muy específicos, el problema puede ser el contenido.

### 3. ¿Qué plantilla está pausada y qué otras tenés activas?

| Patrón | Pista |
|---|---|
| Solo una plantilla pausada | Problema de **esa plantilla** (copy, segmento) |
| Varias plantillas pausadas a la vez | Problema más amplio: del número, de la audiencia, de la WABA |
| La misma plantilla pausada en varios idiomas | Estructura común mala |

### 4. ¿El quality del número cae también?

Si quality_rating del número está en `RED` o `YELLOW`, hay un problema más estructural. Ver [`06-politicas-y-calidad/quality-rating.md`](../06-politicas-y-calidad/quality-rating.md).

## Cómo recuperarse

### Opción A: esperar reactivación automática

Si la pausa fue por una campaña puntual y el resto del número está sano:

1. **No enviar más esa plantilla.**
2. Esperar 3-24h (depende de severidad).
3. Monitorear el estado: vuelve a `APPROVED`.

### Opción B: editar la plantilla

Editarla **dispara nueva revisión** (vuelve a `PENDING`):

- Cambiar copy para mejorarlo.
- Acortar.
- Quitar CTA promocional.
- Re-clasificar categoría si aplica.

Tras aprobación, se puede usar de nuevo. El histórico de pausing queda pero "reseteado".

### Opción C: crear plantilla nueva

Si la plantilla ya tiene historial complicado, conviene crear otra:

| Acción | Razón |
|---|---|
| Nuevo nombre interno | Empezar de cero |
| Copy diferente, no idéntico | Evitar que la heurística la asocie |
| Categoría correcta | Confirmar |

## Prevención

| Acción | Impacto |
|---|---|
| Opt-in real y verificable | Reduce reportes |
| Segmentación cuidadosa | Solo a quienes interactuaron recientemente |
| Cadencia controlada | Máx 1-2 mensajes Marketing por semana por usuario |
| Variar plantillas | No mandar la misma 5 veces seguidas |
| Horarios humanos | 9-21 hora local |
| Limitar batches grandes | Subir gradual |
| Excluir bouncers | Quitar de la lista a los que no responden hace meses |

## Errores comunes en código

| Síntoma | Causa | Solución |
|---|---|---|
| `132015` y reintento automático | Reintentar empeora la situación | Marcar plantilla como pausada en tu DB, no reintentar hasta confirmar reactivación |
| Pausada y reanudada manualmente sin chequear estado vía API | Decisión a ciegas | Implementar refresh periódico del estado |
| No te enteraste que pausó hasta el cliente | Webhook no suscrito | Suscribir `message_template_status_update` y alertar |
| Plantilla pausada pero el bot sigue intentando con ella | Lógica de fallback ausente | Tener plantilla alternativa para la misma intención |

## Patrón: handling de pausing en código

```javascript
async function sendTemplate(name, lang, to, params) {
  // 1. Consultar estado actual de la plantilla en tu DB / cache
  const tpl = await db.templates.findOne({ name, language: lang });

  if (tpl?.status === 'PAUSED') {
    // Buscar alternativa de la misma intención
    const fallback = await db.templates.findOne({
      intent: tpl.intent,
      language: lang,
      status: 'APPROVED'
    });
    if (fallback) return sendTemplate(fallback.name, lang, to, params);
    // Sin alternativa: marcar para revisión y no enviar
    await queue.markForReview({ to, name, reason: 'paused_no_fallback' });
    return { ok: false, reason: 'paused_no_fallback' };
  }

  // 2. Enviar normal
  const result = await wa.send({ type: 'template', template: { name, language: { code: lang }, components: params } });

  // 3. Si Meta responde 132015 igual (cache desincronizado), actualizar estado
  if (result.error?.code === 132015) {
    await db.templates.updateOne({ name, language: lang }, { status: 'PAUSED' });
    return { ok: false, reason: 'paused_now' };
  }

  return result;
}
```

## Plantillas-alternativa: patrón "intent"

Asignar a cada plantilla un `intent` lógico (`bienvenida`, `recordatorio_turno`, `confirmacion_pedido`). Tener 2-3 versiones por intent para fallback en caso de pausing.

| Intent | Versión A | Versión B | Versión C |
|---|---|---|---|
| `bienvenida` | `bienvenida_calida` | `bienvenida_breve` | `bienvenida_formal` |
| `recordatorio_turno` | `recordatorio_turno_v1` | `recordatorio_turno_v2` | — |
| `confirmacion_envio` | `confirmacion_envio_v1` | `confirmacion_envio_breve` | — |

El sender intenta A; si está paused, intenta B; etc.

## Diferencia: pausada vs deshabilitada vs rechazada

| Estado | ¿Recupera? | Acción |
|---|---|---|
| `PAUSED` | Sí, automático o por edición | Esperar o editar |
| `DISABLED` | No automático | Crear nueva |
| `REJECTED` | Sí, editando y reenviando | Editar |

## Referencias

- [Template States · Meta](https://developers.facebook.com/docs/whatsapp/business-management-api/message-templates) — Verificado 2026-05-20.
- [`05-plantillas-hsm/pausing.md`](../05-plantillas-hsm/pausing.md)
- [`05-plantillas-hsm/aprobacion.md`](../05-plantillas-hsm/aprobacion.md)
- [`06-politicas-y-calidad/quality-rating.md`](../06-politicas-y-calidad/quality-rating.md)
