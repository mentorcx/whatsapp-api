---
title: Crear y configurar el Business Portfolio
category: onboarding-meta
tags: [business-portfolio, business-manager, onboarding, meta]
updated: 2026-05-20
source_official:
  - https://www.facebook.com/business/help/1710077379203657
  - https://business.facebook.com/
related:
  - 01-onboarding-meta/verificacion-comercial.md
  - 01-onboarding-meta/system-users-tokens.md
  - 00-fundamentos/arquitectura.md
audience: [integrador, dev, comercial]
---

# Crear y configurar el Business Portfolio

> **TL;DR:** El Business Portfolio (ex Business Manager) es el contenedor de todos los activos comerciales del cliente en Meta. **Tiene que estar a nombre del cliente final**, no del integrador. Se crea en `business.facebook.com/overview` y luego se agregan App, WABA, System User y se invita al integrador con permisos.

## Contexto

Este es el primer recurso que se crea en cualquier proyecto. Si se hace mal acá, todo lo de arriba queda con problemas de propiedad. La regla más importante: **el dueño del Portfolio debe ser el cliente final** (la empresa que vende), no la consultora que integra.

## Por qué importa quién es dueño

| Escenario | Problema |
|---|---|
| Portfolio creado a nombre del integrador | Cliente queda atado, no puede portar el número fácilmente |
| Portfolio del cliente con integrador como admin | Correcto: cliente puede revocar acceso en cualquier momento |
| Portfolio "compartido" con varios clientes | Imposible: 1 Portfolio = 1 entidad legal |
| Sin Portfolio, solo cuenta personal del dueño | No se pueden agregar System Users; bloquea producción |

## Pasos para crear

### 1. Cuenta personal de Facebook del dueño legal

El cliente necesita una cuenta personal de Facebook **real y activa** del director / dueño de la empresa. Meta exige esto: no se puede crear Portfolio desde una cuenta institucional anónima.

| Requisito | Detalle |
|---|---|
| Cuenta validada | Email + teléfono confirmados |
| Sin restricciones | Que no esté en estado limitado por Meta |
| Nombre real | Si difiere mucho del DNI, Meta lo cuestiona |

### 2. Crear el Portfolio

Ir a `https://business.facebook.com/overview` → **Crear cuenta**.

Datos pedidos:

| Campo | Qué cargar |
|---|---|
| Nombre del Portfolio | Razón social o nombre comercial del cliente |
| Tu nombre | Nombre del usuario que crea (cliente, no integrador) |
| Email del negocio | Dominio propio del cliente (ej. `admin@cliente.com`) |

El Portfolio se crea inmediatamente con un `business_id` numérico visible en URL: `business.facebook.com/settings/info?business_id=XXX`.

### 3. Completar info de negocio

Settings → Información del negocio:

| Campo | Importancia |
|---|---|
| Nombre legal | Crítico, debe coincidir con documento |
| Dirección | Crítica para verificación |
| Sitio web | Crítico, debe ser del cliente |
| País | Crítico, no se puede cambiar después |
| Categoría | Importante para reglas regionales |
| Teléfono comercial | Importante para verificación |

Toda esta info entra en juego cuando se solicita verificación comercial.

### 4. Agregar al integrador como Administrador

Settings → Usuarios → **Personas** → Agregar:

| Permiso | Para qué |
|---|---|
| Administrador del Portfolio | Acceso total. Reservado para 1-2 personas. |
| Empleado | Acceso limitado, requiere asignación de activos. |

Recomendación: el integrador entra como **Administrador** mientras dura el proyecto. Si el cliente prefiere mantener el rol de admin reservado, alcanza con `Empleado` + asignación explícita de cada activo (Apps, WABAs).

### 5. Crear o asociar la App de Meta

Settings → Cuentas → **Apps** → Agregar.

Opciones:

| Camino | Cuándo |
|---|---|
| Crear app nueva dentro del Portfolio | Default. Más simple. |
| Asociar app existente | Si el integrador ya tiene una app multi-cliente |
| App del BSP | Si el cliente trabaja con BSP, la app la maneja el BSP |

