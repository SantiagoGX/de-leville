# Clarity Badge — Problema documentado (2026-05-12)

## Objetivo

Reemplazar el badge nativo de Microsoft Clarity ("Shop with AI" — círculo con ícono de chispa + texto expandible) con:

- **Desktop**: círculo negro 52px con el monograma `Monogram_De_Leville.png` centrado. Sin texto. Sin expansión. Al hacer clic, abre el chat de Clarity.
- **Mobile** (`pointer: coarse`): pill negro con logo + "Your Concierge" + botón ✕ para cerrar. Aparece 3 segundos después de cargar. Se descarta por sesión. Al hacer clic, abre el chat de Clarity.

Esto se logró en una sesión anterior pero se perdió con un push posterior.

---

## El problema raíz

Microsoft Clarity inyecta su badge vía el App Embed Block `brandAgents_js` (en `config/settings_data.json`). El elemento tiene `data-testid="entrypoint"` y está en el DOM regular (no shadow DOM).

**Clarity tiene su propio MutationObserver** que vigila el atributo `style` de su badge y restaura con `setProperty(prop, val, 'important')` cualquier cambio a estas propiedades:
- `display`
- `opacity`
- `visibility`
- `clip-path`

Esto hace imposible ocultarlo con CSS `!important` o con JS que modifique esas propiedades.

---

## Enfoques intentados y por qué fallaron

### 1. CSS `display: none !important` / `opacity: 0 !important`
**Falla**: Clarity restaura inmediatamente con `setProperty('display', 'flex', 'important')`. El inline `!important` de JS gana al stylesheet `!important`.

### 2. CSS `top: -9999px !important`
**Falla**: Clarity restaura `top`.

### 3. Prison approach — mover el elemento a un contenedor `overflow:hidden; width:0; height:0`
**Falla**: El badge de Clarity tiene `position: fixed`. Los elementos `position: fixed` se posicionan relativo al **viewport**, no a su padre. `overflow: hidden` en el padre **no clipea** hijos `position: fixed`. El badge sigue visible aunque esté dentro del contenedor oculto.

### 4. Prison approach + `transform: translate(0,0)` en el contenedor padre
**Teoría**: Un padre con `transform` hace que los hijos `position: fixed` se posicionen relativo a ese padre en vez del viewport.
**Falla en la práctica**: No funcionó. Posiblemente porque Clarity re-inyecta el elemento después de que sea movido, o porque el badge usa un mecanismo de posicionamiento diferente al esperado.

### 5. `transform: scale(0) !important` + `pointer-events: none !important` vía JS
**Teoría**: Clarity no vigila `transform` ni `pointer-events`, por lo que no los restauraría. El badge quedaría colapsado a tamaño 0.
**Falla en la práctica**: El badge de Clarity sigue visible. Posibles razones:
- Clarity sí vigila `transform` (no confirmado)
- Clarity crea un nuevo elemento en vez de restaurar el existente, y el MutationObserver no lo catch a tiempo
- El `querySelectorAll('[data-testid="entrypoint"]')` no está encontrando el elemento correcto (puede estar en un web component / shadow root que no es accesible directamente)

---

## Estado actual del código

**Archivo**: `layout/theme.liquid`, líneas 731–852

El bloque actual implementa el enfoque `scale(0)` (intento #5 de arriba). Contiene:

1. **CSS** para `#dl-clarity-badge` (círculo negro 52px, oculto en mobile) y `#dl-concierge-tab` (pill mobile, oculto en desktop)
2. **`#dl-clarity-badge` div** — nuestro badge personalizado con el monograma
3. **Script principal** — `hideEntrypoints()` aplica `transform: scale(0)` + `pointer-events: none` vía `setProperty`, observado con MutationObserver en `document.body`
4. **`#dl-concierge-tab` div** — pill mobile
5. **Script mobile** — muestra el pill en `pointer: coarse`, dismiss por sesión

**`config/settings_data.json`** tiene `brandAgents_js` con `"disabled": false` — Clarity sí inyecta el badge.

---

## Lo que se ve actualmente en la tienda

El badge nativo de Clarity ("Shop with AI") se muestra completo — círculo con ícono de chispa + texto — como si no hubiera ninguna intervención. Nuestro `#dl-clarity-badge` (círculo negro con monograma) no se ve.

---

## Preguntas clave para la próxima sesión

1. **¿Está `[data-testid="entrypoint"]` en el DOM regular o en un shadow root?** Verificar con `document.querySelector('[data-testid="entrypoint"]')` en la consola del browser sobre la tienda live. Si devuelve `null`, el elemento está en un shadow DOM y `querySelectorAll` no lo encuentra.

2. **¿Qué propiedades exactas vigila el MutationObserver de Clarity?** Abrir DevTools → Elements → seleccionar el badge → Breakpoints → "attribute modifications" → ver qué se restaura.

3. **¿El badge se re-inyecta o se modifica?** Si Clarity crea un elemento nuevo cada vez que removemos el existente, el MutationObserver puede estar en un loop pero siempre llegando tarde.

4. **Alternativa nuclear**: Deshabilitar `brandAgents_js` en `settings_data.json` (`"disabled": true`) e inyectar el script de Clarity Brand Agent manualmente con configuración custom (si Clarity expone un API para ocultar el launcher nativo).

---

## Archivos relevantes

- `layout/theme.liquid` — líneas 731–852: todo el código del badge
- `config/settings_data.json` — bloque `"2200952712151285029"` (brandAgents_js)
- `CLARITY-BADGE-ROLLBACK.md` — snapshot de una versión anterior con el enfoque CSS puro (no funciona, solo referencia)
- `assets/Monogram_De_Leville.png` — logo que debe aparecer en el badge
