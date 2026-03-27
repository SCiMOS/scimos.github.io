---
title: Markdown Cheatsheet
category: reference
slug: markdown-cheatsheet
description: A complete reference for every markdown construct supported on this site — headings, text formatting, code blocks, tables, lists, images, blockquotes, and kramdown extras.
---

# Markdown Cheatsheet

A living reference for every construct you can use when writing pages on this site. Each section shows the **rendered output** followed by the **raw markdown** that produced it.

---

## 1. Headings

# Heading 1
## Heading 2
### Heading 3
#### Heading 4
##### Heading 5
###### Heading 6

```markdown
# Heading 1
## Heading 2
### Heading 3
#### Heading 4
##### Heading 5
###### Heading 6
```

---

## 2. Paragraphs & Line Breaks

This is a paragraph. Lorem ipsum dolor sit amet, consectetur adipiscing elit. Aenean commodo ligula eget dolor.

This is a second paragraph separated by a blank line.

This line has a hard break
created by two trailing spaces before the newline.

```markdown
This is a paragraph. Lorem ipsum dolor sit amet.

This is a second paragraph separated by a blank line.

This line has a hard break
created by two trailing spaces before the newline.
```

---

## 3. Emphasis

Normal text, **bold text**, *italic text*, ***bold and italic***, ~~strikethrough~~, and `inline code`.

```markdown
Normal text, **bold text**, *italic text*, ***bold and italic***, ~~strikethrough~~, and `inline code`.
```

You can also use underscores: __bold__, _italic_, ___bold italic___.

```markdown
__bold__, _italic_, ___bold italic___
```

---

## 4. Blockquotes

> This is a single-level blockquote. It can span multiple lines and will render with the terminal green left border.

> Nested blockquotes work too.
>
> > This is a second-level quote inside the first.
> >
> > > And a third level.

> **Tip:** blockquotes support full markdown inside — **bold**, `code`, lists, everything.

```markdown
> This is a single-level blockquote.

> Nested blockquotes work too.
>
> > This is a second-level quote inside the first.
> >
> > > And a third level.

> **Tip:** blockquotes support full markdown inside — **bold**, `code`, lists, everything.
```

---

## 5. Unordered Lists

- Item one
- Item two
- Item three
  - Nested item A
  - Nested item B
    - Deeply nested item

```markdown
- Item one
- Item two
- Item three
  - Nested item A
  - Nested item B
    - Deeply nested item
```

You can also use `*` or `+` as bullets:

* Asterisk bullet
+ Plus bullet
- Dash bullet

```markdown
* Asterisk bullet
+ Plus bullet
- Dash bullet
```

---

## 6. Ordered Lists

1. First item
2. Second item
3. Third item
   1. Sub-item one
   2. Sub-item two
4. Fourth item

```markdown
1. First item
2. Second item
3. Third item
   1. Sub-item one
   2. Sub-item two
4. Fourth item
```

Ordered lists can start from any number (kramdown):

3. Starting at three
4. Four
5. Five

```markdown
3. Starting at three
4. Four
5. Five
```

---

## 7. Task Lists (kramdown GFM)

- [x] Write the markdown cheatsheet
- [x] Add syntax highlighting
- [ ] Add diagrams
- [ ] Deploy to production

```markdown
- [x] Write the markdown cheatsheet
- [x] Add syntax highlighting
- [ ] Add diagrams
- [ ] Deploy to production
```

---

## 8. Inline Code

Use `backticks` for inline code. For code containing a backtick, use double backticks: ``use `backtick` here``.

```markdown
Use `backticks` for inline code.
Use double backticks: ``use `backtick` here``.
```

---

## 9. Fenced Code Blocks

### Bash / Shell

```bash
#!/usr/bin/env bash
# Build and serve the Jekyll site
export PATH="$HOME/.rbenv/shims:$PATH"

bundle exec jekyll serve \
  --port 4000 \
  --livereload \
  --incremental
```

````markdown
```bash
#!/usr/bin/env bash
bundle exec jekyll serve --port 4000 --livereload
```
````

---

### Python

```python
import torch
import torch.nn as nn

class TransformerBlock(nn.Module):
    def __init__(self, d_model: int, n_heads: int, dropout: float = 0.1):
        super().__init__()
        self.attn  = nn.MultiheadAttention(d_model, n_heads, batch_first=True)
        self.norm1 = nn.LayerNorm(d_model)
        self.ff    = nn.Sequential(
            nn.Linear(d_model, 4 * d_model),
            nn.GELU(),
            nn.Linear(4 * d_model, d_model),
        )
        self.norm2   = nn.LayerNorm(d_model)
        self.dropout = nn.Dropout(dropout)

    def forward(self, x):
        attn_out, _ = self.attn(x, x, x)
        x = self.norm1(x + self.dropout(attn_out))
        x = self.norm2(x + self.dropout(self.ff(x)))
        return x
```

