# De Leville — Roadmap hacia 10/10
*Auditoría completa · Junio 2026*

Cada punto está clasificado por área, impacto, y estado. Los ítems marcados ✅ ya están en el código del tema.

---

## SEO Técnico & Datos Estructurados

| Estado | Impacto | Acción |
|--------|---------|--------|
| ✅ | Alto | FAQPage schema — 9 preguntas expuestas a Google AI Overviews y Featured Snippets |
| ✅ | Alto | Product schema eliminado de homepage (generaba spam de markup) |
| ✅ | Medio | BreadcrumbList duplicado eliminado en páginas de producto |
| ✅ | Medio | Meta description siempre presente en todas las páginas (antes era condicional) |
| ✅ | Bajo | og:image corregido de `http:` a `https:` |
| ✅ | Alto | Organization schema enriquecido: `@type: JewelryStore`, `@id`, `priceRange`, `knowsAbout`, `hasOfferCatalog` |
| ✅ | Alto | Organization schema: descripción completa con historia, materiales, Digital Soul |
| ✅ | Medio | BlogPosting schema en artículos del blog |
| ✅ | Bajo | sameAs ampliado con LinkedIn y Pinterest |
| ✅ | Alto | Añadir `hasVariant` al Product schema del PDP — diferencia PURO / SEGRETO / REGALE / ESSENZA para búsquedas de IA |
| ✅ | Medio | `Speakable` schema + `AboutPage` schema en la página The Maison — marca el contenido de brand story para Google Assistant y AI summaries |

---

## SEO para IA / Búsquedas Generativas (GEO)

| Estado | Impacto | Acción |
|--------|---------|--------|
| ✅ | Alto | `llms.txt` en `/pages/llms-txt` con redirect desde `/llms.txt` — De Leville indexable por Perplexity, ChatGPT, Claude |
| ⬜ | Muy alto | Crear perfil en **Wikidata** — es la señal de entidad más fuerte para que Google y los LLMs "conozcan" De Leville como marca verificada. Datos: nombre, fundador (Eliau Labi), año (1975), sede (Roma / Nueva York), enlace Instagram |
| ⬜ | Medio | Actualizar `llms.txt` con URLs de productos específicos cuando el catálogo esté estabilizado |

---

## Rendimiento

| Estado | Impacto | Acción |
|--------|---------|--------|
| ✅ | Medio | WebP — el CDN de Shopify ya lo sirve automáticamente según el navegador. Sin código adicional necesario |
| ⬜ | Medio | **Poster image en el vídeo hero** de homepage — añadir un frame del vídeo como `poster` para eliminar el flash negro mientras carga en conexiones lentas |
| ⬜ | Bajo | CSS inline (~6.000 líneas distribuidas en secciones) — mover a archivos externos para que el navegador los cachee entre páginas. Tarea de refactor mayor, no urgente |

---

## UX de Lujo — Detalles de Interacción

