# StetikShop — Plan Técnico de Implementación

> Plan de construcción. Las **reglas de negocio** (comportamiento) viven en `docs/01..13` con numeración BR-X.Y y son la fuente de verdad del dominio. Aquí está el **cómo construir**: estructura, orden de fases, decisiones de stack y configuración.
> El plan **vivo** (progreso, turnos, pendientes) está en `PLAN.md`. El estado del proyecto está en `docs/CONTEXT.md`.

---

## 1. Resumen y decisiones de stack

| Decisión | Detalle |
|----------|---------|
| Repos | 2 repos independientes: `client/` (React SPA) y `server/` (Express API) en este monorepo-raíz |
| DB / Auth / Storage / Realtime | Supabase (PostgreSQL + Auth + Storage + Realtime) + **Prisma** (acceso a datos del server + migraciones de schema) |
| Capa de datos server | **Prisma** (`@prisma/client`) + Prisma Migrate (DDL del schema); RLS, triggers, realtime, storage y seed en SQL de Supabase (`supabase/migrations/`) |
| Validación | Zod compartido (paquete `shared/` o esquemas duplicados sincronizados por convención) |
| Estado front | Zustand (ui, auth, booking) + React Query (server state) + axios |
| Formularios | react-hook-form + @hookform/resolvers + zod |
| Realtime | Supabase Realtime (solo suscripciones en front; CRUD pasa por Express) |
| Email | Resend |
| Testing | Vitest + Supertest (server) · Vitest + Testing Library (client) |
| Sin pagos online | cobro presencial; la app solo calcula `total_price` |

### Rutas de trabajo
```
C:\src\StetikShop\
├── client/          React + Vite + Tailwind + TS
├── server/          Express + TS (API en /api/v1)
├── docs/            Reglas BR + CONTEXT + ERRORS
├── PLAN.md          Plan vivo
└── AGENTS.md        Reglas de proceso
```

---

## 2. Middleware del servidor (orden)

```
1. cors          (CORS_ORIGIN)
2. helmet        (cabeceras de seguridad)
3. morgan        (logs de request)
4. express.json({ limit: '10mb' })
5. multer        (solo rutas de upload)
6. authenticateToken    (verifica JWT Supabase, adjunta req.user)
7. requireRole(...)     (por ruta; chequea permisos globales o de negocio)
8. rutas → controllers → services
9. errorHandler  (último; formato estándar BR-10)
```

auth: extraer `Authorization: Bearer <jwt>` → verificar con `SUPABASE_JWT_SECRET` → comprobar usuario en `public.users` → fallo `401`.

requireRole: global = `user_roles`→`global_roles` (`*` = todos); negocio = `user_roles`→`business_roles` con `business_id` tomado de `req.params.businessId` o `req.body.business_id`. Fallo `403`.

---

## 3. Supabase (local y producción)

- Local: `supabase start` (Docker). Puertos: Postgres 54322, Studio 54323, API/Auth/Storage/Realtime en 54321.
- Migraciones **de schema/tablas** dentro del repo: `server/prisma/migrations/` (Prisma Migrate → SQL versionado y reproducible).
- Migraciones **de RLS, triggers, realtime, storage y seed** dentro del repo: `supabase/migrations/` (SQL nativo de Supabase).
- Cambio a producción: solo cambiar variables en `.env` (mismas migraciones, mismas tablas, mismos buckets).
- Realtime habilitado en tablas: `queue_entries`, `appointments`, `notifications`, `services`, `employees`, `business_schedules`.

**Canales Realtime:**
| Canal | Tabla | Eventos | Filtro | Consumidor |
|-------|-------|---------|--------|-----------|
| `queue:{business_id}` | `queue_entries` | UPDATE | `business_id=eq.<id>` | Vista cola staff + posición cliente |
| `appointments:{business_id}` | `appointments` | INSERT,UPDATE | `business_id=eq.<id>` | Dashboard staff |
| `my-queue:{user_id}` | `queue_entries` | UPDATE | client_id del usuario | Mi cola |
| `notifications:{user_id}` | `notifications` | INSERT | `user_id=eq.<id>` | Campana notificaciones |
| `business:{business_id}` | `services`,`employees` | INSERT,UPDATE,DELETE | `business_id=eq.<id>` | Páginas de gestión |

Al recibir evento → invalidar cache de React Query (`queryClient.invalidateQueries`); nunca devolver datos obsoletos como verdaderos (BR-10.12).

---

## 4. Storage (Supabase) — buckets y límites

