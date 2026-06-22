# Changelog — De Leville Theme

Full history of changes by date and file. Newest first.

---

## 2026-06-22

| Archivo | Cambio |
|---------|--------|
| `CLAUDE.md` | Reescrito para ser conciso. Change Log movido a `CHANGELOG.md`. Secciones de arquitectura movidas a `docs/ARCHITECTURE-DECISIONS.md`. |
| `docs/WORKFLOW.md` | Nuevo. Documenta el workflow completo: CLI dev → git push → GitHub Actions → live. Incluye protocolo de colaboración para evitar conflictos entre desarrollador y dueño de tienda. |
| `docs/ARCHITECTURE-DECISIONS.md` | Nuevo. Documenta decisiones arquitectónicas: separación Shop/Story, snc-incontro-story, PDP sticky bar. |
| `docs/PLANS.md` | Nuevo. Índice de todos los planes de desarrollo. |
| `.github/workflows/deploy.yml` | Nuevo. GitHub Actions workflow que hace `shopify theme push` en cada commit a `main`. |
| Estructura docs/ | Todos los docs de referencia movidos a `docs/`. Rollback docs archivados en `docs/archive/`. |

---

## 2026-06-01

| Archivo | Cambio |
|---------|--------|
| `sections/snc-faq.liquid` | **FAQPage schema**: JSON-LD añadido. Las 9 preguntas son elegibles para Featured Snippets de Google y AI Overviews. |
| `sections/snc-animated-product.liquid` | **Product schema condicional**: schema `@type: Product` ahora solo se emite si `request.page_type == 'product'`. Antes se emitía en homepage. |
| `layout/theme.liquid` | **BreadcrumbList deduplicado**: ya no se emite en páginas de producto (tienen su propio BreadcrumbList en `snc-pdp-hero`). |
| `snippets/meta-tags.liquid` | **Meta description siempre presente**: removida condición `{% if page_description %}`. Usa `og_description` como fallback. |
| `snippets/meta-tags.liquid` | **og:image corregido**: `http:` → `https:`. Facebook y LinkedIn rechazaban la imagen con protocolo inseguro. |
| `layout/theme.liquid` | **Organization schema enriquecido**: `@type: ["Organization", "JewelryStore"]`, `@id`, `priceRange`, `knowsAbout`, `hasOfferCatalog` con 5 colecciones, `contactPoint`, `sameAs` con LinkedIn y Pinterest. |
| `sections/article.liquid` | **BlogPosting schema**: JSON-LD con `headline`, `datePublished`, `dateModified`, `author`, `publisher`. |
| `sections/snc-pdp-hero.liquid` | **hasVariant en Product schema**: cada variante incluye nombre, SKU, precio, disponibilidad y URL. PURO / SEGRETO / REGALE / ESSENZA son entidades diferenciadas. |
| `sections/snc-the-maison-page.liquid` | **Speakable + AboutPage schema**: JSON-LD con `speakable` apuntando a `.dl-page-intro` y `.dl-page-quote`. |
| `sections/snc-llms-txt.liquid` + `templates/page.llms.json` | **llms.txt**: creados para `/pages/llms-txt`. Redirect `/llms.txt → /pages/llms-txt`. De Leville indexable por Perplexity, ChatGPT, y Claude. |
| `locales/es.json` | **Eliminado**: la web es solo inglés por ahora. |
| `sections/snc-header.liquid` | **ARIA en contadores del carrito**: `role="status" aria-live="polite"` en spans del contador. |
| `sections/snc-header.liquid` | **Aria-labels en inglés**: 6 labels traducidos de español a inglés. |
| `sections/snc-header.liquid` | **Emoji**: `🎉` → `✦` en mensaje de envío gratuito. |
| `sections/snc-pdp-hero.liquid` | **Verified icon**: color `#27AE60` → `#1a1a1a`. |
| `config/settings_schema.json` | **Nombre del tema**: `"Skeleton" v0.1.0` → `"De Leville" v1.0.0`. |
| `sections/snc-plp.liquid` | **Copy botón paginación**: `"Load more"` → `"Discover More"`. |
| `assets/snc-sections.js` | **Bug wishlist**: `dispatchEvent` de `snc:wishlist-updated` movido exclusivamente al handler de click. |
| `layout/theme.liquid` | **Focus states**: `outline: 1.5px solid #1a1a1a` en `:focus-visible`. Solo aplica a navegación por teclado. |

---

## 2026-05-13

| Archivo | Cambio |
|---------|--------|
| `sections/snc-header.liquid` | **Search panel full-width**: reemplazado floating box por panel editorial ancho completo que desliza desde el header. |
| `sections/snc-header.liquid` | **Search — imágenes de colecciones**: `window.__dlCollectionImages` generado via Liquid para mapear `handle → featured_image` con fallback a primer producto. |
| `sections/snc-info-hero.liquid` | **Hero flash blanco**: `background-color: #000` en `.snc-info-hero--root` — negro durante carga del video. |
| `sections/snc-animated-collections.liquid` | **Anillos invisibles al cargar**: transición deshabilitada antes del primer `handleScroll()`, restaurada via doble `requestAnimationFrame`. |

---

## 2026-05-12

| Archivo | Cambio |
|---------|--------|
| Shopify Admin API — 24 productos | **Actualización de descripciones**: 24 descripciones (8 modelos × 3 colores) reemplazadas vía Admin API GraphQL. Estructura: párrafo poético en `<em>` + ficha técnica. |

---

## 2026-05-11

