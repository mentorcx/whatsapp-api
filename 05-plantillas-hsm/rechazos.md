---
title: Causas frecuentes de rechazo de plantillas
category: plantillas-hsm
tags: [hsm, plantillas, rechazo, troubleshooting]
updated: 2026-05-20
source_official:
  - https://developers.facebook.com/docs/whatsapp/business-management-api/message-templates
related:
  - 05-plantillas-hsm/aprobacion.md
  - 10-troubleshooting/plantillas-pausadas.md
audience: [integrador, marketing]
---

# Causas frecuentes de rechazo de plantillas

> **TL;DR:** El 80% de los rechazos cae en cinco familias: categoría incorrecta, lenguaje promocional ambiguo, URLs sospechosas, variables sin contexto y violaciones de policy (datos sensibles, prohibido). Cada caso tiene un patrón de corrección concreto.

## Contexto

Meta no siempre detalla el motivo exacto del rechazo: a veces solo devuelve un código genérico (`INVALID_FORMAT`, `POLICY_VIOLATION`, `TAG_CONTENT_MISMATCH`). Este documento mapea los códigos y patrones más vistos a causas reales y a la corrección típica.

## Códigos de rechazo

| Código | Significado | Causa habitual |
|---|---|---|
| `INVALID_FORMAT` | Formato inválido | Variables mal numeradas, paréntesis sin pareja, header con tipo equivocado |
| `TAG_CONTENT_MISMATCH` | Categoría no coincide con contenido | Marketing enviado como Utility o viceversa |
| `INVALID_BUTTON_COMBINATION` | Mezcla inválida de botones | Más de un tipo de quick reply + URL combinados mal |
| `INVALID_BUTTON_URL` | URL del botón inválida | Dominio sin verificar, variable mal ubicada, shortener |
| `POLICY_VIOLATION` | Viola Business Policy | Sector restringido, datos sensibles, opt-in dudoso |
| `SCAM` | Sospecha de phishing/scam | Lenguaje urgente + link + pedido de credenciales |
| `ABUSIVE_CONTENT` | Lenguaje ofensivo o agresivo | Texto inapropiado |
| `INCORRECT_CATEGORY` | Categoría incorrecta | Caso muy frecuente, ver tabla de re-categorización |

## Familia 1: categoría incorrecta

**Síntoma:** rechazo con `TAG_CONTENT_MISMATCH` o `INCORRECT_CATEGORY`. A veces Meta re-categoriza sola.

| Plantilla declarada como | Pero realmente es | Por qué Meta lo detecta |
|---|---|---|
| Utility | Marketing | Menciona descuentos, promociones, ofertas |
| Utility | Marketing | Re-engagement ("Hace tiempo que no te vemos") |
| Marketing | Utility | Confirmación de compra con tono amistoso |
| Authentication | Utility | El "código" no es un OTP sino un nro de pedido |
| Utility | Authentication | Envía código de verificación con texto extra promocional |

