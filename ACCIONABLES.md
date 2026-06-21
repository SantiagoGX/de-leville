# Accionables — De Leville Store Polish
_Generado: 2026-04-29 — Basado en AUDITORIA-HOMEPAGE.md_

> **Principio rector:** Errores activos primero, luego sistema visual, luego estructura, luego lo nuevo. Nada de timers de urgencia, nada de descuentos visibles, nada que Zara haría.

---

## CÓMO USAR ESTE DOCUMENTO

Cada acción tiene:
- **Ref:** código de la auditoría de origen
- **Effort:** XS (<5 min) · S (5–30 min) · M (30 min–2h) · L (2h+)
- **Tipo:** `Code` · `Admin` · `Decisión` · `Contenido`
- **Estado:** `Listo para ejecutar` · `Consultar primero` · `Necesita contenido`

---

## TIER 0 — ERRORES CRÍTICOS
_Están activos en producción ahora mismo. Prioridad sobre todo lo demás._

---

### T0-1 — Contenido de otra marca en producción
**Ref:** C1 | **Effort:** XS | **Tipo:** Admin | **Estado:** Listo para ejecutar

`templates/page.destacado-en-redes.json` tiene un carrusel de influencers con productos de ropa deportiva y handles ajenos. Si hay una URL pública vinculada, un comprador puede llegar a esa página.

**Fix:**
1. En Shopify Admin → Pages, buscar cualquier página asignada al template `page.destacado-en-redes`
2. Si existe: despublicar la página o cambiarle el template
3. El archivo JSON puede dejarse, pero sin ninguna página asignada no se renderiza

---

### T0-2 — Color KWARA (#2A4432) hardcodeado en CSS activo
**Ref:** C2, R1.1 | **Effort:** M | **Tipo:** Code | **Estado:** Listo para ejecutar

El verde bosque `#2A4432` es el color de KWARA y está hardcodeado en al menos 8 secciones. Se renderiza en producción donde el editor no lo ha sobreescrito.

**Fix — buscar y purgar en todos los archivos:**
```bash
grep -rn "2A4432\|2a4432" sections/ config/
```
Archivos confirmados: `snc-product-carousel.liquid`, `snc-related-grid.liquid`, `snc-blog.liquid`, `snc-contact-form.liquid`, `snc-footer.liquid`, `snc-icon-grid.liquid`, `header-group.json`.

En cada ocurrencia: reemplazar `#2A4432` con `#000000` (negro de marca) o el valor de CSS variable correspondiente.

---

### T0-3 — Add-to-cart no abre el side cart
**Ref:** C4, R3.3 | **Effort:** S | **Tipo:** Code | **Estado:** Listo para ejecutar

El handler del PDP hace POST a `/cart/add.js` pero solo muestra un toast. El side cart no se abre. La secuencia add → review → checkout está rota.

**Fix — `assets/snc-sections.js` ~línea 3362:**
Después de la llamada `window.updateSideCart()` o equivalente, disparar la apertura del side cart:
```javascript
// Después de confirmar que el add fue exitoso:
document.dispatchEvent(new CustomEvent('cart:open'));
// O la función directa que usa el header para abrir el side cart
```
Revisar cómo el header abre el side cart (click en ícono del carrito) y replicar ese mecanismo en el handler del PDP.

---

### T0-4 — Strings en español hardcodeados en JS
**Ref:** C5, R1.4 | **Effort:** XS | **Tipo:** Code | **Estado:** Listo para ejecutar

**Fix 1 — `assets/snc-sections.js` ~línea 3362:**
```
"Agregado al carrito" → "Added to your bag"
```

**Fix 2 — `sections/snc-pdp-hero.liquid` ~línea 2078:**
El badge `"Ahorra X%"` aparece si un producto tiene compare-at-price. Opciones:
- A) Eliminar el badge completamente (recomendado — De Leville no hace descuentos)
- B) Cambiarlo a texto neutral: `"—"` o eliminarlo del DOM
- C) Moverlo al sistema de localización si se quiere mantener

---

