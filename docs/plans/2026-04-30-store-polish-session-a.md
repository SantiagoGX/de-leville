# De Leville Store Polish — Session A Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Eliminar todos los errores activos de la tienda De Leville que contaminan la marca ahora mismo — strings en español, color KWARA verde, side cart sin abrir, y settings rotos — sin tocar funcionalidad nueva.

**Architecture:** Cambios quirúrgicos a archivos existentes. Sin nueva infraestructura. Cada tarea es independiente y se puede verificar en `shopify theme dev` inmediatamente. El orden está pensado para que lo más visible se corrija primero.

**Tech Stack:** Liquid, JSON (Shopify settings), Vanilla JS — sin build process, sin tests automatizados. Verificación = visual en el browser.

**Nota de verificación:** Después de cada tarea, el dev server (`shopify theme dev`) hot-reloads automáticamente. La verificación es visual en el browser — abrir el URL afectado y confirmar el cambio.

---

## Mapa de archivos

| Archivo | Qué cambia |
|---------|-----------|
| `config/settings_data.json` | foreground_color, main_title_color, scheme-3, wishlist color |
| `sections/header-group.json` | Valores activos de color KWARA verde |
| `sections/footer-group.json` | newsletter_btn_bg KWARA verde |
| `assets/snc-sections.js` | "Agregado al carrito" → inglés, add-to-cart abre side cart |
| `sections/snc-pdp-hero.liquid` | Badge "Ahorra X%" eliminado |
| `templates/product.json` | "Related Products" placeholder → vacío |
| `templates/collection.mezza-luna.json` | border_radius 12 → 0 |
| `templates/collection.classico.json` | show_cta false → true |
| `templates/collection.incontro.json` | show_cta false → true |
| `sections/snc-plp.liquid` | Cormorant Garamond → Futura PT |
| `sections/snc-footer.liquid` | "KWARA" defaults → "DE LEVILLE", "Ingresa tu email" → inglés, #2A4432 → #000000 |
| `config/settings_schema.json` | "KWARA XL" default → "DE LEVILLE" |
| `sections/snc-contact-form.liquid` | hello@kwara.mx → hello@deleville.com, #2A4432 → #000000 |
| Múltiples sections | #2A4432 default en schemas y CSS → #000000 |

---

## Task 1: Corregir settings_data.json (V2, V3, V4, V8)

Cuatro valores rotos que afectan el sistema visual globalmente.

**Files:**
- Modify: `config/settings_data.json`

- [ ] **Step 1: Verificar estado actual**

```bash
grep -n "foreground_color\|main_title_color\|global_wishlist_active_color\|scheme-3" /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/config/settings_data.json
```

