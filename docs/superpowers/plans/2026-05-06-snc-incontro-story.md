# snc-incontro-story Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create `sections/snc-incontro-story.liquid` — a single Shopify section that replicates the desktop experience of `snc-horizontal-slides` (buttons mode) + `snc-scroll-video` exactly, and delivers a unified mobile experience where 4 ring variant videos flow into the ESSENZA split animation in one continuous sticky scroll.

**Architecture:** One sticky container (`100svh`) handles both phases. Desktop uses the existing 3-column layout for Phase 1 and full-screen split video for Phase 2, driven by a single scroll listener. Mobile uses fullscreen videos with a tab selector for Phase 1 (0–400vh), then crossfades to the split video + text for Phase 2 (400–560vh). All video elements are `display:block` with opacity toggling — never `display:none` — to keep them buffered.

**Tech Stack:** Shopify Liquid, vanilla CSS, vanilla JS (no dependencies). Class prefix: `sis` (snc-incontro-story).

---

## Files

| Action | Path |
|---|---|
| **Create** | `sections/snc-incontro-story.liquid` |
| **Modify** | `templates/page.incontro.json` |

---

## Reference Data

**Split video URL:** `https://cdn.shopify.com/videos/c/o/v/fc8138e07af642b48cd254e6dd2965c2.mp4`

**Block IDs and new order (ESSENZA last):**
| Position | Label | Block ID | Video file |
|---|---|---|---|
| 1 | REGALE | `categoria_gxU4pP` | `shopify://files/videos/ANILLO 2 VIDEO 3.mp4` |
| 2 | SEGRETO | `categoria_nhmVQn` | `shopify://files/videos/ANILLO 1 VIDEO 5.mp4` |
| 3 | PURO | `categoria_efmkme` | `shopify://files/videos/ANILLO 3 VIDEO 3.mp4` |
| 4 | ESSENZA | `categoria_AeY6WA` | `shopify://files/videos/ANILLO 4 VIDEO 1.mp4` |

**Desktop settings to replicate (from existing `snc_horizontal_slides_egjm8E`):**
```
background_color: #ffffff
btn_layout_padding: 60px, btn_layout_gap: 40px
btn_media_height: 500px, btn_media_object_fit: cover
btn_title_size: 48px, btn_title_weight: 400, btn_title_color: #1a1a1a
btn_desc_size: 16px, btn_desc_weight: 300, btn_desc_color: #4a4a4a
btn_col_width: 220px, btn_gap: 12px, btn_padding_v: 18px, btn_padding_h: 24px
btn_font_size: 14px, btn_font_weight: 400, btn_letter_spacing: 0.1em
btn_border_color: #1a1a1a, btn_text_color: #1a1a1a
btn_active_bg: #1a1a1a, btn_active_color: #ffffff
buttons_scroll_height: 500vh
```

**Split settings (from `snc_scroll_video_story`):**
```
split_eyebrow: "Incontro"
split_title: "THE STORY"
split_description: "<p>A symbol of connection, devotion, and love.<br/><br/>Incontro is the tale of two halves finding each other and connecting in their own unique way, creating one perfect and harmonious union.</p>"
split_scroll_height: 200vh
split_reveal_threshold: 45
split_text_color: #1a1a1a
split_background_color: #ffffff
```

---

## Task 1 — Section scaffold + schema

**Files:**
- Create: `sections/snc-incontro-story.liquid`

- [ ] **Step 1.1: Create the file with the schema block**

```liquid
{% comment %}
  snc-incontro-story
  Unified section: snc-horizontal-slides (buttons mode) + snc-scroll-video.
  Desktop ≥990px: 3-col variant selector (Phase 1) → full-screen split animation (Phase 2).
  Mobile <990px:  single sticky 560vh scroll — 4 variants → ESSENZA split → text.
{% endcomment %}

{% schema %}
{
  "name": "SNC Incontro Story",
  "tag": "section",
  "class": "section--snc-incontro-story",
  "settings": [
    { "type": "header", "content": "General" },
    { "type": "color", "id": "background_color", "label": "Background color", "default": "#ffffff" },

    { "type": "header", "content": "Phase 1 — Variant selector (desktop layout)" },
    { "type": "range", "id": "buttons_scroll_height", "label": "Phase 1 scroll height (vh)", "min": 300, "max": 700, "step": 50, "unit": "vh", "default": 500 },
    { "type": "range", "id": "btn_layout_padding", "min": 0, "max": 120, "step": 10, "unit": "px", "label": "Side padding", "default": 60 },
    { "type": "range", "id": "btn_layout_gap", "min": 0, "max": 200, "step": 10, "unit": "px", "label": "Column gap", "default": 40 },
    { "type": "range", "id": "btn_title_size", "min": 16, "max": 120, "step": 2, "unit": "px", "label": "Title size", "default": 48 },
    { "type": "select", "id": "btn_title_weight", "label": "Title weight", "options": [{ "value": "300", "label": "Light" }, { "value": "400", "label": "Regular" }, { "value": "500", "label": "Medium" }], "default": "400" },
    { "type": "color", "id": "btn_title_color", "label": "Title color", "default": "#1a1a1a" },
    { "type": "checkbox", "id": "btn_title_uppercase", "label": "Title uppercase", "default": false },
    { "type": "range", "id": "btn_desc_size", "min": 10, "max": 32, "step": 1, "unit": "px", "label": "Description size", "default": 16 },
    { "type": "select", "id": "btn_desc_weight", "label": "Description weight", "options": [{ "value": "300", "label": "Light" }, { "value": "400", "label": "Regular" }], "default": "300" },
    { "type": "color", "id": "btn_desc_color", "label": "Description color", "default": "#4a4a4a" },
    { "type": "range", "id": "btn_desc_margin_top", "min": 0, "max": 80, "step": 4, "unit": "px", "label": "Title→desc gap", "default": 20 },
    { "type": "range", "id": "btn_media_height", "min": 200, "max": 800, "step": 50, "unit": "px", "label": "Video height (desktop)", "default": 500 },
    { "type": "select", "id": "btn_media_object_fit", "label": "Video fit", "options": [{ "value": "cover", "label": "Cover" }, { "value": "contain", "label": "Contain" }], "default": "cover" },
    { "type": "range", "id": "btn_col_width", "min": 120, "max": 400, "step": 10, "unit": "px", "label": "Button column width", "default": 220 },
    { "type": "range", "id": "btn_gap", "min": 0, "max": 40, "step": 4, "unit": "px", "label": "Button spacing", "default": 12 },
    { "type": "range", "id": "btn_padding_v", "min": 8, "max": 40, "step": 2, "unit": "px", "label": "Button vertical padding", "default": 18 },
    { "type": "range", "id": "btn_padding_h", "min": 8, "max": 60, "step": 4, "unit": "px", "label": "Button horizontal padding", "default": 24 },
    { "type": "range", "id": "btn_font_size", "min": 10, "max": 24, "step": 1, "unit": "px", "label": "Button text size", "default": 14 },
    { "type": "select", "id": "btn_font_weight", "label": "Button weight", "options": [{ "value": "300", "label": "Light" }, { "value": "400", "label": "Regular" }, { "value": "500", "label": "Medium" }], "default": "400" },
    { "type": "range", "id": "btn_letter_spacing", "min": 0, "max": 0.3, "step": 0.05, "unit": "em", "label": "Button letter spacing", "default": 0.1 },
    { "type": "checkbox", "id": "btn_uppercase", "label": "Button text uppercase", "default": false },
    { "type": "color", "id": "btn_border_color", "label": "Button border", "default": "#1a1a1a" },
    { "type": "color", "id": "btn_text_color", "label": "Button text (inactive)", "default": "#1a1a1a" },
    { "type": "color", "id": "btn_active_bg", "label": "Active button background", "default": "#1a1a1a" },
    { "type": "color", "id": "btn_active_color", "label": "Active button text", "default": "#ffffff" },

    { "type": "header", "content": "Phase 2 — Split animation" },
    { "type": "video", "id": "split_video", "label": "Split video (Shopify picker)" },
    { "type": "text", "id": "split_video_url", "label": "Split video URL (CDN fallback)", "info": "Used when picker above is empty." },
    { "type": "text", "id": "split_eyebrow", "label": "Eyebrow", "default": "Incontro" },
    { "type": "text", "id": "split_title", "label": "Title", "default": "THE STORY" },
    { "type": "richtext", "id": "split_description", "label": "Description" },
    { "type": "range", "id": "split_scroll_height", "label": "Phase 2 scroll height (vh)", "min": 150, "max": 400, "step": 10, "unit": "vh", "default": 200 },
    { "type": "range", "id": "split_reveal_threshold", "label": "Text appears at scroll %", "min": 20, "max": 80, "step": 5, "default": 45 },
    { "type": "color", "id": "split_text_color", "label": "Text color", "default": "#1a1a1a" },
    { "type": "color", "id": "split_background_color", "label": "Split background color", "default": "#ffffff" },

    { "type": "header", "content": "Spacing" },
    { "type": "range", "id": "padding_top", "min": 0, "max": 200, "step": 5, "unit": "px", "label": "Top padding", "default": 0 },
    { "type": "range", "id": "padding_bottom", "min": 0, "max": 200, "step": 5, "unit": "px", "label": "Bottom padding", "default": 0 }
  ],
  "blocks": [
    {
      "type": "categoria",
      "name": "Ring Variant",
      "settings": [
        { "type": "text", "id": "cat_btn_label", "label": "Tab label", "default": "VARIANT" },
        { "type": "text", "id": "cat_title", "label": "Title (desktop left column)", "default": "THE DESIGN" },
        { "type": "richtext", "id": "cat_desc", "label": "Description (desktop left column)" },
        { "type": "radio", "id": "cat_media_type", "label": "Media type", "options": [{ "value": "image", "label": "Image" }, { "value": "video", "label": "Video" }], "default": "video" },
        { "type": "image_picker", "id": "cat_image", "label": "Image" },
        { "type": "video", "id": "cat_video", "label": "Video" },
        { "type": "range", "id": "cat_media_width", "min": 100, "max": 1000, "step": 50, "unit": "px", "label": "Resource width", "default": 600 },
        { "type": "range", "id": "cat_media_height", "min": 100, "max": 1000, "step": 50, "unit": "px", "label": "Resource height", "default": 500 }
      ]
    }
  ],
  "presets": [
    {
      "name": "SNC Incontro Story",
      "blocks": [
        { "type": "categoria" },
        { "type": "categoria" },
        { "type": "categoria" },
        { "type": "categoria" }
      ]
    }
  ]
}
{% endschema %}
```

