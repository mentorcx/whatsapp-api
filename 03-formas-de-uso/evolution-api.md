---
title: Evolution API
category: formas-de-uso
tags: [evolution-api, baileys, no-oficial, self-hosted]
updated: 2026-05-20
source_official:
  - https://doc.evolution-api.com/
  - https://github.com/EvolutionAPI/evolution-api
related:
  - 03-formas-de-uso/comparativa.md
  - 03-formas-de-uso/cloud-api-oficial.md
  - 07-integraciones/evolution-api/README.md
  - 08-despliegue/railway/plantilla-evolution-n8n-openai.md
audience: [integrador, dev]
---

# Evolution API

> **TL;DR:** API REST self-hosted basada en Baileys que se conecta a WhatsApp como si fuera un cliente WhatsApp Web. **No es oficial**, no tiene HSM real ni quality rating, y conlleva riesgo concreto de ban. Útil para prototipos, automatizaciones internas o números no críticos. Para producción seria con marketing o industrias reguladas, usar Cloud API.

## Contexto

Evolution API es un proyecto open source que envuelve la librería [Baileys](https://github.com/WhiskeySockets/Baileys) en una API REST con multi-tenancy, webhooks, persistencia y panel de administración. Permite operar WhatsApp sin pasar por Meta: el número se conecta escaneando un QR o ingresando un pairing code, igual que WhatsApp Web.

**Por qué la gente la elige:**

- Cero fricción con Meta (sin Business Portfolio, sin verificación, sin tokens).
- Costo: solo el hosting (~5 USD/mes en Railway).
- Multi-instancia: un solo servidor maneja muchos números.
- Webhooks ricos: eventos para casi todo lo que pasa en el cliente.
- Comunidad activa en español/portugués, mucho contenido LATAM.

**Por qué no es una bala de plata:**

- WhatsApp puede banear el número en cualquier momento sin aviso.
- No hay plantillas (HSM) reales: los envíos masivos son detectados rápido.
- No hay quality rating ni tiers: o funciona o se bannea.
- Los términos de servicio de WhatsApp prohíben uso no autorizado de su red.
- Cambios del lado de WA pueden romper Baileys (y por ende Evolution) sin previo aviso.

## Cuándo usar Evolution API

| Caso | ¿Apto? | Comentario |
|---|---|---|
| Bot interno para empleados de una empresa | Sí | Volumen bajo, número descartable, riesgo aceptable. |
| Atención al cliente con mensajes 100% inbound | Sí | Usuario inicia siempre, sin patrones de spam. |
| Prototipo / MVP para validar idea con IA | Sí | Antes de invertir en Cloud API y plantillas. |
| Recordatorios a clientes conocidos (bajo volumen) | Sí con cuidado | Mantener cadencia humana, mensajes variados. |
| Marketing masivo / blast a leads fríos | **No** | Ban casi garantizado en horas o días. |
| Salud, finanzas, gobierno | **No** | Industrias reguladas exigen Cloud API + HSM auditables. |
| Operación crítica donde no se puede perder el número | **No** | Sin SLA, sin recuperación garantizada. |

## Cómo se conecta el número

Una "instancia" en Evolution API representa un número conectado. Flujo típico:

1. Crear la instancia con un nombre (`main`, `cliente-acme`, etc.).
2. Pedir QR o pairing code.
3. Escanear desde **un teléfono con WhatsApp ya instalado** o ingresar el pairing code en *Dispositivos vinculados*.
4. La sesión queda persistida (Postgres o filesystem). Si se cae el servidor, al reiniciar reconecta solo.

Crear instancia (ejemplo):

```bash
curl -X POST "https://{{EVO_HOST}}/instance/create" \
  -H "apikey: {{API_KEY}}" \
  -H "Content-Type: application/json" \
  -d '{
    "instanceName": "main",
    "qrcode": true,
    "integration": "WHATSAPP-BAILEYS"
  }'
```

| Placeholder | Descripción |
|---|---|
| `EVO_HOST` | Host público de Evolution (ej: `evo.midominio.com`). |
| `API_KEY` | Valor de `AUTHENTICATION_API_KEY` configurado en el servidor. |

## Endpoints más usados

| Acción | Método + Path | Nota |
|---|---|---|
| Crear instancia | `POST /instance/create` | Una por número. |
| Conectar (QR / pairing) | `GET /instance/connect/{instance}` | Devuelve QR base64 o code. |
| Estado | `GET /instance/connectionState/{instance}` | `open`, `close`, `connecting`. |
| Enviar texto | `POST /message/sendText/{instance}` | Body: `{ "number": "...", "text": "..." }`. |
| Enviar media | `POST /message/sendMedia/{instance}` | URL o base64. |
| Enviar botones | `POST /message/sendButtons/{instance}` | Soporte limitado vs Cloud API. |
| Configurar webhook | `POST /webhook/set/{instance}` | Eventos suscritos. |
| Listar contactos | `GET /chat/findContacts/{instance}` | |
| Logout | `DELETE /instance/logout/{instance}` | Cierra sesión sin borrar instancia. |

Detalle completo y payloads en [`07-integraciones/evolution-api/endpoints.md`](../07-integraciones/evolution-api/endpoints.md) (pendiente).

## Eventos por webhook

Los más relevantes para una integración con n8n + IA:

| Evento | Cuándo se dispara | Uso típico |
|---|---|---|
| `MESSAGES_UPSERT` | Llega o se envía un mensaje | Disparar el bot. |
| `MESSAGES_UPDATE` | Cambio de estado (entregado, leído) | Tracking. |
| `CONNECTION_UPDATE` | Cambio de estado de la sesión | Alertar caída. |
| `QRCODE_UPDATED` | Nuevo QR generado | Re-pairing. |
| `CONTACTS_UPSERT` | Contacto nuevo / actualizado | Sync con CRM. |

## Diferencias clave con Cloud API

| Dimensión | Cloud API | Evolution API |
|---|---|---|
| Oficialidad | Meta | Comunidad (Baileys) |
| HSM (plantillas) | Sí, con aprobación | No reales, simuladas con texto |
| Quality rating | Sí | No |
| Tiers | Sí (250 → ilimitado) | No, sin límite explícito |
| Riesgo de ban | Nulo | Alto |
| Webhooks | Configurados en Meta | Configurados en Evolution |
| Pricing | Free + por conversación | Solo hosting |
| Multi-número | Sí | Sí (instancias) |
| Soporte oficial | Meta | Comunidad |

## Versiones: v1 vs v2

Evolution tiene dos líneas activas con APIs distintas. **No son intercambiables.**

| Aspecto | v1 | v2 |
|---|---|---|
| Estado | Mantenimiento | Activa |
| Paths | `/message/text/...` | `/message/sendText/...` |
| Persistencia | Mongo o filesystem | Postgres recomendado |
| Eventos webhook | Diferentes nombres | Estandarizados |

Recomendación: **usar v2** salvo que se herede una integración v1. Pinear la versión del contenedor para evitar breaks.

## Buenas prácticas para reducir riesgo de ban

1. Usar **un número nuevo dedicado**, no el personal del cliente.
2. No abrir WhatsApp Web ni la app paralelamente en el mismo número durante operación.
3. Mantener **cadencia humana**: pausas entre envíos, no enviar a miles en ráfaga.
4. **Variar el contenido**: el mismo texto enviado a 200 contactos en 10 minutos = ban.
5. No enviar a números que no tienen guardado el número del negocio.
6. Implementar **opt-in real**: confirmar que el usuario quiere recibir mensajes.
7. Reaccionar a `CONNECTION_UPDATE` para alertar caídas y re-pairing.
8. Backup de la sesión (Postgres) por si hay que migrar de servidor.
9. Monitorear el ratio de mensajes salientes vs entrantes (idealmente < 3:1).
10. Evitar shortlinks tipo `bit.ly`, suelen marcar spam.

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| Instancia se queda en `connecting` eternamente | QR expiró o número ya tenía sesión activa en otro lado | Logout y volver a vincular con pairing code. |
| `Error: Connection Closed` recurrente | Múltiples sesiones simultáneas o número banneado | Verificar abriendo WhatsApp Web manualmente; si rebota, número banneado. |
| Webhook no llega a n8n | URL mal configurada o n8n caído | Probar con `webhook.site` primero; revisar `WEBHOOK_GLOBAL_URL`. |
| Mensajes salen pero no llegan | Número destino no tiene WhatsApp | Validar previamente con `/chat/whatsappNumbers`. |
| Tras redeploy se pide QR de nuevo | Persistencia mal configurada | Verificar `DATABASE_PROVIDER=postgresql` y URI correcta. |
| `apikey` rechazada | Header mal escrito | Es `apikey`, no `Authorization` ni `X-API-Key`. |

## Stack típico recomendado

Para producción mínima self-hosted ver la plantilla completa en [`08-despliegue/railway/plantilla-evolution-n8n-openai.md`](../08-despliegue/railway/plantilla-evolution-n8n-openai.md): Evolution + n8n + Postgres + Redis + Caddy en Railway.

## Referencias

- [Documentación oficial de Evolution API](https://doc.evolution-api.com/) — Verificado 2026-05-20.
- [Repositorio en GitHub](https://github.com/EvolutionAPI/evolution-api) — Verificado 2026-05-20.
- [Baileys](https://github.com/WhiskeySockets/Baileys) — librería subyacente.
- [`03-formas-de-uso/comparativa.md`](./comparativa.md) — comparativa cruzada.
- [`07-integraciones/evolution-api/`](../07-integraciones/evolution-api/) — endpoints, eventos, integraciones.