### T0-5 — Announcement bar: español + MXN + tono promo
**Ref:** C6, R1.3 | **Effort:** XS | **Tipo:** Admin | **Estado:** Consultar primero

Texto actual: `"ENVÍO GRATIS DESDE $2,000 MXN."` — tres problemas: idioma, moneda, tono.

**Opciones de reemplazo (elegir una):**
- `"Made to order. Shipped worldwide."` — comunica el diferenciador real
- `"Born in Rome, 1975."` — editorial, no funcional
- `"Each piece crafted to order."` — prepara la expectativa antes del PDP
- Vacío hasta tener copy definitivo

**Fix:** Shopify Admin → Themes → Customize → Announcement Bar, o en el archivo de settings del header.

---

### T0-6 — Títulos de colección pueden renderizar en Times New Roman
**Ref:** C7 | **Effort:** S | **Tipo:** Code | **Estado:** Listo para ejecutar

`snc-plp.liquid` líneas 1666–1675 fuerza `font-family: 'Cormorant Garamond', 'Times New Roman', serif`. Cormorant Garamond no está cargada → Times New Roman en producción.

**Fix:** En las líneas 1666–1675 de `sections/snc-plp.liquid`, reemplazar el bloque de font-family override con Futura PT:
```css
font-family: 'Futura PT', sans-serif;
```

---

## TIER 1 — SISTEMA VISUAL
_Settings rotos que afectan texto e interfaz globalmente._

---

### T1-1 — foreground_color globalmente gris (#333333)
**Ref:** V2, R2.3 | **Effort:** XS | **Tipo:** Code | **Estado:** Listo para ejecutar

**Fix — `config/settings_data.json`:**
No hay una clave `foreground_color` en el archivo actual (cae al default del schema que es `#333333`). Agregar o corregir:
```json
"foreground_color": "#131313"
```

---

### T1-2 — Scheme-3: texto invisible (near-black sobre negro)
**Ref:** V3, R2.4 | **Effort:** XS | **Tipo:** Code | **Estado:** Listo para ejecutar

**Fix — `config/settings_data.json`, scheme-3:**
```json
"title_color": "#ffffff",
"subtitle_color": "#f0f0f0"
```
El background es #000000 — títulos deben ser blancos.

---

### T1-3 — main_title_color: blanco sobre blanco
**Ref:** V4, R2.4 | **Effort:** XS | **Tipo:** Code | **Estado:** Listo para ejecutar

**Fix — `config/settings_data.json` línea 140:**
```json
"main_title_color": "#131313"
```
Actualmente `#ffffff` sobre `background_color: #ffffff`. Las secciones que lo usan son ilegibles si no tienen override local.

---

### T1-4 — Wishlist heart: rojo saturado (#e00707)
**Ref:** V8, R2.5 | **Effort:** XS | **Tipo:** Code | **Estado:** Listo para ejecutar

**Fix — `config/settings_data.json` línea 72:**
```json
"global_wishlist_active_color": "#000000"
```
Negro sobre blanco. Aristócrata, no promocional.

---

### T1-5 — Defaults KWARA en settings schema
**Ref:** V9 | **Effort:** S | **Tipo:** Code | **Estado:** Listo para ejecutar

**Fix — `config/settings_schema.json`:**
- Línea 98: `"default": "KWARA XL"` → `"default": "DE LEVILLE"`

**Fix — `sections/snc-footer.liquid`:**
- Líneas 1082 y 1190: `"default": "KWARA"` → `"default": "DE LEVILLE"`

**Fix — `sections/snc-contact-form.liquid`:**
- Línea 549: `"title": "hello@kwara.mx"` → `"title": "hello@deleville.com"` (o el email real)

---

### T1-6 — Favicon: screenshot de macOS
**Ref:** V10, R2.6 | **Effort:** XS | **Tipo:** Admin | **Estado:** Necesita contenido

El favicon actual es `Captura_de_pantalla_2026-04-23...png`.

**Fix:** Shopify Admin → Themes → Customize → Favicon
Subir: triángulo △ como SVG/PNG 32×32px, o iniciales "DL" en Futura PT Medium. Archivo de 32×32px.