- [ ] **Step 1.2: Verify the schema renders without errors**

Push to Shopify and confirm the section appears in the theme editor without Liquid errors:
```bash
shopify theme push --only sections/snc-incontro-story.liquid
```
Expected: No errors in the CLI output.

- [ ] **Step 1.3: Commit**
```bash
git add sections/snc-incontro-story.liquid
git commit -m "feat: add snc-incontro-story scaffold with schema"
```

---

## Task 2 — HTML structure

**Files:**
- Modify: `sections/snc-incontro-story.liquid` (add HTML before `{% schema %}`)

- [ ] **Step 2.1: Add Liquid variable assignments and HTML structure**

Insert this block at the very top of the file, before `{% comment %}`:

```liquid
{%- liquid
  assign p1_vh     = section.settings.buttons_scroll_height | default: 500
  assign p2_vh     = section.settings.split_scroll_height   | default: 200
  assign total_vh  = p1_vh | plus: p2_vh
  assign reveal_t  = section.settings.split_reveal_threshold | default: 45 | divided_by: 100.0
  assign bg        = section.settings.background_color | default: '#ffffff'
  assign split_bg  = section.settings.split_background_color | default: '#ffffff'
-%}

<style>
  /* placeholder — CSS added in Task 3 */
</style>

<div class="sis" id="sis-{{ section.id }}"
     style="--sis-p1:{{ p1_vh }}; --sis-p2:{{ p2_vh }}; --sis-total:{{ total_vh }}; padding-top:{{ section.settings.padding_top }}px; padding-bottom:{{ section.settings.padding_bottom }}px; background:{{ bg }};">

  <div class="sis__scroller">
    <div class="sis__sticky">

      <!-- ── SPLIT VIDEO (full-screen, both viewports, Phase 2) ── -->
      {%- if section.settings.split_video != blank -%}
        {{ section.settings.split_video | video_tag:
            autoplay: false, loop: false, muted: true,
            controls: false, playsinline: true, preload: 'auto',
            class: 'sis__vid sis__vid--split',
            id: 'sis-split-' | append: section.id }}
      {%- elsif section.settings.split_video_url != blank -%}
        <video class="sis__vid sis__vid--split"
               id="sis-split-{{ section.id }}"
               src="{{ section.settings.split_video_url }}"
               muted playsinline webkit-playsinline preload="auto"></video>
      {%- endif -%}

      <!-- ── MOBILE: variant videos (full-screen, display:none on desktop) ── -->
      {%- for block in section.blocks -%}
        {%- if block.settings.cat_media_type == 'video' and block.settings.cat_video != blank -%}
          <div class="sis__mob-wrap sis__mob-wrap--{{ forloop.index0 }}{% if forloop.first %} sis--active{% endif %}"
               data-idx="{{ forloop.index0 }}" id="sis-mob-wrap-{{ forloop.index0 }}-{{ section.id }}">
            {{ block.settings.cat_video | video_tag:
                autoplay: true, loop: true, muted: true,
                controls: false, playsinline: true, preload: 'auto',
                class: 'sis__vid',
                id: 'sis-mob-' | append: forloop.index0 | append: '-' | append: section.id }}
          </div>
        {%- endif -%}
      {%- endfor -%}

      <!-- ── DESKTOP: 3-col Phase 1 layout (display:none on mobile) ── -->
      <div class="sis__d1" id="sis-d1-{{ section.id }}">

        <div class="sis__d1-text" id="sis-d1-text-{{ section.id }}">
          <h2 class="sis__d1-title" id="sis-d1-title-{{ section.id }}">
            {%- if section.blocks.first -%}{{ section.blocks.first.settings.cat_title }}{%- endif -%}
          </h2>
          <div class="sis__d1-desc" id="sis-d1-desc-{{ section.id }}">
            {%- if section.blocks.first -%}{{ section.blocks.first.settings.cat_desc }}{%- endif -%}
          </div>
        </div>

        <div class="sis__d1-media" id="sis-d1-media-{{ section.id }}">
          {%- for block in section.blocks -%}
            {%- if block.settings.cat_media_type == 'video' and block.settings.cat_video != blank -%}
              <div class="sis__d1-mitem{% if forloop.first %} sis--active{% endif %}"
                   data-cat="{{ block.id }}" id="sis-d1-mitem-{{ block.id }}">
                {{ block.settings.cat_video | video_tag:
                    autoplay: true, loop: true, muted: true,
                    controls: false, playsinline: true, preload: 'auto',
                    class: 'sis__d1-vid' }}
              </div>
            {%- elsif block.settings.cat_image != blank -%}
              <div class="sis__d1-mitem{% if forloop.first %} sis--active{% endif %}"
                   data-cat="{{ block.id }}" id="sis-d1-mitem-{{ block.id }}">
                <img src="{{ block.settings.cat_image | image_url: width: 1200 }}"
                     class="sis__d1-img"
                     loading="{% if forloop.first %}eager{% else %}lazy{% endif %}"
                     alt="{{ block.settings.cat_title | escape }}">
              </div>
            {%- endif -%}
          {%- endfor -%}
        </div>

        <div class="sis__d1-btns" id="sis-d1-btns-{{ section.id }}">
          {%- for block in section.blocks -%}
            <button class="sis__d1-btn{% if forloop.first %} sis--active{% endif %}"
                    data-cat="{{ block.id }}"
                    data-title="{{ block.settings.cat_title | escape }}"
                    data-desc="{{ block.settings.cat_desc | escape }}"
                    {{ block.shopify_attributes }}>
              {{ block.settings.cat_btn_label | default: block.settings.cat_title }}
            </button>
          {%- endfor -%}
        </div>

        <div class="sis__dots" id="sis-dots-{{ section.id }}">
          {%- for block in section.blocks -%}
            <span class="sis__dot{% if forloop.first %} sis--active{% endif %}"
                  data-cat="{{ block.id }}"></span>
          {%- endfor -%}
        </div>

      </div><!-- /.sis__d1 -->

      <!-- ── MOBILE: tab strip (Phase 1) ── -->
      <div class="sis__tabs" id="sis-tabs-{{ section.id }}">
        {%- for block in section.blocks -%}
          <button class="sis__tab{% if forloop.first %} sis--active{% endif %}"
                  data-idx="{{ forloop.index0 }}" data-cat="{{ block.id }}">
            {{ block.settings.cat_btn_label }}
          </button>
        {%- endfor -%}
      </div>

      <!-- ── SPLIT TEXT (Phase 2, both viewports) ── -->
      <div class="sis__split-text" id="sis-split-text-{{ section.id }}"
           style="color:{{ section.settings.split_text_color | default: '#1a1a1a' }};">
        {%- if section.settings.split_eyebrow != blank -%}
          <p class="sis__eyebrow">{{ section.settings.split_eyebrow }}</p>
        {%- endif -%}
        {%- if section.settings.split_title != blank -%}
          <h2 class="sis__s-title">{{ section.settings.split_title }}</h2>
        {%- endif -%}
        {%- if section.settings.split_description != blank -%}
          <div class="sis__s-desc">{{ section.settings.split_description }}</div>
        {%- endif -%}
      </div>

      <!-- ── Progress bar (Phase 2) ── -->
      <div class="sis__bar" id="sis-bar-{{ section.id }}"></div>

    </div><!-- /.sis__sticky -->
  </div><!-- /.sis__scroller -->
</div><!-- /.sis -->
```

