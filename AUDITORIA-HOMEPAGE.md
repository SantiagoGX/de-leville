# Auditoría Completa del Sitio — De Leville
_Actualizado: 2026-04-29 — Cubre homepage, PDP, colecciones, páginas informacionales y sistema visual_

---

## Marco Estratégico

De Leville no es un e-commerce de conversión rápida. Es una maison. El comprador objetivo no responde a urgencia ni descuento — responde a certeza, proveniencia y silencio editorial. Las recomendaciones de esta auditoría están calibradas para ese perfil: nada de countdown timers, nada de "¡Últimas unidades!", nada que Zara haría. El objetivo es remover fricción silenciosa y elevar cada punto de contacto al nivel de la promesa de marca.

---

## Hallazgos Críticos — Requieren Atención Inmediata

Estos no son problemas de optimización. Son errores activos que contaminan la experiencia o la marca ahora mismo.

### C1 — Contenido de otra marca en producción
`templates/page.destacado-en-redes.json` existe y es accesible públicamente. Contiene un carrusel de influencers con productos de otra marca: leggings, handles como `@cynthiamnv`, `@micaabaca`, y copy en español sobre ropa deportiva. Si este template está vinculado a cualquier URL del sitio De Leville, un comprador puede llegar a él. Verificar si está indexado y eliminarlo o desvincular la URL de inmediato.

### C2 — El color de otra marca (#2A4432) está activo en producción
El verde bosque `#2A4432` es el color de marca de KWARA (cliente previo del mismo tema). Está hardcodeado en CSS activo — no solo en defaults del schema — en al menos 8 secciones: `snc-product-carousel.liquid`, `snc-related-grid.liquid`, `snc-blog.liquid`, `snc-contact-form.liquid`, `snc-footer.liquid`, `snc-icon-grid.liquid`. También está en `header-group.json` como valor activo de colores del carrito. Está renderizando en el sitio ahora mismo en los componentes donde el editor no lo ha sobreescrito explícitamente.

### C3 — El carrito standalone (`/cart`) no tiene estilos
`sections/cart.liquid` es el skeleton base de Shopify: una tabla HTML sin CSS de marca, sin tipografía, sin ningún elemento De Leville. El side cart sí está elaborado, pero cualquier usuario que llegue a `/cart` directamente (desde un email, un link externo, el botón de atrás) ve una tabla genérica de Shopify. Esto es el equivalente a entrar a la boutique y que la caja sea de IKEA.

### C4 — Add-to-cart no abre el side cart
El handler del PDP hace un POST a `/cart/add.js` exitosamente pero no dispara la apertura del side cart. El comprador recibe solo un toast en la parte inferior de la pantalla y nada más. No hay confirmación visual de qué se agregó, qué talla se seleccionó, ni un path claro a checkout. La secuencia add → review → checkout está rota.

### C5 — Strings en español hardcodeados en el JS de producción
En `snc-sections.js` (~línea 3362), el toast de confirmación del PDP dice "Agregado al carrito" — hardcodeado en español, no en el sistema de localización. En `snc-pdp-hero.liquid` línea 2078, el badge de descuento dice "Ahorra X%" — también en español. Ambos aparecerán en el sitio en inglés.

### C6 — El announcement bar está en español, en MXN, y es promocional
El texto activo del announcement bar es `"ENVÍO GRATIS DESDE $2,000 MXN."` — tres violaciones simultáneas: idioma incorrecto para un sitio en inglés con posicionamiento en NY/internacional, moneda incorrecta para un comprador no-mexicano, y tono promocional que contradice directamente el "no grita, susurra".

### C7 — Títulos de colección pueden renderizarse en Times New Roman
En `snc-plp.liquid` líneas 1666–1675 hay un bloque de override tardío que fuerza `font-family: 'Cormorant Garamond', 'Times New Roman', serif` para el título de colección. Cormorant Garamond no está cargada en ningún `@font-face` del tema. El browser fallback es Times New Roman. Este es el template genérico `collection.json` que sirve cualquier colección sin template propio.

---

## Sistema Visual — Incoherencias Activas

