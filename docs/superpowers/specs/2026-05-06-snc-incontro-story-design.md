# snc-incontro-story — Design Spec
**Date:** 2026-05-06  
**Status:** Approved for implementation

---

## Goal

Replace the two existing sections on `/pages/incontro` — `snc-horizontal-slides` (4 ring variant videos) and `snc-scroll-video` (ESSENZA split animation) — with a single unified section `snc-incontro-story` that:

- **Desktop (≥990px):** Visually equivalent to the current two sections combined — same 3-column layout for variants, same scroll-scrubbed split animation. Small differences in inter-section padding are acceptable; the user experience must feel the same.
- **Mobile (<990px):** One continuous sticky scroll experience — 4 variants flow directly into the ESSENZA split animation with no section boundary.

The existing sections are **not deleted** — they are disabled via `"disabled": true` in `page.incontro.json` and remain as fallback.

---

## Files Created / Modified

| Action | File |
|---|---|
| Create | `sections/snc-incontro-story.liquid` |
| Modify | `templates/page.incontro.json` — add new section, disable old two |

---

## Architecture

### Section height

```
Desktop:  buttons_scroll_height (default 500vh) + split_scroll_height (default 200vh)
Mobile:   560vh total  (400vh phase-1 + 160vh phase-2)
```

### Single sticky container

The section has **one sticky container** (`position: sticky; top: 0; height: 100svh`). On desktop, it behaves as the two current sections do separately (JS driven). On mobile, it handles both phases without releasing the sticky.

---

## Desktop Layout (≥990px) — Identical to current

### Phase 1: Variant selector (same as `snc-horizontal-slides` buttons mode)
- 3-column layout: text left | video center | variant buttons right
- Scroll-driven button activation via `buttons_scroll_height`
- Videos crossfade on variant change (0.25s)
- All 4 video elements rendered (opacity: 0/1, never display: none) so they stay buffered

### Phase 2: Split animation (same as `snc-scroll-video`)
- After Phase 1 scroll completes, sticky transitions to the split video
- `video.currentTime` scrubbed by scroll progress
- Text overlay fades in at `split_reveal_threshold` (default 45%)
- Desktop text: blur-dissolve reveal (blur 10px → 0 as rings separate)

### Desktop JS transition between phases
The section total height = `buttons_scroll_height` + `split_scroll_height`. One sticky container handles both.

Boundary = `buttons_scroll_height / total_height` (e.g. 500 / 700 = ~0.71).

At the boundary:
1. 3-column layout fades out (opacity 1→0 over last 8% of Phase 1)
2. Split video becomes visible, starts at `currentTime = 0`
3. Phase 2 scroll driver maps remaining scroll progress → `video.currentTime`

Text reveal uses the same word-by-word stagger + blur-dissolve on desktop.

---

## Mobile Layout (<990px) — New unified experience

### Structure inside sticky container

```
┌─────────────────────────────┐  100svh
│                             │
│   VIDEO AREA  (flex: 1)     │  fills all remaining space
│   object-fit: cover         │
│   edge-to-edge              │
│                             │
│   [ESSENZA label overlay]   │  shown only when ESSENZA active in phase 1
│                             │
├─────────────────────────────┤
│  REGALE │ SEGRETO│PURO│ESENZ│  ~44px — tabs (hidden in phase 2)
└─────────────────────────────┘
```

Phase 2 (split animation):
```
┌─────────────────────────────┐  100svh
│  ◯  ring half (moving up)   │
│                             │
│  Incontro                   │
│  THE STORY                  │  text emerges word-by-word
│  A symbol of...             │
│                             │
│  ◯  ring half (moving down) │
└─────────────────────────────┘
```

### Mobile scroll driver

Total section height: **560vh** (400vh phase-1 + 160vh phase-2)

```
progress = scrolled / (sectionHeight - 100svh)

Phase 1: progress 0 → 0.714   (400 / 560)
Bridge:  progress 0.64 → 0.714  (ESSENZA activates, tabs fade out)
Phase 2: progress 0.714 → 1.0  (split video scrubs, text reveals)
```

