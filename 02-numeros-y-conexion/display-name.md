---
title: Display name del número
category: numeros-y-conexion
tags: [display-name, branding, aprobacion, name-policy]
updated: 2026-05-20
source_official:
  - https://www.facebook.com/business/help/757569725593362
  - https://developers.facebook.com/docs/whatsapp/cloud-api/get-started/add-a-phone-number
related:
  - 02-numeros-y-conexion/requisitos-numero.md
  - 01-onboarding-meta/verificacion-comercial.md
audience: [integrador, marketing, comercial]
---

# Display name del número

> **TL;DR:** El display name es el **nombre comercial** que aparece junto al número antes de que el usuario lo agende (`Tienda Acme` en vez de `+54 9 11 ...`). Lo aprueba Meta según una **Name Policy** estricta: debe coincidir con el nombre comercial reconocible, sin claims, sin genéricos, sin descripciones. Aprobación: 1-3 días típico, rechazos frecuentes por nombres demasiado genéricos.

## Contexto

El display name es marketing puro. Una buena aprobación da un primer contacto profesional. Una rechazada deja el número mostrando solo dígitos y obliga al usuario a confiar a ciegas.

## Cómo se ve

Sin display name aprobado:

```
+54 9 11 1234 5678
```

Con display name aprobado (`Tienda Acme`):

```
Tienda Acme
+54 9 11 1234 5678
```

Aparece en el chat list, en el header del chat y en notificaciones.

## Reglas (Display Name Policy)

| Regla | Detalle |
|---|---|
| Reconocible | Debe ser el nombre comercial usado en el sitio web, redes, packaging |
| No genérico | "Atención al cliente", "Ventas", "Soporte" → rechazo casi seguro |
| No claims | "El mejor X", "100% efectivo" → rechazo |
| No descripciones | "Marca Acme que vende calzado" → rechazo |
| No URLs / emails / teléfonos | "Acme.com" → rechazo |
| No referencias a Meta / WhatsApp | "WhatsApp Acme" → rechazo |
| Coincidencia con sitio web | El sitio debe mencionar la marca |
| Capitalización razonable | "ACME" en mayúsculas sostenidas puede pedir revisión |
| Caracteres permitidos | Letras, números, espacios, signos limitados |
| Longitud | Suele 3-25 chars; verificar al cargarlo |
| Multi-idioma | El nombre puede ir en el idioma de la marca |

## Ejemplos

| Propuesto | Probabilidad | Por qué |
|---|---|---|
| `Tienda Acme` | Alta | Nombre de marca claro |
| `Acme Calzado` | Alta | Marca + categoría amplia, sin claim |
| `Atención al Cliente` | Muy baja | Genérico |
| `Acme - El mejor calzado` | Baja | Claim |
| `Acme.com` | Muy baja | Es URL |
| `Soporte Acme` | Media | Combinación genérico + marca, suele aprobarse |
| `WhatsApp Acme` | Cero | Referencia a Meta |
| `+54 Acme` | Baja | Mezcla número con marca |
| `Clínica San Martín` | Alta | Nombre comercial real |
| `Banco XYZ Soporte` | Media-alta | Marca conocida + función |

## Cómo proponerlo

Dos momentos:

### Al crear el número

Durante el alta del Phone Number en la WABA, se pide display name. Se envía a revisión automáticamente.

### Editarlo después

WABA → Phone numbers → seleccionar número → **Edit name**.

Cada edición dispara nueva revisión. **Mientras está en revisión**, el nombre anterior sigue activo (si había uno aprobado).

## Estados

| Estado | Significado |
|---|---|
| `NONE` | Sin nombre asignado (solo número) |
| `PENDING_REVIEW` | En revisión |
| `APPROVED` | Aprobado y visible |
| `DECLINED` | Rechazado, hay que proponer otro |
| `EXPIRED` | Caducado por inactividad o cambio (raro) |

Vía API:

```bash
curl -X GET \
  "https://graph.facebook.com/v21.0/{{PHONE_NUMBER_ID}}?fields=verified_name,display_phone_number,name_status" \
  -H "Authorization: Bearer {{ACCESS_TOKEN}}"
```

