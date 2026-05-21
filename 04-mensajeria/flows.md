---
title: WhatsApp Flows
category: mensajeria
tags: [flows, forms, formularios, interactive, screens]
updated: 2026-05-20
source_official:
  - https://developers.facebook.com/docs/whatsapp/flows
related:
  - 04-mensajeria/interactivos.md
  - 04-mensajeria/tipos-de-mensaje.md
audience: [integrador, dev]
---

# WhatsApp Flows

> **TL;DR:** Flows son **formularios in-chat** con múltiples pantallas (selects, inputs, datepickers, etc.). Se diseñan en JSON con un schema declarativo, se envían como mensaje interactivo, y devuelven los datos al servidor del integrador al completarse. Útil para reservas, calificación de leads complejos, formularios largos, encuestas. Reemplaza el "ping-pong de preguntas" del bot.

## Contexto

Cuando una conversación necesita recolectar 5+ datos estructurados (alta de cliente, reserva, formulario médico), hacerlo turno a turno cansa al usuario. Los Flows muestran un formulario nativo dentro de WhatsApp, llenarlo de una y devolver todos los datos al final.

## Cuándo usar Flows vs interactivos vs texto

| Caso | Mejor |
|---|---|
| 1 sí/no | Reply button |
| 2-3 opciones rápidas | Reply buttons |
| 4-10 opciones | List |
| 5+ campos estructurados (datos personales, preferencias) | **Flow** |
| Datos sensibles que requieren confirmación | **Flow** |
| Reserva con fecha + hora + servicio + cantidad | **Flow** |
| Calificación de lead con múltiples preguntas | **Flow** |
| Conversación abierta | Texto libre + LLM |

## Anatomía de un Flow

| Componente | Detalle |
|---|---|
| **Flow JSON** | Schema declarativo de las pantallas y campos |
| **Screens** | Páginas del formulario; se navegan secuencialmente o por condicional |
| **Components** | Inputs por pantalla: TextInput, Dropdown, DatePicker, CheckboxGroup, etc. |
| **Data exchange** | Endpoint del integrador al que Flow llama para validar / cargar datos dinámicos |
| **Footer button** | Avanzar a la siguiente pantalla o submitear |
| **Completion** | Cierre del Flow, dispara webhook con los datos |

## Tipos de Flow

| Tipo | Cuándo |
|---|---|
| **Sin endpoint** | Pantallas estáticas, todos los datos definidos en el JSON. Datos finales llegan en el `flow_response_message` |
| **Con endpoint** | Validación dinámica, opciones cargadas en runtime. Llamadas firmadas con clave pública del integrador |

Sin endpoint es lo más simple para empezar.

## Cómo crear un Flow

1. Diseñar el JSON del Flow.
2. Crear el Flow vía API o desde el panel:
   ```bash
   curl -X POST \
     "https://graph.facebook.com/v21.0/{{WABA_ID}}/flows" \
     -H "Authorization: Bearer {{ACCESS_TOKEN}}" \
     -H "Content-Type: application/json" \
     -d '{
       "name": "reserva_turno",
       "categories": ["APPOINTMENT_BOOKING"]
     }'
   ```
3. Subir el JSON del Flow:
   ```bash
   curl -X POST \
     "https://graph.facebook.com/v21.0/{{FLOW_ID}}/assets" \
     -H "Authorization: Bearer {{ACCESS_TOKEN}}" \
     -F "file=@flow.json" \
     -F "name=flow.json" \
     -F "asset_type=FLOW_JSON"
   ```
4. Publicar:
   ```bash
   curl -X POST \
     "https://graph.facebook.com/v21.0/{{FLOW_ID}}/publish" \
     -H "Authorization: Bearer {{ACCESS_TOKEN}}"
   ```

## JSON mínimo

```json
{
  "version": "5.0",
  "screens": [
    {
      "id": "WELCOME",
      "title": "Reservá tu turno",
      "data": {},
      "layout": {
        "type": "SingleColumnLayout",
        "children": [
          {
            "type": "TextHeading",
            "text": "Elegí servicio y fecha"
          },
          {
            "type": "Dropdown",
            "label": "Servicio",
            "name": "service",
            "data-source": [
              { "id": "consulta", "title": "Consulta general" },
              { "id": "control", "title": "Control" },
              { "id": "urgencia", "title": "Urgencia" }
            ],
            "required": true
          },
          {
            "type": "DatePicker",
            "label": "Fecha",
            "name": "date",
            "required": true
          },
          {
            "type": "TextInput",
            "label": "Comentarios",
            "name": "notes",
            "required": false
          },
          {
            "type": "Footer",
            "label": "Reservar",
            "on-click-action": {
              "name": "complete",
              "payload": {
                "service": "${form.service}",
                "date": "${form.date}",
                "notes": "${form.notes}"
              }
            }
          }
        ]
      }
    }
  ]
}
```

| Componente | Función |
|---|---|
| `TextHeading` / `TextBody` / `TextCaption` | Texto |
| `TextInput` | Input de una línea |
| `TextArea` | Input multilínea |
| `Dropdown` | Select |
| `RadioButtonsGroup` | Radio |
| `CheckboxGroup` | Multi-select |
| `DatePicker` | Selector de fecha |
| `Image` | Imagen estática |
| `Footer` | Botón de acción al pie |
| `OptIn` | Checkbox de consentimiento (legal) |
| `EmbeddedLink` | Link inline |

## Enviar un Flow como mensaje

