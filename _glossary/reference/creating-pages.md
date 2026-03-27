---
title: Creating & Deploying Pages
category: reference
slug: creating-pages
description: Step-by-step guide to creating glossary entries, tool pages, and new categories in SCiMOS — including front matter, nav wiring, and deployment via GitHub Pages.
---

# Creating & Deploying Pages

This guide covers everything needed to add content to SCiMOS — from a single glossary
entry to a new category or interactive tool page — and how to deploy the result.

---

## Site structure overview

```
scimos/
├── _glossary/              ← markdown content pages
│   ├── device-hardware/
│   │   └── cuda-core.md
│   ├── performance/
│   └── reference/
├── _data/
│   └── nav.yml             ← drives sidebar + homepage
├── tools/
│   └── number-formats.html ← interactive tool pages
├── _layouts/
│   ├── default.html        ← shell (sidebar, terminal)
│   └── entry.html          ← glossary entry layout
├── _sass/                  ← SCSS partials
└── assets/css/main.scss    ← imports all partials
```

---

## 1. Adding a glossary entry

### Step 1 — Create the markdown file

Place it at `_glossary/<category>/<slug>.md`:

```bash
touch _glossary/device-hardware/memory-controller.md
```

### Step 2 — Write the front matter

Every page needs these four fields:

```yaml
---
title: Memory Controller
category: device-hardware
slug: memory-controller
description: Routes data between CPU/GPU cores and DRAM, managing address translation and bandwidth arbitration.
---
```

| Field | Required | Purpose |
|-------|----------|---------|
| `title` | yes | Page heading and sidebar link text |
| `category` | yes | Must match a `slug` in `_data/nav.yml` |
| `slug` | yes | Must match the filename (without `.md`) |
| `description` | recommended | Used in `<meta>` description tag |

### Step 3 — Write the content

Standard kramdown / GFM markdown. Everything supported:

```markdown
# Memory Controller

Brief intro paragraph.

## How it works

...

## Bandwidth calculation

| Parameter | Value |
|-----------|-------|
| Bus width | 512-bit |
| Clock     | 1.2 GHz |
| BW        | 153.6 GB/s |

## Code example

    ```c
    cudaMemcpy(dst, src, size, cudaMemcpyHostToDevice);
    ```
```

See the [Markdown Cheatsheet](/reference/markdown-cheatsheet/) for every supported construct.

### Step 4 — Add to nav

Open `_data/nav.yml` and add the entry under the right category:

```yaml
- label: Device Hardware
  slug: device-hardware
  entries:
    - title: CUDA Core
      slug: cuda-core
    - title: Memory Controller     # ← new
      slug: memory-controller
```

The order here is the order in the sidebar and the prev/next pagination.

### Step 5 — Preview

```bash
bundle exec jekyll serve
```

Visit `http://127.0.0.1:4000/device-hardware/memory-controller/`.

---

## 2. Adding a new category

### Step 1 — Add to nav

```yaml
# _data/nav.yml
- label: Interconnects
  slug: interconnects
  entries:
    - title: NVLink
      slug: nvlink
    - title: PCIe
      slug: pcie
```

### Step 2 — Create the folder and first entry

```bash
mkdir _glossary/interconnects
touch _glossary/interconnects/nvlink.md
```

Front matter for entries in the new category:

```yaml
---
title: NVLink
category: interconnects      ← must match new slug
slug: nvlink
description: NVIDIA's high-bandwidth GPU-to-GPU interconnect.
---
```

No other configuration needed — Jekyll picks up new collections automatically.

---

## 3. Adding a tool page

Tool pages are plain HTML files with a Jekyll front matter header. They live
outside `_glossary/` and need an explicit `href:` in the nav so the sidebar
knows their URL.

### Step 1 — Create the HTML file

```bash
touch tools/my-tool.html
```

Minimal template:

```html
---
layout: default
title: My Tool
permalink: /tools/my-tool/
---

<div class="tool-page">

  <div class="tool-hero">
    <div class="cmd-bar">
      <span class="prompt-g">vipin@scimos-node</span><span class="ps">:</span>
      <span class="pp">~/tools</span><span class="pd">$</span>
      <span class="cmd-text"> ./my-tool --interactive</span>
    </div>
    <h1>My Tool</h1>
    <p>Short description of what this tool does.</p>
  </div>

  <!-- tool UI here -->

</div>

<script>
  // tool logic here
</script>
```

### Step 2 — Register in nav with href override

```yaml
- label: Tools
  slug: tools
  entries:
    - title: Number Formats
      slug: number-formats
      href: /tools/number-formats/
    - title: My Tool               # ← new
      slug: my-tool
      href: /tools/my-tool/        # ← explicit URL required
```

Without `href:`, the sidebar generates `/tools/my-tool/` from the category/slug
pattern — which happens to be the same, but specifying it explicitly makes the
active-link detection work correctly.

### Step 3 — Style with existing classes

The `_sass/_tool.scss` partial already defines reusable classes:

| Class | Purpose |
|-------|---------|
| `.tool-page` | Page wrapper with padding and max-width |
| `.tool-hero` | Header section with cmd-bar + h1 + description |
| `.presets-label` / `.presets-row` / `.preset-btn` | Quick-select button rows |
| `.fmt-card` / `.fmt-input` | Input card with header and editable field |
| `.fmt-group` / `.fmt-group-label` | Labelled section divider |
| `.bits-section` / `.bit-btn` | 32-bit visualiser grid |

---

## 4. Deployment

### Option A — GitHub Pages (recommended)

GitHub Pages builds Jekyll sites automatically on every push to `main`.

```bash
# One-time setup
git remote add origin https://github.com/<user>/scimos.git
git push -u origin main
```

Then in the repository **Settings → Pages**:
- Source: `Deploy from a branch`
- Branch: `main` / `root`

Every subsequent `git push` triggers a rebuild. The site is live at
`https://<user>.github.io/scimos/` within ~60 seconds.

> If you use a custom domain, add a `CNAME` file at the repo root containing
> the domain name (`scimos.example.com`).

### Option B — Manual build and rsync

```bash
# Build
JEKYLL_ENV=production bundle exec jekyll build

# Upload _site/ to any static host
rsync -avz --delete _site/ user@host:/var/www/scimos/
```

### Option C — Netlify / Cloudflare Pages

Connect the GitHub repo and set:

| Setting | Value |
|---------|-------|
| Build command | `bundle exec jekyll build` |
| Publish directory | `_site` |
| Ruby version | `3.3.0` (set via `RUBY_VERSION` env var or `.ruby-version`) |

Netlify and Cloudflare Pages will rebuild automatically on every push.

---

## 5. Local development workflow

```bash
# Install dependencies (first time)
bundle install

# Start dev server with live reload
bundle exec jekyll serve --livereload

# Build for production (minified CSS)
JEKYLL_ENV=production bundle exec jekyll build
```

The dev server watches for file changes and rebuilds automatically. Changes to
`_config.yml` require a server restart.

---

## Quick reference

```
New glossary entry:
  1. _glossary/<category>/<slug>.md  (title, category, slug, description)
  2. _data/nav.yml                   (add entry under right category)

New category:
  1. _data/nav.yml                   (new label + slug + entries list)
  2. _glossary/<new-slug>/           (create folder + at least one .md)

New tool page:
  1. tools/<slug>.html               (layout: default, permalink: /tools/<slug>/)
  2. _data/nav.yml                   (add with href: /tools/<slug>/)

Deploy (GitHub Pages):
  git add . && git commit -m "..." && git push
```
