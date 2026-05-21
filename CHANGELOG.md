# Changelog

Todos los cambios relevantes a esta wiki se documentan acá. Formato basado en [Keep a Changelog](https://keepachangelog.com/es-ES/).

## [Unreleased]

### Added
- `07-integraciones/kommo/` — sección completa: conceptos, conexión WhatsApp, Salesbot, API REST, webhooks, integración con n8n, integración con OpenAI, limitaciones.
- `07-integraciones/evolution-api/` — instancias, endpoints, eventos webhook, integraciones (n8n, Chatwoot, Typebot).
- `09-recetas/rag-kb.md` — receta RAG: ingesta, embeddings, pgvector, runtime, anti-alucinación.
- `09-recetas/kommo-n8n-rag.md` — RAG sobre Kommo con Salesbot delegando a n8n.
- `09-recetas/agente-ventas-catalogo.md` — agente de ventas con catálogo, tools, carrito, checkout.
- `08-despliegue/railway/conceptos.md` — proyectos, servicios, red interna, variables, volúmenes, costos.
- `08-despliegue/railway/plantilla-n8n.md` — n8n queue mode + worker + Postgres + Redis.
- `08-despliegue/alternativas.md` — Fly.io, Render, Hetzner, Coolify, matriz de decisión.
- `08-despliegue/docker-compose/README.md` — stack self-hosted completo con compose + Caddy.
- `06-politicas-y-calidad/prohibiciones.md` — Commerce/Business Policy, rubros prohibidos y restringidos.
- `00-fundamentos/cloud-api-vs-alternativas.md` — decisión oficial vs no oficial, árbol de decisión.
- `00-fundamentos/roles.md` — los 6 roles, propiedad de activos, anti-patrón de lock-in.
- `02-numeros-y-conexion/migrar-numero.md` — migración desde app, On-Premise, entre WABAs, BSPs.
- `02-numeros-y-conexion/multi-numero.md` — qué se comparte, routing, patrones marketing/soporte/multi-país.
- `07-integraciones/openai/costos.md` — estimación de tokens, comparación con Meta, optimizaciones.
- `07-integraciones/openai/prompts.md` — anatomía del system prompt, formato WhatsApp, anti-alucinación.

### Earlier in Unreleased
- `03-formas-de-uso/cloud-api-oficial.md` — API directa, endpoints, ventajas/desventajas, checklist de producción.
- `03-formas-de-uso/respond-io.md` — SaaS omnicanal, comparación con Kommo/n8n, AI Agent, patrón híbrido.
- `09-recetas/recordatorios-turnos.md` — receta con plantilla Utility, botones, Cron, manejo de no-shows.
- `09-recetas/nps-post-venta.md` — receta de encuesta NPS con ramas por categoría y reporting.
- `07-integraciones/openai/contexto-conversacion.md` — ventana deslizante, summarization, facts, tokens.
- `07-integraciones/n8n/contexto.md` — Postgres vs Redis, schema, upserts, dedup, multi-tenant.
- `10-troubleshooting/rate-limits.md` — throughput, tier, pair limit, backoff, diseño de envíos masivos.
- `10-troubleshooting/evolution-desconexion.md` — árbol de diagnóstico, ban vs resoluble, reconexión.
- `01-onboarding-meta/embedded-signup.md` — flujo multi-cliente, OAuth, App Review, manejo de revocación.
- `02-numeros-y-conexion/requisitos-numero.md` — móvil/fijo, estados previos, países, test number, multi-número.
- `02-numeros-y-conexion/display-name.md` — Name Policy, ejemplos, proceso de aprobación, badge verde.
- `10-troubleshooting/webhooks-no-llegan.md` — checklist completo de diagnóstico + script de verificación.
- `10-troubleshooting/tokens.md` — debug_token, tipos, scopes, casos comunes y comandos.
- `10-troubleshooting/plantillas-pausadas.md` — diagnóstico, fallback automático, patrón intent.
- `05-plantillas-hsm/estructura.md` — header/body/footer/buttons, variables, samples, ejemplos.
- `05-plantillas-hsm/pausing.md` — quality score, prevención, rotación, auditoría.
- `07-integraciones/n8n/patrones.md` — router, dedup, sesión, rate limiting, handoff async, subflows.
- `07-integraciones/openai/function-calling.md` — tools, schema, flow completo, strict mode, costos.
- `04-mensajeria/limites-media.md` — tabla por tipo, formatos, validación, optimización.
- `04-mensajeria/flows.md` — JSON, screens, components, envío, webhook nfm_reply, endpoint dinámico.
- `01-onboarding-meta/business-portfolio.md` — crear Portfolio, asignar activos, dueño correcto.
- `01-onboarding-meta/verificacion-comercial.md` — documentos, tiempos, rechazos frecuentes, verificación facial.
- `01-onboarding-meta/system-users-tokens.md` — tipos de token, scopes, manejo seguro, rotación.
- `06-politicas-y-calidad/pricing.md` — conversation-based pricing, categorías, lectura en webhooks, optimizaciones.
- `06-politicas-y-calidad/opt-in.md` — qué exige Meta, formas válidas, schema de evidencia, jurisdicciones.
- `06-politicas-y-calidad/free-entry-points.md` — CTWA, ventana 72h, referral en webhook, patrón recomendado.
- `04-mensajeria/tipos-de-mensaje.md` — text, media, location, contacts, reaction, reply, upload/download de media.
- `04-mensajeria/interactivos.md` — buttons, list, CTA URL, parsing en webhook, patrones de menú.
- `07-integraciones/n8n/webhook-entrante.md` — implementación completa de referencia con verify, HMAC, dedup y parseo.
- `08-despliegue/railway/plantilla-evolution-api.md` — plantilla standalone (sin n8n) para integrar con sistemas existentes.
- `00-fundamentos/que-es-wa-business-platform.md` — diferencia entre WA Business app y Business Platform, Cloud API vs On-Premise.
- `00-fundamentos/arquitectura.md` — jerarquía Portfolio → WABA → Phone Number → App → Token, roles humanos, patrones de despliegue.
- `02-numeros-y-conexion/webhooks.md` — setup, verificación GET, validación de firma, retries, dedup, payload examples.
- `02-numeros-y-conexion/tiers-mensajeria.md` — tabla de tiers, escalado, warming, multi-número.
- `05-plantillas-hsm/categorias.md` — Marketing/Utility/Authentication, árbol de decisión, pricing, vida de una plantilla.
- `10-troubleshooting/codigos-error.md` — tabla maestra (autenticación, rate limits, mensajes 131x, plantillas 132x, media).
- `09-recetas/kommo-cloudapi-openai-handoff.md` — receta completa con Salesbot delegando a n8n, function calling, pipeline.
- `03-formas-de-uso/evolution-api.md` — análisis completo de Evolution API: cuándo usarla, riesgos, endpoints, eventos, buenas prácticas.
- `03-formas-de-uso/kommo-crm.md` — modelos de conexión a WA (Lite, Cloud, proveedor), Salesbot vs n8n, patrón híbrido.
- `05-plantillas-hsm/aprobacion.md` — checklist y buenas prácticas para aprobación.
- `05-plantillas-hsm/rechazos.md` — catálogo de causas de rechazo con código y corrección.
- `06-politicas-y-calidad/ventana-24h.md` — mecánica completa, tracking, errores conceptuales.
- `06-politicas-y-calidad/quality-rating.md` — estados, cómo consultar, cómo prevenir y recuperarse.
- `09-recetas/bot-atencion-handoff.md` — primera receta end-to-end completa con schema, prompts, tools, plantillas y métricas.

### Earlier
- Estructura base del repositorio.
- `README.md` con índice principal y propósito.
- `llms.txt` como índice machine-readable.
- `_meta/conventions.md` con reglas de escritura y frontmatter.
- `_meta/sources.md` con catálogo de fuentes oficiales.
- `_meta/prompt-templates.md` con prompts sugeridos para Claude / ChatGPT / Gemini.
- `glossary.json` con términos canónicos (WABA, HSM, BSP, etc.).
- README de cada sección con esqueleto de contenidos.
- Documento bandera: `08-despliegue/railway/plantilla-evolution-n8n-openai.md`.
- Tabla comparativa: `03-formas-de-uso/comparativa.md`.

## [0.1.0] - 2026-05-20

- Inicialización del proyecto en branch `claude/whatsapp-api-wiki-3MTIE`.
