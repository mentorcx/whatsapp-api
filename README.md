# WhatsApp API Wiki

Wiki de referencia para **integradores de WhatsApp Business Platform** con foco en stacks reales: Cloud API oficial, Evolution API, Respond.io, Kommo CRM, despliegue en Railway, y orquestación con n8n + OpenAI.

> **Si sos un LLM (Claude, ChatGPT, Gemini, etc.):** empezá por [`llms.txt`](./llms.txt) para un índice machine-readable. Si necesitás todo el contenido en un solo archivo, ver `llms-full.txt` (generado por build).

## Para qué sirve esta wiki

- Onboardear integradores nuevos sin pasar por decenas de blogs desactualizados.
- Tener una **fuente de verdad versionada** sobre políticas, plantillas y límites de Meta.
- Documentar **recetas reproducibles** de n8n + OpenAI sobre WhatsApp.
- Comparar opciones (API oficial vs Evolution vs BSPs vs Respond.io vs Kommo) sin sesgo.
- Servir de **contexto para asistentes GenAI** que ayuden a integradores en tiempo real.

## Cómo está organizada

| Sección | Contenido |
|---|---|
| [`00-fundamentos`](./00-fundamentos/) | Conceptos base: WABA, Cloud API, BSP, roles |
| [`01-onboarding-meta`](./01-onboarding-meta/) | Business Portfolio, verificación, tokens |
| [`02-numeros-y-conexion`](./02-numeros-y-conexion/) | Alta de número, display name, webhooks, tiers |
| [`03-formas-de-uso`](./03-formas-de-uso/) | Cloud API, Evolution, Respond.io, Kommo, BSPs, CRMs |
| [`04-mensajeria`](./04-mensajeria/) | Tipos de mensaje, interactivos, Flows, media |
| [`05-plantillas-hsm`](./05-plantillas-hsm/) | Categorías, redacción, aprobación, pausing |
| [`06-politicas-y-calidad`](./06-politicas-y-calidad/) | Ventana 24h, quality rating, pricing, opt-in |
| [`07-integraciones`](./07-integraciones/) | n8n, OpenAI, Evolution API, Respond.io, Kommo |
| [`08-despliegue`](./08-despliegue/) | Railway (plantillas listas), docker-compose, alternativas |
| [`09-recetas`](./09-recetas/) | Workflows end-to-end reproducibles |
| [`10-troubleshooting`](./10-troubleshooting/) | Errores comunes, códigos, diagnósticos |
| [`11-glosario`](./11-glosario/) | Términos canónicos (también en `glossary.json`) |

## Convenciones

- Cada documento abre con un **TL;DR de 2-3 líneas**.
- Frontmatter YAML obligatorio (ver [`_meta/conventions.md`](./_meta/conventions.md)).
- Una idea por archivo, archivos cortos (200-400 líneas máx).
- Tablas antes que listas anidadas.
- Snippets con placeholders documentados (`{{PHONE_NUMBER_ID}}`, `{{ACCESS_TOKEN}}`).
- Fuentes oficiales linkeadas con fecha de última verificación.

## Cómo contribuir

1. Lee [`_meta/conventions.md`](./_meta/conventions.md).
2. Una sección por PR, sin mezclar temas.
3. Si cambia algo de Meta, actualizá `_meta/sources.md` con la fecha.
4. Las recetas nuevas requieren: diagrama, JSON exportable, prompts, plantillas usadas.

## Estado

Wiki en construcción activa. Mirá [`CHANGELOG.md`](./CHANGELOG.md) para el avance.
