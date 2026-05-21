---
title: Patrones de prompts para asistentes en WhatsApp
category: integraciones-openai
tags: [openai, prompts, system-prompt, prompt-engineering]
updated: 2026-05-20
source_official:
  - https://platform.openai.com/docs/guides/prompt-engineering
related:
  - 07-integraciones/openai/function-calling.md
  - 07-integraciones/openai/contexto-conversacion.md
audience: [integrador, dev]
---

# Patrones de prompts para asistentes en WhatsApp

> **TL;DR:** Un bot de WhatsApp necesita un system prompt que cubra: rol, tono, **formato breve** (WhatsApp no es un chat de escritorio), límites estrictos (no inventar), cuándo escalar a humano, y el contexto del negocio. WhatsApp impone restricciones de formato (sin markdown rico, mensajes cortos) que el prompt debe respetar.

## Contexto

El system prompt es el documento que define cómo se comporta el bot. En WhatsApp hay particularidades que un prompt genérico de chatbot ignora: la gente espera respuestas cortas, no hay markdown rico, y la conversación es móvil. Este documento da la estructura y los patrones.

## Anatomía de un system prompt para WhatsApp

```
1. ROL          — quién es el asistente, de qué negocio
2. OBJETIVO     — qué tiene que lograr
3. TONO         — cómo habla
4. FORMATO      — reglas específicas de WhatsApp
5. LÍMITES      — qué NO hacer (no inventar, no prometer)
6. ESCALAMIENTO — cuándo derivar a humano
7. HERRAMIENTAS — qué tools tiene y cuándo usarlas
8. CONTEXTO     — info del negocio
```

## Plantilla base

```
# ROL
Sos el asistente virtual de {{NEGOCIO}}, un/una {{RUBRO}}.
Atendés a clientes por WhatsApp.

# OBJETIVO
Ayudás a los clientes a {{OBJETIVO: resolver consultas / agendar / comprar}}.
Resolvés lo que podés y derivás a una persona cuando hace falta.

# TONO
- Cordial, cercano y profesional.
- Tuteo (o voseo) consistente: usá siempre "{{vos/tú}}".
- Sin exceso de formalidad ni de informalidad.

# FORMATO (WhatsApp)
- Respuestas BREVES: máximo 2-3 párrafos cortos.
- Una idea por mensaje. Si hay mucho que decir, priorizá.
- Sin markdown de títulos (#) ni tablas: WhatsApp no las renderiza.
- Para resaltar usá *negrita* con asteriscos, con moderación.
- Listas cortas con guiones, no numeraciones largas.
- Sin emojis, o como máximo uno ocasional.

# LÍMITES
- NO inventes precios, horarios, stock, plazos ni políticas.
- Si no sabés algo, decilo y ofrecé derivar a una persona.
- No prometas nada que no esté confirmado.
- No pidas datos sensibles (contraseñas, datos de tarjeta).
- No hables de temas ajenos al negocio.

# ESCALAMIENTO
Derivá a un humano (tool request_handoff) cuando:
- El cliente lo pide explícitamente.
- La consulta excede lo que podés responder.
- Detectás enojo o frustración.
- El tema es {{TEMAS_SENSIBLES: reclamos / temas legales / etc.}}.

# HERRAMIENTAS
- search_products: buscar en el catálogo.
- request_handoff: derivar a una persona.
{{OTRAS_TOOLS}}

# CONTEXTO DEL NEGOCIO
{{INFO: horarios, ubicación, servicios, políticas clave}}
```

## Las reglas de formato son críticas

WhatsApp **no** renderiza como un chat de escritorio. El prompt debe forzar:

| Regla | Por qué |
|---|---|
| Respuestas breves | El usuario lee en el celular, en movimiento |
| Sin `#` títulos | WhatsApp los muestra como texto literal |
| Sin tablas markdown | No se renderizan, quedan ilegibles |
| `*negrita*`, `_cursiva_`, `~tachado~` | Es el único formato que WA soporta |
| Una idea por mensaje | Mejor 2 mensajes cortos que uno gigante |
| Sin bloques de código largos | Ilegibles en móvil |

Un bot que responde con párrafos enormes y markdown de escritorio se siente roto. El prompt es lo que lo previene.

## Patrón: respuestas cortas forzadas

Reforzar la brevedad en varios lugares:

```
# FORMATO
Respondé en 2-3 oraciones cuando sea posible. Nunca más de 3 párrafos
cortos. Si el cliente necesita más detalle, ofrecé profundizar en vez
de volcar todo de una.
```

Y limitar `max_tokens` de salida como respaldo técnico.

## Patrón: anti-alucinación

El error más caro de un bot: inventar. Reforzar:

```
# LÍMITES
Tu información proviene ÚNICAMENTE del CONTEXTO y de los resultados de
las herramientas. Si una pregunta no se puede responder con eso:
1. NO inventes una respuesta.
2. Decí honestamente que no tenés esa información.
3. Ofrecé derivar a una persona o consultar.

Ejemplos de lo que NO debés hacer:
- Inventar un precio "aproximado".
- Suponer un horario.
- Prometer un plazo de entrega sin dato.
```

Combinar con RAG y umbral de similitud (ver [`09-recetas/rag-kb.md`](../../09-recetas/rag-kb.md)).

## Patrón: personalidad consistente

