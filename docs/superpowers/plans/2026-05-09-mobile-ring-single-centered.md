# Mobile Ring — Single Centered Ring Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development to implement this plan task-by-task. Launch one fresh subagent per task, review the result, then proceed to the next. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** En móvil, reemplazar los dos anillos divididos (50% ancho cada uno) por un único anillo grande centrado, con animación configurable desde el schema de Shopify (float, rotate_cw, float_rotate, none), y espaciado equilibrado entre el título, los nombres de colección y el anillo.

**Architecture:** Todos los cambios están confinados a `sections/snc-animated-collections.liquid`. Se agrega un tercer contenedor Liquid `.images.mobile-only` que renderiza la imagen seleccionada por el setting `mobile_image` de cada bloque. En desktop se oculta con CSS; en móvil se ocultan `.left` y `.right` y se muestra `.mobile-only` usando `flex: 1` para llenar el espacio disponible sin zona muerta. El JS existente (activa por `data-index`) funciona sin cambios porque el nuevo contenedor usa el mismo atributo.

**Tech Stack:** Liquid, CSS (keyframes, flexbox, CSS custom properties), Shopify schema JSON. Sin build system — cambios directos al archivo.

---

## Archivo afectado

- Modify: `sections/snc-animated-collections.liquid`

---

## Task 1: Agregar settings al schema — mobile_ring_animation y mobile_ring_size

**Files:**
- Modify: `sections/snc-animated-collections.liquid` (bloque `{% schema %}`, sección "Collection Style")

- [ ] **Step 1: Localizar el punto de inserción en el schema**

Abrir `sections/snc-animated-collections.liquid`. El schema está al inicio del archivo (líneas 1–276). Buscar el setting `inactive_opacity` (alrededor de línea 122). El nuevo header y settings se insertan **después** del cierre de ese objeto (después de la línea que dice `"default": 20`).

- [ ] **Step 2: Insertar los dos nuevos settings**

Reemplazar este fragmento en el schema:

```json
    {
      "type": "range",
      "id": "inactive_opacity",
      "min": 0,
      "max": 100,
      "step": 5,
      "unit": "%",
      "label": "Inactive opacity",
      "default": 20
    },
    {
      "type": "header",
      "content": "Button"
    },
```

Con este (agrega el header "Mobile Ring" y los dos settings nuevos):

```json
    {
      "type": "range",
      "id": "inactive_opacity",
      "min": 0,
      "max": 100,
      "step": 5,
      "unit": "%",
      "label": "Inactive opacity",
      "default": 20
    },
    {
      "type": "header",
      "content": "Mobile Ring"
    },
    {
      "type": "select",
      "id": "mobile_ring_animation",
      "label": "Mobile ring animation",
      "options": [
        { "value": "float", "label": "Float (sube y baja)" },
        { "value": "rotate_cw", "label": "Rotate clockwise" },
        { "value": "float_rotate", "label": "Float + rotate" },
        { "value": "none", "label": "None (estático)" }
      ],
      "default": "float"
    },
    {
      "type": "range",
      "id": "mobile_ring_size",
      "min": 50,
      "max": 95,
      "step": 5,
      "unit": "%",
      "label": "Tamaño del anillo en móvil (% del viewport)",
      "default": 78
    },
    {
      "type": "header",
      "content": "Button"
    },
```

- [ ] **Step 3: Verificar que el schema es JSON válido**

```bash
# Desde el directorio del proyecto de-leville
node -e "
const fs = require('fs');
const content = fs.readFileSync('sections/snc-animated-collections.liquid', 'utf8');
const schemaMatch = content.match(/\{%\s*schema\s*%\}([\s\S]*?)\{%\s*endschema\s*%\}/);
JSON.parse(schemaMatch[1]);
console.log('Schema JSON válido');
"
```

Expected: `Schema JSON válido`

---

## Task 2: Agregar contenedor .mobile-only en el HTML Liquid

**Files:**
- Modify: `sections/snc-animated-collections.liquid` (bloque HTML, después del cierre `</div>` del contenedor `.images.right`)

- [ ] **Step 1: Localizar el punto de inserción**

En el HTML (alrededor de línea 587–597), buscar el cierre del contenedor `.images.right`:

```liquid
      </div>
    </div>

    {% if section.settings.button_label != blank %}
```

El nuevo contenedor `.mobile-only` se inserta entre el cierre de `.images.right` y el `{% if section.settings.button_label %}`.

- [ ] **Step 2: Insertar el contenedor .mobile-only**

Reemplazar:

```liquid
      <div class="snc-animated-collections__images right">
        {% for block in section.blocks %}
          <div class="snc-ac-image-wrapper" data-index="{{ forloop.index0 }}"
            style="width: {{ block.settings.image_width | default: section.settings.image_width }}px; {% if block.settings.image_height > 0 %}height: {{ block.settings.image_height }}px;{% endif %}">
            {% if block.settings.image_2 != blank %}
              {{ block.settings.image_2 | image_url: width: 600 | image_tag: loading: 'lazy' }}
            {% endif %}
          </div>
        {% endfor %}
      </div>
    </div>

    {% if section.settings.button_label != blank %}
```

Con:

```liquid
      <div class="snc-animated-collections__images right">
        {% for block in section.blocks %}
          <div class="snc-ac-image-wrapper" data-index="{{ forloop.index0 }}"
            style="width: {{ block.settings.image_width | default: section.settings.image_width }}px; {% if block.settings.image_height > 0 %}height: {{ block.settings.image_height }}px;{% endif %}">
            {% if block.settings.image_2 != blank %}
              {{ block.settings.image_2 | image_url: width: 600 | image_tag: loading: 'lazy' }}
            {% endif %}
          </div>
        {% endfor %}
      </div>

      <div class="snc-animated-collections__images mobile-only">
        {% for block in section.blocks %}
          <div class="snc-ac-image-wrapper" data-index="{{ forloop.index0 }}">
            {% if block.settings.mobile_image == 'image_2' %}
              {% if block.settings.image_2 != blank %}
                {{ block.settings.image_2 | image_url: width: 800 | image_tag: loading: 'lazy' }}
              {% endif %}
            {% else %}
              {% if block.settings.image_1 != blank %}
                {{ block.settings.image_1 | image_url: width: 800 | image_tag: loading: 'lazy' }}
              {% endif %}
            {% endif %}
          </div>
        {% endfor %}
      </div>
    </div>

    {% if section.settings.button_label != blank %}
```

---

## Task 3: CSS — ocultar left/right en móvil, mostrar mobile-only con flex:1

**Files:**
- Modify: `sections/snc-animated-collections.liquid` (bloque `<style>`)

- [ ] **Step 1: Ocultar .mobile-only en desktop**

Al final del bloque `<style>`, antes del cierre `</style>`, agregar:

```css
  /* Desktop: ocultar el contenedor mobile-only */
  @media (min-width: 769px) {
    .snc-animated-collections__images.mobile-only {
      display: none !important;
    }
  }
```

- [ ] **Step 2: En el bloque `@media (max-width: 768px)` — ocultar left y right, mostrar mobile-only**

Dentro del bloque `@media (max-width: 768px)` existente, localizar las reglas de `.snc-animated-collections__images` (alrededor de donde dice `position: absolute !important; bottom: 0 !important; height: 52vh !important; width: 50% !important`).

Reemplazar **todas** las reglas de `.snc-animated-collections__images` dentro del `@media (max-width: 768px)` — incluyendo `.left`, `.right`, `.snc-ac-image-wrapper` y `.snc-ac-image-wrapper img` — con el siguiente bloque completo:

```css
    /* Ocultar left y right en móvil */
    .snc-animated-collections__images.left,
    .snc-animated-collections__images.right {
      display: none !important;
    }

    /* Contenedor único centrado — ocupa todo el espacio vertical restante */
    .snc-animated-collections__images.mobile-only {
      display: flex !important;
      position: relative !important;
      flex: 1;
      width: 100% !important;
      align-items: center;
      justify-content: center;
      pointer-events: none;
      padding-bottom: 40px; /* espacio para el footer absoluto */
    }

    /* Apilar todos los wrappers en el mismo espacio (solo el is-active es visible) */
    .snc-animated-collections__images.mobile-only .snc-ac-image-wrapper {
      position: absolute !important;
      inset: 0 !important;
      width: 100% !important;
      height: 100% !important;
      display: flex;
      align-items: center;
      justify-content: center;
    }

    .snc-animated-collections__images.mobile-only .snc-ac-image-wrapper img {
      width: {{ section.settings.mobile_ring_size }}vw !important;
      max-height: 100%;
      height: auto;
      object-fit: contain;
      padding: 0;
    }
```

- [ ] **Step 3: Asegurarse de que el sticky wrapper tenga display flex en móvil**

Dentro del `@media (max-width: 768px)`, la regla de `.snc-ac-sticky-wrapper` debe incluir `display: flex` y `flex-direction: column`. Verificar que la regla existente lo dice:

```css
    .snc-ac-sticky-wrapper {
      top: var(--custom-header-total-height, 0px);
      height: calc(100vh - var(--custom-header-total-height, 0px)) !important;
      padding: 0 !important;
      justify-content: flex-start;
      overflow: hidden;
      position: sticky !important;
      position: -webkit-sticky !important;
      /* Agregar si no están presentes: */
      display: flex;
      flex-direction: column;
    }
```

Si ya tiene `display: flex` y `flex-direction: column` heredadas del desktop (la regla global de `.snc-ac-sticky-wrapper` sí las tiene implícitamente via el flex del content wrapper), el `flex: 1` del `.mobile-only` funcionará. Verificar en el preview — si el anillo no ocupa el espacio restante, agregar `display: flex; flex-direction: column;` explícitamente a la regla mobile del `.snc-ac-sticky-wrapper`.

---

## Task 4: CSS — keyframes y animación controlada por setting Liquid

**Files:**
- Modify: `sections/snc-animated-collections.liquid` (bloque `<style>`, dentro del `@media (max-width: 768px)`)

- [ ] **Step 1: Agregar keyframes de rotación**

Dentro del `@media (max-width: 768px)`, justo después de los keyframes `snc-mobile-rock-left` y `snc-mobile-rock-right` existentes, agregar:

```css
    @keyframes snc-ring-rotate-cw {
      from { transform: rotate(0deg); }
      to   { transform: rotate(360deg); }
    }

    @keyframes snc-ring-float-rotate {
      0%   { transform: translateY(0px) rotate(0deg); }
      50%  { transform: translateY(-14px) rotate(180deg); }
      100% { transform: translateY(0px) rotate(360deg); }
    }
```

- [ ] **Step 2: Agregar las reglas de animación condicionales con Liquid**

Reemplazar las dos reglas de animación existentes de `.left` y `.right` que aplican `snc-mobile-rock-left` y `snc-mobile-rock-right`:

```css
    .snc-animated-collections__images.left .snc-ac-image-wrapper.is-active img {
      animation: snc-mobile-rock-left 5s ease-in-out infinite;
    }
    .snc-animated-collections__images.right .snc-ac-image-wrapper.is-active img {
      animation: snc-mobile-rock-right 5.5s ease-in-out infinite;
    }
```

Con (estas ya no aplican porque `.left` y `.right` están ocultos, pero se mantienen como fallback y se agrega la nueva regla para `.mobile-only`):

```css
    .snc-animated-collections__images.left .snc-ac-image-wrapper.is-active img {
      animation: snc-mobile-rock-left 5s ease-in-out infinite;
    }
    .snc-animated-collections__images.right .snc-ac-image-wrapper.is-active img {
      animation: snc-mobile-rock-right 5.5s ease-in-out infinite;
    }

    {% if section.settings.mobile_ring_animation == 'float' %}
    .snc-animated-collections__images.mobile-only .snc-ac-image-wrapper.is-active img {
      animation: snc-mobile-rock-left 5s ease-in-out infinite;
    }
    {% elsif section.settings.mobile_ring_animation == 'rotate_cw' %}
    .snc-animated-collections__images.mobile-only .snc-ac-image-wrapper.is-active img {
      animation: snc-ring-rotate-cw 8s linear infinite;
    }
    {% elsif section.settings.mobile_ring_animation == 'float_rotate' %}
    .snc-animated-collections__images.mobile-only .snc-ac-image-wrapper.is-active img {
      animation: snc-ring-float-rotate 6s ease-in-out infinite;
    }
    {% else %}
    .snc-animated-collections__images.mobile-only .snc-ac-image-wrapper.is-active img {
      animation: none;
    }
    {% endif %}
```

---

## Task 5: CSS — espaciado equilibrado entre título, nombres y anillo

**Files:**
- Modify: `sections/snc-animated-collections.liquid` (bloque `<style>`, `@media (max-width: 768px)`)

- [ ] **Step 1: Aumentar el margen inferior del header**

Dentro del `@media (max-width: 768px)`, localizar la regla de `.snc-animated-collections__header`:

```css
    .snc-animated-collections__header {
      padding: 28px 20px 0;
      margin-bottom: 8px !important;
      text-align: center;
      flex-shrink: 0;
    }
```

Cambiar `margin-bottom: 8px` a `margin-bottom: 20px`:

```css
    .snc-animated-collections__header {
      padding: 28px 20px 0;
      margin-bottom: 20px !important;
      text-align: center;
      flex-shrink: 0;
    }
```

