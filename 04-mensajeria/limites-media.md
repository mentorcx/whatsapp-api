---
title: Límites de media en Cloud API
category: mensajeria
tags: [media, limites, image, video, audio, document, sticker]
updated: 2026-05-20
source_official:
  - https://developers.facebook.com/docs/whatsapp/cloud-api/reference/media
related:
  - 04-mensajeria/tipos-de-mensaje.md
audience: [integrador, dev]
---

# Límites de media en Cloud API

> **TL;DR:** Imagen 5 MB (JPG/PNG), Video 16 MB (MP4 H.264+AAC), Audio 16 MB (varios codecs), Document 100 MB (preferentemente PDF), Sticker 100 KB estático / 500 KB animado (WebP). Tamaños y formatos rígidos. Media IDs expiran a 30 días y son privados al `phone_number_id`. URLs en `link` deben ser HTTPS públicas.

## Tabla resumen

| Tipo | Tamaño máx | Formatos soportados |
|---|---|---|
| Image | 5 MB | JPEG, PNG |
| Video | 16 MB | MP4 (H.264 video + AAC audio), 3GP |
| Audio | 16 MB | AAC, AMR, MP3, MP4 audio, OGG (Opus) |
| Document | 100 MB | PDF (preferido), DOC, DOCX, XLS, XLSX, PPT, PPTX, TXT |
| Sticker (estático) | 100 KB | WebP 512×512 |
| Sticker (animado) | 500 KB | WebP animado |
| Voice message | 16 MB | OGG (Opus) |

**Verificado: 2026-05-20** contra docs de Meta.

## Image

| Aspecto | Detalle |
|---|---|
| Formato | JPEG, PNG |
| Tamaño | ≤ 5 MB |
| Resolución | Sin límite duro; WA recomprime |
| Aspect ratio | Cualquiera; WA puede recortar en preview |
| Transparencia | PNG sí, JPEG no |
| Color profile | sRGB recomendado |
| Caption | Opcional, ≤ 1024 chars |

Anti-patterns:

- HEIC (iPhone) no soportado: convertir a JPEG antes.
- BMP, TIFF, RAW no soportados.
- Imágenes con texto crítico muy chico (WA recomprime).

## Video

| Aspecto | Detalle |
|---|---|
| Formato | MP4 (H.264 + AAC), 3GP |
| Tamaño | ≤ 16 MB |
| Duración | Hasta minutos, pero el tamaño lo limita en la práctica |
| Audio codec | AAC obligatorio (no MP3 dentro del MP4) |
| Video codec | H.264 (preferido); H.265 a veces falla |
| Aspect ratio | Cualquiera |
| Caption | Opcional, ≤ 1024 chars |

Anti-patterns:

- MOV (QuickTime) sin re-encode no funciona.
- 4K innecesariamente grande: bajar a 720p o 1080p.
- Audio en formato distinto a AAC.

## Audio

| Aspecto | Detalle |
|---|---|
| Formato | AAC, AMR, MP3, MP4 audio, OGG (Opus) |
| Tamaño | ≤ 16 MB |
| Voice message | OGG con Opus, mono, 16 kHz típicamente |
| Caption | **No soportado** |

Para reproducir como "mensaje de voz" (no archivo adjunto), enviar OGG con codec Opus.

Anti-patterns:

- WAV sin comprimir: pesa mucho, no es ideal.
- M4A puro (sin contenedor MP4): puede no reproducir.

## Document

| Aspecto | Detalle |
|---|---|
| Formato | PDF (recomendado), DOC, DOCX, XLS, XLSX, PPT, PPTX, TXT, ZIP, otros |
| Tamaño | ≤ 100 MB |
| Filename | Recomendado; aparece visible al usuario |
| Caption | Opcional, ≤ 1024 chars |

Anti-patterns:

- Documentos > 100 MB: dividir o linkear a download externo.
- Nombres sin extensión: usuario no entiende qué es.

## Sticker

| Aspecto | Detalle |
|---|---|
| Formato | WebP |
| Estático | ≤ 100 KB, exactamente 512×512 px |
| Animado | ≤ 500 KB, WebP animado |
| Caption | No soportado |
| Transparencia | Recomendada |

Anti-patterns:

- GIF (no soportado como sticker).
- PNG/JPEG (no son stickers, son images).
- Sticker > 100 KB estático: rechazado.

## Cómo enviar media

### Por URL pública

```json
{
  "type": "image",
  "image": {
    "link": "https://midominio.com/img.jpg",
    "caption": "Texto opcional"
  }
}
```

Requisitos del `link`:

| Requisito | Detalle |
|---|---|
| HTTPS | Sí |
| Pública (sin auth) | Meta no pasa credentials |
| `Content-Type` correcto | Meta lo lee del response |
| Latencia | < 5s para evitar timeout |
| Tamaño | El servidor debe permitir el rango |

