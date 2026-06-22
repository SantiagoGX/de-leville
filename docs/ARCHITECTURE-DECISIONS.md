# Architecture Decisions — De Leville Theme

Decisions made during development that are not obvious from the code alone.

---

## Collection Pages — Shop / Story Split (2026-05-04)

### Decision
Each collection operates on two separate URLs:

**1. Shop page — `/collections/{handle}`**
- Template: `collection.{handle}.json` → section `snc-plp`
- 4-column full-bleed grid, Home Showcase style cards
- Optional hero (video/image, configurable per section)
- Subtle CTA "Discover the story →" below the collection title
- Zero editorial content — 100% focus on conversion

**2. Editorial page — `/pages/{handle}`**
- Template: `page.{handle}.json` → editorial sections + `snc-home-showcase` grid at the bottom
- Brand content: history, craftsmanship, campaign video
- Product grid at the bottom to capture conversion from discovery users
- Accessed from the shop page CTA

### Why
Same pattern as LV, Cartier, Bvlgari: the collection URL is for buying (clean grid, no narrative friction). The story lives in a separate editorial that deepens desire. The buyer with intent lands on the grid; the curious buyer lands on the story and then buys.

### Status (as of 2026-05-04)
- [ ] Update `collection.classico.json`, `collection.incontro.json`, `collection.mezza-luna.json`, `collection.chains.json`, `collection.bags.json` → use `snc-plp` as main section
- [ ] Add `story_cta_text` + `story_cta_url` setting to `snc-plp` schema
- [ ] Create `page.classico.json`, `page.incontro.json`, `page.mezza-luna.json` with editorial sections + product grid
- [ ] Update editorial section grids from 3 → 4 columns

---

## snc-incontro-story — Section Architecture (2026-05-07/08)

### Description
`sections/snc-incontro-story.liquid` (1100+ lines). Replaces `snc-horizontal-slides` (buttons mode) + `snc-scroll-video` in `templates/page.incontro.json` (both marked `disabled: true`). CSS/JS prefix: `sis`.

### Desktop (≥990px) — 700vh total
- **Phase 1 (500vh):** 3-column sticky layout — text left, video center, buttons right. Scroll-driven: PURO→SEGRETO→ESSENZA→REGALE in order. On REGALE: text/buttons fade+blur out, video decelerates to pause (playbackRate 1→0), video centers in viewport via `getBoundingClientRect()` + `position:absolute` lerp, vignette `mask-image: radial-gradient` oval hides container edges.
- **Phase 2 (200vh):** `d1El` background→transparent, `splitVid` rises to z-index 6. Frozen ring centered: rise+fade+blur while split video (rings separating) fades in from z-index 6. Blur dissolve peaks at midpoint. "THE STORY" text appears with blur-dissolve at 45% of Phase 2.

### Mobile (<990px) — 560vh total
- **Phase 1 (400vh):** 4 fullscreen sticky videos. `object-fit: contain`, 98% width, max-height 72vh. `preload="none"` in HTML; JS loads only active + next (progressive). Videos seeked to 30% duration on load (mid-spin). playbackRate NOT reset to 0 on deactivate. Tab strip: filled black pills for active, white background with `border-top: 1px`.
- **Phase 2 (160vh):** REGALE fades out → split video crossfade easeInOut 28%. Blur on internal `<video>` (not wrapper) to avoid rectangular border artifact. Word-by-word stagger reveal of text.

### Critical bugs resolved
- `::after { z-index: 20 }` covered tabs (z-index 10) with white background → text invisible → fixed to z-index 3
- `.sis__vid { opacity: 0 }` made mobile variant videos invisible → `.sis__mob-wrap .sis__vid { opacity: 1 }` in mobile media query
- `easeOut3`/`easeInOut2` declared inside `if` block → moved to IIFE scope
- `preload="auto"` on 4 simultaneous videos → progressive loading: only active + next

### Template
`templates/page.incontro.json`: `snc_incontro_story` as first active section. Blocks in order PURO→SEGRETO→ESSENZA→REGALE. `split_video_url` points to CDN URL of the rings-separating video.

---

## PDP Sticky Bar — Mobile Only (2026-05-08)

**Decision:** `.pdp-sticky-bar` is the **only** Add to Bag CTA on mobile. The inline buttons container (`.pdp-hero--buttons-container`) is hidden on mobile (`display: none !important`) to avoid duplication. The inline buttons remain in the DOM and receive programmatic `.click()` from the sticky bar.

**Do not disable or remove `.pdp-sticky-bar` on mobile.**