- [ ] **Step 2.2: Push and verify no Liquid syntax errors**
```bash
shopify theme push --only sections/snc-incontro-story.liquid
```
Expected: No Liquid errors. The section renders a blank white area (CSS/JS not added yet).

- [ ] **Step 2.3: Commit**
```bash
git add sections/snc-incontro-story.liquid
git commit -m "feat: snc-incontro-story HTML structure"
```

---

## Task 3 — CSS

**Files:**
- Modify: `sections/snc-incontro-story.liquid` (replace `/* placeholder — CSS added in Task 3 */`)

- [ ] **Step 3.1: Replace the CSS placeholder with the full stylesheet**

```css
/* ──────────────────────────────────────────────────────────────
   SNC INCONTRO STORY — complete stylesheet
   sis = snc-incontro-story class prefix
   ────────────────────────────────────────────────────────────── */

/* ── Section wrapper ── */
#shopify-section-{{ section.id }} { margin-top: 0 !important; }
.sis { position: relative; width: 100%; }

/* ── Scroller: total scroll height ── */
.sis__scroller {
  height: calc({{ p1_vh }}vh + {{ p2_vh }}vh);
}
@media (max-width: 989px) {
  .sis__scroller { height: 560vh; }
}

/* ── Sticky container ── */
.sis__sticky {
  position: sticky;
  top: 0;
  width: 100%;
  height: 100svh;
  height: 100vh; /* fallback */
  overflow: hidden;
  background: {{ bg }};
  will-change: opacity;
}

/* ── All video elements: absolute, full-screen by default ── */
.sis__vid {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: {{ section.settings.btn_media_object_fit | default: 'cover' }};
  display: block;
  opacity: 0;
  pointer-events: none;
  will-change: opacity;
}

/* Split video inherits full-screen positioning */
.sis__vid--split {
  z-index: 1;
  object-fit: cover;
}

/* ──────────────────────────────────────────────────────────────
   MOBILE: Variant videos (full-screen, hidden on desktop)
   ────────────────────────────────────────────────────────────── */
.sis__mob-wrap {
  position: absolute;
  inset: 0;
  z-index: 2;
  opacity: 0;
  transition: opacity 0.25s ease;
  pointer-events: none;
}
.sis__mob-wrap.sis--active { opacity: 1; pointer-events: auto; }

/* Tab strip: mobile only */
.sis__tabs {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  z-index: 10;
  display: flex;
  flex-direction: row;
  background: {{ bg }};
  transition: opacity 0.3s ease;
  will-change: opacity;
}
.sis__tab {
  flex: 1 1 0;
  padding: 11px 2px;
  font-family: var(--font-body-family, sans-serif);
  font-size: clamp(8px, 2.3vw, 10px);
  letter-spacing: 0.14em;
  text-transform: uppercase;
  border: none;
  border-top: 2px solid transparent;
  background: transparent;
  color: rgba(0,0,0,0.28);
  transition: color 0.3s ease, border-color 0.3s ease;
  text-align: center;
  cursor: pointer;
}
.sis__tab.sis--active { color: #1a1a1a; border-top-color: #1a1a1a; }

/* ──────────────────────────────────────────────────────────────
   DESKTOP: 3-col Phase 1 layout (hidden on mobile)
   ────────────────────────────────────────────────────────────── */
.sis__d1 {
  position: absolute;
  inset: 0;
  z-index: 5;
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  padding: 0 {{ section.settings.btn_layout_padding | default: 60 }}px;
  gap: {{ section.settings.btn_layout_gap | default: 40 }}px;
  background: {{ bg }};
  will-change: opacity;
}

.sis__d1-text {
  flex: 1 1 40%;
  max-width: none;
}
.sis__d1-title {
  font-family: var(--font-heading-family, serif);
  font-size: {{ section.settings.btn_title_size | default: 48 }}px;
  font-weight: {{ section.settings.btn_title_weight | default: 400 }};
  color: {{ section.settings.btn_title_color | default: '#1a1a1a' }};
  line-height: 1.1;
  margin: 0;
  {% if section.settings.btn_title_uppercase %}text-transform: uppercase;{% endif %}
  transition: opacity 0.3s ease, transform 0.35s ease;
}
.sis__d1-text.is-transitioning .sis__d1-title,
.sis__d1-text.is-transitioning .sis__d1-desc { opacity: 0; transform: translateY(6px); }

.sis__d1-desc {
  font-family: var(--font-body-family, sans-serif);
  font-size: {{ section.settings.btn_desc_size | default: 16 }}px;
  font-weight: {{ section.settings.btn_desc_weight | default: 300 }};
  color: {{ section.settings.btn_desc_color | default: '#4a4a4a' }};
  line-height: 1.6;
  margin-top: {{ section.settings.btn_desc_margin_top | default: 20 }}px;
  transition: opacity 0.3s ease, transform 0.35s ease;
}

.sis__d1-media {
  flex: 1 1 40%;
  max-width: none;
  height: {{ section.settings.btn_media_height | default: 500 }}px;
  position: relative;
  overflow: hidden;
}
.sis__d1-mitem {
  position: absolute;
  inset: 0;
  opacity: 0;
  pointer-events: none;
  transition: opacity 0.25s ease;
}
.sis__d1-mitem.sis--active { opacity: 1; pointer-events: auto; }

.sis__d1-vid, .sis__d1-img {
  width: 100%; height: 100%;
  object-fit: {{ section.settings.btn_media_object_fit | default: 'cover' }};
  display: block;
}

.sis__d1-btns {
  flex: 0 0 {{ section.settings.btn_col_width | default: 220 }}px;
  display: flex;
  flex-direction: column;
  gap: {{ section.settings.btn_gap | default: 12 }}px;
}
.sis__d1-btn {
  width: 100%;
  padding: {{ section.settings.btn_padding_v | default: 18 }}px {{ section.settings.btn_padding_h | default: 24 }}px;
  font-family: var(--font-body-family, sans-serif);
  font-size: {{ section.settings.btn_font_size | default: 14 }}px;
  font-weight: {{ section.settings.btn_font_weight | default: 400 }};
  letter-spacing: {{ section.settings.btn_letter_spacing | default: 0.1 }}em;
  border: 1px solid {{ section.settings.btn_border_color | default: '#1a1a1a' }};
  background: transparent;
  color: {{ section.settings.btn_text_color | default: '#1a1a1a' }};
  cursor: pointer;
  text-align: center;
  transition: background 0.15s, color 0.15s;
  {% if section.settings.btn_uppercase %}text-transform: uppercase;{% endif %}
}
.sis__d1-btn.sis--active {
  background: {{ section.settings.btn_active_bg | default: '#1a1a1a' }};
  color: {{ section.settings.btn_active_color | default: '#ffffff' }};
}

/* Dots (desktop only) */
.sis__dots {
  position: absolute;
  bottom: 28px; left: 50%;
  transform: translateX(-50%);
  display: flex; gap: 10px; z-index: 10; align-items: center;
}
.sis__dot {
  width: 5px; height: 5px; border-radius: 50%;
  background: rgba(0,0,0,0.18);
  transition: background 0.4s ease, transform 0.35s ease;
  display: block;
}
.sis__dot.sis--active { background: #1a1a1a; transform: scale(1.5); }

/* ──────────────────────────────────────────────────────────────
   PHASE 2: Split text overlay (both viewports)
   ────────────────────────────────────────────────────────────── */
.sis__split-text {
  position: absolute;
  inset: 0;
  z-index: 8;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
  padding: 40px 60px;
  pointer-events: none;
  opacity: 0;
  will-change: opacity, transform, filter;
}
.sis__eyebrow {
  font-family: 'Futura PT', var(--font-main, sans-serif);
  font-size: 11px;
  font-weight: 400;
  letter-spacing: 0.28em;
  text-transform: uppercase;
  opacity: 0.55;
  margin: 0 0 20px;
}
.sis__s-title {
  font-family: 'Futura PT', var(--font-main, sans-serif);
  font-size: clamp(32px, 4vw, 56px);
  font-weight: 400;
  line-height: 1.08;
  letter-spacing: 0.01em;
  margin: 0 0 28px;
  max-width: 700px;
}
.sis__s-desc {
  font-family: 'Futura PT', var(--font-main, sans-serif);
  font-size: 15px;
  font-weight: 300;
  line-height: 1.8;
  opacity: 0.78;
  max-width: 480px;
  margin: 0;
}
.sis__s-desc p { margin: 0 0 10px; }
.sis__s-desc p:last-child { margin: 0; }

/* Word animation spans */
.sis__word { display: inline-block; }
.sis__word-inner { display: inline-block; }

/* Progress bar */
.sis__bar {
  position: absolute;
  bottom: 0; left: 0;
  height: 1px; width: 0%;
  background: rgba(0,0,0,0.2);
  z-index: 15;
  transition: width 0.05s linear;
  will-change: width;
}

/* ──────────────────────────────────────────────────────────────
   MOBILE overrides (<989px)
   ────────────────────────────────────────────────────────────── */
@media (max-width: 989px) {
  /* Hide desktop 3-col layout */
  .sis__d1 { display: none !important; }
  .sis__dots { display: none !important; }

  /* Split text: large, centered — fills the space between the rings */
  .sis__split-text {
    padding: 36px 20px;
    align-items: center;
    justify-content: center;
    text-align: center;
  }
  .sis__s-title {
    font-size: clamp(44px, 13vw, 60px);
    letter-spacing: 0.02em;
    max-width: 100%;
    margin-bottom: 22px;
  }
  .sis__s-desc { max-width: 90vw; font-size: 15px; line-height: 1.75; }
  .sis__eyebrow { font-size: 10px; letter-spacing: 0.24em; margin-bottom: 18px; }

  /* Sub-elements start invisible for stagger animation */
  .sis__eyebrow, .sis__s-desc { opacity: 0; }

  /* Cover strip: hides any ring artifact at bottom edge */
  .sis__sticky::after {
    content: '';
    position: absolute;
    bottom: 0; left: 0; right: 0;
    height: max(6px, 4vh);
    background: {{ split_bg }};
    z-index: 20;
    pointer-events: none;
  }
}

/* ──────────────────────────────────────────────────────────────
   DESKTOP overrides (≥990px)
   ────────────────────────────────────────────────────────────── */
@media (min-width: 990px) {
  /* Hide mobile variant video wrappers and tabs */
  .sis__mob-wrap { display: none !important; }
  .sis__tabs { display: none !important; }
  /* Split video hidden until Phase 2 — JS sets opacity */
  .sis__vid--split { display: block; }
}
```