### V1 — Conflicto de carga tipográfica (dos sistemas en guerra)
El tema tiene dos mecanismos paralelos cargando Futura PT:
- `critical.css` carga pesos 500/600/700/800 desde Typekit CDN (`use.typekit.net`) — requiere un kit activo con el dominio autorizado. Si el kit no está activo, estos pesos fallan silenciosamente.
- `theme.liquid` carga pesos 400 y 500 como `.woff` self-hosted desde Shopify CDN (variantes Cirílicas: `FuturaCyrillicBook.woff`, `FuturaCyrillicMedium.woff`).
- El Shopify font picker tiene `funnel_sans_n4` (Funnel Sans) como fuente activa. Cualquier componente que use `var(--font-primary--family)` en lugar de `var(--font-main)` o `'Futura PT'` directo renderizará en Funnel Sans. Este conflicto ya existe en producción.

### V2 — `foreground_color` no está definido en `settings_data.json`
El default del schema es `#333333` — un gris medio. Todo texto que use `var(--color-foreground)` renderiza en gris en lugar del near-black de marca (`#131313`/`#1A1A1A`). Afecta body text global.

### V3 — Scheme-3 tiene texto invisible
`settings_data.json`: scheme-3 tiene `background: #000000` pero `title_color: #111111` y `subtitle_color: #333333`. Near-black sobre negro — invisible. Este scheme está activo como opción y cualquier sección que lo use accidentalmente no mostrará texto.

### V4 — `main_title_color` global es blanco sobre fondo blanco
`settings_data.json` línea 140: `main_title_color: #ffffff` con `background_color: #ffffff`. El CSS variable global de color de título principal produce texto blanco sobre blanco. Las secciones que lo usan dependen de overrides locales para ser legibles — el sistema global está roto.

### V5 — Cormorant Garamond hardcodeada en páginas editoriales clave
Las secciones `snc-the-maison-page.liquid`, `snc-pelletteria-page.liquid`, `snc-product-care-page.liquid`, y `dl-scroll-scrub.liquid` usan Cormorant Garamond italic (weight 300) para títulos, párrafos y firmas. Ninguna de estas fuentes está cargada. El resultado es Times New Roman en las páginas más editoriales del sitio — exactamente las páginas donde la marca susurra su herencia. The Maison renderizando en Times New Roman.

### V6 — Font weight 600+ en secciones de producción
El brand spec dice weight 400 exclusivamente. Hay `font-weight: 600` hardcodeado en: `snc-side-cart.liquid` (nombres de producto y totales), `snc-pdp-hero.liquid` (elementos del PDP), `snc-header.liquid`, `snc-search-results.liquid`, `snc-help-center.liquid`, páginas de login/register/account. El peso 600 no tiene fuente cargada — activa synthetic bold del browser.

### V7 — El motivo del triángulo (△) no existe en ningún archivo
El diseño system declara el triángulo como el motivo gráfico central de la marca — aparece en los anillos, en el logo, en divisores editoriales. Búsqueda exhaustiva: cero instancias de `△`, `▲`, `&#9651;`, `&#x25B3;`, o cualquier implementación CSS de triángulo en ninguna sección, snippet o asset del tema. El símbolo más definitorio de la marca no tiene presencia digital.

### V8 — El corazón de wishlist activo es rojo saturado
`settings_data.json` línea 90: `global_wishlist_active_color: "#e00707"`. Es el único acento de color no-negro/blanco en todo el sistema. Un corazón rojo brillante sobre fotografía de joyería dorada en fondo blanco es visualmente disruptivo y semánticamente promocional. No es aristocrático.

### V9 — Defaults de schema con DNA de otra marca
`settings_schema.json` línea 98: `"default": "KWARA XL"` como logo text. `snc-footer.liquid` líneas 1082 y 1190: `"default": "KWARA"`. `snc-contact-form.liquid` línea 549: `"title": "hello@kwara.mx"` en schema preset. Un reset de settings desde el editor restaura el nombre de otra marca.

