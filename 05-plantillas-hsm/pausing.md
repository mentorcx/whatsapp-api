---
title: Pausing y reapertura de plantillas
category: plantillas-hsm
tags: [hsm, plantillas, pausing, engagement, quality]
updated: 2026-05-20
source_official:
  - https://developers.facebook.com/docs/whatsapp/business-management-api/message-templates
related:
  - 05-plantillas-hsm/aprobacion.md
  - 06-politicas-y-calidad/quality-rating.md
  - 10-troubleshooting/plantillas-pausadas.md
audience: [integrador, marketing]
---

# Pausing y reapertura de plantillas

> **TL;DR:** Meta pausa una plantilla cuando su `quality_score` cae a **RED** sostenido. Razones: bloqueos, reportes, "no me interesa", baja respuesta. Durante el pausing, `132015` al intentar enviarla. Reapertura: automática (3-24h) si los disparadores cesan, o por edición de la plantilla. Tras 3 pausings repetidos puede pasar a `DISABLED`.

## Contexto

El pausing es el "freno suave" de Meta antes de medidas más drásticas (deshabilitar plantilla, bajar tier del número, suspender envíos). Una plantilla pausada no es el fin del mundo, pero ignorar la causa lleva a `DISABLED` y a daño al rating del número.

## Quality score de plantillas

Cada plantilla tiene un score:

| Score | Significado |
|---|---|
| `GREEN` / `HIGH` | Engagement bueno |
| `YELLOW` / `MEDIUM` | Señales tempranas |
| `RED` / `LOW` | Riesgo de pausing inminente |
| `UNKNOWN` | Sin datos suficientes |

Consultar:

```bash
curl -X GET \
  "https://graph.facebook.com/v21.0/{{WABA_ID}}/message_templates?fields=name,status,quality_score" \
  -H "Authorization: Bearer {{ACCESS_TOKEN}}"
```

Webhook que avisa cambios: `message_template_quality_update` (en algunas regiones / fields adicionales).

## Cuándo se pausa

Combinación de factores en ventana corta (días):

| Señal | Peso |
|---|---|
| Tasa de bloqueos por destinatario | Muy alto |
| Reportes de spam | Muy alto |
| "No me interesa" botón nativo | Alto |
| Baja tasa de respuesta | Medio |
| Bajo CTR en URL buttons | Bajo |

Una plantilla con quality `RED` durante varios días seguidos → `PAUSED`.

## Webhook de pausing

Si `message_template_status_update` está suscrito:

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

Acción recomendada al recibirlo:

1. Marcar la plantilla en tu DB como `PAUSED`.
2. Cancelar campañas en curso que usen esa plantilla.
3. Alertar al equipo (Slack/email).
4. Auditar segmento y copy.

## Reapertura automática

Meta reactiva la plantilla si:

- Los envíos cesan o reducen drasticamente.
- Los nuevos envíos (a otra plantilla similar) no muestran los mismos patrones.
- Pasaron 3-24h desde el pausing.

Llega webhook con `event: REINSTATED`:

```json
{
  "value": {
    "event": "REINSTATED",
    "message_template_name": "promo_lanzamiento",
    ...
  }
}
```

Tu integración debe escuchar esto y actualizar el estado en DB.

## Reapertura por edición

Editar la plantilla **fuerza nueva aprobación**. Si Meta la aprueba, el `PAUSED` se limpia.

| Cambio | Efecto |
|---|---|
| Texto del body | Vuelve a `PENDING`, luego `APPROVED` (o `REJECTED`) |
| Categoría | Re-categorización + revisión |
| Botones | Idem |
| Header | Idem |

Esto reseta también el quality_score histórico.

## Cuándo dejar de insistir con una plantilla

Patrón de pausings repetidos en pocas semanas:

| Repeticiones | Recomendación |
|---|---|
| 1 | Esperar reapertura, mejorar segmentación |
| 2 | Revisar copy, hacer cambios menores |
| 3+ | Crear plantilla **nueva** con otro nombre, diferente angle |

Plantillas con historial muy malo pueden pasar a `DISABLED` definitivo.

## Estados extras

