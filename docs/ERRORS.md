# StetikShop — ERRORS.md (Registro de errores comunes)

> Propósito: registrar errores cometidos/identificados para NO repetirlos.
> Regla de uso (AGENTS §0/§1.3):
> 1. **Antes de escribir código**, revisar esta lista: si el patrón a usar está marcado como error → no repetirlo.
> 2. **Cada vez que se encuentra un error**, revisar si ya está documentado; si no, añadirlo aquí con causa + cómo evitarlo.
> 3. Cada entrada: **síntoma → causa → prevención** a prueba de repetición.

---

## E-01 — Plan que no deja huella

- **Síntoma:** se implementa sin que el plan/contexto reflejen qué se hizo.
- **Causa:** no leer/actualizar `PLAN.md` y `docs/CONTEXT.md` al inicio y cierre.
- **Prevención:** obligatorio seguir el ciclo de turno de AGENTS §1.2 (leer → escribir alcance → implementar → tests → verificar → actualizar → marcar).

## E-02 — Tests que no cubren el turno completo

- **Síntoma:** tests solo del camino feliz o escritos para "otro turno"; el turno se da por cerrado con casos sin probar.
- **Causa:** delegar tests al final sin revisar casos negativos/permisos/concurrencia/estados.
- **Prevención:** por cada feature, al menos: feliz, validación, permisos, concurrencia (si aplica), todos los estados del ciclo de vida. Un turno NO termina hasta que sus tests pasan.

## E-03 — Errores silenciosos (el gran prohibido)

- **Síntoma:** `catch {}` vacíos, promesas sin `.catch`, `try/catch` que traga, `console.log` sin registro estructurado.
- **Causa:** prisa o creer que "no pasa nada" ante un fallo asíncrono.
- **Prevención:** prohibido por AGENTS §2.2; toda falla → `AppError` con `code`+`message`; los fallos asíncronos van a logs estructurados con contexto; revisión de diff por turno (AGENTS §4.1).

## E-04 — Un botón que hace lo que el backend rechazaría

- **Síntoma:** la UI muestra una acción que el `requireRole` del server no permite (o filtra en RLS).
- **Causa:** diseñar UI sin contrastar permisos con `docs/02-auth-roles.md` y el contrato de `docs/12-api.md`.
- **Prevención:** regla AGENTS §1.4: pantalla nunca muestra datos/acciones fuera de su permiso; cruzar cada ruta con su `requireRole`.

## E-05 — Regla de negocio inventada en el código

- **Síntoma:** comportamiento que no está en `docs/BR` (ej. un límite arbitrario de cancelación).
- **Causa:** decidir lógica "de sentido común" sin documentar.
- **Prevención:** si no está en `docs/`, primero se documenta la BR, luego se implementa (AGENTS §1.4).

## E-06 — Mensajes genéricos del navegador o del framework

- **Síntoma:** `alert()`, `confirm()`, textos crudos de validación de librerías.
- **Causa:** usar defaults en lugar de componentes propios.
- **Prevención:** todos los mensajes visibles son textos propios en español, con tono StetikShop, título+descripción+acción (AGENTS §2.3).

## E-07 — Responsive roto en extremos (240 px y ultra-wide)

- **Síntoma:** layout estirado o cortado en pantallas gigantes o muy pequeñas; bottom bar mal posicionada.
- **Causa:** solo probar en desktop o usar anchos fijos.
- **Prevención:** mobile-first + `clamp()` + `max-w-*`; verificar 375/768/1440/1920+ por componente (AGENTS §5.3).

## E-08 — Bottom bar móvil sin pantalla de Menú

- **Síntoma:** acciones que no caben en la bottom bar quedan inaccesibles en móvil.
- **Causa:** poner todos los accesos en la bottom bar o esconder el resto.
- **Prevención:** bottom bar con accesos principales + botón **Menú** que abre pantalla con el resto (AGENTS §5.2); en desktop/tablet, sidebar completa.

