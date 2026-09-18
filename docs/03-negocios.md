# StetikShop — Reglas de Gestión de Negocios

## 1. Creación de negocios

- **BR-03.1** — Cualquier usuario autenticado puede crear un negocio (`POST /businesses`).
- **BR-03.2** — Campos obligatorios al crear: `name`, `slug`, `type`. El `slug` es único en toda la plataforma (unívoco y URL-friendly).
- **BR-03.3** — `type` solo puede ser: `barbershop`, `salon` o `both`.
- **BR-03.4** — Límite de negocios por dueño: configurable en `platform_settings.max_businesses_per_owner` (valor semilla `5`). Superada → `422 BUSINESS_LIMIT_REACHED`.
- **BR-03.5** — Al crear un negocio se inicializan automáticamente:
  1. Rol por negocio `owner` con permisos `*`.
  2. 7 filas de horario semanal (todas cerradas: `is_open = false`).
  3. Configuración por defecto del negocio (ver §3).
- **BR-03.6** — El creador queda asignado como `owner_id` y con el rol `owner`.

## 2. Perfil público

- **BR-03.7** — Todo negocio tiene perfil público visible sin autenticación si `is_published = true`.
- **BR-03.8** — Los campos del perfil son:
  - `name` (obligatorio)
  - `slug` (obligatorio, único)
  - `description` (opcional, **formato Markdown**)
  - `type` (barbershop | salon | both)
  - `logo_url` (imagen en Supabase Storage)
  - `cover_photo_url` (imagen de portada)
  - `phone`, `email`, `address`
  - `latitude`, `longitude` (opcional, para mapas)
  - `timezone` (IANA, p. ej. `America/Mexico_City`); default de plataforma `UTC` (`platform_settings.default_timezone`)
- **BR-03.9** — El perfil público de un negocio **no publicado** devuelve `404 NOT_FOUND` (no expone datos internos).
- **BR-03.10** — `is_active = false` (negocio desactivado) → inaccesible para reservas: cola y agenda rechazan solicitudes con `423 BUSINESS_INACTIVE`.

## 3. Configuración del negocio (`business_config`)

Configuración clave-valor por negocio (extensible). Valores semilla:

| Clave | Valor por defecto | Regla |
|-------|-------------------|-------|
| `queue_enabled` | `true` | Habilita/deshabilita el modo cola |
| `agenda_enabled` | `true` | Habilita/deshabilita el modo agenda |
| `agenda_default_extra_fee` | `0` | Recargo base por defecto para citas de agenda (ver [08-agenda]). Se puede sobreescribir por servicio |
| `max_daily_queue_entries` | `null` | `null` = sin límite; número = cupo máximo de turnos de cola por día |
| `estimated_service_minutes` | `30` | Duración estimada por defecto para turnos de cola sin servicio específico |
| `queue_wait_buffer` | `1.2` | Factor de holgura del tiempo estimado de espera de cola (multiplica la suma de duraciones) |
| `appointment_reminder_hours` | `2` | Horas de anticipación para el recordatorio de cita |
| `business_color` | `#000000` | Color de marca |
| `slot_interval_minutes` | `15` | Granularidad de los slots de agenda |

- **BR-03.11** — Con `queue_enabled = false`, el endpoint de cola de ese negocio retorna `422 QUEUE_DISABLED` (mensaje: "La cola está deshabilitada en este negocio").
- **BR-03.12** — Con `agenda_enabled = false`, el endpoint de citas de agenda retorna `422 AGENDA_DISABLED`.
- **BR-03.13** — Si se cambia `max_daily_queue_entries` a un valor menor a la cantidad ya en cola, no se elimina nada: simplemente no se aceptan nuevos turnos hasta estar bajo el límite.
- **BR-03.14** — Cambiar `slot_interval_minutes` no afecta citas ya reservadas; solo el cálculo futuro.

## 4. Edición y eliminación

- **BR-03.15** — El dueño o quien tenga `manage_business_settings` puede editar el perfil (`PUT /businesses/:id`).
- **BR-03.16** — El Super Admin puede editar cualquier negocio; un dueño no puede editar negocios ajenos.
- **BR-03.17** — La baja de un negocio es **soft delete** (`is_active = false`). No se eliminan datos históricos (citas, empleados). Solo el Super Admin puede reactivarlo.
- **BR-03.18** — El `slug` no es editable si el negocio tiene citas/cola activas; en caso contrario es editable y debe seguir siendo único.

## 5. Dashboard y estadísticas

- **BR-03.19** — El dashboard del negocio (`GET /businesses/:id/dashboard`) contiene al menos: cola de hoy, citas de hoy y estadísticas básicas del día.
- **BR-03.20** — Solo empleados con roles/permisos del negocio acceden al dashboard. Un cliente no tiene acceso.

## 6. Límites y capacidad

- **BR-03.21** — El sistema impide reservar o ingresar a cola cuando el negocio está cerrado, sin cupo o desactivado — con mensajes específicos, nunca fallas en silencio.

## 7. Casos de prueba sugeridos

1. Crear negocio → se crean horario semanal (7 cerrados), rol `owner` y config por defecto.
2. Slug duplicado → `409 CONFLICT`.
3. Perfil no publicado → `404`.
4. Cupo excedido (`max_daily_queue_entries`) → `422 QUEUE_FULL`.
5. Negocio con `queue_enabled=false` intentando cola → error explícito.
6. Negocio inactivado intentando reserva → `423 BUSINESS_INACTIVE`.
7. Super Admin edita un negocio ajeno; dueño no → `403`.
8. Límite `max_businesses_per_owner` alcanzado → error con mensaje claro.