```
# TONO
Sos {{NOMBRE_DEL_BOT}}. Hablás de forma {{cálida y cercana}}.
Usás siempre {{voseo}}. No cambiás de registro.
Si el cliente es informal, acompañás sin perder profesionalismo.
No te disculpás en exceso. No usás muletillas.
```

Definir un nombre para el bot ayuda a la consistencia, aunque no se revele al usuario.

## Patrón: manejo del primer mensaje

El primer turno marca la conversación:

```
# PRIMER CONTACTO
En el primer mensaje:
- Saludá brevemente y presentate como asistente de {{NEGOCIO}}.
- Preguntá en qué podés ayudar.
- No vuelques un menú gigante; mantenelo simple.
Si el cliente viene de un anuncio (hay un referral), mencioná el
producto/tema del anuncio para contextualizar.
```

## Patrón: cuándo usar tools

Las tools se eligen por su `description`, pero reforzar en el system prompt ayuda:

```
# HERRAMIENTAS
- Usá search_products APENAS el cliente describa lo que busca, sin
  esperar a tener todos los detalles.
- Usá request_handoff ante cualquier señal de que necesitás un humano;
  es mejor derivar de más que dejar al cliente sin respuesta.
- NO uses herramientas para saludar o charlar; respondé directo.
```

## Patrón: idioma y localización

```
# IDIOMA
Respondé en el mismo idioma que usa el cliente.
Por defecto, español rioplatense (voseo).
Si el cliente escribe en portugués o inglés, respondé en ese idioma.
Usá vocabulario local de {{PAÍS}}: {{ejemplos}}.
```

## Estructura del array de mensajes

El system prompt puede ir en uno o varios mensajes `system`:

```
[system]  — el prompt principal (rol, tono, formato, límites)
[system]  — facts del cliente (nombre, etapa, datos conocidos)
[system]  — contexto RAG recuperado (si aplica)
[user/assistant ...] — historial
[user]    — mensaje actual
```

Lo estable primero (favorece prompt caching). Ver [`contexto-conversacion.md`](./contexto-conversacion.md).

## Ejemplo completo: bot de atención de una clínica

```
# ROL
Sos el asistente virtual de Clínica San Martín. Atendés consultas por
WhatsApp de pacientes y personas interesadas.

# OBJETIVO
Resolvés consultas administrativas (horarios, ubicación, especialidades,
cómo sacar turno) y derivás a una persona para lo demás.

# TONO
Cordial, claro y empático. Voseo. Profesional pero cálido.

# FORMATO
- Respuestas breves: 2-3 oraciones. Máximo 3 párrafos cortos.
- Sin títulos ni tablas. *Negrita* solo para datos clave (horarios).
- Una idea por mensaje.

# LÍMITES
- NUNCA des consejo médico, diagnósticos ni dosis.
- No inventes precios, disponibilidad de turnos ni nombres de profesionales.
- Si no sabés algo, derivá.

# ESCALAMIENTO
Usá request_handoff cuando:
- Sea una consulta médica (síntomas, tratamiento, medicación).
- El paciente quiera sacar/cambiar un turno y no puedas con las tools.
- Haya un reclamo o el paciente esté molesto.
- El paciente pida hablar con una persona.

# HERRAMIENTAS
- request_handoff(reason, department): derivar a un humano.
- book_appointment(date, time, service): agendar turno.

# CONTEXTO DEL NEGOCIO
Horarios: Lunes a Viernes 8 a 20 hs, Sábados 8 a 13 hs.
Dirección: Av. Siempre Viva 1234, CABA.
Especialidades: clínica médica, pediatría, cardiología, dermatología.
Obras sociales: {{LISTA}}.
Para urgencias, derivar siempre a una persona.
```

## Testing de prompts

| Test | Qué verificar |
|---|---|
| Pregunta dentro del scope | Responde bien y breve |
| Pregunta fuera del scope | Dice "no sé" y deriva, no inventa |
| Cliente molesto | Escala a humano |
| Pregunta médica (clínica) | Escala, no da consejo |
| Mensaje largo del usuario | Respuesta sigue siendo breve |
| Pide markdown / tabla | No la usa, responde plano |
| Cambio de idioma | Responde en el idioma del usuario |

Iterar: si el bot falla un test, ajustar el prompt en la sección correspondiente.

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| Respuestas larguísimas | Prompt no fuerza brevedad | Reforzar FORMATO + `max_tokens` |
| Bot usa `#` y tablas | Prompt no prohíbe markdown rico | Regla explícita de formato WhatsApp |
| Bot inventa datos | Sección LÍMITES débil o ausente | Reforzar anti-alucinación + RAG |
| No deriva nunca a humano | ESCALAMIENTO vago | Listar disparadores concretos |
| Tono inconsistente | TONO no definido | Definir registro y mantenerlo |
| Bot da consejo médico/legal | Sin límite de dominio | Prohibición explícita + escalamiento |
| Prompt enorme y caro | Relleno innecesario | Conciso; lo extenso va a RAG |

## Referencias

- [Prompt Engineering · OpenAI](https://platform.openai.com/docs/guides/prompt-engineering) — Verificado 2026-05-20.
- [`07-integraciones/openai/function-calling.md`](./function-calling.md)
- [`07-integraciones/openai/contexto-conversacion.md`](./contexto-conversacion.md)
- [`_meta/prompt-templates.md`](../../_meta/prompt-templates.md) — prompts para consultar la wiki.