## E-09 — Mezclar UI de áreas (usuario / negocio / admin) en una misma pantalla

- **Síntoma:** links/componentes de admin dentro del área de usuario o viceversa.
- **Causa:** un solo layout global para todo.
- **Prevención:** tres áreas con layout/rutas/permisos propios; el botón "Ir a Admin" cambia el flujo completo y el rol activo (AGENTS §6).

## E-10 — Cálculo de cupos fuera de la zona horaria del negocio

- **Síntoma:** el "hoy" del negocio no coincide con el del servidor; citas ofrecidas en día equivocado.
- **Causa:** usar `new Date()` del servidor o la timezone del usuario en `getAvailableSlots`.
- **Prevención:** UTC en BD; todo cálculo de disponibilidad en `businesses.timezone` (BR-06.22). Tests con UTC±X.

## E-11 — Posición de cola duplicada (concurrencia)

- **Síntoma:** dos clientes obtienen la misma `queue_position`.
- **Causa:** calcular `MAX(position)+1` fuera de transacción.
- **Prevención:** transacción + `SELECT ... FOR UPDATE` (BR-07.26/07.27) y test de concurrencia simulada.

## E-12 — Secretos commiteados / config en repos

- **Síntoma:** `SUPABASE_SERVICE_ROLE_KEY`, `RESEND_API_KEY`, etc. en código o git.
- **Causa:** pegar `.env` o hardcodear.
- **Prevención:** regla AGENTS §4.6: solo `.env` fuera de git; nunca credenciales en código.

## E-13 — `strict: false` o `any` silenciosos

- **Síntoma:** correr con TS lax o y `any` a mansalva.
- **Causa:** "es más rápido".
- **Prevención:** AGENTS §2.2: `strict: true`, sin `any` silenciosos; el typecheck se corre al cierre de cada turno.

## E-14 — Docs que no se sincronizan (CONTEXT estancado)

- **Síntoma:** `CONTEXT.md` dice "pendiente" lo que ya está hecho.
- **Causa:** cerrar turno sin actualizar.
- **Prevención:** `CONTEXT.md` se actualiza SIEMPRE al cierre (AGENTS §7); si algo no está, no está registrado.

## E-15 — Puertos Supabase locales en conflicto

- **Síntoma:** `supabase start` falla con `Bind for 0.0.0.0:5432x failed: port is already allocated` cuando ya corre otro proyecto local.
- **Causa:** usar los puertos por defecto (54321–54327) sin comprobar `docker ps`.
- **Prevención:** antes de `supabase start`, `docker ps`; si hay otros proyectos, mapear `api/db/studio/inbucket/...` en `supabase/config.toml` a un rango libre y actualizar los `.env` (en este repo: rango 54431+).

## E-16 — Strings de UI fuera del sistema i18n

- **Síntoma:** cadenas visibles hardcodeadas dentro de componentes (`<p>Hola</p>`) o con idioma mezclado.
- **Causa:** escribir texto directo por comodidad sin pasar por i18n.
- **Prevención:** AGENTS §2.3: todo texto visible va por `t('...')` con `es` como fuente; en el cliente usar i18next desde el inicio (no hardcodear ni siquiera en `es`). Los errores del server se traducen por `error.code`.

---

## E-17 — Configuración inválida en `.opencode/agent/*.md`

- **Síntoma:** OpenCode rechaza la configuración del agente con `Expected number | undefined, got [...]`.
- **Causa:** el campo `steps` en el frontmatter YAML se definió como array de strings en vez de entero positivo.
- **Prevención:** `steps`, si se define, debe ser un `integer` positivo (≥1) que represente el máximo de iteraciones del agente. Los pasos detallados del flujo van en el cuerpo del markdown, no en el frontmatter. **Decisión de proyecto (2026-09-16): el campo `steps` no se define en `.opencode/agent/build.md` ni `.kilo/agent/build.md`** — los turnos se ejecutan con el default del esquema `AgentConfig` y no se corta la implementación a mitad de turno. Ver esquema `AgentConfig` de OpenCode.

