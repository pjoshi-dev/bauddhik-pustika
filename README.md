# सिंहगड भाग — मासिक बौद्धिक पुस्तिका

Monthly bulletin pages for RSS Nanded Nagar, Singhad Bhag. Free static hosting on Cloudflare Pages, one permanent link per issue, plus a `/current` link that always points at the latest one.

Live at: **https://bauddhik-pustika.pages.dev**

## Folder structure

```
BP/
├── index.html          ← archive/landing page: "चालू अंक" card + list of every issue
├── _redirects          ← Cloudflare Pages rewrite rule for /current
├── og-card.html         ← source for the social-share preview image (generic, reused every month)
├── og-image.png         ← rendered 1200×630 image, shared by every issue's og:image tag
├── 2026-09/
│   ├── index.html       ← the published page for that issue
│   └── source.md        ← the raw content it was built from
├── 2026-10/             ← next month, same shape
│   ├── index.html
│   └── source.md
└── current/
    └── index.html       ← copy of the latest issue (redundant backup — see "How /current works")
```

Each issue gets its own dated folder (`YYYY-MM`, named after the issue's lead month) so its link **never changes**, even after newer issues are published.

## How `/current` works

`_redirects` rewrites `/current` to the latest issue's folder with an HTTP 200 (a *rewrite*, not a redirect — the address bar keeps showing `/current`):

```
/current       /2026-09/   200
/current/      /2026-09/   200
```

**The destination must be the directory path (`/2026-09/`), never the filename (`/2026-09/index.html`).** Pointing it at the filename was a real bug shipped and fixed in this repo (2026-09-10): Cloudflare Pages' own URL-normalization then 308-redirects `/2026-09/index.html` → `/2026-09/`, which makes `/current` visibly jump to `/2026-09/` in the address bar instead of quietly rewriting — the opposite of the intent.

`current/index.html` (a plain copy of the latest issue) is kept only as a portability fallback in case this ever moves to a host without rewrite support (e.g. GitHub Pages). On the live Cloudflare Pages deployment it is not actually used — `_redirects` handles `/current` directly — but keep it in sync anyway, it costs nothing.

## Publishing a new month — full checklist

Given a new month's raw content (a `source.md`-shaped brief), do all of this in one pass:

1. **Create `YYYY-MM/source.md`** with the raw content as given.
2. **Build `YYYY-MM/index.html`**: copy the previous month's `index.html` as your starting point (this preserves the `<!doctype html>`/`<head>`/charset skeleton, the design system, and the OG meta tag block — never start from a bare fragment, see "Known gotchas" below). Then:
   - Replace each section's content with the new month's text, keeping the existing section structure/markup (मनोगत, सुभाषित, अमृतवचन, बोधकथा, दिनविशेष, वैयक्तिक पद्य, सांघिक पद्य, etc. — match whatever sections the new source actually has).
   - Update `<title>`, `og:title`, `og:description`, `og:url` to the new month. **Leave `og:image` pointed at `/og-image.png`** — it's shared and generic, never touch it per-issue.
   - Update the masthead issue-line (month/date, Hindu months, Shaka year) and the footer address line's date.
   - If the content includes YouTube/Google Drive links for songs/recordings, embed them inline with `<div class="embed-frame"><iframe ...></iframe></div>` plus a small fallback text link underneath (the pattern already used in `2026-09/index.html`) — don't just leave them as bare links.
   - Check any new/unusual conjunct-heavy words for the font bug described below, and verify visually with a headless-browser screenshot before considering the page done (see "Known font issue").
3. **Update root `index.html`**:
   - Change the "चालू अंक" card's `<span class="title">` text to the new issue's title (its link stays `/current`, don't change the href).
   - Add a new `<li>` at the **top** of `.issue-list` linking to `/YYYY-MM/` — keep all older rows below it.
4. **Update `_redirects`**: change both destination lines to `/YYYY-MM/` (directory path — see the warning above).
5. **Copy the new `YYYY-MM/index.html` over `current/index.html`** to keep the portability fallback in sync.
6. **Commit and push** — Cloudflare Pages auto-deploys on push to `main`. Verify the live `/current`, `/YYYY-MM/`, and root URLs afterward.

Nothing else needs to change month to month — no touching `og-card.html`, `og-image.png`, hosting config, or fonts/palette.

## Known font issue — missing "प्" before "ट"

**Tiro Devanagari Marathi** and **Baloo 2** (the two fonts used throughout these pages) both fail to render the प्+ट conjunct — "सप्टेंबर" silently loses its "प्" and reads as "सटेंबर". This is a bug in those specific fonts' rendering tables, not an encoding issue (Noto Serif Devanagari and plain system fonts render it fine). The fix is to insert a zero-width joiner (U+200D) right after the ् in प्, i.e. write सप्&#x200D;टेंबर (invisible in a text editor, but present) instead of सप्टेंबर. Every occurrence of "सप्टेंबर" in these pages already has this fix applied. **Check any new month's content for this or visually similar conjuncts** (not just सप्टेंबर — any प्+ट combination) and verify with a headless-browser screenshot before publishing:

```
node -e "
import('playwright').then(async ({chromium}) => {
  const b = await chromium.launch();
  const p = await b.newPage({ viewport: { width: 900, height: 1400 } });
  await p.goto('file://' + process.cwd() + '/YYYY-MM/index.html');
  await p.evaluate(() => document.fonts.ready);
  await p.waitForTimeout(500);
  await p.screenshot({ path: '/tmp/check.png', fullPage: true });
  await b.close();
});
"
```
(`npx playwright install chromium` once first, if not already installed.)

## Other known gotchas

- **Every page needs a real document skeleton.** These pages were originally drafted for Claude's Artifact preview tool, which auto-injects `<!doctype html>`, `<head>`, and `<meta charset="UTF-8">` — that safety net doesn't exist on real hosting. A page missing this renders Devanagari text as mojibake (browser has to guess the encoding). Always start a new month from a copy of an existing `YYYY-MM/index.html`, never a bare fragment.
- **`body` needs `align-items:flex-start`.** The flex default (`stretch`) combined with `.sheet`'s `overflow:hidden` (used only to clip rounded corners) will silently clip page content below the first screenful and break scrolling.

## Social-share (OG) image

Every page's `<head>` carries `og:title`, `og:description`, `og:image` and `og:url` meta tags so the link shows a proper preview card when shared (WhatsApp, Telegram, etc). `og:image` points at one shared, **generic** `/og-image.png` (no month/date on it) so it never needs updating month to month — it's rendered once from `og-card.html` (plain HTML/CSS, same brand colors/fonts as the bulletin) using a headless browser, since it has to ship as an actual PNG:

```
npx playwright@latest install chromium   # first time only
node -e "
import('playwright').then(async ({chromium}) => {
  const b = await chromium.launch();
  const p = await b.newPage({ viewport: { width: 1200, height: 630 } });
  await p.goto('file://' + process.cwd() + '/og-card.html');
  await p.evaluate(() => document.fonts.ready);
  await p.waitForTimeout(500);
  await p.screenshot({ path: 'og-image.png' });
  await b.close();
});
"
```

Only re-run this if the brand design itself changes (colors, fonts, wording) — not as part of the monthly workflow.

## Hosting — Cloudflare Pages

Live at `https://bauddhik-pustika.pages.dev`, deployed from the `pjoshi-dev/bauddhik-pustika` GitHub repo via Cloudflare Pages' git integration (auto-deploys every push to `main`, no build command, output directory = repo root). Free at this traffic level (~1000 MAU), no domain purchase needed.

Cloudflare's dashboard now funnels new git-connected static sites through a unified "Workers & Pages" flow, and it's easy to accidentally end up with a **Worker with static assets** (`*.workers.dev`) instead of a classic **Pages** project (`*.pages.dev`) — the former does not honor `_redirects`, so `/current` (and even `/`) will 404. If a future redeploy ever ends up back on a `*.workers.dev` domain, that's the symptom to look for; recreate it via **Workers & Pages → Create application → Pages tab → Connect to Git** specifically.

### Alternative — GitHub Pages (simpler, one caveat)

Also free, but GitHub Pages has no server-side rewrite, so `/current` can't transparently mirror another folder — that's what `current/index.html` is for.

1. Repo → **Settings** → **Pages** → Source: deploy from the `main` branch, root folder.
2. Name the repo `<your-username>.github.io` so the site serves from the domain root (needed for root-relative links like `/current` to resolve correctly) — a regular project repo serves under a `/reponame/` prefix instead, breaking those links.

## Notes

- No domain purchase or hosting cost at ~1000 monthly users on Cloudflare Pages' free tier.
- HTTPS is automatic.
- Keep `source.md` per issue — it's the editable source the page is built from.
