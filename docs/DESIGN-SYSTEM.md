# Sistema de Diseño — De Leville
_Actualizado: 2026-04-29 — Referencia canónica para construir nuevas secciones_

> Este documento evita improvisar. Antes de crear cualquier sección nueva, consultar aquí primero. Si algo no está cubierto, añadirlo a este documento después de tomar la decisión.

---

## 1. FILOSOFÍA DE MARCA

**De Leville no es un e-commerce de conversión rápida. Es una maison.**

El comprador objetivo no responde a urgencia ni descuento. Responde a certeza, proveniencia y silencio editorial.

**Principios en código:**
- Nada de countdown timers
- Nada de badges "¡Oferta!" o "Últimas unidades"
- Nada de colores de acento que griten
- El espacio en blanco es deliberado — no llenarlo
- "Made to order" es una feature, no un disclaimer
- Digital Soul es el diferenciador principal — debe ser visible sin interacción

---

## 2. TIPOGRAFÍA

### Fuente principal: Futura PT

```
Font family: 'Futura PT', sans-serif
CSS property: 'Futura PT', sans-serif   ← usar directamente, NO var(--font-primary--family)
```

**Archivos self-hosted disponibles:**
| Archivo | Peso | Uso |
|---------|------|-----|
| `FuturaCyrillicBook.woff` | 400 | Body text, copy editorial, precios |
| `FuturaCyrillicMedium.woff` | 500 | Subtítulos, labels de UI, navegación |

**Regla crítica:** El brand spec dice **weight 400 exclusivamente** para texto editorial. Weight 500 para UI labels y navegación. Weight 600+ activa synthetic bold del browser — no usar.

> ⚠️ `var(--font-primary--family)` resuelve a Funnel Sans (bug activo). Siempre usar `'Futura PT', sans-serif` directamente hasta que se resuelva T1-9.

### Escala tipográfica

| Rol | Tamaño | Peso | Uso |
|-----|--------|------|-----|
| Main title | 32px | 400 | Títulos de hero, secciones principales |
| Secondary title | 16px | 500 | Subtítulos de sección, headers de acordeón |
| Small title | — | 500 | Labels, navegación interna |
| Body / Paragraph | 16px | 400 | Texto editorial, descripciones |
| Small paragraph | 10px | 400 | Captions, microcopy, notas legales |
| Button | 16px | 100 | CTAs principales |

### Letter-spacing y line-height

```css
/* Body text global */
letter-spacing: -0.04em;
line-height: 1.5;

/* Headers editoriales (all-caps) */
letter-spacing: 0.10em–0.15em;
text-transform: uppercase;

/* Microcopy, "made to order", Digital Soul badge */
letter-spacing: 0.15em;
font-size: 12px;
font-weight: 400;
```

### Lo que NO usar
- `font-family: 'Cormorant Garamond'` — no está cargada, renderiza en Times New Roman
- `font-weight: 600` o superior — synthetic bold, no hay fuente cargada para ese peso
- `var(--font-primary--family)` — actualmente resuelve a Funnel Sans (bug)

---

## 3. SISTEMA DE COLOR

### Paleta base

| Token | Valor | Uso |
|-------|-------|-----|
| Background | `#ffffff` | Fondo de página |
| Foreground (body text) | `#131313` | Texto body global (var: `--color-foreground`) |
| Near-black | `#111111` | Títulos principales |
| Dark gray | `#2f2f2f` | Subtítulos, texto secundario |
| Mid gray | `#767676` | Microcopy, captions, metadata |
| Light gray | `#f2f2f2` | Fondos de UI, filter bar |
| Black | `#000000` | CTAs primarios, iconos |
| White | `#ffffff` | Texto sobre negro, elementos invertidos |

**Acento:** Ninguno. El sistema es black/white/gray exclusivamente. Cualquier color de acento que no sea `#000000` está fuera de sistema.

> ⚠️ `#2A4432` (verde KWARA) — purgar de todos los archivos
> ⚠️ `#e00707` (rojo wishlist) — cambiar a `#000000`

### Esquemas de color (Color Schemes)

