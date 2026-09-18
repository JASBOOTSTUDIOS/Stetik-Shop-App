# StetikShop — Contrato de la API REST

> `Base URL: /api/v1`. Autenticación: `Authorization: Bearer <Supabase JWT>`.
> Formato de respuesta/error: ver [10-errores.md].

## 1. Auth

| Método | Ruta | Auth | Descripción |
|--------|------|------|-------------|
| POST | `/auth/signup` | — | Registrar usuario `{email, password, full_name, phone?}` |
| POST | `/auth/login` | — | Iniciar sesión `{email, password}` |
| POST | `/auth/logout` | Sí | Cerrar sesión (elimina token en cliente) |
| GET | `/auth/me` | Sí | Perfil + roles del usuario |
| PUT | `/auth/me` | Sí | Editar perfil `{full_name?, phone?, avatar_url?}` |
| POST | `/auth/refresh` | — | Refrescar token `{refresh_token}` |

## 2. Super Admin

| Método | Ruta | Permiso | Descripción |
|--------|------|---------|-------------|
| GET | `/admin/users` | super_admin | Lista usuarios (paginado, búsqueda, filtro `is_active`) |
| GET | `/admin/users/:id` | super_admin | Detalle de usuario |
| PUT | `/admin/users/:id` | super_admin | Actualizar usuario (admin) |
| GET | `/admin/businesses` | super_admin | Lista negocios |
| GET | `/admin/businesses/:id` | super_admin | Detalle de negocio |
| PUT | `/admin/businesses/:id` | super_admin | Editar cualquier negocio |
| DELETE | `/admin/businesses/:id` | super_admin | Desactivar negocio |
| GET | `/admin/global-roles` | super_admin | Lista roles globales |
| POST | `/admin/global-roles` | super_admin | Crear rol global |
| PUT | `/admin/global-roles/:id` | super_admin | Editar rol global |
| DELETE | `/admin/global-roles/:id` | super_admin | Eliminar rol global (no sistema) |
| GET | `/admin/permissions` | super_admin | Catálogo de permisos |
| GET | `/admin/stats` | super_admin | Estadísticas de la plataforma |
| GET | `/admin/settings` | super_admin | Ajustes de plataforma |
| PUT | `/admin/settings/:key` | super_admin | Actualizar ajuste |
| POST | `/admin/user-roles` | super_admin | Asignar rol global a usuario |
| DELETE | `/admin/user-roles/:id` | super_admin | Quitar asignación |

## 3. Negocios

| Método | Ruta | Permiso | Descripción |
|--------|------|---------|-------------|
| GET | `/businesses` | — | Lista negocios publicados (search, type, paginado) |
| GET | `/businesses/:slug` | — | Perfil público (descripción md, horarios, servicios) |
| POST | `/businesses` | autenticado | Crear negocio (init rol owner, horarios, config) |
| PUT | `/businesses/:id` | owner/`manage_business_settings` | Editar negocio |
| DELETE | `/businesses/:id` | owner | Baja suave |
| GET | `/businesses/:id/dashboard` | staff del negocio | Dashboard (cola hoy, citas hoy, stats) |

### Roles del negocio
| Método | Ruta | Permiso |
|--------|------|---------|
| GET/POST | `/businesses/:id/roles` | owner/`manage_business_roles` |
| PUT/DELETE | `/businesses/:id/roles/:rid` | owner/`manage_business_roles` (no sistema) |

### Asignaciones de rol
| Método | Ruta | Permiso |
|--------|------|---------|
| GET/POST | `/businesses/:id/user-roles` | owner/`manage_business_roles` |
| DELETE | `/businesses/:id/user-roles/:ucid` | owner/`manage_business_roles` |

### Config
| Método | Ruta | Permiso |
|--------|------|---------|
| GET | `/businesses/:id/config` | owner/manager |
| PUT | `/businesses/:id/config` | owner/manager (batch) |

## 4. Empleados

| Método | Ruta | Permiso |
|--------|------|---------|
| GET | `/businesses/:id/employees` | autenticado |
| GET | `/businesses/:id/employees/:eid` | autenticado |
| POST | `/businesses/:id/employees` | owner/`manage_employees` |
| PUT | `/businesses/:id/employees/:eid` | owner/manager/self (campos limitados) |
| DELETE | `/businesses/:id/employees/:eid` | owner (baja suave) |

## 5. Servicios

| Método | Ruta | Permiso |
|--------|------|---------|
| GET | `/businesses/:id/services` | autenticado (filtro `is_active`) |
| GET | `/businesses/:id/services/:sid` | autenticado |
| POST | `/businesses/:id/services` | owner/`manage_services` |
| PUT | `/businesses/:id/services/:sid` | owner/`manage_services` |
| DELETE | `/businesses/:id/services/:sid` | owner/`manage_services` (soft) |
| POST | `/businesses/:id/services/:sid/employees` | owner/`manage_services` |
| DELETE | `/businesses/:id/services/:sid/employees/:eid` | owner/`manage_services` |
| GET | `/businesses/:id/services/:sid/employees` | autenticado |

