---
title: "Troubleshooting: tokens y permisos"
category: troubleshooting
tags: [token, permisos, scopes, system-user, debugging]
updated: 2026-05-20
related:
  - 01-onboarding-meta/system-users-tokens.md
  - 10-troubleshooting/codigos-error.md
audience: [integrador, dev]
---

# Troubleshooting: tokens y permisos

> **TL;DR:** Si te tira `190`, el token está vencido o revocado. Si te tira `200`, el token es válido pero le faltan **scopes** o **activos asignados** al System User. Si te tira `100` con mensaje de "Unsupported request", el endpoint o el ID son incorrectos.

## Errores típicos y qué significan

| Código | Sub-código | Significado | Causa más probable |
|---|---|---|---|
| 190 | 460 | Session expired | Token de usuario corto, no system user |
| 190 | 463 | Session invalid | Token revocado |
| 190 | 467 | Token expired | Token con expiración seteada |
| 200 | — | Permissions error | Scope faltante o activo no asignado |
| 100 | — | Invalid parameter | Endpoint o ID mal |
| 10 | — | Permission denied | Sin permiso para esa operación |
| 17 | — | User request limit | Throttling |

## Cómo inspeccionar un token

```bash
curl -X GET \
  "https://graph.facebook.com/v21.0/debug_token?input_token={{TOKEN_A_INSPECCIONAR}}&access_token={{TOKEN_A_INSPECCIONAR}}"
```

Respuesta típica:

```json
{
  "data": {
    "app_id": "...",
    "type": "SYSTEM_USER",
    "application": "Mi App",
    "data_access_expires_at": 0,
    "expires_at": 0,
    "is_valid": true,
    "scopes": [
      "whatsapp_business_messaging",
      "whatsapp_business_management",
      "business_management"
    ],
    "user_id": "..."
  }
}
```

| Campo | Para qué |
|---|---|
| `type` | `SYSTEM_USER` (correcto para prod) o `USER` (no apto) |
| `is_valid` | Si false → regenerar |
| `expires_at` | `0` = no expira; otro valor = unix timestamp |
| `scopes` | Lista de permisos otorgados |

## Diagnóstico paso a paso

### 1. ¿Tipo de token?

| `type` | Apto para prod | Comentario |
|---|---|---|
| `SYSTEM_USER` | Sí | Lo que querés |
| `USER` | No (corto plazo) | Regenerar como System User |
| `PAGE` | No (no aplica a WA) | Otro contexto |
| `APP` | No (apenas algunos endpoints) | No para mensajería |

### 2. ¿Está vencido?

Si `expires_at` no es 0 y ya pasó:

- **Solución**: regenerar token. Si usás System User, regenerar con opción "Never expires".

### 3. ¿Tiene los scopes correctos?

| Acción | Scope requerido |
|---|---|
| Enviar mensajes | `whatsapp_business_messaging` |
| Crear/editar plantillas | `whatsapp_business_management` |
| Listar/operar WABAs | `business_management` |
| Operar catálogos | `catalog_management` |
| Insights de ads (CTWA) | `ads_management` |

Si te falta uno, **regenerar el token con los scopes correctos**. No se pueden agregar scopes a un token ya emitido.

### 4. ¿El System User tiene los activos asignados?

Aunque el token tenga scope, el System User necesita **acceso a cada recurso**:

- WABA → asignar al System User con permiso `Manage` o `Send messages`.
- App → asignar con `Develop` o `Manage`.

Sin esta asignación, las requests devuelven `200` (`Permissions error`) aun teniendo scopes.

Para verificar desde el panel: Business Manager → Settings → Users → System Users → seleccionar el SU → **Assets**.

### 5. ¿Estás apuntando al recurso correcto?

| Error | Causa |
|---|---|
| Mandás a `PHONE_NUMBER_ID` viejo | Cambió el número o se removió |
| Confundís `PHONE_NUMBER_ID` con `WABA_ID` | Son distintos, no mezclar |
| Apuntás a una WABA que pertenece a otro Portfolio | Token sin acceso |

Verificar el `PHONE_NUMBER_ID` actual:

```bash
curl -X GET \
  "https://graph.facebook.com/v21.0/{{WABA_ID}}/phone_numbers" \
  -H "Authorization: Bearer {{ACCESS_TOKEN}}"
```