### Por media_id (subida previa)

```json
{
  "type": "image",
  "image": {
    "id": "{{MEDIA_ID}}",
    "caption": "Texto opcional"
  }
}
```

Subir primero:

```bash
curl -X POST \
  "https://graph.facebook.com/v21.0/{{PHONE_NUMBER_ID}}/media" \
  -H "Authorization: Bearer {{ACCESS_TOKEN}}" \
  -F "file=@/path/img.jpg" \
  -F "type=image/jpeg" \
  -F "messaging_product=whatsapp"
```

Respuesta:

```json
{ "id": "1234567890" }
```

Ese `id` es el `media_id`.

| Detalle | Valor |
|---|---|
| TTL | 30 días |
| Scope | Privado al `phone_number_id` que lo subió |
| Reuso | Se puede usar muchas veces dentro del TTL |

## Descargar media entrante

Cuando el usuario envía media, llega `id` en el webhook. Para descargar:

```bash
# 1. Obtener URL temporal
curl -X GET \
  "https://graph.facebook.com/v21.0/{{MEDIA_ID}}" \
  -H "Authorization: Bearer {{ACCESS_TOKEN}}"
```

Respuesta:

```json
{
  "url": "https://lookaside.fbsbx.com/...",
  "mime_type": "image/jpeg",
  "sha256": "...",
  "file_size": 12345,
  "id": "...",
  "messaging_product": "whatsapp"
}
```

```bash
# 2. Descargar con el token
curl -X GET "{{URL}}" \
  -H "Authorization: Bearer {{ACCESS_TOKEN}}" \
  --output file.jpg
```

| Detalle | Valor |
|---|---|
| TTL del URL | ~5 minutos |
| Auth requerida | Sí, con el token |
| Reuso | Cada GET regenera URL |

**Patrón:** descargar de inmediato al recibir el webhook, no diferir.

## Validar tamaño antes de enviar

Si manejás contenido de usuarios:

```javascript
async function validateImage(buffer, mimeType) {
  const limits = {
    'image/jpeg': 5 * 1024 * 1024,
    'image/png': 5 * 1024 * 1024
  };
  if (!limits[mimeType]) return { ok: false, reason: 'unsupported_format' };
  if (buffer.length > limits[mimeType]) return { ok: false, reason: 'too_large' };
  return { ok: true };
}
```

Idem para video, audio, document. Ahorra errores 131053.

## Optimización de tamaño

| Caso | Estrategia |
|---|---|
| Imagen 5+ MB | Re-encode a JPEG 80-85% calidad |
| Video con audio alto bitrate | Bajar audio a 128 kbps AAC |
| Video 1080p innecesario | Bajar a 720p |
| PDF con muchas imágenes | Usar compresión de PDF (ghostscript) |

Herramientas:

- `ffmpeg -i in.mp4 -vcodec h264 -acodec aac -b:a 128k -b:v 1500k out.mp4`
- `convert in.jpg -quality 80 out.jpg` (ImageMagick)
- `gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.4 -dPDFSETTINGS=/ebook -o out.pdf in.pdf`

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| 131052 (`Media download error`) | URL no accesible, lenta, requiere auth | Probar la URL desde fuera, subir vía `/media` |
| 131053 (`Media upload error`) | Tamaño/formato inválido | Validar antes |
| Imagen llega rota | Formato no soportado (HEIC, BMP) | Convertir a JPEG |
| Audio no reproduce en algunos teléfonos | OGG sin Opus | Usar AAC o MP3 |
| Sticker llega como imagen plana | Tamaño excede o WebP malformado | Re-encode a WebP 512×512 |
| Video se ve sin audio | Codec audio no AAC | Re-encode con AAC |
| Media_id de hace 31 días no funciona | Expiró | Re-subir |

## Buenas prácticas

- Para envíos repetitivos de la misma media, **subir 1 vez** y reusar `media_id`.
- Servir media desde CDN propio si el volumen es alto.
- Cachear las descargas de media entrante: la URL expira pero el binario lo guardás vos.
- Filename descriptivo: `Factura-2026-05-Cliente.pdf` mejor que `1234.pdf`.
- No enviar > 1 archivo por mensaje (no soportado; mandar en mensajes separados).
- Imagen + caption > imagen + texto suelto.

## Referencias

- [Media · Cloud API](https://developers.facebook.com/docs/whatsapp/cloud-api/reference/media) — Verificado 2026-05-20.
- [Supported Media Types](https://developers.facebook.com/docs/whatsapp/cloud-api/reference/media#supported-media-types) — Verificado 2026-05-20.
- [`04-mensajeria/tipos-de-mensaje.md`](./tipos-de-mensaje.md)
