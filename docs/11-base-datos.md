# StetikShop — Esquema de Base de Datos (Reglas asociadas)

> El esquema vive en Supabase/PostgreSQL. Todas las PK son `uuid` (`gen_random_uuid()`), timestamps `timestamptz`. Esta página resume las **reglas** de cada tabla (detalles de negocio en los documentos 02–08).

## 1. Tablas y propósito

| Tabla | Propósito |
|-------|-----------|
| `users` | Perfil de usuario (sincronizada desde `auth.users` por trigger) |
| `global_roles` | Roles de plataforma (configurables por super admin) |
| `business_roles` | Roles por negocio (configurables por dueño) |
| `user_roles` | Asignación usuario→rol (global o por negocio) |
| `permission_definitions` | Catálogo de permisos disponibles |
| `businesses` | Negocio (perfil público + datos) |
| `business_config` | Configuración clave-valor por negocio |
| `business_schedules` | Horario semanal del negocio (7 filas) |
| `business_schedule_overrides` | Excepciones de horario por fecha |
| `employees` | Empleado = relación usuario↔negocio |
| `employee_availability` | Disponibilidad semanal del empleado (7 filas) |
| `employee_availability_overrides` | Excepciones de disponibilidad por fecha |
| `services` | Servicio ofrecido por el negocio |
| `employee_services` | Asignación de servicios a empleados (N:M) |
| `appointments` | Cita unificada (cola o agenda) |
| `queue_entries` | Datos específicos de turnos de cola (1:1 con appointments de tipo cola) |
| `notifications` | Notificaciones in-app (más flag de email) |
| `platform_settings` | Configuración global de la plataforma |

## 2. Reglas por tabla (resumen)

### users
- `id` = mismo id de `auth.users`. Email único. `is_active` para baja suave.
- Trigger `on_auth_user_created`: al crear en `auth.users` → inserta fila en `users`.

### global_roles / business_roles
- `is_system = true` → no se puede eliminar ni cambiar permisos base (`super_admin`, `owner`).
- `permissions` = `jsonb` (array de códigos; `["*"]` = todos).
- `business_roles`: único `(business_id, name)`.

### user_roles
- `role_source` ∈ (`global`|`business`). `business_id` NULL para roles globales.
- Único `(user_id, role_id, business_id)`.

### permission_definitions
- Catálogo semilla (ver [02-auth-roles.md] §3), con `code` único y `level` (`global`|`business`).

### businesses
- `slug` único. `timezone` IANA. `type` ∈ (barbershop|salon|both).
- Trigger de creación: rol `owner`, 7 filas de `business_schedules` cerradas, `business_config` por defecto.

### business_config
- Único `(business_id, key)`. Claves: ver [03-negocios.md] §3.

### business_schedules
- Único `(business_id, day_of_week)`, `day_of_week` 0–6.
- Cerrado ⇒ `open_time/close_time` NULL; abierto ⇒ ambos NOT NULL. Cross de medianoche permitido.

### business_schedule_overrides
- Único `(business_id, override_date)`. Gana sobre `business_schedules`.

### employees
- Único `(business_id, user_id)`.
- Trigger de creación: rol `employee`, 7 filas de `employee_availability` no disponibles.

### employee_availability / overrides
- Único `(employee_id, day_of_week)` / `(employee_id, override_date)`. Mismas reglas de apertura/cierre.

### services
- `price` ≥ 0 (numeric(10,2)); `duration_minutes` > 0; `agenda_extra_fee` ≥ 0.
- `service_type` ∈ (queue|agenda|both); `max_daily_slots` > 0 o NULL.
- Único `(business_id, name)` para evitar duplicados de servicio.

### employee_services
- Único `(employee_id, service_id)`.

### appointments
- `appointment_type` ∈ (queue|agenda). `status` ∈ (pending|confirmed|in_progress|completed|cancelled|no_show|expired).
- Trigger `set_appointment_price`: `total_price = price` (+ `agenda_extra_fee` si `agenda`).
- `scheduled_at` + `scheduled_date` solo para agenda (denormalizado para consultas eficientes).
- Transiciones validadas en capa de negocio (ver [08-agenda.md] §4 y [07-cola.md] §7).

### queue_entries
- `appointment_id` único (1:1). `queue_position` entero. `estimated_wait_minutes` calculado.
- Índice parcial para la cola activa (solo `pending`/`confirmed`).

### notifications
- `type` según disparadores ([09-notificaciones.md] §2). Flags `is_read`, `email_sent`.

### platform_settings
- `key` único; valor semilla: `platform_name`, `allow_self_registration`, `default_queue_estimate_minutes`, `max_businesses_per_owner`, `default_timezone` (`"UTC"`), `default_currency` (`"USD"`), `default_slot_interval_minutes` (`15`).

## 3. Seguridad (RLS)

- **BR-11.1** — RLS habilitado en **todas** las tablas (matriz completa en [02-auth-roles.md] §6).
- **BR-11.2** — Las funciones helper `is_super_admin()`, `has_business_permission(business_id, permission)`, `is_business_owner(business_id)`, `is_employee_at(business_id)` se usan en las políticas.
- **BR-11.3** — El CRUD de la app pasa por Express (con permisos en código); RLS es defensa en profundidad.

## 4. Índices de rendimiento

- `idx_users_email` (único)
- `idx_businesses_slug` (único), `idx_businesses_owner`
- `idx_user_roles_user`, `idx_user_roles_business`
- `idx_services_business`, `idx_services_active`
- `idx_employees_business`, `idx_employees_user`
- `idx_appointments_business_date`, `idx_appointments_client`, `idx_appointments_employee`, `idx_appointments_status`, `idx_appointments_type`, índice parcial de agenda por `scheduled_at`
- `idx_queue_business_position`, `idx_queue_business_joined`, `idx_queue_active` (parcial)
- `idx_notifications_user`

## 5. Reglas transversales de integridad

- **BR-11.4** — Toda FK respeta integridad referencial; las bajas suaves de entidades padre no rompen referencias históricas.
- **BR-11.5** — Los timestamps: `created_at` NOT NULL (default `now()`), `updated_at` NOT NULL (default `now()`, actualizado en cada UPDATE).
- **BR-11.6** — Cada tabla con permisos/roles registra `created_at`/`updated_at` y, cuando aplica, `created_by`/`updated_by` para auditoría.
- **BR-11.7** — Las migraciones se versionan en `supabase/migrations/` y son reproducibles de local a producción (solo cambia la `.env`).