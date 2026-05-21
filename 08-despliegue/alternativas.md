---
title: Alternativas de despliegue
category: despliegue
tags: [despliegue, fly-io, render, hetzner, vps, docker-compose]
updated: 2026-05-20
related:
  - 08-despliegue/railway/conceptos.md
  - 08-despliegue/docker-compose/README.md
audience: [integrador, dev]
---

# Alternativas de despliegue

> **TL;DR:** Railway es el default de esta wiki por simplicidad, pero no el único. **Fly.io** para despliegue global y control fino; **Render** muy similar a Railway; **Hetzner / VPS** mucho más barato a escala pero requiere ops; **Coolify** para self-hosting con UX tipo PaaS. La elección depende de volumen, presupuesto y capacidad de operación.

## Contexto

Railway está bien para arrancar y para la mayoría de los clientes. Pero a cierta escala el costo sube, o el cliente exige infra propia, o necesitás presencia en una región específica. Este documento compara las opciones reales.

## Comparativa

| Plataforma | Modelo | Costo relativo | Ops requeridas | Cuándo |
|---|---|---|---|---|
| **Railway** | PaaS | Medio | Mínimas | Default, stacks chicos-medios |
| **Render** | PaaS | Medio | Mínimas | Alternativa directa a Railway |
| **Fly.io** | PaaS global | Medio | Bajas-medias | Despliegue multi-región, control fino |
| **Hetzner + Docker** | VPS | Muy bajo | Altas | Escala, presupuesto ajustado |
| **Hetzner + Coolify** | VPS + PaaS UI | Bajo | Medias | Self-host con UX de PaaS |
| **DigitalOcean / Vultr / Linode** | VPS | Bajo-medio | Altas | VPS clásico |
| **AWS / GCP / Azure** | Cloud | Variable, puede ser alto | Altas | Empresas con equipo cloud |

## Railway

Cubierto en [`railway/conceptos.md`](./railway/conceptos.md).

| Pro | Contra |
|---|---|
| Deploy en minutos | Costo sube con el uso |
| Plantillas listas | Menos control que un VPS |
| Red privada simple | Lock-in moderado |
| Buena DX | — |

## Render

Muy parecido a Railway.

| Pro | Contra |
|---|---|
| PaaS simple, buena DX | Similar a Railway en costo |
| Free tier para servicios chicos | Free tier con cold starts |
| Postgres y Redis gestionados | — |

Migrar Railway ↔ Render es directo: ambos toman imágenes Docker y variables de entorno.

## Fly.io

PaaS con foco en despliegue **global** (apps cerca del usuario).

| Pro | Contra |
|---|---|
| Multi-región nativo | Curva un poco mayor |
| Control fino (máquinas, escalado) | Configuración por `fly.toml` |
| Buen pricing a escala | Postgres gestionado más limitado |
| Ideal si la latencia importa | — |

Para WhatsApp la latencia rara vez es crítica (los webhooks toleran), así que Fly.io brilla más por el control que por la geo-distribución.

## Hetzner + Docker Compose

La opción más barata. Un VPS de Hetzner (CX/CPX) corre todo el stack con `docker compose`.

| Pro | Contra |
|---|---|
| Costo muy bajo (un VPS chico alcanza) | Vos administrás el servidor |
| Control total | Updates, backups, seguridad a tu cargo |
| Sin lock-in | Sin red privada mágica |
| Recursos generosos por el precio | Requiere saber Linux / Docker |

Recomendado cuando: el integrador tiene capacidad de ops y maneja varios clientes (el costo fijo del VPS se amortiza).

Ver [`docker-compose/`](./docker-compose/) para los compose files.

## Hetzner + Coolify

[Coolify](https://coolify.io/) es un PaaS self-hosted open source: instalás Coolify en un VPS y obtenés una UX tipo Railway/Render pero en tu servidor.

| Pro | Contra |
|---|---|
| UX de PaaS sobre infra propia | Coolify mismo hay que mantenerlo |
| Costo de VPS, no de PaaS | Una pieza más que puede fallar |
| Deploy con UI, dominios, TLS automático | — |
| Bueno para multi-cliente | — |

Punto dulce para integradores que quieren bajar costos sin renunciar a la comodidad.

## VPS clásico (DigitalOcean, Vultr, Linode)

Similar a Hetzner pero generalmente algo más caro. Mismo modelo: VPS + Docker.

| Cuándo | El cliente exige un proveedor específico o ya tiene cuenta |

## Cloud hyperscalers (AWS / GCP / Azure)

| Pro | Contra |
|---|---|
| Todo lo imaginable | Complejidad alta |
| Escala infinita | Costo difícil de predecir |
| Requerido por algunas empresas | Requiere equipo cloud |

Para la mayoría de proyectos WhatsApp de integradores, es sobre-ingeniería. Tiene sentido si el cliente **ya** está en ese cloud y exige consistencia.

## Matriz de decisión

| Situación | Recomendación |
|---|---|
| Primer proyecto, querés rapidez | Railway |
| Cliente chico, presupuesto mínimo | Railway (hobby) o Render free |
| Varios clientes, querés bajar costos | Hetzner + Coolify o Docker Compose |
| Necesitás baja latencia multi-región | Fly.io |
| El cliente exige su propia infra | VPS del cliente + Docker Compose |
| Empresa grande ya en AWS/GCP | El cloud que ya usan |
| Evolution API (no oficial) | Cualquiera self-host; Railway o VPS |

## Qué NO cambia según la plataforma

Independientemente de dónde despliegues, necesitás:

| Componente | Siempre |
|---|---|
| HTTPS público para el webhook | Sí |
| Postgres persistente | Sí |
| Redis (dedup, cache, queue) | Recomendado |
| Backups | Sí |
| Healthchecks | Sí |
| Variables de entorno seguras | Sí |

Los `docker-compose` de [`docker-compose/`](./docker-compose/) corren igual en Hetzner, DigitalOcean, o cualquier VPS.

## Migrar entre plataformas

Como todo corre en containers Docker:

1. Exportar las variables de entorno.
2. Hacer `pg_dump` del Postgres.
3. Levantar el stack en la nueva plataforma.
4. Restaurar el dump.
5. Reapuntar DNS / webhook de Meta.
6. Verificar.

El lock-in real es bajo si mantenés todo containerizado y la data en Postgres.

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| Migrar a VPS y olvidar backups | El PaaS los hacía automático | Configurar `pg_dump` por cron |
| VPS sin HTTPS | No configuraste reverse proxy + TLS | Caddy o Traefik con Let's Encrypt |
| Costos de cloud disparados | Recursos sobredimensionados | Empezar chico, escalar con datos |
| Coolify cae y se lleva todo | Coolify no monitoreado | Monitorear Coolify mismo |
| Latencia alta del webhook | Servidor en región lejana a Meta | Elegir región cercana (US/EU) |

## Referencias

- [Railway](https://docs.railway.com/) — Verificado 2026-05-20.
- [Fly.io](https://fly.io/docs/) — Verificado 2026-05-20.
- [Render](https://render.com/docs) — Verificado 2026-05-20.
- [Coolify](https://coolify.io/docs/) — Verificado 2026-05-20.
- [Hetzner Cloud](https://docs.hetzner.com/cloud/) — Verificado 2026-05-20.
- [`08-despliegue/railway/conceptos.md`](./railway/conceptos.md)
- [`08-despliegue/docker-compose/`](./docker-compose/)
