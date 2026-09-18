# StetikShop — CONTEXT.md (Contexto Actual)

> Contexto SIEMPRE actualizado del proyecto: progreso, decisiones, arquitectura y estado de fases.
> Se sincroniza al cierre de cada turno. El plan maestro vive en `PLAN.md`.

## 1. Resumen en 1 pantalla

- **Qué es:** plataforma multi-inquilino de gestión para barberías y salones. Permite a cada negocio gestionar empleados, servicios, horarios y clientes bajo dos modalidades: **cola** (orden de llegada) y **agenda** (citas programadas).
- **Stack:** React + Tailwind + TS (`client`) · Express + TS (`server`) · Supabase (Auth, Postgres, Storage, Realtime) · Resend (email).
- **Estado:** Documentación y reglas de negocio completas. F0, F1 (schema BD), F2 (Auth end-to-end), F3 (RBAC), F4 (Negocios), F5 (Empleados), F6 (Servicios) y Dashboard Operativo terminados y verificados. Navegación multi-entorno y línea gráfica alineadas. **Turno 18:** auditoría de cumplimiento de AGENTS.md completada (RBAC server, middleware auth, 0 alertas nativas, i18n 100% de la UI, sin datos inventados). Siguiente: **F7 Horarios/cupos**.

## 2. Decisiones clave (por qué hacemos lo que hacemos)

| Decisión | Razón |
|----------|-------|
| Monorepo en 2 repos (`client` + `server`) | Ciclos de deploy y build independientes |
| Supabase como BD, auth, storage y realtime | No inventar auth; RLS; realtime nativo postgres |
| **Prisma (server) = capa de acceso a datos + Prisma Migrate para DDL** (tablas/índices/constraints) | Tipado extremo a extremo y migraciones versionadas en SQL |
| **RLS, triggers, realtime, storage y seed siguen en SQL de Supabase** (`supabase/migrations/`) | No son expresables en `schema.prisma`; defensa en profundidad intacta |
| **Todo CRUD pasa por Express** (vía Prisma); Supabase solo realtime en el front | Control total de validación, permisos y auditoría |
| RBAC de dos capas (global + por negocio) | Un usuario puede ser dueño del A y barbero del B |
| `appointments` unificada con discriminador `type` | Queue y agenda comparten ~80% del esquema |
| Horario semanal (7 filas) + tabla de excepciones por fecha | Más limpio que 365 filas/año |
| JWT de Supabase verificado en Express (`SUPABASE_JWT_SECRET`) | Autorización en capa de negocio |
| Sin pagos online | Requisito del cliente; cobro presencial |
| Descripciones en Markdown | Formato rico sin editor WYSIWYG |
| Zustand + React Query + react-hook-form + Zod compartido | Stack moderno, menos boilerplate, validación en ambas capas |
| Resend para email | API sencilla, SDK TS, free tier |
| **i18n desde el inicio** (i18next, `es` por defecto) | Todo texto visible es traducible sin refactor; `es` es la fuente y el idioma de producto |
| Supabase local en puertos `54431+` | Evita colisión con otros proyectos Supabase locales presentes en la máquina |
| Config Kilo Code espejo de `.opencode/` (`kilo.jsonc` + `.kilo/`) | Kilo ya no hace fallback a config de `.opencode/`; replicar rules garantiza el mismo flujo de turno en Kilo CLI/VS Code/JetBrains. `AGENTS.md` sí se auto-descubre (estándar multi-herramienta). |
| Signup con email pendiente de verificar → NO guarda tokens (solo `user`); UI muestra pantalla "Revisa tu email" | BR-02.9 (email obligatorio verificado) sin obligar al usuario a reintentar; al verificar, login normal |
| `logout` nunca rechaza: la revocación remota falla en silencio local y SIEMPRE cierra la sesión local | El usuario nunca queda atascado por un fallo de red; la sesión local es la fuente de verdad |
| Mapeo de errores de Supabase Auth en `supabaseAuth.ts` (429→RATE_LIMITED, 403→FORBIDDEN, 401→AUTH_REQUIRED, duplicados→CONFLICT, weak→VALIDATION_ERROR, 5xx→INTERNAL_ERROR) | El cliente traduce por `error.code` (catálogo `docs/10-errores.md` §3); un solo formato de respuesta |
| La señal "email verificado" de GoTrue es `access_token` presente en signup + `user.email_confirmed_at`; el server normaliza el signup (sesión vs `{ requiresEmailVerification: true, user }`) y el cliente decide SOLO por esa llave (E-26) | GoTrue REST no expone `app_metadata.email_verified`; la normalización en el server evita que la UI dependa del shape de `supabase-js` |
| Bootstrap: `connectDb()` siempre ANTES de `listen` (`server/src/server.ts`); errores de arranque logueados + `process.exit(1)` | Sin BD inicializada, cualquier ruta Prisma respondía 503 DATABASE_UNAVAILABLE (E-27) |
| `GET /auth/me` devuelve 404 para usuarios sin fila en `users` (la sync GoTrue→tabla todavía no existe) | La sincronización de usuarios a `public.users` queda pendiente de implementar (triggers SQL Supabase F1 / F3 RBAC); esperado en la prueba en vivo |
| Sync de usuarios a `public.users` vía `syncUserToPublic` | Trigger SQL de Supabase NO se ejecuta cuando GoTrue inserta en `auth.users` vía HTTP signup (el trigger existe y está activo, pero GoTrue lo saltea). Solución: función `syncUserToPublic` en `src/services/auth.ts` que hace `prisma.users.upsert` después de cada signup exitoso. Los tests mockean `getPrisma` para no tocar BD real. |
| Endpoints de administración usan `requireRole('manage_all_businesses')` (nunca `'*'`) | El catálogo de permisos de `docs/02-auth-roles.md` §2 no define un permiso `'*'`; el comodín solo es válido como valor de permisos de rol en el middleware, no como nombre de permiso en `requireRole` (E-04) |
| `supabaseAuthRequest<T>` tipado con `SupabaseUserResponse` y `401 AUTH_REQUIRED` de Supabase → `TokenInvalidError` | El shape real de GoTrue `GET /user` es plano (`id`, `email`, `email_confirmed_at`, …), no el de `supabase-js`; un 401 de GoTrue significa token inválido/expirado, pero el resto de fallos (p. ej. `429 RATE_LIMITED`) se propaga como error real, sin ocultarlo |
| Componentes compartidos `ConfirmDialog` y `Toast` para confirmaciones y avisos | Prohibido `alert()`/`window.confirm()` (AGENTS §6); toda confirmación/aviso pasa por UI propia, traducida e integrada en la línea gráfica |
| Ninguna pantalla muestra métricas inventadas: `BusinessPublicScreens` lee la cola real vía `useQueueSocket`/`queueStore`, y sus botones navegan a las pantallas reales de cola/agenda | AGENTS §5 (estados válidos, sin datos falsos) y E-35: una vista nunca simula datos o acciones que el backend sí puede realizar |