#### Scheme-2 — Default Light (uso estándar)
```json
{
  "background": "#ffffff",
  "title_color": "#111111",
  "subtitle_color": "#2f2f2f",
  "text": "#2f2f2f",
  "button_background": "rgba(0,0,0,0)",
  "button_label": "#000000",
  "secondary_button_label": "#000000"
}
```
Usar para: páginas de producto, colecciones, páginas informacionales, cualquier sección estándar.

#### Scheme-1 — Overlay Dark (texto sobre foto oscura)
```json
{
  "background": "#ffffff",
  "title_color": "#ffffff",
  "subtitle_color": "#ffffff",
  "text": "#e3e3e3",
  "button_background": "rgba(0,0,0,0)",
  "button_label": "#ffffff",
  "secondary_button_label": "#f5f5f5"
}
```
Usar para: secciones con fotografía de fondo oscura, heros con overlay.

#### Scheme-3 — Dark Editorial (fondo negro)
```json
{
  "background": "#000000",
  "title_color": "#ffffff",
  "subtitle_color": "#f0f0f0",
  "text": "#ffffff",
  "button_background": "#000000",
  "button_label": "#ffffff",
  "secondary_button_label": "#000000"
}
```
Usar para: secciones editoriales oscuras, separadores de ritmo visual. **Nunca mezclar con texto near-black sobre negro.**

---

## 4. BOTONES Y CTAs

### Convención universal: pill shape
```css
border-radius: 999px;  /* TODOS los botones, sin excepción */
```

### Variantes

#### CTA Primario (Add to Bag, Checkout)
```css
background: #000000;
color: #ffffff;
border-radius: 999px;
font-family: 'Futura PT', sans-serif;
font-size: 16px;
font-weight: 100;
letter-spacing: 0.10em;
text-transform: uppercase;
padding: 14px 32px;
border: none;
```

#### CTA Secundario / Ghost
```css
background: transparent;
color: #000000;
border: 1px solid #000000;
border-radius: 999px;
font-family: 'Futura PT', sans-serif;
font-size: 16px;
font-weight: 100;
letter-spacing: 0.10em;
text-transform: uppercase;
```

#### Botón de texto / Editorial link
```css
background: transparent;
border: none;
border-bottom: 1px solid currentColor;
border-radius: 0;
padding: 5px 0;
font-family: 'Futura PT', sans-serif;
font-size: 14px;
font-weight: 400;
letter-spacing: 0.08em;
```

### Botones que NO usar
- `border-radius: 4px` o menor — fuera de convención
- Colores de acento en CTAs
- `font-weight: 600+` en texto de botones

---

## 5. PRODUCT CARDS

### Especificaciones

```css
/* Imagen */
aspect-ratio: 1 / 1;
object-fit: cover;
border-radius: 0;           /* sin esquinas redondeadas */
filter: brightness(95%);

/* Texto: bajo la imagen */
padding: 0;                 /* sin padding horizontal */
```

### Comportamiento hover
```css
/* Estado base */
.card-title, .card-price {
  transition: opacity 0.4s ease;
  opacity: 1;
}

/* En hover */
.card:hover .card-title,
.card:hover .card-price {
  opacity: 0;
}

/* Segunda imagen: crossfade */
.card-image-secondary {
  transition: opacity 0.4s ease;
  opacity: 0;
}
.card:hover .card-image-secondary {
  opacity: 1;
}
```

### Colores de texto en card
```css
.card-title { color: #2c2c2c; }
.card-price { color: #414141; }
.card-compare-price { color: #6c6c6c; text-decoration: line-through; }
```

### Grid de productos
```css
/* Desktop */
grid-template-columns: repeat(3, 1fr);
gap: 10px;

/* Mobile */
grid-template-columns: repeat(2, 1fr);
gap: 5px;
```

---

## 6. GRID Y LAYOUT

### Page width
```css
/* Controlado por settings.max_page_width y settings.min_page_margin */
max-width: var(--page-width);
padding: 0 var(--page-margin);
```

### Secciones hero
```css
/* Full-bleed: siempre */
border-radius: 0;           /* nunca border-radius en heros */
width: 100vw;
```