| Bucket | Ruta | Lectura | Máx | Tipos |
|--------|------|---------|-----|-------|
| `business-images` | `{business_id}/logo/{uuid}.{ext}` | público | 2 MB | jpg/png/webp |
| `business-images` | `{business_id}/cover/{uuid}.{ext}` | público | 5 MB | jpg/png/webp |
| `business-images` | `{business_id}/service/{service_id}/{uuid}.{ext}` | público | 3 MB | jpg/png/webp |
| `employee-photos` | `{business_id}/{employee_id}/{uuid}.{ext}` | público | 2 MB | jpg/png/webp |
| `user-avatars` | `{user_id}/{uuid}.{ext}` | público | 2 MB | jpg/png/webp |

Escritura: siempre desde Express (las rutas `/upload/*` validan permiso y suben con multer → `storage.from(bucket).upload`). La URL pública se guarda en el campo `*_url` del registro.

---

## 5. Email (Resend)

- Plantillas: `booking_confirmation`, `appointment_reminder`, `queue_update`, `appointment_cancelled`, `you_are_called`, `employee_new_booking`.
- Recordatorio: job `node-cron` cada 15 min → citas agenda que inician en las próximas `appointment_reminder_hours` horas (config del negocio) → email + notificación in-app con flag `email_sent` (una sola vez; error → log + reintento; BR-08.27).
- Falla de email nunca revierte la operación de negocio (BR-09.8) → log estructurado.

---

## 6. Estructura de carpetas

### server/
```
server/src/
  index.ts  app.ts
  config/     database.ts supabase.ts env.ts cors.ts
  middleware/ authenticate.ts requireRole.ts validate.ts errorHandler.ts upload.ts rateLimiter.ts
  routes/     index.ts auth admin businesses employees services schedule appointments queue notifications upload stats
  controllers/  (mismos nombres + .controller.ts)
  services/     auth businesses employees services schedule appointments queue notifications upload stats email
  types/        index.ts express.d.ts api.types.ts roles.types.ts
  utils/        errors.ts logger.ts date.ts queue.ts pagination.ts
  validators/   auth business employee service schedule appointment (esquemas Zod)
```

### client/
```
client/src/
  main.tsx App.tsx index.css
  components/  ui/ layout/ business/ queue/ appointment/ services/ employees/ schedule/ roles/ booking/ notifications/ admin/
  pages/       auth/ public/ client/ booking/ business-dashboard/ admin/
  hooks/       useAuth useUser useRoles useBusiness useBusinesses useServices useEmployees useSchedule useAppointments useMyAppointments useQueue useMyQueue useNotifications useSlots useUpload useDebounce usePagination
  services/    api.ts y *.service.ts (axios)
  stores/      authStore uiStore bookingStore (zustand)
  types/ utils/ contexts/
```

---

## 7. Fases de implementación (resumen operativo)

> Detalle de comportamiento por dominio en `docs/` (columna). Cada fase termina solo con sus tests completos y actualizando PLAN.md/CONTEXT.md/ERRORS.md (AGENTS §1.2).

| Fase | Alcance | Docs NR | Test principal |
|------|---------|---------|----------------|
| **F0 Scaffolding** | Repos, Vite + Express, Tailwind, TS strict, Supabase local, middleware base, `/health`, formato respuesta/error | 10 | health + errorHandler |
| **F1 Schema BD** | Migraciones: 18 tablas, triggers (users↔auth, dueño→roles/horarios/config, empleado→disponibilidad/rol, precio), seeds, RLS+helpers, índices | 11, 02 | migración reproducible; RLS por rol |
| **F2 Auth** | signup/login/logout/refresh/me; JWT verification; store + rutas protegidas | 02 | auth e2e |
| **F3 RBAC** | requireRole, roles globales + por negocio CRUD, permiso matrix, admin user-roles | 02 | permisos por rol + `*` |
| **F4 Negocios** | CRUD negocio + init automático, config, dashboard, perfil público `.md`, slug | 03 | límite dueño, slug único, 404 no publicado |
| **F5 Empleados** | CRUD + init disponibilidad/rol, autogestión, excepciones | 04 | alta→init; baja suave; permisos |
| **F6 Servicios** | CRUD, `service_type`, precio+recargo, `employee_services`, `max_daily_slots` | 05 | oferta pública filtra |
| **F7 Horarios + cupos** | horario semanal, excepciones, disponibilidad, **algoritmo slots** | 06 | casos borde §9 |
| **F8 Cola** | join (posición), estimación, call-next/skip/complete, recálculo, transacciones, realtime | 07 | concurrencia §8 |
| **F9 Agenda** | booking (reserva + recargo), estados, cancelación, slots compartidos | 08 | transiciones + doble reserva |
| **F10 Notificaciones** | tabla + email, cron recordatorio, campana realtime | 09 | email one-shot; fallo no rompe |
| **F11 Imágenes** | multer → storage, buckets, hook upload, integraciones | (11 §Storage) | tamaño/tipo/permisos |
| **F12 Analíticas** | stats negocio + platform, charts | — | sumas exactas por período |
| **F13 Pulido responsive** | 240→16K, bottom bar móvil + pantalla Menú, áreas Usuario/Negocio/Admin, accesibilidad | AGENTS §5–6 | snapshots por breakpoint |
| **F14 Deploy** | Supabase prod, Railway, Vercel, CI, monitoreo | — | smoke e2e prod |