---

### T1-7 — Cormorant Garamond en páginas editoriales → Times New Roman
**Ref:** V5, R2.2 | **Effort:** M | **Tipo:** Code | **Estado:** Consultar primero

Páginas afectadas: `snc-the-maison-page.liquid`, `snc-pelletteria-page.liquid`, `snc-product-care-page.liquid`, `dl-scroll-scrub.liquid`.

Todas usan `font-family: 'Cormorant Garamond'` que no está cargada → Times New Roman.

**Decisión requerida:** ¿Futura PT pura en todas estas páginas (consistencia), o se quiere un contraste serif/sans explícito? Si se elige serif, hay que cargarla vía @font-face o Google Fonts. Si se elige Futura PT pura, el fix es directo.

**Fix si Futura PT pura:**
En cada archivo, reemplazar `'Cormorant Garamond'` con `'Futura PT'`.

---

### T1-8 — font-weight 600 sin fuente cargada (synthetic bold)
**Ref:** V6 | **Effort:** M | **Tipo:** Code | **Estado:** Listo para ejecutar

El brand spec dice weight 400. Weight 600 activa synthetic bold del browser (ugly).

**Archivos a auditar y corregir:**
```bash
grep -n "font-weight: 600\|font-weight:600" sections/snc-side-cart.liquid sections/snc-pdp-hero.liquid sections/snc-header.liquid sections/snc-search-results.liquid sections/snc-help-center.liquid
```
Cambiar todas las instancias a `font-weight: 400` o `font-weight: 500` según el caso.

---

### T1-9 — Sistema de fuentes: conflicto Typekit vs self-hosted
**Ref:** V1, R2.1 | **Effort:** M | **Tipo:** Decisión + Code | **Estado:** Consultar primero

Hay tres fuentes cargando en paralelo:
- `critical.css` → Typekit CDN (pesos 500/600/700/800)
- `theme.liquid` → `.woff` self-hosted (Book 400, Medium 500)
- `settings` → `funnel_sans_n4` (Funnel Sans) como `type_primary_font`

`var(--font-primary--family)` resuelve a Funnel Sans en lugar de Futura PT.

**Decisión requerida:** ¿Typekit activo con dominio autorizado, o solo self-hosted?

**Fix si solo self-hosted (recomendado para evitar dependencia):**
1. `config/settings_data.json` línea 14: `"type_primary_font"` → cambiar `funnel_sans_n4` a un valor que no sobreescriba el self-hosted, o usar un font de Shopify como fallback visual equivalente
2. Auditar todos los usos de `var(--font-primary--family)` y reemplazar con `'Futura PT', sans-serif` donde corresponda
3. Decidir si se eliminan los imports de Typekit de `critical.css` para no depender de un kit externo

---

## TIER 2 — PDP QUICK WINS
_Errores visibles en cada página de producto._

---

### T2-1 — Placeholder "Related Products" visible en producción
**Ref:** P8, R3.8 | **Effort:** XS | **Tipo:** Code | **Estado:** Listo para ejecutar

**Fix — `templates/product.json` línea 158:**
```json
"sub_text": ""
```
O reemplazar con copy on-brand: `"From the same collection"` / `"You may also wear"`.

---

### T2-2 — Badge "Ahorra X%" en español
**Ref:** P9, C5 | **Effort:** XS | **Tipo:** Code | **Estado:** Listo para ejecutar
_(Cubierto en T0-4, Fix 2 — ejecutar en el mismo pase)_

---

### T2-3 — made_to_order_text vacío: compradores no saben que es hecho a pedido
**Ref:** P2, R3.2 | **Effort:** XS | **Tipo:** Admin + Contenido | **Estado:** Necesita contenido

El campo existe en el schema. Solo necesita ser rellenado en el Shopify editor para cada producto.

**Copy sugerido:** `"Crafted to order — please allow 3–4 weeks."`

Ubicación en el PDP: debajo del precio, antes del selector de talla. Ya tiene soporte completo en la sección — solo falta el contenido.

