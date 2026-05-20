# 03 · Formas de uso

Las distintas maneras de operar WhatsApp Business según el tipo de proyecto, presupuesto y madurez técnica del integrador.

## Documentos

- `comparativa.md` — Tabla cruzada de todas las opciones. **Empezar acá.**
- `cloud-api-oficial.md` — API oficial directa de Meta.
- `evolution-api.md` — No oficial, basado en Baileys. Útil pero riesgoso.
- `respond-io.md` — SaaS omnicanal partner oficial.
- `kommo-crm.md` — CRM con WA nativo, varios modelos de conexión.
- `bsps/` — Twilio, Gupshup y otros Business Solution Providers.
- `crms/` — HubSpot, Salesforce, Zoho con sus integraciones de WA.

## Cómo elegir

Mirar `comparativa.md` y cruzar con:

1. ¿El cliente necesita HSM oficiales y quality rating? → Cloud API, BSP, Respond.io o Kommo (vía Cloud).
2. ¿Es un prototipo interno o un cliente con bajo volumen y sin riesgo regulatorio? → Evolution API.
3. ¿El cliente ya tiene CRM y solo quiere mensajería integrada? → Kommo, HubSpot, Salesforce o Zoho.
4. ¿El cliente quiere una UI lista para agentes humanos? → Respond.io o Kommo.