## 3. Arquitectura en 1 pantalla

```
[ Cliente React (SPA) ] --HTTPS/JSON--> [ Express API /api/v1 ]
        │  ▲                                    │
        │  │ Supabase Realtime (solo lecturas   │ JWT → permisos (RBAC en código)
        │  └──────────── en vivo)               ▼
        │                    ┌──────────────────────────────────┐
        └──── Supabase ─────┤ Postgres + RLS · Auth · Storage   │
                            │ + Realtime (queue, appointments,  │
                            └──────────── notifications) ───────┘
        Email (Resend) servicio → notificaciones y recordatorios
```

## 4. Estado de fases

| Fase | Estado | Notas |
|------|--------|-------|
| Docs negocio BR 01–12 | ✅ | Fuente de verdad del dominio |
| TECH_PLAN (cómo construir) | ✅ | Fases, stack, estructura, env, realtime, storage |
| AGENTS.md (reglas) | ✅ | Proceso: PLAN.md, turnos, tests, seguridad |
| PLAN / CONTEXT / ERRORS | ✅ | Apuntalando el "nada sin huella" |
| Auditoría de coherencia | ✅ | Códigos de error completos, TBDs resueltos, BR sin contradicciones |
| F0 Scaffolding | ✅ | Repos creados, health+, errores estándar, Zod, tests, Supabase local 54431+, i18n es |
| F1 Schema BD | ✅ | Schema.prisma 18 modelos, migración 20260916000850_init, Prisma client generado, 20 tests schema. Theme JSON en businesses.theme para colores por negocio. |
| F2 Auth | ✅ | Supabase Auth ↔ Express JWT: mapeo de errores (401/403/409/429/400), authStore (signup/login/refresh/fetchMe/logout/401-handler), validate AuthScreen + gate en App (BR-02.4). Server tests 62 ✓, client tests 38 ✓. |
| F3 RBAC | ✅ | Seed con catálogo de permisos (6 globales + 15 de negocio) + rol `super_admin`. Middleware `requireRole` + helpers de permisos con comodín `*`. Rutas de gestión de roles (global y por negocio). Panel super admin. Sync usuarios vía `syncUserToPublic`. Tests: server 77 ✓, client 23 ✓. typecheck ✓, build ✓. |
| F4 Negocios | ✅ | Servicio `businesses.ts` y rutas CRUD: list, getById, getBySlug (público), create (inicializa owner+horarios+config en transacción atómica), update, softDelete. Tests completos 12/12 ✓ con ejecución secuencial (E-32). |
| Navegación y Entornos | ✅ | Navegación contextual y dinámica según rol/entorno (Cliente/Público, Empleado/Negocio, Super Admin). Acceso público inicial, BottomBar ultra-compacta de 4 botones sin duplicados en Menú, soporte anónimo para cola y personalización de colores por negocio (`--biz-primary`). Client tests 40/40 ✓. |
| F5 Empleados | ✅ | Backend F5 completo: alta empleado (auto-rol employee + 7 días disponibilidad), perfil, soft-delete y disponibilidad semanal. Tests 10/10 ✓. |
| F6 Servicios | ✅ | Backend F6 completo: creación servicios (cola/agenda/ambas), precios, duración, recargo por cita y asignación a empleados. Tests 10/10 ✓. |
| Dashboard Operativo | ✅ | Endpoint `GET /businesses/:id/dashboard` con métricas operativas y clientes en cola en vivo. |
| Auditoría de cumplimiento (AGENTS §3/§5/§6/§7/§8/§9) | ✅ | Turno 18: RBAC server corregido (`manage_all_businesses`, validación de rol previo en solicitudes), middleware auth tipado con el shape real de GoTrue, 21 alertas nativas reemplazadas por `ConfirmDialog`/`Toast`, i18n 100% de la UI (es+en), `catch` silenciosos eliminados, datos inventados de `BusinessPublicScreens` conectados a la cola real, tests UI nuevos. Server 99 ✓, client 46 ✓. |
| F7 Horarios/cupos | ⏳ | Siguiente fase: cálculo dinámico de slots para agenda y disponibilidad. |

