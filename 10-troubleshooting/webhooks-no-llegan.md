---
title: "Troubleshooting: webhooks no llegan"
category: troubleshooting
tags: [webhooks, debugging, hmac, suscripcion]
updated: 2026-05-20
related:
  - 02-numeros-y-conexion/webhooks.md
  - 10-troubleshooting/codigos-error.md
audience: [integrador, dev]
---

# Troubleshooting: webhooks no llegan

> **TL;DR:** Checklist: 1) GET de verificación pasó, 2) WABA suscrita a la App vía `subscribed_apps`, 3) Field `messages` activo en la App, 4) URL es HTTPS pública y responde 200 < 20s, 5) Firewall no bloquea, 6) No estás filtrando IPs. Si todo eso está y igual no llega, el problema es del lado de Meta: revisar logs del panel.

## Síntomas

| Síntoma | Frecuencia |
|---|---|
| No llega ningún webhook | Muy común al inicio |
| Llega el primer mensaje y nada más | Suscripción parcial |
| Llegan webhooks de `messages` pero no de `statuses` | Field específico no suscrito |
| Webhooks llegan duplicados | Sin dedup |
| Webhooks llegan desordenados | Comportamiento esperado, no es bug |
| Llegan algunos, otros se pierden | Endpoint inestable o timeout |
| Llegan pero la firma falla siempre | Body parseado en vez de raw |

## Diagnóstico paso a paso

### 1. ¿Pasó el GET de verificación?

Cuando configuraste la URL en Meta, debe haber pasado el handshake inicial:

```
GET /webhook?hub.mode=subscribe&hub.verify_token={{VERIFY_TOKEN}}&hub.challenge=XXX
```

Verificar:

| Check | Cómo |
|---|---|
| Tu endpoint responde con `hub.challenge` como text/plain | Probar con `curl` desde fuera |
| `verify_token` coincide entre tu env y Meta | Comparar literalmente |
| Status 200, no 301 / 302 | Sin redirects |
| Content-Type `text/plain` | No JSON |

Test:

```bash
curl -v "https://tudominio.com/webhook/wa?hub.mode=subscribe&hub.verify_token=MI_TOKEN&hub.challenge=12345"
```

Esperado: `200 OK` con body `12345`.

### 2. ¿La WABA está suscrita a la App?

Es el error más frecuente. **Tener la App con webhook configurado no alcanza**: hay que suscribir la WABA puntualmente.

```bash
# Verificar suscripciones actuales
curl -X GET \
  "https://graph.facebook.com/v21.0/{{WABA_ID}}/subscribed_apps" \
  -H "Authorization: Bearer {{ACCESS_TOKEN}}"
```

Esperado:

```json
{
  "data": [{
    "whatsapp_business_api_data": {
      "name": "Tu App",
      "id": "{{APP_ID}}"
    }
  }]
}
```

Si `data` está vacío:

```bash
# Suscribir
curl -X POST \
  "https://graph.facebook.com/v21.0/{{WABA_ID}}/subscribed_apps" \
  -H "Authorization: Bearer {{ACCESS_TOKEN}}"
```

### 3. ¿Los fields correctos están activos?

En Meta → App → Webhooks → WhatsApp Business Account → ver fields suscritos.

| Field | Para qué |
|---|---|
| `messages` | Mensajes entrantes + statuses salientes |
| `message_template_status_update` | Plantillas aprobadas/rechazadas/pausadas |
| `phone_number_quality_update` | Quality rating cambios |
| `account_update` | Cambios cuenta |
| `template_category_update` | Recategorización |

Si no está `messages` → no llegan mensajes. Suscribir.

### 4. ¿La URL es accesible públicamente?

| Check | Detalle |
|---|---|
| HTTPS válido | Certificado público, no self-signed |
| DNS resuelve | `dig tudominio.com` |
| No detrás de VPN o LAN | Internet abierto |
| Sin Basic Auth ni auth previa | Meta no se autentica |
| Firewall permite HTTPS desde IPs de Meta | Sin restricciones por IP |

Test desde una IP arbitraria:

```bash
curl -X POST "https://tudominio.com/webhook/wa" \
  -H "Content-Type: application/json" \
  -d '{}'
```

Esperado: 200 OK (aunque sea con cuerpo vacío; lo que importa es que llegue).

### 5. ¿El endpoint responde a tiempo?

Meta espera **≤ 20 segundos**. Si tarda más, retry.

| Causa de lentitud | Solución |
|---|---|
| Procesamiento sync (llamás a OpenAI antes de responder) | Responder 200 inmediato y procesar async |
| n8n en single-process mode | Activar queue mode |
| DB sin pool | Aumentar conexiones |
| Webhook hace una llamada externa antes de 200 | Mover a after-response |

