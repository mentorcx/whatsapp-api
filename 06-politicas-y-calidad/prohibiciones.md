---
title: Industrias restringidas y contenidos prohibidos
category: politicas-y-calidad
tags: [prohibiciones, commerce-policy, business-policy, restricciones]
updated: 2026-05-20
source_official:
  - https://www.whatsapp.com/legal/commerce-policy
  - https://www.whatsapp.com/legal/business-policy
related:
  - 06-politicas-y-calidad/opt-in.md
  - 05-plantillas-hsm/rechazos.md
audience: [integrador, comercial, legal]
---

# Industrias restringidas y contenidos prohibidos

> **TL;DR:** WhatsApp tiene dos políticas que limitan qué se puede vender y comunicar: la **Commerce Policy** (qué productos) y la **Business Policy** (cómo comunicar). Productos totalmente prohibidos: armas, drogas, alcohol/tabaco (venta directa), suplementos peligrosos, animales vivos, productos médicos riesgosos, contenido adulto, servicios financieros engañosos. Antes de onboardear un cliente, verificar que su rubro es viable.

## Contexto

Onboardear a un cliente cuyo rubro está prohibido es perder tiempo y arriesgar el número. Esta verificación va **antes** de cualquier desarrollo. Las dos políticas relevantes:

| Política | Qué regula |
|---|---|
| Commerce Policy | Qué productos/servicios se pueden ofrecer y vender |
| Business Policy | Cómo se comunica, opt-in, spam, contenido de mensajes |

## Productos totalmente prohibidos (Commerce Policy)

No se pueden ofrecer ni vender vía WhatsApp Business:

| Categoría | Detalle |
|---|---|
| Drogas | Ilegales y recreativas; parafernalia |
| Tabaco y productos relacionados | Cigarrillos, vapeadores, accesorios |
| Alcohol | Venta directa (varía por región, en general restringido) |
| Armas, municiones, explosivos | Incluye réplicas y partes |
| Animales vivos | Venta de animales |
| Productos de salud peligrosos | Suplementos no aprobados, productos con claims médicos falsos |
| Productos médicos / farmacéuticos | Medicamentos de venta bajo receta |
| Partes del cuerpo / fluidos humanos | — |
| Contenido para adultos | Pornografía, servicios sexuales |
| Productos o servicios sexuales | — |
| Productos robados | — |
| Documentos / moneda / instrumentos financieros | Falsificados o reales para reventa |
| Productos que infrinjan propiedad intelectual | Falsificaciones, réplicas de marca |
| Apuestas y juego con dinero real | Muy restringido; varía por jurisdicción |
| Productos peligrosos / sustancias controladas | — |

## Sectores restringidos (operan con condiciones)

Pueden operar pero con limitaciones, revisión más estricta o solo en ciertas regiones:

| Sector | Condición |
|---|---|
| Servicios financieros | Permitidos si son legítimos y transparentes; nada de préstamos predatorios o esquemas |
| Criptomonedas | Restringido / requiere autorización según región |
| Salud | Clínicas, turnos, info general sí; venta de medicamentos no |
| Farmacias | Pueden comunicar, no vender medicamentos de receta por el canal |
| Juego / apuestas | Solo con licencia y en jurisdicciones donde es legal, con autorización |
| Suplementos / nutrición | Sin claims terapéuticos no probados |
| Citas / dating | Permitido el servicio, no contenido sexual |
| MLM / venta multinivel | Bajo escrutinio; nada de esquemas piramidales |

## Contenidos prohibidos en los mensajes (Business Policy)

Independientemente del rubro, no se puede:

| Prohibición | Detalle |
|---|---|
| Spam | Mensajes masivos sin opt-in |
| Engaño / phishing | Hacerse pasar por otra entidad |
| Contenido ilegal | Cualquier cosa ilegal en la jurisdicción |
| Acoso / amenazas | — |
| Discurso de odio | — |
| Desinformación dañina | — |
| Solicitar datos sensibles indebidamente | Contraseñas, datos de tarjeta completos por chat |
| Contenido violento o gráfico | — |
| Suplantación de identidad | — |