## Tiempos de aprobación

| Caso | Tiempo típico |
|---|---|
| Nombre claro, sitio web coherente, marca reconocible | 1-2 días |
| Nombre nuevo, sitio web modesto | 2-5 días |
| Cambios sucesivos | Más lento; Meta endurece criterio |
| Apelación tras rechazo | Variable |

## Cómo aumentar probabilidad de aprobación

| Acción | Por qué ayuda |
|---|---|
| Sitio web con el nombre propuesto visible (header, footer, About) | Demuestra que es la marca real |
| Página de Facebook con el mismo nombre | Coincidencia cross-platform |
| Email corporativo del dominio de la marca | Coherencia |
| Razón social cercana al nombre comercial | Coincidencia documental |
| Esperar a tener verificación comercial aprobada | Mejora el case |

## Verificación comercial y display name

| Estado verificación | ¿Puedo tener display name? |
|---|---|
| Sin verificar | Sí, pero con menos margen y nombres custom limitados |
| Verificado | Más posibilidades, nombres custom permitidos, opciones avanzadas |

Por eso conviene en el siguiente orden:

1. Iniciar verificación comercial.
2. Mientras espera, agregar número con un display name razonable.
3. Una vez verificado, ajustar si hace falta.

## Causas comunes de rechazo

| Razón | Solución |
|---|---|
| Nombre genérico | Reemplazar por la marca |
| Nombre no aparece en el sitio web | Actualizar el sitio antes de reenviar |
| Mezcla con teléfono / URL / email | Quitar |
| Mayúsculas sostenidas | Capitalización normal |
| Marca registrada que no se demuestra que pertenece | Aportar evidencia (sitio, marca registrada, documentos) |
| Idioma raro o mezcla | Mantener un idioma |

## Cambiar display name de un número en producción

Riesgos:

- Durante la nueva revisión, el anterior puede quedar **temporalmente reemplazado** según la región y la política vigente.
- Si el nuevo se rechaza varias veces, queda registro y se vuelve más estricto.

Recomendación: hacerlo **fuera de horario pico** y avisar al equipo. Y solo cambiarlo si hay razón clara (cambio de marca, error de tipeo).

## Verificación de la marca (badge verde) ≠ display name

| Concepto | Qué es |
|---|---|
| Display name | Texto que aparece como nombre |
| Verificación de cuenta oficial (badge verde) | Tilde verde al lado del nombre |

El badge verde es **otro proceso, mucho más restrictivo**, reservado a marcas conocidas con volumen. Casi ningún cliente común califica. No confundir.

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| Nombre rechazado sin razón clara | Categoría amplia (`policy violation`) | Probar variante con marca primero, función después |
| Aprobado pero no aparece al usuario | Cache del cliente WhatsApp | Esperar 24h |
| Nombre aparece para algunos usuarios y no otros | Distribución gradual | Normal, esperar |
| Cambié el nombre y se rechazó; el viejo no vuelve | Política de "última aprobada" | Reenviar el viejo o uno nuevo |
| Display name no aparece en CTWA ads | Las ads pueden mostrar el nombre de la Page, no el del número | Verificar config de la campaña |

## Anti-patterns

- Cambiar display name 3 veces en una semana "probando".
- Proponer el nombre del producto en lugar del de la empresa.
- Usar emojis (raramente aprobados y rompen branding).
- Proponer nombre en un idioma distinto al de la audiencia.

## Referencias

- [WhatsApp Display Name Guidelines](https://www.facebook.com/business/help/757569725593362) — Verificado 2026-05-20.
- [Add a phone number](https://developers.facebook.com/docs/whatsapp/cloud-api/get-started/add-a-phone-number) — Verificado 2026-05-20.
- [`02-numeros-y-conexion/requisitos-numero.md`](./requisitos-numero.md)
- [`01-onboarding-meta/verificacion-comercial.md`](../01-onboarding-meta/verificacion-comercial.md)