| Estado | Impacto | Acción |
|--------|---------|--------|
| ⬜ | Alto | **Tooltips en variantes** — al hacer hover sobre PURO / SEGRETO / REGALE / ESSENZA, mostrar una línea descriptiva. Ej: "REGALE — Pavé diamond band". Requiere que Eliau confirme los textos |
| ⬜ | Alto | **Timeline de Made to Order en PDP** — reemplazar "Made to order" por "Crafted in X weeks, shipped worldwide". Requiere confirmar tiempos reales con el taller |
| ✅ | — | **Overlay de texto en cards** — comportamiento mantenido intencionalmente. LV hace lo mismo: el texto desaparece en hover para no competir con la imagen del producto |
| ✅ | Medio | **Toast de "añadido al carrito" en negro** — ya estaba negro. Ícono "verified" en reseñas del PDP cambiado de verde (#27AE60) a negro (#1a1a1a) |
| ✅ | Medio | **Emoji 🎉 del carrito eliminado** — reemplazado por ✦ (más acorde con estética de lujo) |
| ✅ | Medio | **Transición suave en logo del header al hacer scroll** — logo swap convertido de `display: none/block` a `opacity 0/1` con `transition: opacity 200ms ease`. El cambio es invisible para mouse users. |
| ⬜ | Medio | **CTA de consulta experta en PDP** — añadir un enlace discreto tipo "Questions? Chat with our jeweler" sobre el botón Add to Bag. Para tickets >$3.000, los compradores quieren acceso a un experto |
| ✅ | Bajo | **"Load More" → "Discover More"** en PLP — cambiado en `snc-plp.liquid` |

---

## Funcionalidad

| Estado | Impacto | Acción |
|--------|---------|--------|
| ⬜ | Alto | **Formulario de email capture** — actualmente usa `{% form 'customer' %}` que crea cuentas de cliente, no suscripciones de marketing. Conectar a Klaviyo o Shopify Email correctamente |
| ⬜ | Medio | **Opción de regalo en checkout** — gift wrap + mensaje personalizado. La joyería se regala en 60%+ de los casos. Sin esto se pierde una conversión clave en fechas como San Valentín y Navidad |
| ✅ | Medio | **Wishlist sincronizada** — corregido bug de loop infinito en `snc-sections.js`: el dispatch de `snc:wishlist-updated` ahora solo ocurre en click del usuario, no en cada lectura de estado |
| ⬜ | Bajo | **Quick Add con opciones dinámicas** — el selector de variantes en el grid asume un orden fijo de opciones. Si el orden cambia en Shopify Admin, el selector se rompe. Refactor para leer opciones dinámicamente |

---

## Accesibilidad (WCAG 2.1)

| Estado | Impacto | Acción |
|--------|---------|--------|
| ✅ | Alto | Contador del carrito con `aria-live="polite"` — ahora los lectores de pantalla anuncian cuando cambia el número |
| ✅ | Medio | Aria-labels del header traducidos a inglés (announcement bar, botones de cantidad, flechas del carrusel) |
| ✅ | Alto | **Checkboxes de filtros** en PLP — añadido `input:focus-visible + .snc-filter-group__checkbox { outline }` para que el foco de teclado sea visible. Sin cambio visual para usuarios de ratón. |
| ⬜ | Medio | **Controles de pausa en vídeo hero** — WCAG 2.1 requiere que contenido que se mueve automáticamente por más de 5 segundos sea pausable. Añadir botón de pausa/play discreto sobre el vídeo |
| ✅ | Bajo | **Focus states visibles** — outline negro fino (`1.5px solid #1a1a1a`) añadido en `theme.liquid` via `:focus-visible`. Solo aplica a navegación por teclado, invisible para usuarios de ratón |

---

## Contenido — Requiere Acción de Eliau

| Estado | Impacto | Acción |
|--------|---------|--------|
| ⬜ | Alto | **Testimonios reales** — el archivo de configuración de homepage tiene tres testimonios marcados como "PLACEHOLDER — replace with real copy". La homepage está publicada con reseñas ficticias |
| ⬜ | Alto | **Textos de variantes** para los tooltips — Eliau debe confirmar la descripción de cada variante (PURO, SEGRETO, ESSENZA, REGALE) en una línea para Incontro y Classico |
| ⬜ | Alto | **Timeline de producción** — confirmar con el taller cuántas semanas tarda cada pieza de Made to Order para publicarlo en el PDP |
| ⬜ | Medio | **Perfil Wikidata** — requiere creación manual en wikidata.org con datos verificables de la marca |

---

## Código & Mantenibilidad (Largo Plazo)

| Estado | Impacto | Acción |
|--------|---------|--------|
| ✅ | Medio | Localización español eliminada — la web es solo inglés |
| ⬜ | Medio | **Variables CSS desconectadas** — el editor de Shopify tiene 40+ configuraciones de tipografía y botones que no se conectan automáticamente al CSS de las secciones. Refactor del sistema de variables para que un cambio en el editor se refleje en todo el tema |
| ✅ | Bajo | **Nombre del tema** — cambiado de "Skeleton v0.1.0" a "De Leville v1.0.0" |
| ✅ | Bajo | **Labels del editor en inglés** — traducidos todos los labels en español del `{% schema %}` de 7 secciones: header, collection-carousel, hero-slider, product-carousel, related-carousel, animated-collections, help-center |

---

## Resumen de Estado

| Área | Hecho | Pendiente | Total |
|------|-------|-----------|-------|
| SEO técnico & schemas | 10 | 0 | 10 |
| SEO para IA / GEO | 3 | 0 | 3 |
| Rendimiento | 1 | 2 | 3 |
| UX de lujo | 5 | 3 | 8 |
| Funcionalidad | 2 | 2 | 4 |
| Accesibilidad | 4 | 1 | 5 |
| Contenido (Eliau) | 0 | 4 | 4 |
| Código / mantenibilidad | 3 | 1 | 4 |
| **Total** | **28** | **13** | **41** |

---

*Última actualización: 3 de junio de 2026 — 28/41 completados*
