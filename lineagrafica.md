# PROMPT MAESTRO — SISTEMA DE DISEÑO DE STETIKSHOP

## 1. Contexto del proyecto

Estoy desarrollando una aplicación llamada **StetikShop**, una plataforma digital relacionada con la belleza, el cuidado personal, la estética y la gestión de negocios de belleza.

La aplicación conecta a clientes con negocios como salones de belleza, barberías, centros de estética, spas y otros establecimientos de cuidado personal. Debe permitir explorar servicios, consultar productos, gestionar citas, consultar turnos y utilizar diferentes paneles según el tipo de usuario.

La identidad de StetikShop debe transmitir:

* Elegancia.
* Belleza y cuidado personal.
* Exclusividad accesible.
* Confianza y profesionalismo.
* Modernidad.
* Limpieza visual.
* Tecnología sofisticada.
* Una experiencia premium, pero fácil de utilizar.

El nombre de la marca es **StetikShop**. No quiero una interfaz genérica de administración ni una aplicación que parezca una plantilla de comercio electrónico convencional. Quiero una experiencia visual propia, coherente y reconocible.

---

## 2. Dirección artística de la marca

La línea gráfica de StetikShop se basa en una estética **Luxury Beauty Tech**: una combinación de lujo minimalista, belleza contemporánea y tecnología moderna.

La inspiración visual debe incluir:

* Marcas premium de belleza y cosmética.
* Envases elegantes de skincare y maquillaje.
* Superficies negras mate.
* Detalles dorados sutiles.
* Iluminación cálida y refinada.
* Espacios amplios y composición minimalista.
* Tipografía limpia y sofisticada.
* Elementos redondeados y una interfaz cómoda.
* Fotografías de productos de alta calidad.
* Una sensación de boutique digital de belleza.

La aplicación debe verse premium sin estar sobrecargada de adornos. El lujo debe comunicarse mediante la proporción, los espacios, la tipografía, los colores y los detalles, no mediante exceso de sombras, gradientes o elementos decorativos.

**Regla fundamental:** la estética debe ser consistente en todas las pantallas, pero la usabilidad, la legibilidad y la accesibilidad siempre tienen prioridad.

---

## 3. Paleta de colores oficial

Utiliza la siguiente paleta como base del sistema de diseño:

### Colores principales

* **Oro principal:** `#D4AF7C`

  * Uso: acentos de marca, elementos destacados, iconos seleccionados y detalles premium.
* **Negro:** `#0B0B0B`

  * Uso: fondos oscuros, logotipo, navegación oscura y superficies principales.
* **Gris oscuro:** `#1F1F1F`

  * Uso: tarjetas, paneles, menús y superficies secundarias.
* **Gris medio:** `#6B6B6B`

  * Uso: textos secundarios, descripciones y elementos de apoyo.
* **Crema:** `#F8F8F8`

  * Uso: fondos claros, superficies limpias y textos sobre fondos oscuros.

### Colores funcionales

Define también colores específicos para estados de la interfaz:

* Éxito: verde sobrio, claramente legible.
* Advertencia: ámbar.
* Error: rojo elegante, con contraste suficiente.
* Información: azul neutro.
* Texto principal en fondo claro: negro o gris muy oscuro.
* Texto principal en fondo oscuro: crema o blanco.
* Texto secundario: gris con contraste adecuado.

No inventes una nueva paleta para cada pantalla. Los colores deben utilizarse mediante variables o tokens de diseño reutilizables.

Ejemplo conceptual:

```css
:root {
  --color-brand-gold: #D4AF7C;
  --color-brand-black: #0B0B0B;
  --color-brand-dark: #1F1F1F;
  --color-brand-gray: #6B6B6B;
  --color-brand-cream: #F8F8F8;
}
```

El oro debe utilizarse como **acento**, no como color predominante de todos los componentes. Evita fondos dorados extensos y combinaciones que dificulten la lectura.

---

## 4. Logotipo y símbolo de marca

El logotipo de StetikShop está inspirado en una letra **S** estilizada, con una forma fluida semejante a una cinta o bolsa de compras elegante, acompañada de un asa.

El símbolo debe transmitir:

* Belleza.
* Movimiento.
* Estilo.
* Comercio.
* Exclusividad.
* Identidad propia.