- [ ] **Step 2: Agregar flex-shrink: 0 a la lista para que el anillo absorba el espacio restante**

Dentro del `@media (max-width: 768px)`, localizar la regla de `.snc-animated-collections__list`:

```css
    .snc-animated-collections__list {
      width: 100%;
      gap: 10px;
      padding: 0 20px;
      flex-shrink: 0;
      margin-bottom: 0;
    }
```

Verificar que tenga `flex-shrink: 0` (ya está). Si no está, agregarlo. El anillo con `flex: 1` absorberá todo el espacio vertical que la lista no ocupe.

- [ ] **Step 3: Verificar que .snc-animated-collections__footer tenga z-index sobre el anillo**

El footer tiene `position: absolute; bottom: 8px; z-index: 20`. El anillo `.mobile-only` tiene `position: relative` y no tiene z-index. No hay conflicto — el footer ya está sobre todo. Verificar visualmente en el preview que "VIEW ALL" es visible.

---

## Task 6: Verificación visual en los tres tamaños de móvil

No hay tests automatizados — la verificación es visual en el Shopify theme preview.

- [ ] **Step 1: Push del tema al store de desarrollo**

```bash
shopify theme push --development
```

O iniciar dev server:

```bash
shopify theme dev
```

- [ ] **Step 2: Verificar en móvil pequeño (320–375px)**

Abrir la homepage en Chrome DevTools con viewport 375px (iPhone SE). Verificar:
- [ ] Solo un anillo visible, centrado horizontalmente
- [ ] El anillo ocupa ~78% del ancho (sin el setting modificado, el default)
- [ ] No hay zona muerta entre los nombres de colección y el anillo
- [ ] "THE COLLECTIONS" tiene espacio respirable sobre los nombres
- [ ] Al hacer scroll, el anillo cambia según la colección activa
- [ ] La animación seleccionada (`float` por defecto) funciona

- [ ] **Step 3: Verificar en móvil mediano (390–414px)**

Viewport 390px (iPhone 14). Mismos checks.

- [ ] **Step 4: Verificar en móvil grande (428px+)**

Viewport 428px (iPhone 14 Pro Max). Mismos checks.

- [ ] **Step 5: Verificar en desktop (≥769px)**

Viewport 1280px. Verificar:
- [ ] Los dos anillos (izquierdo y derecho) siguen igual que antes
- [ ] El contenedor `.mobile-only` no es visible
- [ ] El hover de colecciones funciona igual
- [ ] El scroll-driven activation funciona igual

- [ ] **Step 6: Verificar los 3 modos de animación desde el editor de Shopify**

En el Shopify Customizer, cambiar `Mobile ring animation` a cada valor y verificar en el preview móvil:
- [ ] `Float` — anillo sube y baja suavemente
- [ ] `Rotate clockwise` — anillo gira continuo en sentido horario
- [ ] `Float + rotate` — anillo sube/baja mientras gira 360°
- [ ] `None` — anillo estático

- [ ] **Step 7: Verificar el setting mobile_ring_size**

Cambiar `Tamaño del anillo en móvil` a 50%, 78% (default), 95% y verificar que el anillo cambia de tamaño en el preview.

---

## Notas de implementación

**Por qué `flex: 1` en lugar de `position: absolute`:**
El layout anterior usaba `position: absolute; bottom: 0; height: 52vh` para ambos contenedores `.left` y `.right`. Esto creaba una zona muerta entre el final de los nombres de colección y el inicio del área de las imágenes. Con `flex: 1`, el contenedor `.mobile-only` ocupa exactamente todo el espacio vertical restante entre la lista de nombres y el footer, eliminando el gap sin necesidad de medir heights manualmente.

**Por qué el JS no necesita cambios:**
El JS activa imágenes buscando todos los `.snc-ac-image-wrapper[data-index="N"]` dentro del section. El nuevo contenedor `.mobile-only` usa el mismo `data-index` por bloque. En desktop, `.mobile-only` está oculto con `display: none`, así que el JS puede activar sus wrappers sin efecto visual. En móvil, `.left` y `.right` están ocultos, así que solo el `.mobile-only` muestra el cambio. Sin colisiones.

**Si el anillo no ocupa el espacio restante (flex:1 no funciona):**
Verificar que `.snc-ac-sticky-wrapper` tenga `display: flex; flex-direction: column` en móvil. Agregar estas propiedades explícitamente dentro del `@media (max-width: 768px)` de `.snc-ac-sticky-wrapper` si no están presentes.
