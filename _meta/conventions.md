---
title: Convenciones de escritura
category: meta
tags: [contribucion, estilo, llm-friendly]
updated: 2026-05-20
---

# Convenciones de escritura

> Reglas para que la wiki sea consistente, navegable por humanos y consumible por LLMs sin alucinaciones.

## 1. Frontmatter obligatorio

Cada `.md` (excepto `README.md` de carpetas) abre con frontmatter YAML:

```yaml
---
title: Título humano del documento
category: carpeta-padre               # ej: numeros-y-conexion
tags: [tag1, tag2]                    # minúsculas, kebab-case
updated: 2026-05-20                   # ISO date
source_official:                      # opcional, lista de URLs oficiales
  - https://developers.facebook.com/docs/...
prerequisites:                        # opcional, paths relativos a la raíz
  - 01-onboarding-meta/business-portfolio.md
related:                              # opcional, paths relativos a la raíz
  - 02-numeros-y-conexion/webhooks.md
audience: [integrador, dev, no-code]  # opcional
---
```

## 2. Estructura de cada documento

```markdown
# Título

> **TL;DR:** 2-3 líneas que un LLM o un humano apurado pueda citar.

## Contexto

Por qué existe este documento, qué problema resuelve.

## Pasos / Detalle / Comparativa

Contenido principal. Usar tablas antes que listas anidadas profundas.

## Errores comunes

Tabla `error → causa → solución`.

## Referencias

- Fuentes oficiales (con fecha de última verificación).
- Docs relacionados de esta wiki.
```

No todas las secciones son obligatorias, pero el orden sí lo es cuando aparecen.

## 3. Estilo de escritura

- **Español neutro**, sin modismos regionales fuertes.
- Frases cortas. Voz activa.
- No usar emojis decorativos.
- No usar ASCII art.
- No abreviar términos canónicos la primera vez (escribir "WhatsApp Business Account (WABA)" la primera vez, luego "WABA").
- Si un término aparece en `glossary.json`, usar exactamente esa grafía.

## 4. Snippets de código

- Declarar siempre el lenguaje del bloque (` ```bash`, ` ```json`, ` ```yaml`).
- Usar placeholders con doble llave: `{{PHONE_NUMBER_ID}}`, `{{ACCESS_TOKEN}}`.
- Documentar cada placeholder en una tabla debajo del snippet.
- Snippets `curl` deben ser reproducibles tal cual, sin dependencias ocultas.

Ejemplo:

```bash
curl -X POST \
  "https://graph.facebook.com/v21.0/{{PHONE_NUMBER_ID}}/messages" \
  -H "Authorization: Bearer {{ACCESS_TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "to": "{{DESTINATION_E164}}",
    "type": "text",
    "text": { "body": "Hola" }
  }'
```

| Placeholder | Descripción | Dónde se obtiene |
|---|---|---|
| `PHONE_NUMBER_ID` | ID del número conectado | Meta Business Suite → WhatsApp → Configuración |
| `ACCESS_TOKEN` | Token del System User | Meta Business Settings → Usuarios del sistema |
| `DESTINATION_E164` | Número destino en formato E.164 sin `+` | Input del integrador |

## 5. Tablas

- Una columna por concepto.
- Encabezados en mayúscula inicial.
- Si una celda crece mucho, mover el contenido a un documento aparte y linkearlo.

## 6. Links

- Internos: rutas relativas desde la raíz del repo (`./04-mensajeria/flows.md`).
- Externos oficiales: pasar también por `_meta/sources.md` con fecha.
- No linkear blogs ni medios secundarios salvo que la información no exista en fuente oficial.

## 7. Tamaño y atomicidad

- Una idea por archivo.
- 200-400 líneas como máximo recomendado.
- Si un documento crece, dividir y dejar un índice en el padre.

## 8. Diagramas

- Preferir [Mermaid](https://mermaid.js.org/) inline en Markdown.
- Si no alcanza, exportar SVG a `assets/` y linkear.
- Nunca dejar diagramas como capturas de pizarra.

## 9. Cambios y versionado

- Cada PR debe actualizar `CHANGELOG.md`.
- Si cambia una política de Meta, actualizar `updated:` del documento y la entrada correspondiente en `_meta/sources.md`.
- Breaking changes en convenciones requieren bump del CHANGELOG con sección "Breaking".

## 10. Reglas específicas para LLM-friendliness

- No escribir contenido relevante solo en imágenes o diagramas.
- Definir siglas la primera vez.
- Si un dato puede quedar desactualizado (precio, tier, versión de API), marcarlo con `**Verificado: YYYY-MM-DD**` al lado.
- Evitar pronombres ambiguos cuando el sujeto está lejos.
- Cuando se documente un endpoint, incluir: método, URL completa, headers, body de ejemplo, response de ejemplo, errores posibles.