## Verificación pre-onboarding

Checklist antes de tomar un cliente:

| Pregunta | Si la respuesta es problemática |
|---|---|
| ¿Qué vende exactamente? | Cruzar con la lista de prohibidos |
| ¿El producto requiere receta / licencia? | Sector restringido, evaluar |
| ¿Hay claims de salud / resultados? | Riesgo de rechazo de plantillas |
| ¿Tiene las habilitaciones legales del rubro? | Sin ellas, no avanzar |
| ¿De dónde sale la base de contactos? | Si es comprada, problema de opt-in |
| ¿En qué países opera? | Algunas restricciones son regionales |

Si el rubro está prohibido: **no onboardear**. Si está restringido: verificar licencias y consultar la política específica antes de avanzar.

## Casos límite frecuentes en LATAM

| Caso | Lectura |
|---|---|
| Farmacia que quiere dar turnos y recordatorios | Viable: comunicación, no venta de medicamentos por chat |
| Clínica estética con tratamientos | Viable con cuidado en los claims |
| Casa de cambio / fintech | Viable si es legítima y regulada |
| Venta de suplementos deportivos | Viable sin claims terapéuticos |
| Inmobiliaria | Viable |
| Casino / apuestas online | Solo con licencia y autorización; muchas trabas |
| Venta de vinos / bodega | Restringido (alcohol); evaluar región |
| Tienda de vapeadores | Prohibido |
| Préstamos personales | Viable solo si es transparente, sin letra chica engañosa |

## Consecuencias de violar las políticas

| Severidad | Consecuencia |
|---|---|
| Leve / primera vez | Advertencia, plantilla rechazada |
| Media | Quality drop, baja de tier, plantillas deshabilitadas |
| Grave | Suspensión del número |
| Muy grave / reincidente | Baneo de la WABA o del Business Portfolio |

Un baneo de Portfolio afecta **todos** los activos: difícil de revertir.

## Diferencias regionales

Las políticas tienen variaciones por país (especialmente alcohol, juego, salud, finanzas). Ante un rubro sensible:

1. Leer la Commerce Policy y Business Policy vigentes.
2. Verificar la sección regional si existe.
3. Ante la duda, consultar a Meta Business Support o a un BSP con experiencia local.

## Plantillas y contenido prohibido

Las políticas de producto se reflejan en la aprobación de plantillas. Una plantilla de un sector prohibido será rechazada con `POLICY_VIOLATION`. Ver [`05-plantillas-hsm/rechazos.md`](../05-plantillas-hsm/rechazos.md).

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| Onboardeás un cliente y todas las plantillas se rechazan | Rubro restringido/prohibido | Verificar antes de desarrollar |
| Número suspendido a las semanas | Producto prohibido detectado | No había que tomar ese cliente |
| Cliente de alcohol bloqueado | Venta directa de alcohol | Evaluar región; a veces solo se puede comunicar, no vender |
| Cliente farmacéutico con plantillas rechazadas | Claims o venta de recetados | Reformular: comunicación, no venta |
| Base comprada → quality drop inmediato | Sin opt-in (Business Policy) | Ver [`opt-in.md`](./opt-in.md) |

## Recomendación para integradores

Incluir en el contrato / onboarding del cliente una **declaración** de que su rubro y prácticas cumplen las políticas de WhatsApp, y que la base de contactos tiene opt-in. Protege al integrador si el cliente miente y el número cae.

## Referencias

- [WhatsApp Commerce Policy](https://www.whatsapp.com/legal/commerce-policy) — Verificado 2026-05-20.
- [WhatsApp Business Policy](https://www.whatsapp.com/legal/business-policy) — Verificado 2026-05-20.
- [`06-politicas-y-calidad/opt-in.md`](./opt-in.md)
- [`05-plantillas-hsm/rechazos.md`](../05-plantillas-hsm/rechazos.md)
