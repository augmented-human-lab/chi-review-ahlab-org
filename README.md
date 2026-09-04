# CHI 2027 Review Desk — frontend (chi-review.ahlab.org)

The Augmented Human Lab pre-submission reviewer for ACM CHI 2027 papers. A single
static page, served by **GitHub Pages** at **https://chi-review.ahlab.org**, gated to
`@ahlab.org` members and themed in AHL brand colours.

Upload a submission PDF and get a seven-pass internal review (paper map → compliance
audit → five-dimension rubric → claims/method/objections → revision plan → executive
verdict → CHI reviewer keywords). Reviews run on the lab's **shared Anthropic key** —
nobody needs their own.

## How it fits together

- **This repo** — the frontend. Static; GitHub Pages serves it at the custom domain.
- **Backend** — a Cloudflare Worker (`chi-review-ahlab-worker`) at a `sankhacooray.com`
  custom domain, set in `window.API_BASE` near the top of `index.html`. It holds the
  shared key, streams the review, and verifies the AHL login on every call. Users never
  see that origin.
- **Auth** — the shared AHL member-login broker (`ahl-site-appscript`). `index.html`
  redirects to it; `/auth-callback/` finishes sign-in and caches a 7-day session in
  `localStorage`. The session's HMAC token is sent to the Worker (`x-ahl-token`) so the
  shared key can't be spent without a real `@ahlab.org` login.

Same architecture as `nusisb.ahlab.org` (GitHub Pages frontend + off-domain backend).

## The one page, three modes

`index.html` picks its mode at load (unchanged from the teammate's upstream in
`../chi-review-desk-server`):

1. **Server (shared key)** — `window.API_BASE + /api/config` answers, so it runs on the
   Worker's shared key. This is production. The PDF rides along with each pass (the
   Worker is stateless; Anthropic prompt-caches the document across the passes).
2. **Standalone (personal key)** — no backend reachable, so it asks each person for their
   own Anthropic key, kept in their browser. The offline fallback.
3. **Claude artifact** — `window.claude` present; asks Claude on the viewer's account.

## Deploy

GitHub Pages from this repo (`augmented-human-lab/chi-review-ahlab-org`). The `CNAME`
file pins `chi-review.ahlab.org`; add the matching `CNAME` DNS record
(`chi-review → augmented-human-lab.github.io`). `.nojekyll` keeps Pages from touching
the files. Everything is static — a push is the deploy.

Backend setup lives in `chi-review-ahlab-worker`. One-time broker step: add
`https://chi-review.ahlab.org/auth-callback/` to `ALLOWED_RETURN_URLS` in
`ahl-site-appscript/Code.js` and redeploy the broker.

## prompts.js

Loaded by the browser (standalone/artifact modes) and copied into the Worker as
`prompts.cjs` (server mode assembles prompts there). Keep the two in sync — this file is
the source of truth. See the upstream `../chi-review-desk-server` for the original.
