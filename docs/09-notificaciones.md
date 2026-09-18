# StetikShop — Reglas de Notificaciones y Email

## 1. Modelo

- **BR-09.1** — Las notificaciones se crean **siempre** dentro de la plataforma (tabla `notifications`) para notificaciones in-app.
- **BR-09.2** — Toda notificación in-app puede ir acompañada de un **email** enviado por Resend. El envío es *asíncrono*: si falla, se registra en logs y el `email_sent` queda en `false` para reintento — **jamás** un error en silencio.
- **BR-09.3** — Campos: `user_id`, `business_id` (opcional), `type`, `title`, `message`, `data` (JSON con contexto: `appointment_id`, `queue_position`, etc.), `is_read`, `email_sent`, timestamps.

## 2. Tipos de notificación y disparadores

| Tipo | Destinatario | Disparador |
|------|--------------|------------|
| `booking_confirmed` | Cliente | Cita de agenda creada |
| `employee_new_booking` | Empleado | Cliente reserva con él |
| `queue_joined` | Cliente | Se une a la cola (con posición y espera) |
| `queue_update` | Cliente | Cambió su posición/espera u otra relevancia en la cola |
| `you_are_called` | Cliente | Lo llaman al puesto |
| `appointment_reminder` | Cliente | Cron ante la cita (ver [08-agenda.md] BR-08.26) |
| `appointment_cancelled` | Cliente + Empleado | Cita cancelada |
| `appointment_status_change` | Cliente/Empleado | Cambios de estado (inicio, completado, no-show) |

- **BR-09.4** — Cada evento de negocio relevante genera su notificación; el sistema no elimina silenciosamente ninguna.

## 3. Reglas de entrega

- **BR-09.5** — El cliente recibe las suyas; el empleado las suyas (solo las que le conciernen); el staff las del negocio si configurado.
- **BR-09.6** — El email se envía a la casilla del usuario registrado (`users.email`).
- **BR-09.7** — El recordatorio de cita se programa con `appointment_reminder_hours` del negocio y solo una vez (flag `email_sent`).
- **BR-09.8** — Los emails no son transaccionales para el negocio: un fallo de envío **no** revierte la operación (la cita se crea igual); el error queda en logs de monitoreo.

## 4. Centro de notificaciones (cliente/empleado)

- **BR-09.9** — `/my/notifications`: listar (filtro `is_read`), marcar leída una o todas, eliminar.
- **BR-09.10** — `/my/notifications/unread-count`: recuento no leído para el badge de la campana.
- **BR-09.11** — La campana se actualiza **en tiempo real** (Supabase Realtime: INSERT en `notifications` con `user_id = usuario`).

## 5. Plantillas y contenido

- **BR-09.12** — Plantillas: `booking_confirmation`, `appointment_reminder`, `queue_update`, `appointment_cancelled`, `you_are_called`, `employee_new_booking`.
- **BR-09.13** — Cada plantilla incluye datos contextuales (negocio, servicio, fecha/hora, posición, espera estimada, dirección/contacto).

## 6. Casos de prueba sugeridos

1. Reserva agenda → confirmación al cliente + aviso al empleado (in-app y email).
2. Unirse a cola → notificación con número y espera.
3. Llamar al siguiente → el cliente recibe "es tu turno" (in-app + email).
4. Recordatorio programado y enviado exactamente una vez.
5. Falla de email (key inválida) → la operación de negocio continúa; error en logs.
6. Marcado de leídas y badge de no leídos en tiempo real.