# 07 · Integraciones

Cómo conectar WhatsApp con los orquestadores y plataformas que más usamos.

## Subcarpetas

- `n8n/` — Nodo oficial, patrones de orquestación, subflows.
- `openai/` — Function calling, contexto, moderación, costos.
- `evolution-api/` — Endpoints, eventos, instancias, diferencias con Cloud.
- `respond-io/` — Workflows, webhooks, dynamic variables.
- `kommo/` — Salesbot, API REST, webhooks, integración con n8n y OpenAI.

## Patrón híbrido recomendado

```
Usuario WhatsApp
    ↓
WhatsApp (Cloud API o Evolution API)   ← Transporte
    ↓ webhook
n8n                                     ← Orquestación
    ↓ ↑
OpenAI                                  ← Cerebro (NLU + respuestas)
    ↓ ↑
Kommo / CRM                             ← Estado, UI agentes, handoff
```

Cada nodo del diagrama tiene su sección dedicada con patrones, ejemplos y errores comunes.
