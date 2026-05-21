---
title: Costos de OpenAI en bots de WhatsApp
category: integraciones-openai
tags: [openai, costos, tokens, pricing, optimizacion]
updated: 2026-05-20
source_official:
  - https://openai.com/api/pricing/
  - https://platform.openai.com/docs/guides/prompt-caching
related:
  - 07-integraciones/openai/contexto-conversacion.md
  - 06-politicas-y-calidad/pricing.md
audience: [integrador, comercial, dev]
---

# Costos de OpenAI en bots de WhatsApp

> **TL;DR:** OpenAI cobra por tokens (entrada + salida). Para un bot de WhatsApp con `gpt-4o-mini`, una conversación típica de ~10 turnos cuesta del orden de **$0.002-0.01**. El costo de OpenAI suele ser **menor** que el de las conversaciones de WhatsApp (Meta). Optimizaciones clave: modelo adecuado, ventana de contexto acotada, prompt caching.

## Contexto

Una pregunta recurrente del cliente: "¿cuánto sale el bot?". Hay que separar tres costos: **OpenAI** (este documento), **conversaciones de WhatsApp** (Meta, ver [`06-politicas-y-calidad/pricing.md`](../../06-politicas-y-calidad/pricing.md)) y **hosting**. Este documento estima el primero.

## Cómo cobra OpenAI

Por **tokens**, separando entrada y salida:

| Componente | Qué incluye |
|---|---|
| Tokens de entrada (input) | System prompt + historial + mensaje del usuario + tools |
| Tokens de salida (output) | La respuesta generada por el modelo |
| Tokens cacheados | Input repetido que se cobra con descuento (prompt caching) |

Output suele costar más por token que input. Embeddings tienen su propio precio (mucho más barato).

## Estimación de tokens

| Regla práctica | Valor |
|---|---|
| 1 token | ~4 caracteres en español |
| Mensaje de usuario típico | 20-60 tokens |
| Respuesta del bot | 100-300 tokens |
| System prompt mediano | 300-800 tokens |
| Definición de tools (5 tools) | 200-500 tokens |
| Ventana de 10 mensajes | 500-1.500 tokens |
| Contexto RAG (5 chunks) | 1.000-3.000 tokens |

## Costo por turno (orden de magnitud)

Con `gpt-4o-mini`, un turno típico de un bot conversacional:

| Escenario | Input aprox | Output aprox | Costo aprox/turno |
|---|---|---|---|
| Bot simple (sin tools, ventana corta) | ~800 tok | ~150 tok | ~$0.0002 |
| Bot con tools + ventana media | ~1.500 tok | ~200 tok | ~$0.0004 |
| Bot con RAG (contexto grande) | ~3.500 tok | ~250 tok | ~$0.0008 |
| Function calling con 2 llamadas (tool + final) | ~2× lo anterior | | ~2× |

| Conversación completa | Turnos | Costo aprox |
|---|---|---|
| Consulta corta | 3-5 | ~$0.001-0.003 |
| Conversación media | 10 | ~$0.003-0.008 |
| Conversación larga / con RAG | 20+ | ~$0.01-0.03 |