---

## E-18 — Relación polimórfica en Prisma

- **Síntoma:** `Error parsing attribute "@relation"` al tener dos FKs desde la misma tabla a tablas diferentes usando el mismo campo.
- **Causa:** `user_roles.roleId` es polimórfico (puede apuntar a `global_roles.id` o `business_roles.id`). Dos `@relation` con `fields: [roleId]` generan FK duplicadas sobre la misma columna.
- **Prevención:** Para relaciones polimórficas en Prisma, crear columnas FK separadas (`global_role_id`, `business_role_id`) con sus propias `@relation`, una por tabla destino. La columna `role_id` queda como referencia no-FK (la integridad la maneja RLS/triggers en Supabase). Ver `docs/11-base-datos.md` §3.

## E-19 — Prisma Client sin driver adapter

- **Síntoma:** `PrismaClientInitializationError: A driver adapter is required` al instanciar `new PrismaClient()` sin adapter.
- **Causa:** Prisma v7 requiere un driver adapter (PrismaPg) para conectar a la BD; no usa conexión directa como v6.
- **Prevención:** Siempre crear PrismaClient con adapter: `new PrismaClient({ adapter: new PrismaPg({ connectionString: env.DATABASE_URL }) })` en `src/db/prisma.ts`. Los tests no deben instanciar PrismaClient sin adapter.

---

## E-20 — Kilo Code ignora las rules de `.opencode/`

- **Síntoma:** Kilo Code (CLI/VS Code/JetBrains) no aplica el flujo de turnos ni las reglas del proyecto aunque existan `.opencode/`, `opencode.json`, `PLAN.md` y `docs/`.
- **Causa:** Kilo ya no hace fallback a la configuración de `.opencode/`; exige su propia config (`kilo.jsonc` en raíz o `.kilo/kilo.jsonc`) y sus directorios (`.kilo/agent/*.md`, `.kilo/skills/`, `.kilo/instructions.md`). Solo `AGENTS.md`/`CLAUDE.md` se auto-descubren (estándar multi-herramienta).
- **Prevención:** mantener un espejo de las rules para Kilo: `kilo.jsonc` con `instructions` = AGENTS.md + PLAN.md + docs/CONTEXT.md + docs/ERRORS.md; agentes en `.kilo/agent/*.md`; skill de dominio en `.kilo/skills/*/SKILL.md`; `$schema` = `https://app.kilo.ai/config.json`. Verificar claves top-level contra el esquema oficial (`Kilo-Org/kilocode` `config.ts`): `default_agent`, `instructions`, `permission`, `snapshot`, `share`, `model`. No usar `references` (no soportado por Kilo).

## E-21 — Clave i18n duplicada en resources.ts

- **Síntoma:** `error TS1117: An object literal cannot have multiple properties with the same name` en `resources.ts`.
- **Causa:** al mantener traducciones `es` y `en` en un solo archivo, se copia un bloque de claves (ej. `nav`) dentro del mismo objeto de locale en vez de tener una sola definición por locale.
- **Prevención:** cada clave (`nav`, `home`, `app`, etc.) debe aparecer **una sola vez** dentro de cada objeto de locale (`es.translation`, `en.translation`). El idioma se selecciona con `lng`/`fallbackLng` en `i18n/index.ts`, no con duplicados.

## E-22 — Incompatibilidad de tipos `Dispatch<SetStateAction<T>>` con callback en React

