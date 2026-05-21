---
title: Códigos de error de la Cloud API
category: troubleshooting
tags: [errores, codigos, cloud-api, debugging]
updated: 2026-05-20
source_official:
  - https://developers.facebook.com/docs/whatsapp/cloud-api/support/error-codes
  - https://developers.facebook.com/docs/graph-api/guides/error-handling
related:
  - 10-troubleshooting/webhooks-no-llegan.md
  - 10-troubleshooting/tokens.md
  - 10-troubleshooting/rate-limits.md
  - 10-troubleshooting/plantillas-pausadas.md
audience: [integrador, dev]
---

# Códigos de error de la Cloud API

> **TL;DR:** Cada error de la Cloud API trae `code` numérico + `message` + `error_subcode` + `fbtrace_id`. Las familias más vistas: **autenticación** (190.x, 200.x), **rate limits** (4, 80007, 130429, 131056), **permisos** (10, 200, 100), **mensajes** (131x), **plantillas** (132x), **media** (1310xx). Tabla maestra abajo.

## Contexto

Cuando una request falla, Meta devuelve una estructura uniforme. Aprender a leerla ahorra horas.

```json
{
  "error": {
    "message": "...",
    "type": "OAuthException",
    "code": 190,
    "error_subcode": 460,
    "error_user_msg": "...",
    "fbtrace_id": "AbCdEf..."
  }
}
```

| Campo | Para qué sirve |
|---|---|
| `code` | Categoría general (tabla abajo) |
| `error_subcode` | Detalle dentro de la categoría |
| `error_data.details` | Información estructurada (cuando aplica) |
| `error_user_msg` | Mensaje en idioma del usuario (puede usarse para UI) |
| `fbtrace_id` | ID para reportar a soporte de Meta |

## Tabla maestra de errores

### Autenticación y permisos

| Código | Subcode típico | Significado | Causa habitual | Solución |
|---|---|---|---|---|
| 0 | — | `Authentication failed` | Token inválido al inicializar | Regenerar token |
| 3 | — | `API does not support this operation` | Endpoint mal | Verificar versión y path |
| 10 | — | `Permission denied` | Falta scope | Re-generar token con scopes correctos |
| 100 | — | `Invalid parameter` | Param mal escrito o vacío | Revisar body |
| 190 | 460 | `Session has expired` | User token expirado | Usar System User token |
| 190 | 463 | `Session is invalid` | Token revocado | Regenerar |
| 200 | — | `Permissions error` | Falta scope para el recurso | Agregar `whatsapp_business_messaging` o `_management` |

### Rate limits

| Código | Significado | Causa | Solución |
|---|---|---|---|
| 4 | `Application request limit reached` | Exceso de calls de la App | Backoff exponencial, distribuir carga |
| 80007 | `Rate limit hit` | Demasiados envíos en poco tiempo | Throttling client-side |
| 130429 | `Too many messages` | Volumen por número | Respetar tier, espaciar envíos |
| 131056 | `Pair rate limit hit` | Demasiados mensajes al **mismo** usuario | Pausar, no insistir |
| 368 | `Temporarily blocked for policies violations` | Comportamiento detectado como abuso | Revisar políticas, esperar |

### Mensajes (familia 131x)

| Código | Significado | Causa | Solución |
|---|---|---|---|
| 131000 | Generic error | Indefinido | Reintentar; si persiste, abrir ticket con `fbtrace_id` |
| 131005 | Access denied | Sin permiso para enviar | Revisar token y suscripción |
| 131008 | Required parameter missing | Body incompleto | Validar antes de enviar |
| 131009 | Parameter value invalid | Valor fuera de rango | Validar formato (E.164, IDs, etc.) |
| 131016 | Service unavailable | Caída momentánea | Reintentar con backoff |
| 131021 | Recipient cannot be sender | Te enviaste a vos mismo | Cambiar destinatario |
| 131026 | Message undeliverable | Número sin WA, en otro país que no permite, bloqueado | Validar previamente con `contacts` endpoint o saltear |
| 131031 | Account locked | Cuenta bloqueada por políticas | Revisar Business Manager, soporte |
| 131042 | Business eligibility | Cuenta no apta para iniciar conversación | Verificar Business Portfolio |
| 131045 | Tier limit reached | Llegaste al tope diario del tier | Esperar 24h o pedir aumento |
| 131047 | Re-engagement message | Sin ventana 24h, intentaste free-form | Enviar plantilla |
| 131048 | Spam rate limit hit | Calidad muy baja | Pausar campañas, recuperar calidad |
| 131049 | Meta chose not to deliver | Mensaje no entregado por algorithm de Meta | Revisar contenido y calidad |
| 131051 | Unsupported message type | Tipo no soportado en este contexto | Verificar tipo |
| 131052 | Media download error | Falla al descargar media | Reintentar |
| 131053 | Media upload error | Falla al subir | Reintentar |
| 131057 | Account in maintenance | Mantenimiento en cuenta | Esperar |

### Plantillas (familia 132x)

