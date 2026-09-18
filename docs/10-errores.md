# StetikShop — Reglas de Manejo de Errores (Sin Errores Silenciosos)

> **Principio rector:** la aplicación **jamás** termina una operación en silencio. Toda falla conocida genera una respuesta estructurada con código y mensaje; toda falla desconocida se loguea y se responde de forma genérica pero siempre explícita.

## 1. Formato de respuesta exitosa

```json
{
  "success": true,
  "data": { "...": "..." },
  "meta": { "page": 1, "limit": 20, "total": 150, "totalPages": 8 }
}
```

## 2. Formato de error

```json
{
  "success": false,
  "error": {
    "code": "SLOT_UNAVAILABLE",
    "message": "The requested time slot is no longer available.",
    "details": {
      "requested_time": "2026-09-15T10:00:00Z",
      "conflicting_appointment_id": "abc-123"
    }
  }
}
```

- **BR-10.1** — Todo error tiene `code` + `message` + (opcional) `details`. Sin excepción.
- **BR-10.2** — Los mensajes son legibles para el usuario final; los `details` son técnicos.

## 3. Códigos de error canónicos

| Código | HTTP | Significado |
|--------|------|-------------|
| `VALIDATION_ERROR` | 400 | El body/query no pasó la validación (Zod). `details` lista los campos fallidos. |
| `AUTH_REQUIRED` | 401 | Falta token de autenticación. |
| `TOKEN_INVALID` | 401 | Token expirado o inválido. |
| `INVALID_CREDENTIALS` | 401 | Credenciales incorrectas en login (email o contraseña). |
| `EMAIL_NOT_CONFIRMED` | 401 | El usuario intenta iniciar sesión con un email aún no verificado (GoTrue `email_not_confirmed`). |
| `FORBIDDEN` | 403 | Permisos insuficientes para la acción. |
| `NOT_FOUND` | 404 | Recurso inexistente (incluye negocios no publicados y empleados/servicios en otra empresa). |
| `CONFLICT` | 409 | Conflicto de estado/unicidad: slug tomado, rol duplicado, ya hay un turno en la misma cola, calendario cerrado puntual. |
| `SLOT_UNAVAILABLE` | 422 | El slot solicitado ya no está disponible (doble reserva / horario ocupado). |
| `QUEUE_FULL` | 422 | Límite diario de turnos de cola alcanzado. |
| `BUSINESS_LIMIT_REACHED` | 422 | El usuario alcanzó `max_businesses_per_owner`. |
| `QUEUE_DISABLED` | 422 | El negocio tiene `queue_enabled = false`. |
| `AGENDA_DISABLED` | 422 | El negocio tiene `agenda_enabled = false`. |
| `BUSINESS_CLOSED` | 422 | Se intenta unirse a cola mientras el negocio está cerrado en ese instante. |
| `EMPLOYEE_UNAVAILABLE` | 422 | El empleado no ofrece ese servicio o no está disponible en esa franja. |
| `INVALID_STATUS_TRANSITION` | 409 | Cambio de estado de una cita no permitido (ej. `completed → cancelled`). |
| `BUSINESS_INACTIVE` | 423 | Negocio desactivado o sin operación. |
| `RATE_LIMITED` | 429 | Demasiadas solicitudes. |
| `INTERNAL_ERROR` | 500 | Error inesperado del servidor. |

## 3.1 Capa de datos (Prisma)

- **BR-10.15** — Los fallos de conexión o indisponibilidad de la BD (`P1001`, `P1002`, timeouts, conexiones rechazadas) se traducen a `503 DATABASE_UNAVAILABLE` con `message` en `es` orientado al cliente. Nunca se exponen detalles de infraestructura (`details` solo interno/log).
- **BR-10.16** — `$connect`/`$disconnect` se invocan de forma **lazy** (singleton) y registrada (pino): arranque→`connect` con log; apagado→`disconnect` idempotente (llamar sin estar conectado no lanza). Cualquier error real se propaga como `AppError` — **prohibido** tragar con `catch {}` o `$disconnect` no `await`'d.



## 4. Reglas de servidor

- **BR-10.3** — Todo controlador usa `asyncHandler` con el error pasando al `errorHandler` central; **no hay `catch` vacíos** que traguen errores.
- **BR-10.4** — Las clases `AppError` (y subclases `ValidationError`, `AuthenticationError`, `ForbiddenError`, `NotFoundError`, `ConflictError`, `UnprocessableError`) se usan para errores de negocio con su `code` correspondiente.
- **BR-10.5** — Las violaciones de restricciones de BD (`unique`, `foreign key`, `check`) se traducen a `ConflictError`/`ValidationError` entendibles; nunca llegan al cliente como errores brutos de Postgres.
- **BR-10.6** — Errores inesperados → se loguea pila completa (`logger.error`) y se responde `500 INTERNAL_ERROR` con mensaje genérico (nunca se exponen detalles internos).
- **BR-10.7** — La validación usa **Zod en ambos lados**: el cliente valida antes de enviar y el servidor revalida (la validación del cliente nunca es la única barrera).
- **BR-10.8** — Todo endpoint de escritura es **transaccional**: si una operación falla a mitad, la base queda consistente (sin citas huérfanas, sin posiciones duplicadas).

## 5. Reglas de cliente

- **BR-10.9** — Interceptor de Axios:
  - `401` → cerrar sesión local + redirigir a `/login`.
  - `403` → toast "Permisos insuficientes".
  - `422` → mostrar errores por campo en los formularios.
  - resto → toast "Algo falló. Inténtalo de nuevo" + log.
- **BR-10.10** — React Query: `onError` por mutación muestra el mensaje del servidor; reintentos: 1 para errores de red, 0 para errores 4xx.
- **BR-10.11** — Errores a nivel de página → `ErrorBoundary`; no se deja pantalla en blanco sin explicación.
- **BR-10.12** — Las operaciones de tiempo real (queue) tienen *fallback*: si la suscripción falla/cae, el cliente puede recargar desde la API y nunca muestra posiciones de cola obsoletas como verdaderas.

## 6. Logging y monitoreo

- **BR-10.13** — Se registra: `userId`, `path`, `method`, `statusCode`, `code`, `duration`, y stack de errores inesperados — en formato estructurado (pino), con auditoría en producción (Sentry).
- **BR-10.14** — Los fallos de procesos asíncronos (email, jobs) quedan en logs con contexto suficiente para diagnosticar sin "silencios".

## 7. Casos de prueba sugeridos

1. Validación fallida → `400 VALIDATION_ERROR` con campos.
2. Doble reserva → `409 CONFLICT` o `422 SLOT_UNAVAILABLE` según corresponda.
3. Cola llena → `422 QUEUE_FULL`.
4. Negocio inactivo → `423 BUSINESS_INACTIVE`.
5. Error inesperado simulado → `500` genérico + log completo sin leak de detalles.
6. Violación de unique → `409` amigable.
7. Fallo a mitad de transacción → sin citas huérfanas (test de integración).
8. 401 en cliente → logout automático.