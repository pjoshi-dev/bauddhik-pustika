# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Monthly bulletin ("मासिक बौद्धिक पुस्तिका") pages for RSS Nanded Nagar, Singhad Bhag — plain static HTML, no build system, no dependencies, no tests. Deployed as a static site (Cloudflare Pages recommended; GitHub Pages works with one caveat, see README.md) to give each monthly issue a permanent link plus a `/current` link that always points at the latest one.

## Structure

- `index.html` — archive/landing page: a "चालू अंक" (current issue) link to `/current`, plus a list of every past issue.
- `_redirects` — Cloudflare Pages / Netlify rewrite rules. `/current` is rewritten (HTTP 200, not a redirect — the URL bar keeps showing `/current`) to the newest issue's folder.
- `YYYY-MM/` — one folder per issue, named after the issue's lead month (e.g. `2026-09/` for the सप्टेंबर–ऑक्टोबर issue). Each contains:
  - `index.html` — the published page.
  - `source.md` — the raw Marathi/Hindi content it was built from.
- `current/index.html` — a fallback copy of the latest issue's `index.html`, only needed if hosting on GitHub Pages (which has no server-side rewrite).

## Publishing a new month

1. Create `YYYY-MM/source.md` with the raw content, and `YYYY-MM/index.html` with the built page.
2. Add a row for the new issue to the archive list in the root `index.html`.
3. Update both lines in `_redirects` to point at the new folder.
4. If also deploying to GitHub Pages, overwrite `current/index.html` with the new issue's `index.html`.
5. Commit and push.

## Design system for issue pages

Each `YYYY-MM/index.html` is a self-contained page (fonts loaded via Google Fonts `<link>`, no external JS/CSS files) styled as a saffron/maroon-themed printed booklet:

- Fonts: **Baloo 2** for headings/titles, **Tiro Devanagari Marathi** (with Noto Serif Devanagari as fallback) for body text.
- Color tokens (`--paper`, `--ink`, `--saffron`, `--maroon`, `--brass`, etc.) are defined in `:root` with light/dark variants — keep new pages consistent with this palette rather than introducing new colors.
- Recurring motifs: a toran/bunting border under the masthead, a sun emblem, section-specific line-icons (quill, lotus, droplet, book, calendar, flame), and a diamond-dot divider between sections.
- `body` must keep `align-items:flex-start` (not the flex default `stretch`) — combined with `.sheet`'s `overflow:hidden` (used only to clip rounded corners), `stretch` will silently clip page content and break scrolling.
- Every page needs a real `<!doctype html><html><head>...</head><body>` skeleton with an explicit `<meta charset="UTF-8">`. These pages were originally drafted for Claude's Artifact preview tool, which auto-injects that skeleton — copy an existing `YYYY-MM/index.html` as your starting point rather than starting from a bare fragment, or the Devanagari text will render as mojibake once deployed (charset left to browser guessing).
- **Font bug**: Tiro Devanagari Marathi and Baloo 2 both drop the "प्" glyph in the प्+ट conjunct (e.g. "सप्टेंबर" renders as "सटेंबर"). Fix by inserting a zero-width joiner after the ् : `सप्‍टें...`. Check any new month's content for this or visually similar conjuncts and verify with a headless-browser screenshot (Playwright) before publishing — see README.md's "Known font issue" section for the exact fix and verification method.
- Each issue's `<head>` carries `og:title`/`og:description`/`og:image`/`og:url` for social-share previews. `og:image` always points at the shared, generic root-level `/og-image.png` (no month/date baked in, so it's never regenerated per issue) — only `og:title`/`og:description`/`og:url` change per issue. The image is rendered from root `og-card.html` via headless Chromium (README.md has the exact command) — not hand-drawn, since OG tags need a real image file.

## Hosting

No domain purchase or hosting cost is expected at the traffic levels this is built for (~1000 MAU). The live deployment currently landed as a Cloudflare **Worker with static assets** (`*.workers.dev`) rather than classic **Pages** (`*.pages.dev`) — this means `_redirects` is not honored yet and `/current` currently 404s on the live site. See README.md's "Known issue — current deployment is Workers, not Pages" section before assuming `/current` works.
