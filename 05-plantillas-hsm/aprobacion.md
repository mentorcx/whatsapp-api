---
title: Buenas prácticas para aprobación de plantillas
category: plantillas-hsm
tags: [hsm, plantillas, aprobacion, redaccion, meta]
updated: 2026-05-20
source_official:
  - https://developers.facebook.com/docs/whatsapp/business-management-api/message-templates
  - https://www.whatsapp.com/legal/business-policy
related:
  - 05-plantillas-hsm/categorias.md
  - 05-plantillas-hsm/estructura.md
  - 05-plantillas-hsm/rechazos.md
audience: [integrador, marketing, dev]
---

# Buenas prácticas para aprobación de plantillas

> **TL;DR:** Aprobación rápida = categoría correcta + redacción clara + sin promesas vagas + sin URLs sospechosas + variables justificadas. La mayoría de los rechazos vienen por categoría equivocada o lenguaje promocional encubierto como utility.

## Contexto

Meta revisa cada plantilla antes de habilitarla para enviar. La revisión es parte automática, parte humana, y demora entre minutos y varias horas (a veces más de 24h). Una plantilla rechazada se puede editar y reenviar, pero acumular rechazos en una WABA degrada la confianza y enlentece futuras aprobaciones.

Este documento es la checklist que conviene aplicar **antes** de enviar a revisión.

## Checklist pre-envío

| Ítem | Verificación |
|---|---|
| Categoría | ¿Es Marketing, Utility o Authentication? Ver tabla abajo. |
| Idioma | Código correcto (`es_AR`, `es_MX`, `es`, `pt_BR`, etc.). |
| Nombre interno | `snake_case`, descriptivo, sin nombres de marca de terceros. |
| Variables | Cada `{{n}}` tiene un ejemplo concreto y es necesaria. |
| Header | Si es media, declarar el tipo correcto y proveer sample. |
| Footer | Máximo 60 caracteres, sin links. |
| Botones | Máximo 10. URL buttons con dominio propio verificado. |
| Opt-in | El destinatario debe haber dado consentimiento previo. |
| Llamadas a la acción | Claras y consistentes con la categoría. |

## Cómo elegir la categoría correcta

Equivocar la categoría es la causa #1 de rechazo y de re-categorización forzada por Meta.

| Categoría | Cuándo usarla | Ejemplos |
|---|---|---|
| **Authentication** | Códigos OTP, verificación de identidad | "Tu código es {{1}}. No lo compartas." |
| **Utility** | Acciones del usuario, transacciones, actualizaciones de cuenta | Confirmación de compra, estado de envío, recordatorio de turno |
| **Marketing** | Promociones, ofertas, contenido educativo, re-engagement | "Tenemos un 20% off para vos esta semana" |

**Regla práctica:** si la plantilla puede enviarse sin que haya pasado nada del lado del usuario, es Marketing. Si responde a un evento concreto del usuario (compró, agendó, se registró), es Utility. Si entrega un código, es Authentication.

## Redacción que ayuda a aprobar

### Sí

- Lenguaje directo y específico (`"Tu pedido #{{1}} fue despachado"`).
- Variables con propósito claro y muestra concreta al cargar.
- Mencionar el nombre del negocio cuando agregue contexto.
- Saludos cortos: "Hola {{1}}," es suficiente.
- Indicar próximos pasos: "Respondé STOP para no recibir más mensajes".

### No

- Promesas vagas ("ofertas imperdibles", "no te lo podés perder").
- Texto en MAYÚSCULAS sostenidas.
- Más de un emoji por mensaje (mejor cero).
- Repetir signos de exclamación (`!!!`).
- Pedir datos sensibles (CVV, contraseñas) en el cuerpo.
- Variables sin contexto: "Hola {{1}}, {{2}}".
- URLs acortadas tipo `bit.ly` o `tinyurl`.
- Enlazar dominios distintos al del negocio.

## Variables y muestras

Cada variable `{{n}}` requiere cargar un valor de ejemplo al enviar a aprobación. Reglas:

- El ejemplo debe ser **realista**, no `123` ni `xxx`.
- No usar datos personales reales en el ejemplo.
- Si la variable contiene un nombre, usar uno genérico ("Juan", "María").
- Si contiene una fecha, usar formato local consistente (`15/06/2026`).
- Si contiene un código, usar uno del largo real (`A4F8K2`, no `123`).

Variables en URL buttons deben aparecer al final del path o como query param, no en medio del dominio.

## Categorías y reglas específicas

### Authentication

- No incluir llamadas a la acción comerciales.
- No usar variables en el header.
- Texto del botón "Copiar código" debe ser exactamente eso o equivalente literal.
- Footer opcional pero útil: "Este código expira en 10 minutos".

### Utility

- Debe referenciar una acción concreta del usuario.
- Evitar lenguaje promocional ("aprovechá", "no te pierdas").
- Si incluye un link, debe llevar a información directamente relacionada (factura, tracking, ticket).

### Marketing

- Mencionar el opt-out es obligatorio en muchas regiones.
- Limit Ad Tracking: si hay tracking, declararlo o usar dominio propio.
- Evitar superlativos sin sustento ("el mejor precio del país").

## Botones

| Tipo | Reglas |
|---|---|
| **Quick Reply** | Texto corto (≤20 chars), sin emojis, claro y accionable. |
| **URL** | URL pública. Si tiene variable, validar que resuelve siempre. |
| **Phone** | Número en E.164. Idealmente del propio negocio. |
| **Copy code** | Solo en Authentication. |

No mezclar tipos cuando no es necesario: si todos los botones son quick reply, mantenerlo consistente.

## Header

| Tipo | Recomendaciones |
|---|---|
| Text | ≤60 chars. Una variable máximo. |
| Image | JPG/PNG, ratio aproximado 1.91:1, sin texto sobre la imagen si es posible. |
| Video | MP4, primer frame representativo. |
| Document | PDF preferentemente, con nombre descriptivo. |
| Location | Coordenadas válidas. |

Subir media de sample en buena resolución; Meta a veces rechaza por sample borroso aunque el contenido sea correcto.

## Anti-patterns comunes

| Anti-pattern | Por qué falla | Cómo arreglarlo |
|---|---|---|
| Marketing disfrazado de Utility | Meta re-categoriza o rechaza | Honestidad: si es promo, va como Marketing. |
| Plantilla con `{{1}}` para "lo que se ofrezca" | Sample no representativo, uso inconsistente | Una plantilla por caso de uso. |
| Texto idéntico en muchos idiomas con copy/paste | Mismo error replicado | Revisar versión por versión. |
| Footer con link | No permitido | Mover link al body o botón. |
| Variable en medio del dominio de URL button | Rechazo automático | Variable solo en path/query. |

## Después de la aprobación

- Una plantilla aprobada puede **pausarse** por bajo engagement o reportes de spam. Ver [`pausing.md`](./pausing.md).
- Editar el contenido de una plantilla **devuelve** el estado a "pendiente" hasta nueva aprobación.
- Cambiar la categoría desde el panel también dispara nueva revisión.

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| Plantilla rechazada por "categoría incorrecta" | Marketing enviado como Utility | Re-enviar como Marketing. |
| Rechazo sin detalle claro | Política amplia (`policy violation`) | Revisar lenguaje promocional, URLs, opt-in implícito. |
| Aprobada pero pausada al primer envío | Tasa de bloqueos alta o contenido reportado | Bajar volumen, revisar segmentación, considerar opt-in real. |
| Variables no se renderizan | Mismatch entre cantidad declarada y enviada | Verificar `components` del payload de envío. |

## Referencias

- [Message Templates · Meta docs](https://developers.facebook.com/docs/whatsapp/business-management-api/message-templates) — Verificado 2026-05-20.
- [WhatsApp Business Policy](https://www.whatsapp.com/legal/business-policy) — Verificado 2026-05-20.
- [`05-plantillas-hsm/rechazos.md`](./rechazos.md) — catálogo de rechazos frecuentes.
- [`05-plantillas-hsm/categorias.md`](./categorias.md) — pricing por categoría.
