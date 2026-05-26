---
title: 2FA del número y de la cuenta Meta
category: onboarding-meta
tags: [2fa, two-factor, pin, security, recuperacion]
updated: 2026-05-20
source_official:
  - https://developers.facebook.com/docs/whatsapp/cloud-api/reference/registration
related:
  - 01-onboarding-meta/system-users-tokens.md
  - 02-numeros-y-conexion/migrar-numero.md
audience: [integrador, dev, comercial]
---

# 2FA del número y de la cuenta Meta

> **TL;DR:** Hay dos 2FAs distintos: el **PIN del número** en Cloud API (6 dígitos, requerido al `register`, evita robo de la WABA) y el **2FA de la cuenta personal de Meta** (Authenticator/SMS, protege el Portfolio). Confundirlos cuesta tiempo. Guardar ambos en el gestor de secretos del cliente.

## Dos 2FAs distintos

| 2FA | Qué protege | Forma |
|---|---|---|
| **PIN del número (Cloud API)** | El número conectado a la WABA | 6 dígitos numéricos |
| **2FA de la cuenta personal de Meta** | El Portfolio y todos sus activos | Authenticator app / SMS |

Ambos importantes; ambos suelen olvidarse.

## PIN del número en Cloud API

Cada número registrado en Cloud API tiene un **PIN de 6 dígitos** asociado. Sirve para:

| Caso | Detalle |
|---|---|
| `register` del número tras agregarlo a la WABA | Lo definís vos |
| Re-registro tras downtime | Lo necesitás |
| Protección contra robo del número | Si alguien logra acceder a otra parte del setup, sin el PIN no completa |

### Configurar al registrar

```bash
curl -X POST \
  "https://graph.facebook.com/v21.0/{{PHONE_NUMBER_ID}}/register" \
  -H "Authorization: Bearer {{ACCESS_TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "pin": "{{PIN_6_DIGITOS}}"
  }'
```

| Campo | Detalle |
|---|---|
| `pin` | 6 dígitos numéricos. NO usar `000000`, `123456`, fechas obvias. |

### Cambiar el PIN

```bash
curl -X POST \
  "https://graph.facebook.com/v21.0/{{PHONE_NUMBER_ID}}/register" \
  -H "Authorization: Bearer {{ACCESS_TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "pin": "{{NUEVO_PIN}}"
  }'
```

Sobrescribe el anterior.

### Si se pierde el PIN

Hay un período de **espera** para reset (varía, suele ser 7 días). Durante ese tiempo, el número queda bloqueado para re-registro. Por eso: **guardar el PIN en el gestor de secretos**.

## 2FA en WhatsApp (el del usuario, no Cloud API)

Distinto del PIN de Cloud API: cuando un número está en **la app de WhatsApp Business**, el usuario puede tener un PIN de 2FA configurado en la app (Settings → Account → Two-step verification).

| Caso | Detalle |
|---|---|
| Migrar número a Cloud API | Hay que conocer ese PIN o desactivarlo antes |
| Si está activo y no se conoce | La migración falla |

Verificar / desactivar **antes** de migrar. Ver [`02-numeros-y-conexion/migrar-numero.md`](../02-numeros-y-conexion/migrar-numero.md).

## 2FA de la cuenta personal de Meta

El dueño del Portfolio tiene una cuenta personal de Facebook. Esa cuenta **debe tener 2FA activado**.

| Método | Recomendado |
|---|---|
| Authenticator app (Google Authenticator, Authy) | **Sí** |
| SMS | Backup, menos seguro |
| Llave física (YubiKey) | Excelente para cuentas críticas |

Si esa cuenta se compromete, alguien puede acceder al Portfolio entero. El integrador no puede compensar esto: depende del cliente.

## Checklist de seguridad post-onboarding

| Ítem | ✓ |
|---|---|
| 2FA activado en la cuenta personal del dueño del Portfolio | |
| 2FA activado en cualquier cuenta con rol Admin | |
| PIN del número registrado y guardado en gestor de secretos | |
| Tokens (System User) guardados en gestor de secretos | |
| App Secret guardado | |
| Encryption key de n8n guardada aparte | |
| Cliente conoce dónde están los accesos críticos | |

## Quién guarda qué

| Secreto | Quién lo guarda |
|---|---|
| Cuenta personal Meta + 2FA | El cliente (dueño legal) |
| PIN del número | Cliente + integrador (compartido) |
| Tokens System User | Integrador (en su vault) |
| App Secret | Integrador |
| Credenciales n8n / Postgres / Redis | Integrador |

**Importante:** el cliente debe poder recuperar acceso al Portfolio sin el integrador. La cuenta personal y 2FA son del cliente, no del integrador.

## Rotación

| Secreto | Cuándo rotar |
|---|---|
| PIN del número | Si se sospecha que se filtró |
| Token del System User | Cuando alguien con acceso deja el equipo |
| App Secret | Si se filtró |
| 2FA personal | Si se pierde el dispositivo |

Tener un proceso documentado, no improvisar.

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| `register` falla con "PIN incorrect" | PIN olvidado o mal | Esperar período de reset; mejor: guardar bien |
| Migración desde app falla | 2FA de WhatsApp activo en la app | Desactivar o conocer el PIN |
| Cliente pierde acceso al Portfolio | Sin 2FA, alguien cambió contraseña | Recuperar vía Meta (lento) |
| Equipo del integrador rota y nadie tiene los secretos | Sin documentación | Documentar y compartir con cliente |

## Referencias

- [Register Number · Cloud API](https://developers.facebook.com/docs/whatsapp/cloud-api/reference/registration) — Verificado 2026-05-20.
- [WhatsApp Two-step Verification](https://faq.whatsapp.com/1920866721452534/) — Verificado 2026-05-20.
- [`01-onboarding-meta/system-users-tokens.md`](./system-users-tokens.md)
- [`02-numeros-y-conexion/migrar-numero.md`](../02-numeros-y-conexion/migrar-numero.md)