### V10 — Favicon es un screenshot de macOS
`settings_data.json` línea 12: el favicon activo es `Captura_de_pantalla_2026-04-23_a_la_s_6.53.33_p.m..png`. Visible en cada pestaña del browser para cada visitante del sitio.

---

## PDP — Gaps de Experiencia de Compra

### P1 — Digital Soul está enterrado donde nadie lo verá
El diferenciador más fuerte de la marca — el único en su categoría — está en el quinto ítem de una pila de acordeones colapsados, con color de título `#767676` (el gris más débil del sistema). La jerarquía visual comunica que es lo menos importante de la página. Cartier pone su garantía de autenticidad antes del precio. De Leville pone su certificado blockchain detrás de cuatro clics.

La solución no es moverlo de acordeón — es sacarlo del acordeón completamente. Un elemento visual fijo entre el precio y el CTA, breve, con el sello triangular y una línea: "Every piece carries a Digital Soul — blockchain-authenticated, permanently yours." Con link a `/pages/digital-soul`.

### P2 — "Made to order" no existe encima del CTA
El campo `made_to_order_text` existe en el schema y tiene soporte completo en la sección. Está intencionalmente en blanco. El único lugar donde el comprador descubre que el producto es made-to-order es en el acordeón de Delivery & Returns — si lo abre. Un comprador que no lo abre agrega un anillo al carrito sin saber que el tiempo de entrega no es "3-5 días hábiles". Esto genera expectativas incorrectas y potencialmente chargebacks.

Para una marca de lujo, "made to order" no es un disclaimer — es parte de la propuesta de valor. "Crafted to order — allow 3–4 weeks" debajo del precio, en Futura PT Book 12px, comunica exclusividad, no inconveniencia.

### P3 — El selector de talla no controla variants de Shopify
El UI de selección de talla es un componente visual custom que escribe el valor como line-item property (`Size: 7`), no como variant selection. Shopify no puede rastrear disponibilidad por talla. Cualquier talla siempre estará "disponible" independientemente del stock real. Para made-to-order esto puede ser aceptable operativamente, pero si alguna talla requiere materiales especiales o tiempos distintos, no hay mecanismo para comunicarlo.

### P4 — No hay zoom, lightbox, ni soporte de video en el PDP
El loop de media del PDP filtra exclusivamente `media_type == 'image'` (`snc-pdp-hero.liquid` línea 1977). Videos y modelos 3D subidos en Shopify son ignorados. No hay zoom al click, no hay lightbox. Para un anillo de 18K gold con grabado interior, los compradores necesitan ver el detalle. La fotografía macro de la textura del band, la transición de oro blanco a amarillo, el grabado — nada de eso es explorable actualmente.

### P5 — No hay navegación de imágenes en desktop
En desktop, las imágenes del PDP son una columna sticky vertical sin contador, sin thumbnails, sin dots. Un producto con 6 fotografías no da ninguna indicación de cuántas hay ni permite saltar a un ángulo específico.

### P6 — El certificado de autenticidad no tiene presencia visual
El acordeón de Specifications menciona "Certificate of authenticity" en texto plano. No hay fotografía del certificado, no hay imagen del sello. Van Cleef muestra el certificado. Bulgari muestra el estuche y el certificado. De Leville lo lista en un bullet.

### P7 — El packaging no está en ninguna parte del PDP
"De Leville signature packaging" aparece como texto en el acordeón de Specifications. No hay una sola foto del estuche, el ribbon, el sobre, la bolsa. Para joyería de lujo a este precio, el unboxing es parte de la decisión de compra — especialmente si es un regalo.

### P8 — "Related Products" como copy activo en producción
La sección "Complete the Look" al final del PDP tiene `sub_text: "Related Products"` en `product.json` línea 158 — un placeholder que nunca fue actualizado. Está visible en cada página de producto.

### P9 — "Ahorra X%" viola el tono de marca y está en español
`snc-pdp-hero.liquid` línea 2078: el badge de precio rebajado usa "Ahorra X%". Aunque De Leville no haga descuentos ahora, si algún producto tiene compare-at-price, este texto aparecerá. La frase es de e-commerce masivo y está en el idioma incorrecto.

---

