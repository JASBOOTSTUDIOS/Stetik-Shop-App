# StetikShop — BR-13 · Sistema de Diseño (Línea Gráfica Oficial)

> Fuente de verdad del **diseño** de StetikShop. La dirección artística es **Luxury Beauty Tech**: lujo minimalista + belleza contemporánea + tecnología moderna.
> Regla de oro: la estética es consistente en TODAS las pantallas, pero **la usabilidad, legibilidad y accesibilidad SIEMPRE tienen prioridad**.
> El nombre de la marca se escribe exactamente **StetikShop** (Stetik en crema/blanco + Shop en oro principal).

---

## BR-13.1 — Paleta de colores oficial (tokens de diseño)

Colores usados SIEMPRE vía tokens/variables (Tailwind theme, CSS custom properties o constantes compartidas). NUNCA se inventa una paleta nueva por pantalla.

### Principales (identidad)

| Token | Hex | Uso |
|-------|-----|-----|
| `brand-gold` | `#D4AF7C` | Acentos de marca, elementos destacados, iconos seleccionados, detalles premium. **Acento**, nunca fondo predominante |
| `brand-black` | `#0B0B0B` | Fondos oscuros, logo, navegación oscura, superficies principales |
| `brand-dark` | `#1F1F1F` | Tarjetas, paneles, menús, superficies secundarias |
| `brand-gray` | `#6B6B6B` | Textos secundarios, descripciones, elementos de apoyo |
| `brand-cream` | `#F8F8F8` | Fondos claros, superficies limpias, texto sobre fondos oscuros |

### Funcionales (estados)

| Estado | Directriz |
|--------|-----------|
| Éxito | Verde sobrio, claramente legible |
| Advertencia | Ámbar |
| Error | Rojo elegante con contraste suficiente |
| Información | Azul neutro |
| Texto principal en fondo claro | Negro o gris muy oscuro |
| Texto principal en fondo oscuro | Crema o blanco |
| Texto secundario | Gris con contraste adecuado |

Prohibido: fondos dorados extensos, combinaciones que dificulten la lectura, oro como color dominante de componentes.

## BR-13.2 — Logotipo y símbolo

- Símbolo: **S estilizada** con forma fluida semejante a cinta/bolsa de compras elegante con asa. Transmite belleza, movimiento, estilo, comercio, exclusividad e identidad propia.
- Wordmark: **Stetik** en crema/blanco + **Shop** en oro principal (`#D4AF7C`).
- Reglas: no deformar el símbolo, no cambiar proporciones arbitrariamente, no exceso de brillos, no colocar sobre fondos de bajo contraste, no crear versiones distintas por pantalla. Versión clara sobre fondos oscuros y oscura sobre fondos claros. Zona de protección alrededor. Si existe archivo oficial del logo, usarlo.
- La identidad debe reconocerse solo con el símbolo "S" (favicon, avatar, botón de inicio).

## BR-13.3 — Tipografía

- **Poppins** como única familia principal (máx. una complementaria solo con justificación visual clara).
- Jerarquía: títulos principales Poppins 600/700 · títulos de sección 600 · texto normal 400 · botones 500/600 · etiquetas/metadatos 400/500.
- Clara, moderna, geométrica, elegante, legible en móvil y escritorio.
- Prohibidas letras ornamentales para textos funcionales (formularios, precios, citas, información importante).

## BR-13.4 — Estilo de interfaz (dos ambientes de la misma marca)

1. **Experiencia de marca** (fondos oscuros negro mate / gris carbón / crema / detalles dorados): pantalla de bienvenida, splash, logo, branding, cabeceras especiales, secciones promocionales, componentes destacados, experiencias premium.
2. **Experiencia funcional** (composición clara, limpia, ordenada — crema, blanco y grises suaves): catálogo, búsqueda, categorías, reserva de citas, horarios, turnos, formularios, paneles negocio/admin, tablas, estadísticas, notificaciones.

Ambos ambientes comparten identidad por colores, tipografía, iconografía, espaciado y componentes. No todas las pantallas deben ser negras.

## BR-13.5 — Componentes visuales

### Botones
- Bordes redondeados, apariencia premium, cómodos en móvil.
- **Primario:** fondo oro `#D4AF7C`, texto oscuro (contraste), peso 500/600, estados hover/focus/disabled/loading, transiciones discretas.
- **Secundario:** transparente o superficie oscura, borde fino dorado o neutro, texto crema/blanco/dorado según contraste.
- **Terciario:** sin fondo fuerte, texto de marca o neutro (acciones secundarias).

