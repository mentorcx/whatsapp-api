---
title: "Kommo: conexión con WhatsApp"
category: integraciones-kommo
tags: [kommo, whatsapp, wa-lite, cloud-api, proveedor]
updated: 2026-05-20
source_official:
  - https://www.kommo.com/help/
related:
  - 03-formas-de-uso/kommo-crm.md
  - 07-integraciones/kommo/conceptos.md
audience: [integrador, comercial, dev]
---

# Kommo: conexión con WhatsApp

> **TL;DR:** Kommo conecta WhatsApp por tres vías: **WhatsApp Lite** (QR, no oficial, limitado), **Cloud API integrada** (oficial, HSM reales, multi-agente — recomendado para producción), y **proveedor externo** (Wazzup, etc.). Para cualquier cliente serio: Cloud API integrada.

## Contexto

La decisión de cómo conectar WhatsApp en Kommo define capacidades, costo y riesgo. Es lo primero que se resuelve al integrar.

## Las tres vías

| Vía | Oficial | HSM real | Multi-agente | Riesgo ban | Producción |
|---|---|---|---|---|---|
| WhatsApp Lite | No | No | Limitado | Medio | No |
| Cloud API integrada | Sí | Sí | Sí | Nulo | **Sí** |
| Proveedor externo | Depende | Depende | Sí | Variable | Según proveedor |

## WhatsApp Lite

Conecta el número escaneando un QR con la app de WhatsApp Business.

| Característica | Detalle |
|---|---|
| Conexión | QR, como WhatsApp Web |
| Naturaleza | No oficial (similar a Baileys por debajo) |
| HSM | No reales; mensajes business-initiated simulados |
| Dispositivos | Limitado (multi-device de WhatsApp) |
| Costo | Incluido en el plan de Kommo |
| Apto para | Emprendedores, pruebas, volumen bajo |

No usar para clientes que dependen del canal o hacen marketing.

## Cloud API integrada (recomendada)

Kommo es partner de Meta: el onboarding de la Cloud API se hace **desde dentro de Kommo**, con flujo embebido.

| Característica | Detalle |
|---|---|
| Conexión | Embedded Signup dentro de Kommo |
| Naturaleza | Oficial |
| HSM | Reales, creadas/sincronizadas con Meta |
| Multi-agente | Sí, asignación por pipeline |
| Quality / tiers | Visibles en Kommo |
| Costo | Plan Kommo + conversaciones que cobra Meta |

### Pasos de conexión

1. En Kommo → Settings → Integrations → buscar WhatsApp.
2. Elegir la opción de **WhatsApp Business (Cloud API)**.
3. Se abre el Embedded Signup de Meta.
4. El cliente: login de Facebook, elegir/crear Business Portfolio y WABA, conectar número.
5. Verificar el número (SMS/llamada).
6. Kommo queda vinculado a la WABA.
7. Las plantillas se gestionan desde Kommo o Meta Business Manager (sincronizan).

### Qué necesita el cliente

| Requisito | Detalle |
|---|---|
| Business Portfolio | Lo crea en el flujo o ya lo tiene |
| Número | No usado activamente en WA app |
| Verificación comercial | Recomendada para producción seria |

Ver [`01-onboarding-meta/`](../../01-onboarding-meta/).

## Vía proveedor externo

Algunos proveedores (Wazzup, Chat API, B2Chat, etc.) ofrecen integración Kommo + WhatsApp.

| Cuándo conviene | Detalle |
|---|---|
| El cliente ya tiene contrato con ese proveedor | Reusar |
| Features que la Cloud nativa de Kommo no expone | Caso puntual |
| Casos legacy | Migración pendiente |

Implica una capa más: más costo, más puntos de falla. Para proyectos nuevos, preferir Cloud API integrada.

## Comparación de costos

| Vía | Costo |
|---|---|
| WhatsApp Lite | Incluido en plan Kommo |
| Cloud API integrada | Plan Kommo + conversaciones (Meta) |
| Proveedor externo | Plan Kommo + suscripción del proveedor + (a veces) conversaciones |

## Plantillas en Kommo

Con Cloud API integrada:

- Las plantillas se crean desde Kommo o desde Meta Business Manager.
- Kommo las sincroniza con la WABA.
- El estado (aprobada/rechazada/pausada) se refleja en Kommo.
- Para enviar fuera de la ventana de 24h, se usa una plantilla aprobada.

Ver [`05-plantillas-hsm/`](../../05-plantillas-hsm/).

## Multi-número en Kommo

Kommo soporta varios números de WhatsApp en la misma cuenta. Cada uno aparece como un canal. Útil para separar ventas/soporte o multi-marca. Ver [`02-numeros-y-conexion/multi-numero.md`](../../02-numeros-y-conexion/multi-numero.md).

## Decisión rápida

| Cliente | Vía |
|---|---|
| Emprendedor, 1 número, volumen bajo, sin marketing | WhatsApp Lite (o directamente Cloud) |
| Empresa con agentes, marketing, soporte | Cloud API integrada |
| Ya usa un proveedor externo y funciona | Mantener proveedor, evaluar migración |
| Cualquier proyecto nuevo serio | Cloud API integrada |

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| Cliente con WhatsApp Lite quiere hacer campañas | Lite no tiene HSM reales | Migrar a Cloud API integrada |
| Número se cae seguido en Lite | Naturaleza no oficial | Cloud API |
| Plantillas no aparecen en Kommo | No sincronizadas | Forzar sync del canal |
| Doble costo inesperado | Proveedor externo + conversaciones Meta | Revisar el modelo de costos del proveedor |
| Número ya en uso al conectar | Activo en WA app | Liberar primero |

## Referencias

- [Kommo Help · WhatsApp](https://www.kommo.com/help/) — Verificado 2026-05-20.
- [`03-formas-de-uso/kommo-crm.md`](../../03-formas-de-uso/kommo-crm.md)
- [`07-integraciones/kommo/conceptos.md`](./conceptos.md)