| F8 Cola | ⏳ | Riesgo de concurrencia |
| F9 Agenda | ⏳ | |
| F10 Notifs | ⏳ | |
| F11 Imágenes | ⏳ | |
| F12 Analíticas | ⏳ | |
| F13 Pulido responsive | 🔄 | Bottom bar móvil + áreas separadas (header+bottom bar ya implementados, falta separación de áreas) |
| F14 Deploy | ⏳ | |

## 5. Riesgos y contramedidas

| Riesgo | Severidad | Contramedida |
|--------|-----------|--------------|
| Cálculo de cupos (F7) | Alta | Tests con casos borde (`06-horarios.md` §9) |
| Posición en cola bajo concurrencia (F8) | Alta | Transacciones + `SELECT FOR UPDATE`; tests de concurrencia |
| Realtime edge (F8) | Media | Fallback a refetch por API; reconnection |
| Zonas horarias (F7–F9) | Media | UTC en BD; conversión por timezone del negocio; tests UTC±X |
| Errores silenciosos | Alta | Regla obligatoria §2.2 de AGENTS.md; revisión de diff por turno |

## 6. Requisitos de negocio que el cliente pidió y quedan como pendientes HU (para F0+)

- [ ] Al menos **10 roles configurables por pantalla** (super admin edita nombres/permisos).
- [ ] **Multi-negocio** por usuario (dueño de varias y empleado en otras).
- [ ] **Agenda por empleado** independiente del negocio; **servicios configurables por empleado**.
- [ ] **Cola remota**: el cliente se une desde su celular y ve posición en vivo.
- [ ] Panel super admin completo (usuarios, negocios, roles, permisos, settings, stats).
- [ ] **Recargo extra configurable** en turnos de agenda.
- [ ] **Supabase local para pruebas**; cambio a prod solo ajustando `.env`.
- [ ] **Imágenes en bucket de Supabase** (logo, portada, empleado, servicio, avatar).
- [ ] Excepciones de horario: cierre temprano, abrir antes un día, feriados. (**BR-06**)
- [ ] Horarios por día/semana/hora: L–V 05:00–20:00, Sá 07:00–00:00, etc. (**BR-06.6**)
- [ ] Responsive universal (240 px → 8K/16K) con bottom bar móvil + pantalla de Menú. (**AGENTS §5**)
- [ ] Áreas separadas Usuario / Negocio / Admin con cambio de rol explícito. (**AGENTS §6**)