### Espaciado entre secciones
No existe un sistema de spacing tokens establecido. Usar valores consistentes:
- Padding vertical de sección: `80px–120px` desktop, `48px–64px` mobile
- Gap entre elementos internos: `24px–32px`
- Microcopy / metadata: `8px–12px` del elemento padre

---

## 7. MOTION Y TRANSICIONES

### Estándar de transición
```css
transition: opacity 0.4s ease;     /* hover de imágenes y texto */
transition: transform 0.3s ease;   /* movimientos de UI (paneles, dropdowns) */
transition: all 0.2s ease;         /* microinteracciones (focus states) */
```

### Principio
- Transiciones largas (>0.5s) solo para elementos editoriales y de scroll
- Nada de bounce, spring, o easing exagerado
- El movimiento debe ser invisible — el usuario nota la ausencia, no la presencia

---

## 8. SECCIÓN DE ACORDEONES / COLLAPSE

```css
/* Trigger */
.accordion-trigger {
  font-family: 'Futura PT', sans-serif;
  font-size: 13px;
  font-weight: 400;
  letter-spacing: 0.10em;
  text-transform: uppercase;
  color: #111111;
  padding: 16px 0;
  border-bottom: 1px solid #f0f0f0;
}

/* No usar color #767676 para labels de acordeón */
/* El gris débil comunica que es lo menos importante — jerarquía incorrecta */
```

---

## 9. SIDE CART

El side cart es el componente de conversión más elaborado del tema.

### Convenciones establecidas
- Switch Cart/Favourites: pill shape (border-radius: 999px)
- Botones de empty state (Explore/Best Sellers): pill shape
- Botón Checkout: pill shape, negro, full-width
- Nombres de producto: **NO** font-weight: 600 (usar 400 o 500)

### Digital Soul note (pendiente implementar)
Cuando hay un anillo en el carrito, mostrar debajo del item:
```
"This piece will receive its Digital Soul 30 days after shipping."
font-size: 11px, color: #767676, font-weight: 400
```

---

## 10. ANNOUNCEMENT BAR

**Copy on-brand (no promo, no urgencia):**
- `"Made to order. Shipped worldwide."` ← recomendado
- `"Born in Rome, 1975."`
- `"Each piece crafted to order."`

**Copy fuera de sistema:**
- Cualquier mención de envío gratis + precio + moneda
- Cualquier copy en español si el sitio está en inglés
- Countdown, urgencia, stock

---

## 11. ICONOGRAFÍA

### Motivo de marca: △ (triángulo)
El triángulo es el símbolo definitorio de De Leville. **No implementado todavía** — debe aparecer en:
- Badge junto a "Digital Soul" en el PDP
- Divisor editorial entre secciones en The Maison
- Footer como watermark
- SVG favicon (prioridad)

```html
<!-- Triangulo Unicode (para texto) -->
△  <!-- U+25B3 WHITE UP-POINTING TRIANGLE -->

<!-- Para SVG -->
<polygon points="50,0 100,100 0,100" fill="currentColor" />
```

### Iconos de navegación (header)
Todos configurables vía settings (dark/light version). Usar versión dark (negro sobre blanco) como default.

---

## 12. PÁGINAS EDITORIALES — THE MAISON, PELLETTERIA, DIGITAL SOUL

Estas páginas son las más "maison" del sitio. Tienen más licencia editorial pero deben cumplir:

- Futura PT exclusivamente (no Cormorant, no Times New Roman)
- Fotografía full-bleed, border-radius: 0
- Copy en weight 400, nunca 600+
- Títulos uppercase con letter-spacing generoso (0.15em–0.20em)
- Espacio en blanco como elemento activo del diseño
- El triángulo △ como divisor entre secciones de copy largo

---

## 13. LOCALIZACIÓN

- **Idioma de producción:** Inglés
- Cualquier string hardcodeado en una sección debe estar en inglés o referenciado al sistema de traducción (`{{ 'key' | t }}`)
- No usar texto en español en JS, HTML inline, o en valores por defecto del schema

---

## 14. CONVENCIONES DE NAMING