| Código | Significado | Causa | Solución |
|---|---|---|---|
| 132000 | Template parameter count mismatch | Variables enviadas ≠ declaradas | Contar `{{n}}` y enviar exactamente lo necesario |
| 132001 | Template does not exist | Nombre o idioma incorrecto | Verificar `name` y `language.code` |
| 132005 | Template hydrated text too long | Texto resultante excede límite | Acortar variables |
| 132007 | Template format character policy violated | Caracteres no permitidos | Limpiar (zero-width, RTL marks, etc.) |
| 132012 | Template parameter format mismatch | Variable con formato inválido (ej. URL) | Verificar contenido de variable |
| 132015 | Template is paused | Pausada por bajo engagement | Esperar reapertura automática o crear otra |
| 132016 | Template is disabled | Desactivada por reportes | Crear nueva plantilla con otra propuesta |
| 132068 | Flow template currently blocked | Flow asociado en revisión | Esperar / revisar Flow |
| 132069 | Flow template currently throttled | Flow con throttling | Bajar volumen |

### Media

| Código | Significado | Causa | Solución |
|---|---|---|---|
| 131052 | Failed to download media | URL inválida o lenta | Subir vía `/{phone_id}/media` o reintentar |
| 131053 | Failed to upload media | Tamaño / formato incorrecto | Validar antes |
| 2388023 | Unsupported media format | Codec / extensión no permitida | Convertir a formato soportado (mp4, opus, jpeg) |

Tamaños máximos resumidos:

| Tipo | Tamaño máx |
|---|---|
| Imagen | 5 MB |
| Documento | 100 MB |
| Audio | 16 MB |
| Video | 16 MB |
| Sticker | 100 KB (estático) / 500 KB (animado) |

Detalle en [`04-mensajeria/limites-media.md`](../04-mensajeria/limites-media.md) (pendiente).

### Webhooks

| Código | Significado | Causa | Solución |
|---|---|---|---|
| 100 | `Invalid parameter` en verificación | `hub.verify_token` no coincide | Revisar variable de entorno |
| 401 | Firma inválida (no es código Meta, es tu validación) | Body parseado en vez de raw | Usar raw body |
| Timeout | Tu endpoint tardó > 20s | Procesamiento sync | Responder 200 y encolar |

## Cómo manejar errores en código

### Patrón básico

```javascript
async function sendMessage(payload) {
  try {
    const res = await fetch(url, options);
    const data = await res.json();
    if (data.error) {
      handleError(data.error);
      return { ok: false, error: data.error };
    }
    return { ok: true, data };
  } catch (e) {
    // network / timeout
    return { ok: false, error: { code: 'NETWORK', message: e.message } };
  }
}

function handleError(err) {
  switch (err.code) {
    case 131047: return retryWithTemplate();
    case 131045: return scheduleForTomorrow();
    case 4:
    case 80007:
    case 130429: return backoff(err);
    case 131026: return markUnreachable();
    case 132015:
    case 132016: return useAlternativeTemplate();
    default: return logAndAlert(err);
  }
}
```

### Backoff sugerido

| Intento | Espera |
|---|---|
| 1 | 2s |
| 2 | 4s |
| 3 | 8s |
| 4 | 16s |
| 5+ | Abandonar y alertar |

Total: 30 segundos máximo. Más allá, mejor enviar a una cola de "para revisar" en vez de seguir intentando.

## Errores que NO conviene reintentar

| Código | Razón |
|---|---|
| 131008 | Falta param: no se arregla solo |
| 131009 | Valor inválido: idem |
| 131021 | Lógica del cliente |
| 131026 | Número no usable |
| 132000 | Mismatch de variables |
| 132001 | Plantilla no existe |
| 190 | Token expirado: hay que reemitir, no reintentar |

## Cómo reportar a Meta

1. Capturar el JSON completo del error, incluido `fbtrace_id`.
2. Capturar la request exacta (sin token).
3. Adjuntar timestamp UTC.
4. Abrir caso en [Direct Support](https://business.facebook.com/business/help) si tenés WABA verificada, o en el grupo del BSP si aplica.

## Errores comunes

| Síntoma operacional | Código probable | Acción |
|---|---|---|
| "El bot no responde después de la primera plantilla" | 131047 | Implementar tracking de ventana 24h |
| "La promo cortó a mitad de la campaña" | 131045 | Esperar 24h, planificar split |
| "Mando mensaje y vuelve sin status" | 131026 | Verificar que el número exista en WA |
| "Plantilla aprobada deja de funcionar" | 132015 | Revisar pausing, usar variante |
| "Errores 4 cada cierto rato" | App rate limit | Bajar paralelismo |
| "Funciona en dev y falla en prod con 200" | Permission | Token de prod sin scopes |

## Referencias

- [Error Codes · Cloud API](https://developers.facebook.com/docs/whatsapp/cloud-api/support/error-codes) — Verificado 2026-05-20.
- [Graph API Error Handling](https://developers.facebook.com/docs/graph-api/guides/error-handling) — Verificado 2026-05-20.
- [`10-troubleshooting/webhooks-no-llegan.md`](./webhooks-no-llegan.md)
- [`10-troubleshooting/plantillas-pausadas.md`](./plantillas-pausadas.md)