### Tarjetas
- Bordes redondeados, espaciado interno generoso, jerarquía visual clara, imágenes de calidad, bordes sutiles, sombras suaves cuando hagan falta, estados de carga y vacío.
- Premium: superficies oscuras + pequeños detalles dorados. Funcionales: prioridad a la legibilidad.

### Inputs y formularios
- Etiquetas claras, bordes discretos, radio consistente, foco visible, mensajes de error comprensibles, iconos solo si aportan valor, contraste suficiente, tamaño táctil móvil. Nada de formularios recargados.

### Iconos
- Lineales, minimalistas, consistentes: grosor uniforme, tamaño coherente, estilo moderno, color crema/gris/negro/dorado según contexto, buena alineación.
- Prohibido mezclar iconos 3D, emojis, estilos incompatibles o ilustraciones aleatorias.

## BR-13.6 — Elementos gráficos distintivos (sutiles, no decorativos)

- Líneas curvas doradas, formas fluidas de la "S", arcos/ondas suaves, líneas finas de separación, gradientes muy discretos, destellos dorados limitados, patrones abstractos belleza/movimiento, fondos con textura muy ligera.
- Prohibido: exceso de neón, fondos saturados, cristal exagerado, sombras muy fuertes, animaciones constantes, brillos excesivos, estética de videojuegos.

## BR-13.7 — Imágenes y fotografía

- Estética premium de belleza. Productos: fondos limpios, iluminación suave, composición editorial, tonos crema/negro/beige/dorado, consistencia entre imágenes. Negocios: fotos auténticas y profesionales de espacios de belleza. Banners: composiciones elegantes, espacio para texto, sin texto sobre zonas complejas.
- No generar imágenes artificiales que oculten información, ni decorativas donde se necesita imagen real del negocio/servicio.

## BR-13.8 — Navegación y UX

- Móvil: barra de navegación inferior (bottom bar), iconos claros, etiquetas legibles, acciones importantes fáciles de alcanzar, sin saturar.
- Escritorio: navegación lateral o superior, contenido centrado, tarjetas sin estirar.
- No sacrificar usabilidad por sofisticación.

## BR-13.9 — Técnica (React + Tailwind + TS)

1. Colores de marca como tokens/variables.
2. Componentes reutilizables: botones, inputs, tarjetas, modales, badges, navegación, encabezados.
3. Escala de espaciado consistente; radios de borde coherentes.
4. Estados hover/focus/active/disabled/loading/error definidos.
5. Jerarquía tipográfica uniforme (Poppins).
6. Variantes claras y oscuras cuando hagan falta.
7. Sin estilos duplicados entre pantallas.
8. Estructura responsive (AGENTS §5).
9. Texto legible y controles accesibles.
10. **No modificar lógica de negocio existente por estética; no eliminar funcionalidades por estética; mantener intactos permisos, reservas, turnos, horarios y estados.**
11. Separar componentes visuales de la lógica de negocio.
12. Si ya existe un sistema de componentes, **extenderlo**, no crear estilos incompatibles.

## BR-13.10 — Animaciones y microinteracciones

- Suaves, rápidas, discretas: aparición de tarjetas, transiciones de botones, cambios de estado, animaciones de carga, transiciones de navegación.
- Prohibido: animaciones permanentes, zoom excesivo, rebotes constantes, transiciones que retrasen acciones, animaciones que dañen accesibilidad.
- **Respetar `prefers-reduced-motion`**.

## BR-13.11 — Resultado visual esperado

**Negro carbón + crema + oro elegante + Poppins + formas fluidas + espacios limpios + componentes redondeados.**

Mensaje de marca: "Más que productos, es una experiencia de belleza."

Toda pantalla (cliente, negocio o admin) debe parecer de la misma marca, del icono de la app al dashboard. Antes de implementar una pantalla: analizar propósito, definir jerarquía visual y aplicar este sistema. Si una decisión visual entra en conflicto con accesibilidad o facilidad de uso → priorizar accesibilidad/facilidad sin perder identidad.

---

## Aplicación práctica (qué hace el agente por cada pantalla)

1. Leer el propósito de la pantalla y su jerarquía visual.
2. Usar SOLO tokens de BR-13.1 (nunca hex inventados).
3. Tipografía Poppins con jerarquía BR-13.3.
4. Componentes del design system existente; si no existe, crearlos reutilizables (BR-13.9) con textos i18n `t('...')` en `es`.
5. Elegir ambiente de marca (oscuro) vs funcional (claro) según BR-13.4.
6. Verificar responsive 375/768/1440/1920+ y accesibilidad antes de cerrar.
7. No inventar otra identidad visual.