## Casos comunes

### "Anduvo ayer, hoy no"

Causas frecuentes:

| Causa | Cómo confirmar |
|---|---|
| Token corto (user token) que expiró | `debug_token` muestra `type: USER` |
| Token revocado | `debug_token` muestra `is_valid: false` |
| El System User fue eliminado | Buscar en panel; si no existe, recrear |
| Permisos del SU cambiaron | Verificar `Assets` del SU |
| Cambió el App Secret y eso invalidó el token | Raro pero posible |

### "Anda en local, falla en prod"

| Causa | Solución |
|---|---|
| Estás usando tu user token personal en local | Usar el mismo System User token en ambos |
| Variables de entorno distintas | Verificar `dotenv` cargado correctamente en prod |
| Token de un Portfolio de prueba en local, prod usa el real | Sincronizar tokens al deploy |
| Diferencia de scopes entre tokens | Regenerar con scopes completos |

### "Mando mensajes pero no puedo crear plantillas"

`POST /{{WABA_ID}}/message_templates` requiere `whatsapp_business_management`. Probablemente tu token tiene solo `whatsapp_business_messaging`. Regenerar.

### "Webhook valida firma pero las requests salientes fallan"

| Validación de firma | Requests salientes |
|---|---|
| Usa **App Secret** | Usa **Access Token** |

Son **dos credenciales distintas**. Ambas deben estar bien.

## Multi-cliente (token por cliente)

Si tenés N clientes y usás Embedded Signup:

| Almacenamiento | Para qué |
|---|---|
| Por cliente: `token`, `waba_id`, `phone_number_ids[]` | Usar la combinación correcta por request |
| Marcar `revoked: true` si el cliente revoca acceso | Para no seguir intentando |

Si un cliente revoca el acceso desde su Portfolio, el token de tu lado deja de funcionar (190.463). No hay recuperación: hay que re-correr el Embedded Signup con ese cliente.

## Buenas prácticas

| Práctica | Razón |
|---|---|
| Un System User por sistema integrador (no compartido) | Revocación granular |
| Tokens en gestor de secretos, no en código | Higiene |
| Loggear `fbtrace_id` en cada error | Útil al reportar a Meta |
| Healthcheck que llama a `me?` cada N minutos | Detectar revocaciones temprano |
| Alerta automática si `is_valid === false` | Reaccionar antes que el cliente reclame |

## Comandos útiles

### Test rápido del token

```bash
curl -X GET \
  "https://graph.facebook.com/v21.0/me?access_token={{TOKEN}}"
```

Si devuelve `{ "name": "...", "id": "..." }` → token básicamente válido.
Si devuelve error → revisar `debug_token`.

### Listar las WABAs accesibles

```bash
curl -X GET \
  "https://graph.facebook.com/v21.0/{{BUSINESS_ID}}/owned_whatsapp_business_accounts" \
  -H "Authorization: Bearer {{ACCESS_TOKEN}}"
```

Si tu WABA no está en la lista, no tenés acceso desde este token.

### Probar permisos de mensajería sin enviar

```bash
curl -X GET \
  "https://graph.facebook.com/v21.0/{{PHONE_NUMBER_ID}}?fields=display_phone_number,verified_name" \
  -H "Authorization: Bearer {{ACCESS_TOKEN}}"
```

Si responde 200 con datos → tenés acceso al número.

### Probar permisos de management sin tocar plantillas reales

```bash
curl -X GET \
  "https://graph.facebook.com/v21.0/{{WABA_ID}}/message_templates?limit=1" \
  -H "Authorization: Bearer {{ACCESS_TOKEN}}"
```

Si responde 200 → tenés `whatsapp_business_management`.

## Referencias

- [Access Tokens · Graph](https://developers.facebook.com/docs/facebook-login/guides/access-tokens) — Verificado 2026-05-20.
- [System Users · Meta](https://developers.facebook.com/docs/marketing-api/system-users) — Verificado 2026-05-20.
- [`01-onboarding-meta/system-users-tokens.md`](../01-onboarding-meta/system-users-tokens.md)
- [`10-troubleshooting/codigos-error.md`](./codigos-error.md)