**Ruta crítica:** F1 → F2 → F7 → F8 → F9 → F10.

**Algoritmo de slots (F7):** detalle en `docs/06-horarios.md` §5. Resumen: excepción > semanal (negocio y empleado), intersección de franjas, restar citas vigentes solapadas, granularidad `slot_interval_minutes`, respetar `max_daily_slots`, en timezone del negocio.

**Algoritmo de cola (F8):** posición = `MAX(position del día)+1` en transacción con `SELECT ... FOR UPDATE`; espera = Σ duración de servicios pendientes por delante × buffer (`queue_wait_buffer`, default 1.2; BR-07.13); recalcular al llamar/saltar/cancelar; empujar por Realtime.

---

## 8. Variables de entorno

### server/.env
```env
SUPABASE_URL=http://localhost:54321
SUPABASE_ANON_KEY=eyJ...
SUPABASE_SERVICE_ROLE_KEY=eyJ...
SUPABASE_JWT_SECRET=super-secret-jwt-secret
PORT=3001
NODE_ENV=development
CORS_ORIGIN=http://localhost:5173
RESEND_API_KEY=re_...
EMAIL_FROM=noreply@stetikshop.com
DEFAULT_TIMEZONE=UTC
```

### client/.env
```env
VITE_API_URL=http://localhost:3001/api/v1
VITE_SUPABASE_URL=http://localhost:54321
VITE_SUPABASE_ANON_KEY=eyJ...
```

> Regla: **ninguna credencial en código**; `.env` fuera de git (AGENTS §4.6). Cambio a producción = solo cambiar valores.

---

## 9. Testing

- **server:** Vitest + Supertest. Unit: slots, posición de cola, precio, permisos, zod, fechas. Integration: auth, business CRUD, service CRUD, booking (doble reserva), queue (join/call/recalc), schedule.
- **client:** Vitest + RTL. Unit: formatters, permisos, hooks. Component: BookingStepper, ServiceSelector, TimeSlotPicker, QueuePosition.
- **E2E:** Playwright — flujos críticos: registro→crear negocio→empleado→servicio; cliente→cola; agenda→confirmación; call-next→realtime; completa.
- Toda feat se prueba al menos: feliz, negativo/validación, permisos, concurrencia (si aplica), todos los estados (AGENTS §3).
- Comandos: `npm test` en cada repo (escribir scripts `test`, `test:watch`, `typecheck`, `lint`).

---

## 10. Despliegue (referencia producción)

| Componente | Servicio |
|------------|----------|
| Client | Vercel / Netlify / Cloudflare Pages |
| Server | Railway / Render / Fly.io |
| Supabase | Cloud (mismas migraciones, RLS y buckets) |
| Email | Resend |

CI: push a `main` → client: build+deploy; server: build+tests (deben pasar) → deploy. Monitoreo: Sentry (errores), Supabase dashboard, Resend dashboard.

---

## 11. Checklist de turno (enlaza AGENTS §1.2)

1. Leer `PLAN.md`, `docs/CONTEXT.md`, `docs/ERRORS.md`.
2. Escribir alcance en `PLAN.md`.
3. Implementar siguiendo BR del dominio.
4. Tests completos del turno → correr y pasar.
5. Seguridad: sin errores silenciosos, formatos de error correctos, `requireRole` en qué corresponda, Zod en servidor, transacciones, sin secrets (AGENTS §4).
6. Actualizar `CONTEXT.md` + `ERRORS.md`.
7. Marcar avance en `PLAN.md`.