Al crear la app:

- **Tipo de app**: `Business`.
- **Caso de uso**: `Other` → más adelante se agregan productos.
- **Productos**: agregar **WhatsApp** desde el dashboard de la App.

### 6. Crear la WABA

Settings → Cuentas → **Cuentas de WhatsApp** → Agregar → **Crear una WABA**.

Datos:

| Campo | Detalle |
|---|---|
| Nombre WABA | Identificador interno (ej `Cliente · Soporte`) |
| Zona horaria | Importante para reportes |
| Categoría | Default `Other`, refinar después |

Una vez creada, asignar la App al WABA: WABA → **Configuración** → **Configurar API**.

### 7. Crear System User

Settings → Usuarios → **Usuarios del sistema** → Agregar:

| Campo | Detalle |
|---|---|
| Nombre | Descriptivo, ej `integracion-prod` |
| Rol | `Empleado` (no `Admin` salvo necesidad real) |

Asignar activos al System User:

- **App** → permiso `Develop`.
- **WABA** → permiso `Manage` o `Send messages`.

Generar token: System User → **Generar token** → seleccionar App → seleccionar scopes:

- `whatsapp_business_messaging`
- `whatsapp_business_management`
- `business_management`

**Guardar el token de inmediato** en gestor de secretos. Es de larga duración pero no recuperable (hay que regenerar si se pierde).

Detalle en [`system-users-tokens.md`](./system-users-tokens.md).

### 8. Verificación comercial

Solo necesaria si se quiere:

- Más de 1-2 números de WA.
- Display name personalizado.
- Más allá del tier base.

Iniciar desde Security Center del Portfolio. Ver [`verificacion-comercial.md`](./verificacion-comercial.md).

## Checklist final

| Ítem | ✓ |
|---|---|
| Portfolio a nombre del cliente | |
| Cliente listado como creador / admin principal | |
| Integrador agregado como Admin o Empleado con permisos suficientes | |
| Info de negocio completa (legal name, address, web) | |
| App creada y con WhatsApp habilitado | |
| WABA creada y vinculada a la App | |
| System User con tokens y scopes correctos | |
| Token guardado en gestor de secretos del cliente o integrador | |
| (Opcional) Verificación comercial iniciada | |

## Errores comunes

| Error | Causa | Solución |
|---|---|---|
| "No puedo crear Portfolio" | Cuenta personal limitada o nueva | Esperar a que la cuenta tenga historia o usar otra |
| "Cliente perdió acceso al Portfolio" | No quedó admin nadie de su lado | Pedir reset vía soporte Meta (proceso lento) |
| Webhook funciona pero plantillas fallan | App no asociada a la WABA correcta | Verificar el binding en WABA → Configurar API |
| Token rechaza con 200 (`Permissions`) | Scopes faltantes en el System User | Regenerar con scopes completos |
| Cliente quiere migrar a otro proveedor | Portfolio del cliente facilita la portabilidad | (Si está bien creado, basta cambiar permisos) |

## Anti-patterns

- **No** crear un Portfolio por cada cliente dentro del Portfolio del integrador.
- **No** usar la cuenta personal del integrador como dueño.
- **No** compartir credenciales del cliente final con el equipo del integrador (usar System Users + roles).
- **No** habilitar 2FA solamente en la cuenta personal del integrador: el cliente también debe tenerlo en la suya.

## Referencias

- [Business Manager · Help Center](https://www.facebook.com/business/help) — Verificado 2026-05-20.
- [Crear una Business Account](https://www.facebook.com/business/help/1710077379203657) — Verificado 2026-05-20.
- [`01-onboarding-meta/verificacion-comercial.md`](./verificacion-comercial.md)
- [`01-onboarding-meta/system-users-tokens.md`](./system-users-tokens.md)
- [`00-fundamentos/arquitectura.md`](../00-fundamentos/arquitectura.md)
