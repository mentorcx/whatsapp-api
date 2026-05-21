---
title: Estructura de una plantilla HSM
category: plantillas-hsm
tags: [hsm, plantillas, estructura, header, body, footer, buttons, variables]
updated: 2026-05-20
source_official:
  - https://developers.facebook.com/docs/whatsapp/business-management-api/message-templates
related:
  - 05-plantillas-hsm/aprobacion.md
  - 05-plantillas-hsm/categorias.md
audience: [integrador, dev, marketing]
---

# Estructura de una plantilla HSM

> **TL;DR:** Una plantilla tiene **cuatro componentes**: `header` (opcional, text/image/video/document/location), `body` (obligatorio, ≤1024 chars con variables), `footer` (opcional, ≤60 chars sin variables ni links), `buttons` (opcional, hasta 10 entre Quick Reply / URL / Phone / Copy code). Las variables van como `{{1}}`, `{{2}}`, ... y necesitan sample al crear.

## Contexto

Conocer la estructura permite diseñar plantillas que pasen aprobación a la primera y se envíen sin errores 132x. Es la referencia técnica que el copywriter y el dev miran al mismo tiempo.

## Estructura general

```
┌─────────────────────────────┐
│  HEADER (opcional)          │  ← text/image/video/document/location
├─────────────────────────────┤
│  BODY (obligatorio)         │  ← texto con variables {{n}}
├─────────────────────────────┤
│  FOOTER (opcional)          │  ← texto corto, sin variables
├─────────────────────────────┤
│  BUTTONS (opcional)         │  ← Quick Reply / URL / Phone / Copy code
└─────────────────────────────┘
```

## Componente Body

| Aspecto | Valor |
|---|---|
| Obligatorio | Sí |
| Longitud | ≤ 1024 caracteres |
| Variables | `{{1}}`, `{{2}}`, ... numeradas y consecutivas |
| Formato | Texto plano (markdown limitado: `*bold*`, `_italic_`, `~strike~`, ``` ` ``` para code) |
| Saltos de línea | Permitidos (`\n`) |
| Emojis | Permitidos, no abusar |

Ejemplo:

```
Hola {{1}}, tu pedido #{{2}} fue despachado.
Llega aproximadamente el *{{3}}*. 
Podés rastrearlo desde el botón de abajo.
```

Variables aquí: `{{1}}` = nombre, `{{2}}` = nro de pedido, `{{3}}` = fecha estimada.

## Componente Header

Opcional. Si está, es de un único tipo.

### Header de texto

```json
{
  "type": "HEADER",
  "format": "TEXT",
  "text": "Confirmación de tu pedido #{{1}}"
}
```

| Restricción | Valor |
|---|---|
| Longitud | ≤ 60 chars |
| Variables | 1 sola, opcional |

### Header media (image / video / document)

```json
{
  "type": "HEADER",
  "format": "IMAGE",
  "example": {
    "header_handle": ["{{HANDLE_DE_LA_MUESTRA_SUBIDA}}"]
  }
}
```

Para crearlo:

1. Subir media de muestra a Meta usando el endpoint de Resumable Upload o `/media`.
2. Obtener `header_handle`.
3. Incluirlo en `example.header_handle`.

| Formato | Detalles |
|---|---|
| IMAGE | JPG/PNG |
| VIDEO | MP4 |
| DOCUMENT | PDF |
| LOCATION | Coordenadas fijas (no variable en sample) |

Al enviar la plantilla, se reemplaza el media de muestra por el real.

## Componente Footer

| Aspecto | Valor |
|---|---|
| Obligatorio | No |
| Longitud | ≤ 60 chars |
| Variables | **No** permitidas |
| Links | **No** permitidos |
| Uso típico | Disclaimer, "No respondas este mensaje", marca |

Ejemplo:

```
"Atención: 9 a 18 hs · Lunes a Viernes"
```

## Componente Buttons

Hasta **10 botones**, distribuidos en estas categorías:

### Quick Reply

```json
{
  "type": "QUICK_REPLY",
  "text": "Confirmar"
}
```

| Restricción | Valor |
|---|---|
| Longitud `text` | ≤ 25 chars |
| Cantidad | Hasta 10 (compartidos con otros tipos) |

Cuando el usuario lo toca, llega webhook con `interactive.button_reply.id` (Meta usa el `text` como `id` salvo que especifiques).

### URL (estático o con variable)

```json
{
  "type": "URL",
  "text": "Ver mi pedido",
  "url": "https://midominio.com/orders/{{1}}",
  "example": ["https://midominio.com/orders/12345"]
}
```

| Restricción | Valor |
|---|---|
| `text` | ≤ 25 chars |
| `url` | HTTPS, dominio propio recomendado |
| Variable | 1 max, **al final** del path o como query |
| Sample | Obligatorio en `example` |

### Phone

```json
{
  "type": "PHONE_NUMBER",
  "text": "Llamar",
  "phone_number": "+5491100000000"
}
```

| Restricción | Valor |
|---|---|
| `text` | ≤ 25 chars |
| `phone_number` | E.164 con `+` |
| Variable | No |

### Copy code (solo Authentication)

```json
{
  "type": "OTP",
  "otp_type": "COPY_CODE"
}
```

Genera automáticamente el botón "Copiar código" que copia el código del body al portapapeles del usuario.

## Variables: cómo declararlas

Al crear la plantilla:

```json
{
  "name": "recordatorio_turno",
  "language": "es_AR",
  "category": "UTILITY",
  "components": [
    {
      "type": "BODY",
      "text": "Hola {{1}}, te recordamos tu turno del {{2}} a las {{3}}.",
      "example": {
        "body_text": [["María", "15 de junio", "10:00 AM"]]
      }
    }
  ]
}
```

| Detalle | |
|---|---|
| `example.body_text` | Array de arrays; cada array interno es un set de sample valores |
| Mínimo 1 sample | Realista, no `xxx` ni `123` |
| Múltiples samples | Permitido, ayuda a Meta a validar |

## Cómo enviar una plantilla

```bash
curl -X POST \
  "https://graph.facebook.com/v21.0/{{PHONE_NUMBER_ID}}/messages" \
  -H "Authorization: Bearer {{ACCESS_TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "to": "{{DESTINATION_E164}}",
    "type": "template",
    "template": {
      "name": "recordatorio_turno",
      "language": { "code": "es_AR" },
      "components": [
        {
          "type": "body",
          "parameters": [
            { "type": "text", "text": "María" },
            { "type": "text", "text": "15 de junio" },
            { "type": "text", "text": "10:00 AM" }
          ]
        }
      ]
    }
  }'