- **Síntoma:** `error TS2322: Type 'Dispatch<SetStateAction<T>>' is not assignable to type '(...: T) => void'` al pasar `setState` directamente como prop callback.
- **Causa:** `useState`'s `dispatch` acepta tanto el valor como una función actualizadora, pero la prop del componente hijo espera solo `(value) => void`.
- **Prevención:** envolver en arrow function: `onChange={(val) => setVal(val as Tipo)}` o definir la prop como `Dispatch<SetStateAction<T>>` en ambos lados si el hijo también usa el setter directamente.

---

## E-23 — Error handler registrado ANTES de las rutas en Express

- **Síntoma:** en tests de API, `expect(res.body.error).toBe('...')` falla con `undefined` y la respuesta es el HTML por defecto de Express (`Cannot GET` / stacktrace), no el JSON estándar del proyecto.
- **Causa:** en `buildApp()` el `errorHandler` se registraba antes de montar las rutas. Los error-middlewares de Express solo se ejecutan si quedan DESPUÉS del punto de fallo en la cadena de middlewares; si van antes, Express usa su handler por defecto.
- **Prevención:** el orden en `buildApp()` debe ser: middlewares globales → rutas → `errorHandler` (y `notFoundHandler` justo detrás de las rutas). Ver `server/src/app.ts`.

## E-24 — `vi.clearAllMocks()` no resetea implementaciones (fuga de mocks entre tests)

- **Síntoma:** tests de `requireRole` pasan o fallan según el orden de ejecución: un `findFirst` devuelve `null` cuando otro test lo dejó como tal.
- **Causa:** `vi.clearAllMocks()` (o `vi.resetAllMocks()`) en `beforeEach` limpia llamadas/implementaciones, PERO no revierte `vi.mock(...)` module-level: el mock raíz (ej. `getPrisma`) sigue devolviendo el valor del último `mockResolvedValue` de otro test.
- **Prevención:** en `beforeEach`, además de limpiar, usar `vi.mocked(getPrisma).mockReset()` y redefinir explícitamente la implementación por test (`mockResolvedValue`, `mockReturnValue`). Nunca asumir el estado del mock de otro test. Ver `server/tests/auth.test.ts`.

## E-25 — Claves i18n parciales dejadas por trabajo paralelo (es con inglés, en sin traducción)

- **Síntoma:** en `resources.ts`, `es.translation.status.*` contiene strings en inglés ('Connecting to server...') y las claves `screens.*` existen solo en `es` (el locale `en` cae en fallback silencioso).
- **Causa:** scaffolding de navegación (screens/status) añadido por un flujo paralelo al turno de branding/auth sin pasar por el catálogo i18n ni cubrir ambos locales.
- **Prevención:** toda clave i18n se añade en `es` (fuente) y `en` a la vez; nada de inglés dentro de `es`. **Corregido en turno 10**: `status.*` en español, `status.*`/`screens.*` completos en `en`, añadida sección `navAccess` en ambos idiomas.

## E-26 — Señal de email verificado mal detectada en GoTrue (`app_metadata.email_verified` no existe)

- **Síntoma (detectado en prueba en vivo local):** tras un signup exitoso en Supabase local (confirmación de email desactivada), GoTrue devuelve sesión con `access_token` y `user.email_confirmed_at > null`…, pero el cliente miraba `tokens.user?.app_metadata?.email_verified === true`, que GoTrue NO expone en `app_metadata` (solo `user_metadata.email_verified` y `email_confirmed_at`). Resultado: la UI mostraba "Revisa tu email" y NO autenticaba pese a haber sesión válida.
- **Causa:** la señal se interpretaba en el cliente mirando un campo que la API real no devuelve; se asumió el shape de `supabase-js` en vez del de la GoTrue REST que consume el server.
- **Prevención:** la normalización del signup vive en el **server** (`services/auth.ts`): si GoTrue devuelve `access_token` → sesión válida (email confirmado) → se devuelve tal cual; si no → `{ requiresEmailVerification: true, user }`. El cliente solo distingue por la llave `requiresEmailVerification` del shape normalizado (nunca por `app_metadata`). Ver `server/src/services/auth.ts` y `client/src/stores/authStore.ts`.

