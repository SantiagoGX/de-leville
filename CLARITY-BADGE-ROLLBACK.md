# Clarity Badge — Snapshot para rollback (estado al 2026-05-12)

Si los cambios de collapse/expand + mobile no funcionan y quieres regresar exactamente a este estado, reemplaza el bloque en `layout/theme.liquid` desde la línea `<!-- De Leville: Clarity agent badge — brand override -->` hasta el cierre de `</body>` con el código de abajo.

---

## Comportamiento en este snapshot

- **Desktop**: badge siempre expandido (círculo + "YOUR CONCIERGE" siempre visible). Logo monograma funciona. Hover flash fix activo. Collapse/expand NO funciona porque `getTextWrapper` busca en DOM normal, no en shadow root.
- **Mobile**: badge de Clarity sigue visible (Clarity aplica inline styles que ganan al CSS). Pill `#dl-concierge-tab` no aparece porque el badge no se oculta primero.

---

## Código completo del bloque (líneas 731–1030 en theme.liquid)

```liquid
    <!-- De Leville: Clarity agent badge — brand override -->
    <style>
      /* Always black badge */
      [data-testid="entrypoint"] [aria-label="Shop with AI"] {
        background: #000 !important;
      }
      [data-testid="entrypoint"] .entrypoint-shadow {
        border-color: rgba(255,255,255,0.12) !important;
        transition: box-shadow 0.4s ease !important;
      }
      /* Subtle glow on hover (CSS can do this safely) */
      @media (pointer: fine) {
        [data-testid="entrypoint"]:hover .entrypoint-shadow {
          box-shadow: 0 6px 28px rgba(0,0,0,0.4) !important;
        }
      }

      /* Icon area: white circle, DL monogram via background-image (set by JS) */
      [data-testid="entrypoint"] .agent-avatar-bg {
        display: flex !important;
        align-items: center !important;
        justify-content: center !important;
      }

      /* Brand logo — white mark on black (compact: 40% of circle width, contained) */
      [data-testid="entrypoint"] .dl-badge-logo {
        width: 40% !important;
        height: 40% !important;
        max-width: 22px !important;
        max-height: 22px !important;
        display: block !important;
        object-fit: contain !important;
        filter: invert(1) !important;
        flex-shrink: 0 !important;
      }

      /* Text: hide original via font-size 0, inject via ::after (zero flash) */
      [data-testid="entrypoint"] .min-w-0.truncate {
        font-size: 0 !important;
        padding: 0 4px !important;
      }
      [data-testid="entrypoint"] .min-w-0.truncate::after {
        content: 'YOUR CONCIERGE';
        font-family: 'Futura PT', sans-serif !important;
        font-weight: 400 !important;
        font-size: 11px !important;
        letter-spacing: 0.18em !important;
        text-transform: uppercase !important;
        color: #fff !important;
        white-space: nowrap !important;
      }

      /* Mobile (touch): hide Clarity badge entirely — custom tab handles it */
      @media (pointer: coarse) {
        [data-testid="entrypoint"] {
          opacity: 0 !important;
          pointer-events: none !important;
          position: fixed !important;
          top: -9999px !important;
        }
      }

      /* ── Mobile concierge pill (bottom-right) ── */
      #dl-concierge-tab {
        display: none;
        position: fixed;
        bottom: 24px;
        right: 16px;
        z-index: 9999;
        background: #000;
        border-radius: 999px;
        padding: 11px 16px 11px 12px;
        align-items: center;
        gap: 8px;
        cursor: pointer;
        opacity: 0;
        transform: translateY(8px);
        transition: opacity 0.4s ease, transform 0.4s ease;
        -webkit-tap-highlight-color: transparent;
        user-select: none;
        box-shadow: 0 4px 20px rgba(0,0,0,0.45);
      }
      #dl-concierge-tab.dl-tab--visible {
        opacity: 1;
        transform: translateY(0);
      }
      #dl-concierge-tab .dl-tab__logo {
        width: 16px;
        height: 14px;
        display: block;
        flex-shrink: 0;
        filter: invert(1);
      }
      #dl-concierge-tab .dl-tab__text {
        font-family: 'Futura PT', sans-serif;
        font-size: 9px;
        letter-spacing: 0.2em;
        text-transform: uppercase;
        color: #fff;
        white-space: nowrap;
      }
      #dl-concierge-tab .dl-tab__close {
        font-size: 11px;
        color: rgba(255,255,255,0.4);
        margin-left: 4px;
        line-height: 1;
      }
      @media (pointer: fine) {
        #dl-concierge-tab { display: none !important; }
      }
    </style>
    <script>
      (function () {
        var TARGET = 'Your Concierge';
        var LOGO_URL = '{{ "Monogram_De_Leville.png" | asset_url }}';
        var isMouse = window.matchMedia('(pointer: fine)').matches;
        var isHovered = false;
        var labelEl = null;
        var labelObs = null;

        // ── Text label observer ──────────────────────────────────────────
        function attachLabelObserver(el) {
          if (labelObs) labelObs.disconnect();
          labelEl = el;
          labelObs = new MutationObserver(function () {
            if (labelEl.textContent.trim() !== TARGET) labelEl.textContent = TARGET;
          });
          labelObs.observe(labelEl, { childList: true, characterData: true, subtree: true });
        }

        // ── Locate text wrapper inside badge ─────────────────────────────
        function getTextWrapper() {
          var label = document.querySelector('[data-testid="entrypoint"] .min-w-0.truncate') ||
                      document.querySelector('[data-testid="entrypoint"] .min-w-0');
          return label ? label.parentElement : null;
        }

        // ── Collapse / expand via setProperty (beats Clarity inline styles) ──
        function collapseText(tw, instant) {
          if (!tw) return;
          if (instant) tw.style.setProperty('transition', 'none', 'important');
          tw.style.setProperty('overflow', 'hidden', 'important');
          tw.style.setProperty('max-width', '0', 'important');
          tw.style.setProperty('opacity', '0', 'important');
          var inner = tw.querySelector('.min-w-0');
          if (inner) inner.style.setProperty('transform', 'translateX(-5px)', 'important');
          if (instant) {
            requestAnimationFrame(function () {
              requestAnimationFrame(function () {
                if (tw.parentNode) {
                  tw.style.setProperty('transition',
                    'max-width 0.52s cubic-bezier(.22,.61,.36,1), opacity 0.32s ease',
                    'important');
                }
              });
            });
          }
        }

        function expandText(tw) {
          if (!tw) return;
          tw.style.setProperty('max-width', '192px', 'important');
          tw.style.setProperty('opacity', '1', 'important');
          var inner = tw.querySelector('.min-w-0');
          if (inner) {
            if (!inner.style.transition) {
              inner.style.setProperty('transition',
                'transform 0.48s cubic-bezier(.22,.61,.36,1) 0.06s', 'important');
            }
            inner.style.setProperty('transform', 'translateX(0)', 'important');
          }
        }

        // ── Apply current hover state to text wrapper ─────────────────────
        function applyHoverState() {
          if (!isMouse) return;
          var tw = getTextWrapper();
          if (!tw) return;
          if (!tw._dlReady) {
            tw._dlReady = true;
            if (isHovered) {
              expandText(tw);
            } else {
              collapseText(tw, true);
            }
          } else {
            if (isHovered) expandText(tw); else collapseText(tw, false);
          }
        }

        // ── DOM patching (logo, text label) ──────────────────────────────
        function patchRoot(root) {
          var ctx = root || document;

          ctx.querySelectorAll('.agent-avatar-bg').forEach(function (el) {
            el.querySelectorAll('svg').forEach(function (s) {
              s.style.setProperty('display', 'none', 'important');
            });
            el.style.setProperty('background-color', '#ffffff', 'important');
            el.style.setProperty('background-image', 'url("' + LOGO_URL + '")', 'important');
            el.style.setProperty('background-size', '55%', 'important');
            el.style.setProperty('background-repeat', 'no-repeat', 'important');
            el.style.setProperty('background-position', 'center', 'important');
          });

          ctx.querySelectorAll('div').forEach(function (el) {
            if (!el.firstElementChild && el.textContent.trim() === 'Shop with AI') {
              el.textContent = TARGET;
              attachLabelObserver(el);
            }
          });
        }

        function patchAll() {
          if (labelEl && !document.contains(labelEl)) {
            if (labelObs) { labelObs.disconnect(); labelObs = null; }
            labelEl = null;
          }
          patchRoot(document);
          document.querySelectorAll('*').forEach(function (el) {
            if (el.shadowRoot) patchRoot(el.shadowRoot);
          });
          applyHoverState();
        }

        patchAll();

        new MutationObserver(patchAll).observe(document.body, {
          childList: true, subtree: true, characterData: true
        });

        // ── Hover event delegation (desktop only) ────────────────────────
        if (isMouse) {
          document.addEventListener('mouseover', function (e) {
            if (!e.target.closest || !e.target.closest('[data-testid="entrypoint"]')) return;
            patchAll();
            setTimeout(patchAll, 30);
            setTimeout(patchAll, 80);
            setTimeout(patchAll, 200);
            if (isHovered) return;
            isHovered = true;
            var tw = getTextWrapper();
            expandText(tw);
            setTimeout(function () { expandText(getTextWrapper()); }, 40);
            setTimeout(function () { expandText(getTextWrapper()); }, 120);
          }, true);

          document.addEventListener('mouseout', function (e) {
            if (!e.target.closest || !e.target.closest('[data-testid="entrypoint"]')) return;
            if (e.relatedTarget && e.relatedTarget.closest &&
                e.relatedTarget.closest('[data-testid="entrypoint"]')) return;
            isHovered = false;
            collapseText(getTextWrapper(), false);
          }, true);
        }

        // Fast polling 5s → slow 25s
        var fast = setInterval(patchAll, 120);
        setTimeout(function () {
          clearInterval(fast);
          var slow = setInterval(patchAll, 1000);
          setTimeout(function () { clearInterval(slow); }, 25000);
        }, 5000);
      })();
    </script>

    <!-- ── Mobile Concierge Pill ── -->
    <div id="dl-concierge-tab" role="button" aria-label="Your Concierge">
      <img class="dl-tab__logo" src="{{ 'Monogram_De_Leville.png' | asset_url }}" alt="" width="16" height="14">
      <span class="dl-tab__text">Your Concierge</span>
      <span class="dl-tab__close" aria-label="Dismiss">✕</span>
    </div>
    <script>
      (function () {
        if (!window.matchMedia('(pointer: coarse)').matches) return;
        if (sessionStorage.getItem('dl-tab-dismissed')) return;

        var tab = document.getElementById('dl-concierge-tab');
        if (!tab) return;
        tab.style.display = 'flex';

        setTimeout(function () { tab.classList.add('dl-tab--visible'); }, 3000);

        tab.querySelector('.dl-tab__close').addEventListener('click', function (e) {
          e.stopPropagation();
          tab.classList.remove('dl-tab--visible');
          setTimeout(function () { tab.style.display = 'none'; }, 400);
          sessionStorage.setItem('dl-tab-dismissed', '1');
        });

        tab.addEventListener('click', function (e) {
          if (e.target.classList.contains('dl-tab__close')) return;
          var badge = document.querySelector('[data-testid="entrypoint"]');
          if (badge) badge.dispatchEvent(new MouseEvent('click', { bubbles: true, cancelable: true }));
        });
      })();
    </script>
</body>
```
