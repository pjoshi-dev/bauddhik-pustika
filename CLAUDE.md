# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Monthly bulletin ("मासिक बौद्धिक पुस्तिका") pages for RSS Nanded Nagar, Singhad Bhag — plain static HTML, no build system, no dependencies, no tests. Live on Cloudflare Pages at **https://bauddhik-pustika.pages.dev**, auto-deployed from the `pjoshi-dev/bauddhik-pustika` GitHub repo on every push to `main`. Each monthly issue gets a permanent link (`/YYYY-MM/`) plus a `/current` link that always mirrors the latest one.

## Structure

- `index.html` — archive/landing page: a "चालू अंक" (current issue) card linking to `/current`, plus a list of every past issue.
- `_redirects` — Cloudflare Pages rewrite rule. `/current` is rewritten (HTTP 200, not a redirect — URL bar keeps showing `/current`) to the newest issue's **directory** (`/YYYY-MM/`, never `/YYYY-MM/index.html` — the filename form makes Cloudflare Pages' own URL-normalization 308-redirect instead of silently rewriting, a real bug shipped and fixed 2026-09-10).
- `YYYY-MM/` — one folder per issue, named after the issue's lead month (e.g. `2026-09/` for सप्टेंबर–ऑक्टोबर). Each contains `index.html` (the published page) and `source.md` (the raw Marathi/Hindi content it was built from).
- `current/index.html` — a plain copy of the latest issue's `index.html`. Not actually used on the live Cloudflare Pages deployment (`_redirects` handles `/current` directly) — kept only as a portability fallback for hosts without rewrite support (e.g. GitHub Pages). Keep it in sync anyway.
- `og-card.html` / `og-image.png` — one shared, **generic** (no month/date) social-share preview image at repo root, referenced by every page's `og:image`. Never regenerate this per month.

## Publishing a new month — do all of this in one pass

Given a new month's raw content, README.md's "Publishing a new month — full checklist" section has the exact steps (build `YYYY-MM/index.html` from a copy of the previous month's, update the root archive card + list, update `_redirects` to the new directory, sync `current/index.html`, commit, push). Follow it exactly — it exists specifically so this can be done as one pass without re-deriving the structure.

## Design system for issue pages

Each `YYYY-MM/index.html` is a self-contained page (fonts loaded via Google Fonts `<link>`, no external JS/CSS files) styled as a saffron/maroon-themed printed booklet:

- Fonts: **Baloo 2** for headings/titles, **Tiro Devanagari Marathi** (with Noto Serif Devanagari as fallback) for body text.
- Color tokens (`--paper`, `--ink`, `--saffron`, `--maroon`, `--brass`, etc.) are defined in `:root` with light/dark variants — keep new pages consistent with this palette rather than introducing new colors.
- Recurring motifs: a toran/bunting border under the masthead, a sun emblem, section-specific line-icons (quill, lotus, droplet, book, calendar, flame), and a diamond-dot divider between sections.
- `body` must keep `align-items:flex-start` (not the flex default `stretch`) — combined with `.sheet`'s `overflow:hidden` (used only to clip rounded corners), `stretch` will silently clip page content and break scrolling.
- Every page needs a real `<!doctype html><html><head>...</head><body>` skeleton with an explicit `<meta charset="UTF-8">`. These pages were originally drafted for Claude's Artifact preview tool, which auto-injects that skeleton — always copy an existing `YYYY-MM/index.html` as your starting point rather than starting from a bare fragment, or the Devanagari text renders as mojibake once deployed (charset left to browser guessing).
- **Font bug**: Tiro Devanagari Marathi and Baloo 2 both drop the "प्" glyph in the प्+ट conjunct (e.g. "सप्टेंबर" renders as "सटेंबर"). Fix by inserting a zero-width joiner after the ् : `सप्‍टें...`. Check any new month's content for this or visually similar conjuncts and verify with a headless-browser screenshot (Playwright) before publishing — README.md's "Known font issue" section has the exact fix and a ready-to-run verification script.
- Each issue's `<head>` carries `og:title`/`og:description`/`og:image`/`og:url` for social-share previews. `og:image` always points at the shared root-level `/og-image.png` (never per-issue) — only `og:title`/`og:description`/`og:url` change per issue.
- YouTube/Google Drive links for songs/recordings get embedded inline (`.embed-frame` + `<iframe>` + a small fallback text link below), not left as bare links — see the पद्य sections in `2026-09/index.html` for the pattern.

## Hosting

Cloudflare Pages, free at this traffic level (~1000 MAU), no domain purchase. Watch out: Cloudflare's dashboard can funnel a new git-connected static site into a **Worker with static assets** (`*.workers.dev`) instead of classic **Pages** (`*.pages.dev`) — the former doesn't honor `_redirects`, breaking `/current` (this happened once during initial setup and was resolved by recreating the project as an actual Pages project). If `/current` or `/` ever 404 on the live site, check whether the deployment is still a genuine Pages project before debugging anything else.