## E-27 — Bootstrap sin `connectDb()`: rutas con Prisma respondían siempre 503 DATABASE_UNAVAILABLE

- **Síntoma (detectado en prueba en vivo):** `POST /auth/login` y `/auth/signup` funcionaban, pero `GET /auth/me` respondía `503 DATABASE_UNAVAILABLE` siempre.
- **Causa:** `index.ts` arrancaba `app.listen` sin llamar a `connectDb()`; `getPrisma()` lanza `DATABASE_UNAVAILABLE` si la instancia no fue inicializada, y ninguna ruta/middleware llama `connectDb()`.
- **Prevención:** el bootstrap es `connectDb()` ANTES de `listen` (`server/src/server.ts` `startServer`), con errores de arranque logueados y `process.exit(1)` (`index.ts`). Tests en `server/tests/bootstrap.test.ts` (conecta antes de escuchar; no escucha si la BD cae).

## E-28 — Consulta SQL con columna de tipo `char` falla en Prisma

- **Síntoma:** `Error parsing attribute "@relation"` o error de deserialización al hacer `prisma.$queryRaw` con columnas `char`/`bytea` (ej. `t.tgtype` en triggers de PostgreSQL).
- **Causa:** Prisma v7 no puede deserializar tipos `char`/`bytea` sin castear.
- **Prevención:** hacer `::text` cast en columnas de tipo `char` o `bytea` en consultas SQL: `t.tgtype::text`. Aplica a consultas que lean metadatos del sistema (`pg_trigger`, `pg_attribute`, etc.).

## E-29 — `createBusinessRoleSchema` exigía `businessId` en el body

- **Síntoma:** test de crear rol de negocio fallaba con 400 porque el body no incluía `businessId`, pero el `businessId` se pasa por URL params, no por body.
- **Causa:** el schema Zod definía `businessId: z.string().uuid()` como requerido.
- **Prevención:** hacer `businessId: z.string().uuid().optional()` cuando el valor viene de los params de la URL, no del body. Ver `server/src/routes/rbac.ts`.

## E-30 — Regex de nombre de rol no permitía números

- **Síntoma:** test de crear rol global con nombre `test_role_123456` fallaba con 400 (Zod validation).
- **Causa:** el regex `/^[a-z_]+$/` en `createGlobalRoleSchema` y `createBusinessRoleSchema` no permitía dígitos.
- **Prevención:** cambiar a `/^[a-z0-9_]+$/` para permitir letras minúsculas, números y guiones bajos. Ver `server/src/routes/rbac.ts`.

## E-31 — Seed no ejecutado en tests de RBAC

- **Síntoma:** `GET /admin/permissions` devolvía 0 permisos; los tests de crear roles fallaban por falta de permisos en BD.
- **Causa:** el `afterAll` del test limpiaba `permission_definitions` y `global_roles` entre ejecuciones, y el seed no se corría en `beforeAll`.
- **Prevención:** ejecutar `execSync('npx tsx prisma/seed.ts', { cwd: serverRoot, stdio: 'ignore' })` en `beforeAll` y NO limpiar `permission_definitions`/`global_roles` en `afterAll`. Ver `server/tests/rbac.test.ts`.

## E-32 — Tests de integración concurrentes con TRUNCATE en BD compartida

- **Síntoma:** tests que pasan de forma aislada fallan intermitentemente con 404/500 cuando se corre la suite completa (`vitest run`).
- **Causa:** Vitest ejecuta archivos de tests en paralelo por defecto; cuando un test file termina, su `afterAll` ejecuta `TRUNCATE TABLE ... CASCADE`, eliminando registros que otro test file concurrente necesita en ese mismo instante.
- **Prevención:** en `server/vitest.config.ts`, configurar `fileParallelism: false` para que los tests de integración contra la base de datos se ejecuten de forma estrictamente secuencial.

