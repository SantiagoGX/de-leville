# Header Smoothness — Rollback al estado PRE-mejora (2026-06-17)

Este archivo documenta el estado EXACTO del header ANTES de la mejora de transición smooth.
El bug de colores (header blanco en secciones blancas) YA ESTÁ CORREGIDO en este estado.
Si la mejora de smoothness falla, revertir a estos bloques exactos.

---

## Archivo: `sections/snc-header.liquid`

### Bloque A — `.custom-header--icon-button` (línea ~2810)
```css
.custom-header--icon-button {
  background: none;
  border: none;
  padding: 0;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  text-decoration: none;
  color: inherit;
  margin: 0;
}
```

### Bloque B — `.custom-header--icon` (línea ~2828)
```css
.custom-header--icon {
  display: block;
  object-fit: contain;
  filter: brightness(0) saturate(100%) invert(20%) sepia(15%) saturate(1500%) hue-rotate(90deg) brightness(0.3);
}
```

### Bloque C — `.custom-header--logo-image` (línea ~2741)
```css
.custom-header--logo-image {
  width: {{ settings.logo_width | default: 135 }}px;
  height: auto;
  display: block;
  object-fit: contain;
  object-position: center;
  grid-area: 1 / 1;
  transition: opacity 200ms ease;
}
```

### Bloque D — System 1 defaults + active-theme + force-light-theme (líneas ~3996-4047)
```css
/* 2. Default: dark icons — correct for light/white sections */
.custom-header--root .icon-static,
.custom-header--root .logo-static {
  display: none !important;
}
.custom-header--root .icon-sticky,
.custom-header--root .logo-sticky {
  display: block !important;
}

/* 3. Dark section in view (hero, dark video) → white icons.
      Applies in ALL states: sticky, non-sticky, force-sticky.
      opacity overrides here beat the mid-file is-sticky opacity rules
      (same specificity 0,3,0 — last source wins with !important). */
.custom-header--root[data-active-theme="light"] .icon-static,
.custom-header--root[data-active-theme="light"] .logo-static {
  display: block !important;
  opacity: 1 !important;
  pointer-events: auto !important;
}
.custom-header--root[data-active-theme="light"] .icon-sticky,
.custom-header--root[data-active-theme="light"] .logo-sticky {
  display: none !important;
  opacity: 0 !important;
  pointer-events: none !important;
}

/* 4. force-light-theme — HIGHEST PRIORITY OVERRIDE for non-home pages. */
.custom-header--root.force-light-theme .logo-sticky,
.custom-header--root.force-light-theme .icon-sticky,
.custom-header--root.force-light-theme[data-active-theme="light"] .logo-sticky,
.custom-header--root.force-light-theme[data-active-theme="light"] .icon-sticky,
.custom-header--root.force-light-theme.is-sticky .logo-sticky,
.custom-header--root.force-light-theme.is-sticky .icon-sticky {
  display: block !important;
}
.custom-header--root.force-light-theme .logo-static,
.custom-header--root.force-light-theme .icon-static,
.custom-header--root.force-light-theme[data-active-theme="light"] .logo-static,
.custom-header--root.force-light-theme[data-active-theme="light"] .icon-static,
.custom-header--root.force-light-theme.is-sticky .logo-static,
.custom-header--root.force-light-theme.is-sticky .icon-static {
  display: none !important;
}
```

### Bloque E — System 2 transparent-dark/light display rules (líneas ~4100-4127)
```css
/* TRANSPARENT · DARK hero — white logo + white icons over dark imagery */
html[data-dl-header-mode="transparent-dark"] .custom-header--root .logo-static,
html[data-dl-header-mode="transparent-dark"] .custom-header--root .icon-static {
  display: block !important;
}
html[data-dl-header-mode="transparent-dark"] .custom-header--root .logo-sticky,
html[data-dl-header-mode="transparent-dark"] .custom-header--root .icon-sticky {
  display: none !important;
}

/* TRANSPARENT · LIGHT hero — black logo + black icons over light imagery */
html[data-dl-header-mode="transparent-light"] .custom-header--root .logo-sticky,
html[data-dl-header-mode="transparent-light"] .custom-header--root .icon-sticky {
  display: block !important;
}
html[data-dl-header-mode="transparent-light"] .custom-header--root .logo-static,
html[data-dl-header-mode="transparent-light"] .custom-header--root .icon-static {
  display: none !important;
}
```

### Bloque F — System 2 solid display rules (líneas ~4145-4156)
```css
html[data-dl-header-mode="solid"] .custom-header--root .logo-sticky,
html[data-dl-header-mode="solid"] .custom-header--root .icon-sticky {
  display: block !important;
  opacity: 1 !important;
  pointer-events: auto !important;
}
html[data-dl-header-mode="solid"] .custom-header--root .logo-static,
html[data-dl-header-mode="solid"] .custom-header--root .icon-static {
  display: none !important;
  opacity: 0 !important;
  pointer-events: none !important;
}
```

### Bloque G — Transition list (líneas ~4164-4171)
```css
.custom-header--root,
.custom-header--root a,
.custom-header--root .custom-header--nav-link,
.custom-header--root .custom-header--burger-icon,
.custom-header--root .custom-header--icon {
  transition: background-color 300ms ease, color 300ms ease, filter 300ms ease, border-color 300ms ease;
}
```

### Bloque H — `applyAutoTheme()` JS (líneas ~1106-1144)
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
