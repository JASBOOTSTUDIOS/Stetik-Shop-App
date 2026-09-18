# Reglas de Trabajo — StetikShop

## 1. Principios Básicos y Stack
- **Lenguaje**: Todas las respuestas e interfaz de usuario estrictamente en **español** (`es`).
- **Stack**: React + Tailwind + TypeScript (client), Express + TypeScript + Prisma + Supabase (server).
- **Código Limpio**: Nombres en inglés en código; sin comentarios innecesarios salvo que se soliciten.
- **Línea Gráfica Oficial**: Fondo negro profundo (`#0B0B0B`), oro satinado/metálico (`#D4AF7C` / `#E5C287`), textos crema (`#F8F8F8`), tipografía **Poppins** y gradientes/resplandores sutiles que aporten profundidad sin rigidez estática.

## 2. Adaptación Responsive Universal (Multi-Dispositivo)
- En **toda implementación o corrección**, el diseño debe adaptarse armónicamente a cada formato de pantalla:
  - **Móvil (320px – 640px)**: compacto, minimalista, sin ruido visual, elementos reducidos, barra inferior de exactamente 4 botones principales.
  - **Tablet (768px – 1024px)**: sin desbordamiento horizontal ni solapamiento entre barras y encabezados.
  - **Desktop / Web App (1024px – 1440px)**: apariencia completa de aplicación web con barra de navegación superior refinada y menú contextual.
  - **Pantallas Grandes / Ultra-Wide (1920px+)**: contenedores centrados con anchos máximos controlados (`max-w-* mx-auto`) para evitar interfaces estiradas.

## 3. Navegación y Entornos Separados
- **Flujos Separados por Entorno**: Cliente / Negocio / Admin con vistas, permisos y barras de navegación contextuales que no se mezclan.
- **Menú Complementario**: La pantalla de Menú contiene únicamente opciones que no están repetidas en las barras de navegación.
- **Acceso Público**: Pantalla inicial y explorador de negocios abiertos a todo público.

## 4. Implementación Lógica de Cada Flujo (Reglas de Negocio Clave)
- **Flujo de Cola (Atención por Turno / Orden de Llegada)**:
  - Abierto a todo público: permite turnos de clientes autenticados o anónimos con nombre completo.
  - Cálculo dinámico de tiempos: tiempo estimado de espera basado en la suma de duraciones de los servicios pendientes de las personas delante en cola.
  - Visualización cliente: el cliente ve únicamente su turno, la cantidad de personas delante y la estimación de tiempo (sin exponer datos privados de otros clientes).
  - Los tiempos y orden se actualizan en tiempo real tras cada llamado, atención o cancelación.
- **Flujo de Agenda (Citas Programadas)**:
  - Exige autenticación obligatoria del cliente por motivos de seguridad y trazabilidad.
  - Validación de slots: solo permite seleccionar turnos disponibles dentro del horario hábil y disponibilidad del empleado, sin solapamientos.
  - Reglas de tolerancia y posposición: el empleado puede posponer turnos según el tiempo de espera configurado por el dueño (ej. 10 minutos). Al posponer, el turno se cede al siguiente en espera; si se excede el límite de posposiciones configurado, el turno se cancela automáticamente.
  - Citas fijas: las citas agendadas se respetan a su hora programada para mantener el control y no alterar turnos arbitrariamente.
- **Flujo de Negocio y Personalización**:
  - Cada dueño puede personalizar la identidad de su local (colores de marca en `theme`, logo, portada, descripción, tiempos y disponibilidad de servicios).
  - Al ingresar a un negocio, la app refleja sus colores y muestra su cola y servicios específicos.
  - Dashboard de rendimiento: histórico de ingresos filtrable (horas, días, semanas, años), puntuaciones y comentarios de clientes.
- **Flujo Administrativo (Super Admin)**:
  - Supervisión global de locales, roles globales, usuarios y configuración de la plataforma sin mezclar elementos con el área de usuario ni de negocio.