El nombre debe escribirse exactamente así:

**StetikShop**

Utiliza preferiblemente:

* `Stetik` en crema o blanco.
* `Shop` en oro principal.

El símbolo puede utilizarse de forma independiente como icono de la aplicación, favicon, avatar, botón de inicio o elemento de marca.

### Reglas del logotipo

* No deformar el símbolo.
* No cambiar arbitrariamente sus proporciones.
* No utilizar demasiados efectos de brillo.
* No colocar el logotipo sobre fondos con poco contraste.
* No crear versiones diferentes en cada pantalla.
* Utilizar una versión clara para fondos oscuros y una versión oscura para fondos claros.
* Mantener una zona de protección alrededor del logotipo.
* Si existe un archivo oficial del logotipo, utilizarlo en lugar de redibujarlo.

La identidad debe poder reconocerse incluso cuando solo se muestra el símbolo “S”.

---

## 5. Tipografía

Utiliza **Poppins** como tipografía principal de la interfaz.

Jerarquía recomendada:

* Títulos principales: Poppins, peso 600 o 700.
* Títulos de secciones: Poppins, peso 600.
* Texto normal: Poppins, peso 400.
* Botones: Poppins, peso 500 o 600.
* Etiquetas y metadatos: Poppins, peso 400 o 500.

La tipografía debe ser:

* Clara.
* Moderna.
* Geométrica.
* Elegante.
* Fácil de leer en dispositivos móviles y ordenadores.

Evita utilizar demasiadas familias tipográficas. Como máximo, utiliza una tipografía complementaria únicamente si existe una justificación visual clara.

No utilices letras excesivamente ornamentales para textos funcionales, formularios, precios, citas o información importante.

---

## 6. Estilo de la interfaz

La interfaz debe combinar dos ambientes visuales:

### A. Experiencia de marca

Utiliza fondos oscuros, negro mate, gris carbón, crema y detalles dorados para:

* Pantalla de bienvenida.
* Splash screen.
* Logotipo.
* Elementos de branding.
* Cabeceras especiales.
* Secciones promocionales.
* Componentes destacados.
* Experiencias de presentación de productos premium.

### B. Experiencia funcional

Utiliza una composición clara, limpia y ordenada para:

* Catálogo de productos.
* Búsqueda.
* Categorías.
* Reserva de citas.
* Selección de horarios.
* Gestión de turnos.
* Formularios.
* Paneles de negocio.
* Panel de administración.
* Tablas y estadísticas.
* Notificaciones.

Los fondos claros pueden utilizar crema, blanco y grises muy suaves. Los fondos oscuros deben utilizar negro y gris carbón. Ambos ambientes deben sentirse parte de la misma marca.

No es necesario que todas las pantallas sean completamente negras. La identidad de StetikShop debe mantenerse mediante sus colores, tipografía, iconografía, espaciado y componentes.

---

## 7. Componentes visuales

### Botones

Diseña botones con bordes redondeados y una apariencia premium.

Botón primario:

* Fondo oro `#D4AF7C`.
* Texto oscuro para mantener contraste.
* Peso tipográfico 500 o 600.
* Bordes suaves.
* Estados hover, focus, disabled y loading.
* Transiciones discretas.

Botón secundario:

* Fondo transparente o superficie oscura.
* Borde fino dorado o neutro.
* Texto crema, blanco o dorado según el contraste.
* Apariencia elegante y menos dominante.

Botón terciario:

* Sin fondo fuerte.
* Texto de marca o texto neutro.
* Utilizar para acciones secundarias.

Los botones deben ser cómodos para tocar en móviles. No utilizar botones pequeños o difíciles de identificar.

### Tarjetas

Las tarjetas de productos, servicios, negocios y citas deben tener:

* Bordes redondeados.
* Espaciado interno generoso.
* Jerarquía visual clara.
* Imágenes de buena calidad.
* Bordes sutiles.
* Sombras suaves cuando sean necesarias.
* Información bien organizada.
* Estados de carga y vacío.

Las tarjetas premium pueden utilizar superficies oscuras y pequeños detalles dorados. Las tarjetas funcionales deben priorizar la legibilidad.

