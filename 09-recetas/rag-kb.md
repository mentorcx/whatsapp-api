---
title: "Receta: RAG sobre base de conocimiento"
category: recetas
tags: [n8n, openai, rag, embeddings, vector-db, cloud-api]
updated: 2026-05-20
stack:
  - WhatsApp Cloud API (o Evolution API)
  - n8n
  - OpenAI (embeddings + chat)
  - Vector DB (Qdrant / pgvector / Pinecone)
complejidad: media-alta
tiempo_estimado: 8-12 horas
prerequisites:
  - 07-integraciones/openai/function-calling.md
  - 07-integraciones/openai/contexto-conversacion.md
  - 09-recetas/bot-atencion-handoff.md
related:
  - 09-recetas/kommo-n8n-rag.md
audience: [integrador, dev]
---

# Receta: RAG sobre base de conocimiento

> **TL;DR:** Bot de WhatsApp que responde usando **Retrieval-Augmented Generation**: indexa los documentos del cliente (FAQ, manuales, políticas) como embeddings en una vector DB, y en cada consulta recupera los fragmentos relevantes y se los pasa al LLM como contexto. Resultado: respuestas fundadas en la documentación real del cliente, sin alucinar. Stack: WA + n8n + OpenAI + vector DB.

## Caso de uso

Un cliente tiene mucha documentación (catálogo de servicios, FAQ extensa, manual de producto, políticas de garantía) y quiere que el bot responda con precisión sobre eso, sin inventar. Requisitos:

- Respuestas basadas en la documentación real, citables.
- Actualizar la base sin reentrenar nada.
- "No sé" cuando la respuesta no está en la documentación.
- Escalar a humano si el usuario lo pide o si no hay match.

## Qué es RAG y por qué

| Enfoque | Problema |
|---|---|
| Meter toda la doc en el system prompt | No escala, caro, supera el límite de contexto |
| Fine-tuning del modelo | Caro, lento de actualizar, sigue pudiendo alucinar |
| **RAG** | Recupera solo lo relevante por consulta, actualizable, fundado |

RAG = en cada pregunta, buscar los fragmentos más parecidos en una base vectorial y dárselos al LLM como contexto.

## Arquitectura

```mermaid
flowchart TD
  subgraph Ingesta (offline)
    DOCS[Documentos del cliente] --> CHUNK[Chunking]
    CHUNK --> EMB1[OpenAI embeddings]
    EMB1 --> VDB[(Vector DB)]
  end

  subgraph Runtime
    U[Usuario WhatsApp] --> WH[Webhook n8n]
    WH --> EMB2[Embedding de la consulta]
    EMB2 --> SEARCH[Buscar top-K en Vector DB]
    SEARCH --> VDB
    VDB --> CTX[Fragmentos relevantes]
    CTX --> LLM[OpenAI chat:<br/>system + contexto + pregunta]
    LLM --> ANSWER[Respuesta fundada]
    ANSWER --> WA[Enviar a WA]
  end
```

## Componentes

| Componente | Tecnología | Rol |
|---|---|---|
| Transporte | Cloud API o Evolution | Enviar/recibir |
| Orquestador | n8n | Webhook, ingesta, runtime |
| Embeddings | OpenAI `text-embedding-3-small` | Vectorizar texto |
| Vector DB | Qdrant, pgvector, o Pinecone | Búsqueda por similitud |
| LLM | OpenAI `gpt-4o-mini` o superior | Generar respuesta |
| Estado | Postgres | Conversaciones (ver [`bot-atencion-handoff.md`](./bot-atencion-handoff.md)) |

### Elegir vector DB

| Opción | Cuándo |
|---|---|
| **pgvector** | Ya tenés Postgres; volumen chico-medio; menos piezas |
| **Qdrant** | Self-hosted, performante, fácil en Railway/Docker |
| **Pinecone** | Managed, cero ops, costo recurrente |

Recomendado para integradores: **pgvector** (reusa el Postgres que ya tenés) o **Qdrant** self-hosted.

## Parte 1: Ingesta (indexar la documentación)

### Chunking

Partir los documentos en fragmentos. Reglas:

| Parámetro | Recomendación |
|---|---|
| Tamaño de chunk | 300-800 tokens |
| Overlap | 10-15% entre chunks consecutivos |
| Respetar estructura | Cortar en párrafos / secciones, no a mitad de frase |
| Metadata por chunk | `source`, `title`, `section`, `url` |

### Schema con pgvector