---

### T2-4 — Digital Soul enterrado en acordeón
**Ref:** P1, R3.1 | **Effort:** M | **Tipo:** Code | **Estado:** Consultar primero

El diferenciador más fuerte de la marca está en el 5to acordeón colapsado.

**Fix propuesto:** Un elemento no-colapsable entre el precio y el CTA en `snc-pdp-hero.liquid`:
```html
<div class="pdp-digital-soul-badge">
  △ Every piece carries a Digital Soul →
</div>
```
Link a `/pages/digital-soul`. Futura PT Book, 12px, letter-spacing 0.15em.

---

## TIER 3 — COLECCIONES

---

### T3-1 — border_radius: 12 en hero de Mezza Luna
**Ref:** CO3, R4.4 | **Effort:** XS | **Tipo:** Code | **Estado:** Listo para ejecutar

**Fix — `templates/collection.mezza-luna.json` línea 22:**
```json
"border_radius": 0
```
Full-bleed consistente con Classico e Incontro.

---

### T3-2 — Digital Soul CTA deshabilitado en colecciones
**Ref:** CO8, R4.7 | **Effort:** XS | **Tipo:** Code | **Estado:** Listo para ejecutar

`show_cta: false` en todas las instancias de `snc-scroll-gallery` dentro de los templates de colección.

**Fix — en `templates/collection.classico.json`, `collection.incontro.json`, `collection.mezza-luna.json`:**
```json
"show_cta": true
```
El comprador que terminó la galería necesita un camino a `/pages/digital-soul`.

---

### T3-3 — Template Incontro: el momento editorial solo existe en preview
**Ref:** CO6, R4.5 | **Effort:** S | **Tipo:** Code | **Estado:** Consultar primero

`dl-scroll-scrub` con el texto "You are not wearing a ring. You are wearing something he wrote in 1975." está en `collection.incontro-preview.json` pero no en el canónico `collection.incontro.json`.

**Fix:** Copiar el bloque `dl-scroll-scrub` del preview al template canónico. **Prerequisito:** resolver V5 (Cormorant Garamond) si `dl-scroll-scrub` usa esa fuente.

---

### T3-4 — Filtros visibles en colecciones de 3–12 SKUs
**Ref:** CO2, R4.3 | **Effort:** S | **Tipo:** Code | **Estado:** Listo para ejecutar

**Fix — `sections/snc-plp.liquid`:**
Condicionar el render de la barra de filtros:
```liquid
{% if collection.products_count >= 20 %}
  {% render 'snc-filter-bar' %}
{% endif %}
```
Threshold ajustable según criterio editorial.

---

### T3-5 — "Made to order" ausente a nivel de colección
**Ref:** CO4, R4.1 | **Effort:** S | **Tipo:** Code + Contenido | **Estado:** Listo para ejecutar

**Fix:** Agregar una línea de copy después del subtítulo de colección en los templates named (Classico, Incontro, Mezza Luna). En el schema de `snc-plp.liquid`, agregar un campo de texto `made_to_order_note` opcional. En el template, renderizar si está definido:
```liquid
{% if section.settings.made_to_order_note != blank %}
  <p class="collection-made-to-order">{{ section.settings.made_to_order_note }}</p>
{% endif %}
```
**Copy sugerido:** `"Each piece crafted to order"`

---

### T3-6 — Sin navegación cross-colección al final del scroll
**Ref:** CO7, R4.6 | **Effort:** M | **Tipo:** Code | **Estado:** Consultar primero

Al terminar Classico, no hay invitación a explorar Incontro o Mezza Luna.

**Fix propuesto:** Una sección de footer en cada template de colección con los nombres de las otras dos colecciones, en negro, centrados, con link. Puede reutilizar `snc-custom-content` existente con bloque de links.

---

## TIER 4 — PÁGINAS INFORMACIONALES

---

### T4-1 — Newsletter placeholder en español, confirmación en inglés
**Ref:** I9 | **Effort:** XS | **Tipo:** Code | **Estado:** Listo para ejecutar

