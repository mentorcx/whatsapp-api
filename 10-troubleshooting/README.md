# 10 · Troubleshooting

Errores frecuentes y diagnósticos rápidos.

## Documentos

- `codigos-error.md` — Tabla de códigos de error de la Cloud API.
- `webhooks-no-llegan.md` — Diagnóstico paso a paso.
- `plantillas-pausadas.md` — Por qué se pausan y cómo recuperar.
- `tokens.md` — Token expirado, permisos faltantes, scopes mal seteados.
- `rate-limits.md` — Identificar y respetar límites.
- `evolution-desconexion.md` — Por qué Evolution pierde la sesión.

## Cómo reportar un error nuevo

1. Capturar el request completo (curl reproducible).
2. Capturar el response (status, headers, body).
3. Adjuntar versión de la API (`v21.0`, etc.) y timestamp UTC.
4. Anonimizar datos sensibles.
5. Si aplica, agregar el código a `codigos-error.md`.