| Estado | Significado | ¿Recupera? |
|---|---|---|
| `PAUSED` | Pausada por engagement bajo | Sí, auto o por edición |
| `LIMITED_PAUSED` (rare) | Pausing parcial | Sí |
| `DISABLED` | Deshabilitada permanentemente | No |
| `FLAGGED` | Marcada para review | Variable |

## Prevención: cómo no pausar

### Calidad de la audiencia

| Acción | Impacto |
|---|---|
| Opt-in real, no listas compradas | Alto |
| Excluir usuarios inactivos > 60 días | Alto |
| Segmentar por intereses reales | Medio |
| Re-confirmar opt-in cada N meses | Medio |

### Copy y diseño

| Acción | Impacto |
|---|---|
| Variar el copy entre campañas | Medio-alto |
| Plantillas con valor real para el usuario | Alto |
| Evitar tono comercial agresivo | Medio |
| Personalización con `{{1}}` = nombre | Medio |
| CTAs claros y específicos | Medio |

### Operación

| Acción | Impacto |
|---|---|
| Cadencia controlada (máx 1-2/semana Marketing) | Alto |
| Horarios humanos (9-21 local) | Medio |
| Batches escalonados (no 10K en 5 minutos) | Alto |
| A/B testing antes de masivo | Alto |
| Monitor de quality_score por plantilla diario | Crítico |

## Patrón: rotación de plantillas

Para Marketing recurrente, **rotar entre 3-5 variantes** por intent. Reduce fatiga del audience y dispara menos pausings.

| Semana | Plantilla activa |
|---|---|
| 1 | `promo_v1` |
| 2 | `promo_v2` |
| 3 | `promo_v3` |
| 4 | `promo_v1` (segunda vuelta) |

Si una entra en YELLOW, sacarla de la rotación.

## Patrón: fallback automático en código

```javascript
async function sendWithFallback(intent, lang, to, params) {
  const candidates = await db.templates.find({
    intent,
    language: lang,
    status: 'APPROVED'
  }).sort({ quality_score: -1, updated_at: -1 });

  for (const tpl of candidates) {
    const result = await wa.send({
      type: 'template',
      template: { name: tpl.name, language: { code: lang }, components: params }
    });
    if (result.ok) return result;
    if (result.error?.code === 132015) {
      await db.templates.updateOne({ _id: tpl._id }, { status: 'PAUSED' });
      continue; // probar la siguiente
    }
    return result;
  }

  return { ok: false, reason: 'no_template_available' };
}
```

## Auditoría periódica de plantillas

Job diario que:

1. Llama `GET /{{WABA_ID}}/message_templates?fields=name,status,quality_score`.
2. Compara con la DB local.
3. Actualiza estados.
4. Alerta si:
   - Alguna pasó a `PAUSED` o `RED`.
   - Tasa de pausings > N por semana.
   - Quality_score promedio cae.

## Métricas a trackear

| Métrica | Cómo |
|---|---|
| % plantillas en `RED` | Cuántas con quality bajo |
| % plantillas activas vs pausadas | Salud general |
| Tiempo medio de pausing | Cuánto duran pausadas |
| Tasa de re-pausing post-reapertura | Indica copy malo persistente |
| Bloqueos por 1000 envíos por plantilla | Métrica leading |

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| Bot insiste en plantilla `PAUSED` | Sin tracking de estado | Implementar audit y fallback |
| Plantilla recién creada se pausa rápido | Audiencia muy fría | Calentar con segmento pequeño primero |
| Pausing cíclico | Mismo problema cada vez | Re-clasificar como Marketing si no lo era; cambiar copy |
| `DISABLED` repentino | Demasiados pausings o reporte grave | Crear plantilla nueva con angle diferente, revisar opt-in |

## Referencias

- [Quality Score · Templates](https://developers.facebook.com/docs/whatsapp/business-management-api/message-templates) — Verificado 2026-05-20.
- [`10-troubleshooting/plantillas-pausadas.md`](../10-troubleshooting/plantillas-pausadas.md)
- [`06-politicas-y-calidad/quality-rating.md`](../06-politicas-y-calidad/quality-rating.md)
- [`05-plantillas-hsm/aprobacion.md`](./aprobacion.md)