````markdown
```python
class TransformerBlock(nn.Module):
    ...
```
````

---

### C / CUDA

```c
// CUDA kernel: parallel vector addition
__global__ void vector_add(
    const float* __restrict__ a,
    const float* __restrict__ b,
    float*       __restrict__ c,
    int n
) {
    int idx = blockIdx.x * blockDim.x + threadIdx.x;
    if (idx < n) {
        c[idx] = a[idx] + b[idx];
    }
}

int main(void) {
    const int N     = 1 << 20;   // 1M elements
    const int BLOCK = 256;
    const int GRID  = (N + BLOCK - 1) / BLOCK;

    float *d_a, *d_b, *d_c;
    cudaMalloc(&d_a, N * sizeof(float));
    cudaMalloc(&d_b, N * sizeof(float));
    cudaMalloc(&d_c, N * sizeof(float));

    vector_add<<<GRID, BLOCK>>>(d_a, d_b, d_c, N);
    cudaDeviceSynchronize();
    return 0;
}
```

````markdown
```c
__global__ void vector_add(...) { ... }
```
````

---

### JavaScript / TypeScript

```typescript
interface GpuEntry {
  title:       string;
  category:    string;
  slug:        string;
  description: string;
}

async function fetchEntry(category: string, slug: string): Promise<GpuEntry> {
  const res = await fetch(`/api/${category}/${slug}`);
  if (!res.ok) throw new Error(`${res.status}: ${res.statusText}`);
  return res.json() as Promise<GpuEntry>;
}

// Usage
const entry = await fetchEntry('device-hardware', 'cuda-core');
console.log(entry.title);  // "CUDA Core"
```

````markdown
```typescript
async function fetchEntry(...): Promise<GpuEntry> { ... }
```
````

---

### YAML

```yaml
# _config.yml
title: SCiMOS
description: GPU architecture reference

collections:
  glossary:
    output: true
    permalink: /:path/

defaults:
  - scope:
      type: glossary
    values:
      layout: entry

sass:
  style: compressed
```

````markdown
```yaml
collections:
  glossary:
    output: true
```
````

---

### JSON

```json
{
  "gpu": "H100 SXM5",
  "specs": {
    "sm_count": 132,
    "cuda_cores": 16896,
    "tensor_cores": 528,
    "vram_gb": 80,
    "memory_type": "HBM3",
    "bandwidth_gbps": 3350,
    "fp16_tflops": 989.5,
    "tdp_watts": 700
  },
  "nvlink": {
    "generation": 4,
    "links": 18,
    "bandwidth_gbps": 900
  }
}
```

````markdown
```json
{ "gpu": "H100 SXM5", "specs": { ... } }
```
````

---

### HTML

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>SCiMOS</title>
  <link rel="stylesheet" href="/assets/css/main.css">
</head>
<body>
  <div class="layout">
    <aside class="sidebar"><!-- nav here --></aside>
    <main class="content">
      <article class="prose">
        <h1>CUDA Core</h1>
        <p>The fundamental parallel processing unit…</p>
      </article>
    </main>
  </div>
</body>
</html>
```

````markdown
```html
<!DOCTYPE html>
<html lang="en">…</html>
```
````

---

### CSS / SCSS

```scss
// Terminal green theme variables
$bg:       #0a0c0a;
$green:    #62de61;
$font:     'JetBrains Mono', monospace;

.prose {
  h1 {
    color:         $green;
    font-size:     28px;
    border-bottom: 1px solid #1e2a1e;
    padding-bottom: 12px;
  }

  pre {
    background:  #070908;
    border:      1px solid #1e2a1e;
    border-radius: 4px;
    padding:     20px;
    overflow-x:  auto;
  }

  code {
    color:      $green;
    background: rgba(98, 222, 97, 0.08);
    padding:    2px 6px;
    border:     1px solid #1e4a1e;
  }
}
```

````markdown
```scss
$green: #62de61;
.prose { h1 { color: $green; } }
```
````

---

### SQL

```sql
-- GPU benchmark results table
CREATE TABLE benchmarks (
    id          SERIAL PRIMARY KEY,
    gpu_model   VARCHAR(64)    NOT NULL,
    precision   VARCHAR(8)     NOT NULL,   -- FP32, FP16, INT8
    tflops      DECIMAL(8, 2),
    bandwidth   DECIMAL(8, 1),             -- GB/s
    tdp_watts   INTEGER,
    tested_at   TIMESTAMP DEFAULT NOW()
);

