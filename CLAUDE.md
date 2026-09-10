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

## Hosting

No domain purchase or hosting cost is expected at the traffic levels this is built for (~1000 MAU). See README.md for the exact Cloudflare Pages / GitHub Pages setup steps and the root-relative-link caveat on GitHub Pages project repos.
