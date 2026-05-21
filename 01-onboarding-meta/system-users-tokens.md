---
title: System Users y tokens
category: onboarding-meta
tags: [system-user, token, oauth, permisos, scopes]
updated: 2026-05-20
source_official:
  - https://developers.facebook.com/docs/marketing-api/system-users
  - https://developers.facebook.com/docs/whatsapp/cloud-api/get-started
related:
  - 01-onboarding-meta/business-portfolio.md
  - 10-troubleshooting/tokens.md
audience: [integrador, dev]
---

# System Users y tokens

> **TL;DR:** Para integraciones server-to-server **siempre** usar tokens de **System Users**, no de cuentas personales. Los user tokens duran ~60 minutos, los system user tokens duran **muchos meses o no expiran** (depende del tipo). Scopes mínimos: `whatsapp_business_messaging`, `whatsapp_business_management`, `business_management`.

## Contexto

Confundir tipos de token es la causa #1 de errores `190` (`Session has expired`) en producción. Este documento aclara qué token usar, cómo generarlo y cómo manejarlo.

## Tipos de token

| Tipo | Quién lo emite | Duración | Cuándo usar |
|---|---|---|---|
| User Access Token (corto) | Login del usuario | ~1-2 horas | Pruebas iniciales en Graph Explorer |
| User Access Token (largo) | Exchange del corto | 60 días | Pruebas extendidas (no producción) |
| **System User Token** | System User del Portfolio | Larga / sin expiración | **Producción server-to-server** |
| App Access Token | App + secret | No expira | Operaciones a nivel app, no de usuario |
| Page Access Token | Página + user token | Varía | No aplica a Cloud API de WA |

## System User: cómo se crea

Prerequisito: el Business Portfolio del cliente existe. Ver [`business-portfolio.md`](./business-portfolio.md).

### 1. Crear el System User

Business Manager → **Configuración** → **Usuarios del sistema** → **Agregar**.

| Campo | Detalle |
|---|---|
| Nombre | Descriptivo: `integracion-prod`, `n8n-worker`, etc. |
| Rol | `Empleado` (default) o `Administrador` (solo si hace falta) |

Crear un System User **por sistema integrador**, no compartir el mismo entre 5 servicios distintos. Facilita revocar permisos puntuales.

### 2. Asignar activos al System User

Sin asignar activos, el System User no puede hacer nada. Hay que darle acceso explícito a:

| Activo | Permiso típico |
|---|---|
| App de Meta | `Develop` o `Manage` |
| WABA | `Manage` (o `Send messages` si solo va a enviar) |
| Catálogo de Commerce (opcional) | `Manage` |
| Páginas (opcional) | Según necesidad |

En el panel del System User → **Asignar activos** → seleccionar el activo → permisos.

### 3. Generar el token

System User → **Generar nuevo token** → seleccionar **App** → elegir **scopes**:

| Scope | Para qué |
|---|---|
| `whatsapp_business_messaging` | Enviar y recibir mensajes |
| `whatsapp_business_management` | Crear/editar plantillas, manejar números, webhooks |
| `business_management` | Operar sobre el Business Portfolio (necesario para algunos endpoints) |
| `catalog_management` | Si vas a usar catálogo de Commerce |

**Importante:** Meta muestra el token **una sola vez**. Copiarlo y guardarlo de inmediato. Si se pierde, hay que regenerar.

### 4. Duración del token

| Opción al generar | Duración |
|---|---|
| **Never expires** | Sin expiración (recomendado prod) |
| **60 days** | 60 días desde la generación |

Para producción siempre `Never expires`. Para experimentación se puede usar 60 días.

## Cómo se ve un token

Tokens largos opacos tipo:

```
EAAGm0PX4ZCpsBAOZA... (~200 chars)
```

No empieza con un prefijo identificable: no se puede distinguir a simple vista un user token de un system user token. Hay que verificar:

```bash
curl -X GET \
  "https://graph.facebook.com/v21.0/debug_token?input_token={{TOKEN}}&access_token={{TOKEN}}"
```

Respuesta incluye `type` (`USER`, `SYSTEM_USER`, `PAGE`), `expires_at`, `scopes`.

## Manejo seguro

### Almacenamiento