## Colecciones — Gaps de Experiencia de Descubrimiento

### CO1 — El template genérico `collection.json` es una falla de marca total
Cualquier colección sin template propio (bags, chains, o cualquier nueva colección futura) sirve `collection.json` con `snc-plp`: sin hero editorial, con barra de filtros para 3 productos, con sort dropdown, con contador "X products". Un comprador que llega a `/collections/bags` ve exactamente lo que vería en cualquier Shopify genérico.

### CO2 — Filtros y sort para colecciones de 3–12 SKUs
Los filtros de `snc-plp` se renderizan incondicionalmente. Para colecciones con menos de 20 productos y sin atributos filtrables significativos, la barra de filtros es ruido. Louis Vuitton no muestra "Sort by: Price, Low to High" en su colección de joyería.

### CO3 — Mezza Luna tiene `border_radius: 12` en el hero
`collection.mezza-luna.json` línea 22: el hero fullscreen de Mezza Luna tiene esquinas redondeadas. Classico e Incontro tienen `border_radius: 0`. El efecto full-bleed se rompe en Mezza Luna — hay un gap visible entre el edge del browser y la imagen.

### CO4 — "Made to order" ausente en todos los niveles de colección
Ninguna de las tres páginas de colección (Classico, Incontro, Mezza Luna) comunica que los productos son made-to-order antes del grid. La página de lista de colecciones sí lo dice: "Each piece numbered, made to order." — pero desaparece al entrar a cualquier colección.

### CO5 — Estado vacío muestra placeholders con "$999.00"
`snc-mixed-grid.liquid` líneas 700–714: cuando una colección no tiene productos, el grid renderiza SVG placeholders con "Product example 1" y precios hardcodeados "$999.00". Si una colección está temporalmente sin stock o deshabilitada, los compradores ven esto.

### CO6 — Dos templates competidores para Incontro
`collection.incontro.json` y `collection.incontro-preview.json` son dos rutas hacia la misma colección con stacks de secciones diferentes. El preview tiene `dl-scroll-scrub` con el texto poético animado: "You are not wearing a ring. You are wearing something he wrote in 1975." Este momento editorial existe solo en el template de preview. El URL canónico de Incontro no lo tiene.

### CO7 — Sin navegación cross-colección
Al final de cualquier página de colección named (Classico, Incontro, Mezza Luna), el scroll termina. No hay link hacia las otras colecciones, no hay "Explore THE MAISON", no hay continuidad de recorrido. El comprador tiene que usar el header para ir a otra colección.

### CO8 — Digital Soul CTA deshabilitado en todas las colecciones
`show_cta: false` en todas las instancias de `snc-scroll-gallery` dentro de colecciones. La galería animada de Digital Soul termina sin un path de acción.

---

## Páginas Informacionales — Gaps

### I1 — `page.about.json` es un shell vacío
Mismo tipo de sección que `page.the-maison.json` (`snc-the-maison-page`) pero con todos los campos de copy en blanco: `origins_text`, `middle_text`, `today_text`, `heritage_text`, `close_text`. Si `/pages/about` está en el nav, los compradores ven una página con imágenes pero sin texto narrativo.

### I2 — Digital Soul necesita un flujo operacional claro
La página explica el qué y el por qué. No explica el cómo en formato escaneable: qué pasa exactamente después de la compra, cuándo, qué hace el comprador, dónde. El FAQ lo responde pero está colapsado. Un comprador no-crypto necesita ver "1. Compras. 2. En 30 días recibes un email. 3. Haces clic y el certificado es tuyo. Permanente." antes del FAQ, no dentro de él.

### I3 — "De Leville Circle" mencionado sin página ni programa
La página Digital Soul menciona el Circle como un concepto de comunidad/lealtad pero no hay link, no hay página, no hay información. Introduce una expectativa que no se puede satisfacer actualmente.

### I4 — Captions "Joya" en la página Digital Soul
Múltiples bloques de imagen en `page.nft.json` tienen captions que dicen "Joya" — español genérico, sin relación con las piezas específicas que se muestran. Visualmente correcto pero editorialmente vacío.