- [ ] **Step 3.2: Push and verify layout renders**
```bash
shopify theme push --only sections/snc-incontro-story.liquid
```
Expected: On desktop, section shows the 3-col white layout with placeholder text. On mobile, empty white sticky area.

- [ ] **Step 3.3: Commit**
```bash
git add sections/snc-incontro-story.liquid
git commit -m "feat: snc-incontro-story CSS"
```

---

## Task 4 — Desktop JavaScript

**Files:**
- Modify: `sections/snc-incontro-story.liquid` (add `<script>` block before `{% schema %}`)

- [ ] **Step 4.1: Add the desktop JS block**

```js
<script>
(function () {
  var sId     = '{{ section.id }}';
  var sEl     = document.getElementById('sis-' + sId);
  if (!sEl) return;

  /* ── Constants from Liquid ── */
  var P1_VH      = {{ p1_vh }};
  var P2_VH      = {{ p2_vh }};
  var TOTAL_VH   = {{ total_vh }};
  var REVEAL_AT  = {{ section.settings.split_reveal_threshold | default: 45 | divided_by: 100.0 }};
  var NUM_BLOCKS = {{ section.blocks.size }};
  var BOUNDARY   = P1_VH / TOTAL_VH; /* progress ratio where Phase 1 ends */

  var isMobile = window.matchMedia('(max-width: 989px)').matches;

  /* ────────────────────────────────────────────────────────────
     SHARED HELPERS
  ──────────────────────────────────────────────────────────── */
  function safePlay(v) {
    if (!v) return;
    v.muted = true;
    var p = v.play();
    if (p && p.catch) p.catch(function () {});
  }

  function wrapWords(el) {
    if (!el) return [];
    var words = el.textContent.trim().split(/\s+/);
    el.innerHTML = words.map(function (w) {
      return '<span class="sis__word"><span class="sis__word-inner">' + w + '</span></span>';
    }).join(' ');
    return Array.from(el.querySelectorAll('.sis__word-inner'));
  }

  /* ── Entry/exit fade on sticky container ── */
  var stickyEl = sEl.querySelector('.sis__sticky');
  if (stickyEl) {
    stickyEl.style.willChange = 'opacity';
    var efRaf = null;
    function updateFade() {
      efRaf = null;
      var rect = sEl.getBoundingClientRect();
      var scrolled = -rect.top;
      var totalRoom = sEl.offsetHeight - window.innerHeight;
      if (totalRoom <= 0) return;
      var p = Math.max(0, Math.min(1, scrolled / totalRoom));
      var entryT = p < 0.05 ? p / 0.05 : 1;
      var exitT  = p > 0.82 ? (p - 0.82) / 0.18 : 0;
      stickyEl.style.opacity = Math.max(0, entryT - exitT).toFixed(3);
    }
    window.addEventListener('scroll', function () {
      if (efRaf) return;
      efRaf = requestAnimationFrame(updateFade);
    }, { passive: true });
    updateFade();
  }

  if (isMobile) {
    /* Mobile init runs in Task 5 */
    initMobile();
    return;
  }

  /* ════════════════════════════════════════════════════════════
     DESKTOP
  ════════════════════════════════════════════════════════════ */

  /* ── Elements ── */
  var d1El      = document.getElementById('sis-d1-' + sId);
  var d1TextEl  = document.getElementById('sis-d1-text-' + sId);
  var titleEl   = document.getElementById('sis-d1-title-' + sId);
  var descEl    = document.getElementById('sis-d1-desc-' + sId);
  var dotsEl    = document.getElementById('sis-dots-' + sId);
  var splitVid  = document.getElementById('sis-split-' + sId);
  var splitText = document.getElementById('sis-split-text-' + sId);
  var barEl     = document.getElementById('sis-bar-' + sId);

  var d1Btns  = Array.from(sEl.querySelectorAll('.sis__d1-btn'));
  var d1Items = Array.from(sEl.querySelectorAll('.sis__d1-mitem'));
  var dots    = dotsEl ? Array.from(dotsEl.querySelectorAll('.sis__dot')) : [];

  /* ── Ensure all desktop videos have correct attributes ── */
  sEl.querySelectorAll('.sis__d1-vid').forEach(function (v) {
    v.muted = true;
    v.setAttribute('playsinline', '');
    v.setAttribute('preload', 'auto');
  });
  if (splitVid) {
    splitVid.muted = true;
    splitVid.setAttribute('playsinline', '');
    splitVid.setAttribute('preload', 'auto');
    splitVid.load();
  }

  /* ── Play active desktop variant on load ── */
  function playActiveDesktop() {
    var activeItem = sEl.querySelector('.sis__d1-mitem.sis--active');
    if (activeItem) safePlay(activeItem.querySelector('video'));
  }
  playActiveDesktop();
  if (document.readyState !== 'complete') {
    window.addEventListener('load', playActiveDesktop, { once: true });
  }

  /* ── IntersectionObserver: pause when off-screen ── */
  var ioD = new IntersectionObserver(function (entries) {
    entries.forEach(function (entry) {
      if (entry.isIntersecting) {
        playActiveDesktop();
      } else {
        sEl.querySelectorAll('.sis__d1-vid').forEach(function (v) { v.pause(); });
      }
    });
  }, { threshold: 0.1 });
  ioD.observe(sEl);

  /* ── Sync dots to active cat ── */
  function syncDots(cat) {
    dots.forEach(function (d) {
      d.classList.toggle('sis--active', d.getAttribute('data-cat') === cat);
    });
  }

  /* ── Activate variant by cat ID ── */
  function activateCat(cat) {
    d1Btns.forEach(function (b) { b.classList.toggle('sis--active', b.getAttribute('data-cat') === cat); });
    d1Items.forEach(function (m) {
      var active = m.getAttribute('data-cat') === cat;
      m.classList.toggle('sis--active', active);
      var v = m.querySelector('video');
      if (active) { safePlay(v); }
      else if (v) { v.pause(); v.currentTime = 0; }
    });
    syncDots(cat);
  }

  /* ── Button click ── */
  d1Btns.forEach(function (btn) {
    btn.addEventListener('click', function () {
      var cat = btn.getAttribute('data-cat');
      activateCat(cat);
      if (d1TextEl) {
        d1TextEl.classList.add('is-transitioning');
        setTimeout(function () {
          if (titleEl) titleEl.textContent = btn.getAttribute('data-title') || '';
          if (descEl) descEl.innerHTML = btn.getAttribute('data-desc') || '';
          d1TextEl.classList.remove('is-transitioning');
        }, 180);
      }
    });
  });

  /* ── Scroll-driven phase management ── */
  var splitDuration = 0;
  var splitReady    = false;
  var lastP         = -1;
  var dRaf          = null;
  var lastDesktopIdx = 0;
  var phase2Active  = false;

  function seekSplit(t) {
    t = Math.max(0, Math.min(splitDuration, t));
    if (splitVid && !splitVid.seeking) splitVid.currentTime = t;
  }

  if (splitVid) {
    function onSplitReady() {
      if (!splitVid.duration || isNaN(splitVid.duration)) return;
      splitDuration = splitVid.duration;
      splitReady    = true;
    }
    if (splitVid.readyState >= 1 && splitVid.duration > 0) {
      onSplitReady();
    } else {
      ['loadedmetadata', 'canplay', 'durationchange'].forEach(function (e) {
        splitVid.addEventListener(e, onSplitReady, { once: true });
      });
      setTimeout(function () { if (!splitReady) onSplitReady(); }, 2000);
    }
  }

  function updateDesktop() {
    dRaf = null;
    var rect      = sEl.getBoundingClientRect();
    var scrolled  = -rect.top;
    var totalRoom = sEl.offsetHeight - window.innerHeight;
    if (totalRoom <= 0 || scrolled < 0) return;

    var p = Math.max(0, Math.min(1, scrolled / totalRoom));
    if (Math.abs(p - lastP) < 0.001) return;
    lastP = p;

    if (p <= BOUNDARY) {
      /* ── Phase 1: variant selector ── */
      var p1 = p / BOUNDARY; /* 0→1 within Phase 1 */

      /* Fade out Phase 1 layout at very end (last 8%) */
      var p1Exit = p1 > 0.92 ? (p1 - 0.92) / 0.08 : 0;
      if (d1El) d1El.style.opacity = (1 - p1Exit).toFixed(3);

      /* Ensure Phase 2 elements hidden */
      if (splitVid && phase2Active) {
        splitVid.style.opacity = '0';
        if (splitText) splitText.style.opacity = '0';
        phase2Active = false;
      }

      /* Scroll-driven variant switching with hysteresis */
      var stepSize   = 1 / NUM_BLOCKS;
      var hysteresis = 0.04;
      var idx = lastDesktopIdx;
      var next = Math.min(lastDesktopIdx + 1, NUM_BLOCKS - 1);
      var prev = Math.max(lastDesktopIdx - 1, 0);
      if (lastDesktopIdx < NUM_BLOCKS - 1 && p1 > (next * stepSize) + hysteresis) idx = next;
      else if (lastDesktopIdx > 0 && p1 < (lastDesktopIdx * stepSize) - hysteresis) idx = prev;

      if (idx !== lastDesktopIdx) {
        lastDesktopIdx = idx;
        var targetBtn = d1Btns[idx];
        if (targetBtn) {
          var cat = targetBtn.getAttribute('data-cat');
          activateCat(cat);
          if (d1TextEl) {
            d1TextEl.classList.add('is-transitioning');
            var capturedBtn = targetBtn;
            setTimeout(function () {
              if (titleEl) titleEl.textContent = capturedBtn.getAttribute('data-title') || '';
              if (descEl) descEl.innerHTML = capturedBtn.getAttribute('data-desc') || '';
              d1TextEl.classList.remove('is-transitioning');
            }, 180);
          }
        }
      }

    } else {
      /* ── Phase 2: split animation ── */
      var p2 = (p - BOUNDARY) / (1 - BOUNDARY); /* 0→1 within Phase 2 */

      /* Hide Phase 1 layout */
      if (d1El) d1El.style.opacity = '0';

      /* Show + scrub split video — ramp opacity over first 5% of P2 */
      if (splitVid) {
        var crossT = Math.min(1, p2 / 0.05);
        splitVid.style.opacity = crossT.toFixed(3);
        if (splitReady && splitDuration > 0) seekSplit(p2 * splitDuration);
        phase2Active = true;
      }

      /* Text reveal: blur-dissolve on desktop */
      var revealRange = 0.22;
      var t = Math.max(0, Math.min(1, (p2 - REVEAL_AT) / revealRange));
      if (splitText) {
        var blur = Math.max(0, 10 * (1 - t));
        splitText.style.opacity   = t.toFixed(3);
        splitText.style.transform = 'translateY(' + (18 * (1 - t)).toFixed(1) + 'px)';
        splitText.style.filter    = 'blur(' + blur.toFixed(1) + 'px)';
      }
      if (barEl) barEl.style.width = (p2 * 100) + '%';
    }
  }

  window.addEventListener('scroll', function () {
    if (dRaf) return;
    dRaf = requestAnimationFrame(updateDesktop);
  }, { passive: true });
  updateDesktop();

  /* Shopify editor reload */
  document.addEventListener('shopify:section:load', function (e) {
    if (e.detail.sectionId === sId) window.location.reload();
  });

  /* ── placeholder for initMobile (Task 5 replaces this) ── */
  function initMobile() {}
})();
</script>
```