### Inputs y formularios

Los campos de entrada deben tener:

* Etiquetas claras.
* Bordes discretos.
* Radio consistente.
* Estados de foco visibles.
* Mensajes de error comprensibles.
* Iconos solo cuando aporten valor.
* Contraste suficiente.
* Tamaño apropiado para móviles.

Evita formularios visualmente recargados.

### Iconos

Utiliza iconos lineales, minimalistas y consistentes.

Los iconos deben tener:

* Grosor uniforme.
* Tamaño coherente.
* Estilo moderno.
* Color crema, gris, negro o dorado según el contexto.
* Buena alineación con el texto.

Evita mezclar iconos 3D, emojis, iconos de estilos incompatibles o ilustraciones aleatorias.

---

## 8. Elementos gráficos distintivos

La línea gráfica puede incorporar detalles sutiles inspirados en la marca:

* Líneas curvas doradas.
* Formas fluidas inspiradas en la “S” del logotipo.
* Arcos y ondas suaves.
* Líneas finas de separación.
* Gradientes muy discretos.
* Destellos dorados limitados.
* Patrones abstractos relacionados con belleza y movimiento.
* Formas geométricas minimalistas.
* Fondos con textura muy ligera, cuando sea apropiado.

Estos elementos son decorativos y no deben dificultar la navegación ni competir con el contenido.

**No utilizar:** exceso de neón, fondos saturados, efectos de cristal exagerados, sombras muy fuertes, animaciones constantes, brillos excesivos o diseños que parezcan una aplicación de videojuegos.

---

## 9. Imágenes y fotografía

La fotografía debe reflejar una estética premium de belleza.

Para productos:

* Fondos limpios.
* Iluminación suave.
* Composición editorial.
* Detalles del producto claramente visibles.
* Tonos crema, negro, beige y dorado cuando sean apropiados.
* Imágenes consistentes entre sí.

Para negocios:

* Fotografías auténticas y profesionales.
* Espacios de belleza bien iluminados.
* Salones, barberías, spas y centros de estética.
* Imágenes que transmitan confianza.

Para banners:

* Composiciones elegantes.
* Espacio suficiente para el texto.
* No colocar texto sobre zonas visualmente complejas.
* Mantener coherencia con la paleta de marca.

No generar imágenes artificiales que oculten información importante del producto ni utilizar fotografías decorativas donde se necesita una imagen real del negocio o servicio.

---

## 10. Navegación y experiencia de usuario

La navegación debe sentirse sencilla, moderna y fluida.

En dispositivos móviles:

* Utilizar una barra de navegación inferior cuando corresponda.
* Iconos claros.
* Etiquetas legibles.
* Espaciado cómodo.
* Acciones importantes fáciles de alcanzar.
* No saturar la pantalla con elementos.

En escritorio:

* Utilizar una navegación lateral o superior según el contexto.
* Mantener el contenido centrado.
* Aprovechar el espacio disponible sin estirar excesivamente las tarjetas.
* Mantener una jerarquía clara entre navegación, contenido y acciones.

La interfaz debe adaptarse correctamente a distintos tamaños de pantalla.

No sacrificar la usabilidad por conseguir una apariencia visualmente sofisticada.

---

## 11. Pantallas que deben respetar la identidad

Aplica este sistema de diseño a todas las áreas de StetikShop:

### Cliente

* Inicio.
* Explorar negocios.
* Categorías de belleza.
* Catálogo de productos.
* Detalle del producto.
* Detalle del servicio.
* Reserva de citas.
* Selección de horarios.
* Cola de espera.
* Mi agenda.
* Mis turnos.
* Notificaciones.
* Perfil de usuario.

### Negocio

* Dashboard.
* Perfil público del negocio.
* Gestión de servicios.
* Gestión de empleados.
* Horarios.
* Disponibilidad.
* Citas.
* Cola de clientes.
* Estadísticas.
* Configuración.
* Gestión de imágenes.

### Administración

* Dashboard general.
* Usuarios.
* Negocios.
* Roles y permisos.
* Estadísticas.
* Configuración del sistema.

Cada área puede tener pequeñas diferencias de densidad y organización, pero todas deben utilizar la misma identidad visual.

---

