---
title: Plantillas multi-idioma
category: plantillas-hsm
tags: [hsm, plantillas, idiomas, i18n, multilenguaje]
updated: 2026-05-20
source_official:
  - https://developers.facebook.com/docs/whatsapp/business-management-api/message-templates
related:
  - 05-plantillas-hsm/estructura.md
  - 05-plantillas-hsm/aprobacion.md
audience: [integrador, marketing]
---

# Plantillas multi-idioma

> **TL;DR:** Una plantilla "lógica" multi-idioma son **N plantillas físicas** en Meta, una por idioma. Mismo `name`, distinto `language.code`. Cada una se aprueba por separado. Códigos siguen formato ISO (`es`, `es_AR`, `pt_BR`, `en_US`). Detectar idioma del usuario y elegir la plantilla correspondiente al enviar.

## Modelo de datos

Meta trata cada idioma como una plantilla independiente que comparte `name`:

| `name` | `language.code` | Estado |
|---|---|---|
| `recordatorio_turno` | `es_AR` | APPROVED |
| `recordatorio_turno` | `es_MX` | APPROVED |
| `recordatorio_turno` | `pt_BR` | PENDING |
| `recordatorio_turno` | `en_US` | REJECTED |

Al enviar:

```json
"template": {
  "name": "recordatorio_turno",
  "language": { "code": "es_AR" }
}
```

## Códigos de idioma

| Código | Idioma / región |
|---|---|
| `es` | Español genérico |
| `es_AR` | Español Argentina |
| `es_MX` | Español México |
| `es_ES` | Español España |
| `pt` | Portugués genérico |
| `pt_BR` | Portugués Brasil |
| `pt_PT` | Portugués Portugal |
| `en` | Inglés genérico |
| `en_US` | Inglés EEUU |
| `en_GB` | Inglés UK |
| `fr` | Francés |
| `it` | Italiano |
| `de` | Alemán |

Lista completa en [docs de Meta](https://developers.facebook.com/docs/whatsapp/api/messages/message-templates).

## Localización vs traducción literal

Mal:

> `pt_BR`: "Tu turno é amanhã às 10" — traducción a Google Translate de `es_AR`

Bien:

> `pt_BR`: "Seu agendamento é amanhã às 10h"

Localizar (no solo traducir):

| Aspecto | Detalle |
|---|---|
| Vocabulario regional | `vos` (AR) vs `tú` (MX) |
| Formato de fecha/hora | DD/MM vs MM/DD, AM/PM vs 24h |
| Moneda | Símbolo y separador |
| Tono | Más formal o informal según mercado |
| Saludos | "Hola" vs "Olá" vs "Hi" |

## Detectar idioma del usuario

Estrategias:

| Estrategia | Detalle |
|---|---|
| País del número (E.164 prefix) | `+54` → es_AR, `+55` → pt_BR, etc. Aproximado |
| Idioma del primer mensaje | Detectar con OpenAI o lib (cld3, franc) |
| Preferencia explícita guardada | Campo en el contacto / lead |
| Default por mercado del cliente | Si solo opera en LATAM, default es_LA |

Recomendado: combinar — default por país + override por detección del mensaje + persistir en `metadata.lang` de la conversación.

## Patrón en n8n

```javascript
// Resolver el idioma
const lang = conversation.metadata?.lang
          || langByCountry(phoneNumber)
          || 'es_AR';

// Buscar plantilla aprobada en ese idioma
const tpl = await db.templates.findOne({
  name: 'recordatorio_turno',
  language: lang,
  status: 'APPROVED'
});

if (!tpl) {
  // Fallback a otro idioma
  const fallback = await db.templates.findOne({
    name: 'recordatorio_turno',
    language: 'es_AR',
    status: 'APPROVED'
  });
  // Enviar fallback...
}
```

## Aprobación: una a una

| Realidad | Implicancia |
|---|---|
| Cada idioma se aprueba aparte | Uno puede ser aprobado y otro rechazado |
| Calidad / pausing también es independiente | Tracking por idioma |
| Edición es por idioma | No se propaga |

Si replicás un copy buen aprobado en `es_AR` a `es_MX`, puede rechazarse por particularidades regionales. No asumir paridad.

## Plantillas idénticas: ¿conviene?

Para un mercado que habla múltiples idiomas:

| Caso | Estrategia |
|---|---|
| Cliente solo opera en AR | Solo `es_AR` (o `es`) |
| Cliente opera en LATAM | `es_AR`, `es_MX`, `pt_BR` |
| Cliente global | Crear las top 5-10 según audiencia |

Empezar con los que tienen volumen real. Agregar más solo si justifica.

## Variables consistentes

Mantener la **misma cantidad y orden de variables** entre idiomas. Permite que el código sea simétrico:

```javascript
const params = [name, date, time];
sendTemplate(name, lang, to, params);
```

Si el idioma X tiene una variable más, el código tiene que ramificarse y se vuelve frágil.

## Estrategia de rollout

Para sumar un idioma nuevo:

1. Definir copy localizado (no traducir literal).
2. Crear plantilla con `language: nuevo_codigo`.
3. Enviar a aprobación.
4. **No activar** hasta que esté aprobada.
5. Agregar el routing en el código.
6. Probar con un segmento chico antes de full rollout.

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| Plantilla `es` no encontrada | El cliente WhatsApp espera `es_AR` | Probar variantes regionales |
| Mandás `es_AR` a un usuario MX | Detección mala | Localizar |
| Cantidad de variables distinta entre idiomas | Inconsistencia | Estandarizar |
| Una traducción literal feo aprobada | Sin revisión nativa | Revisar con hablante nativo |
| Confusión al editar uno y olvidar replicar | Trabajo manual | Checklist de propagación |

## Referencias

- [Message Templates · Languages](https://developers.facebook.com/docs/whatsapp/api/messages/message-templates) — Verificado 2026-05-20.
- [`05-plantillas-hsm/estructura.md`](./estructura.md)
- [`05-plantillas-hsm/aprobacion.md`](./aprobacion.md)
