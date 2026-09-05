# CLAUDE.md — CHI Review Desk frontend

Guidance for Claude Code in this repo. See `CONTRIBUTING.md` too; this is the operational
cheat-sheet, especially for **deploying**.

## What this is

The static site at https://chi-review.ahlab.org — one `index.html` (the whole app), gated to
`@ahlab.org` via the shared AHL login broker, themed in AHL brand colours. Backend is a
separate repo: `chi-review-ahlab-worker` (Cloudflare Worker; holds the shared key).

The page auto-detects its mode: **server** (talks to `window.API_BASE`, the shared-key
Worker — this is production), **standalone** (personal key), or **artifact** (`window.claude`).

## The other half — the backend repo (clone it alongside)

This frontend is one of **two** repos. The backend is:

> **https://github.com/augmented-human-lab/chi-review-ahlab-worker** — the Cloudflare Worker
> that holds the shared key, verifies the login, and runs the review. It has its own
> `CLAUDE.md` — read it when touching the API, auth/HMAC, prompts, or deploy/CI.

Clone both in the **same parent folder** so they sit as siblings — the prompts-sync command
uses `../chi-review-ahlab-worker/…`:

```
chi-review/
  chi-review-ahlab-org/      # this repo (git clone …/chi-review-ahlab-org.git)
  chi-review-ahlab-worker/   # backend   (git clone …/chi-review-ahlab-worker.git)
```

Production reviews run on the backend, so a change to the review prompts touches **both**
repos (see "Prompts" below).

## How to deploy (do it this way)

**Deployment = push to `main`.** GitHub Pages rebuilds automatically (~1–2 min). There is no
Actions workflow and no build step here — a push is the deploy.

```bash
git add -A && git commit -m "…"
git push origin main
# wait ~1-2 min, then confirm the change is live (cache-bust the request):
curl -s "https://chi-review.ahlab.org/?v=$(date +%s)" | grep -o "<title>[^<]*</title>"
```

To confirm a Pages build finished: `gh api repos/augmented-human-lab/chi-review-ahlab-org/pages/builds/latest -q .status` → `built`.

## Verify UI changes with a headless screenshot

Editing `index.html` and curling is not enough — render it. Serve locally and screenshot with
headless Chrome (the app boots behind the sign-in gate; seed a fake session in `localStorage`
to see the app, as needed). Always eyeball a screenshot after HTML/CSS edits.

## Prompts: keep identical to the backend

`prompts.js` here must be byte-identical to `chi-review-ahlab-worker/prompts.cjs` (which is
what production reviews actually use). When changing prompts, edit both and commit both — the
worker's CI fails if they drift. Check: `cmp prompts.js ../chi-review-ahlab-worker/prompts.cjs`.

## The share card (OG image)

Source: `assets/og-card.html` → `assets/og-image.jpg` (1200×630). To regenerate after editing
the card:

```bash
chrome --headless=new --window-size=1200,630 --virtual-time-budget=3500 \
  --screenshot=assets/og-image.png "file://$PWD/assets/og-card.html"
sips -s format jpeg -s formatOptions 82 assets/og-image.png --out assets/og-image.jpg
rm assets/og-image.png
```

Keep the OG image a JPG/PNG (crawlers reject SVG) and under ~300 KB.

## Don't change without reason

- `window.API_BASE` (top of `index.html`) — the backend origin; only if the Worker moves.
- `CNAME` — pins `chi-review.ahlab.org`. Leave it.
- The gate/auth block (`ahlLogin`/`ahlGetSession`/`/auth-callback/`) — changing storage keys
  or the broker URL breaks sign-in.

## No hover translate-Y lifts

Per house style, UI elements must not move on `:hover` via `transform: translateY(...)`. Use
box-shadow / background / border / colour for hover affordances instead.