- [ ] **Step 4.2: Push and test desktop**
```bash
shopify theme push --only sections/snc-incontro-story.liquid
```
Expected on desktop:
- Phase 1: 3-col layout visible. Scrolling 500vh activates variants in order.
- At 500vh: layout fades out.
- At 700vh: split video plays from first frame.
- Text fades in at 45% of Phase 2.

- [ ] **Step 4.3: Commit**
```bash
git add sections/snc-incontro-story.liquid
git commit -m "feat: snc-incontro-story desktop JS"
```

---

## Task 5 — Mobile JavaScript

**Files:**
- Modify: `sections/snc-incontro-story.liquid` (replace `function initMobile() {}` at bottom of script)

- [ ] **Step 5.1: Replace the `initMobile` stub with the full mobile driver**

```js
  function initMobile() {
    /* ── Mobile constants ── */
    var MOB_P1_VH    = 400;
    var MOB_P2_VH    = 160;
    var MOB_TOTAL_VH = 560;
    var MOB_BOUNDARY = MOB_P1_VH / MOB_TOTAL_VH; /* ≈ 0.714 */
    var MOB_BRIDGE   = (MOB_P1_VH - MOB_P1_VH * 0.1) / MOB_TOTAL_VH; /* bridge starts at 90% of P1 ≈ 0.643 */

    /* ── Mobile elements ── */
    var tabsEl    = document.getElementById('sis-tabs-' + sId);
    var tabs      = tabsEl ? Array.from(tabsEl.querySelectorAll('.sis__tab')) : [];
    var mobWraps  = Array.from(sEl.querySelectorAll('.sis__mob-wrap'));
    var splitVid  = document.getElementById('sis-split-' + sId);
    var splitText = document.getElementById('sis-split-text-' + sId);
    var barEl     = document.getElementById('sis-bar-' + sId);

    /* ── Preload + force-play all mobile variant videos ── */
    sEl.querySelectorAll('.sis__mob-wrap video').forEach(function (v) {
      v.muted = true;
      v.setAttribute('playsinline', '');
      v.setAttribute('preload', 'auto');
    });

    if (splitVid) {
      splitVid.muted = true;
      splitVid.setAttribute('playsinline', '');
      splitVid.setAttribute('preload', 'auto');
      splitVid.load();
    }

    /* Play first variant immediately */
    var firstWrap = mobWraps[0];
    if (firstWrap) safePlay(firstWrap.querySelector('video'));

    /* IntersectionObserver: play/pause when off-screen */
    var ioM = new IntersectionObserver(function (entries) {
      entries.forEach(function (entry) {
        if (entry.isIntersecting) {
          var activeWrap = sEl.querySelector('.sis__mob-wrap.sis--active');
          if (activeWrap) safePlay(activeWrap.querySelector('video'));
        } else {
          sEl.querySelectorAll('.sis__mob-wrap video').forEach(function (v) { v.pause(); });
        }
      });
    }, { threshold: 0.1 });
    ioM.observe(sEl);

    /* Allow manual tab taps to override scroll during Phase 1 */
    tabs.forEach(function (tab) {
      tab.addEventListener('click', function () {
        var idx = parseInt(tab.getAttribute('data-idx'), 10);
        activateMobVariant(idx);
      });
    });

    /* ── Split video readiness ── */
    var mobSplitDuration = 0;
    var mobSplitReady    = false;

    if (splitVid) {
      function onMobSplitReady() {
        if (!splitVid.duration || isNaN(splitVid.duration)) return;
        mobSplitDuration = splitVid.duration;
        mobSplitReady    = true;
      }
      if (splitVid.readyState >= 1 && splitVid.duration > 0) {
        onMobSplitReady();
      } else {
        ['loadedmetadata', 'canplay', 'durationchange'].forEach(function (e) {
          splitVid.addEventListener(e, onMobSplitReady, { once: true });
        });
        setTimeout(function () { if (!mobSplitReady) onMobSplitReady(); }, 2000);
      }
    }

    /* ── Word wrapping for mobile title ── */
    var wordEls = null;
    if (splitText) {
      var mobTitleEl = splitText.querySelector('.sis__s-title');
      if (mobTitleEl) {
        wordEls = wrapWords(mobTitleEl);
        wordEls.forEach(function (el) {
          el.style.opacity = '0';
          el.style.transform = 'translateY(14px)';
        });
      }
    }

    /* ── Activate mobile variant by index ── */
    var lastMobIdx = 0;
    function activateMobVariant(idx) {
      if (idx === lastMobIdx && mobWraps[idx] && mobWraps[idx].classList.contains('sis--active')) return;
      lastMobIdx = idx;
      mobWraps.forEach(function (w, i) {
        var active = i === idx;
        w.classList.toggle('sis--active', active);
        var v = w.querySelector('video');
        if (active) safePlay(v);
        else if (v) { v.pause(); v.currentTime = 0; }
      });
      tabs.forEach(function (t, i) { t.classList.toggle('sis--active', i === idx); });
    }

    /* ── Main mobile scroll driver ── */
    var mobLastP   = -1;
    var mobRaf     = null;
    var inPhase2   = false;

    function updateMobile() {
      mobRaf = null;
      var rect      = sEl.getBoundingClientRect();
      var scrolled  = -rect.top;
      var totalRoom = sEl.offsetHeight - window.innerHeight;
      if (totalRoom <= 0 || scrolled < 0) return;

      var p = Math.max(0, Math.min(1, scrolled / totalRoom));
      if (Math.abs(p - mobLastP) < 0.0005) return;
      mobLastP = p;

      if (p <= MOB_BOUNDARY) {
        /* ════ Phase 1: variant switching ════ */

        if (inPhase2) {
          /* Returning from Phase 2 — restore Phase 1 state */
          inPhase2 = false;
          if (splitVid) splitVid.style.opacity = '0';
          if (splitText) { splitText.style.opacity = '0'; splitText.style.filter = ''; }
          if (tabsEl) { tabsEl.style.opacity = '1'; tabsEl.style.pointerEvents = ''; }
        }

        /* Tab strip fade: transparent during bridge zone */
        if (tabsEl) {
          var bridgeT = p > MOB_BRIDGE
            ? Math.min(1, (p - MOB_BRIDGE) / (MOB_BOUNDARY - MOB_BRIDGE))
            : 0;
          tabsEl.style.opacity = (1 - bridgeT).toFixed(3);
          tabsEl.style.pointerEvents = bridgeT > 0.5 ? 'none' : '';
        }

        /* Map p to variant index */
        var p1 = p / MOB_BOUNDARY; /* 0→1 within Phase 1 */
        var idx = Math.min(mobWraps.length - 1, Math.floor(p1 * mobWraps.length));
        activateMobVariant(idx);

      } else {
        /* ════ Phase 2: split animation ════ */
        inPhase2 = true;

        /* Ensure last variant (ESSENZA) stays active as base */
        if (lastMobIdx !== mobWraps.length - 1) {
          activateMobVariant(mobWraps.length - 1);
        }

        /* Hide tabs */
        if (tabsEl) { tabsEl.style.opacity = '0'; tabsEl.style.pointerEvents = 'none'; }

        var p2 = (p - MOB_BOUNDARY) / (1 - MOB_BOUNDARY); /* 0→1 within Phase 2 */

        /* Crossfade: ESSENZA rotation → split video (over first 5% of P2) */
        var crossT = Math.min(1, p2 / 0.05);
        if (splitVid) splitVid.style.opacity = crossT.toFixed(3);
        /* Fade out the last variant wrap simultaneously */
        if (mobWraps[mobWraps.length - 1]) {
          mobWraps[mobWraps.length - 1].style.opacity = (1 - crossT).toFixed(3);
        }

        /* Scrub split video */
        if (splitVid && mobSplitReady && mobSplitDuration > 0) {
          var t = Math.max(0, Math.min(mobSplitDuration, p2 * mobSplitDuration));
          if (!splitVid.seeking) splitVid.currentTime = t;
        }

        /* Mobile reveal threshold: 25% into Phase 2 */
        var MOB_REVEAL_AT = 0.25;
        var revealRange   = 0.22;
        var revT = Math.max(0, Math.min(1, (p2 - MOB_REVEAL_AT) / revealRange));

        if (splitText) {
          splitText.style.opacity   = '1'; /* container always visible in P2 */
          splitText.style.transform = '';
          splitText.style.filter    = '';
        }

        /* Word-by-word stagger with blur */
        if (wordEls && wordEls.length > 0) {
          var n = wordEls.length;
          wordEls.forEach(function (el, i) {
            var wStart = (i / (n || 1)) * 0.6;
            var wT     = Math.max(0, Math.min(1, (revT - wStart) / 0.55));
            el.style.opacity   = wT.toFixed(3);
            el.style.transform = 'translateY(' + (8 * (1 - wT)).toFixed(1) + 'px)';
            el.style.filter    = 'blur(' + (3 * (1 - wT)).toFixed(1) + 'px)';
          });
        }

        /* Eyebrow */
        var eyebrowEl = splitText ? splitText.querySelector('.sis__eyebrow') : null;
        if (eyebrowEl) {
          var eyeT = Math.max(0, Math.min(1, revT / 0.35));
          eyebrowEl.style.opacity   = eyeT.toFixed(3);
          eyebrowEl.style.transform = 'translateY(' + (6 * (1 - eyeT)).toFixed(1) + 'px)';
        }

        /* Description */
        var descMobEl = splitText ? splitText.querySelector('.sis__s-desc') : null;
        if (descMobEl) {
          var dT = Math.max(0, Math.min(1, (revT - 0.4) / 0.6));
          descMobEl.style.opacity   = dT.toFixed(3);
          descMobEl.style.transform = 'translateY(' + (8 * (1 - dT)).toFixed(1) + 'px)';
        }

        if (barEl) barEl.style.width = (p2 * 100) + '%';
      }
    }

    window.addEventListener('scroll', function () {
      if (mobRaf) return;
      mobRaf = requestAnimationFrame(updateMobile);
    }, { passive: true });
    updateMobile();

    /* First touch/click to unblock autoplay */
    sEl.addEventListener('touchstart', function onTouch() {
      var activeWrap = sEl.querySelector('.sis__mob-wrap.sis--active');
      if (activeWrap) safePlay(activeWrap.querySelector('video'));
      sEl.removeEventListener('touchstart', onTouch);
    }, { once: true, passive: true });
  }
```

