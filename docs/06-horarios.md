# StetikShop — Reglas de Horarios y Cálculo de Cupos

> Documento núcleo del cálculo de disponibilidad. La complejidad principal está en la intersección de horarios del negocio con disponibilidad de empleados y cupos de agenda.

## 1. Horario base del negocio (`business_schedules`)

- **BR-06.1** — Cada negocio tiene un patrón **semanal** de 7 filas (una por día de la semana, `0`=domingo … `6`=sábado). Cada día: `is_open`, `open_time`, `close_time`.
- **BR-06.2** — Día cerrado: `is_open = false` y `open_time/close_time` deben ser `NULL`.
- **BR-06.3** — Día abierto: `is_open = true` y ambos horarios obligatorios.
- **BR-06.4** — **Cruce de medianoche:** se permite que `close_time < open_time` para significar que el negocio cruza la medianoche. Ej. viernes 22:00 → sábado 02:00 se modela como viernes `is_open=true, open=22:00, close=02:00`.
- **BR-06.5** — El horario base define el día "abierto" por defecto. Cuando cruza medianoche, las horas de la madrugada pertenecen al turno del día anterior.
- **BR-06.6** — Ejemplo solicitado por el cliente: lunes–viernes 05:00–20:00; sábado 07:00–24:00 (o 00:00) se registra como 5 días con `05:00–20:00` y sábado `07:00–00:00`/`23:59` según se defina el cierre.

## 2. Excepciones de horario del negocio (`business_schedule_overrides`)

- **BR-06.7** — Una excepción es puntual por **fecha** (`override_date`): puede abrir, cerrar antes, cerrar del todo, o abrir un feriado con horario diferente. Tiene `is_open`, `open_time`, `close_time` y `reason` opcional.
- **BR-06.8** — Para una fecha dada, la excepción **gana sobre** el patrón semanal. Si no hay excepción, se usa el patrón semanal.
- **BR-06.9** — Solo una excepción por fecha y por negocio (restricción única `(business_id, override_date)`).
- **BR-06.10** — El dueño/`manage_business_schedule` crea, edita y elimina excepciones. Ejemplos de uso: "cierro temprano el domingo a las 15:00", "abro el 25/12", "cerrado el 1/01".

## 3. Disponibilidad semanal del empleado (`employee_availability`)

- **BR-06.11** — Análoga al horario del negocio pero por empleado: 7 filas con `is_available`, `start_time`, `end_time`, con las mismas reglas de cerrado/abierto y cruce de medianoche.
- **BR-06.12** — Excepciones por fecha por empleado (`employee_availability_overrides`): gana sobre el patrón semanal del empleado.
- **BR-06.13** — Un empleado puede elegir no trabajar un día aunque el negocio abra; y a la inversa, puede tener disponibilidad en un día que el negocio cierra (pero no será efectivo porque el negocio no atiende).

## 4. Cálculo del horario efectivo de atención (un día cualquiera)

Para saber si el negocio puede atender en una fecha y franja:

```
horario_negocio(día):
   1. ¿Hay excepción del negocio para esa fecha? → usarla
   2. Si no → usar patrón semanal (business_schedules)

disponibilidad_empleado(empleado, día):
   1. ¿Hay excepción del empleado para esa fecha? → usarla
   2. Si no → usar patrón semanal (employee_availability)

franja_efectiva(empleado, día):
   intersectar(horario_negocio, disponibilidad_empleado)
   inicio_efectivo = max(inicio_negocio, inicio_empleado)
   fin_efectivo    = min(fin_negocio,   fin_empleado)
   si inicio_efectivo >= fin_efectivo → sin franja ese día
```

- **BR-06.14** — El negocio debe estar abierto, la fecha no puede estar en el pasado, y el día debe tener franja efectiva no vacía para poder operar.

## 5. Cálculo de slots disponibles para agenda (endpoint `GET /businesses/:id/slots`)

Inputs: `date`, `service_id`, `employee_id` (opcional).

