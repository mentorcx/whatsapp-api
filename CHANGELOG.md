# Changelog

Todos los cambios relevantes a esta wiki se documentan acá. Formato basado en [Keep a Changelog](https://keepachangelog.com/es-ES/).

## [Unreleased]

### Added
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