| Archivo | Cambio |
|---------|--------|
| `sections/snc-animated-product.liquid` | **Fix mobile single-banner invisible**: early return en `initMobileHorizontalScroll()` si la sección tiene clase `snc-animated-product--simple-banner`. |
| `sections/snc-mixed-grid.liquid` | **Fix segunda imagen en touch**: clase `.snc-mg--touched` via JS en `touchstart`/`touchend`. |
| `sections/snc-pdp-recommended.liquid` | **Fix segunda imagen en touch**: clase `.pdp-rec--touched` via JS en `touchstart`/`touchend`. |
| `sections/snc-pdp-hero.liquid` | **Lightbox LV-style**: click en imagen abre lightbox fullscreen. Botón cerrar, flechas, contador, "Click to zoom" badge. srcset 600/900/1200/1800w + `data-zoom-src` a 2400px. |
| `sections/snc-pdp-hero.liquid` | **Zoom en lightbox**: 2.5× centrado en punto tocado. Pan con drag. Mobile: pinch-to-zoom (1×–4×), swipe para navegar. |

---

## 2026-05-09

| Archivo | Cambio |
|---------|--------|
| `sections/snc-animated-product.liquid` `templates/index.json` | **Banner LEATHERGOODS LV-style**: clase `full-width`, altura `min(90vh)` desktop / `min(80vh)` mobile, `padding_bottom: 0`. |
| `sections/snc-animated-product.liquid` | **Single-banner mode**: clase `snc-animated-product--simple-banner`, sin scroll animado, `height: auto`. |
| `sections/snc-animated-collections.liquid` | **Anillo único centrado mobile**: contenedor `.mobile-only` con imagen seleccionada. Settings: `mobile_ring_animation` y `mobile_ring_size`. |
| `sections/snc-collections-editorial.liquid` | **Gap cero entre tarjetas** + **grid full-bleed** (`100vw` con `translateX(-50%)`). |
| `snc-plp.liquid` `snc-home-showcase.liquid` `snc-pdp-recommended.liquid` | **Texto overlay mobile pequeño**: `clamp()` en font-sizes. |
| `snc-plp.liquid` | **Gap cero entre tarjetas**: `column-gap: 1px` + `row-gap: 1px`. |
| `snc-plp.liquid` `snc-home-showcase.liquid` `snc-pdp-recommended.liquid` | **Imágenes alta resolución**: srcset hasta 2400/2800w. |

---

## 2026-05-08

| Archivo | Cambio |
|---------|--------|
| `sections/snc-incontro-story.liquid` `templates/page.incontro.json` | **Nueva sección**: reemplaza `snc-horizontal-slides` + `snc-scroll-video`. Ver `docs/ARCHITECTURE-DECISIONS.md`. |
| `sections/snc-pdp-hero.liquid` | **PDP mobile sticky bar**: barra "Add to Bag" fija debajo del header. Botones inline ocultos en mobile. |
| `sections/snc-pdp-hero.liquid` | **PDP mobile gap fix**: `top: 0` en info card para `position: relative` en mobile. |
| `sections/snc-plp.liquid` | **CTAs en colección**, **story CTA shimmer**, **cursor badge editorial**, **price filter slider**, **srcset en cards**, **gap uniforme 1px**. |

---

## 2026-05-04

| Archivo | Cambio |
|---------|--------|
| `sections/snc-collections-editorial.liquid` | **Rediseño LV-style**: hero full-viewport, capítulos por colección, grid 4 cols full-bleed, imagen editorial 2×2. |
| `sections/snc-plp.liquid` | **Hero opcional**, grid 4 cols full-bleed, filter bar sticky, story CTA. |
| `templates/collection.*.json` | Reemplazados por `snc-plp`. |
| `templates/page.*.json` | Nuevos templates editoriales por colección. |

---

## 2026-05-02

| Archivo | Cambio |
|---------|--------|
| `sections/snc-pdp-hero.liquid` | Breadcrumb simplificado, selector de variantes 100% Liquid, Ajax variant switching, Mobile LV-style. |
| `snippets/meta-tags.liquid` `layout/theme.liquid` | OG/Twitter meta tags deduplicados. `og:image` a 1200px. |
| `sections/snc-pdp-recommended.liquid` | Rediseño LV-style: `aspect-ratio: 4/5`, overlay que desaparece en hover. |
| `assets/snc-sections.js` | T0-2: `#2A4432` → `#000000` (5 instancias). T0-3: quick-add abre cart drawer. |
| `config/settings_data.json` | `type_primary_font: funnel_sans_n4` → `assistant_n4`. |
| `sections/snc-pdp-hero.liquid` | Badge "Every piece carries a Digital Soul". Digital Soul movido a primer acordeón. |

---

## 2026-05-01

| Archivo | Cambio |
|---------|--------|
| `snc-home-showcase.liquid` | **Rediseño LV-style**: grid 4×4, `aspect-ratio: 4/5`, overlay transparente, fade en hover. |
| `sections/snc-pdp-recommended.liquid` | Nueva sección PDP: 4 cols, auto-detects collection, excluye producto actual. |
| `assets/snc-sections.js` | "Agregar al carrito" → "Added to your bag". |
| `layout/theme.liquid` | Cookie consent font: Cormorant Garamond → `var(--font-main)`. |
| `snc-header.liquid` | `border-bottom` eliminados del header. |

---

## 2026-04-29

| Archivo | Cambio |
|---------|--------|
| `snc-header.liquid` | Fix cart counter: número blanco sobre círculo negro. |
| `snc-mixed-grid.liquid` | Hover en product card: título y precio fade out al mostrar segunda imagen. |
| `snc-side-cart.liquid` | Botones y switch convertidos a píldora (`border-radius: 999px`). |
