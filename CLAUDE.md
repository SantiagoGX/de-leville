# CLAUDE.md

Guidance for Claude Code when working in this repository.

---

## FIRST — Ask who is working before doing anything else

**At the start of every conversation, before any task, ask:**

> "Hi — before we start, are you Santiago (the developer) or Eliado (the store owner)?"

Then follow the instructions for that person below.

---

## If the person is Santiago

Santiago is the developer. He knows this project is on GitHub and how the workflow works. Give him a brief reminder:

> "This project is on GitHub (`github.com/delevillegit/De-Leville-Shopify-theme`). Changes pushed to `main` auto-deploy to the live store in ~40 seconds via GitHub Actions. Use `shopify theme dev` to test locally before pushing. Ready — what are we working on?"

Then proceed normally.

---

## If the person is Eliado

Eliado is the store owner. He may not be aware of the GitHub setup. Explain it clearly before anything else:

> "Before we start, there's something important you need to know about how this theme works.
>
> **This theme is connected to GitHub.** All the code lives at `github.com/delevillegit/De-Leville-Shopify-theme`. Every time the developer pushes a code change to GitHub, it automatically deploys to the live store (`deleville.co`) in about 40 seconds.
>
> **What this means for you:**
> - If you make changes in the Shopify theme editor (colors, section settings, layout), those changes live in Shopify. If the developer then pushes code from GitHub without pulling your changes first, your edits could be overwritten.
> - Before making any theme editor changes, let Santiago know so he can coordinate.
> - Editing products, pages, blog posts, orders, or customers in Shopify Admin is always safe — those are not affected by code pushes.
>
> **If you want to preview code changes before they go live**, the developer can use `shopify theme dev` — this opens a live preview URL of the theme where you can review everything before it goes to the real store. Nothing goes live until a `git push` is made.
>
> **The workflow in short:**
> 1. Developer makes changes locally and tests with `shopify theme dev`
> 2. You review the preview URL and approve
> 3. Developer runs `git push` — changes deploy to live in ~40 seconds
> 4. You verify on `deleville.co`
>
> Do you have any questions about this before we continue?"

---

## What This Is

A custom Shopify 2.0 theme for **De Leville**, a luxury jewelry brand. Built on the Shopify "Skeleton" starter theme (v0.1.0). No build system — pure Liquid, CSS, and vanilla JavaScript.

**GitHub repo:** `github.com/delevillegit/De-Leville-Shopify-theme`
**Store:** `34jgvc-dr.myshopify.com` / `deleville.co`

---

## Development Commands

```bash
shopify theme dev       # Local dev server with hot reload — test here before pushing
shopify theme push      # Manual push (normally handled by GitHub Actions on git push)
shopify theme pull      # Pull latest from Shopify to local
shopify theme check     # Lint Liquid files
```

```bash
git add .
git commit -m "description"
git push                # → triggers auto-deploy to live via GitHub Actions
```

---

## Architecture

- **`sections/`** — 49+ Liquid files, each self-contained with its own schema. Custom sections prefixed `snc-`.
- **`templates/`** — JSON files declaring which sections appear on each page. Avoid manual edits to `product.json` and `index.json`.
- **`snippets/`** — Reusable partials via `{% render %}`. Receive data via parameters.
- **`layout/theme.liquid`** — Root HTML shell. Includes critical CSS inline, loads `snc-sections.js`.
- **`assets/snc-sections.js`** — Single 131KB bundled JS file. Section registry pattern with lifecycle management.
- **`snippets/css-variables.liquid`** — Outputs `:root {}` CSS custom properties from theme settings. All dynamic theming flows through here.
- **`config/settings_schema.json`** / **`settings_data.json`** — Theme settings definition and current values.

---

## Key Conventions

- Custom files prefixed `snc-` to distinguish from base theme files.
- CSS custom properties (`--color-foreground`, `--font-body-family`) are the styling contract — change values in `css-variables.liquid`, not hardcoded in section files.
- Button shape: `border-radius: 999px` (pill) across all interactive elements — maintain this everywhere.
- Each `{% schema %}` block at the bottom of a section file is the source of truth for that section's settings.

---

## Reference Docs

| Doc | Contents |
|-----|---------|
| [`docs/WORKFLOW.md`](docs/WORKFLOW.md) | Full dev workflow, GitHub Actions, CLI testing, collaboration rules |
| [`docs/DESIGN-SYSTEM.md`](docs/DESIGN-SYSTEM.md) | Typography, colors, spacing, component patterns |
| [`docs/ROADMAP-10-DE-10.md`](docs/ROADMAP-10-DE-10.md) | Roadmap toward a 10/10 store — status of all audit items |
| [`docs/ACCIONABLES.md`](docs/ACCIONABLES.md) | Actionable items by tier with effort and type |
| [`docs/ARCHITECTURE-DECISIONS.md`](docs/ARCHITECTURE-DECISIONS.md) | Collection page pattern, snc-incontro-story architecture |
| [`docs/PLANS.md`](docs/PLANS.md) | Index of all development plans |
| [`CHANGELOG.md`](CHANGELOG.md) | Full history of changes by date and file |
