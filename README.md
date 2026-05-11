# Gvensan's Tools — Landing Page

Static landing page that showcases every Chrome extension in this
monorepo. Designed to be served by GitHub Pages or any static host.

```
webpage/
├── index.html        — the page
├── style.css         — modern minimal styling (no framework, no JS)
└── icons/            — 128×128 PNG per plugin (copies of each
    ├── linkedin.png    package's public/icons/icon-128.png)
    ├── github.png
    ├── x.png
    ├── youtube.png
    ├── reddit.png
    ├── huggingface.png
    └── hackernews.png
```

No build step. No JavaScript. No third-party dependencies. Just open
`index.html` in a browser to preview locally.

---

## Update before publishing

The page currently has `<gvensan>` placeholders. Replace `gvensan` in
`index.html` everywhere your GitHub handle would go if it's different
(I see `gvensan` is your CWS publisher name; confirm your GitHub
handle matches).

Once each plugin is approved on the Chrome Web Store, update its
"Coming soon to Chrome Web Store" button:

```html
<!-- before -->
<a class="btn btn-primary disabled" aria-disabled="true">Coming soon to Chrome Web Store</a>

<!-- after -->
<a class="btn btn-primary" href="https://chromewebstore.google.com/detail/linkedin-feed-toolkit-unofficial/<32-char-id>" target="_blank" rel="noopener">Install from Chrome Web Store</a>
```

The 7 "Coming soon" pills are independent — flip them one at a time
as each listing goes live.

---

## Deploy to GitHub Pages

GitHub Pages can deploy from `/` (repo root) or `/docs`, but not from
an arbitrary folder. Three options:

### Option 1: rename `webpage/` → `docs/` (simplest)

```bash
git mv webpage docs
git commit -m "chore(webpage): rename to docs for GitHub Pages"
git push
```

Then in repo Settings → Pages → Source: **Deploy from a branch** →
Branch: `main` → Folder: `/docs` → Save. Page appears at
`https://<your-handle>.github.io/<repo-name>/` within ~1 minute.

### Option 2: keep `webpage/`, deploy via GitHub Actions

Add `.github/workflows/pages.yml`:

```yaml
name: Pages
on:
  push:
    branches: [main]
    paths: ['webpage/**']
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/configure-pages@v5
      - uses: actions/upload-pages-artifact@v3
        with:
          path: webpage
      - id: deployment
        uses: actions/deploy-pages@v4
```

Then in repo Settings → Pages → Source: **GitHub Actions**. Push to
`main` and the workflow publishes the contents of `webpage/`.

### Option 3: custom domain

Whichever option you pick above, the page goes live at the GitHub
Pages URL. To point a custom domain (e.g. `tools.example.com`):

1. In repo Settings → Pages → Custom domain → enter your domain.
2. Create a `CNAME` DNS record pointing the domain at
   `<your-handle>.github.io`.
3. Drop a file named `CNAME` (no extension) into the served folder
   (`docs/` or `webpage/`) with the bare domain inside.

---

## Local preview

```bash
# from this folder
python3 -m http.server 8080
# → open http://localhost:8080
```

Or just double-click `index.html` — most browsers serve from `file://`
fine, but using a server avoids any cross-origin oddities with the
icon loading.

---

## Design notes

- Tokens (colors, spacing, radii) in CSS custom properties at the top
  of `style.css`. Brand color: `#2557D3` matches the `brand-600`
  Tailwind token used inside each extension.
- Auto dark mode via `prefers-color-scheme: dark`.
- Mobile-first responsive grid; tool cards reflow at <320 px.
- No JavaScript. Hover affordances + smooth scroll are pure CSS.
- One web font: none. Uses system stack for speed and zero requests.