### 6. ¿Estás validando firma correctamente?

Si la firma falla, vos rechazás el evento, pero **el log del lado de Meta lo muestra como entregado**. Resultado: Meta no reintenta pero vos no procesás.

| Check | Solución |
|---|---|
| Estás usando raw body? | Activar Raw Body en n8n / Express raw middleware |
| Estás usando el `App Secret` correcto? | No confundir con Access Token |
| Comparación segura? | `crypto.timingSafeEqual` |

Si dudás, **temporalmente** loggear el resultado de la validación y procesar igual; verás si la firma es el problema.

### 7. ¿Hay filtros por IP / WAF?

| Causa | Solución |
|---|---|
| Cloudflare con WAF estricto | Desactivar reglas de Bot Management para el path del webhook |
| Firewall con whitelist | Meta no publica IPs estables, mejor no filtrar por IP |
| Geo-blocking | Permitir IPs de EEUU y Europa donde está Meta |

### 8. Revisar el panel de Meta

App → Webhooks → debajo de cada field hay un **log reciente** con los últimos intentos y status codes. Si Meta intentó y vos no respondiste 200, aparece ahí.

## Casos puntuales

### Llegan webhooks de tu Test Number pero no del número productivo

| Causa | Solución |
|---|---|
| WABA productiva no suscrita | `subscribed_apps` POST a la WABA correcta |
| Webhook configurado en App distinta a la asociada a la WABA productiva | Vincular WABA a la App correcta |

### Llegan duplicados

| Causa | Solución |
|---|---|
| Tu endpoint no respondió 200 a tiempo, Meta reintentó | Dedup por `wamid` en Redis |
| Tenés dos webhooks configurados en distintas apps | Dejar uno solo |

### Llegan desordenados

Comportamiento esperado. Ordenar por `timestamp` y `wamid` si el orden importa.

### Llegan solo statuses, no mensajes

| Causa | Solución |
|---|---|
| `messages` no suscrito | Suscribir |
| Tipo de mensaje no soportado | Verificar campos del payload |

### Llegan mensajes pero los `statuses` no aparecen

| Causa | Solución |
|---|---|
| `messages` field cubre ambos, suscripto OK | Verificar parser, los statuses vienen en `value.statuses[]` |
| Estás filtrando por `value.messages` | Aceptar también `value.statuses` |

### Webhooks llegan pero llegan tarde (minutos)

| Causa | Solución |
|---|---|
| Tu endpoint responde lento | Optimizar primer ack |
| Cola interna de Meta en momentos de alta carga | Generalmente solo, transitorio |

## Comando útil: forzar reentregar webhooks

Meta **no** ofrece endpoint público para reenviar webhooks viejos. Si perdiste eventos, los perdiste. Mitigaciones:

- Tener idempotencia siempre.
- Tener un job que, para mensajes salientes sin status final tras N minutos, marque como "estado desconocido".
- Para mensajes entrantes, mostrar al agente que llegaron desde el lado del usuario aunque tu sistema no los tenga (si tenés CRM como Kommo o Respond.io, ellos los tienen).

## Test integral

Script de verificación:

```bash
#!/bin/bash
# verify-webhook.sh

PHONE_NUMBER_ID="..."
WABA_ID="..."
ACCESS_TOKEN="..."
WEBHOOK_URL="https://tudominio.com/webhook/wa"
VERIFY_TOKEN="..."

echo "1. GET verificación..."
curl -s "${WEBHOOK_URL}?hub.mode=subscribe&hub.verify_token=${VERIFY_TOKEN}&hub.challenge=12345"
echo ""

echo "2. Suscripciones de la WABA..."
curl -s -X GET \
  "https://graph.facebook.com/v21.0/${WABA_ID}/subscribed_apps" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" | jq .

echo "3. Estado del número..."
curl -s -X GET \
  "https://graph.facebook.com/v21.0/${PHONE_NUMBER_ID}?fields=verified_name,quality_rating,messaging_limit_tier" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" | jq .

echo "4. Enviar mensaje de prueba a vos mismo..."
# Pide reemplazar TU_NUMERO_DE_PRUEBA
```

## Referencias

- [Webhooks · Cloud API](https://developers.facebook.com/docs/whatsapp/cloud-api/webhooks) — Verificado 2026-05-20.
- [`02-numeros-y-conexion/webhooks.md`](../02-numeros-y-conexion/webhooks.md)
- [`10-troubleshooting/codigos-error.md`](./codigos-error.md)
