---
title: Despliegue con Docker Compose
category: despliegue
tags: [docker-compose, self-host, vps, evolution-api, n8n, caddy]
updated: 2026-05-20
source_official:
  - https://docs.docker.com/compose/
related:
  - 08-despliegue/alternativas.md
  - 08-despliegue/railway/plantilla-evolution-n8n-openai.md
audience: [integrador, dev]
---

# Despliegue con Docker Compose

> **TL;DR:** Equivalente self-hosted de las plantillas de Railway, para correr el stack en cualquier VPS (Hetzner, DigitalOcean, etc.). Un `docker-compose.yml` levanta Evolution + n8n + Postgres + Redis + Caddy (TLS automático). Más barato que un PaaS a escala, a cambio de administrar el servidor.

## Contexto

Las plantillas de Railway describen el mismo stack que estos compose files; la diferencia es quién administra la infraestructura. Docker Compose en un VPS da control total y costo bajo.

## Requisitos del VPS

| Recurso | Mínimo | Recomendado |
|---|---|---|
| CPU | 2 vCPU | 4 vCPU |
| RAM | 4 GB | 8 GB |
| Disco | 40 GB SSD | 80 GB SSD |
| OS | Debian / Ubuntu LTS | Idem |
| Docker + Docker Compose | Instalados | Idem |

Un Hetzner CPX21 / CPX31 alcanza para un stack típico.

## Stack completo: Evolution + n8n + OpenAI

`docker-compose.yml`:

```yaml
services:
  caddy:
    image: caddy:2-alpine
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - caddy_data:/data
      - caddy_config:/config
    depends_on:
      - evolution-api
      - n8n

  postgres:
    image: postgres:16-alpine
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    restart: unless-stopped
    command: redis-server --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis_data:/data

  evolution-api:
    image: atendai/evolution-api:v2.1.1
    restart: unless-stopped
    environment:
      SERVER_URL: https://evo.${DOMAIN}
      AUTHENTICATION_API_KEY: ${EVOLUTION_API_KEY}
      DATABASE_PROVIDER: postgresql
      DATABASE_CONNECTION_URI: postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres:5432/${POSTGRES_DB}?schema=evolution
      CACHE_REDIS_ENABLED: "true"
      CACHE_REDIS_URI: redis://:${REDIS_PASSWORD}@redis:6379/1
      CACHE_LOCAL_ENABLED: "false"
      WEBHOOK_GLOBAL_URL: http://n8n:5678/webhook/evolution
      WEBHOOK_GLOBAL_ENABLED: "true"
      WEBHOOK_EVENTS_MESSAGES_UPSERT: "true"
      WEBHOOK_EVENTS_CONNECTION_UPDATE: "true"
      LANGUAGE: es
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started

  n8n:
    image: n8nio/n8n:1.70.0
    restart: unless-stopped
    environment:
      N8N_HOST: n8n.${DOMAIN}
      N8N_PROTOCOL: https
      N8N_PORT: 5678
      WEBHOOK_URL: https://n8n.${DOMAIN}/
      DB_TYPE: postgresdb
      DB_POSTGRESDB_HOST: postgres
      DB_POSTGRESDB_DATABASE: ${POSTGRES_DB}
      DB_POSTGRESDB_USER: ${POSTGRES_USER}
      DB_POSTGRESDB_PASSWORD: ${POSTGRES_PASSWORD}
      EXECUTIONS_MODE: queue
      QUEUE_BULL_REDIS_HOST: redis
      QUEUE_BULL_REDIS_PASSWORD: ${REDIS_PASSWORD}
      N8N_ENCRYPTION_KEY: ${N8N_ENCRYPTION_KEY}
      N8N_BASIC_AUTH_ACTIVE: "true"
      N8N_BASIC_AUTH_USER: ${N8N_USER}
      N8N_BASIC_AUTH_PASSWORD: ${N8N_PASSWORD}
      GENERIC_TIMEZONE: ${TIMEZONE}
    volumes:
      - n8n_data:/home/node/.n8n
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started

volumes:
  postgres_data:
  redis_data:
  n8n_data:
  caddy_data:
  caddy_config:
```

## Caddyfile

```caddyfile
evo.{$DOMAIN} {
    reverse_proxy evolution-api:8080
}

n8n.{$DOMAIN} {
    reverse_proxy n8n:5678
}
```

Caddy emite y renueva certificados TLS automáticamente vía Let's Encrypt.

## Archivo `.env`

