---
title: Prompts sugeridos para consultar la wiki
category: meta
tags: [llm, prompts, claude, chatgpt, gemini]
updated: 2026-05-20
---

# Prompts sugeridos

> Plantillas de prompt para usar esta wiki como contexto en Claude, ChatGPT o Gemini. Los placeholders van entre `{{ }}`.

## 1. Cargar contexto base

```
Sos un asistente experto en WhatsApp Business Platform.
Tu única fuente de verdad es la wiki ubicada en {{REPO_URL}}.
Empezá leyendo llms.txt. Si te piden algo que no está en la wiki, decilo
explícitamente en lugar de inventar. Citá siempre el path del archivo
del que sacaste la información.
```

## 2. Diagnóstico de error

```
Tengo este error trabajando con {{Cloud API | Evolution API | n8n}}:

{{PEGAR_ERROR}}

Diagnosticá usando:
- 10-troubleshooting/codigos-error.md
- 10-troubleshooting/webhooks-no-llegan.md
- 10-troubleshooting/tokens.md

Dame: causa probable, archivo de la wiki que lo respalda, pasos para
resolverlo, y un comando de verificación.
```

## 3. Diseñar una receta nueva

```
Quiero armar un workflow que: {{DESCRIPCION_CASO_DE_USO}}.

Restricciones:
- Stack: {{n8n + OpenAI + Cloud API | Evolution API + n8n | Kommo + n8n + OpenAI}}
- Hosting: {{Railway | self-hosted}}
- Volumen estimado: {{N}} mensajes/día

Basate en las recetas existentes en 09-recetas/ y en los patrones de
07-integraciones/. Devolvé:
1. Diagrama Mermaid del flujo.
2. Lista de plantillas HSM necesarias y categoría sugerida.
3. Esqueleto JSON del workflow de n8n.
4. Prompts de OpenAI con function calling si aplica.
5. Variables de entorno requeridas.
```

## 4. Elegir solución

```
Para un cliente con estas características:
- Industria: {{INDUSTRIA}}
- Volumen mensual: {{N}} conversaciones
- Equipo: {{N}} agentes humanos
- Necesita: {{REQUISITOS}}

Comparalos usando 03-formas-de-uso/comparativa.md y recomendá una
opción entre Cloud API directa, Evolution API, Respond.io o Kommo.
Justificá con costos, riesgos y curva de implementación.
```

## 5. Auditoría de cuenta

```
Vamos a auditar una cuenta de WhatsApp Business. Pediime de a una:
1. Estado de verificación del Business Portfolio.
2. Tier de mensajería actual.
3. Quality rating del número.
4. Plantillas activas, pausadas, rechazadas.
5. Webhooks configurados y health.

Para cada hallazgo, citá el archivo de la wiki con el criterio aplicado.
```

## 6. Preparar despliegue en Railway

```
Quiero desplegar {{Evolution API | n8n | stack completo}} en Railway.
Usá la plantilla documentada en 08-despliegue/railway/ que aplique.
Devolvé:
- Servicios a crear.
- Variables de entorno con valores sugeridos (marcando cuáles son secretos).
- Tamaño de volumen.
- Pasos para apuntar dominio propio y configurar el webhook en Meta.
- Estimación de costo mensual.
```

## 7. Revisión de plantilla HSM

```
Esta es una plantilla que quiero enviar a aprobación:

Categoría propuesta: {{Marketing | Utility | Authentication}}
Texto: {{TEXTO}}
Variables: {{LISTA}}
Botones: {{LISTA}}

Revisala contra 05-plantillas-hsm/aprobacion.md y 05-plantillas-hsm/rechazos.md.
Indicá: probabilidad de aprobación, categoría correcta, sugerencias de redacción,
y riesgos de pausing post-aprobación.
```

## Notas para autores de prompts

- Mantener prompts cortos pero con referencia explícita a paths de la wiki.
- Pedir siempre que cite el archivo fuente.
- Pedir que admita "no lo sé" si no está en la wiki, para evitar alucinaciones.
