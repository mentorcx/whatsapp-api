# Kommo CRM

CRM con foco en mensajería. Tratamiento de primer nivel por su fuerte adopción en LATAM.

## Documentos planeados

- `conceptos.md` — Lead, pipeline, salesbot, chats.
- `conexion-whatsapp.md` — WA Lite vs Cloud API vs proveedor externo (Wazzup, etc.).
- `salesbot.md` — Motor de bots nativo, bloques, condiciones.
- `webhooks.md` — Eventos entrantes (lead status, incoming message, bot ended).
- `api-rest.md` — Endpoints clave, autenticación con long-lived tokens.
- `integracion-n8n.md` — Patrones puente Kommo ↔ n8n.
- `integracion-openai.md` — Salesbot llamando a un endpoint con LLM.
- `limitaciones.md` — Rate limits, delays de Salesbot.

## Patrón híbrido recomendado

Kommo como CRM/UI de agentes humanos + n8n como orquestador + OpenAI como cerebro + Cloud API o Evolution como transporte. Ver recetas en [`09-recetas/kommo-cloudapi-openai-handoff.md`](../../09-recetas/kommo-cloudapi-openai-handoff.md).
