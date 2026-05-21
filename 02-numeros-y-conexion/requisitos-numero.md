---
title: Requisitos del número de teléfono
category: numeros-y-conexion
tags: [numero, telefono, requisitos, conexion, e164]
updated: 2026-05-20
source_official:
  - https://developers.facebook.com/docs/whatsapp/cloud-api/get-started/add-a-phone-number
related:
  - 02-numeros-y-conexion/migrar-numero.md
  - 02-numeros-y-conexion/display-name.md
  - 01-onboarding-meta/business-portfolio.md
audience: [integrador, dev, comercial]
---

# Requisitos del número de teléfono

> **TL;DR:** El número que conectás a Cloud API debe: ser **propio del cliente**, **recibir SMS o llamadas** para verificación, **no estar registrado activamente en WhatsApp ni WhatsApp Business app** (o migrarlo primero), y estar en **formato E.164**. Móvil o fijo válidos. Una vez conectado a Cloud API, **no se puede usar en la app de WhatsApp** simultáneamente.

## Contexto

Antes de empezar la integración, elegir y preparar el número correctamente ahorra rollbacks. Errores comunes acá retrasan proyectos varios días.

## Requisitos básicos

| Requisito | Detalle |
|---|---|
| Propiedad | El número debe ser del cliente o de su entidad legal |
| Recepción SMS / llamada | Para el código de verificación inicial |
| Formato | E.164 con código país (`+5491133334444`) |
| País | Soportado por Cloud API (la mayoría lo está) |
| Estado previo | No registrado activamente en WA personal ni Business app |
| Acceso al panel | Acceso al dueño del Portfolio para confirmar |

## Móvil vs fijo

| Tipo | ¿Sirve? | Consideraciones |
|---|---|---|
| Móvil con SIM | Sí | El más común |
| Móvil eSIM | Sí | Igual que SIM |
| Fijo (landline) | Sí | Solo verificación por **llamada de voz**, no SMS |
| VoIP / virtual | Sí (con cuidado) | Algunos proveedores no reciben verificación; testear antes |
| 0800 / 800 | Depende del país y operador | Probar verificación primero |

Para fijo y VoIP, el código llega como **llamada de voz** automática. Hay que tener a alguien atendiendo.

## Estado del número antes de conectar

**No se puede tener el número activo en dos lados simultáneamente.** Estados posibles:

| Estado actual | ¿Listo para Cloud API? | Acción |
|---|---|---|
| Número nunca usado en WA | Sí | Conectar directo |
| Activo en WA personal | No | Pedirle al usuario que **se baje** de WA, esperar 24h, conectar |
| Activo en WA Business app | No | Migrar (ver [`migrar-numero.md`](./migrar-numero.md)) |
| Estaba en Cloud API y fue desconectado | Sí (con cool-down a veces) | Conectar |
| Banneado por WhatsApp | No | Inutilizable, usar otro |

## Países y restricciones

Cloud API opera en la mayoría de los países con WhatsApp. Restricciones conocidas:

| Caso | Detalle |
|---|---|
| Países sancionados | No soportados (varían) |
| China, Cuba, Irán, Corea del Norte, Siria | Limitaciones / no soportados |
| Argentina | Soportado |
| México | Soportado |
| Brasil | Soportado |
| España | Soportado |
| EEUU | Soportado |
| India | Soportado, pricing especial |

Verificar la lista actualizada en docs oficiales si el cliente opera en geografías complicadas.

## Número dedicado o existente

Recomendación según escenario:

| Escenario | Recomendación |
|---|---|
| Cliente con número de atención conocido | Usar ese mismo número (continuidad de marca) |
| Cliente arranca de cero | Comprar número dedicado para WA |
| Cliente con número personal del dueño en WA | Comprar otro, no usar el personal |
| Cliente con número en WA Business app que funciona | Migrar (cuidando timing) |
| Multi-marca | Un número por marca o por canal (ventas/soporte) |

**Regla práctica:** un número dedicado a Cloud API es **lo mejor**. Permite cambios sin afectar comunicaciones personales.

## Cómo agregar el número en la WABA

