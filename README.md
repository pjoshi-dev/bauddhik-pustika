# सिंहगड भाग — मासिक बौद्धिक पुस्तिका

Monthly bulletin pages for RSS Nanded Nagar, Singhad Bhag. Free static hosting, one permanent link per issue, plus a `/current` link that always points at the latest one.

## Folder structure

```
BP/
├── index.html          ← archive/landing page, lists all issues
├── _redirects          ← Cloudflare Pages / Netlify rewrite rule for /current
├── 2026-09/
│   ├── index.html       ← the published page for that issue
│   ├── source.md        ← the raw content it was built from
│   ├── og-card.html      ← source for the social-share preview image (edit + re-render each month)
│   └── og-image.png      ← rendered 1200×630 image referenced by the og:image meta tag
├── 2026-10/             ← next month, same shape
│   ├── index.html
│   └── source.md
└── current/
    └── index.html       ← fallback copy of the latest issue (only needed on GitHub Pages)
```

Each issue gets its own dated folder (`YYYY-MM`, named after the issue's lead month) so its link **never changes**, even after newer issues are published.

## Publishing a new month

1. Create the new folder, e.g. `2026-10/`, with `source.md` (the raw content) and `index.html` (the built page).
2. Add a row for it to the archive list in the root `index.html`.
3. Update `_redirects` so both lines point at the new folder:
   ```
   /current       /2026-10/index.html   200
   /current/      /2026-10/index.html   200
   ```
4. If hosting on GitHub Pages (no rewrite support there — see below), also copy the new `index.html` into `current/index.html`, overwriting the old one.
5. Duplicate `og-card.html` into the new folder, edit its issue-line text, and re-render it to `og-image.png` (see "Social-share (OG) image" below). Update the `og:image`/`og:url` meta tags in the new `index.html` (and `current/index.html`) to point at it.
6. Commit and push. Ask Claude to do steps 1–5 each month; it can build the page and update these files directly.

## Social-share (OG) image

Each issue's `<head>` carries `og:title`, `og:description`, `og:image` and `og:url` meta tags so the link shows a proper preview card when shared (WhatsApp, Telegram, etc). The image is rendered from `og-card.html` (plain HTML/CSS, same brand colors/fonts as the bulletin) using a headless browser, since it has to ship as an actual PNG:

```
npx playwright@latest install chromium   # first time only
node -e "
import('playwright').then(async ({chromium}) => {
  const b = await chromium.launch();
  const p = await b.newPage({ viewport: { width: 1200, height: 630 } });
  await p.goto('file://' + process.cwd() + '/2026-09/og-card.html');
  await p.evaluate(() => document.fonts.ready);
  await p.waitForTimeout(500);
  await p.screenshot({ path: '2026-09/og-image.png' });
  await b.close();
});
"
```

Ask Claude to do this each month — it can edit `og-card.html`'s text and re-render in one step.

## Known font issue — missing "प्" before "ट"

**Tiro Devanagari Marathi** and **Baloo 2** (the two fonts used throughout these pages) both fail to render the प्+ट conjunct — "सप्टेंबर" silently loses its "प्" and reads as "सटेंबर". This is a bug in those specific fonts' rendering tables, not an encoding issue (Noto Serif Devanagari and plain system fonts render it fine). The fix is to insert a zero-width joiner (U+200D) right after the ् in प्, i.e. write सप्&#x200D;टेंबर (invisible in a text editor, but present) instead of सप्टेंबर. Every occurrence of "सप्टेंबर" in these pages already has this fix applied. If a future month's content includes this or a visually similar word where a conjunct looks like it's dropping a letter, apply the same ZWJ fix and verify with a headless-browser screenshot before publishing.

## Hosting — Cloudflare Pages (recommended)

Free, no domain purchase needed, no cost at this traffic level, and supports the `_redirects` rewrite so `/current` never needs a duplicated copy.

1. Push this folder to a GitHub (or GitLab) repository.
2. Go to the Cloudflare dashboard → **Workers & Pages** → **Create application** → **Pages** → **Connect to Git**, sign in, and pick this repository.
3. Build settings: no build command, output directory = `/` (repo root).
4. Deploy. You'll get a URL like `https://<project-name>.pages.dev`.
5. Share links like `https://<project-name>.pages.dev/current` and `https://<project-name>.pages.dev/2026-09`.

Every future push (new month's folder + updated `_redirects`) redeploys automatically.

## Hosting — GitHub Pages (simpler, one caveat)

Also free, but GitHub Pages has no server-side rewrite, so `/current` can't transparently mirror another folder — that's what the `current/` folder with its own copied `index.html` is for (step 4 above).

1. Push this folder to a GitHub repository.
2. Repo → **Settings** → **Pages** → Source: deploy from the `main` branch, root folder.
3. Name the repo `<your-username>.github.io` so the site serves from the domain root (needed for the root-relative links like `/current` to resolve correctly). A regular project repo would serve under a `/reponame/` prefix instead, which breaks those links unless you adjust them.

## Known issue — current deployment is Workers, not Pages

The live deployment (`https://bauddhik-pustika.prafulla-b6a.workers.dev`) was created through Cloudflare's unified "Workers & Pages" dashboard flow, and landed as a **Worker with static assets** rather than a classic **Pages** project. This matters because `_redirects` rewrites (what makes `/current` work) are a Pages-only feature — on the current deployment, `/current` and even `/` return 404. Two ways to resolve, still undecided as of this writing:

1. Recreate the deployment as an actual Pages project (look for a distinct **Pages** tab/option in **Workers & Pages → Create application**) — gives a cleaner `<project>.pages.dev` URL too, with no account-name segment.
2. Stay on Workers and implement the `/current` alias a different way (Workers static assets may need its own redirect mechanism rather than `_redirects` — needs checking against current Cloudflare docs).

Until this is resolved, only direct issue links (e.g. `/2026-09/`) are reliable on the live site; don't share `/current` or the bare root URL yet.

## Notes

- No domain purchase or hosting cost at ~1000 monthly users on either platform's free tier.
- HTTPS is automatic on both.
- Keep `source.md` per issue — it's the editable source the page is built from.
