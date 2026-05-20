# 08 · Despliegue

Cómo poner en producción los componentes que la wiki cubre. Foco en **Railway** por su modelo de plantillas listas para usar; también docker-compose para self-hosting clásico.

## Subcarpetas

- `railway/` — Conceptos y plantillas listas (Evolution, n8n, stack completo).
- `docker-compose/` — Equivalentes self-hosted en VPS propio.
- `alternativas.md` — Fly.io, Render, Hetzner, VPS bare-metal.

## Plantillas Railway destacadas

| Plantilla | Stack | Cuándo usar |
|---|---|---|
| [Evolution API](./railway/plantilla-evolution-api.md) | Evolution + Postgres + Redis | Prototipos rápidos con WA no oficial. |
| [n8n](./railway/plantilla-n8n.md) | n8n + Postgres + Redis (queue mode) | Orquestador puro. |
| [Cloud API + n8n + OpenAI](./railway/plantilla-cloudapi-n8n-openai.md) | n8n + OpenAI sobre Cloud API directa | Producción oficial con IA. |
| [Stack completo](./railway/plantilla-evolution-n8n-openai.md) | Evolution + n8n + Postgres + Redis + Caddy | Stack todo-en-uno self-hosted en Railway. |
