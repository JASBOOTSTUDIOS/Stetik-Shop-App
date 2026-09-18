# StetikShop — Modelo de Dominio y Glosario

## 1. Propósito

StetikShop es una plataforma multi-inquilino de gestión para **barberías y salones**. Permite a cada negocio:

- Tener un perfil público (logo, portada, nombre, descripción en Markdown, horarios).
- Gestionar empleados, servicios y horarios de atención.
- Atender clientes bajo **dos modalidades**: cola por orden de llegada y citas programadas.
- Controlar cupos, tiempos estimados y notificaciones.

## 2. Actores del sistema

| Actor | Descripción |
|-------|-------------|
| **Super Admin** | Administra toda la plataforma: negocios, usuarios, roles globales, ajustes. |
| **Dueño de negocio** | Crea y administra uno o varios negocios. Tiene control total sobre su negocio. |
| **Empleado** (barbero/estilista) | Presta servicios. Tiene su propia agenda y disponibilidad. |
| **Recepcionista / personal** | Roles configurados por el dueño con permisos parciales (ver agenda, marcar completado, etc.). |
| **Cliente** | Usuario que agenda citas o se une a la cola de un negocio. |
| **Visitante** | Usuario no autenticado que solo navega perfiles públicos de negocios. |

Un mismo usuario puede combinar roles: ser cliente en un negocio y empleado en otro, o dueño de varios negocios.

## 3. Entidades principales (glosario)

| Término | Definición |
|---------|------------|
| **Negocio** | Una barbería o salón registrado en la plataforma. Tiene perfil público, horarios, empleados y servicios propios. |
| **Empleado** | Relación entre un usuario y un negocio donde presta servicios. Tiene nombre visible, bio, foto y disponibilidad propia. |
| **Servicio** | Un trabajo que ofrece un negocio (ej. "Corte clásico"). Tiene precio, duración y modalidad (cola/agenda/ambas). |
| **Turno de cola** | Entrada de atención por orden de llegada dentro del horario del negocio. |
| **Cita / agenda** | Reserva programada a una hora específica con un empleado concreto. Puede tener un recargo configurable. |
| **Horario base** | Patrón semanal del negocio: qué días abre y a qué horas (7 filas, una por día de la semana). |
| **Excepción de horario** | Cambio puntual de horario en una fecha concreta (festivo, cierre temprano, apertura especial). |
| **Disponibilidad de empleado** | Patrón semanal de trabajo del empleado. |
| **Cupo** | Unidad de capacidad disponible en una franja horaria para agenda. |
| **Posición de cola** | Número ordinal de un cliente dentro de la cola del día. |
| **Tiempo estimado de espera** | Suma de duraciones de servicios pendientes por delante. |
| **Permiso** | Acción concreta que un rol habilita (ej. `manage_services`). |
| **Rol** | Conjunto de permisos asignable a usuarios, global o por negocio. |

## 4. Flujo principal (cliente)

```
Visitante
  └─ Registro/Login
        ├─ (Cliente) Navega negocios → ve perfil público
        │     ├─ Une a la COLA (orden de llegada) ──────────────┐
        │     └─ Agenda una CITA (fecha/hora/empleado/servicio)  │
        ├─ Recibe confirmación y recordatorio por email          │
        └─ Sigue su posición/estado en tiempo real ──────────────┘
              → Es atendido → Servicio completado
```

## 5. Flujo principal (negocio)

```
Dueño crea negocio
  ├─ Configura perfil (logo, portada, descripción .md, teléfono, dirección)
  ├─ Configura hora base semanal + excepciones (festivos, cierres)
  ├─ Registra empleados (cuentas creadas por el dueño)
  ├─ Configura disponibilidad y servicios por empleado
  ├─ Define servicios (precio, duración, modalidad, recargo de agenda, límite diario)
  ├─ Configuración global del negocio (cola activa, agenda activa, cantidad de turnos diarios)
  └─ Operación diaria:
        VISTA DE COLA: llama al siguiente, salta, marca atendido
        VISTA DE AGENDA: gestiona citas del día/semana/mes
        ANALÍTICAS: ingresos, servicios populares, horas pico, espera promedio
```

## 6. Distinción clave: Cola vs. Agenda

| Aspecto | COLA | AGENDA |
|---------|------|--------|
| Orden de atención | Orden de llegada (primero en llegar, primero en ser atendido) | En la hora reservada |
| Empleado asignado | El que esté disponible (o uno específico si el cliente lo elige) | Uno fijo elegido por el cliente |
| Hora | Sin hora fija; se estima por duraciones acumuladas | Hora exacta reservada |
| Precio | Precio base del servicio | Precio base + recargo de agenda (configurable) |
| Cupos | Límite opcional diario (`max_daily_queue_entries`) | Slots calculados; límite por servicio (`max_daily_slots`) |
| Requisito | Negocio abierto + servicio activo + cola habilitada + hay cupo | Negocio abierto + empleado disponible + slot libre + servicio agenda |
| Estado inicial | `pending` | `confirmed` |

Un servicio puede operar en cualquiera de las dos modalidades (campo `service_type`: `queue`, `agenda`, `both`).

## 7. Reglas transversales

- **BR-01.1** — Todos los sellos de tiempo se almacenan en UTC (`timestamptz`). La conversión a la zona horaria del negocio solo ocurre al mostrar o calcular horarios del negocio.
- **BR-01.2** — La moneda no tiene tipo cambiario: el precio se guarda como número decimal (2 dígitos), tal cual lo registra el negocio.
- **BR-01.3** — Las eliminaciones de entidades usan *soft delete* (bandera `is_active = false`) salvo que esta documentación indique lo contrario.
- **BR-01.4** — Ninguna operación del sistema puede terminar en silencio: toda falla conocida retorna un error estructurado (ver [10-errores.md](./10-errores.md)).
- **BR-01.5** — La descripción de un negocio se escribe en Markdown y se renderiza con `react-markdown`; se valida longitud máxima (sin límites de tamaño especificados por negocio, ver schema).

## 8. Supuestos de negocio

- No hay pagos en línea: el cobro es presencial. La app solo calcula `total_price` estimado.
- La cola es por *día del negocio*; un cliente que ingresa a cola pertenece al día actual en la zona horaria del negocio.
- El super admin crea los roles globales; el dueño crea los roles de su negocio. El rol `owner` y `super_admin` son de sistema (no editables ni eliminables).
- El dueño crea las cuentas de sus empleados (no los empleados se auto-registran dentro de un negocio).