### I5 — Página de Contact comparte imagen con otra página
`page.contact.json` y la página de Digital Soul usan `Img_7.png` como hero. Dos páginas con identidades completamente distintas abriendo con la misma fotografía.

### I6 — Páginas ausentes que una maison necesita
- **Press / Editorial:** No existe. Para posicionarse junto a Cartier y Hermès, "As Seen In" o una página de prensa es una señal de legitimidad que el comprador busca antes de una compra de 4 cifras.
- **Cuidado y mantenimiento en el PDP:** Existe como página standalone (`/pages/product-care`) pero no está referenciada ni vinculada desde el PDP.
- **Guía de tallas interactiva:** La selección de talla en el PDP es manual — no hay guía visual de medición.
- **Bespoke / Encargo personalizado:** Made-to-order es core a la marca pero no hay flujo de intake para comisiones especiales.
- **Verificación de autenticidad para mercado secundario:** El Digital Soul cubre esto técnicamente, pero no hay página de "¿Cómo verificar que tu pieza De Leville es auténtica?"

### I7 — Duplicate OG meta tags
`snippets/meta-tags.liquid` y `theme.liquid` ambos generan `og:site_name`, `og:type`, `og:title`, `og:description`, `og:url`, y `twitter:card`. Los crawlers de redes sociales y los validadores de schema los reportarán como duplicados. Además, en páginas informacionales (The Maison, Digital Soul, Contact) `og:image` será null si `page_image` no está seteado — shares en redes sociales sin imagen.

### I8 — Footer tiene "KWARA" como default de copyright
El campo `signature_logo` del footer tiene `"default": "KWARA"` en el schema. Si el campo no ha sido editado en el Shopify admin, el footer puede mostrar el nombre de otra marca en el copyright.

### I9 — Newsletter y footer con idioma mixto
El placeholder del campo de email en el newsletter dice "Ingresa tu email" (español). El mensaje de confirmación dice "Successfully subscribed!" (inglés). Un mismo componente, dos idiomas.

---

## Recomendaciones Estratégicas

Estas están ordenadas por impacto y calibradas para una marca de lujo — no son tácticas de conversión masiva. Son acciones para que cada punto de contacto esté al nivel de la promesa.

---

### NIVEL 1 — Remover lo que no pertenece

**R1.1 — Purga de KWARA**
Reemplazar todos los defaults `#2A4432`, `"KWARA"`, `"KWARA XL"`, y `"hello@kwara.mx"` en schema y en CSS hardcodeado. Auditar `header-group.json` por los valores activos del carrito en verde. Esta no es una tarea de diseño — es una tarea de limpieza de código.

**R1.2 — Eliminar o aislar `page.destacado-en-redes.json`**
Verificar si tiene URL pública. Si la tiene, redirigir a 404 o a homepage ahora mismo.

**R1.3 — Corregir el announcement bar**
Reemplazar `"ENVÍO GRATIS DESDE $2,000 MXN."` con copy en inglés que comunique valor sin promotion: `"Made to order. Shipped worldwide."` o `"Born in Rome, 1975."` o simplemente dejarlo vacío hasta tener copy on-brand.

**R1.4 — Corregir strings en español del JS**
`snc-sections.js` línea 3362: cambiar "Agregado al carrito" → "Added to your bag". `snc-pdp-hero.liquid` línea 2078: eliminar el badge "Ahorra X%" o reemplazarlo con una alternativa neutral o sin texto.

---

### NIVEL 2 — Corregir el sistema visual

**R2.1 — Resolver el sistema de fuentes**
Decisión binaria: Typekit o self-hosted. Si Typekit está activo y el dominio está autorizado, usarlo como fuente primaria y eliminar el conflicto. Si no está activo, operar solo con los woff self-hosted (Book/Medium) y no referenciar pesos 600/700/800. Actualizar `type_primary_font` en settings a un valor que no sea `funnel_sans_n4`. Auditar y corregir `var(--font-primary--family)` en todos los componentes que lo usan.