- [ ] **Step 5.2: Push and test mobile**
```bash
shopify theme push --only sections/snc-incontro-story.liquid
```
Expected on mobile (use DevTools device emulation, 390×844):
- Phase 1 (0–400vh): 4 variants cycle REGALE→SEGRETO→PURO→ESSENZA. Tabs update.
- At ~357vh (bridge zone): tabs start fading.
- At 400vh: ESSENZA rotation fades out, split video fades in.
- Phase 2 (400–560vh): split video scrubs. At ~440vh text starts word-by-word reveal.
- No double scroll. No white flash.

- [ ] **Step 5.3: Commit**
```bash
git add sections/snc-incontro-story.liquid
git commit -m "feat: snc-incontro-story mobile JS — unified scroll driver"
```

---

## Task 6 — Template update

**Files:**
- Modify: `templates/page.incontro.json`

- [ ] **Step 6.1: Add new section + reorder blocks + disable old sections**

In `templates/page.incontro.json`, make three changes:

**A) Add the new section before `snc_horizontal_slides_egjm8E`:**

```json
"snc_incontro_story": {
  "type": "snc-incontro-story",
  "name": "Incontro — Story (unified)",
  "blocks": {
    "categoria_gxU4pP": {
      "type": "categoria",
      "settings": {
        "cat_btn_label": "REGALE",
        "cat_title": "REGALE",
        "cat_desc": "<p>Maximum brilliance. Pavé diamonds cover every surface of the ring, turning the geometric Incontro form into a continuous field of light. The most opulent expression of the collection.</p>",
        "cat_media_type": "video",
        "cat_video": "shopify://files/videos/ANILLO 2 VIDEO 3.mp4",
        "cat_media_width": 500,
        "cat_media_height": 500
      }
    },
    "categoria_nhmVQn": {
      "type": "categoria",
      "settings": {
        "cat_btn_label": "SEGRETO",
        "cat_title": "SEGRETO",
        "cat_desc": "<p>A hidden contrast. White diamonds are set within the yellow gold structure, revealing a dual nature — warm and cool, bold and subtle — that shifts with every angle of light.</p>",
        "cat_media_type": "video",
        "cat_video": "shopify://files/videos/ANILLO 1 VIDEO 5.mp4",
        "cat_media_width": 500,
        "cat_media_height": 500
      }
    },
    "categoria_efmkme": {
      "type": "categoria",
      "settings": {
        "cat_btn_label": "PURO",
        "cat_title": "PURO",
        "cat_desc": "<p>Pure 18-karat gold. No diamonds. The geometric architecture of Incontro stands entirely on its own — honest, precise, and unapologetically minimal.</p>",
        "cat_media_type": "video",
        "cat_video": "shopify://files/videos/ANILLO 3 VIDEO 3.mp4",
        "cat_media_width": 500,
        "cat_media_height": 500
      }
    },
    "categoria_AeY6WA": {
      "type": "categoria",
      "settings": {
        "cat_btn_label": "ESSENZA",
        "cat_title": "ESSENZA",
        "cat_desc": "<p>The essential balance. Diamonds highlight the architectural lines of the ring without overwhelming the form — a refined equilibrium between brilliance and structure.</p>",
        "cat_media_type": "video",
        "cat_video": "shopify://files/videos/ANILLO 4 VIDEO 1.mp4",
        "cat_media_width": 500,
        "cat_media_height": 500
      }
    }
  },
  "block_order": [
    "categoria_gxU4pP",
    "categoria_nhmVQn",
    "categoria_efmkme",
    "categoria_AeY6WA"
  ],
  "settings": {
    "background_color": "#ffffff",
    "buttons_scroll_height": 500,
    "btn_layout_padding": 60,
    "btn_layout_gap": 40,
    "btn_title_size": 48,
    "btn_title_weight": "400",
    "btn_title_color": "#1a1a1a",
    "btn_title_uppercase": false,
    "btn_desc_size": 16,
    "btn_desc_weight": "300",
    "btn_desc_color": "#4a4a4a",
    "btn_desc_margin_top": 20,
    "btn_media_height": 500,
    "btn_media_object_fit": "cover",
    "btn_col_width": 220,
    "btn_gap": 12,
    "btn_padding_v": 18,
    "btn_padding_h": 24,
    "btn_font_size": 14,
    "btn_font_weight": "400",
    "btn_letter_spacing": 0.1,
    "btn_uppercase": false,
    "btn_border_color": "#1a1a1a",
    "btn_text_color": "#1a1a1a",
    "btn_active_bg": "#1a1a1a",
    "btn_active_color": "#ffffff",
    "split_video_url": "https://cdn.shopify.com/videos/c/o/v/fc8138e07af642b48cd254e6dd2965c2.mp4",
    "split_eyebrow": "Incontro",
    "split_title": "THE STORY",
    "split_description": "<p>A symbol of connection, devotion, and love.<br/><br/>Incontro is the tale of two halves finding each other and connecting in their own unique way, creating one perfect and harmonious union.</p>",
    "split_scroll_height": 200,
    "split_reveal_threshold": 45,
    "split_text_color": "#1a1a1a",
    "split_background_color": "#ffffff",
    "padding_top": 0,
    "padding_bottom": 0
  }
},
```