1. Business Manager → WhatsApp Accounts → WABA → **Phone numbers** → **Add phone number**.
2. Ingresar el número en formato internacional.
3. Ingresar **display name** propuesto (ver [`display-name.md`](./display-name.md)).
4. Elegir categoría del negocio.
5. Recibir código por SMS o llamada.
6. Ingresar el código → número conectado.

Tras conectarse, aparece con un `phone_number_id` listo para usar en endpoints.

## Multi-número en la misma WABA

| Aspecto | Detalle |
|---|---|
| Cuántos | Múltiples (típicamente sin tope técnico, sí prácticos) |
| Verificación independiente | Cada uno se verifica por separado |
| Plantillas compartidas | Las plantillas son a nivel WABA, no número |
| Quality rating | Independiente por número |
| Tier | Independiente por número |
| Display name | Independiente por número |

Útil para:

- Separar marketing (calidad volátil) de soporte (calidad estable).
- Multi-país: un número local por mercado.
- Multi-marca bajo misma empresa.

## Portabilidad

Una vez conectado a Cloud API, **el número se puede mover entre WABAs** (con downtime) y **se puede liberar** para volver a WA Business app o usar en otro lado. No es one-way.

Pasos:

1. Desde el panel: WABA → Phone number → **Remove**.
2. Esperar 24-72h.
3. Conectar a la nueva WABA o usar en WA Business app.

**Importante:** durante el período, el número no recibe mensajes. Avisar a usuarios.

## Lo que NO se puede usar

| Caso | Por qué |
|---|---|
| Número de WA personal del dueño que sigue usándolo en su celular | Se va a desinstalar de su celular cuando lo conectes |
| Número compartido por otra cuenta | Conflicto |
| Número de un servicio shared SMS (Twilio sin reservar) | Cada vez que liberan, lo agarra otro |
| Número que ya fue banneado por WhatsApp | No se puede reactivar |
| Número de prueba de Meta más allá de su límite | Tier muy bajo, no apto para prod |

## Test phone number de Meta (para desarrollo)

Meta provee un **número de prueba** gratis al crear la App, ideal para desarrollo:

| Característica | Detalle |
|---|---|
| Disponibilidad | Gratis con cualquier App |
| Límite | Hasta 5 números **destino** verificados |
| Mensajes salientes | Solo a los 5 destinatarios verificados |
| Mensajes entrantes | Solo desde esos 5 |
| Plantillas | Limitadas, no productivo |
| Pricing | Sin costo |

Útil para:

- Probar el flujo del webhook.
- Validar workflows en n8n.
- Demos cortas sin verificación comercial.

**No** usarlo para clientes reales.

## Display name

El nombre que el usuario ve junto al número (en lugar del número mismo) cuando aún no agendó el contacto. Requiere proceso aparte de aprobación: ver [`display-name.md`](./display-name.md).

## Checklist pre-conexión

| Ítem | ✓ |
|---|---|
| Número dedicado decidido (no personal) | |
| Número en formato E.164 | |
| Acceso para recibir SMS o llamada | |
| Confirmado: no está activo en WA app | |
| Display name propuesto cumple políticas | |
| Cliente entiende que perderá uso de la app de WA en ese número | |
| Categoría de negocio elegida | |
| WABA y System User listos | |

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| Código de verificación no llega | Operador bloquea SMS internacionales de Meta | Probar verificación por llamada |
| "Número ya está registrado" | Está en WA app o en otra WABA | Bajar de WA app o liberar |
| Verificación llega pero falla al ingresarlo | TTL del código vencido (~5min) | Pedir nuevo código |
| Tras conectar, el cliente sigue usando WA en el celular | App no se autodesinstaló | Avisar y desinstalar manualmente |
| País no soportado | Restricción regional | Cambiar a número de otro país |

## Referencias

- [Add a phone number · Cloud API](https://developers.facebook.com/docs/whatsapp/cloud-api/get-started/add-a-phone-number) — Verificado 2026-05-20.
- [Test Numbers](https://developers.facebook.com/docs/whatsapp/cloud-api/get-started) — Verificado 2026-05-20.
- [`02-numeros-y-conexion/migrar-numero.md`](./migrar-numero.md)
- [`02-numeros-y-conexion/display-name.md`](./display-name.md)