**R2.2 — Eliminar Cormorant Garamond de páginas editoriales**
`snc-the-maison-page.liquid`, `snc-pelletteria-page.liquid`, `snc-product-care-page.liquid`, `dl-scroll-scrub.liquid`. Reemplazar con Futura PT. Si se busca contraste tipográfico serif/sans en The Maison, tomar la decisión explícitamente con una fuente que esté cargada.

**R2.3 — Setear `foreground_color` en settings_data.json**
Cambiar de `#333333` (no seteado, cae al default) a `#131313` o `#1A1A1A`. Todo el body text global se acerca a negro real.

**R2.4 — Corregir scheme-3 y `main_title_color`**
Scheme-3: cambiar `title_color` y `subtitle_color` a `#ffffff` para ser legibles sobre negro. `main_title_color: #ffffff` → `#131313` o manejarlo como variable que no se use globalmente.

**R2.5 — Reemplazar el corazón rojo de wishlist**
Cambiar `global_wishlist_active_color` de `#e00707` a `#000000`. Un corazón negro sólido es más coherente con el sistema y menos promotional.

**R2.6 — Corregir el favicon**
Usar el triángulo (△) o las iniciales "DL" como SVG/PNG de 32×32px. Es un cambio de 5 minutos en el Shopify admin.

---

### NIVEL 3 — Elevar el PDP al nivel de la promesa de marca

**R3.1 — Digital Soul fuera del acordeón**
Entre el precio y el botón "Add to Bag", un elemento visual fijo — no colapsable. No tiene que ser largo. Puede ser una línea con el símbolo △ y "Every piece carries a Digital Soul →". Link a `/pages/digital-soul`. El diferenciador principal de la marca debe ser visible sin interacción.

**R3.2 — Activar el campo `made_to_order_text`**
Rellenar en el Shopify editor: algo como "Crafted to order — please allow 3–4 weeks." Una línea, en Futura PT Book, gris claro, debajo del precio. Para el comprador esto es información crítica. Para la marca es una señal de exclusividad.

**R3.3 — Abrir el side cart después de add-to-cart**
En `snc-sections.js`, después de `window.updateSideCart()`, disparar el evento o función que abre el panel del side cart. El comprador necesita confirmación visual inmediata de lo que acaba de agregar — talla, nombre, precio — y un path directo a checkout.

**R3.4 — Añadir navegación de imágenes en desktop**
Un strip de thumbnails vertical en el lado izquierdo de la imagen principal, o dots numerados. Para un producto con 6 ángulos de fotografía, el comprador actual no sabe que existen.

**R3.5 — Añadir zoom o lightbox**
Click en imagen → lightbox fullscreen con zoom. Para joyería de lujo, el detalle es parte de la decisión. El grabado, la textura del band, la transición de color del oro — nada de eso es explorable actualmente.

**R3.6 — Añadir soporte de video en el media loop**
Remover el filtro `media_type == 'image'` del loop o añadir un branch para video. Un video de 15 segundos del anillo rotando bajo luz natural convierte más que 6 fotografías estáticas para compradores internacionales que no pueden ver la pieza físicamente.

**R3.7 — Fotografiar el packaging**
Una imagen del estuche De Leville — el exterior, el interior, el ribbon — en el gallery del producto. No en el acordeón. En el gallery principal, como la última imagen. Para muchos compradores esto es un regalo; el unboxing es parte de la experiencia que están comprando.

**R3.8 — Corregir "Related Products" placeholder**
`product.json` línea 158: cambiar `sub_text` de "Related Products" a copy on-brand: "From the same collection" o "You may also wear" o simplemente vacío.

---

### NIVEL 4 — Coherencia de narrativa entre páginas

**R4.1 — Agregar "made to order" al nivel de colección**
Una línea de copy encima del grid en cada página de colección named. No tiene que ser un banner — puede ser `"Each piece crafted to order"` en Futura PT Book 12px, letter-spacing 0.15em, entre el subtítulo de la colección y el primer producto.

**R4.2 — Crear un template editorial para colecciones sin página propia**
El `collection.json` genérico necesita al menos: un hero con el nombre de la colección en grande, una línea de copy, y el grid. Actualmente es el skeleton de Shopify. Bags, Chains, y cualquier colección nueva sirven esto.

