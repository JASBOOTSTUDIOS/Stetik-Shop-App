# StetikShop — Documentación de Reglas de Negocio

Documentación de referencia para la aplicación de barberías y salones **StetikShop**.

> **Stack:** React + TailwindCSS + TypeScript (client) · Express + TypeScript (server) · Supabase (Auth, PostgreSQL, Storage, Realtime) · Resend (email)

---

## Índice de documentación

> **Índice de proceso y plan:** `PLAN.md` (plan vivo) · `docs/CONTEXT.md` (contexto actual) · `docs/ERRORS.md` (errores comunes) · `docs/TECHNICAL_PLAN.md` (cómo construir el sistema).

| Archivo | Contenido |
|---------|-----------|
| [01-dominio.md](./01-dominio.md) | Modelo de dominio, glosario y flujos principales |
| [02-auth-roles.md](./02-auth-roles.md) | Autenticación, roles globales, roles por negocio y permisos |
| [03-negocios.md](./03-negocios.md) | Reglas de creación y gestión de negocios |
| [04-empleados.md](./04-empleados.md) | Reglas de empleados y su gestión |
| [05-servicios.md](./05-servicios.md) | Reglas de servicios ofrecidos |
| [06-horarios.md](./06-horarios.md) | Horarios del negocio, disponibilidad de empleados y cálculo de cupos |
| [07-cola.md](./07-cola.md) | Sistema de cola (turnos por orden de llegada) |
| [08-agenda.md](./08-agenda.md) | Sistema de agenda (citas programadas) |
| [09-notificaciones.md](./09-notificaciones.md) | Notificaciones in-app y por email |
| [10-errores.md](./10-errores.md) | Formato de errores y códigos (sin errores silenciosos) |
| [11-base-datos.md](./11-base-datos.md) | Esquema de base de datos con sus reglas |
| [12-api.md](./12-api.md) | Contrato de la API REST |
| [13-diseno.md](./13-diseno.md) | Sistema de diseño (línea gráfica oficial, Luxury Beauty Tech) |
| [TECHNICAL_PLAN.md](./TECHNICAL_PLAN.md) | Plan técnico: cómo construir (stack, fases, estructura, env) |

---

## Cómo leer esta documentación

- Cada archivo describe **qué debe cumplirse** en términos de negocio.
- Las **reglas** son obligatorias; se enumeran con numeración propia de cada archivo.
- Los **casos de prueba sugeridos** al final de cada archivo sirven de guía para validar el comportamiento.
- Esta documentación es la fuente de verdad para el desarrollo: si el código contradice una regla, la regla manda.

## Convenciones usadas

- **BR-X.Y** → Business Rule del archivo X, regla Y (ej. `BR-07.3` = regla 3 del sistema de cola; `BR-13.1` = paleta oficial de diseño).
- `time` → formato `HH:MM` de 24h.
- `timestamptz` → instante con zona horaria (siempre en UTC en BD).
- Slugs de permisos: minúsculas con `_` (ej. `manage_services`).

## Decisiones de arquitectura relevantes para las reglas

1. **Dos modos de atención:** cola (`queue`) y agenda (`agenda`). Un servicio puede soportar ambos.
2. **Multiinquilino:** un usuario puede ser dueño de varios negocios y a la vez empleado en otros.
3. **RBAC de dos capas:** roles *globales* (plataforma) y roles *por negocio* (dueño, barbero, recepcionista, etc.), ambos configurables.
4. **Todo CRUD pasa por Express**; Supabase Realtime se usa solo para suscripciones en vivo del cliente.
5. **Sin pagos online** — los montos se cobran en persona; la app solo calcula el total estimado.
6. **Sin errores silenciosos** — cada fallo retorna un código y mensaje; ver [10-errores.md](./10-errores.md).