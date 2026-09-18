# 💈 StetikShop — Documentación Oficial y Arquitectura

Bienvenido al repositorio oficial de **documentación, arquitectura de software, reglas de negocio y diseño** de **StetikShop** (Plataforma Luxury Beauty Tech para barberías y salones de estética).

---

## 🔗 Repositorios Oficiales del Proyecto

* 💻 **Frontend / Cliente SPA (React + Vite + Tailwind)**: [JASBOOTSTUDIOS/ss-client](https://github.com/JASBOOTSTUDIOS/ss-client)
* ⚙️ **Backend / API REST & WebSockets (Express + Prisma + Supabase)**: [JASBOOTSTUDIOS/ss-server](https://github.com/JASBOOTSTUDIOS/ss-server)

---

## 📑 Índice de Reglas de Negocio y Arquitectura

Toda la documentación técnica y de negocio se encuentra organizada en la carpeta [docs/](./docs):

| Documento | Descripción |
|---|---|
| [00-README.md](./docs/00-README.md) | Introducción y convenciones generales de la documentación. |
| [01-dominio.md](./docs/01-dominio.md) | Modelo de dominio, glosario de términos y flujos principales. |
| [02-auth-roles.md](./docs/02-auth-roles.md) | Autenticación, roles globales, roles por negocio y permisos (RBAC). |
| [03-negocios.md](./docs/03-negocios.md) | Reglas de creación, personalización y gestión de negocios. |
| [04-empleados.md](./docs/04-empleados.md) | Gestión de empleados, disponibilidades y roles de staff. |
| [05-servicios.md](./docs/05-servicios.md) | Catálogo de servicios, modalidades (cola/agenda) y precios. |
| [06-horarios.md](./docs/06-horarios.md) | Horarios de atención, turnos y cálculo dinámico de slots. |
| [07-cola.md](./docs/07-cola.md) | Flujo de atención por turno / orden de llegada (cálculo dinámico de tiempos). |
| [08-agenda.md](./docs/08-agenda.md) | Flujo de citas programadas y control de solapamientos. |
| [09-notificaciones.md](./docs/09-notificaciones.md) | Notificaciones in-app y correos transaccionales (SMTP/Resend). |
| [10-errores.md](./docs/10-errores.md) | Catálogo oficial de errores y directriz de cero errores silenciosos. |
| [11-base-datos.md](./docs/11-base-datos.md) | Esquema relacional de PostgreSQL, triggers y RLS. |
| [12-api.md](./docs/12-api.md) | Especificación de endpoints de la API REST. |
| [13-diseno.md](./docs/13-diseno.md) | Sistema de diseño (línea gráfica oficial, Luxury Beauty Tech). |
| [15-diseno-responsive.md](./docs/15-diseno-responsive.md) | Directrices para adaptación universal (móvil, tablet, desktop). |
| [TECHNICAL_PLAN.md](./docs/TECHNICAL_PLAN.md) | Plan técnico y fases de implementación (F0 a F14). |
| [CONTEXT.md](./docs/CONTEXT.md) | Estado mental del proyecto y decisiones tomadas. |
| [ERRORS.md](./docs/ERRORS.md) | Registro histórico de anti-patrones y soluciones aplicadas. |
| [MANUAL_GMAIL_SMTP.md](./docs/MANUAL_GMAIL_SMTP.md) | Manual de configuración de envío de correos vía SMTP. |

---

## 🛠️ Directrices de Trabajo y Coordinación de Agentes

* [AGENTS.md](./AGENTS.md): Reglas estrictas de calidad (§7 gates obligatorios: 100% tests, typecheck, build, lint), internacionalización (§8 i18n), SEO y coordinación multi-agente en paralelo (Antigravity & OpenCode).
* [PLAN.md](./PLAN.md): Plan vivo con el historial de turnos y la pizarra de tareas compartida.
* [lineagrafica.md](./lineagrafica.md): Guía oficial de diseño (fondo #0B0B0B, acentos dorado satinado #D4AF7C, tipografía Poppins).