**Verificado: 2026-05-20** — son órdenes de magnitud; los precios exactos cambian, consultar [pricing oficial](https://openai.com/api/pricing/).

## Comparación: OpenAI vs WhatsApp vs hosting

Para 1.000 conversaciones/mes:

| Costo | Estimación mensual |
|---|---|
| OpenAI (`gpt-4o-mini`, ~10 turnos/conv) | ~$3-8 |
| Conversaciones WhatsApp (mix service/utility/marketing) | ~$50-200 (depende del país y categorías) |
| Hosting (n8n + DB) | ~$15-30 |
| **Total** | **~$70-240** |

Conclusión: **OpenAI no es el costo dominante**. El gasto fuerte está en las conversaciones de Meta. Optimizar agresivamente OpenAI a costa de calidad rara vez vale la pena.

## Elegir el modelo según costo/calidad

| Modelo | Costo relativo | Cuándo |
|---|---|---|
| `gpt-4o-mini` | Bajo | Default para bots conversacionales; alcanza casi siempre |
| `gpt-4o` | Medio | Conversaciones complejas, muchas tools, razonamiento |
| `gpt-4.1` / superiores | Más alto | Casos que lo justifiquen y haya presupuesto |
| Embeddings `text-embedding-3-small` | Muy bajo | RAG; suficiente para casi todo |

Regla: arrancar con `gpt-4o-mini`. Subir de modelo solo si la calidad observada lo exige, no "por las dudas".

## Optimizaciones reales

### 1. Acotar la ventana de contexto

No mandar 50 mensajes de historial. Ventana deslizante de 8-12 mensajes. Ver [`contexto-conversacion.md`](./contexto-conversacion.md).

| Efecto | Detalle |
|---|---|
| Reduce input tokens | El costo dominante en conversaciones largas |
| Más rápido | Menos tokens = menos latencia |

### 2. Prompt caching

Si el system prompt + tools son grandes y estables, ponerlos **al principio** del array para que el proveedor los cachee. El input cacheado se cobra con descuento.

| Estructura | Resultado |
|---|---|
| Estable primero (system, tools, contexto del negocio) | Cache hit alto |
| Variable después (historial, mensaje actual) | — |

### 3. Una sola llamada cuando alcanza

En function calling, la segunda llamada (para generar la respuesta natural tras el tool result) duplica el costo del turno. Para acciones simples, generar la respuesta con código a partir del resultado, sin segunda llamada al modelo. Ver [`function-calling.md`](./function-calling.md).

### 4. System prompt conciso

Un system prompt de 2.000 tokens se paga en **cada** llamada. Escribirlo denso, sin relleno. Mover info estática extensa a RAG en vez de meterla toda en el prompt.

### 5. Limitar `max_tokens` de salida

Las respuestas de WhatsApp deben ser cortas igual (UX). Limitar `max_tokens` evita respuestas largas innecesarias y caras.

### 6. No llamar al modelo cuando no hace falta

| Caso | Sin LLM |
|---|---|
| El usuario tocó un botón con `id` conocido | Switch directo |
| FAQ exacta | Match de keyword |
| Saludo / despedida | Respuesta canned |

Reservar el LLM para lo que realmente necesita lenguaje natural.

## Presupuestar para un cliente

Fórmula simple:

```
Costo OpenAI mensual ≈ conversaciones/mes × turnos promedio × costo/turno
```

Ejemplo: 2.000 conversaciones × 8 turnos × $0.0004 = **~$6.4/mes**.

Agregar margen y redondear hacia arriba. Lo importante: comunicar al cliente que OpenAI es la parte chica; el grueso es Meta.

## Monitoreo de costos

| Práctica | Detalle |
|---|---|
| Loggear `usage` de cada respuesta | La API devuelve `prompt_tokens`, `completion_tokens` |
| Acumular por conversación / cliente | Para reporting y detección de anomalías |
| Alertas de gasto | Configurar límites en el dashboard de OpenAI |
| Revisar conversaciones caras | Una conversación de $0.50 indica un loop o contexto descontrolado |

Guardar el `usage` en la tabla de mensajes:

```sql
ALTER TABLE wa_messages ADD COLUMN prompt_tokens INT;
ALTER TABLE wa_messages ADD COLUMN completion_tokens INT;
```

## Señales de gasto descontrolado

| Señal | Causa probable |
|---|---|
| Conversación con costo 10× lo normal | Loop de tool calls, contexto sin recortar |
| Costo sube sin que suba el volumen | Ventana de contexto creciendo |
| Picos puntuales | Conversaciones con RAG y muchos chunks |
| Gasto en embeddings alto | Re-indexando todo seguido en vez de incremental |

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| Costo OpenAI inesperadamente alto | Historial sin recortar | Ventana deslizante |
| Loop de tool calls cobra de más | Bug en el handler | Limitar tool calls por turno |
| Usar `gpt-4o` para todo "por las dudas" | Sobre-especificación | `gpt-4o-mini` de default |
| Segunda llamada innecesaria | Generás respuesta con LLM cuando alcanzaba código | Respuesta determinística cuando se puede |
| No loggear `usage` | Imposible saber qué cuesta | Guardar tokens por mensaje |

## Referencias

- [OpenAI Pricing](https://openai.com/api/pricing/) — Verificado 2026-05-20.
- [Prompt Caching](https://platform.openai.com/docs/guides/prompt-caching) — Verificado 2026-05-20.
- [`07-integraciones/openai/contexto-conversacion.md`](./contexto-conversacion.md)
- [`06-politicas-y-calidad/pricing.md`](../../06-politicas-y-calidad/pricing.md)