**R4.3 — Eliminar filtros en colecciones pequeñas**
Condicionar la barra de filtros en `snc-plp.liquid` para que no se renderice cuando `collection.products_count < 20` (o un threshold razonable). Para 3 anillos, los filtros son ruido.

**R4.4 — Corregir el border-radius de Mezza Luna**
`collection.mezza-luna.json` línea 22: cambiar `border_radius: 12` a `border_radius: 0`. Full-bleed consistente con Classico e Incontro.

**R4.5 — Consolidar los templates de Incontro**
Decidir cuál es el canónico. El momento editorial de `dl-scroll-scrub` ("You are not wearing a ring. You are wearing something he wrote in 1975.") debe estar en el URL principal de la colección, no en el preview. Si `dl-scroll-scrub` usa Cormorant Garamond, resolver primero V5.

**R4.6 — Añadir navegación cross-colección al final de cada colección**
Al terminar el scroll de Classico, que haya una invitación a explorar Incontro o Mezza Luna. No tiene que ser un componente complejo — tres nombres en grande, en negro, centrados, con link. El comprador que terminó de ver Classico y no encontró su talla perfecta debería poder continuar explorando sin volver al header.

**R4.7 — Habilitar el CTA de Digital Soul en las páginas de colección**
`show_cta: true` en las instancias de `snc-scroll-gallery` dentro de colecciones. El comprador que terminó de ver la galería debe tener un camino a `/pages/digital-soul`.

**R4.8 — Corregir `page.about.json`**
Completar el copy del template o redirigir `/pages/about` a `/pages/the-maison`. No deben existir dos URLs distintas para el mismo contenido con uno vacío.

**R4.9 — Reemplazar el carrito standalone**
Estilizar `sections/cart.liquid` con el mismo nivel de cuidado que el side cart. O bien, redirigir `/cart` directamente a checkout y remover la página standalone — algunas marcas de lujo hacen esto deliberadamente (el carrito como estado transitorio, no como destino).

**R4.10 — OG images explícitas para páginas informacionales**
Setear metafields de imagen en Shopify admin para The Maison, Digital Soul, Contact. Actualmente los shares en redes sociales de estas páginas no tienen imagen. Para una marca que se posiciona visualmente, un share sin imagen es una oportunidad perdida.

---

### NIVEL 5 — Lo que todavía no existe pero debería

**R5.1 — Implementar el triángulo (△)**
El motivo más definitorio de la marca no aparece en ningún archivo del tema. Puede ser un SVG inline en el footer, como divisor entre secciones editoriales, como watermark en el hero de The Maison, como sello junto a Digital Soul. No requiere un rediseño — requiere que alguien lo ponga.

**R5.2 — Página de Press / Editorial**
Una página simple con logotipos de publicaciones y quotes de cobertura. Para el comprador que investiga antes de una compra de 4 cifras, "As seen in" es una señal de legitimidad. Puede construirse con `snc-custom-content` existente.

**R5.3 — Guía de tallas integrada en el PDP**
Un modal o panel deslizable desde el selector de talla con instrucciones visuales de medición en español e inglés. Para joyería hecha a pedido que no se puede devolver fácilmente, la guía de tallas reduce la fricción más que cualquier otra cosa en el PDP.

**R5.4 — Fotografía del certificado y el packaging**
No como sección nueva — como imágenes adicionales en el gallery del producto. El certificado como penúltima imagen, el packaging como última. Ya existe la infraestructura; solo falta el contenido.

**R5.5 — Comunicación de Digital Soul pre-compra en el flow**
Cuando el side cart se abre con el anillo agregado, una línea debajo del producto: "This piece will receive its Digital Soul 30 days after shipping." Pequeño, en Book weight, gris. Prepara la expectativa, diferencia, y no grita.

---

## Cambios Técnicos Realizados