```

Si la plantilla tiene header media:

```json
"components": [
  {
    "type": "header",
    "parameters": [
      { "type": "image", "image": { "link": "https://midominio.com/factura.jpg" } }
    ]
  },
  {
    "type": "body",
    "parameters": [ ... ]
  }
]
```

Si la plantilla tiene URL button con variable:

```json
"components": [
  {
    "type": "body",
    "parameters": [ ... ]
  },
  {
    "type": "button",
    "sub_type": "url",
    "index": 0,
    "parameters": [
      { "type": "text", "text": "12345" }
    ]
  }
]
```

## Crear plantilla vía API

```bash
curl -X POST \
  "https://graph.facebook.com/v21.0/{{WABA_ID}}/message_templates" \
  -H "Authorization: Bearer {{ACCESS_TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "recordatorio_turno",
    "language": "es_AR",
    "category": "UTILITY",
    "components": [
      {
        "type": "BODY",
        "text": "Hola {{1}}, te recordamos tu turno del {{2}}.",
        "example": {
          "body_text": [["María", "15 de junio 10:00"]]
        }
      },
      {
        "type": "FOOTER",
        "text": "Para reprogramar respondé este mensaje"
      },
      {
        "type": "BUTTONS",
        "buttons": [
          { "type": "QUICK_REPLY", "text": "Confirmar" },
          { "type": "QUICK_REPLY", "text": "Reprogramar" }
        ]
      }
    ]
  }'
```

Respuesta:

```json
{
  "id": "...",
  "status": "PENDING",
  "category": "UTILITY"
}
```

## Patrones de plantillas comunes

### Recordatorio transaccional

| Componente | Contenido |
|---|---|
| Body | Hola {{1}}, te recordamos {{2}}. |
| Footer | Para más info, respondé este mensaje. |
| Buttons | `Confirmar` / `Reprogramar` |

### Confirmación de pago

| Componente | Contenido |
|---|---|
| Header | IMAGE (logo o ícono) |
| Body | Confirmamos tu pago de {{1}} por la orden #{{2}}. |
| Buttons | URL `Ver factura` con `{{1}}` = id de la factura |

### OTP

| Componente | Contenido |
|---|---|
| Body | Tu código es {{1}}. No lo compartas. |
| Footer | El código expira en 10 minutos. |
| Buttons | OTP COPY_CODE |

### Re-engagement

| Componente | Contenido |
|---|---|
| Header | TEXT: "Te extrañamos" |
| Body | Hola {{1}}, hace tiempo que no nos vemos. ¿Querés conocer nuestras novedades? |
| Buttons | QUICK_REPLY `Sí, contame` / QUICK_REPLY `Ahora no` |
| Categoría | Marketing |

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| Variables enviadas ≠ declaradas | `132000` | Contar `{{n}}` y matchear cantidad |
| Footer con link | Rechazo automático | Mover link al body o botón |
| URL button con variable en subdominio | Rechazo automático | Variable solo en path/query |
| Body excede 1024 chars al hidratarse | `132005` | Acortar texto fijo o variables |
| Botones con texto > 25 chars | Error al crear | Acortar |
| Sample faltante | Error al crear | Agregar `example.body_text` |

## Referencias

- [Message Templates · Reference](https://developers.facebook.com/docs/whatsapp/business-management-api/message-templates) — Verificado 2026-05-20.
- [Send Templates · Cloud API](https://developers.facebook.com/docs/whatsapp/cloud-api/guides/send-message-templates) — Verificado 2026-05-20.
- [`05-plantillas-hsm/aprobacion.md`](./aprobacion.md)
- [`05-plantillas-hsm/categorias.md`](./categorias.md)
