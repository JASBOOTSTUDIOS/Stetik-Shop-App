# StetikShop — PLAN.md (Plan Vivo)

> Archivo de plan único. Toda sesión comienza aquí. Se actualiza durante y al final de cada turno.
> Contexto y errores: `docs/CONTEXT.md` y `docs/ERRORS.md`.

## Coordinación en Paralelo (Antigravity & OpenCode)

> **Regla de oro**: Antes de editar código, marca la tarea como `[EN CURSO por <Agente>]`. El otro agente no modificará esos archivos hasta que se complete y verifiquen los 4 gates (§10 de `AGENTS.md`).

| Agente | Terminal / Entorno | Tarea Actual | Módulo / Archivos | Estado |
|---|---|---|---|---|
| **Antigravity** | IDE Agent | Coordinación y verificación | `PLAN.md`, `AGENTS.md` | 🟢 Listo |
| **OpenCode** | PowerShell (PID 19552) | Por asignar | — | 🟢 Listo |

## Estado global

| Ítem | Estado |
|------|--------|
| Documentación de reglas de negocio (`docs/01..13`) | ✅ Hecho |
| Plan técnico (`docs/TECHNICAL_PLAN.md`) | ✅ Hecho |
| Reglas de trabajo (`AGENTS.md`) + config `.opencode/` | ✅ Hecho |
| Config Kilo Code (`kilo.jsonc` + `.kilo/` espejo) | ✅ Hecho |
| Línea gráfica oficial (BR-13 diseño) en rules de agentes | ✅ Hecho |
| `PLAN.md` / `CONTEXT.md` / `ERRORS.md` | ✅ Creados |
| Auditoría de coherencia (códigos, TBDs, contradicciones) | ✅ Hecho |
| F0 Scaffolding (server + client + Supabase local + i18n) | ✅ Hecho |
| Repos (client + server) | ✅ Hechos |
| Implementación | 🔄 F0–F6 hechas; F5 Empleados COMPLETO; F6 Servicios COMPLETO; Dashboard COMPLETO; auditoría de cumplimiento de reglas COMPLETA; siguiente F7 |

## Turno actual

- **Fecha:** 2026-09-17
- **Alcance (turno 19):** Verificación de configuración de Prisma v7 en producción, ejecución exitosa de migraciones a Supabase Prod, seed de permisos/roles globales, adición de scripts de migración/seed en `package.json`, documentación en `RENDER_DEPLOY.md` y sincronización con OpenCode.
  1. **Prisma v7 Direct/Pooler URL:** `prisma.config.ts` actualizado para preferir `env("DIRECT_URL")` (puerto 5432, session mode) en migraciones y `env("DATABASE_URL")` (puerto 6543, transaction mode) en runtime con `sslmode=no-verify`.
  2. **Migración Producción:** `npx prisma migrate deploy` ejecutado con éxito contra la base de datos Supabase de producción; 18 modelos y tablas creadas.
  3. **Seed Producción:** `npx tsx prisma/seed.ts` ejecutado en producción; catálogo de permisos globales/de negocio y rol `super_admin` creados.
  4. **Scripts server:** agregados `"db:migrate:deploy"` y `"db:seed"` a `package.json`.
  5. **Render:** actualización de `RENDER_DEPLOY.md` con `--include=dev`, `prisma migrate deploy` automático y tabla de variables de entorno (`DIRECT_URL`).
  6. **Render Entrypoint Fix:** corregido el path de arranque a `dist/src/index.js` con generador automático de puente `dist/index.js` en el script `build` para eliminar cualquier fallo de módulo no encontrado.
- **Próximo:** F7 Horarios (negocio + empleados + excepciones + cálculo dinámico de slots para agenda).


## Registro de turnos (historial)