**Fix:** Buscar el campo de email del newsletter en las secciones correspondientes:
```bash
grep -rn "Ingresa tu email\|ingresa tu" sections/
```
Cambiar `"Ingresa tu email"` → `"Your email address"` o el placeholder en inglés que corresponda.

---

### T4-2 — Captions "Joya" en página Digital Soul
**Ref:** I4 | **Effort:** XS | **Tipo:** Admin | **Estado:** Necesita contenido

`page.nft.json`: múltiples bloques de imagen con caption genérico "Joya".

**Fix:** En Shopify Admin → Pages → Digital Soul (o equivalente) → Customize, actualizar cada caption con el nombre específico de la pieza que muestra la imagen. Alternativa rápida: dejar captions vacíos.

---

### T4-3 — OG meta tags duplicados + sin imagen en páginas informacionales
**Ref:** I7 | **Effort:** S | **Tipo:** Code | **Estado:** Listo para ejecutar

`snippets/meta-tags.liquid` y `theme.liquid` generan los mismos tags OG. Páginas como The Maison, Digital Soul, Contact comparten sin `og:image`.

**Fix 1:** Revisar cuál de los dos archivos genera los OG tags y eliminar el duplicado.
**Fix 2:** En Shopify Admin → Pages, asignar una imagen featured a cada página informacional clave para que `og:image` no sea null.

---

### T4-4 — page.about.json vacío: todos los campos de copy en blanco
**Ref:** I1, R4.8 | **Effort:** XS | **Tipo:** Admin | **Estado:** Consultar primero

**Opciones:**
- A) En Shopify Admin → Pages, redirigir `/pages/about` a `/pages/the-maison` (recomendado — no deben existir dos URLs para el mismo contenido)
- B) Completar el copy en el editor

---

### T4-5 — "De Leville Circle" mencionado sin página ni programa
**Ref:** I3 | **Effort:** — | **Tipo:** Decisión | **Estado:** Consultar primero

La página Digital Soul menciona el Circle pero no hay destino. Opciones:
- A) Remover la referencia del copy hasta que el programa exista
- B) Crear una página placeholder con lista de espera
- C) Dejarlo como concepto sin link (menor daño)

---

## TIER 5 — FUNCIONALIDAD NUEVA
_Requiere diseño conjunto. No ejecutar sin alinear primero._

---

### T5-1 — Implementar el triángulo △
**Ref:** V7, R5.1 | **Effort:** M | **Tipo:** Code | **Estado:** Consultar primero

El motivo central de la marca no aparece en ningún archivo del tema.

**Candidatos de implementación:**
- SVG inline en el footer (watermark)
- Divisor entre secciones editoriales
- Badge junto a "Digital Soul" en el PDP
- Watermark semitransparente en el hero de The Maison

---

### T5-2 — Zoom / lightbox en el PDP
**Ref:** P4, R3.5 | **Effort:** L | **Tipo:** Code | **Estado:** Consultar primero

Para joyería a este precio, el detalle es parte de la decisión. Click en imagen → lightbox fullscreen con zoom.

---

### T5-3 — Navegación de imágenes en desktop
**Ref:** P5, R3.4 | **Effort:** M | **Tipo:** Code | **Estado:** Consultar primero

6 fotografías sin indicación visual de cuántas hay ni forma de saltar. Strip de thumbnails verticales a la izquierda, o dots numerados.

---

### T5-4 — Soporte de video en el media loop del PDP
**Ref:** P4, R3.6 | **Effort:** M | **Tipo:** Code | **Estado:** Consultar primero

`snc-pdp-hero.liquid` línea 1977 filtra solo `media_type == 'image'`. Añadir un branch para video.

---

### T5-5 — Digital Soul en el side cart post add-to-cart
**Ref:** R5.5 | **Effort:** S | **Tipo:** Code | **Estado:** Consultar primero

Cuando el side cart se abre con el anillo, una línea debajo del producto:
`"This piece will receive its Digital Soul 30 days after shipping."`
Futura PT Book, ~11px, gris #767676.

---