```bash
DOMAIN=tudominio.com
TIMEZONE=America/Argentina/Buenos_Aires

POSTGRES_USER=wa_admin
POSTGRES_PASSWORD=         # openssl rand -hex 24
POSTGRES_DB=wa_platform

REDIS_PASSWORD=            # openssl rand -hex 24

EVOLUTION_API_KEY=         # openssl rand -hex 32

N8N_ENCRYPTION_KEY=        # openssl rand -hex 16
N8N_USER=admin
N8N_PASSWORD=              # contraseña fuerte
```

**Nunca** versionar el `.env` real. Versionar solo un `.env.example`.

## Puesta en marcha

```bash
# 1. Apuntar DNS: A records de evo.tudominio.com y n8n.tudominio.com -> IP del VPS

# 2. Generar secretos y completar .env
openssl rand -hex 24   # passwords
openssl rand -hex 32   # evolution api key
openssl rand -hex 16   # n8n encryption key

# 3. Levantar el stack
docker compose up -d

# 4. Verificar
docker compose ps
docker compose logs -f evolution-api
```

## Variante: solo n8n (Cloud API oficial)

Si usás Cloud API directa (sin Evolution), quitar el servicio `evolution-api` y dejar `n8n` + `postgres` + `redis` + `caddy`. El Caddyfile mantiene solo el bloque de `n8n`.

## Backups

### Postgres

```bash
# Cron diario
docker compose exec -T postgres pg_dump -U wa_admin wa_platform | gzip > backup-$(date +%F).sql.gz
```

Subir los backups a almacenamiento externo (S3, Backblaze, etc.).

### Restore

```bash
gunzip -c backup-2026-05-20.sql.gz | docker compose exec -T postgres psql -U wa_admin wa_platform
```

### Volúmenes

`n8n_data` y `caddy_data` también conviene respaldarlos (workflows binarios, certificados).

## Actualización de versiones

```bash
# Editar las versiones pineadas en docker-compose.yml, luego:
docker compose pull
docker compose up -d
```

Probar en un entorno de staging antes de actualizar producción. Nunca usar `:latest`.

## Hardening del VPS

| Acción | Detalle |
|---|---|
| Firewall (ufw) | Permitir solo 22, 80, 443 |
| SSH con clave, no password | Deshabilitar `PasswordAuthentication` |
| Fail2ban | Contra fuerza bruta SSH |
| Updates automáticos de seguridad | `unattended-upgrades` |
| Usuario no-root para Docker | Grupo `docker` |
| Secretos fuera del repo | `.env` con permisos `600` |
| Backups verificados | Probar el restore periódicamente |

## Monitoreo

| Qué | Herramienta |
|---|---|
| Estado de containers | `docker compose ps`, healthchecks |
| Recursos del VPS | `docker stats`, `htop` |
| Uptime de los endpoints | Uptime Kuma, Healthchecks.io |
| Logs | `docker compose logs`, o stack de logs (Loki) |
| Alertas | Webhook a Slack ante caídas |

## Compose vs Railway

| Aspecto | Docker Compose (VPS) | Railway |
|---|---|---|
| Costo | Más bajo a escala | Más alto con el uso |
| Ops | A tu cargo | Mínimas |
| Backups | Vos los configurás | Plugin los hace |
| TLS | Caddy lo resuelve | Automático |
| Setup inicial | Más trabajo | Minutos |
| Control | Total | Limitado |

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| n8n no conecta a Postgres | `depends_on` sin healthcheck | Usar `condition: service_healthy` |
| Caddy no emite TLS | DNS no propagado o puerto 80 cerrado | Esperar DNS, abrir firewall |
| Evolution pide QR tras `up -d` | Volumen no persistente o schema mal | Verificar persistencia en Postgres |
| Datos perdidos tras `docker compose down -v` | `-v` borra volúmenes | Nunca usar `-v` en producción |
| `N8N_ENCRYPTION_KEY` cambió | Credenciales ilegibles | Fijar la key, restaurar la original |
| Webhook de Evolution no llega a n8n | URL con `localhost` en vez del nombre del servicio | Usar `http://n8n:5678/...` |

## Referencias

- [Docker Compose](https://docs.docker.com/compose/) — Verificado 2026-05-20.
- [Caddy](https://caddyserver.com/docs/) — Verificado 2026-05-20.
- [`08-despliegue/railway/plantilla-evolution-n8n-openai.md`](../railway/plantilla-evolution-n8n-openai.md)
- [`08-despliegue/alternativas.md`](../alternativas.md)
