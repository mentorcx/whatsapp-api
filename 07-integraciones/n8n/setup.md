---
title: "n8n: setup"
category: integraciones-n8n
tags: [n8n, instalacion, queue-mode, credentials]
updated: 2026-05-20
source_official:
  - https://docs.n8n.io/hosting/
related:
  - 08-despliegue/railway/plantilla-n8n.md
  - 07-integraciones/n8n/patrones.md
audience: [integrador, dev]
---

# n8n: setup

> **TL;DR:** Para WhatsApp en producción, n8n debe correr en **queue mode** (main + workers + Redis), persistir en **Postgres**, tener una **`N8N_ENCRYPTION_KEY`** fija que nunca se rota, y **Basic Auth** activado en la UI. Las credenciales (WA, OpenAI, DB) se cargan dentro de n8n y se referencian desde los workflows.

## Requisitos mínimos para producción

| Componente | Por qué |
|---|---|
| n8n en queue mode | Para responder rápido a los webhooks de Meta y procesar async |
| Worker separado | Throughput; el main no se bloquea |
| Postgres | Persistir workflows, credentials, executions |
| Redis | Cola de ejecuciones |
| HTTPS público | Webhook de Meta requiere HTTPS |
| Basic Auth en la UI | Que cualquiera no pueda entrar |
| Encryption key fija | Sin ella se pierden las credentials al redeploy |

## Variables de entorno clave

| Variable | Para qué |
|---|---|
| `EXECUTIONS_MODE=queue` | Activar queue mode |
| `DB_TYPE=postgresdb` | Usar Postgres |
| `DB_POSTGRESDB_*` | Conexión a la base |
| `QUEUE_BULL_REDIS_*` | Conexión a Redis |
| `N8N_ENCRYPTION_KEY` | Encripta credentials. **Nunca rotar.** |
| `N8N_HOST`, `WEBHOOK_URL` | Dominio público |
| `N8N_BASIC_AUTH_ACTIVE=true` | Activar auth UI |
| `N8N_BASIC_AUTH_USER` / `_PASSWORD` | Credenciales UI |
| `GENERIC_TIMEZONE` | Zona horaria |

Detalle por servicio en [`08-despliegue/railway/plantilla-n8n.md`](../../08-despliegue/railway/plantilla-n8n.md).

Generar la encryption key:

```bash
openssl rand -hex 16
```

## Worker

El worker se levanta con la misma imagen pero comando `n8n worker`. Comparte la misma DB, Redis y `N8N_ENCRYPTION_KEY` que el main. Sin el worker, n8n procesa todo en el main: aceptable para volumen muy bajo, no apto para producción WhatsApp.

## Credentials

Dentro de n8n: **Credentials** (UI) → crear una por servicio. Quedan encriptadas en Postgres con la `N8N_ENCRYPTION_KEY`.

| Credential típica de WhatsApp | Detalle |
|---|---|
| WhatsApp Business Cloud API | `Access Token` (System User), `Phone Number ID` |
| OpenAI | `API Key` |
| Postgres | Connection params |
| Redis | Connection params |
| Kommo | `Subdomain` + `Long-lived token` |

Los workflows referencian las credentials por nombre, no por valor literal. Esto permite cambiar credenciales sin tocar workflows.

## Importar / exportar workflows

| Operación | Cómo |
|---|---|
| Exportar | UI → Workflow → menú → Download (JSON) |
| Importar | UI → Workflows → Import from File |
| Versionar | Guardar los JSON en Git (sin credentials, las refieren por nombre) |

Para multi-cliente: los workflows estables van en Git; cada tenant en su instancia de n8n importa la versión vigente.

## Endpoints útiles

| Endpoint | Para qué |
|---|---|
| `/healthz` | Healthcheck del main |
| `/webhook/...` | Webhooks productivos |
| `/webhook-test/...` | Webhooks en modo test (al editar un workflow) |
| `/` | UI (con Basic Auth) |

## Modo test vs production

| Modo | Path |
|---|---|
| Production | `/webhook/{{path}}` — activo cuando el workflow está activado |
| Test | `/webhook-test/{{path}}` — activo mientras editás el workflow con "Listen for Event" |

Para Meta: **siempre el path de production**. El de test solo dispara una vez.

## Logs y monitoreo

| Qué | Cómo |
|---|---|
| Logs de ejecuciones | UI → Executions |
| Logs del proceso | `docker logs` o equivalente |
| Webhook activity | Loggear input/output en Code nodes |
| Errores | Workflows con "Error Trigger" para capturar fallos |

## Backups

Lo crítico:

| Qué | Cómo |
|---|---|
| Workflows | `pg_dump` de la DB de n8n |
| Credentials | Backup de la DB + guardar la `N8N_ENCRYPTION_KEY` aparte |
| `N8N_ENCRYPTION_KEY` | En gestor de secretos del cliente y del integrador |

Sin la encryption key, un backup de credentials es ilegible.

## Multi-tenant

Para varios clientes, dos opciones:

| Opción | Pro | Contra |
|---|---|---|
| Una instancia de n8n por cliente | Aislamiento total | Más infra |
| Una instancia compartida con tagging | Menos infra | Hay que cuidar accidentes |

Recomendado: una instancia por cliente grande; compartida para clientes chicos con buena disciplina de tagging y permisos.

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| Credentials se pierden tras redeploy | `N8N_ENCRYPTION_KEY` cambió o no fija | Fijarla y guardarla |
| Webhooks no se ejecutan | Worker caído, Redis desconectado | Verificar worker y Redis |
| Webhook tarda y Meta reintenta | No está en queue mode | Activar |
| UI no responde | DB con muchas executions | Limpiar / aumentar recursos |
| `too many connections` | Pool de Postgres chico | Ajustar o usar pgbouncer |

## Referencias

- [n8n Hosting](https://docs.n8n.io/hosting/) — Verificado 2026-05-20.
- [`08-despliegue/railway/plantilla-n8n.md`](../../08-despliegue/railway/plantilla-n8n.md)
- [`07-integraciones/n8n/patrones.md`](./patrones.md)
- [`07-integraciones/n8n/webhook-entrante.md`](./webhook-entrante.md)