### Archivos
- Secciones custom: prefijo `snc-` → `snc-nombre-seccion.liquid`
- Snippets reutilizables: prefijo `snc-` → `snc-nombre-snippet.liquid`
- NO crear secciones sin prefijo (se confunden con las del skeleton base)

### CSS custom properties (definidas en `css-variables.liquid`)
| Property | Valor correcto |
|----------|---------------|
| `--color-background` | `#ffffff` |
| `--color-foreground` | `#131313` |
| `--style-border-radius-inputs` | del setting `input_corner_radius` |
| `--page-width` | del setting `max_page_width` |
| `--page-margin` | del setting `min_page_margin` |

> Para fuentes: NO usar `var(--font-primary--family)` — usar `'Futura PT', sans-serif` directo.

---

## 15. RESPONSIVE / MOBILE-FIRST

**Regla absoluta: toda sección nueva se diseña y verifica en móvil. No es una adaptación posterior — es parte del diseño original.**

### Principios

- Diseñar desktop con excelencia, pero siempre con una perspectiva de mobile-first desde el inicio
- Nunca se entrega una sección sin haber verificado su comportamiento en móvil (375px–430px)
- Adaptar no es suficiente — la versión móvil debe sentirse igual de intencional que desktop

### CSS obligatorio para heros de imagen en móvil

```css
/* Viewport height correcto para móvil (excluye chrome del browser) */
@media (max-width: 767px) {
  .hero {
    min-height: 100vh;     /* fallback para browsers sin svh */
    min-height: 100svh;    /* browsers modernos */
  }

  /* Prevenir zoom en iOS al hacer focus en inputs */
  input {
    font-size: 16px;
  }
}
```

### Tap targets

```css
/* Mínimo 54px de altura en botones e inputs en móvil */
@media (max-width: 767px) {
  .btn, input { height: 54px; }
}
```

### Imágenes de fondo

- Siempre ofrecer `mobile_image_position` como setting separado de desktop
- El subject de la fotografía debe quedar visible en el recorte móvil (portrait 9:16)

### Breakpoints usados en el tema

| Breakpoint | Uso |
|------------|-----|
| `≤ 767px` | Mobile — breakpoint principal |
| `≤ 560px` | No usar como único breakpoint; usar 767px |
| `≥ 768px` | Desktop / tablet landscape |

---

## 16. CHECKLIST PARA NUEVA SECCIÓN

Antes de dar una sección por terminada, verificar:

- [ ] Usa `'Futura PT', sans-serif` — no Cormorant, no Funnel Sans, no Times New Roman
- [ ] Todos los botones tienen `border-radius: 999px`
- [ ] No hay `font-weight: 600` o superior en texto visible
- [ ] No hay `#2A4432` ni ningún color de otra marca
- [ ] No hay texto hardcodeado en español
- [ ] Los heros son full-bleed con `border-radius: 0`
- [ ] Transiciones de hover son `0.4s ease` o menos
- [ ] El Digital Soul badge / referencia está considerado si es contexto de producto
- [ ] El "made to order" está comunicado si es contexto de producto o colección
- [ ] No hay elementos de urgencia, countdown, o tono promo
- [ ] **Verificado en móvil (≤ 767px) — no solo adaptado, diseñado**
- [ ] **Inputs con `font-size: 16px` en móvil** (previene zoom iOS)
- [ ] **Heros usan `min-height: 100svh`** con fallback `100vh`
- [ ] **Tap targets ≥ 54px** en botones e inputs en móvil
- [ ] **`mobile_image_position`** como setting independiente si hay imagen de fondo

---

## HISTORIAL DE DECISIONES DE DISEÑO

| Fecha | Decisión | Razón |
|-------|----------|-------|
| 2026-04-29 | Todos los botones pill shape (border-radius: 999px) | Establecido en PDP y side cart — consistencia global |
| 2026-04-29 | Hover en product card: título y precio → opacity 0 | La segunda imagen es el foco — el texto distrae |
| 2026-04-29 | Cart counter: blanco sobre negro | Fix de especificidad CSS — forzado por selectores force-light-theme |