| Lugar | Apto |
|---|---|
| Variable de entorno en Railway / Render / Fly | Sí |
| GitHub Actions Secrets | Sí |
| Vault (HashiCorp, AWS Secrets Manager) | Sí, ideal |
| `.env` versionado en Git | **No** |
| Credentials de n8n | Sí (encriptado con `N8N_ENCRYPTION_KEY`) |
| Hardcoded en código | **No** |

### Rotación

| Caso | Acción |
|---|---|
| Token comprometido | Regenerar de inmediato, revocar el viejo |
| Empleado del integrador se va | Revisar System Users que tenía y revocar si aplica |
| Cliente termina relación con el integrador | Revocar el System User completo desde el Portfolio del cliente |
| Rotación rutinaria | Opcional con `Never expires`, recomendado si la política interna lo exige |

### Revocar

Business Manager → System User → **Tokens** → **Revocar**.

Esto invalida el token inmediatamente. Cualquier servicio que lo use empieza a fallar con `190`.

## Scopes por uso típico

| Caso de uso | Scopes mínimos |
|---|---|
| Solo enviar mensajes | `whatsapp_business_messaging` |
| Enviar + recibir webhooks | `whatsapp_business_messaging` + suscripción a webhooks vía App |
| Enviar + manejar plantillas | + `whatsapp_business_management` |
| Operación administrativa completa | + `business_management` |
| Embedded Signup para terceros | + flujo OAuth con scopes adicionales |

Cuando hay duda, dar los 3 estándar: `whatsapp_business_messaging`, `whatsapp_business_management`, `business_management`.

## Verificar un token

```bash
curl -X GET \
  "https://graph.facebook.com/v21.0/me?access_token={{TOKEN}}"
```

Respuesta esperable para un System User token:

```json
{
  "name": "integracion-prod",
  "id": "{{SYSTEM_USER_ID}}"
}
```

Si tira error `190.460` → token expirado o revocado.
Si tira error `200` → token válido pero sin scopes para `me`. Probar otro endpoint conocido.

## Embedded Signup y OAuth (caso multi-cliente)

Si vas a onboardear **decenas de clientes** que no tienen Portfolio armado, usar Embedded Signup en lugar de crear System User a mano por cliente.

El flujo:

1. El cliente cliquea un botón en tu app.
2. Se abre el modal de Meta dentro de tu UI.
3. El cliente autoriza tu App como Tech Provider sobre su Portfolio/WABA.
4. Recibís un código que intercambiás por un token con el endpoint de OAuth.
5. Almacenás el token por cliente.

Detalle en [`embedded-signup.md`](./embedded-signup.md).

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| `190.460` después de unos días | User token largo, no System User | Regenerar como System User token |
| `200` con todos los scopes | Token válido pero el System User no tiene activos asignados | Asignar la WABA al System User |
| Funciona local con tu token, falla prod | Token de tu cuenta personal en lugar del System User | Reemplazar por System User token |
| No puedo crear plantillas | Falta `whatsapp_business_management` | Regenerar con scope correcto |
| Webhook no llega tras cambiar token | Suscripción de la WABA se mantuvo, pero validación de firma usa el secret de la App, no el token | Verificar `App Secret` (es otra cosa) |
| Cliente quiere "su propio token" | Cliente puede crear su System User desde su Portfolio | Documentar el proceso o asistir |

## Anti-patterns

- Compartir el mismo token entre `dev` y `prod`.
- Generar token como `Empleado` y luego escalar a `Admin` por flojera: regenerarlo bien desde el inicio.
- Versionar tokens en `.env.example`.
- No revocar tokens cuando alguien deja el equipo.
- Usar user tokens en producción "porque andan por ahora".

## Referencias

- [System Users · Meta](https://developers.facebook.com/docs/marketing-api/system-users) — Verificado 2026-05-20.
- [Get Started · Cloud API](https://developers.facebook.com/docs/whatsapp/cloud-api/get-started) — Verificado 2026-05-20.
- [Access Tokens · Graph API](https://developers.facebook.com/docs/facebook-login/guides/access-tokens) — Verificado 2026-05-20.
- [`10-troubleshooting/tokens.md`](../10-troubleshooting/tokens.md)
- [`01-onboarding-meta/business-portfolio.md`](./business-portfolio.md)
