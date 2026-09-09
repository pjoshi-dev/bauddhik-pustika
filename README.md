# सिंहगड भाग — मासिक बौद्धिक पुस्तिका

Monthly bulletin pages for RSS Nanded Nagar, Singhad Bhag. Free static hosting, one permanent link per issue, plus a `/current` link that always points at the latest one.

## Folder structure

```
BP/
├── index.html          ← archive/landing page, lists all issues
├── _redirects          ← Cloudflare Pages / Netlify rewrite rule for /current
├── 2026-09/
│   ├── index.html       ← the published page for that issue
│   └── source.md        ← the raw content it was built from
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
5. Commit and push. Ask Claude to do steps 1–4 each month; it can build the page and update these files directly.

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

## Notes

- No domain purchase or hosting cost at ~1000 monthly users on either platform's free tier.
- HTTPS is automatic on both.
- Keep `source.md` per issue — it's the editable source the page is built from.
