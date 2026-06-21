# Header Transition — Estado actual (2026-06-17)

## Qué se cambió en esta sesión (fix del bug de colores)

### 1. `sections/snc-animated-product.liquid` — línea 624
**Antes:**
```html
<div class="snc-animated-product full-width color-{{ section.settings.color_scheme }}{% if is_single_banner %} snc-animated-product--simple-banner{% endif %}" {% unless is_single_banner %}style="height: {{ scroll_height }}vh;"{% endunless %}>
```
**Después:**
```html
<div class="snc-animated-product full-width color-{{ section.settings.color_scheme }}{% if is_single_banner %} snc-animated-product--simple-banner{% endif %}" data-header-theme="light" {% unless is_single_banner %}style="height: {{ scroll_height }}vh;"{% endunless %}>
```

### 2. `layout/theme.liquid` — dentro del `<script>` de header mode (~línea 649)
**Añadida línea:**
```js
window.__dlHeroTheme = heroTheme;
```
(Después de `var heroTheme = {{ dl_hero_theme | json }};`, antes del primer `if`)

### 3. `sections/snc-header.liquid` — función `applyAutoTheme()` (líneas 1106–1141)
**Antes (función original):**
```js
function applyAutoTheme() {
  if (!header) return;
  // Use elementsFromPoint to find what section is behind the header center
  const hRect = header.getBoundingClientRect();
  const cx = Math.round(window.innerWidth / 2);
  const cy = Math.round(hRect.top + hRect.height / 2);
  const stack = document.elementsFromPoint(cx, cy);
  for (const el of stack) {
    if (header.contains(el) || el === header) continue;
    // Walk up to find data-header-theme
    let node = el;
    while (node && node !== document.body) {
      if (node.dataset && node.dataset.headerTheme) {
        header.dataset.activeTheme = node.dataset.headerTheme;
        return;
      }
      node = node.parentElement;
    }
  }
  delete header.dataset.activeTheme;
}
```

**Después (función con fix):**
```js
function applyAutoTheme() {
  if (!header) return;
  // Only update data-dl-header-mode on pages that start transparent (homepage,
  // collection pages with dark hero). Solid and always-dark pages are excluded.
  var heroTheme = window.__dlHeroTheme || 'none';
  var canUpdate = heroTheme === 'dark' || heroTheme === 'light';

  // Use elementsFromPoint to find what section is behind the header center
  const hRect = header.getBoundingClientRect();
  const cx = Math.round(window.innerWidth / 2);
  const cy = Math.round(hRect.top + hRect.height / 2);
  const stack = document.elementsFromPoint(cx, cy);
  for (const el of stack) {
    if (header.contains(el) || el === header) continue;
    // Walk up to find data-header-theme
    let node = el;
    while (node && node !== document.body) {
      if (node.dataset && node.dataset.headerTheme) {
        const sectionTheme = node.dataset.headerTheme;
        header.dataset.activeTheme = sectionTheme;
        if (canUpdate) {
          // "light" theme = dark-bg section needs white icons → transparent-dark
          // "dark"  theme = light-bg section needs black icons → transparent-light
          document.documentElement.setAttribute('data-dl-header-mode',
            sectionTheme === 'light' ? 'transparent-dark' : 'transparent-light');
        }
        return;
      }
      node = node.parentElement;
    }
  }
  delete header.dataset.activeTheme;
  // Fallback: no tagged section found → assume light/white section
  if (canUpdate) {
    document.documentElement.setAttribute('data-dl-header-mode', 'transparent-light');
  }
}
```

## Cómo hacer rollback
Revertir los tres bloques de código de arriba a sus versiones "Antes".
