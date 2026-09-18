# StetikShop — Reglas de Servicios

## 1. Definición

- **BR-05.1** — Un **servicio** es un trabajo ofrecido por un negocio (ej. "Corte clásico", "Barba", "Tinte"). Los servicios pertenecen a un negocio; cada negocio define los suyos.
- **BR-05.2** — Cada servicio puede operar en **cola**, **agenda** o **ambos** (campo `service_type` con valores `queue`, `agenda`, `both`).

## 2. Campos de un servicio

| Campo | Regla |
|-------|-------|
| `name` | Obligatorio. Unívoco dentro del negocio. |
| `description` | Opcional. |
| `price` | Obligatorio, mayor o igual a 0. Moneda: decimal con 2 dígitos. |
| `duration_minutes` | Obligatorio, mayor a 0. Define el tiempo que tarda el servicio y, por ende, la duración del turno en cola y del slot en agenda. |
| `service_type` | Obligatorio: `queue` / `agenda` / `both`. |
| `agenda_extra_fee` | Recargo extra que el cliente paga cuando el servicio se agenda por cita (por defecto `0`). Se suma al total de la cita. |
| `image_url` | Opcional (imagen en Storage). |
| `max_daily_slots` | Opcional; `null` = sin límite. Número = cupo máximo de citas de agenda por día para este servicio. |
| `is_active` | Baja suave. |
| `display_order` | Orden de visualización en UI. |

## 3. Reglas de operación

- **BR-05.3** — Solo pueden crear/editar/eliminar servicios: el dueño o quien tenga `manage_services`.
- **BR-05.4** — **Eliminación suave**: `is_active = false`. No se borran registros históricos (citas pasadas conservan su referencia).
- **BR-05.5** — Un servicio **no puede eliminarse/desactivarse** si tiene turnos de cola **pendientes** del día actual; primero deben atenderse o cancelarse. (Las citas de agenda futuras sí se pueden cancelar/reasignar antes de desactivar.)
- **BR-05.6** — Un servicio desactivado no aparece en la selección pública de servicios; tampoco se puede reservar.
- **BR-05.7** — Un servicio no asignado a ningún empleado activo **no aparece** en la oferta pública (la oferta = servicios que realmente se pueden prestar).
- **BR-05.8** — Asignación de servicios por empleado (`employee_services`): el dueño/`manage_services` controla qué empleado ejecuta qué servicio. Sin asignación, nadie puede prestarlo.

## 4. Precio total

- **BR-05.9** — En **cola**: `total_price = price` del servicio.
- **BR-05.10** — En **agenda**: `total_price = price + agenda_extra_fee` (el recargo puede ser 0). Este cálculo se aplica automáticamente al crear la cita (trigger de BD) y se muestra al cliente antes de confirmar.
- **BR-05.11** — El `agenda_extra_fee` del servicio puede anular/heredar el `agenda_default_extra_fee` del negocio: si el servicio define uno explícitamente, se usa el del servicio; si no, el del negocio.

## 5. Cupos y duración

- **BR-05.12** — La duración del servicio define:
  - En cola: la contribución al tiempo estimado de los que están detrás (ver [07-cola.md]).
  - En agenda: el tamaño del slot reservado y el cálculo de solapamientos (ver [08-agenda.md]).
- **BR-05.13** — `max_daily_slots` limita cuántas citas de agenda se aceptan para ese servicio en un mismo día. Cuando el cupo se llena, el slot deja de ofrecerse (ver [06-horarios.md]).
- **BR-05.14** — El sistema no permite crear dos servicios con el mismo `name` dentro del mismo negocio (`409 CONFLICT`).

## 6. Determinación de la oferta para un cliente

Al navegar el perfil público de un negocio, el cliente ve solo servicios:
1. Activos (`is_active = true`)
2. Con al menos un empleado activo asignado
3. Compatibles con la modalidad abierta del negocio (`queue` requiere `queue_enabled`; `agenda` requiere `agenda_enabled`)
4. Con cupo disponible (cola: límite diario no alcanzado; agenda: `max_daily_slots` no alcanzado)

## 7. Casos de prueba sugeridos

1. Crear servicio `queue`, `agenda`, `both` → comportamiento correcto en cada modalidad.
2. Precio en agenda incluye `agenda_extra_fee`; en cola no.
3. Servicio sin empleado asignado → invisible en oferta pública.
4. Servicio desactivado con cola pendiente → rechaza la baja con `409 CONFLICT` y mensaje claro (ver BR-05.5).
5. Nombre duplicado en el mismo negocio → `409`.
6. `max_daily_slots` alcanzado → deja de ofrecerse el servicio para ese día.
7. Servicio `agenda` en negocio con `agenda_enabled=false` → no se ofrece o `422 AGENDA_DISABLED` al reservar.