## 6. Horarios

| Método | Ruta | Permiso |
|--------|------|---------|
| GET | `/businesses/:id/schedule` | autenticado |
| PUT | `/businesses/:id/schedule` | owner/`manage_business_schedule` (7 días) |
| GET | `/businesses/:id/schedule/overrides` | autenticado (`?from&to`) |
| POST | `/businesses/:id/schedule/overrides` | owner/`manage_business_schedule` |
| PUT/DELETE | `/businesses/:id/schedule/overrides/:oid` | owner/`manage_business_schedule` |
| GET | `/businesses/:id/employees/:eid/availability` | autenticado |
| PUT | `/businesses/:id/employees/:eid/availability` | self/owner/`manage_employee_schedules` |
| GET | `/businesses/:id/employees/:eid/availability/overrides` | autenticado |
| POST | `/businesses/:id/employees/:eid/availability/overrides` | self/owner/`manage_employee_schedules` |
| DELETE | `/businesses/:id/employees/:eid/availability/overrides/:oid` | self/owner/`manage_employee_schedules` |

### Slots (cálculo de cupos)
| Método | Ruta | Permiso | Query |
|--------|------|---------|-------|
| GET | `/businesses/:id/slots` | autenticado | `date`, `service_id`, `employee_id?` |

Algoritmo completo en [06-horarios.md] §5.

## 7. Citas y cola

| Método | Ruta | Permiso |
|--------|------|---------|
| POST | `/businesses/:id/appointments` | cliente (reserva agenda) |
| POST | `/businesses/:id/queue/join` | cliente (unirse a cola) |
| GET | `/businesses/:id/appointments` | staff/`view_appointments` (filtros) |
| GET | `/businesses/:id/appointments/:aid` | cliente (propia)/staff/owner |
| PUT | `/businesses/:id/appointments/:aid` | staff/`manage_appointments` (estado/notas) |
| POST | `/businesses/:id/appointments/:aid/cancel` | cliente (propia) o staff |
| POST | `/businesses/:id/appointments/:aid/start` | empleado asignado/owner |
| POST | `/businesses/:id/appointments/:aid/complete` | empleado/owner |
| POST | `/businesses/:id/appointments/:aid/no-show` | staff |
| GET | `/businesses/:id/queue` | staff/`view_queue` |
| POST | `/businesses/:id/queue/call-next` | staff/`manage_queue` |
| POST | `/businesses/:id/queue/:aid/call` | staff/`manage_queue` |
| POST | `/businesses/:id/queue/:aid/skip` | staff/`manage_queue` |

## 8. Cliente (perfil propio)

| Método | Ruta | Autenticación |
|--------|------|---------------|
| GET | `/my/appointments` | sí (historial con filtros/paginación) |
| GET | `/my/queue-status` | sí (posiciones de cola activas) |
| GET | `/my/notifications` | sí (filtro `is_read`) |
| GET | `/my/notifications/unread-count` | sí |
| PUT | `/my/notifications/:nid/read` | sí |
| PUT | `/my/notifications/read-all` | sí |
| DELETE | `/my/notifications/:nid` | sí |

## 9. Uploads

| Método | Ruta | Permiso | Límites |
|--------|------|---------|---------|
| POST | `/upload/logo` | owner del negocio | 2MB jpg/png/webp |
| POST | `/upload/cover` | owner | 5MB |
| POST | `/upload/employee-photo` | owner/manager/self | 2MB |
| POST | `/upload/service-image` | owner/service-manager | 3MB |
| POST | `/upload/avatar` | autenticado | 2MB |
| DELETE | `/upload/:id` | owner del archivo/negocio | — |

Buckets: `business-images`, `employee-photos`, `user-avatars`. Rutas: ver plan técnico §8.

## 10. Estadísticas

| Método | Ruta | Permiso |
|--------|------|---------|
| GET | `/businesses/:id/stats` | owner/manager/`view_business_analytics` (`?period=today|week|month|year`) |
| GET | `/businesses/:id/stats/queue` | owner/manager |

Respuesta de stats típica: `total_appointments`, `completed`, `cancelled`, `no_show`, `total_revenue`, `avg_wait_minutes`, `popular_services`, `busy_hours`, `employee_stats`.

## 11. Health

| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/health` | Estado del servidor (200 si vive) |

## 12. Reglas del contrato

- **BR-12.1** — Toda respuesta usa el formato estándar (§1–2 de [10-errores.md]).
- **BR-12.2** — Los permisos se evalúan en el servidor por ruta (`requireRole`); el contexto del negocio se toma de la ruta o body.
- **BR-12.3** — Paginación: `?page&limit` (meta: `page, limit, total, totalPages`).
- **BR-12.4** — Búsquedas con parámetro `search`; filtros por estado/tipo/fecha según cada listado.
- **BR-12.5** — Toda escritura que desafíe restricciones de negocio retorna el código de error correcto de [10-errores.md] §3.