**B) Add `"disabled": true` to both old sections:**

```json
"snc_horizontal_slides_egjm8E": {
  "disabled": true,
  "type": "snc-horizontal-slides",
  ...
},
"snc_scroll_video_story": {
  "disabled": true,
  "type": "snc-scroll-video",
  ...
},
```

**C) Add `"snc_incontro_story"` first in the `"order"` array:**

```json
"order": [
  "main",
  "snc_incontro_story",
  "snc_animated_collections_home",
  ...rest unchanged...
]
```

Wait — looking at the actual order in `page.incontro.json`, the correct placement is first in order after `main`. Check the actual order array and prepend `"snc_incontro_story"` right after `"main"` but before `"snc_horizontal_slides_egjm8E"`.

- [ ] **Step 6.2: Push and verify**
```bash
shopify theme push --only templates/page.incontro.json sections/snc-incontro-story.liquid
```
Navigate to `http://127.0.0.1:9292/pages/incontro`. Expected:
- New unified section appears at the top of the page (before the two disabled sections).
- The two old sections are no longer visible.

- [ ] **Step 6.3: Commit**
```bash
git add templates/page.incontro.json
git commit -m "feat: add snc-incontro-story to page.incontro, disable old sections"
```

---

## Task 7 — Final verification

- [ ] **Step 7.1: Desktop smoke test**

