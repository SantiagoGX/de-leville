# CLAUDE.md

Guidance for Claude Code when working in this repository.

## ⚠️ Read Before Working — Collaboration Warning

**This theme is synced to GitHub. Every `git push` to `main` auto-deploys to the live store in ~40 seconds.**

Before starting any work, identify who you are:

| Who | What to do |
|-----|-----------|
| **Developer (Santiago / Claude Code)** | Use the CLI + GitHub workflow in `docs/WORKFLOW.md` |
| **Store owner making content edits** | Edit directly in Shopify Admin (theme editor, products, pages) — but **notify the developer first**. A `git push` will overwrite any unsaved theme editor changes. |
| **Both working at the same time** | Stop. Coordinate. Decide who does what before touching anything. |

→ Full workflow details: [`docs/WORKFLOW.md`](docs/WORKFLOW.md)

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