| Turno | Fecha | Alcance | Estado |
|-------|-------|---------|--------|
| 0 | 2026-09-15 | Bases del proyecto (docs, reglas, contexto) | ✅ |
| 1 | 2026-09-15 | Auditoría de coherencia: TECH_PLAN, códigos de error, TBDs, contradicciones | ✅ |
| 2 | 2026-09-15 | F0 scaffolding: server + client + Supabase local + i18n es + tests | ✅ |
| 3 | 2026-09-15 | Prisma v7 en server: schema, cliente singleton + adapter-pg, tests | ✅ |
| 4 | 2026-09-16 | F1 Schema BD: 18 modelos, migración, tests schema, typecheck, lint | ✅ |
| 5 | 2026-09-16 | Config Kilo Code: `kilo.jsonc` + `.kilo/agent/` + `.kilo/skills/` (espejo de `.opencode/`) | ✅ |
| 6 | 2026-09-16 | Línea gráfica oficial BR-13 en rules + HomeScreen + tokens Tailwind + tests | ✅ |
| 7 | 2026-09-16 | Resolver errores residuales: duplicate key i18n, tagline es, tipos App.tsx, <main> duplicado. Typecheck/build/lint/tests ✓ | ✅ |
| 8 | 2026-09-16 | F2 Auth end-to-end: mapeo errores Supabase, authStore, AuthScreen + gate, i18n auth.*, tests (server 62 ✓, client 38 ✓) | ✅ |
| 9 | 2026-09-16 | Prueba en navegador (F2): entorno vivo Supabase 54431–54434 + API :3001 + client :5173; verificación HTTP (health/signup/login/errores 401/409) + navegador abierto. Fixes E-26 (normalización signup en server) y E-27 (bootstrap `startServer` con `connectDb()`). Server 65 ✓, client 38 ✓ | ✅ |
| 10 | 2026-09-16 | Alineación de línea gráfica oficial en cliente: i18n E-25, tokens visuales, branding, navegación móvil, auth y placeholders alineados. | ✅ |
| 11 | 2026-09-16 | Revisión de navegación, bordes más delgados/grises claro, y SEO. typecheck + build + tests OK. | ✅ |
| 12 | 2026-09-16 | F3 RBAC: seed, middleware requireRole, rutas de roles, panel super admin, sync usuarios. Tests 77 ✓ server + 23 ✓ client. | ✅ |
| 13 | 2026-09-16 | F4 Gestión de negocios: servicio, CRUD completo, validación slug único, límites dueño, tests secuenciales E-32, lint/typecheck/build al 100%. Tests 89 ✓ server + 38 ✓ client. | ✅ |
| 14 | 2026-09-16 | Navegación congruente universal (DesktopNav + BottomBar + MenuScreen + AdminScreen/BusinessScreen) y asignación rol super_admin a astaciosanchezjefryagustin@gmail.com. Tests 89 ✓ server + 38 ✓ client. | ✅ |
| 15–16 | 2026-09-16 | **Sin registro en el historial** (gap de registro, ver E-33); trabajo reflejado en `docs/CONTEXT.md`. | ⚠️ |
| 17 | 2026-09-16 | Flujo completo de Gestión de Negocio: F5 Empleados, F6 Servicios, Dashboard Operativo y BusinessScreen. Server 99 ✓, client 40 ✓. | ✅ |
| 18 | 2026-09-17 | Auditoría de cumplimiento de AGENTS.md (§3/§5/§6/§7/§8/§9): RBAC server, middleware auth, 21 alertas nativas → ConfirmDialog/Toast, i18n 100% de la UI, cola real en BusinessPublicScreens, tests UI, docs de errores. Server 99 ✓, client 46 ✓. | ✅ |
| 19 | 2026-09-17 | Prisma v7 producción: soporte DIRECT_URL, migración a Supabase Prod (18 tablas), seed de permisos/roles, doc Render. | ✅ |

## Fases de implementación (plan maestro)

| Fase | Descripción | Días | Estado |
|------|-------------|------|--------|
| 0 | Scaffolding: repos `client` + `server`, Supabase local, middleware base | 1–2 | ✅ |
| 1 | Schema de BD + triggers + seeds + RLS + índices | 3–5 | ✅ |
| 2 | Autenticación (Supabase Auth → Express JWT) | 6–7 | ✅ |
| 3 | RBAC: roles globales y por negocio, panel super admin | 8–10 | ✅ |
| 4 | Gestión de negocios (crud, config, dashboard, perfil público) | 11–13 | ✅ |
| 5 | Empleados (alta, disponibilidad, autogestión) | 14–16 | ✅ |
| 6 | Servicios + asignación a empleados | 17–18 | ✅ |
| 7 | Horarios (negocio + empleados + excepciones + cálculo de slots) | 19–22 | ⏳ |
| 8 | Cola / turnos (posición, estimación, realtime, staff) | 23–26 | ⏳ |
| 9 | Agenda / citas (booking flow, gestión staff, historial) | 27–29 | ⏳ |
| 10 | Notificaciones + recordatorios email | 30–31 | ⏳ |
| 11 | Imágenes / Storage | 32–33 | ⏳ |
| 12 | Analíticas y dashboards | 34–35 | ⏳ |
| 13 | Pulido: responsive, estados, accesibilidad | 36–40 | ⏳ |
| 14 | Deploy y verificación producción | 41–45 | ⏳ |

**Ruta crítica:** F1 → F2 → F7 → F8 → F9 → F10.

## Tests pendientes acumulados (se completa por turno)

- [x] F0: health-check API, middleware de error, validación Zod básica.
- [x] F1: schema 18 modelos, migración reproducible, Prisma v7 client, tests schema (20 ✓).
- [x] F2: auth end-to-end (signup/login/refresh, rutas protegidas).
- [x] F3: roles configurables ≥10; asignación; `requireRole` por permiso.
- [x] F4: negocio CRUD, slug único, límite por dueño, perfil público vs no publicado.
- [x] F5: alta empleado (init disponibilidad+rol), baja suave, excepciones.
- [x] F6: servicios (modalidades, precio+recargo, cupos diarios).
- [ ] F7: cálculo de slots — casos borde `docs/06-horarios.md` §9.
- [ ] F8: cola — casos borde `docs/07-cola.md` §8 (concurrencia incluida).
- [ ] F9: agenda — transiciones de estado, doble reserva, cancelación libera slot.
- [ ] F10: notificaciones + email (fallo de email no rompe el negocio).
- [ ] F11: uploads (tamaño/tipo/permisos/storage).
- [ ] F12: estadísticas exactas por período.
- [ ] F13: responsive móvil/tablet/desktop/ultra-ancho; bottom bar + pantalla menú.
- [ ] F14: smoke E2E producción (auth, cola, agenda, emails, realtime).