## 7. TBD resueltos (decisiones de default tomadas)

Para que la implementación no bloquee, quedaron **resueltos** con default. Cualquier cambio de estos es decisión de producto, no del implementador:

| TBD | Decisión por defecto | Dónde se refleja |
|-----|----------------------|------------------|
| Roles globales iniciales | Seed `super_admin` (sistema), `support`, `viewer`. El resto (≥7 más) los crea el super admin en pantalla (requisito: ≥10 configurables en total) | `11-base-datos.md` §2 |
| Roles de negocio iniciales | Seed `owner` (sistema, `*`) y `employee` (mínimo). El dueño crea más en pantalla | `11-base-datos.md` §2 |
| Timezone por defecto | `UTC` (platform_settings `default_timezone`); el negocio lo sobreescribe | `03-negocios.md` BR-03.8 |
| Moneda por defecto | `USD` (platform_settings `default_currency`); sin pagos online, es solo etiqueta | `03-negocios.md`, `11-base-datos.md` |
| `max_daily_queue_entries` cuenta | Por "día del negocio" (timezone del negocio), igual que la cola | `07-cola.md` §1 |
| Buffer de espera de cola | `queue_wait_buffer = 1.2` (config por negocio) | `03-negocios.md` §3, `07-cola.md` BR-07.13 |
| Empleado edita su propia disponibilidad | Sí, por defecto; dueño puede restringirlo vía rol/permiso `manage_employee_schedules` | `04-empleados.md` BR-04.14 |
| Modo inmersivo UltraWide | Desactivado por defecto; contenido `max-w` centrado. Activable por pantalla | `AGENTS.md` §5.1 |
| Cola fuera de horario | NO permitida (`422 BUSINESS_CLOSED`); se modela con excepciones de horario | `07-cola.md` BR-07.4 |

## 8. Verificación de seguridad (última realizada)