```
getAvailableSlots(businessId, date, serviceId, employeeId?):
  1. horario del negocio para 'date' (excepción > semanal)
     Si cerrado → []
  2. empleados candidatos:
       employeeId dado → ese solo (validar que ofrezca el servicio y esté activo)
       null → todos los empleados activos que ofrezcan el servicio
  3. Por cada empleado:
     a. disponibilidad del empleado para 'date' (excepción > semanal)
     b. franja_efectiva = intersección con horario del negocio
     c. si franja vacía → saltar empleado
     d. citas existentes del empleado en 'date' con status
        IN ('pending','confirmed','in_progress')
     e. interval = business_config.slot_interval_minutes (def. 15)
     f. generar slots desde inicio_efectivo, cada uno de
        duration_minutes del servicio, avanzando 'interval'
     g. eliminar slots que SOLAPAN citas existentes:
          slot_inicio < cita_fin  Y  slot_fin > cita_inicio
     h. límite de cupo diario del servicio (max_daily_slots):
          si citas de ese servicio en la fecha >= limite → no ofrecer más
  4. ordenar por hora → [{time, employee_id, employee_name}]
```

- **BR-06.15** — Se retorna **todos los slots libres**; el cliente elige. Un slot se considera ocupado si se solapa con cualquier cita existente (estado activo) — regla de no doble reserva.
- **BR-06.16** — El slot arranca en franja efectiva; la hora exacta respeta `slot_interval_minutes` (15 por defecto).
- **BR-06.17** — Si el servicio tiene `max_daily_slots`, una vez alcanzado el cupo ese servicio no muestra más slots ese día (y la reserva adicional se rechaza).
- **BR-06.18** — Solo se evalúan empleados con `is_active = true` y que tengan asignado el servicio.

## 6. Validación de una reserva de agenda

Antes de confirmar una cita se revalida (nunca se confía solo en lo que el cliente vio):

- **BR-06.19** — La fecha/hora solicitada debe:
  1. Estar en el futuro.
  2. Caer dentro del horario efectivo del negocio (con excepciones).
  3. Caer dentro de la franja efectiva del empleado.
  4. No solaparse con otra cita del empleado.
  5. Cumplir cupo diario del servicio (`max_daily_slots`) si aplica.
  6. Cumplir `agenda_enabled` del negocio.
- Cualquier fallo → código de error explícito de [10-errores.md] (`SLOT_UNAVAILABLE`, `INVALID_STATUS_TRANSITION`, etc.).

## 7. Horizonte de reservas

- **BR-06.20** — La agenda solo permite reservar **en días futuros** (no se agenda ayer ni hoy en el pasado). Para hoy: solo franjas que aún no hayan ocurrido.
- **BR-06.21** — Se permite reservar para hoy mientras la hora solicitada sea futura y queden cupos.

## 8. Zonas horarias

- **BR-06.22** — Todo se guarda en UTC; los cálculos de "hoy", "día del negocio" y horarios se hacen en la zona horaria **del negocio** (`businesses.timezone`).
- **BR-06.23** — El sistema soporta negocios en cualquier IANA timezone; nunca se asume la zona del servidor/usuarios en los cálculos de disponibilidad.

## 9. Casos de prueba sugeridos (mínimos)

1. Día normal: negocio 09:00–18:00, empleado 08:00–16:00 → franja 09:00–16:00.
2. Día con excepción del negocio (cierre 14:00) → slots solo hasta 14:00.
3. Día con excepción del empleado (una hora, ej. 09:00–12:00) → solo esa franja.
4. Cruce de medianoche: negocio 22:00–02:00 → slots después de medianoche pertenecen al día del negocio correspondiente.
5. Empleado sin disponibilidad ese día → no aparece.
6. Doble reserva: slot solapado no se ofrece ni se acepta.
7. `max_daily_slots` alcanzado → sin más slots ese día para ese servicio.
8. Negocio cerrado un feriado → `[]` de slots.
9. Reserva para el pasado → `SLOT_UNAVAILABLE`/`VALIDATION_ERROR`.
10. Zona horaria: negocio en UTC-5 vs servidor en UTC → cálculo correcto del "hoy".