```sql
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE kb_chunks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id TEXT NOT NULL,
  source TEXT NOT NULL,            -- nombre del documento
  title TEXT,
  section TEXT,
  content TEXT NOT NULL,
  embedding vector(1536),          -- dim de text-embedding-3-small
  metadata JSONB DEFAULT '{}'::jsonb,
  indexed_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_kb_embedding ON kb_chunks
  USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);

CREATE INDEX idx_kb_tenant ON kb_chunks(tenant_id);
```

### Workflow de ingesta (n8n)

1. **Trigger**: manual, o webhook cuando se sube un documento nuevo.
2. **Leer el documento** (PDF, Markdown, texto): nodo de extracción de texto.
3. **Code**: chunking con overlap.
4. **Loop** por chunk:
   - **HTTP Request** a OpenAI embeddings:
     ```bash
     curl -X POST "https://api.openai.com/v1/embeddings" \
       -H "Authorization: Bearer {{OPENAI_API_KEY}}" \
       -H "Content-Type: application/json" \
       -d '{ "model": "text-embedding-3-small", "input": "{{CHUNK_TEXT}}" }'
     ```
   - **Postgres INSERT** en `kb_chunks` con el embedding.
5. Marcar el documento como indexado.

### Re-indexar

Cuando la documentación cambia: borrar los chunks de ese `source` y re-indexar.

```sql
DELETE FROM kb_chunks WHERE tenant_id = '{{T}}' AND source = '{{DOC}}';
```

No requiere reentrenar nada: solo re-vectorizar el documento cambiado.

## Parte 2: Runtime (responder consultas)

### Workflow

1. **Webhook** → validar firma → dedup → parsear.
2. **HTTP Request** a OpenAI embeddings: vectorizar la pregunta del usuario.
3. **Postgres**: búsqueda por similitud:
   ```sql
   SELECT content, source, title, section,
          1 - (embedding <=> '{{QUERY_EMBEDDING}}') AS similarity
   FROM kb_chunks
   WHERE tenant_id = '{{TENANT}}'
   ORDER BY embedding <=> '{{QUERY_EMBEDDING}}'
   LIMIT 5;
   ```
4. **Code**: filtrar por umbral de similitud (ej: `similarity > 0.75`). Si nada supera el umbral → "no encontrado".
5. **HTTP Request** a OpenAI chat: system prompt + contexto recuperado + pregunta.
6. **HTTP Request** a Cloud API: enviar respuesta.
7. **Postgres**: guardar en el histórico de conversación.

### System prompt para RAG

```
Sos el asistente de {{NEGOCIO}}. Respondés consultas por WhatsApp
basándote ÚNICAMENTE en la información de contexto que te paso abajo.

Reglas estrictas:
- Si la respuesta está en el contexto, respondé de forma clara y breve.
- Si la respuesta NO está en el contexto, decí honestamente que no tenés
  esa información y ofrecé derivar a un humano. NO inventes.
- No menciones "el contexto" ni "los documentos" al usuario; respondé natural.
- Citá la fuente cuando sea útil (ej: "según nuestra política de garantía...").
- Respondé en español, tono cordial, máximo 3 párrafos cortos.

CONTEXTO:
{{FRAGMENTOS_RECUPERADOS}}

Si el CONTEXTO está vacío o no es relevante, indicá que no tenés la
información y sugerí hablar con un agente.
```

### Construir el bloque de contexto

```javascript
const chunks = $json.retrievedChunks; // del paso de búsqueda
if (chunks.length === 0) {
  return [{ json: { context: '', noMatch: true } }];
}
const context = chunks
  .map((c, i) => `[Fuente ${i + 1}: ${c.source}${c.section ? ' - ' + c.section : ''}]\n${c.content}`)
  .join('\n\n---\n\n');
return [{ json: { context, noMatch: false } }];
```

## Manejo de "no encontrado"

Cuando ningún chunk supera el umbral:

| Opción | Comportamiento |
|---|---|
| Responder honesto + ofrecer handoff | "No tengo esa información. ¿Querés que te conecte con alguien del equipo?" |
| Loggear la consulta sin match | Para ampliar la documentación |
| NO dejar que el LLM invente | El system prompt ya lo previene; el umbral es el segundo filtro |

Las consultas sin match son **oro**: indican gaps en la documentación. Loggearlas en una tabla `kb_gaps`.

## RAG como tool (alternativa avanzada)

En vez de hacer RAG siempre, exponerlo como una **tool** de function calling:

```json
{
  "type": "function",
  "function": {
    "name": "search_knowledge_base",
    "description": "Busca en la documentación del negocio. Usar cuando el usuario hace una pregunta sobre productos, servicios, políticas o procedimientos.",
    "parameters": {
      "type": "object",
      "properties": {
        "query": { "type": "string", "description": "La consulta a buscar" }
      },
      "required": ["query"]
    }
  }
}
```

Así el LLM decide cuándo buscar (no busca si el usuario solo saluda). Ver [`07-integraciones/openai/function-calling.md`](../07-integraciones/openai/function-calling.md).

## Variables de entorno

| Variable | Descripción | Secreto |
|---|---|---|
| `WA_ACCESS_TOKEN` | Token System User | Sí |
| `WA_PHONE_NUMBER_ID` | ID del número | No |
| `WA_APP_SECRET` | Validación firma | Sí |
| `OPENAI_API_KEY` | API key | Sí |
| `OPENAI_EMBED_MODEL` | `text-embedding-3-small` | No |
| `OPENAI_CHAT_MODEL` | `gpt-4o-mini` | No |
| `POSTGRES_URL` | Con extensión pgvector | Sí |
| `RAG_SIMILARITY_THRESHOLD` | Umbral (ej `0.75`) | No |
| `RAG_TOP_K` | Cuántos chunks recuperar (ej `5`) | No |

## Cómo probar

1. Indexar un documento de prueba (FAQ corta).
2. Verificar que `kb_chunks` tiene filas con embeddings.
3. Preguntar algo que SÍ está en la doc → verificar respuesta correcta y fundada.
4. Preguntar algo que NO está → verificar que responde "no sé" y ofrece handoff, sin inventar.
5. Preguntar algo ambiguo → verificar recuperación razonable.
6. Cambiar el documento, re-indexar, verificar que la respuesta cambió.
7. Revisar `kb_gaps` para ver consultas sin match.

## Tuning

| Parámetro | Efecto | Ajuste |
|---|---|---|
| `top_k` | Cuántos chunks al LLM | 3-7; más = más contexto pero más ruido |
| Umbral de similitud | Filtro de relevancia | 0.7-0.8; subir si trae basura |
| Tamaño de chunk | Granularidad | Chunks chicos = preciso pero fragmentado |
| Overlap | Continuidad | 10-15% |
| Modelo de embeddings | Calidad vs costo | `3-small` alcanza casi siempre |

## Costos

| Concepto | Estimación |
|---|---|
| Embeddings de ingesta | ~$0.02 por millón de tokens (`3-small`); un manual entero: centavos |
| Embedding de cada consulta | Despreciable |
| Chat por respuesta | ~$0.0003-0.001 (`gpt-4o-mini`, con contexto RAG) |
| Vector DB self-hosted (Qdrant/pgvector) | Solo hosting |
| 1.000 consultas/mes | ~$1-2 en OpenAI |

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| El bot inventa datos | Sin umbral o system prompt débil | Aplicar umbral + prompt estricto |
| Recupera chunks irrelevantes | Chunks muy grandes o umbral bajo | Chunks más chicos, subir umbral |
| Respuestas incompletas | `top_k` muy bajo o chunk corta info | Subir `top_k`, revisar chunking |
| Ingesta lenta | Embeddings uno por uno | Batch de embeddings (la API acepta arrays) |
| Dim mismatch en pgvector | Modelo de embedding distinto al del schema | `text-embedding-3-small` = 1536 dims |
| Cruza datos entre clientes | Falta filtro `tenant_id` | Filtrar siempre por tenant |

## Variantes

| Variante | Cambio |
|---|---|
| Con Kommo | Ver [`kommo-n8n-rag.md`](./kommo-n8n-rag.md) |
| Híbrido (keyword + vector) | Combinar full-text search con similitud |
| Con re-ranking | Pasar los top-K por un re-ranker antes del LLM |
| Multi-idioma | Embeddings multilingües; indexar en el idioma original |
| Con citas clickeables | Devolver links a la fuente en la respuesta |

## Referencias

- [Embeddings · OpenAI](https://platform.openai.com/docs/guides/embeddings) — Verificado 2026-05-20.
- [pgvector](https://github.com/pgvector/pgvector) — Verificado 2026-05-20.
- [Qdrant](https://qdrant.tech/documentation/) — Verificado 2026-05-20.
- [`07-integraciones/openai/function-calling.md`](../07-integraciones/openai/function-calling.md)
- [`09-recetas/bot-atencion-handoff.md`](./bot-atencion-handoff.md)
