# StetikShop — Autenticación, Roles y Permisos

## 1. Autenticación

- **BR-02.1** — La autenticación la provee **Supabase Auth** (email + contraseña). El servidor Express valida el JWT de Supabase en cada request protegido.
- **BR-02.2** — Al registrarse (`POST /auth/signup`), se crea el usuario en `auth.users`. Un trigger de Supabase inserta su fila en `public.users` con `email`, `full_name` y `phone` (opcional).
- **BR-02.3** — Todo endpoint protegido exige el header `Authorization: Bearer <token>`.
- **BR-02.4** — Token inválido o expirado → `401 AUTH_REQUIRED`/`TOKEN_INVALID`. El cliente no navega a otra vista engañosamente: se despide la sesión y se redirige a `/login`.
- **BR-02.5** — El usuario puede consultar y editar su propio perfil (`/auth/me`): `full_name`, `phone`, `avatar_url`.

## 2. Estructura de roles (dos capas)

```
Roles GLOBALES  →  aplican a toda la plataforma        (business_id = NULL)
Roles POR NEGOCIO →  aplican solo dentro de un negocio  (business_id = <negocio>)
```

- **BR-02.6** — Un usuario puede tener varios roles, globales y por negocio, simultáneamente (ej. dueño del Negocio A y barbero del Negocio B).
- **BR-02.7** — `super_admin` es un rol global de sistema: tiene el permiso comodín `*`. No puede eliminarse ni modificarse su conjunto de permisos.
- **BR-02.8** — `owner` es un rol por negocio de sistema: se crea automáticamente al crear el negocio, tiene permiso comodín `*` y no puede eliminarse.
- **BR-02.9** — Solo el Super Admin gestiona los roles globales (`/admin/global-roles`). Solo el dueño (o quien tenga `manage_business_roles`) gestiona los roles de su negocio (`/businesses/:id/roles`).
- **BR-02.10** — El sistema permite al menos **10 roles** en total, todos configurables por pantalla (nombre y permisos), cumpliendo: super_admin (sistema) + roles globales configurables + owner (sistema por negocio) + roles de negocio configurables.

## 3. Permisos

Un rol contiene una lista de permisos. Cada permiso tiene: `code`, `display_name`, `description`, `category`, `level` (`global`|`business`).

### 3.1 Permisos globales (nivel plataforma)

| Código | Descripción |
|--------|-------------|
| `manage_all_businesses` | Gestionar todos los negocios |
| `manage_all_users` | Gestionar todos los usuarios |
| `manage_global_roles` | Gestionar roles globales |
| `manage_platform_settings` | Gestionar ajustes de plataforma |
| `view_analytics_global` | Ver analíticas globales |
| `manage_support_tickets` | Gestionar tickets de soporte |

### 3.2 Permisos por negocio

| Código | Categoría | Descripción |
|--------|-----------|-------------|
| `manage_business_settings` | negocio | Editar perfil y configuración del negocio |
| `manage_business_schedule` | negocio | Editar horarios y excepciones |
| `view_business_analytics` | negocio | Ver analíticas del negocio |
| `manage_business_roles` | negocio | Gestionar roles dentro del negocio |
| `manage_employees` | empleado | Crear/editar/desactivar empleados |
| `manage_employee_schedules` | empleado | Editar disponibilidad de empleados |
| `manage_services` | servicio | Crear/editar/eliminar servicios y asignarlos |
| `view_services` | servicio | Ver servicios |
| `manage_appointments` | cita | Editar citas y su estado |
| `view_appointments` | cita | Ver citas |
| `cancel_appointments` | cita | Cancelar citas |
| `mark_appointments_complete` | cita | Marcar citas completadas |
| `view_queue` | cola | Ver cola |
| `manage_queue` | cola | Llamar, saltar, gestionar cola |
| `manage_notifications` | notificación | Gestionar notificaciones |

### Reglas del comodín

- **BR-02.11** — El permiso comodín `*` (presente en `owner` y `super_admin`) concede TODOS los permisos de su nivel (global o negocio) sin necesidad de enumerarlos.

## 4. Verificación de permisos (middlevare `requireRole`)

```
requireRole('manage_services')
  1. Extraer business_id de la ruta/body según el endpoint
  2. Buscar roles del usuario (user_roles → business_roles)
  3. Pasa si alguno contiene el permiso solicitado O el comodín '*'
  4. Falla → 403 FORBIDDEN
```

- **BR-02.12** — La verificación se hace en el servidor. El frontend solo oculta/permite acciones como UX; nunca es la única barrera.
- **BR-02.13** — Los permisos globales se evalúan contra `global_roles`; los permisos de negocio contra `business_roles` del negocio en contexto.

## 5. Asignación de roles

- **BR-02.14** — El Super Admin asigna roles globales: `POST /admin/user-roles { user_id, role_id }`.
- **BR-02.15** — El dueño/gestor asigna roles de negocio: `POST /businesses/:id/user-roles { user_id, role_id }`.
- **BR-02.16** — Un usuario no puede tener el mismo rol dos veces en el mismo negocio (restricción única `(user_id, role_id, business_id)`). Pero sí el mismo nombre de rol en negocios distintos (son registros distintos).
- **BR-02.17** — Al crear un empleado en un negocio, se asigna automáticamente el rol por negocio `employee` (permisos mínimos: `view_services`, `view_business_schedule`).
- **BR-02.18** — No se puede eliminar un rol si usuarios lo tienen asignado: primero deben reasignarse o removerse esas asignaciones.

## 6. Protección de datos (RLS en base de datos)

Como seguridad en profundidad, todas las tablas tienen RLS activado. Reglas resumidas:

| Tabla | Acceso |
|-------|--------|
| `users` | Solo propia fila (super admin: todas) |
| `global_roles` | Lectura: autenticados · escritura: super admin |
| `business_roles` | Lectura: autenticados · escritura: owner/`manage_business_roles` |
| `businesses` | Público: published · dueño: las suyas · super admin: todas |
| `appointments` | Cliente: propias · empleado: asignadas · staff: las del negocio |
| `queue_entries` | Igual que citas + vista pública de cola |
| `notifications` | Solo el propio usuario |
| `platform_settings` | Lectura: autenticados · escritura: super admin |

- **BR-02.19** — RLS es una red de seguridad: el control principal de permisos ocurre en Express. Ninguna regla anterior se deroga si RLS es más permisivo; las políticas de RLS se mantienen tan restrictivas como las de la API.

## 7. Casos de prueba sugeridos

1. Registro/Login/Logout y persistencia de sesión al refrescar.
2. Usuario con 2 negocios: como owner del A y barbero del B — permisos correctos en cada contexto.
3. Super admin crea/edita/elimina un rol global y un rol por negocio configurable.
4. Eliminar un rol con usuarios asignados → error `CONFLICT` con mensaje claro.
5. Empleado intentando `manage_services` → `403 FORBIDDEN`.
6. Asignación duplicada de rol en el mismo negocio → `CONFLICT`.
7. Token inválido → `401`; refrescar sesión funciona.
8. Al menos 10 roles configurables por pantalla cumpliendo el requisito del cliente.