### T5-6 — Carrito standalone (/cart) sin estilos de marca
**Ref:** C3, R4.9 | **Effort:** L | **Tipo:** Code | **Estado:** Consultar primero

`sections/cart.liquid` es el skeleton de Shopify. Opciones:
- A) Estilizarlo completamente al nivel del side cart
- B) Redirigir `/cart` directamente a checkout (algunas marcas de lujo hacen esto — el carrito como estado transitorio)

---

### T5-7 — Guía de tallas integrada en el PDP
**Ref:** P3, R5.3 | **Effort:** L | **Tipo:** Code + Contenido | **Estado:** Consultar primero

Modal o panel deslizable desde el selector de talla con instrucciones visuales de medición. Para joyería hecha a pedido, reduce la fricción más que cualquier otra cosa.

---

### T5-8 — Página Press / Editorial
**Ref:** I6, R5.2 | **Effort:** M | **Tipo:** Code + Contenido | **Estado:** Necesita contenido

Logotipos de publicaciones + quotes. Puede construirse con `snc-custom-content` existente. No requiere nueva sección.

---

---

## TIER 6 — SNC MIXED GRID: AUDIT DE USO
_Sección usada en 6 templates con propósitos distintos sin un criterio claro._

---

### Contexto: dónde está y qué hace en cada template

| Template | Propósito real | Configuración | Problema |
|----------|---------------|---------------|---------|
| `index.json` | Teaser de homepage | Colección: incontro, 1 row, ratio 3/4 fixed 450px | Solo muestra Incontro. Homepage privilegia una colección sobre las demás |
| `product.json` | "Complete the Look" (related products) | **Colección: vacía** | Sin colección asignada — no muestra productos |
| `collection.classico.json` | Grid principal de la colección | Colección: classico, 3 rows, ratio 4/5 | Funciona, pero ver nota abajo |
| `collection.incontro.json` | Grid principal de la colección | Colección: incontro, 4 rows | Funciona |
| `collection.incontro-preview.json` | Grid principal de la colección | Colección: incontro, 4 rows | Duplicado del canónico |
| `collection.mezza-luna.json` | Grid principal de la colección | Colección: mezza-luna, 2 rows, patrón 3-2 | Funciona |

---

### T6-1 — PDP "Complete the Look": colección vacía
**Effort:** XS | **Tipo:** Admin/Code | **Estado:** Listo para ejecutar

`product.json` tiene snc-mixed-grid con `"collection": ""`. Sin colección asignada, la sección no muestra productos (o muestra placeholders).

**Fix:** En Shopify Admin → Themes → Customize → Product template → sección "Complete the Look", asignar una colección relevante. Opciones:
- Asignar la colección que corresponda al producto activo (Classico, Incontro, o Mezza Luna) — requiere una colección genérica "all rings" o lógica Liquid
- Asignar una colección fija de "best sellers" como fallback
- **Fix más correcto (código):** En `sections/snc-mixed-grid.liquid`, si `section.settings.collection` está vacío y el contexto es PDP, usar `product.collections.first` para mostrar productos de la misma colección del producto actual

---

### T6-2 — Homepage: solo muestra Incontro, ignora Classico y Mezza Luna
**Effort:** S | **Tipo:** Decisión + Admin | **Estado:** Consultar primero

La homepage tiene snc-mixed-grid configurada fijamente con `collection: incontro` y 1 fila. Classico y Mezza Luna no tienen representación en el homepage actual.

**Decisión requerida:** ¿La homepage debe mostrar todas las colecciones en paralelo (tres grids, una por colección), o mostrar una colección rotativa como editorial feature?

**Opción A — Tres secciones separadas:** Añadir dos instancias más de snc-mixed-grid (una por colección), cada una con 1 row. Separadas por copy editorial.

**Opción B — Grid mixto con colección "all":** Crear una colección "Featured" en Shopify con las mejores piezas de cada colección y asignarla al grid de homepage.

---

### T6-3 — Named collections usando snc-mixed-grid como PLP: decisión intencional vs. accidental
**Effort:** — | **Tipo:** Decisión | **Estado:** Revisar con el cliente