## 12. Reglas técnicas para implementar el diseño

Si el proyecto utiliza React, TypeScript y Tailwind CSS, crea un sistema de diseño reutilizable.

Debes:

1. Definir los colores de marca como tokens o variables.
2. Crear componentes reutilizables para botones, inputs, tarjetas, modales, badges, navegación y encabezados.
3. Mantener una escala de espaciado consistente.
4. Mantener radios de borde coherentes.
5. Definir estados hover, focus, active, disabled, loading y error.
6. Utilizar una jerarquía tipográfica uniforme.
7. Crear variantes claras y oscuras cuando sean necesarias.
8. Evitar estilos duplicados entre pantallas.
9. Mantener una estructura responsive.
10. Garantizar que el texto sea legible y que los controles sean accesibles.
11. No modificar la lógica de negocio existente sin una razón funcional.
12. No eliminar funcionalidades para mejorar únicamente la estética.
13. Mantener intactas las reglas de permisos, reservas, turnos, horarios y estados de la aplicación.
14. Separar claramente los componentes visuales de la lógica de negocio.

Si ya existe un sistema de componentes, extiéndelo en lugar de crear estilos incompatibles.

---

## 13. Animaciones y microinteracciones

Las animaciones deben ser suaves, rápidas y discretas.

Se permiten:

* Aparición suave de tarjetas.
* Transiciones de botones.
* Cambios de estado.
* Animaciones de carga.
* Transiciones de navegación.
* Pequeños movimientos de elementos decorativos.

Evita:

* Animaciones permanentes.
* Excesivos efectos de zoom.
* Rebotes constantes.
* Transiciones que retrasen acciones.
* Animaciones que dificulten la accesibilidad.

Respeta la preferencia del usuario por reducir el movimiento.

---

## 14. Resultado visual esperado

Quiero que StetikShop se perciba como una **plataforma premium de belleza y cuidado personal**, con una estética moderna, refinada y tecnológica.

La combinación principal debe ser:

**Negro carbón + crema + oro elegante + tipografía Poppins + formas fluidas + espacios limpios + componentes redondeados.**

La aplicación debe comunicar:

> “Más que productos, es una experiencia de belleza.”

Cada pantalla debe parecer parte de la misma marca, desde el icono de la aplicación hasta el dashboard administrativo.

Antes de implementar cualquier pantalla, analiza su propósito, define su jerarquía visual y aplica este sistema de diseño. Si una decisión visual entra en conflicto con la accesibilidad o la facilidad de uso, prioriza la accesibilidad y la facilidad de uso sin perder la identidad de StetikShop.

**No quiero que inventes otra identidad visual. Quiero que desarrolles StetikShop respetando estrictamente esta línea gráfica.**


## 15. DISEÑO RESPONSIVE MULTIDISPOSITIVO — REQUISITO OBLIGATORIO

StetikShop debe ser una aplicación completamente responsive, funcional y visualmente consistente en todos los dispositivos.

La línea gráfica de la marca debe mantenerse en móviles, tablets, laptops, monitores de escritorio y pantallas de alta resolución. Sin embargo, la distribución, el tamaño de los componentes, la navegación y la densidad de información deben adaptarse al espacio disponible.

### Enfoque de implementación

Utiliza un enfoque **mobile-first** con React, TypeScript y Tailwind CSS, aprovechando los breakpoints responsive del proyecto.

Como referencia inicial, utiliza los siguientes rangos:

* Extra pequeño: 320–479 px.
* Móvil: 480–767 px.
* Tablet: 768–1023 px.
* Laptop: 1024–1279 px.
* Desktop: 1280–1535 px.
* Pantallas grandes: 1536 px o más.

Estos rangos son orientativos. La interfaz debe adaptarse al ancho real disponible y no depender exclusivamente del tipo de dispositivo.

### Reglas obligatorias

