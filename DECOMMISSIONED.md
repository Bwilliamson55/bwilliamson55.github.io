# Decommissioned — legacy GitHub Pages blog

This repository hosted a **Jekyll + Chirpy** blog (GitHub Pages). It is **archived** and no longer published.

## Canonical blog

- **Live site:** [https://blog.weeumson.com](https://blog.weeumson.com)
- **Source app:** [quasar-triangle-ssr](https://github.com/Bwilliamson55/quasar-triangle-ssr) → `blog-web/` (Quasar PWA)

## Legacy URL redirects (configure at DNS / edge)

| Legacy path | Target |
|-------------|--------|
| `https://bwilliamson55.github.io/` | `https://blog.weeumson.com/` |
| `https://www.weeumson.com/posts/<slug>/` | `https://blog.weeumson.com/post/<slug>` |
| `https://www.weeumson.com/` | Your chosen homepage (blog or other) |

Post slugs were migrated to PocketBase / blog snapshot; use `web/scripts/import-jekyll-posts-to-pocketbase.mjs` in the triangle repo with local `_posts/` for re-import.

## This repo

- Markdown under `_posts/` remains the **historical source archive**.
- CI publish workflow **removed** (`.github/workflows/pages-deploy.yml`) — do not re-add without intent.
- **Archive** this repository on GitHub (Settings → Danger zone → Archive) when redirects to `blog.weeumson.com` are confirmed.
