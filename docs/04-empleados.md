# StetikShop — Reglas de Empleados

## 1. Definición

- **BR-04.1** — Un **empleado** es la relación entre un usuario y un negocio donde presta servicios (ej. un barbero trabaja en la barbería X). Un usuario puede ser empleado en varios negocios.
- **BR-04.2** — Cada empleado tiene dentro del negocio: `display_name` (obligatorio), `bio`, `photo_url`, `is_active`, `hire_date`. El `display_name` puede diferir del nombre de su cuenta.

## 2. Alta de empleados

- **BR-04.3** — **El dueño (o quien tenga `manage_employees`) crea las cuentas/altas de los empleados**, no se auto-registran dentro de un negocio.
- **BR-04.4** — Al dar de alta un empleado se ejecutan automáticamente:
  1. Registrar `user_id` + negocio en la tabla `employees`.
  2. Asignar el rol por negocio `employee` (permisos mínimos: `view_services`, `view_business_schedule`).
  3. Crear 7 filas de disponibilidad semanal, todas `is_available = false` (el horario se configura después).
- **BR-04.5** — No se permite duplicar la relación (usuario, negocio): un usuario es empleado de un negocio solo una vez.
- **BR-04.6** — Para dar de alta a un usuario que aún no tiene cuenta, el dueño lo invita/crea cuenta; la relación se establece con el mismo `user_id` resultante.

## 3. Disponibilidad

- **BR-04.7** — Cada empleado tiene su **propia disponibilidad semanal** (`employee_availability`, 7 filas, una por día) **independiente del horario del negocio**.
- **BR-04.8** — La disponibilidad puede tener **excepciones por fecha** (`employee_availability_overrides`): campos `override_date`, `is_available`, `start_time`, `end_time`, `reason`. Ejemplo: "el 25/12 no trabajo" o "el 1/12 entro a las 10 en vez de las 9".
- **BR-04.9** — La prioridad de consulta para un día concreto es: excepción por fecha → si no existe, patrón semanal.
- **BR-04.10** — La efectividad de atención de un empleado en cualquier momento = intersección entre **disponibilidad del empleado** y **horario del negocio** (ver [06-horarios.md]).

## 4. Servicios por empleado

- **BR-04.11** — No todos los empleados ejecutan todos los servicios. La relación empleado → servicio es explícita (`employee_services`).
- **BR-04.12** — El dueño/`manage_services` asigna o revoca servicios por empleado. Un cliente no puede agendar con un empleado un servicio que ese empleado no ofrece.
- **BR-04.13** — Al reservar cola o agenda sin especificar empleado, solo se consideran empleados **activos** que ofrezcan el servicio seleccionado.

## 5. Autogestión del empleado

- **BR-04.14** — El empleado puede editar sus propios datos limitados: `display_name`, `bio`, `photo_url`, y su propia disponibilidad semanal/excepciones. **Por defecto puede (self-service)**; el dueño puede restringirlo quitándole `manage_employee_schedules` o vía rol, en cuyo caso solo el dueño / quien tenga `manage_employee_schedules` lo gestiona.
- **BR-04.15** — El empleado puede ver sus propias citas asignadas y su calendario; no puede ver citas de otros empleados a menos que tenga rol de staff con permisos (`view_appointments`).

## 6. Baja / desactivación

- **BR-04.16** — La baja es **soft delete** (`is_active = false`). El empleado deja de aparecer en selecciones públicas y de tomar nuevos turnos.
- **BR-04.17** — Las citas ya asignadas a un empleado desactivado **no se eliminan automáticamente**: quedan visibles para el negocio; el dueño decide reasignarlas o cancelarlas.
- **BR-04.18** — Solo el dueño (o rol con `manage_employees`) puede desactivar/reactivar empleados del negocio.

## 7. Reglas de exclusión

- El dueño es técnicamente propietario, no necesariamente registrado como empleado. Si quiere aparecer en selecciones debe darse de alta como empleado también.
- Un empleado con rol `owner` del mismo negocio tiene ambos papeles: datos de empleado (si se dio de alta) y permisos completos.

## 8. Casos de prueba sugeridos

1. Alta de empleado → rol `employee` asignado + 7 filas de disponibilidad creadas.
2. Alta duplicada (usuario + negocio) → `409 CONFLICT`.
3. Excepción por fecha gana sobre patrón semanal.
4. Cliente intenta agendar con empleado que no ofrece el servicio → `422 EMPLOYEE_UNAVAILABLE`.
5. Empleado desactivado no aparece en el selector público de empleados.
6. Empleado consulta citas de otro empleado sin permiso → `403`.
7. Citas existentes al desactivar empleado siguen visibles para el negocio.