## 5. Validación Preventiva y Blindaje de Integridad
- **Análisis Profundo Anti-Error**: Antes de implementar, analizar exhaustivamente cualquier acción o error que el usuario pueda cometer y que comprometa la integridad de la app.
- **Diseño a Prueba de Errores**: Deshabilitar estados inválidos en la UI, sanitizar entradas y validar de forma estricta tanto en cliente como en servidor (Zod compartido). El usuario no debe tener forma de provocar estados inconsistentes.
- **Validación de Todos los Casos de Uso**: Cubrir casos felices, entradas vacías, límites de concurrencia, datos maliciosos, cancelaciones y condiciones de carrera.

## 6. Manejo de Errores y Reputación de la App
- **Cero Errores Silenciosos**: Prohibidos terminantemente los `catch` vacíos, promesas sin capturar o errores tragados en silencio.
- **Mensajes Claros y Empáticos**: Todo error debe comunicarse al usuario con un mensaje claro, amigable y comprensible en español, ofreciendo una acción de salida o reintento, protegiendo siempre la reputación, elegancia y confiabilidad de la app (sin exponer trazas técnicas, errores crudos del servidor ni alertas nativas del navegador).

## 7. Verificación Obligatoria de Cada Turno
- Ninguna tarea se da por completada sin verificar:
  1. **Tests al 100%**: Ejecutar y pasar todas las suites de pruebas (`npm test` en `client` y `server`).
  2. **Typecheck Cero Errores**: Verificación estricta de TypeScript (`tsc --noEmit`).
  3. **Build Exitoso**: Compilación de producción limpia (`npm run build`).
  4. **Linter Sin Advertencias**: Código limpio bajo las reglas de ESLint (`npm run lint`).

## 8. Internacionalización (i18n) y SEO de Alto Nivel
- **Textos 100% Dinámicos**: Absolutamente ningún texto debe estar *hardcodeado* (estático) en los componentes. Todo texto visible por el usuario DEBE servirse mediante el sistema de idiomas (e.g. `react-i18next`). Esto es innegociable.
- **Mejor SEO Posible**: La aplicación web debe incluir el mejor SEO técnico (meta tags dinámicas, títulos por pantalla, descripciones, Open Graph, Twitter Cards y etiquetas semánticas) usando herramientas como `react-helmet-async` para garantizar indexación y una presentación inmaculada al compartir enlaces.

## 9. Protección Estricta de Navegación
- **Navegación Siempre Protegida**: Nadie que no tenga el rol requerido puede acceder a funcionalidades o pantallas protegidas. 
- **Verificación Continua**: Si el token se vence, se invalida, o el usuario pierde el rol, la aplicación debe bloquear inmediatamente el acceso y redirigir fuera de las pantallas protegidas (ej. hacia login o home). Esto debe implementarse de forma proactiva en el cliente.

## 10. Coordinación Multi-Agente en Paralelo (Antigravity & OpenCode)
- **Pizarra Compartida Viva**: `PLAN.md` es la única fuente de la verdad compartida entre Antigravity y OpenCode. Todo agente DEBE leer `PLAN.md` antes de actuar y actualizarlo al finalizar su turno.
- **Reserva Exclusiva de Tareas (Task Claiming)**:
  - Ningún agente puede trabajar sobre una tarea o archivo que el otro tenga en curso.
  - Al iniciar una tarea, marcarla en `PLAN.md` como: `[EN CURSO por Antigravity]` o `[EN CURSO por OpenCode]`.
  - Al completar la tarea, verificar los 4 gates de calidad (§7) y cambiarla a `[x]` con su nota de turno.
- **División de Trabajo Recomendada**:
  - **Horizontal**: Un agente asume el Backend (`server/`) y el otro el Frontend (`client/`).
  - **Vertical**: Funcionalidades desacopladas (ej. un agente trabaja en la lógica de cálculo de slots mientras el otro diseña la pantalla de gestión).
- **Cero Regresiones**: Antes de commitear, siempre verificar que los cambios no rompan el trabajo previo del otro agente. Ambos agentes están obligados a mantener el 100% de cumplimiento de estas reglas.