| Turno | Resultado |
|-------|-----------|
| 0 (docs/reglas) | N/A — no hay código aún. Se aplicará desde F0. |
| 1 (auditoría de coherencia) | ✅ Sin código afectado. Corregidos: TECH_PLAN vacío, códigos de error ambiguos, contradicción BR-07.4 vs BR-03.21, TBDs sin default, typos. Ver `PLAN.md` turno 1. |
| 2 (F0 scaffolding) | ✅ Sin secrets en código (solo `.env`). Sin catch vacíos (msgs personalizados + pino por todas partes). Middlewares operan antes del `requireRole`. `express.json` limitado (10mb) + Helmet + CORS restringido + `x-powered-by` apagado. Errores de negocio = `AppError` con `code` (E-14 checks en `docs/10-errores.md` §3). Tests completos en `server/tests/` y `client/tests/`. |
| 2 (F0 scaffolding) | ✅ Verificado sobre el diff: sin `catch` vacíos/promesas sin manejo (msgs personalizados es + pino por todas partes); middlewares operan antes del `requireRole`; `express.json` limitado (10mb) + Helmet + CORS restringido + `x-powered-by` off; errores de negocio = `AppError` con `code` (E-14 checks en `ERRORS.md`); tests exhaustivos en `client/tests/` + `server/tests/`; `.env` fuera de git; Supabase local corriendo. |
| 2 (F0 scaffolding) | ✅ Sin secrets en código (solo `.env`). Sin catch vacíos. Errores con formato estándar + pino estructurado. Zod valida en server. TS `strict` en ambos repos. Tests en `server/tests/` y `client/tests/`. Flags de seguridad: Helmet + CORS + sin `x-powered-by`. |
| Config OpenCode | ✅ `.opencode/agent/build.md` con `steps` validado (entero, E-17) y posteriormente **eliminado** (`steps` sin límite explícito, se usa el default del esquema `AgentConfig`). `opencode` ejecuta sin errores de configuración. |
| F1 Schema BD | ✅ Schema.prisma 18 modelos aplicados. Migración 20260916000850_init ejecutada contra Supabase local. Tests schema: 20/20 ✓. Typecheck ✓. Lint ✓. Sin errores silenciosos. Sin secrets en código. Zod valida en server. TS strict en ambos repos. |
| F2 HomeScreen | ✅ | Pantalla principal creada con sistema gráfico oficial (negro #0B0B0B, crema #F8F8F8, oro #D4AF7C, Poppins). Logo integrado. Sin emojis (iconos SVG lineales). Responsive móvil/tablet/desktop/ultra. Colores del logo extraídos y aplicados como tokens Tailwind. Tests: 14/14 ✓. Typecheck ✓. Lint ✓. Build ✓. Sin `<main>` duplicado (App tiene el principal). |
| Resolución errores residuales (turno 7) | ✅ Se corrigieron: clave `nav` duplicada en `resources.ts` es, tagline `es.app.tagline` corregida a español, tipo `onScreenChange` en App.tsx (cast a `Screen`), `<main>` duplicado en HomeScreen (cambiado a `<div>`). Typecheck ✓, lint ✓, build ✓, tests client 14/14 ✓, server 62/62 ✓. Sin errores silenciosos. |
| Verificación seguridad (F1+F2+turno 7) | ✅ Sin `catch` vacíos ni promesas sin manejo. Sin errores silenciosos. Zod valida en server. TS `strict` en ambos repos. Tests exhaustivos: server 62 ✓, client 14 ✓. Sin secrets en código. `.env` fuera de git. Prisma schema sin XSS. Tailwind utilities sanitizadas. i18n sin claves duplicadas. |
| Config Kilo Code (turno 5) | ✅ Sin secrets (ninguna credencial en `kilo.jsonc` ni `.kilo/`). Permisos restringidos: `bash` solo allow-list (`npm run *`, `npx *`, `node *`, `git status/diff/log*`), resto `ask`; `edit: ask`; `external_directory: deny`; agent build con `git commit/push` y `rm -rf` en `deny`. Instrucciones: AGENTS.md auto-descubierto + `instructions` = PLAN.md + docs/CONTEXT.md + docs/ERRORS.md. `steps` sin límite explícito (E-17 respetado: si se define debe ser entero positivo). `kilo.jsonc` validado con `JSON.parse` ✓. Sin `references` (clave no soportada por Kilo, excluida del espejo). |
| F2 Auth (turno 7) | ✅ Sin `catch` vacíos ni promesas sin manejo. Errores de negocio con `code`+`message` (catálogo §3). Client traduce por `error.code` vía i18n. Validación Zod espejo cliente↔servidor (server siempre revalida). Auth gate solo UI: el enforcement sigue en Express (`authenticateToken`/`requireRole`) — defensa en profundidad intacta. `logout` falla en silencio local a propósito (decisión documentada) y SIEMPRE cierra sesión. Sin secrets. TS strict. Server 62 ✓ + typecheck ✓ + lint ✓; client 38 ✓ + typecheck ✓ + lint ✓ + build ✓. |
| Prueba en navegador (turno 9) | ✅ Entorno vivo: Supabase local (54431–54434), API `:3001`, client `:5173`. Verificado por HTTP: health 200, signup 201 (sesión), login 200, login inválido 401, email duplicado 409. Bugs reales detectados y corregidos: E-26 (señal email verificado → normalización en server) y E-27 (bootstrap sin `connectDb()` → `startServer`). `GET /auth/me` → 404 esperado (sin sync a `users` aún). Server 65 ✓ + client 38 ✓ + typecheck/lint ✓. |
| Alineación línea gráfica (turno 10) | ✅ Aplicación cliente alineada 100% a `lineagrafica.md` y `AGENTS.md` §2.2/§5. Tokens oficiales en CSS+Tailwind (oro #D4AF7C, negro #0B0B0B, ink #1F1F1F, gris #6B6B6B, crema #F8F8F8, Poppins). Branding unificado `BrandName.tsx`. Header premium (fondo negro, backdrop-blur, borde oro). Bottom bar móvil 6 ítems con iconos lineales, hitbox 56px, solo `md:hidden`. `LanguageSwitcher` con border radius consistente. `AuthScreen`: inputs premium, ErrorBanner con ícono+acción, color error funcional. `HomeScreen`: tagline premium, iconos QuickActions, hover effects. Placeholders alineados: `ServerStatus`, `MenuScreen`, `QueueScreen`, `AppointmentsScreen`, `NotificationsScreen`, `ProfileScreen`. typecheck ✓, tests 38/38 ✓. Sin errores silenciosos. Sin secrets. |
| Revisión navegación + bordes + SEO (turno 11) | ✅ Flujo de navegación verificado: `SCREEN_COMPONENTS` registra todos los 6 screens (home, queue, appointments, notifications, profile, menu); cada ítem de la bottom bar renderiza su componente. Bordes ajustados a `border-ink-700/60` (más delgado, gris claro) en App, BottomBar, HomeScreen, AuthScreen, LanguageSwitcher, ServerStatus y 5 placeholders. `rounded`/`rounded-lg` consistentes. SEO: `index.html` con título descriptivo, meta description/keywords/author/robots, Open Graph (og:title/description/type/locale), canonical, theme-color #0B0B0B, mobile-web-app-capable, apple-mobile-web-app-capable, favicon. typecheck ✓, build ✓ (79 módulos, dist/ 18 kB CSS + 285 kB JS), tests 38/38 ✓. Sin errores silenciosos. Sin secrets. |
| F4 Negocios (turno 13) | ✅ Sin catch vacíos ni promesas sin manejo. Zod valida en servidor (create, update, params). Control de permisos por rol (`manage_all_businesses`, dueño). Operaciones de inicialización en transacción atómica (`createBusiness` crea negocio, rol owner, 7 business_schedules y business_config). Sin secrets en código. Vitest configurado con `fileParallelism: false` para evitar colisiones en BD compartida (E-32). Server: 89/89 tests ✓, typecheck ✓, lint ✓, build ✓. Client: 38/38 tests ✓, typecheck ✓, lint ✓, build ✓. |
| Navegación congruente + Super Admin + IP local (turno 14) | ✅ Rol `super_admin` asignado en PostgreSQL a `astaciosanchezjefryagustin@gmail.com`. Navegación universal responsive completa: `DesktopNav` en tablet/desktop (≥md) y `BottomBar` en móvil (<md) con todos los destinos accesibles sin botones ocultos. `MenuScreen` con accesos a Negocio, Administración (si es admin), Ajustes y Soporte. Pantallas `AdminScreen` y `BusinessScreen` integradas. Soporte de IP en red local (`SERVER_IP`, `HOST` en server y `VITE_SERVER_IP` en client; `host: true` en Vite). Server: 89/89 tests ✓, typecheck ✓, lint ✓, build ✓. Client: 38/38 tests ✓, typecheck ✓, lint ✓, build ✓. |
| Auditoría de cumplimiento AGENTS.md (turno 18) | ✅ **§9 RBAC:** `requireRole('*')` → `manage_all_businesses` en `categories` y administración de `requests`; `requestStatusSchema` con `z.enum` + validación del rol previo al crear solicitud. **§3/§9 Auth:** `supabaseAuthRequest<SupabaseUserResponse>` con el shape plano real de GoTrue; `401` de Supabase → `TokenInvalidError`, el resto de fallos se propaga sin ocultarse. **§6:** 0 `alert()`/`window.confirm()` (sustituidos por `ConfirmDialog`/`Toast`); los `catch` que solo limpiaban estado ahora registran `console.error` + mensaje i18n. **§8:** sin textos hardcodeados en la UI (grep de nodos JSX y de emojis), claves nuevas en `es` y `en`. **§5:** `BusinessPublicScreens` sin métricas inventadas (cola real vía realtime) ni acciones muertas. **§7:** Server 99/99 ✓ (8 suites) + typecheck ✓ + lint ✓ + build ✓; Client 46/46 ✓ (6 suites, incl. `ui.test.tsx`) + typecheck ✓ + lint ✓ + build ✓. Sin secrets en el diff. |