-- Top GPUs by FP16 throughput
SELECT
    gpu_model,
    tflops AS fp16_tflops,
    ROUND(tflops / tdp_watts, 2) AS tflops_per_watt
FROM benchmarks
WHERE precision = 'FP16'
ORDER BY tflops DESC
LIMIT 5;
```

````markdown
```sql
SELECT gpu_model, tflops FROM benchmarks ORDER BY tflops DESC;
```
````

---

### Diff

```diff
- old_bandwidth = 900   # V100 HBM2, GB/s
+ new_bandwidth = 3350  # H100 HBM3, GB/s

  def roofline(arithmetic_intensity, bandwidth, peak_flops):
-     return min(bandwidth * arithmetic_intensity, peak_flops)
+     return min(bandwidth * arithmetic_intensity, peak_flops * tensor_boost)
```

````markdown
```diff
- old line
+ new line
```
````

---

### Plain text / no highlighting

```
user@gpu-node:~/glossary$ bundle exec jekyll serve
Configuration file: _config.yml
            Source: /Users/vipinvc/ws/sites/scimos
       Destination: /Users/vipinvc/ws/sites/scimos/_site
      Generating... done in 0.841 seconds.
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.
```

````markdown
```
plain text — no language tag, no highlighting
```
````

---

## 10. Tables

### Basic table

| Column A | Column B | Column C |
|---|---|---|
| Row 1 A | Row 1 B | Row 1 C |
| Row 2 A | Row 2 B | Row 2 C |
| Row 3 A | Row 3 B | Row 3 C |

```markdown
| Column A | Column B | Column C |
|---|---|---|
| Row 1 A  | Row 1 B  | Row 1 C  |
```

---

### Column alignment

| Left-aligned | Centre-aligned | Right-aligned |
|:---|:---:|---:|
| Apple | 12 | $1.20 |
| Banana | 6 | $0.60 |
| GPU node | 8 | $2.40/hr |

```markdown
| Left-aligned | Centre-aligned | Right-aligned |
|:-------------|:--------------:|---------------:|
| Apple        |       12       |          $1.20 |
```

---

### Table with inline formatting

| Feature | Value | Notes |
|---|---|---|
| Architecture | **Hopper** | 4th-gen Tensor Cores |
| VRAM | `80 GB` | HBM3 |
| FP16 throughput | ~~800 TFLOPS~~ **989 TFLOPS** | With sparsity: 1,979 |
| NVLink | [v4](https://www.nvidia.com) | 900 GB/s |

```markdown
| Feature | Value | Notes |
|---|---|---|
| Architecture | **Hopper** | 4th-gen Tensor Cores |
| VRAM | `80 GB` | HBM3 |
| FP16 | ~~800~~ **989 TFLOPS** | With sparsity: 1,979 |
```

---

## 11. Horizontal Rules

Three or more dashes, asterisks, or underscores on their own line:

---

```markdown
---
```

***

```markdown
***
```

___

```markdown
___
```

---

## 12. Links

[Inline link](https://jekyllrb.com)

[Inline link with title](https://jekyllrb.com "Jekyll docs")

[Reference-style link][jekyll-ref]

[jekyll-ref]: https://jekyllrb.com

Auto-link: <https://jekyllrb.com>

```markdown
[Inline link](https://jekyllrb.com)

[Inline link with title](https://jekyllrb.com "Jekyll docs")

[Reference-style link][jekyll-ref]

[jekyll-ref]: https://jekyllrb.com

Auto-link: <https://jekyllrb.com>
```

---

## 13. Images

Standard inline image:

![H100 GPU die architecture](/images/gpu-architecture.svg)

Image with title attribute:

![Roofline model](/images/roofline-model.svg "Roofline model — H100 SXM5")

```markdown
![Alt text](/images/gpu-architecture.svg)

![Alt text](/images/roofline-model.svg "Optional title")
```

---

## 14. HTML Entities & Special Characters

Useful escapes when angle brackets would be parsed as HTML tags:

| Result | Entity | Use case |
|---|---|---|
| &lt; | `&lt;` | Less-than in table cells |
| &gt; | `&gt;` | Greater-than in table cells |
| &amp; | `&amp;` | Literal ampersand |
| &mdash; | `&mdash;` | Em dash — |
| &ndash; | `&ndash;` | En dash – |
| &times; | `&times;` | Multiplication × |
| &ge; | `&ge;` | Greater-or-equal ≥ |
| &le; | `&le;` | Less-or-equal ≤ |
| &ne; | `&ne;` | Not equal ≠ |
| &infin; | `&infin;` | Infinity ∞ |
| &copy; | `&copy;` | Copyright © |
| &nbsp; | `&nbsp;` | Non-breaking space |

```markdown
AI &lt; 6 FLOP/byte → memory-bound
throughput &ge; 312 TFLOPS
```

Backslash-escape markdown special characters:

\*not italic\* \`not code\` \[not a link\] \# not a heading

```markdown
\*not italic\* \`not code\` \[not a link\] \# not a heading
```

---

## 15. Kramdown Extras

### Footnotes

CUDA cores execute one scalar operation per clock cycle.[^cuda-note]

Tensor Cores execute a 4×4 matrix multiply in a single cycle.[^tc-note]

[^cuda-note]: A CUDA core contains an FP32 ALU and, since Turing, a simultaneous INT32 ALU.
[^tc-note]: "Single cycle" refers to one warp instruction cycle; the underlying pipeline may be deeper.

```markdown
CUDA cores execute one scalar op per cycle.[^cuda-note]

[^cuda-note]: A CUDA core contains an FP32 ALU…
```

---

### Definition Lists

CUDA Core
: The scalar arithmetic unit inside an NVIDIA GPU SM. Each executes one FP32 or INT32 operation per clock.

Tensor Core
: A matrix multiply-accumulate unit. Performs D = A × B + C on 4×4 tiles in a single warp instruction.

Warp
: A group of 32 threads that execute in lockstep on a single SM. The fundamental scheduling unit.

```markdown
CUDA Core
: The scalar arithmetic unit inside an NVIDIA GPU SM.

Tensor Core
: A matrix multiply-accumulate unit.
```

---

### Attribute Lists (IAL)

Apply a CSS class or id directly to any element:

This paragraph has a custom class applied.
{: .tagline }

| GPU | TFLOPS |
|---|---|
| H100 | 989 |
{: style="width: auto;" }

```markdown
This paragraph has a custom class.
{: .tagline }

| GPU | TFLOPS |
|---|---|
| H100 | 989 |
{: style="width: auto;" }
```

---

### Abbreviations

CUDA and HBM are auto-expanded wherever they appear in the page once defined below.

*[CUDA]: Compute Unified Device Architecture
*[HBM]: High Bandwidth Memory

```markdown
*[CUDA]: Compute Unified Device Architecture
*[HBM]: High Bandwidth Memory
```

---

## 16. Front Matter Reference

Every page starts with YAML front matter between triple-dash fences:

```yaml
---
title: Page Title           # shown in <title> and h1
category: device-hardware   # matches a slug in _data/nav.yml
slug: cuda-core             # used in breadcrumb, cmd-bar, and pagination
description: One sentence.  # used in <meta name="description">
layout: entry               # which _layouts/*.html to use (default: entry)
---
```

Optional extra fields you can add:

```yaml
---
title: My Entry
category: performance
slug: my-entry
description: Short description.

# Optional
date: 2026-03-26            # ISO 8601
author: vipinvc
tags: [cuda, memory, perf]
image: /images/my-diagram.svg
published: true             # set false to exclude from build
---
```

---

## 17. Liquid Tags (Jekyll-only)

These are available inside `.html` files and `.md` files in the Jekyll project:

```liquid
{% raw %}
{{ site.title }}                   — output a variable
{{ page.category | upcase }}       — output with filter
{% if page.category == 'performance' %}…{% endif %}
{% for entry in site.data.nav %}…{% endfor %}
{% include sidebar.html %}
{{ '/assets/css/main.css' | relative_url }}
{% endraw %}
```

> **Note:** Liquid tags are **not** processed inside fenced code blocks, so you can document them safely.

---

## Quick Reference Card

| Element | Syntax |
|---|---|
| **Bold** | `**text**` or `__text__` |
| *Italic* | `*text*` or `_text_` |
| ***Bold italic*** | `***text***` |
| ~~Strikethrough~~ | `~~text~~` |
| `Inline code` | `` `code` `` |
| [Link](/) | `[text](url)` |
| Heading 1 | `# Title` |
| Heading 2 | `## Section` |
| Heading 3 | `### Subsection` |
| Blockquote | `> text` |
| Unordered list | `- item` |
| Ordered list | `1. item` |
| Task list | `- [x] done` / `- [ ] todo` |
| Code block | ` ```lang ` … ` ``` ` |
| Table | `\| A \| B \|` with `\|---|---|` separator |
| Horizontal rule | `---` |
| Image | `![alt](url)` |
| Footnote | `text[^ref]` + `[^ref]: note` |
| Definition | `Term` + `: Definition` |
| HTML entity | `&lt;` `&gt;` `&amp;` `&ge;` |
| Escape | `\*` `\`` `\[` `\#` |