```bash
curl -X POST \
  "https://graph.facebook.com/v21.0/{{PHONE_NUMBER_ID}}/messages" \
  -H "Authorization: Bearer {{ACCESS_TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "to": "{{DESTINATION_E164}}",
    "type": "interactive",
    "interactive": {
      "type": "flow",
      "header": { "type": "text", "text": "Reservá tu turno" },
      "body": { "text": "Completá los datos para confirmar." },
      "footer": { "text": "Solo te lleva 1 minuto" },
      "action": {
        "name": "flow",
        "parameters": {
          "flow_message_version": "3",
          "flow_token": "{{TOKEN_INTERNO}}",
          "flow_id": "{{FLOW_ID}}",
          "flow_cta": "Reservar ahora",
          "flow_action": "navigate",
          "flow_action_payload": {
            "screen": "WELCOME"
          }
        }
      }
    }
  }'
```

| Placeholder | Descripción |
|---|---|
| `FLOW_ID` | ID del Flow publicado |
| `TOKEN_INTERNO` | Token tuyo para correlacionar el Flow completado con el usuario / contexto |
| `flow_cta` | Texto del botón que abre el Flow |

## Webhook de Flow completado

Cuando el usuario completa el Flow, llega como mensaje entrante con `type: "interactive"` y subtipo `nfm_reply`:

```json
{
  "type": "interactive",
  "interactive": {
    "type": "nfm_reply",
    "nfm_reply": {
      "response_json": "{\"flow_token\":\"...\",\"service\":\"consulta\",\"date\":\"2026-06-10\",\"notes\":\"Primera visita\"}",
      "body": "Datos enviados",
      "name": "flow"
    }
  }
}
```

`response_json` es **string**, parseá:

```javascript
const flowData = JSON.parse(msg.interactive.nfm_reply.response_json);
const { flow_token, service, date, notes } = flowData;
```

Con `flow_token` recuperás el contexto (qué usuario, qué reserva en curso, etc.) y procesás.

## Categorías de Flow

Al crear el Flow se declaran categorías para que Meta lo apruebe correctamente:

| Categoría | Cuándo |
|---|---|
| `APPOINTMENT_BOOKING` | Reservas de turnos |
| `LEAD_GENERATION` | Calificación de leads |
| `CUSTOMER_SUPPORT` | Soporte estructurado |
| `SURVEY` | Encuestas |
| `SIGN_UP` | Alta de usuarios |
| `SIGN_IN` | Login (poco común) |
| `OTHER` | Genérico |

## Flows con endpoint (validación dinámica)

Para Flows que necesitan validar o cargar datos en vivo (ej: mostrar horarios disponibles según fecha elegida), se configura un **endpoint** del integrador.

Flujo:

1. Usuario llena la fecha en la pantalla.
2. Flow llama a tu endpoint con `INIT` o `data_exchange`.
3. Tu endpoint devuelve la lista de horarios disponibles para esa fecha.
4. Usuario elige horario y completa.
5. Flow llama a tu endpoint con `BACK` / `complete`.

| Detalle | Valor |
|---|---|
| Endpoint | HTTPS, key pública subida a Meta |
| Cifrado | Payloads cifrados con la clave pública/privada |
| Latencia | Debe ser baja (< 1s ideal) |
| Errores | Devolver con formato específico para que el Flow muestre mensaje al usuario |

Detalle de criptografía y formato en la [documentación oficial de Flows](https://developers.facebook.com/docs/whatsapp/flows).

## Cuándo conviene Flow con endpoint

| Caso | ¿Endpoint? |
|---|---|
| Reserva con horarios fijos publicados | No, hardcode en JSON |
| Reserva con disponibilidad dinámica | Sí |
| Catálogo de productos cambiante | Sí |
| Calificación con preguntas fijas | No |
| Formulario que valida CUIT/RFC contra DB | Sí |

## Patrón en n8n

| Etapa | Nodo |
|---|---|
| Generar `flow_token` único | Code (UUID) + insert en tabla `flow_sessions` |
| Enviar mensaje con Flow | HTTP Request a Cloud API |
| (Opcional) Endpoint para data_exchange | Webhook trigger separado + manejo de cifrado |
| Recibir `nfm_reply` | Webhook entrante con switch para `nfm_reply` |
| Procesar datos completados | Switch sobre `flow_token` o sobre la categoría |
| Confirmar al usuario | Enviar texto/template de confirmación |

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| Flow no abre al tocar el botón | `flow_id` mal o no publicado | Confirmar publicación y permisos |
| Footer no aparece | JSON inválido | Validar contra schema oficial |
| `nfm_reply` no llega tras completar | Webhook no suscrito o filtrado | Verificar suscripción `messages` |
| Datos parseados vacíos | `response_json` es string, no objeto | `JSON.parse` |
| Endpoint del Flow tira 500 | Cifrado mal implementado o JSON inválido en response | Loggear payload entrante, validar formato |
| Multi-pantalla no avanza | `on-click-action: navigate` mal | Verificar `next.name` y screen id |

## Buenas prácticas

- Empezar **sin endpoint** y migrar a endpoint solo si hace falta.
- Mantener Flows cortos (1-3 pantallas).
- Validar inputs en cliente con `required` y `pattern` antes de submitear.
- Mostrar feedback al usuario tras completar (mensaje de confirmación, no silencio).
- Usar `flow_token` único por instancia para correlación.
- Versionar el JSON del Flow en Git.

## Referencias

- [WhatsApp Flows · Meta](https://developers.facebook.com/docs/whatsapp/flows) — Verificado 2026-05-20.
- [Flow JSON Schema](https://developers.facebook.com/docs/whatsapp/flows/reference/flowjson) — Verificado 2026-05-20.
- [Sending a Flow](https://developers.facebook.com/docs/whatsapp/flows/guides/sendingaflow) — Verificado 2026-05-20.
- [`04-mensajeria/interactivos.md`](./interactivos.md)
