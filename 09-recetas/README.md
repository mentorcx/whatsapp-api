# 09 · Recetas end-to-end

Workflows reproducibles que combinan WhatsApp con n8n, OpenAI, Kommo, etc. Cada receta es un caso real con todos los artefactos necesarios para implementarlo.

## Estructura obligatoria de cada receta

1. Frontmatter completo con `stack`, `complejidad`, `tiempo_estimado`.
2. TL;DR.
3. Diagrama Mermaid.
4. Plantillas HSM necesarias (texto, categoría, variables).
5. Esqueleto JSON del workflow de n8n (o link a `examples/`).
6. Prompts de OpenAI con function calling si aplica.
7. Variables de entorno requeridas.
8. Cómo probarla.

## Recetas planeadas

- `bot-atencion-handoff.md` — Bot conversacional con derivación a humano.
- `agente-ventas-catalogo.md` — Agente con catálogo de productos.
- `recordatorios-turnos.md` — Recordatorios con confirmación por botones.
- `nps-post-venta.md` — Encuesta NPS automatizada.
- `rag-kb.md` — RAG sobre base de conocimiento del cliente.
- `kommo-cloudapi-openai-handoff.md` — Stack con Kommo como UI de agentes.
- `kommo-n8n-rag.md` — Salesbot de Kommo consultando RAG vía n8n.