Expected output:
```
72:    "global_wishlist_active_color": "#e00707",
140:    "main_title_color": "#ffffff",
168:      "scheme-3": {
```
(foreground_color NO aparece — ese es el bug: cae al default #333333 del schema)

- [ ] **Step 2: Aplicar los 4 fixes**

En `config/settings_data.json`:

**Fix A — wishlist heart (línea 72):** `#e00707` → `#000000`
**Fix B — main_title_color (línea 140):** `#ffffff` → `#131313`
**Fix C — scheme-3 title_color (línea 173):** `#111111` → `#ffffff`
**Fix D — scheme-3 subtitle_color (línea 174):** `#333333` → `#f0f0f0`
**Fix E — agregar foreground_color** después de `main_title_color`:

```json
"main_title_color": "#131313",
"foreground_color": "#131313",
```

Cambios exactos en el archivo:

```
Línea 72: "global_wishlist_active_color": "#e00707"  →  "#000000"
Línea 140: "main_title_color": "#ffffff"  →  "#131313"
Línea 173: "title_color": "#111111"  →  "#ffffff"
Línea 174: "subtitle_color": "#333333"  →  "#f0f0f0"
Después de línea 140, agregar: "foreground_color": "#131313",
```

- [ ] **Step 3: Verificar el resultado**

```bash
grep -n "foreground_color\|main_title_color\|global_wishlist_active_color\|title_color.*111\|subtitle_color.*333" /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/config/settings_data.json
```

Expected: `foreground_color: #131313`, `main_title_color: #131313`, `global_wishlist_active_color: #000000`. Los near-blacks de scheme-3 NO deben aparecer.

- [ ] **Step 4: Verificación visual**

Abrir cualquier página en `shopify theme dev`. El body text debe verse near-black en lugar de gris. El corazón de wishlist si está visible debe ser negro.

---

## Task 2: Corregir valores activos KWARA en header-group.json y footer-group.json

Estos son valores ACTIVOS (no defaults), renderizando verde en producción ahora mismo.

**Files:**
- Modify: `sections/header-group.json`
- Modify: `sections/footer-group.json`

- [ ] **Step 1: Verificar los valores activos**

```bash
grep -n "2a4432\|2A4432" /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/header-group.json /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/footer-group.json
```

Expected: 3 ocurrencias en header-group.json (líneas ~40, ~190, ~198) y 1 en footer-group.json (línea ~34).

- [ ] **Step 2: Reemplazar en header-group.json**

Tres valores activos a cambiar:
- `"search_text_color": "#2a4432"` → `"#000000"`
- `"cart_title_fallback_color": "#2a4432"` → `"#000000"`
- `"cart_switcher_active_color": "#2a4432"` → `"#000000"`

```bash
sed -i '' 's/"search_text_color": "#2a4432"/"search_text_color": "#000000"/g; s/"cart_title_fallback_color": "#2a4432"/"cart_title_fallback_color": "#000000"/g; s/"cart_switcher_active_color": "#2a4432"/"cart_switcher_active_color": "#000000"/g' /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/header-group.json
```

- [ ] **Step 3: Reemplazar en footer-group.json**

```bash
sed -i '' 's/"newsletter_btn_bg": "#2a4432"/"newsletter_btn_bg": "#000000"/g' /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/footer-group.json
```

- [ ] **Step 4: Verificar que no quedan instancias activas**

```bash
grep -n "2a4432\|2A4432" /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/header-group.json /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/footer-group.json
```

Expected: sin output.

- [ ] **Step 5: Verificación visual**

Abrir el sitio en `shopify theme dev`. El switch del carrito, el botón del newsletter en el footer, y el texto de búsqueda deben ser negros.

---

## Task 3: Purga de KWARA #2A4432 en defaults de secciones Liquid

El color verde de KWARA aparece como `default` en CSS inline y schemas de múltiples secciones. Cuando el admin no ha seteado un valor, renderiza verde. Cambiar todos a #000000.

**Files (los más críticos — los que tienen default activo en CSS, no solo en schema):**
- Modify: `sections/snc-contact-form.liquid`
- Modify: `sections/snc-blog.liquid`
- Modify: `sections/snc-footer.liquid`
- Modify: `sections/snc-icon-grid.liquid`
- Modify: `sections/snc-product-carousel.liquid`
- Modify: `sections/snc-related-grid.liquid`
- Modify: `sections/article.liquid`
- Modify: `sections/snc-header.liquid`
- Modify: `sections/snc-pdp-hero.liquid`
- Modify: `sections/snc-side-cart.liquid`
- Modify: `sections/snc-plp.liquid`
- Modify: `sections/snc-related-carousel.liquid`
- Modify: `sections/snc-search-results.liquid`
- Modify: `sections/snc-collection-carousel.liquid`
- Modify: `sections/snc-faq.liquid`
- Modify: `sections/snc-help-center.liquid`
- Modify: `sections/snc-icon-text-carousel.liquid`
- Modify: `sections/snc-influencer-carousel.liquid`
- Modify: `sections/snc-main-login.liquid`
- Modify: `sections/snc-main-register.liquid`
- Modify: `sections/snc-main-account.liquid`
- Modify: `sections/snc-trust-badges.liquid`
- Modify: `sections/snc-video-carousel.liquid`
- Modify: `sections/snc-custom-content-text.liquid`
- Modify: `config/settings_schema.json`

- [ ] **Step 1: Contar ocurrencias antes**

```bash
grep -rn "2A4432\|2a4432" /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/ /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/config/ | grep -v "header-group.json\|footer-group.json" | wc -l
```

Anotar el número para verificar después.

- [ ] **Step 2: Reemplazar en bulk — todas las secciones y config**

```bash
# macOS sed requiere -i '' para edición in-place
find /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/ /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/config/ -type f \( -name "*.liquid" -o -name "*.json" \) ! -name "header-group.json" ! -name "footer-group.json" -exec sed -i '' 's/#2A4432/#000000/g; s/#2a4432/#000000/g' {} \;
```

- [ ] **Step 3: Verificar que no quedan instancias**

```bash
grep -rn "2A4432\|2a4432" /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/ /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/config/ | grep -v "header-group.json\|footer-group.json"
```

Expected: sin output.

- [ ] **Step 4: Verificar que los archivos modificados tienen JSON válido**

```bash
python3 -m json.tool /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/config/settings_schema.json > /dev/null && echo "settings_schema.json OK"
```

---

## Task 4: Purga de strings KWARA (nombre de marca)

Defaults de nombre de marca en schemas que restaurarían "KWARA" si se hace reset desde el editor.

**Files:**
- Modify: `config/settings_schema.json`
- Modify: `sections/snc-footer.liquid`
- Modify: `sections/snc-contact-form.liquid`

- [ ] **Step 1: Verificar estado actual**

```bash
grep -n "KWARA\|kwara.mx" /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/config/settings_schema.json /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/snc-footer.liquid /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/snc-contact-form.liquid
```

Expected: ~4 ocurrencias totales.

- [ ] **Step 2: Corregir settings_schema.json (línea ~98)**

```bash
sed -i '' 's/"default": "KWARA XL"/"default": "DE LEVILLE"/g' /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/config/settings_schema.json
```

- [ ] **Step 3: Corregir snc-footer.liquid (líneas ~1082 y ~1190)**

```bash
sed -i '' 's/"default": "KWARA"/"default": "DE LEVILLE"/g' /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/snc-footer.liquid
```

- [ ] **Step 4: Corregir snc-contact-form.liquid (email de contacto, línea ~549)**

```bash
sed -i '' "s/\"title\": \"hello@kwara.mx\"/\"title\": \"hello@deleville.com\"/g" /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/snc-contact-form.liquid
```

- [ ] **Step 5: Verificar**

```bash
grep -n "KWARA\|kwara.mx" /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/config/settings_schema.json /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/snc-footer.liquid /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/snc-contact-form.liquid
```

Expected: sin output.

---

## Task 5: Fix JS — "Agregado al carrito" → inglés + side cart se abre

Dos fixes en `snc-sections.js`. El add-to-cart principal (botón "Add to Bag") llama `updateSideCart()` pero NO llama `openCartDrawer()` — la secuencia add → review → checkout está rota.

**Files:**
- Modify: `assets/snc-sections.js`

- [ ] **Step 1: Verificar las líneas exactas con el string en español**

```bash
grep -n "Agregado al carrito" /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/assets/snc-sections.js
```

Expected: 2 ocurrencias (~líneas 3311 y 3362).

- [ ] **Step 2: Reemplazar "Agregado al carrito"**

```bash
sed -i '' "s/Agregado al carrito/Added to your bag/g" /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/assets/snc-sections.js
```

- [ ] **Step 3: Verificar el handler del Add to Bag principal (~línea 3189)**

```bash
sed -n '3185,3210p' /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/assets/snc-sections.js
```

Expected: el `xhr.onload` llama `window.updateSideCart()` y `updateCartCount()` pero NO llama `window.openCartDrawer()`. Ese es el bug — la función existe (`window.openCartDrawer = openCart` en snc-side-cart.liquid línea 1456) pero no se invoca.

- [ ] **Step 4: Agregar `openCartDrawer()` en el handler principal**

En `assets/snc-sections.js`, en el bloque `xhr.onload` del Add to Bag principal (alrededor de línea 3190–3197), el código actual es:

```javascript
        xhr.onload = function() {
          if (xhr.status >= 200 && xhr.status < 300) {
            if (typeof window.updateSideCart === 'function') {
              window.updateSideCart();
            }
            if (typeof updateCartCount === 'function') {
              updateCartCount();
            }
          }
          addToCartButton.disabled = false;
          addToCartButton.style.opacity = '';
        };
```

Reemplazar con:

```javascript
        xhr.onload = function() {
          if (xhr.status >= 200 && xhr.status < 300) {
            if (typeof window.updateSideCart === 'function') {
              window.updateSideCart();
            }
            if (typeof updateCartCount === 'function') {
              updateCartCount();
            }
            if (typeof window.openCartDrawer === 'function') {
              window.openCartDrawer();
            }
          }
          addToCartButton.disabled = false;
          addToCartButton.style.opacity = '';
        };
```

- [ ] **Step 5: Verificar el resultado**

```bash
grep -n "Agregado al carrito\|Added to your bag\|openCartDrawer" /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/assets/snc-sections.js
```

Expected: "Agregado al carrito" = 0 ocurrencias. "Added to your bag" = 2. "openCartDrawer" = al menos 3 (la definición en side cart + los 2 usos en quick-add + el nuevo en add-to-bag).

- [ ] **Step 6: Verificación funcional**

En `shopify theme dev`, ir a cualquier PDP y hacer clic en "Add to Bag". El side cart debe abrirse inmediatamente mostrando el producto agregado.

---

## Task 6: Eliminar badge "Ahorra X%" del PDP (C5/P9)

El badge es en español, es de tono e-commerce masivo, y De Leville no hace descuentos. Eliminar el elemento del DOM.

**Files:**
- Modify: `sections/snc-pdp-hero.liquid`

- [ ] **Step 1: Verificar la línea exacta**

```bash
grep -n "Ahorra\|discount-badge" /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/snc-pdp-hero.liquid | head -10
```

Expected: línea ~2077 con el bloque `{%- if compare_price and discount_percent > 0 -%}` y línea ~2078 con el badge.

- [ ] **Step 2: Eliminar el bloque del badge**

El bloque a eliminar en `sections/snc-pdp-hero.liquid` (líneas ~2077–2079):

```liquid
            {%- if compare_price and discount_percent > 0 -%}
              <span class="pdp-hero--discount-badge">Ahorra {{ discount_percent }}%</span>
            {%- endif -%}
```

Reemplazar con: _(nada — eliminar las 3 líneas)_

- [ ] **Step 3: Verificar que el badge no existe**

```bash
grep -n "Ahorra\|discount-badge.*span" /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/snc-pdp-hero.liquid
```

Expected: sin ocurrencias del badge en el HTML (puede quedar el CSS class `.pdp-hero--discount-badge` en el `<style>` — eso es OK).

---

## Task 7: Template quick fixes — Related Products, Mezza Luna, show_cta

Tres fixes XS en templates JSON.

**Files:**
- Modify: `templates/product.json`
- Modify: `templates/collection.mezza-luna.json`
- Modify: `templates/collection.classico.json`
- Modify: `templates/collection.incontro.json`

- [ ] **Step 1: Corregir "Related Products" en product.json**

```bash
grep -n "Related Products\|sub_text" /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/templates/product.json
```

Expected: línea ~158 con `"text": "<p>Related Products</p>"`.

Reemplazar con vacío:

```bash
sed -i '' 's|"text": "<p>Related Products</p>"|"text": ""|g' /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/templates/product.json
```

Verificar:
```bash
grep -n "Related Products" /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/templates/product.json
```
Expected: sin output.

- [ ] **Step 2: Corregir border_radius en collection.mezza-luna.json**

```bash
grep -n "border_radius" /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/templates/collection.mezza-luna.json | head -5
```

Expected: línea ~21 con `"border_radius": 12`.

```bash
sed -i '' '0,/"border_radius": 12/{s/"border_radius": 12/"border_radius": 0/}' /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/templates/collection.mezza-luna.json
```

Verificar:
```bash
grep -n "border_radius" /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/templates/collection.mezza-luna.json | head -3
```
Expected: primera ocurrencia debe ser `0`, no `12`.

- [ ] **Step 3: Habilitar Digital Soul CTA en las 3 colecciones**

```bash
# classico
sed -i '' 's/"show_cta": false/"show_cta": true/g' /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/templates/collection.classico.json

# incontro
sed -i '' 's/"show_cta": false/"show_cta": true/g' /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/templates/collection.incontro.json

# mezza-luna
sed -i '' 's/"show_cta": false/"show_cta": true/g' /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/templates/collection.mezza-luna.json
```

Verificar:
```bash
grep "show_cta" /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/templates/collection.classico.json /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/templates/collection.incontro.json /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/templates/collection.mezza-luna.json
```
Expected: todas las líneas muestran `"show_cta": true`.

- [ ] **Step 4: Verificación visual**

En `shopify theme dev`:
- Ir a cualquier PDP → sección "Complete the Look" debajo no debe decir "Related Products"
- Ir a `/collections/mezza-luna` → el hero debe ser full-bleed sin esquinas redondeadas
- Ir a cualquier colección → la galería de Digital Soul debe tener un CTA visible

---

## Task 8: Fix PLP — Cormorant Garamond → Futura PT (C7)

El título de colección en `snc-plp.liquid` referencia Cormorant Garamond (no cargada) → Times New Roman en producción.

**Files:**
- Modify: `sections/snc-plp.liquid`

- [ ] **Step 1: Verificar las líneas exactas**

```bash
grep -n "Cormorant\|Garamond\|Times New Roman" /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/snc-plp.liquid
```

Expected: líneas ~1665–1667 con `font-family: 'Cormorant Garamond', 'Times New Roman', serif !important;`

- [ ] **Step 2: Reemplazar**

```bash
sed -i '' "s/font-family: 'Cormorant Garamond', 'Times New Roman', serif !important;/font-family: 'Futura PT', sans-serif !important;/g" /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/snc-plp.liquid
```

- [ ] **Step 3: Verificar**

```bash
grep -n "Cormorant\|Garamond\|Times New Roman" /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/snc-plp.liquid
```

Expected: sin output. Solo debe quedar el comentario de referencia si lo hay.

```bash
grep -n "Futura PT" /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/snc-plp.liquid | tail -3
```

Expected: la línea recién modificada con `'Futura PT', sans-serif`.

- [ ] **Step 4: Verificación visual**

En `shopify theme dev`, ir a cualquier colección que use `snc-plp` (bags, chains, o la genérica). El título debe renderizar en Futura PT, no en Times New Roman.

---

## Task 9: Fix newsletter placeholder — "Ingresa tu email" → inglés (I9)

El placeholder del campo de email está en español y el schema tiene el default en español.

**Files:**
- Modify: `sections/snc-footer.liquid`

- [ ] **Step 1: Verificar las líneas**

```bash
grep -n "Ingresa tu email" /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/snc-footer.liquid
```

Expected: 2 ocurrencias — línea ~665 (HTML placeholder) y línea ~1338 (schema default).

- [ ] **Step 2: Reemplazar el default del schema y el fallback inline**

```bash
sed -i '' "s/Ingresa tu email/Your email address/g" /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/snc-footer.liquid
```

- [ ] **Step 3: Verificar**

```bash
grep -n "Ingresa tu email\|Your email address" /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/snc-footer.liquid
```

Expected: 0 ocurrencias de "Ingresa", 2 ocurrencias de "Your email address".

---

## Task 10: Audit font-weight 600 → 400/500 (V6)

El brand spec dice weight 400. Font-weight 600 activa synthetic bold del browser porque esa variante no está cargada.

**Files:**
- Modify: `sections/snc-side-cart.liquid`
- Modify: `sections/snc-pdp-hero.liquid`
- Modify: `sections/snc-header.liquid`
- Modify: `sections/snc-search-results.liquid`

- [ ] **Step 1: Auditar ocurrencias**

```bash
grep -n "font-weight: 600\|font-weight:600" /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/snc-side-cart.liquid /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/snc-pdp-hero.liquid /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/snc-header.liquid /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/snc-search-results.liquid
```

Anotar el número de líneas y contexto (nombres de producto, totales, títulos de header, resultados).

- [ ] **Step 2: Reemplazar 600 por 500 en estas secciones**

La fuente self-hosted tiene Book (400) y Medium (500). Weight 500 es el máximo cargado — usar 500 para elementos que necesitan énfasis.

```bash
for file in /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/snc-side-cart.liquid /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/snc-pdp-hero.liquid /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/snc-header.liquid /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/snc-search-results.liquid; do
  sed -i '' 's/font-weight: 600/font-weight: 500/g; s/font-weight:600/font-weight:500/g' "$file"
done
```

- [ ] **Step 3: Verificar**

```bash
grep -n "font-weight: 600\|font-weight:600" /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/snc-side-cart.liquid /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/snc-pdp-hero.liquid /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/snc-header.liquid /Users/santiagosalinas/Documents/Shopify-Projects/de-leville/sections/snc-search-results.liquid
```

Expected: sin output.

- [ ] **Step 4: Verificación visual**

En `shopify theme dev`:
- Abrir el side cart → nombres de producto y total no deben tener synthetic bold
- Ir a un PDP → todos los textos deben verse en peso regular/medium
- Verificar el header y resultados de búsqueda

---

## Resumen de verificación final

Después de completar todas las tareas, verificar en `shopify theme dev`:

| Check | URL | Qué buscar |
|-------|-----|-----------|
| Body text near-black | Cualquier página | El texto del body debe verse casi negro, no gris |
| Wishlist negro | Cualquier página con wishlist | Corazón negro, no rojo |
| Switch del carrito negro | Abrir side cart | El switch Cart/Favourites debe ser negro |
| Add to Bag abre side cart | Cualquier PDP | Click en Add to Bag → side cart se abre |
| Toast en inglés | Cualquier PDP | "Added to your bag" (via quick-add buttons) |
| Sin badge "Ahorra" | PDP con compare-at-price | No debe aparecer ningún badge de descuento |
| Sin "Related Products" | Cualquier PDP | La sección "Complete the Look" sin ese texto |
| Mezza Luna full-bleed | `/collections/mezza-luna` | Hero sin esquinas redondeadas |
| Digital Soul CTA | Cualquier colección | CTA visible al final de la galería |
| Futura PT en PLP | `/collections/bags` o genérico | Título en Futura PT, no Times New Roman |
| Newsletter en inglés | Footer de cualquier página | Placeholder "Your email address" |
