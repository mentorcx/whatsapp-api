---
title: Railway · Conceptos
category: despliegue
tags: [railway, conceptos, servicios, variables, redes, volumenes]
updated: 2026-05-20
source_official:
  - https://docs.railway.com/
related:
  - 08-despliegue/railway/plantilla-evolution-api.md
  - 08-despliegue/railway/plantilla-evolution-n8n-openai.md
audience: [integrador, dev]
---

# Railway · Conceptos

> **TL;DR:** Railway es un PaaS donde un **proyecto** agrupa **servicios** (containers, bases de datos). Los servicios se comunican por **red privada interna** (`servicio.railway.internal`) sin exponer puertos. Las **variables de entorno** soportan referencias entre servicios (`${{Postgres.DATABASE_URL}}`). Cobra por uso de CPU/RAM/red. Ideal para stacks WhatsApp por sus plantillas y simplicidad.

## Contexto

Railway aparece tanto en esta wiki porque permite levantar stacks completos (Evolution, n8n, DB) en minutos, con plantillas listas y sin gestionar servidores. Este documento explica los conceptos que necesitás antes de seguir las plantillas concretas.

## Jerarquía

```
Account
└── Project
    ├── Service: evolution-api   (container)
    ├── Service: n8n             (container)
    ├── Service: Postgres        (database plugin)
    ├── Service: Redis           (database plugin)
    └── Service: caddy           (container)
```

| Concepto | Detalle |
|---|---|
| **Project** | Contenedor lógico. Un stack = un project. |
| **Service** | Una unidad desplegable: container Docker, repo de GitHub, o database plugin. |
| **Environment** | Variante del project (`production`, `staging`). Cada uno con sus variables. |
| **Deployment** | Una versión desplegada de un servicio. |
| **Plugin / Database** | Postgres, Redis, MySQL, MongoDB gestionados por Railway. |

## Cómo se crea un servicio

| Origen | Cuándo |
|---|---|
| Docker Image | Imágenes públicas (`atendai/evolution-api`, `n8nio/n8n`) |
| GitHub Repo | Tu propio código; Railway buildea con Nixpacks |
| Database plugin | Postgres, Redis, etc. con un click |
| Template | Stack pre-armado (uno o varios servicios juntos) |

## Templates

Railway tiene un marketplace de **templates**: stacks pre-configurados que se despliegan con un botón.

| Ventaja | Detalle |
|---|---|
| Deploy en 1 click | Variables y servicios pre-armados |
| Botón "Deploy on Railway" | Embebible en docs / README |
| Punto de partida | Después se customiza |

Para los stacks de esta wiki, las plantillas documentadas (`plantilla-evolution-api.md`, etc.) describen qué servicios y variables crear, ya sea desde un template del marketplace o a mano.

## Red privada interna

Los servicios de un mismo project se comunican por DNS interno **sin exponerse a Internet**:

```
{nombre-del-servicio}.railway.internal
```

Ejemplos:

- `n8n` accede a Postgres vía `postgres.railway.internal:5432`.
- Evolution manda webhooks a `n8n.railway.internal:5678`.

| Ventaja | Detalle |
|---|---|
| Seguridad | El tráfico interno no sale a Internet |
| Sin latencia de red pública | Más rápido |
| No hace falta exponer puertos | Solo lo que el usuario final accede va público |

Solo lo que necesita acceso externo (el webhook de Meta, la UI de n8n) se expone con un **dominio público**.

## Dominios

| Tipo | Detalle |
|---|---|
| Dominio generado | `nombre-production-xxxx.up.railway.app` gratis |
| Dominio propio | CNAME desde tu DNS al servicio; Railway emite TLS |

Para WhatsApp: el endpoint de webhook necesita un dominio público con HTTPS. El generado por Railway sirve; un dominio propio es más profesional.

## Variables de entorno

### Variables simples

Se setean por servicio en el panel. Marcar como secretas las sensibles.

### Referencias entre servicios

Railway permite referenciar variables de otro servicio:

```
DATABASE_URL = ${{Postgres.DATABASE_URL}}
REDIS_URL    = ${{Redis.REDIS_URL}}
PGHOST       = ${{Postgres.PGHOST}}
```

Cuando Railway crea un plugin de Postgres, expone variables como `DATABASE_URL`, `PGHOST`, `PGUSER`, `PGPASSWORD`, `PGDATABASE`. Otros servicios las referencian sin copiar valores.

### Variables compartidas

A nivel project, variables que todos los servicios heredan.

## Volúmenes

Los containers son **efímeros**: lo que escriben en disco se pierde al redeploy salvo que uses un **volumen**.

| Necesita volumen | Servicio |
|---|---|
| Sí | n8n (`/home/node/.n8n`), Caddy (certificados) |
| Gestionado por el plugin | Postgres, Redis |
| No, si el estado está en DB | Evolution API (con persistencia en Postgres) |

Sin volumen + sin persistencia externa = pérdida de datos en cada deploy.

## Healthchecks y restart

| Configuración | Detalle |
|---|---|
| Healthcheck path | Railway pingea esa URL para saber si el servicio está sano |
| Restart policy | `on-failure` con `maxRetries` |
| Start command | Override del comando del container si hace falta |

Para los stacks WhatsApp:

| Servicio | Healthcheck |
|---|---|
| Evolution | `GET /` → 200 |
| n8n | `GET /healthz` → 200 |

## Costos

Railway cobra por **uso real**:

| Recurso | Se cobra por |
|---|---|
| CPU | vCPU-hora consumida |
| RAM | GB-hora consumida |
| Red | Egress (GB salientes) |
| Volumen | GB-mes de almacenamiento |

| Plan | Detalle |
|---|---|
| Hobby | Crédito mensual incluido, suficiente para stacks chicos |
| Pro | Mayor capacidad, soporte |

Estimación de un stack WhatsApp típico: $15-40/mes. Las plantillas concretas dan números desglosados.

## Buenas prácticas

| Práctica | Razón |
|---|---|
| Un project por stack/cliente | Aislamiento |
| Environments `staging` y `production` separados | Probar sin romper prod |
| Pinear versiones de imágenes (`:v2.0.1`, no `:latest`) | Evitar breaking changes |
| Marcar secretos como secretos | No exponerlos en logs |
| Backups de Postgres activados | Recuperación |
| Healthchecks en todos los servicios | Detectar caídas |
| Usar red interna, exponer lo mínimo | Seguridad |

## Railway vs otras opciones

| Plataforma | Cuándo |
|---|---|
| Railway | Stacks multi-servicio, plantillas, simplicidad |
| Fly.io | Despliegue global, control fino |
| Render | Similar a Railway |
| Hetzner / VPS | Control total, más barato a escala, más ops |

Comparación detallada en [`08-despliegue/alternativas.md`](../alternativas.md).

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| Servicio pierde datos al redeploy | Sin volumen ni persistencia externa | Agregar volumen o usar DB |
| Servicios no se comunican | Usás dominio público en vez del interno | Usar `servicio.railway.internal` |
| Variable `${{X.Y}}` no resuelve | Nombre del servicio mal escrito | Verificar nombre exacto |
| Costo más alto de lo esperado | Servicio con mucha RAM ociosa o egress alto | Ajustar recursos, revisar tráfico |
| TLS no se emite | DNS no propagado | Esperar propagación del CNAME |
| Container reinicia en loop | Falla al arrancar (DB no lista, var faltante) | Revisar logs del deployment |

## Referencias

- [Railway Docs](https://docs.railway.com/) — Verificado 2026-05-20.
- [Private Networking](https://docs.railway.com/reference/private-networking) — Verificado 2026-05-20.
- [Variables](https://docs.railway.com/guides/variables) — Verificado 2026-05-20.
- [`08-despliegue/railway/plantilla-evolution-api.md`](./plantilla-evolution-api.md)
- [`08-despliegue/railway/plantilla-evolution-n8n-openai.md`](./plantilla-evolution-n8n-openai.md)