Open `http://127.0.0.1:9292/pages/incontro` in a desktop browser (≥1200px wide).

Check:
1. Section loads with 3-column layout (text | video | buttons).
2. Scrolling 500vh activates variants in order: REGALE → SEGRETO → PURO → ESSENZA.
3. At 500vh: 3-col layout fades out.
4. At 500–700vh: split video plays, text fades in around 45% of Phase 2 progress.
5. Text uses blur-dissolve effect.
6. Entry and exit fades on sticky container work.
7. No JS errors in console.

- [ ] **Step 7.2: Mobile smoke test**

Open DevTools → Dimensions: 390×844 (iPhone 14), reload.

Check:
1. Video fills 100% of viewport. Tab strip visible at bottom.
2. Scrolling 400vh cycles through REGALE → SEGRETO → PURO → ESSENZA.
3. Tab highlights correct variant.
4. At ~360vh: tabs start fading.
5. At 400vh: ESSENZA rotation fades, split video crossfades in.
6. At 400–560vh: split video scrubs. Text reveals word-by-word with blur.
7. No second scroll container anywhere.
8. No white flash at the boundary.

- [ ] **Step 7.3: Narrow screen test**

Set DevTools to Galaxy Z Fold outer screen (344×968). Verify tabs are readable and video fills full width.

- [ ] **Step 7.4: Final commit**
```bash
git add -A
git commit -m "feat: snc-incontro-story — unified Incontro experience, desktop + mobile"
```
