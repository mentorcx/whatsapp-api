---
title: Verificación comercial del Business Portfolio
category: onboarding-meta
tags: [verificacion, business-portfolio, documentos, meta]
updated: 2026-05-20
source_official:
  - https://www.facebook.com/business/help/2058515294227817
  - https://business.facebook.com/security
related:
  - 01-onboarding-meta/business-portfolio.md
  - 02-numeros-y-conexion/display-name.md
audience: [integrador, dev, comercial]
---

# Verificación comercial del Business Portfolio

> **TL;DR:** Meta verifica que la empresa **existe y es lo que dice ser** validando documentación legal + presencia online + comprobante de domicilio. Demora entre 1 día y 2 semanas. Es prerequisito para varias features (multi-número productivo, display name personalizado, tier alto). Errores frecuentes: nombre legal que no coincide, sitio web sin info corporativa, número de teléfono no contestable.

## Contexto

Sin verificación, el cliente puede operar igual (1-2 números, tier inicial, etc.) pero queda limitado. Casi cualquier proyecto serio requiere verificación tarde o temprano.

## Qué desbloquea

| Feature | ¿Necesita verificación? |
|---|---|
| Crear WABA | No |
| Conectar 1 número de prueba | No |
| Enviar mensajes free-form | No |
| Crear plantillas básicas | No |
| Tier > 250 | Sí (idealmente) |
| Múltiples números en producción | Sí |
| Display name custom (`MiNegocio` en vez de número) | Sí |
| Embedded Signup para terceros | Sí |
| Algunas features de Ads (CTWA premium) | Sí |

## Cuándo iniciar

Recomendado: **antes** de poner el bot en producción real. La verificación lleva tiempo y bloquea features clave.

Mal momento: en medio de una campaña que necesita tier alto urgente.

## Requisitos

### Identidad de la empresa

| Documento | Detalle |
|---|---|
| Razón social legal | Tal cual figura en la documentación oficial |
| Tipo de organización | SA, SRL, monotributo, ONG, etc. (varía por país) |
| ID fiscal | CUIT (AR), RFC (MX), CNPJ (BR), NIF (ES), etc. |
| Dirección | Coincidente con documentación |

### Documentos a presentar

Meta acepta una combinación de:

| Categoría | Ejemplos |
|---|---|
| Documento gubernamental | Constitución, alta de impuestos, certificado de inscripción |
| Comprobante de domicilio | Factura de servicios reciente (luz, agua, gas, internet) a nombre de la empresa |
| Documento bancario | Resumen o certificado bancario reciente |
| Documento de utility | Recibo a nombre de la empresa |

Reglas:

- Documentos en idioma local aceptados (no exige traducción).
- PDF nítido, no fotos torcidas.
- Fecha de emisión < 90 días para comprobantes de domicilio.
- Nombre legal **exacto** en todos los documentos.

### Presencia online

Tan o más importante que los documentos:

| Activo | Qué chequea Meta |
|---|---|
| Sitio web | Dominio propio, no template genérico, con info corporativa (Quiénes somos, contacto, RUT/CUIT) |
| Email | Dominio propio (`info@empresa.com`, no `gmail.com`) |
| Teléfono | Real, atendible. Meta llama a veces. |
| Pagina de Facebook | Recomendada, vinculada al Portfolio |

## Pasos para iniciar

1. **Business Portfolio → Security Center** → sección **Verificación del negocio**.
2. Cargar info de negocio (legal name, address, web, phone).
3. Seleccionar país y subir documentos.
4. Indicar método de contacto preferido (email o teléfono).
5. Submit.
6. Esperar revisión.

## Tiempos típicos

| Etapa | Tiempo |
|---|---|
| Revisión inicial | 24-48 horas |
| Pedido de info adicional | +1 a 5 días por cada ronda |
| Aprobación final | 3-10 días total típico |
| Casos complejos (rechazos previos, info ambigua) | 2-4 semanas |

