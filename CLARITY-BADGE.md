# Clarity Badge — Estado (2026-05-12)

## Qué funciona ✓

### Desktop
- Círculo negro → hover expand "YOUR CONCIERGE" Futura PT → click abre chat → reaparece al cerrar ✓

### Mobile
- Pill aparece expandido 2.5s después de cargar ✓
- Colapsa a círculo en 3s o al primer scroll ✓
- Logo centrado en círculo (`padding: 17px` simétrico en `.dl-tab--collapsed`) ✓
- Tap abre chat ✓
- Dismiss (✕) por página (`dl-tab-dismissed-<pathname>`) — reaparece en nuevas páginas ✓
- Al cerrar chat → pill regresa como círculo ✓

---

## Bug activo — Scroll al top en mobile al abrir el chat ✗

### Comportamiento observado (confirmado en video de dispositivo real)
1. Usuario está scrolleado en la página (ej. mitad del grid de colección)
2. Toca el pill → **la página salta al top inmediatamente**
3. El chat de Clarity se abre como **bottom sheet** (~60% de la pantalla)
4. La página es visible DETRÁS del chat — en posición 0 (top)
5. Usuario cierra el chat
6. La página scrollea de vuelta a donde estaba (visible, no ideal pero funciona)

### Lo que se debuggeó esta sesión

**Debugging en Chrome (equivocado):**
- Se interceptaron `window.scrollTo`, `window.scrollBy`, `Element.prototype.scrollIntoView`, `document.body.scrollTop` — ninguno disparó
- MutationObserver en `body.style` — no disparó
- `window.scrollY` devuelve 2223.5 correctamente
- El debug fue en Chrome desktop, no en Safari iOS donde ocurre el bug — todos los resultados son inválidos

**Fixes intentados (todos fallaron en Safari iOS real):**
1. `lockBodyScroll()` original: `position: fixed; top: -scrollY` en body via `setProperty('!important')` — fallaba, el jump ocurría igual
2. Sin CSS lock: solo guardar `_savedScrollY`, sin tocar DOM — jump sigue (confirma que es Clarity/iOS, no nuestro CSS)
3. Scroll event listener: interceptar primer `scroll` y llamar `scrollTo(savedY)` — empeoró (ping-pong visible)
4. Cover div fullscreen (`z-index: 99998`) con `setTimeout(hideCover, 400)` — incorrecto para bottom sheet (la página sigue siendo visible detrás del chat)
5. CSS class `dl-scroll-locked` en body con `--dl-lock-top` en html — fallaba igual
6. CSS class en `html` Y `body` + doble `requestAnimationFrame` antes del dispatch + unlock mejorado — fallaba igual

### Lo que SE SABE con certeza
- El chat de Clarity en iOS Safari es un **bottom sheet**, no pantalla completa — la página es visible arriba del chat durante toda la conversación
- El scroll jump ocurre cuando Clarity abre — ninguna API JS de scroll es la causa detectable (desde Chrome)
- iOS Safari hace algo a nivel del motor del browser que resetea el scroll cuando Clarity inicializa
- No se puede debuggear desde Chrome — se necesita **Safari Web Inspector** (iPhone + Mac + USB)

### Estado actual del código en `theme.liquid`
```javascript
// Implementación actual (no resuelve el bug):
function lockBodyScroll() {
  _savedScrollY = window.scrollY || window.pageYOffset;
  document.documentElement.style.setProperty('--dl-lock-top', '-' + _savedScrollY + 'px');
  document.documentElement.classList.add('dl-scroll-locked');
  document.body.classList.add('dl-scroll-locked');
}

function unlockBodyScroll() {
  // Técnica robusta para close: freeze visual → restore scroll → unfreeze
  document.body.style.position = 'fixed';
  document.body.style.top = '-' + _savedScrollY + 'px';
  document.body.style.width = '100%';
  document.documentElement.classList.remove('dl-scroll-locked');
  document.body.classList.remove('dl-scroll-locked');
  document.documentElement.style.removeProperty('--dl-lock-top');
  window.scrollTo(0, _savedScrollY);
  document.body.style.position = '';
  document.body.style.top = '';
  document.body.style.width = '';
}
// PointerEvent dispatch usa doble requestAnimationFrame
```

```css
html.dl-scroll-locked { overflow: hidden !important; height: 100% !important; }
body.dl-scroll-locked { position: fixed !important; top: var(--dl-lock-top, 0px) !important; width: 100% !important; overflow: hidden !important; }
```

### Para resolver en próxima sesión

**Prerequisito**: Conectar iPhone al Mac con cable USB y usar **Safari Web Inspector** (Safari en Mac → Develop → [nombre del iPhone] → deleville.co). Sin esto seguimos debuggeando a ciegas.

Con Safari Web Inspector activo:
1. Interceptar todas las APIs de scroll en el contexto real de iOS Safari
2. Ver si el jump viene de JS (Clarity) o del motor del browser (iOS)
3. Ver si nuestro CSS class lock se aplica correctamente o si algo lo sobreescribe

**Opciones de fix no intentadas aún:**
- `body-scroll-lock` npm library (librería probada en producción para exactamente este problema de iOS Safari)
- Interceptar `window.scrollTo` como noop durante la apertura del chat (si se confirma que Clarity lo llama)
- Investigar si la arquitectura de PointerEvent dispatch está causando el problema (alternativa: exponer el botón nativo de Clarity directamente)

---

## Arquitectura técnica

### Host element
`#ads-agent-host` en DOM normal. Badge `[data-testid="entrypoint"]` en `host.shadowRoot` (open).

### Ocultamiento
`transform: scale(0)` en el host. Clarity no vigila el host, solo el badge interno.

### Click
`PointerEvent({ composed: true, isPrimary: true })` — cruza shadow DOM boundary a React. Dispatch con doble `requestAnimationFrame`.

### Flag `_chatOpen`
Pausa el MO de `hideHost()` mientras el chat está activo.

### Detección de cierre
MO sobre shadow root. `sr.querySelectorAll('[data-testid]').length <= 1` → chat cerrado.

### Colapso pill mobile
`max-width: 260px → 52px` + `padding 14px → 17px` simétrico. Timer 3s O primer scroll.

### Dismiss por página
`sessionStorage` key = `dl-tab-dismissed-<window.location.pathname>`.

---

## Código
`layout/theme.liquid` — bloque `<!-- De Leville: Concierge badge -->` hasta `</body>`.