Las tres colecciones named (Classico, Incontro, Mezza Luna) usan snc-mixed-grid en lugar de snc-plp como su grid de productos.

**Consecuencias:**
- **Sin paginación:** Solo los productos que quepan en `rows` configurado (3 para Classico, 4 para Incontro, 2 para Mezza Luna). Si se añaden más SKUs, no aparecen automáticamente.
- **Sin filtros ni sort:** Deliberado para estas colecciones (pocas SKUs, tono luxury). ✓
- **Visual pattern fijo:** El patrón 3-3-3-3 o 3-2-3-2 es una decisión estética manual, no adaptativa.
- **Hover funcional:** El crossfade de imagen secundaria + fade de título/precio ya funciona en este componente.

**Recomendación:** Para colecciones con ≤12 SKUs made-to-order, snc-mixed-grid es la elección correcta. Documentarlo como intencional. La única acción necesaria: **asegurarse de actualizar el setting `rows` cuando se añadan nuevos productos.**

---

## TIER 7 — ESTADO DE BOTONES (YA EJECUTADO)
_Documentado aquí para registro — no requieren acción._

---

### ✅ T7-1 — Botones PDP Hero: ya en píldora
**Ref:** T0-3 (relacionado) | **Estado:** COMPLETADO (sesión anterior)

`sections/snc-pdp-hero.liquid`:
- Botón "Add to Bag": `border-radius: 999px !important` (línea 1261) ✓
- Botón "Buy Now": `border-radius: 999px` (línea 1173) ✓
- Selector de talla: `border-radius: 999px` (línea 1419) ✓

No requiere cambio.

---

### ✅ T7-2 — Side cart: switch y botones ya en píldora
**Ref:** Fix 3 CLAUDE.md | **Estado:** COMPLETADO (sesión 2026-04-29)

`sections/snc-side-cart.liquid`:
- Switch Cart/Favourites: `border-radius: 999px` (líneas 387, 401) ✓
- Botones empty state (Explore/Best Sellers): `border-radius: 999px` (línea 419) ✓
- Botón Checkout: `border-radius: 999px` (línea 937) ✓
- Wishlist card CTA: `border-radius: 999px` (línea 1017) ✓

No requiere cambio.

---

## RESUMEN DE QUICK WINS
_Estos se pueden ejecutar en una sola sesión sin consultas, < 2 horas en total:_

| # | Acción | Archivo | Effort |
|---|--------|---------|--------|
| 1 | Wishlist heart → #000000 | settings_data.json | XS |
| 2 | main_title_color → #131313 | settings_data.json | XS |
| 3 | Scheme-3 títulos → #ffffff | settings_data.json | XS |
| 4 | foreground_color → #131313 | settings_data.json | XS |
| 5 | KWARA defaults en schema | settings_schema.json + snc-footer.liquid + snc-contact-form.liquid | S |
| 6 | "Agregado al carrito" → inglés | snc-sections.js | XS |
| 7 | "Related Products" placeholder | product.json | XS |
| 8 | border_radius Mezza Luna → 0 | collection.mezza-luna.json | XS |
| 9 | Digital Soul CTA → true | collection templates | XS |
| 10 | Newsletter placeholder → inglés | sección de newsletter | XS |
| 11 | Times New Roman → Futura PT en PLP | snc-plp.liquid | XS |
| 12 | font-weight 600 → 400/500 | múltiples secciones | S |

---

## ORDEN DE EJECUCIÓN RECOMENDADO

1. **Session A (Quick Wins):** T0-4, T0-5, T1-1, T1-2, T1-3, T1-4, T1-5, T2-1, T3-1, T3-2, T4-1 — sin consultas, solo código
2. **Session B (Con consulta):** T0-1, T0-2, T1-7, T1-9, T2-3, T2-4 — alinear decisiones y ejecutar
3. **Session C (Estructural):** T0-3, T3-3, T3-4, T3-5, T5-5 — cambios en componentes
4. **Session D (Nuevo):** T5-1 a T5-8 — diseño conjunto para funcionalidad nueva