## Estados durante el proceso

| Estado | Significado |
|---|---|
| `Not started` | No se envió todavía |
| `Pending submission` | Faltan documentos |
| `In review` | Meta revisando |
| `Pending verification` | Meta pidió info adicional |
| `Approved` | Verificación completada |
| `Rejected` | Rechazada, puede reapelarse |

## Causas frecuentes de rechazo

| Causa | Cómo prevenirla |
|---|---|
| Nombre legal en documentos ≠ nombre en Portfolio | Verificar que coincida **exacto**, hasta espacios y mayúsculas |
| Sitio web sin info de la empresa (landing genérica) | Agregar "Quiénes somos", contacto, política de privacidad |
| Dominio del email no coincide con el del sitio web | Usar email corporativo del mismo dominio |
| Teléfono no atendido cuando Meta llama | Pre-avisar al cliente para que esté disponible |
| Comprobante de domicilio vencido | Subir uno de < 90 días |
| Documento de otra empresa relacionada | Usar documento de la entidad exacta |
| País del Portfolio ≠ país de los documentos | Cambiar país antes de submitear (algunos países no se pueden cambiar después) |
| Foto del documento en lugar de PDF escaneado | Escanear con app tipo CamScanner / Adobe Scan |

## Después de la aprobación

- Las features bloqueadas se habilitan en horas.
- Se puede solicitar display name personalizado.
- El badge "Verificado" no aparece automáticamente; ese es otro proceso (verificación de marca/cuenta), no requerido para operar.

## Verificación vs Cuenta Oficial (badge verde)

Confundir esto cuesta sesiones de soporte. Son **dos cosas distintas**:

| Concepto | Qué es | Cómo se consigue |
|---|---|---|
| Verificación del Business Portfolio | Confirma que la empresa existe legalmente | Documentos legales |
| Cuenta Oficial (Official Business Account) | Tilde verde sobre el nombre en chats | Solicitud aparte, criterios estrictos (marca conocida, volumen, etc.) |

Casi todos los clientes necesitan la verificación del Portfolio; muy pocos califican para el badge verde.

## Verificación facial (en algunos casos)

Meta a veces pide **video selfie** del director / dueño con su DNI. Si pasa:

- Usar buen lighting.
- Coincidir nombre con el del documento.
- Cara visible, no tapada.

Si la persona ya no está en la empresa, abrir caso por soporte explicando el cambio de directorio.

## Checklist pre-envío

| Ítem | ✓ |
|---|---|
| Razón social exacta en todos los documentos | |
| Sitio web propio con info corporativa visible | |
| Email de dominio propio | |
| Teléfono atendible en horario comercial | |
| Comprobante de domicilio < 90 días | |
| Documentos en PDF nítido (no fotos) | |
| País del Portfolio coincide con país de los documentos | |
| Cliente avisado para responder llamadas de Meta | |

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| Verificación trabada en `Pending verification` por semanas | Sin responder al pedido de info | Revisar bandeja de notificaciones del Portfolio |
| Rechazo sin explicación clara | Razón genérica | Apelar agregando 1-2 documentos extras |
| El sitio web "no califica" | Landing demasiado simple | Agregar página About / contacto / privacy |
| Meta no responde después de 2 semanas | Caso atorado | Abrir ticket en Direct Support si tenés acceso, o esperar |
| Aprobada pero features siguen bloqueadas | Cache del Portfolio | Esperar 24h, refrescar |

## Referencias

- [Business Verification · Meta](https://www.facebook.com/business/help/2058515294227817) — Verificado 2026-05-20.
- [Security Center](https://business.facebook.com/security) — Verificado 2026-05-20.
- [`01-onboarding-meta/business-portfolio.md`](./business-portfolio.md)
- [`02-numeros-y-conexion/display-name.md`](../02-numeros-y-conexion/display-name.md)
