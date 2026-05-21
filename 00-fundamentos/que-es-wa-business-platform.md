---
title: Qué es WhatsApp Business Platform
category: fundamentos
tags: [fundamentos, cloud-api, business-platform, bsp]
updated: 2026-05-20
source_official:
  - https://developers.facebook.com/docs/whatsapp
  - https://business.whatsapp.com/products/business-platform
related:
  - 00-fundamentos/arquitectura.md
  - 00-fundamentos/cloud-api-vs-alternativas.md
  - 03-formas-de-uso/comparativa.md
audience: [integrador, dev, no-code, comercial]
---

# Qué es WhatsApp Business Platform

> **TL;DR:** WhatsApp Business Platform es la oferta de Meta para que **empresas medianas y grandes** se conecten a WhatsApp vía API. Tiene dos versiones: **Cloud API** (hosteada por Meta, recomendada) y la On-Premise API (deprecada). No confundirla con la **app WhatsApp Business**, que está pensada para emprendedores y opera desde un único teléfono.

## Contexto

WhatsApp ofrece tres productos para empresas, en orden creciente de complejidad y capacidades:

| Producto | Para quién | Cómo se opera | Capacidad |
|---|---|---|---|
| WhatsApp (consumidor) | Personas | App móvil + Web | Conversaciones personales |
| **WhatsApp Business app** | Emprendedores, micro PyMEs | App móvil + Web | Catálogo, respuestas rápidas, etiquetas |
| **WhatsApp Business Platform** | Medianas, grandes, integradores | API REST | Multi-agente, automatización, marketing, soporte |

Esta wiki cubre el tercero: **Business Platform**.

## Cloud API vs On-Premise API

Business Platform se ofrecía históricamente en dos modalidades:

| Versión | Estado | Quién hostea | Cuándo elegir |
|---|---|---|---|
| **Cloud API** | Activa, recomendada | Meta | Default. Sin infraestructura propia. |
| **On-Premise API** | Deprecada (sunset 2025) | El integrador o un BSP | Solo legacy. No iniciar nuevos proyectos. |

**Regla:** si arrancás un proyecto nuevo, es Cloud API. Punto.

## Qué te da Cloud API

- Envío y recepción de mensajes vía endpoints HTTP REST (Graph API).
- Webhooks para eventos entrantes (mensajes, estados, alertas de calidad).
- Plantillas (HSM) aprobadas por Meta para mensajes business-initiated.
- Multi-agente: un mismo número puede ser atendido por muchos usuarios desde el sistema integrador.
- Multi-número: una misma WABA puede tener varios números.
- Quality rating y tiers de mensajería.
- Disponibilidad gestionada por Meta (sin SLA contractual para uso gratuito, pero alto uptime).

## Qué NO te da

- UI lista para agentes humanos: tenés que construirla o usar un CRM/SaaS encima.
- Free-form fuera de la ventana de 24h: necesitás plantillas aprobadas.
- Onboarding instantáneo: el cliente final debe pasar por verificación comercial de Meta.
- Garantías sobre cambios de pricing o políticas: Meta los actualiza con frecuencia.

## Vías para llegar a Cloud API

Una empresa puede acceder a Cloud API por distintos caminos según su perfil:

| Vía | Descripción | Cuándo conviene |
|---|---|---|
| **Acceso directo** | El integrador crea su propia app de Meta y se conecta | Equipo técnico, ahorro de fees, control total |
| **Vía BSP** | Un Business Solution Provider revende el acceso con servicios añadidos | Soporte premium, simplificar facturación, features extras |
| **Vía SaaS partner** (Respond.io, Kommo, HubSpot, etc.) | El SaaS oculta la API y expone UI lista | Cero código, agentes humanos, time-to-market |

Comparación detallada en [`03-formas-de-uso/comparativa.md`](../03-formas-de-uso/comparativa.md).

## Conceptos transversales que vas a leer en toda la wiki

| Concepto | Resumen |
|---|---|
| **Meta Business Portfolio** | Contenedor de los activos comerciales de la empresa en Meta |
| **WABA** | WhatsApp Business Account, agrupa números y plantillas |
| **Phone Number ID** | Identificador del número conectado, usado en endpoints |
| **System User** | Usuario no humano que genera tokens permanentes |
| **HSM (plantilla)** | Mensaje pre-aprobado para iniciar conversaciones |
| **Ventana 24h** | Período free-form tras último mensaje del usuario |
| **Quality Rating** | Semáforo de calidad del número (green/yellow/red) |
| **Tier de mensajería** | Cuántas conversaciones nuevas por día |
| **CTWA** | Click-to-WhatsApp Ads, anuncios que abren conversación |
| **BSP** | Business Solution Provider |

Definiciones canónicas en [`glossary.json`](../glossary.json).

## ¿Y las soluciones "no oficiales"?

Existen librerías y servicios (Baileys, Evolution API, whatsapp-web.js) que se conectan a WhatsApp **sin pasar por la Business Platform**, simulando un cliente de WhatsApp Web. **No son parte de Business Platform**, no usan plantillas reales, no tienen quality rating y violan los términos de servicio si se usan comercialmente sin consentimiento.

Las cubrimos en la wiki porque están muy extendidas y a veces son la única alternativa viable para prototipos. Pero conviene tener clarísimo que son una categoría distinta. Ver [`03-formas-de-uso/evolution-api.md`](../03-formas-de-uso/evolution-api.md).

## Próximos pasos

1. Leer [`arquitectura.md`](./arquitectura.md) para entender la jerarquía Portfolio → WABA → Número.
2. Leer [`01-onboarding-meta/business-portfolio.md`](../01-onboarding-meta/business-portfolio.md) para crear los activos.
3. Pasar por la verificación comercial: [`01-onboarding-meta/verificacion-comercial.md`](../01-onboarding-meta/verificacion-comercial.md).

## Referencias

- [WhatsApp Business Platform · Meta](https://business.whatsapp.com/products/business-platform) — Verificado 2026-05-20.
- [Cloud API Overview](https://developers.facebook.com/docs/whatsapp/cloud-api) — Verificado 2026-05-20.
- [On-Premise sunset notice](https://developers.facebook.com/docs/whatsapp/on-premises) — Verificado 2026-05-20.
