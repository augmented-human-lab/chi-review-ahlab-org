# Contributing — CHI Review Desk frontend

This is the site served at **https://chi-review.ahlab.org** (a single static `index.html`,
gated to `@ahlab.org` members). The backend is a separate repo, `chi-review-ahlab-worker`.

## How to make a change and ship it

1. Edit `index.html` (the whole app) — or `prompts.js` (see the prompts rule below).
2. Commit and **push to `main`**.
3. That's the deploy. GitHub Pages rebuilds automatically (~1–2 min). No workflow, no build
   step, no credentials needed.

Check it's live:

```bash
curl -s -o /dev/null -w "%{http_code}\n" "https://chi-review.ahlab.org/?v=$(date +%s)"
```

## Local preview (optional)

```bash
python3 -m http.server 8787      # then open http://localhost:8787
```

Port **8787** matters: it's the sign-in callback allow-listed in the login broker, so the
real Google login works on localhost. The page talks to the live backend (CORS allows
localhost).

## Changing the review prompts

Production reviews are assembled by the **backend** from its `prompts.cjs`. This repo's
`prompts.js` is only for the standalone/artifact fallback modes — but the two must stay
identical. So: change prompts in `chi-review-ahlab-worker/prompts.cjs`, copy the same content
here into `prompts.js`, and commit both. The worker's CI fails if they differ.

## Don't change without reason

- `window.API_BASE` (top of `index.html`) — the backend origin. Only change it if the Worker
  moves.
- `CNAME` — pins the custom domain. Leave it.

## The share card (OG image)

`assets/og-image.jpg` is rendered from `assets/og-card.html`. If you change the card, re-render
it (headless Chrome, 1200×630) and re-optimise to JPEG. See `CLAUDE.md`.