**Corrección:** declarar la categoría correcta. Si la duda es legítima, mirar el cuadro de [`aprobacion.md`](./aprobacion.md#cómo-elegir-la-categoría-correcta).

## Familia 2: lenguaje promocional encubierto

**Síntoma:** rechazo con `POLICY_VIOLATION` aunque la plantilla parezca inocente.

Frases que disparan re-categorización a Marketing (o rechazo si fue declarada Utility):

- "Aprovechá esta oportunidad"
- "Tenemos novedades para vos"
- "Mirá lo que preparamos"
- "Te invitamos a conocer..."
- "Descuento exclusivo"
- "Por tiempo limitado"
- "Última oportunidad"

**Corrección:** si el contenido es realmente promocional, declararlo como Marketing. Si es transaccional, eliminar el adjetivo o llamada a la acción comercial.

| Antes (rechazado como Utility) | Después (aprobado como Utility) |
|---|---|
| "Hola {{1}}, aprovechá nuestras nuevas funciones. Ingresá acá." | "Hola {{1}}, tu cuenta {{2}} fue actualizada. Detalles: {{URL}}." |
| "Tu pedido llegó, ¡no te pierdas nuestras ofertas!" | "Tu pedido #{{1}} fue entregado." |

## Familia 3: URLs y dominios

**Síntoma:** rechazo con `INVALID_BUTTON_URL` o pausa inmediata post-aprobación.

Causas:

- Shorteners (`bit.ly`, `tinyurl.com`, `t.co`, `cutt.ly`).
- Dominios genéricos no asociados al negocio.
- Dominios con TLDs sospechosos (`.click`, `.xyz`, `.zip`).
- Dominios de tracking de terceros expuestos directamente.
- Variable en posición inválida: `https://{{1}}.midominio.com/...` (variable en subdominio).

**Corrección:**

- Usar dominio propio verificado en Meta Business Manager.
- Variables solo en path o query: `https://midominio.com/order/{{1}}` o `?id={{1}}`.
- Si hace falta tracking, hacerlo redirigiendo desde dominio propio.

## Familia 4: variables sin contexto

**Síntoma:** rechazo con `INVALID_FORMAT` o "el sample no es representativo".

Casos típicos:

| Body | Problema |
|---|---|
| `Hola {{1}}, {{2}}` | Variables sin texto fijo que las contextualice |
| `{{1}}` | El mensaje completo es una variable |
| `Tu código es {{1}} {{2}} {{3}}` | Múltiples variables consecutivas sin separación clara |
| `Hola {{1}}` con sample `xxx` | Sample no realista |

**Corrección:** rodear cada variable de texto fijo que indique qué representa, y cargar samples realistas (nombres, números, fechas creíbles).

| Antes | Después |
|---|---|
| `Hola {{1}}, {{2}}` | `Hola {{1}}, tu turno del {{2}} fue confirmado.` |
| `Tu código: {{1}}` (sample: `xxx`) | `Tu código: {{1}}` (sample: `A4F8K2`) |

## Familia 5: datos sensibles y sectores restringidos

**Síntoma:** rechazo con `POLICY_VIOLATION` o `SCAM`.

Contenidos problemáticos:

- Pedido de contraseñas, PIN o CVV en el body.
- Solicitud de datos de tarjeta o transferencia.
- Mensajes que se hacen pasar por entidad bancaria sin serlo.
- Sectores restringidos: alcohol, tabaco, cannabis, juegos de azar, armas, contenido adulto.
- Productos médicos con claims terapéuticos sin respaldo.
- Préstamos con tasas o letra chica oculta.

**Corrección:** revisar [WhatsApp Business Policy](https://www.whatsapp.com/legal/business-policy) y reformular. En sectores restringidos suele haber lineamientos específicos por región.

## Familia 6: estructura técnica

**Síntoma:** rechazo casi inmediato con `INVALID_FORMAT` o `INVALID_BUTTON_COMBINATION`.

| Problema | Solución |
|---|---|
| Variables no consecutivas (`{{1}}` y `{{3}}` sin `{{2}}`) | Renumerar empezando en 1 y consecutivas |
| Header con variable y body con la misma | Permitido, pero el sample del header debe coincidir |
| Más de 10 botones | Reducir a 10 máximo |
| Mezcla de quick reply con CTA en orden mal definido | Quick replies juntos al final |
| Footer con link o variable | Footer es solo texto fijo y corto |
| Caracteres de control raros (zero-width, RTL marks) | Limpiar el texto |

## Anti-patterns regionales

| Región | Anti-pattern |
|---|---|
| LATAM | Mezclar tuteo y voseo (`tú` y `vos`) en el mismo mensaje |
| Brasil | Mensajes en `pt` cuando el público es `pt_BR` (puede ser flag de spam) |
| España | Lenguaje LATAM marcado para audiencia ES o viceversa |
| Multi-país | Una sola plantilla para varios países sin localización |

No suelen causar rechazo directo pero sí bajan el engagement, que después dispara pausing.

## Cómo apelar un rechazo

1. Editar la plantilla corrigiendo lo que se detectó.
2. Reenviar a aprobación (la edición dispara nueva revisión).
3. Si se cree que el rechazo es injusto: cambiar mínimamente el copy y reenviar para forzar nueva revisión humana.
4. En casos persistentes, abrir ticket a Meta Business Support con `Template Name`, `WABA ID` y captura del rechazo.

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| Plantilla aprobada en sandbox pero rechazada al pasar a número productivo | Diferencias de WABA / categoría detectada distinto en producción | Reenviar en la WABA productiva, no asumir parity |
| Plantilla aprobada pero pausada al primer envío masivo | Tasa de bloqueo alta | Bajar volumen, revisar segmentación y opt-in |
| Misma plantilla, distintos idiomas: uno aprobado, otro rechazado | Texto traducido literal sin localizar | Revisar cada idioma individualmente |

## Referencias

- [Template Guidelines · Meta](https://developers.facebook.com/docs/whatsapp/business-management-api/message-templates/guidelines) — Verificado 2026-05-20.
- [WhatsApp Business Policy](https://www.whatsapp.com/legal/business-policy) — Verificado 2026-05-20.
- [`05-plantillas-hsm/aprobacion.md`](./aprobacion.md)
- [`05-plantillas-hsm/pausing.md`](./pausing.md)
