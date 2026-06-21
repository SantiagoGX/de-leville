# De Leville — Gaps Pendientes

Análisis profundo del tema completo post-auditoría. Estos ítems van más allá de los accionables documentados en `ACCIONABLES.md` y `AUDITORIA-HOMEPAGE.md`. Identificados en sesión 2026-04-30/05-01.

---

## G1 — El flujo de compra se rompe en el carrito standalone ✅ RESUELTO

**Estado:** Resuelto en 2026-05-01.  
**Archivo:** `sections/cart.liquid`  
**Problema:** La página `/cart` era HTML básico de Shopify (tabla sin estilos, cero branding). Si un cliente llega desde email, link externo, o el botón "atrás" del browser, la ilusión de maison se destruía completamente.  
**Solución:** Sección `cart.liquid` rediseñada completamente. Layout dos columnas (items + resumen), tipografía Futura PT, botones píldora, qty selectors idénticos al side cart, AJAX para updates sin recarga, estado vacío branded, aviso de made-to-order.

---

## G2 — Digital Soul existe solo en el PDP, ausente del flujo de compra

**Estado:** Pendiente.  
**Archivos afectados:** `sections/snc-side-cart.liquid`, posiblemente `layout/theme.liquid`  
**Problema:** El diferenciador principal de la marca (blockchain authentication) aparece en el quinto acordeón del PDP. Después: silencio total. El carrito no lo menciona, el checkout no lo menciona, el post-compra no lo menciona. El comprador paga $4,000 por un anillo con "Digital Soul" y no vuelve a escuchar de eso.  
**Acción sugerida:** Añadir bloque breve de Digital Soul en el footer del side cart (entre el botón de checkout y la garantía). Texto sugerido: "Every De Leville piece comes with a Digital Soul — a blockchain certificate of authenticity sent to your email within 30 days of purchase."

---

## G3 — Made-to-order nunca se comunica al comprador

**Estado:** Parcialmente resuelto en G1 (aviso añadido al cart page).  
**Archivos afectados:** `sections/snc-pdp-hero.liquid`, `sections/snc-side-cart.liquid`  
**Problema:** El campo `made_to_order_text` está construido en el schema del PDP y tiene su lógica de display lista, pero nunca se llenó en ningún producto. El comprador agrega un anillo sin saber que espera 3–4 semanas.  
**Acción sugerida:**  
1. Llenar `made_to_order_text` en cada producto del Shopify admin con texto como "This piece is crafted to order. Please allow 3–4 weeks for production."  
2. Añadir el mismo aviso al footer del side cart (después de checkout button).

---

## G4 — Mobile homepage no existe como experiencia

**Estado:** Pendiente.  
**Sub-items:**

- **M1 — snc-animated-collections (mobile):** En mobile, las colecciones deben listarse visiblemente con el nombre de la colección activa resaltado al scroll. Mostrar dos imágenes simultáneas para la colección activa. Animación constante (anillos rotando en eje) independiente del scroll.

- **M2 — snc-mixed-grid (mobile):** Las cards se ven cortadas en mobile. Revisar gap y aspect-ratio en breakpoints. `snc-home-showcase` ya existe como sección separada con modos de selección manual de productos/colecciones.

- **M3 — snc-info-3d (mobile):** Ocultar la primera imagen del anillo en mobile, mostrar solo texto + segunda imagen del anillo.

---

## G5 — "Join the Waitlist" lleva a ningún lado

**Estado:** Pendiente.  
**Archivo:** `sections/snc-animated-product.liquid` (bloque con botón "Join the waitlist")  
**Problema:** El CTA existe en la sección de producto animado pero no hay página `/pages/waitlist` ni formulario. El botón lleva a un 404 o a una página vacía.  
**Acción sugerida:** Crear una página de waitlist simple con formulario de email, o cambiar el link del botón a `/pages/contact` mientras se crea la página dedicada.

---

## G6 — Sizing guide existe pero no está linkeada desde el PDP

**Estado:** Pendiente.  
**Archivos:** `templates/page.sizing-guide.json` (existe), `sections/snc-pdp-hero.liquid`  
**Problema:** La página de guía de tallas existe (`/pages/sizing-guide`) pero no hay ningún link desde el PDP. Los clientes de joyería necesitan esta información antes de comprar.  
**Acción sugerida:** Añadir link "Size Guide" cerca del selector de tallas en el PDP. Puede ser un link de texto simple o un modal trigger.

---

## G7 — Cookie consent banner usa Cormorant Garamond

**Estado:** Pendiente.  
**Archivo:** `layout/theme.liquid` (banner de cookies inline)  
**Problema:** El banner de consentimiento de cookies usa `font-family: 'Cormorant Garamond'` que no está cargado en el tema. El browser fallback es Times New Roman, lo que rompe la consistencia visual justo en el primer elemento que ve un nuevo visitante.  
**Acción sugerida:** Cambiar el font-family del banner a `var(--font-main)` o `'Futura PT', sans-serif`.

---

## G8 — Colecciones genéricas sin template editorial

**Estado:** Pendiente.  
**Archivos:** `templates/collection.json` (template genérico)  
**Problema:** Solo Mezza Luna, Classico e Incontro tienen templates dedicados con diseño editorial. Bags, chains, y cualquier colección nueva usan `collection.json` que muestra el esqueleto básico de Shopify (grid plano sin hero, sin story, sin contexto de marca).  
**Acción sugerida:** Mejorar `templates/collection.json` para que use al menos un hero mínimo y el grid de `snc-plp.liquid` con los estilos correctos. O crear un template `collection.editorial.json` aplicable a colecciones que tengan contenido de marca.

---

*Documento creado: 2026-05-01*  
*Contexto: Sesión de polish post-accionables. Análisis por exploración de 15 archivos clave del tema.*