---

## E-33 — Turno cerrado sin registro en el historial de `PLAN.md`

- **Síntoma:** el "Turno actual" dice *turno 17* pero la tabla de historial se quedó en el *14*: los turnos 15 y 16 se ejecutaron sin fila propia (huella incompleta).
- **Causa:** se actualizó el bloque "Turno actual" sin añadir la fila correspondiente en el registro de turnos.
- **Prevención:** al cerrar el turno, además de reescribir "Turno actual", **añadir la fila del turno en la tabla de historial** con fecha, alcance y resultado de tests. Si un turno quedó sin registrar, dejarlo explícito como `⚠️` en vez de inventar su contenido.

---

## E-34 — i18n y manejo de errores aplicados de forma parcial

- **Síntoma:** tras una auditoría de i18n quedaban textos hardcodeados y `catch` que solo limpiaban estado: `QueueScreen` (`catch { setServices([]); }`), `BusinessPublicScreens`, `HomeScreen` (banner con emoji 🎉), `AuthScreen` (aviso de email no confirmado, reenvío, `aria-label` de contraseña) y los bloques `catch` de varias pantallas admin.
- **Causa:** los refactors se hicieron pantalla a pantalla sin un grep de control global, y el `catch` corto se percibía como "manejo suficiente" (no lo es: oculta el fallo).
- **Prevención:** al cerrar cualquier turno con UI, ejecutar dos greps de control antes de dar por hecho el i18n:
  1. nodos de texto JSX: `>[^<>{}\n]*[A-Za-zÁÉÍÓÚáéíóúÑñ¿¡][^<>{}\n]*<`
  2. líneas de texto suelto con acentos (`[ÁÉÍÓÚáéíóúÑñ¿¡]`) excluyendo comentarios y `console.error`.
  Y revisar cada `catch`: prohibido que no registre el error (`console.error` con contexto) o no informe al usuario.

---

## E-35 — Datos inventados y acciones sin handler en la UI

- **Síntoma:** `BusinessPublicScreens` mostraba "2 personas delante" y "~25 min" como si fueran reales, un modal "Ponerme en turno" que solo cambiaba estado local sin llamar a la API, y el botón "Agendar Cita" sin `onClick`.
- **Causa:** pantalla maquetada con datos de ejemplo durante el scaffolding y nunca conectada al estado real.
- **Prevención:** ninguna vista puede renderizar métricas inventadas ni mostrar acciones sin efecto. Toda acción visible debe (a) llamar al endpoint real, (b) navegar a la pantalla real que lo hace, o (c) estar deshabilitada con explicación. Las métricas se leen siempre de la fuente real (`queueStore`/API).

---

## E-36 — Import de helper local sin extensión `.js` (TS2835)

- **Síntoma:** `error TS2835: Relative import paths need explicit file extensions in ECMAScript imports when '--moduleResolution' is 'node16' or 'nodenext'`.
- **Causa:** importar `./helpers/supabaseAuthMock` desde un test de un proyecto con `moduleResolution: nodenext` (ESM).
- **Prevención:** en `server/tests/*.ts`, los imports relativos llevan extensión `.js` (`import { ... } from './helpers/supabaseAuthMock.js'`), igual que en `src`.

---

## Checklist de pre-commit mental (por turno)

☐ ¿Esto ya está documentado como error? (revisar este archivo)  
☐ ¿Actualicé PLAN.md (Hecho/Pendiente/Próximo)?  
☐ ¿Los tests cubren TODO lo implementado en este turno y pasan?  
☐ ¿Revisé el diff en busca de silenciadores/catches vacíos?  
☐ ¿Cada ruta responde con formato de error estándar y su permiso?  
☐ ¿Actualicé CONTEXT.md? ¿Dejé nuevos errores aquí?  
☐ ¿Algún secreto en el diff?