1. Ninguna pantalla debe generar desplazamiento horizontal involuntario.
2. Ningún texto, botón, formulario, tarjeta, tabla o imagen debe quedar cortado.
3. Las tarjetas deben adaptarse de manera fluida al ancho disponible.
4. Los tamaños de fuente, espacios y columnas deben ajustarse cuando sea necesario.
5. Las imágenes deben conservar proporciones correctas y utilizar dimensiones adaptables.
6. Los botones deben ser cómodos de utilizar con los dedos en dispositivos táctiles.
7. Los formularios deben funcionar correctamente en pantallas pequeñas.
8. Las tablas y dashboards complejos deben contar con una solución responsive adecuada, como columnas reorganizadas, tarjetas, desplazamiento horizontal localizado o vistas alternativas.
9. Los modales, menús desplegables, selectores de horarios y calendarios deben caber correctamente en la pantalla.
10. No utilizar alturas fijas que oculten contenido importante.
11. No depender únicamente de hover para mostrar información o ejecutar acciones.
12. Mantener visibles y accesibles las acciones principales en móviles.
13. No utilizar breakpoints arbitrarios que rompan el diseño entre tamaños intermedios.
14. Mantener contraste, legibilidad, accesibilidad y consistencia visual en todos los tamaños.

### Adaptación por dispositivo

#### Móvil

* Utilizar una composición de una columna cuando sea necesario.
* Implementar navegación inferior para el área del cliente cuando corresponda.
* Adaptar la navegación del negocio y la administración a un menú accesible.
* Utilizar tarjetas apiladas para estadísticas, citas, servicios y productos.
* Ajustar el tamaño de los botones y campos de entrada.
* Evitar paneles excesivamente densos.
* Priorizar las acciones más importantes.
* Permitir desplazamiento localizado en contenido que realmente lo necesite.

#### Tablet

* Utilizar una o dos columnas según el ancho disponible.
* Aprovechar el espacio adicional sin crear una interfaz demasiado dispersa.
* Adaptar la navegación y los paneles laterales.
* Mantener una buena proporción entre imágenes, texto y controles.
* Permitir que dashboards, catálogos y formularios se reorganizen correctamente.

#### Laptop y desktop

* Utilizar una navegación lateral o superior según el módulo.
* Mostrar dashboards de varias columnas cuando el espacio lo permita.
* Utilizar grids de productos y servicios adaptables.
* Mantener el contenido dentro de un ancho máximo razonable.
* Evitar que los elementos se estiren excesivamente en monitores muy anchos.
* Aprovechar el espacio disponible para mostrar información relacionada sin saturar la interfaz.
* Adaptar las tablas, calendarios y paneles de gestión al ancho real.

### Componentes que deben ser responsive

Comprueba específicamente la adaptación de:

* Splash screen y pantalla de bienvenida.
* Login, registro y recuperación de contraseña.
* Página de inicio.
* Búsqueda y categorías.
* Catálogo de productos.
* Detalle de producto.
* Perfil de negocio.
* Detalle de servicios.
* Reserva de citas y selección de horarios.
* Cola de espera y posición del cliente.
* Agenda y calendario.
* Notificaciones.
* Perfil de usuario.
* Dashboard del negocio.
* Gestión de empleados.
* Gestión de servicios.
* Horarios y disponibilidad.
* Estadísticas y gráficos.
* Panel administrativo.
* Modales, formularios, menús y mensajes de error.

### Pruebas responsive obligatorias

Antes de considerar terminada cada pantalla, verifica su comportamiento en:

* 320 × 568 px.
* 360 × 800 px.
* 390 × 844 px.
* 414 × 896 px.
* 768 × 1024 px.
* 1024 × 768 px.
* 1280 × 720 px.
* 1440 × 900 px.
* 1920 × 1080 px.

También verifica anchos intermedios para detectar problemas que no aparecen en los breakpoints principales.

Comprueba especialmente:

* Ausencia de desbordamiento horizontal.
* Texto legible.
* Botones accesibles.
* Imágenes correctamente ajustadas.
* Menús funcionales.
* Formularios completos.
* Tablas y dashboards utilizables.
* Correcta visualización de estados de carga, vacío y error.
* Coherencia de colores, tipografía y espaciado.

### Regla de calidad

No consideres una pantalla terminada solamente porque se vea bien en desktop.

Una pantalla solo estará terminada cuando funcione correctamente en móvil, tablet y escritorio, manteniendo la identidad visual de StetikShop y sin sacrificar la experiencia de usuario.

**La prioridad será: funcionalidad + responsive + accesibilidad + identidad visual premium.**
