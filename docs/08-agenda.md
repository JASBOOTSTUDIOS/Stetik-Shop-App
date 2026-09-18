# StetikShop — Reglas del Sistema de Agenda (Citas Programadas)

## 1. Definición

- **BR-08.1** — Una **cita de agenda** es una reserva programada (fecha + hora + servicio + empleado concreto) en `appointments` con `appointment_type='agenda'` y `status='confirmed'`.
- **BR-08.2** — A diferencia de la cola, la agenda **no** tiene entrada de cola y **sí** tiene hora exacta, empleado fijo y recargo opcional sobre el precio base.

## 2. Reservar una cita (`POST /businesses/:id/appointments`)

Input: `{ service_id, employee_id, scheduled_at, notes? }`.

Validación (revalidada en servidor, nunca solo en el cliente):

- **BR-08.3** — Negocio activo (`423 BUSINESS_INACTIVE` si no) y `agenda_enabled = true` (`422 AGENDA_DISABLED` si no).
- **BR-08.4** — Servicio activo con `service_type` en (`agenda`, `both`).
- **BR-08.5** — Empleado válido: activo, ofrece el servicio, y su `scheduled_at` cae en su disponibilidad (excepciones incluidas). Un empleado que no ofrezca el servicio o no esté disponible en esa franja → `422 EMPLOYEE_UNAVAILABLE`.
- **BR-08.6** — `scheduled_at` dentro de la franja efectiva de atención (negocio + empleado, ver [06-horarios.md]).
- **BR-08.7** — `scheduled_at` en el futuro (`400 VALIDATION_ERROR` si no).
- **BR-08.8** — Sin solapamiento con otra cita del empleado en esa franja (duración = `duration_minutes` del servicio) → `422 SLOT_UNAVAILABLE`.
- **BR-08.9** — Cupo diario del servicio respetado (`max_daily_slots`) → `422 SLOT_UNAVAILABLE`.
- **BR-08.10** — Cualquier fallo retorna el código de error estándar de [10-errores.md]: `SLOT_UNAVAILABLE`, `INVALID_STATUS_TRANSITION`, `VALIDATION_ERROR`, etc.

Efectos de una reserva exitosa:

- **BR-08.11** — `total_price = price + agenda_extra_fee` se calcula automáticamente (trigger de BD) y se muestra antes de confirmar.
- **BR-08.12** — Notificaciones: cliente (confirmación), empleado (nueva reserva). Email a ambos.
- **BR-08.13** — La cita queda `confirmed` desde el momento de la reserva (no requiere re-confirmación del negocio salvo configuración contraria por negocio).

## 3. Recargo de agenda

- **BR-08.14** — La agenda conlleva un **recargo extra configurable**:
  - Por defecto del negocio: `business_config.agenda_default_extra_fee`.
  - Por servicio: `services.agenda_extra_fee` (si está definido, gana sobre el default).
- **BR-08.15** — El recargo puede ser `0`. Se muestra al cliente siempre en el resumen, desglosado: `precio base + recargo = total`.

## 4. Ciclo de estados de una cita

```
pending ──► confirmed ──► in_progress ──► completed
   │            │               │
   │            └──► cancelled   └──► no_show
   └─► cancelled
```

Transiciones válidas (aplicadas en capa de negocio):

| Desde | Hacia | Quién |
|-------|-------|-------|
| `confirmed` | `in_progress` | Empleado asignado / owner (`start`) |
| `in_progress` | `completed` | Empleado / owner (`complete`) |
| `confirmed` | `no_show` | Staff (`no-show`) |
| `confirmed` o `pending` | `cancelled` | Cliente (propia/pendiente) o staff (`cancel`) |

- **BR-08.16** — Un cliente **solo puede cancelar** sus propias citas futuras (`confirmed`/`pending`). Un staff con `cancel_appointments` cancela cualquier cita.
- **BR-08.17** — Las transiciones no permitidas se rechazan con `409 INVALID_STATUS_TRANSITION` y mensaje claro (ej. pasar `completed → cancelled`).
- **BR-08.18** — Al cancelar se libera el slot (vuelve a ofrecerse); los motivos quedan en `cancel_reason`.
- **BR-08.19** — Al completarse se registra `completed_at`; al iniciarse, `started_at`.

## 5. Consulta de citas

- **BR-08.20** — El staff con `view_appointments` ve las citas del negocio con filtros: fecha, estado, tipo, empleado.
- **BR-08.21** — Cada empleado ve sus propias citas asignadas.
- **BR-08.22** — El cliente ve sus citas (todas o solo activas según filtro) vía `/my/appointments`.
- **BR-08.23** — Las citas **canceladas y completadas** quedan visibles en el historial (nunca desaparecen en silencio).

## 6. Cupos en agenda

- **BR-08.24** — Los cupos de agenda son los **slots libres** calculados por [06-horarios.md]: intersección de disponibilidad y horario, sin solapamiento con citas vigentes, y respetando `slot_interval_minutes` y el `max_daily_slots` del servicio.
- **BR-08.25** — Un negocio puede ofrecer agenda y cola a la vez: los slots de agenda y los turnos de cola **comparten la capacidad del empleado**. Una cita agenda ya reservada resta disponibilidad al empleado; los turnos de cola se estiman sobre lo que quede (esto se refleja en el cálculo de espera de [07-cola.md]).

## 7. Recordatorios

- **BR-08.26** — `appointment_reminder_hours` (por negocio, semilla 2 h) define con cuánta anticipación se envía el recordatorio por email + notificación in-app.
- **BR-08.27** — El recordatorio solo se envía una vez por cita (flag `email_sent`); al fallar se reintenta y se registra en logs (sin errores silenciosos).

## 8. Casos de prueba sugeridos

1. Reserva exitosa con envío de confirmación y cálculo correcto de total con recargo.
2. Doble reserva en el mismo slot → `422 SLOT_UNAVAILABLE`.
3. Cita ayer → `400 VALIDATION_ERROR`.
4. Empleado no disponible ese día → `422 EMPLOYEE_UNAVAILABLE`.
5. Cupo diario alcanzado → sin slots / `422 SLOT_UNAVAILABLE`.
6. Cliente cancela cita ajena → `403`.
7. Transición inválida `completed → cancelled` → `409 INVALID_STATUS_TRANSITION`.
8. Cancelación libera el slot (vuelve en `/slots`).
9. Recordatorio enviado una sola vez por cita.
10. Agenda y cola coexistente: la agenda resta capacidad a la estimación de la cola.