### Fix 1 — Cart counter blanco sobre círculo negro (`snc-header.liquid`)
**Causa:** `.custom-header--cart-count` incluido en selectores de `force-light-theme` y `data-dl-header-mode` con `color: #000000 !important`. Los selectores con atributo tienen mayor especificidad que la regla correctora del final, ganando a pesar del `!important`.
**Fix:** Removido `.custom-header--cart-count` de 5 ocurrencias en 2 bloques de `force-light-theme` y 3 reglas `data-dl-header-mode`. La regla `background: #000 !important; color: #fff !important` al final del archivo ahora aplica correctamente.

### Fix 2 — Título y precio se ocultan en hover de product card (`snc-mixed-grid.liquid`)
**Requerimiento:** Durante el hover (que muestra la segunda imagen), el nombre y precio del producto deben ocultarse. Al quitar el hover, regresan.
**Fix:** Añadido `transition: opacity 0.4s ease` a `.snc-mixed-grid__card-title` y `.snc-mixed-grid__card-price` en estado base. Regla de hover: `opacity: 0` con misma duración (0.4s) que el crossfade de imagen. Mobile no afectado.

### Fix 3 — Botones y switch del side cart convertidos a píldora (`snc-side-cart.liquid`)
**Requerimiento:** Todos los botones y el switch Cart/Favourites del side cart deben tener forma de píldora, consistente con el estilo del botón Add to Bag del PDP.
**Archivos modificados:** `sections/snc-side-cart.liquid`
**Cambios:**
- `.custom-header--cart-switch` (contenedor del toggle): `border-radius: 4px` → `999px`
- `.custom-header--cart-switch-btn` (tabs Cart / Favourites): `border-radius: 4px` → `999px`
- `.custom-header--cart-empty-btn` (botones Explore / Best Sellers en carrito vacío): `border-radius: 4px` → `999px`
- `.custom-header--cart-checkout-button` (botón de checkout): `border-radius: 4px` → `999px`

**Nota:** Los botones Add to Bag y Buy Now del PDP (`snc-pdp-hero.liquid`) ya tenían `border-radius: 999px !important` — no requirieron cambio.

---

## Cambios a Realizar - Homepage Mobile (Pendientes)

### M1 — SNC Animated Collections (Reorganización Móvil)
- **Problema:** En versión móvil, la sección tiene una altura excesiva que deja las imágenes de los anillos flotando en un espacio blanco enorme. Además, el scroll es tan suave/rápido que el contenido se pierde al navegar, mostrando solo una imagen de anillo y ocultando el resto de manera ineficiente.
- **Solución:** Reestructurar la versión móvil para acercarla a la experiencia de escritorio. 
  - Debajo del título "The Collections", listar los nombres de las colecciones clásicas de forma visible.
  - Al hacer scroll, se resaltará la colección activa.
  - Debajo de los nombres, se mostrarán *dos* imágenes de anillos de la colección activa de forma simultánea.
  - Implementar una animación constante (ej. anillos rotando sobre su eje) para que haya movimiento incluso cuando el usuario no hace scroll.

### M2 — SNC Mixed Grid y Nueva Sección SNC Home Showcase
- **Problema:** La sección `snc-mixed-grid` en la página de inicio en su versión de escritorio muestra tres tarjetas de productos. Sin embargo, en móvil estas tarjetas salen cortadas/incompletas.
- **Solución:** 
  - Ajustar `snc-mixed-grid` para que en versión móvil muestre cuatro productos (o se acomoden mejor) en lugar de tres incompletos.
  - **Nueva Sección:** Duplicar `snc-mixed-grid` para crear `snc-home-showcase`. Esta sección será de uso exclusivo para el homepage y permitirá dos modos de selección manual: 1) Seleccionar productos específicos uno por uno. 2) Seleccionar una colección y mostrar los productos en el orden exacto de dicha colección.

### M3 — SNC 3D Info Section (Optimización Móvil)
- **Problema:** La primera imagen/anillo principal ocupa demasiado espacio innecesario en la versión móvil, perjudicando el flujo de lectura.
- **Solución:** Mantener la versión de escritorio intacta. En versión móvil, ocultar por completo la primera imagen del anillo. Se debe mostrar únicamente el texto en la parte superior y, justo debajo, la segunda imagen del anillo.
