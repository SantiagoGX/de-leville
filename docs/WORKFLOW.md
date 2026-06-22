# Workflow — De Leville Theme

---

## ⚠️ Collaboration Protocol — Read First

**This theme is connected to GitHub. Every `git push` to `main` deploys automatically to the live store.**

The source of truth is GitHub. Not Shopify Admin.

> If you edit the theme directly in the Shopify theme editor, and someone pushes code to GitHub, **your editor changes will be overwritten.**

### Before you start working, answer this:

**"Who else might be touching the theme right now?"**

- If the developer is about to push code → the store owner should not be in the theme editor.
- If the store owner is making content changes → the developer should not push code until those changes are saved and pulled.
- If unsure → ask before doing anything.

### Safe zones (no coordination needed):
- Shopify Admin: editing **products, pages, blog posts, metafields** — these are not part of the theme files.
- Shopify Admin: editing **orders, customers, discounts** — unrelated to theme.

### Coordination required:
- Shopify Admin: **Theme editor** (customizer, section settings, colors, fonts) — these write to `config/settings_data.json` and `templates/*.json`, which are in git.
- Any **code changes** via Claude Code or CLI.

---

## The Normal Development Workflow

### Step 1 — Test locally with Shopify CLI

Before touching the live store, run the local dev server:

```bash
shopify theme dev
```

This uploads a temporary **development theme** to Shopify and gives you a live preview URL. Changes you make locally appear instantly. The live store is not affected.

Use this to:
- Test new features before they go live
- Catch visual issues across pages
- Verify mobile behavior

### Step 2 — Approve the change

Review the local preview. If it looks right, proceed. If not, iterate locally until it's ready.

### Step 3 — Push to GitHub → deploys to live

```bash
git add .
git commit -m "brief description of what changed and why"
git push
```

GitHub Actions picks this up and runs `shopify theme push` automatically. The live store is updated in ~40 seconds.

### Step 4 — Verify on live

Check `deleville.co` to confirm the change is live and correct.

---

## Monitoring Deploys

**See all deploys:** `github.com/delevillegit/De-Leville-Shopify-theme/actions`

Each row is one deploy. Green ✅ = deployed. Red ❌ = failed (check the logs).

---

## If Something Goes Wrong

### Rollback a bad deploy

Find the last good commit hash:
```bash
git log --oneline
```

Revert to it:
```bash
git revert HEAD
git push
```

This creates a new commit that undoes the last one, then deploys the reverted version.

### If the store owner made changes in the theme editor and they got overwritten

```bash
shopify theme pull
git add .
git commit -m "restore theme editor changes"
git push
```

Pull the current Shopify state back into git first, then push — this makes Shopify the source of truth for that moment.

---

## GitHub Actions Setup Reference

The deploy workflow lives at `.github/workflows/deploy.yml`.

Required GitHub secrets (set in repo Settings → Secrets and variables → Actions):

| Secret | Value |
|--------|-------|
| `SHOPIFY_STORE` | `34jgvc-dr.myshopify.com` |
| `SHOPIFY_CLI_THEME_TOKEN` | Password from the Theme Access app in Shopify Admin |

The Theme Access app: `admin.shopify.com/store/34jgvc-dr/apps` → search "Theme Access".

---

## Commit Message Convention

```
feat: add tooltip to variant selector
fix: correct sticky bar z-index on mobile
chore: update gitignore
revert: undo tooltip change
```

Short, present tense, lowercase. The commit message becomes the deploy label in GitHub Actions — make it readable.
