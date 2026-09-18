# StetikShop — Reglas del Sistema de Cola

> Sistema de atención **por orden de llegada** (first-come, first-served) con posiciones y tiempos estimados en tiempo real.

## 1. Conceptos

- **Turno de cola** = cita de tipo `queue` + una fila en `queue_entries` (relación 1:1) con `queue_position`.
- La cola es **por día del negocio** (en la zona horaria del negocio). El día se determina al unirse.
- La cola funciona mientras el negocio esté abierto (o en sesión) según [06-horarios.md].

## 2. Unirse a la cola (`POST /businesses/:id/queue/join`)

Input: `{ service_id, employee_id?, notes? }`.

Validaciones previas (todas con error explícito):

- **BR-07.1** — El negocio debe estar activo (`→ 423 BUSINESS_INACTIVE` si no).
- **BR-07.2** — `queue_enabled` del negocio debe ser `true` (`422 QUEUE_DISABLED` si no).
- **BR-07.3** — El servicio debe estar activo y tener `service_type` en (`queue`, `both`).
- **BR-07.4** — El negocio debe estar **abierto en el instante actual** (horario efectivo del día según [06-horarios.md] §4): si está cerrado → `422 BUSINESS_CLOSED`. Para atender fuera de horario el dueño usa excepciones de horario (abrir antes/después un día concreto), no turnos a la fuerza.
- **BR-07.5** — Límite diario (`max_daily_queue_entries`): si no es `null` y la cantidad de turnos de hoy alcanzó el límite → `422 QUEUE_FULL`.
- **BR-07.6** — Si `employee_id` se envía: ese empleado debe estar activo, ofrecer el servicio y estar disponible en ese momento (`422 EMPLOYEE_UNAVAILABLE` si no). Si `null`: el turno entra a la cola **general**.
- **BR-07.7** — Un cliente no puede tener dos turnos de cola **pendientes** en el mismo negocio: si ya tiene uno activo se rechaza con `409 CONFLICT` (mensaje: "Ya tienes un turno en esta cola").
- **BR-07.8** — Se asigna el servicio elegido; la posición y tiempo estimado se calculan sobre el servicio.

Al unirse:

- **BR-07.9** — `queue_position = MAX(posiciones de la cola del día) + 1`. Concurrencia segura: transacción + `SELECT ... FOR UPDATE` para evitar posiciones repetidas.
- **BR-07.10** — Se crea una cita `appointment_type='queue'` con `status='pending'` y una entrada de cola con `queue_position`, `joined_at`, y `estimated_wait_minutes` calculado.
- **BR-07.11** — El cliente recibe notificación in-app + email con su número y la espera estimada.

## 3. Posición y tiempo estimado

- **BR-07.12** — La posición se ordena por `queue_position` ascendente (nunca por `created_at`, para no romper la secuencia tras saltos).
- **BR-07.13** — **Tiempo estimado de espera** = suma de `duration_minutes` de todos los servicios pendientes por delante, con un factor de holgura configurable por negocio (`business_config.queue_wait_buffer`, semilla `1.2`). Si no se conoce el servicio de un turno se usa `estimated_service_minutes` (por defecto 30).
  ```
  estimated_wait(posición actual) =
     Σ (duración de servicio de cada turno con menor posición y no atendido) × buffer
  ```
- **BR-07.14** — Los turnos atendidos (`in_progress`) y sus completados no cuentan para la espera de los demás.
- **BR-07.15** — Cuando un turno se atiende, salta o cancela, el sistema **recalcula la espera de todos los pendientes** y empuja la actualización en tiempo real.
- **BR-07.16** — La espera mostrada es siempre la **más reciente calculada**, nunca un valor pasado en silencio.

## 4. Operación de la cola (staff)

- **BR-07.17** — Ver la cola: quienes tengan `view_queue` (por negocio). El público (usuarios no autenticados) puede ver la cola del día de forma resumida (posición ocupada) si el negocio lo habilita.
- **BR-07.18** — **Llamar al siguiente** (`POST /businesses/:id/queue/call-next`): requiere `manage_queue`.
  1. Toma el turno con menor `queue_position` en `pending`.
  2. Cambia estado a `confirmed` (o `in_progress` según config del negocio).
  3. Registra `called_at`.
  4. Notifica al cliente (in-app + email: "¡Es tu turno!").
  5. Recalcula esperas de los restantes.
- **BR-07.19** — **Llamar a uno específico** (`/queue/:aid/call`): requiere `manage_queue`; permite atender fuera de orden (p. ej. una cita agenda que llegó temprano). El estado pasa a atendido y la cola de posiciones se recalcula.
- **BR-07.20** — **Saltar** (`/queue/:aid/skip`): requiere `manage_queue`. Marca el turno para volver a llamarlo después o cancela; al saltar se reordenan las posiciones.
- **BR-07.21** — **Marcar completado** (`/appointments/:aid/complete`): requiere `mark_appointments_complete`. El turno sale de la cola activa.
- **BR-07.22** — Toda operación de la cola ocurre en **transacción**; la actualización de posiciones es atómica (evita que dos staff salten al mismo tiempo y rompan el orden).

## 5. Cliente y su posición en tiempo real

- **BR-07.23** — El cliente ve su `queue_position` y `estimated_wait_minutes` actualizados **en vivo** vía Supabase Realtime (canal `queue:{business_id}`). Si la conexión cae, el cliente recarga vía API (`/my/queue-status`).
- **BR-07.24** — El cliente puede **abandonar la cola** (cancelar su turno `pending`). Se recalcula la cola.
- **BR-07.25** — Si el negocio cierra o el día cambia, los turnos `pending` restantes quedan marcados `expired` (cierre diario) — nunca se borran en silencio.

## 6. Concurrencia (regla crítica)

- **BR-07.26** — Toda la asignación de posiciones y recálculos se ejecuta bajo **transacciones con bloqueo de filas** (`SELECT ... FOR UPDATE`) para que dos clientes que se unan a la vez jamás obtengan la misma posición.
- **BR-07.27** — Las operaciones del staff (call-next, skip, complete) se serializan por negocio para evitar dobles llamadas a un mismo turno.

## 7. Estados de un turno de cola

```
pending ──► confirmed ──► in_progress ──► completed
   │            │              │
   └─► cancelled│              └─► expired (cierre del día)
   └─► no_show
```

- **BR-07.28** — Cualquier cambio de estado dispara notificación a las partes correspondientes (cliente y/o staff).

## 8. Casos de prueba sugeridos

1. Dos clientes se unen casi simultáneamente → posiciones 1 y 2, nunca repetidas.
2. Posición y espera correctas con servicios de distinta duración (30′ + 45′ + 15′).
3. Saltar un turno → posiciones recalculadas y esperas actualizadas (se emite evento real-time).
4. Llamar al siguiente → estado `confirmed`, `called_at` grabado, cliente notificado.
5. Límite diario alcanzado → `422 QUEUE_FULL`.
6. Cliente con turno pendiente intentando otro → `CONFLICT`.
7. Negocio `queue_enabled=false` → rechazo con mensaje claro.
8. Conexión real-time caída → el cliente puede refrescar manualmente.
9. Cierre del día → turnos restantes a `expired`.
10. Double-click en "llamar siguiente" → solo se llama una vez (transacción).