### Phase 1 — Variant switching (mobile)

- Scroll progress mapped to 4 variants (each gets equal share of 400vh)
- Variant order: **REGALE → SEGRETO → PURO → ESSENZA** (ESSENZA last)
- Tab strip: top-border indicator, active tab gets `border-top: 2px solid #1a1a1a`
- All videos rendered (opacity toggle), not display:none
- No text description visible on mobile (client preference: video + tabs only)

### Bridge zone (progress 0.64 → 0.714)

At 90% through Phase 1:
- ESSENZA video is already active (last variant)
- Tab strip fades out (`opacity: 0`, pointer-events: none)
- ESSENZA variant video is at full opacity, no crossfade needed

At progress = 0.714 (Phase 1 complete):
- ESSENZA rotation video fades out (0→0 opacity)
- Split video fades in (0→1 opacity) over ~30px of scroll
- Crossfade duration: mapped to progress 0.714 → 0.75 (fast, ~0.5s of scroll)

### Phase 2 — Split animation (mobile)

Progress within phase 2: `p2 = (progress - 0.714) / 0.286`

- `video.currentTime = p2 * video.duration`
- Text reveal threshold: at `p2 = 0.25` (text starts appearing)
- Text layout: **centered** in viewport (same as corrected snc-scroll-video)
- Title: `clamp(44px, 13vw, 60px)` — large, fills screen
- Word-by-word stagger animation with blur (3px → 0)
- Description: full width, `max-width: 90vw`

---

## Video Elements

The sticky container renders **5 video elements** on mobile:
1. `video.tp-variant-0` — REGALE (opacity 0 or 1)
2. `video.tp-variant-1` — SEGRETO (opacity 0 or 1)
3. `video.tp-variant-2` — PURO (opacity 0 or 1)
4. `video.tp-variant-3` — ESSENZA rotation (opacity 0 or 1)
5. `video.snc-split` — ESSENZA split animation (opacity 0 or 1, currentTime scrubbed)

All 5 are `display: block`, `position: absolute`, `inset: 0`, `object-fit: cover`. Only one is visible at a time via opacity. All are muted, playsinline, preload="auto".

---

## Schema

Inherits all settings from both existing sections, plus:

**New settings:**
| id | type | label | default |
|---|---|---|---|
| `split_video` | video | Split animation video | — |
| `split_video_url` | text | Split video CDN URL | — |
| `split_eyebrow` | text | Eyebrow text | "Incontro" |
| `split_title` | text | Title | "THE STORY" |
| `split_description` | richtext | Description | — |
| `split_scroll_height` | range | Split scroll height (vh) | 200 |
| `split_reveal_threshold` | range | Text appears at scroll % | 45 |
| `split_background_color` | color | Split background | #ffffff |
| `split_text_color` | color | Split text color | #1a1a1a |

**Blocks (unchanged):** `categoria` type with `cat_btn_label`, `cat_title`, `cat_desc`, `cat_video`, `cat_media_type`

---

## page.incontro.json changes

```json
// Add above existing sections:
"snc_incontro_story": {
  "type": "snc-incontro-story",
  "name": "Incontro — Story (unified)",
  ...settings + blocks copied from the two disabled sections...
}

// Disable existing sections:
"snc_horizontal_slides_egjm8E": { "disabled": true, ... }
"snc_scroll_video_story":        { "disabled": true, ... }
```

Block order: `["categoria_gxU4pP"(REGALE), "categoria_nhmVQn"(SEGRETO), "categoria_efmkme"(PURO), "categoria_AeY6WA"(ESSENZA)]`

---

## What does NOT change

- Desktop experience of both sections: pixel-identical to current
- All existing schema settings from both sections (preserved in new section)
- The descriptions written for each variant in the previous session
- The exit/entry fade transitions between sections (already implemented in the two sections; will be replicated in the new section)
- All other sections on the page (Fixed Frame, Mixed Grid, Scroll Gallery)
