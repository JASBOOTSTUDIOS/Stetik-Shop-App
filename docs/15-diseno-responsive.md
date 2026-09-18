# BR-15: Diseño Responsive Multidispositivo

StetikShop es una aplicación completamente responsive, funcional y visualmente consistente en todos los dispositivos.

## Enfoque de implementación

**Mobile-first** con Tailwind CSS breakpoints: `sm:` (640px), `md:` (768px), `lg:` (1024px), `xl:` (1280px), `2xl:` (1536px).

## Rangos orientativos

| Rango | Ancho |
|-------|-------|
| Extra pequeño | 320–479 px |
| Móvil | 480–767 px |
| Tablet | 768–1023 px |
| Laptop | 1024–1279 px |
| Desktop | 1280–1535 px |
| Pantallas grandes | 1536 px+ |

## Reglas obligatorias

1. Ninguna pantalla genera desplazamiento horizontal involuntario.
2. Ningún texto, botón, formulario, tarjeta, tabla o imagen queda cortado.
3. Las tarjetas se adaptan de manera fluida al ancho disponible.
4. Los tamaños de fuente, espacios y columnas se ajustan cuando es necesario.
5. Las imágenes conservan proporciones correctas y utilizan dimensiones adaptables.
6. Los botones son cómodos de utilizar con los dedos en dispositivos táctiles (mínimo 44px de alto).
7. Los formularios funcionan correctamente en pantallas pequeñas.
8. Las tablas y dashboards complejos cuentan con solución responsive (columnas reorganizadas, tarjetas, desplazamiento horizontal localizado o vistas alternativas).
9. Los modales, menús desplegables, selectores de horarios y calendarios caben correctamente en la pantalla.
10. No se utilizan alturas fijas que oculten contenido importante.
11. No se depende únicamente de hover para mostrar información o ejecutar acciones.
12. Las acciones principales permanecen visibles en móviles.
13. No se utilizan breakpoints arbitrarios que rompan el diseño entre tamaños intermedios.
14. Se mantiene contraste, legibilidad, accesibilidad y consistencia visual en todos los tamaños.

## Adaptación por dispositivo

### Móvil (320–767 px)
- Composición de una columna.
- Navegación inferior para el área del cliente (`BottomBar` con `md:hidden`).
- Navegación del negocio/admin a menú accesible.
- Tarjetas apiladas para estadísticas, citas, servicios y productos.
- Tamaños de botones y campos de entrada adaptados al tacto.
- Paneles no excesivamente densos.
- Priorización de acciones importantes.
- Desplazamiento localizado en contenido que lo necesita.

### Tablet (768–1023 px)
- Una o dos columnas según el ancho disponible.
- Espacio adicional sin dispersión.
- Navegación y paneles laterales adaptados.
- Buena proporción entre imágenes, texto y controles.
- Dashboards, catálogos y formularios se reorganizan correctamente.

### Laptop y desktop (1024 px+)
- Navegación lateral o superior según el módulo.
- Dashboards de varias columnas cuando el espacio lo permite.
- Grids de productos y servicios adaptables.
- Contenido dentro de ancho máximo razonable (`max-w-6xl`).
- Elementos no se estiran excesivamente en monitores muy anchos.
- Tablas, calendarios y paneles adaptados al ancho real.

## Componentes responsive verificados

- Splash screen y pantalla de bienvenida (HomeScreen).
- Login, registro y recuperación de contraseña.
- Página de inicio (HomeScreen).
- Búsqueda y categorías.
- Catálogo de productos.
- Detalle de producto/servicio.
- Perfil de negocio.
- Reserva de citas y selección de horarios.
- Cola de espera.
- Agenda y calendario.
- Notificaciones.
- Perfil de usuario.
- Dashboard del negocio.
- Gestión de empleados, servicios, horarios.
- Estadísticas y gráficos.
- Panel administrativo.
- Modales, formularios, menús y mensajes de error.

## Pruebas responsive obligatorias

Verificar en: 320×568, 360×800, 390×844, 414×896, 768×1024, 1024×768, 1280×720, 1440×900, 1920×1080 y anchos intermedios.

Comprobar: ausencia de desbordamiento horizontal, texto legible, botones accesibles, imágenes correctamente ajustadas, menús funcionales, formularios completos, tablas utilizables, estados de carga/vacío/error correctos, coherencia visual.

## Regla de calidad

Una pantalla solo estará terminada cuando funcione correctamente en móvil, tablet y escritorio, manteniendo la identidad visual de StetikShop y sin sacrificar la experiencia de usuario.

**Prioridad: funcionalidad + responsive + accesibilidad + identidad visual premium.**
