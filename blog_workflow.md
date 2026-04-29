# Blog Workflow

How to write blog posts and add hand-drawn diagrams to this portfolio.

---

## Writing a Post

### 1. Create the file

```bash
hugo new blog/your-post-slug.md
```

Creates `content/blog/your-post-slug.md` with frontmatter pre-filled from `archetypes/blog.md`. The slug becomes the URL: `/blog/your-post-slug/`.

### 2. Frontmatter

```yaml
---
title: "Your Title — Optional Subtitle"
date: 2026-04-29
draft: false
summary: "One-line hook. Used in the blog listing card AND OG/Twitter meta — make it count."
tags:
  - "Privacy"
  - "Cryptography"
---
```

- `title`: 50–80 chars, em-dash works for subtitles
- `date`: ISO format, drives sort order on the listing
- `draft: false` to publish · `draft: true` hides from production
- `summary`: 120–160 chars sweet spot for SEO + card preview
- `tags`: 2–5, reuse existing tags where possible (Privacy, ZK Proofs, Cryptography, DPDP)

### 3. Content patterns

Standard markdown. Goldmark with `unsafe = true` is on — raw HTML works too.

- `## H2` for sections · `### H3` for subsections (auto-anchored)
- Fenced code blocks with language hint → Chroma syntax highlighting
- Tables render with theme styling (orange accent on headers)
- Blockquotes get an orange left border
- `[text](url)` links auto-style with the theme orange

### 4. Voice and rhythm

- Open with the contrarian hook — the claim a thoughtful reader would push back on
- Lay out the problem before the solution
- **Mix text · diagram · code · table · text** — avoid more than ~4 paragraphs in a row
- Close with a "what to build next" or "where the open questions are" pointer
- Default to no comments in code blocks; let identifiers do the work

### 5. Preview locally

```bash
hugo server --buildDrafts
# → http://localhost:1313/me/blog/your-slug/
```

### 6. Publish

```bash
git add content/blog/your-slug.md
git commit -m "add post on <topic>"
git push origin master
```

GitHub Actions builds and deploys to Pages in ~60 seconds.

---

## Adding Hand-Drawn Diagrams

Inline SVG diagrams with a sketchy aesthetic via the `{{< sketch >}}` shortcode.

### Quick start

1. Drop a new SVG into `static/images/blog/diagrams/your-diagram.svg`
2. Reference it in your markdown:

```markdown
{{</* sketch "your-diagram" "Optional caption rendered as an italic figcaption." */>}}
```

The shortcode reads the file, **inlines** the SVG into the page (so site-level fonts and CSS apply), and wraps it in a `<figure>`.

### SVG starter template

Copy into a new file and adapt — every diagram should follow this skeleton:

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 720 600" preserveAspectRatio="xMidYMid meet" role="img" aria-labelledby="d-title d-desc">
  <title id="d-title">Diagram Title</title>
  <desc id="d-desc">One-sentence description for screen readers.</desc>

  <defs>
    <filter id="sketch" x="-3%" y="-3%" width="106%" height="106%">
      <feTurbulence type="fractalNoise" baseFrequency="0.022" numOctaves="2" seed="7" result="noise"/>
      <feDisplacementMap in="SourceGraphic" in2="noise" scale="2.5"/>
    </filter>
    <filter id="sketch-strong" x="-4%" y="-4%" width="108%" height="108%">
      <feTurbulence type="fractalNoise" baseFrequency="0.028" numOctaves="2" seed="11" result="noise"/>
      <feDisplacementMap in="SourceGraphic" in2="noise" scale="3.5"/>
    </filter>
  </defs>

  <style>
    text { font-family: 'Patrick Hand', 'Comic Sans MS', cursive; }
  </style>

  <!-- Title -->
  <text x="360" y="42" font-size="28" fill="#ffffff" text-anchor="middle">Diagram Title</text>
  <text x="360" y="68" font-size="17" fill="rgba(255,255,255,0.55)" text-anchor="middle">subtitle text</text>

  <!-- Sketchy shapes — wrap in a <g> with the filter applied -->
  <g filter="url(#sketch)" fill="none" stroke-linejoin="round" stroke-linecap="round">
    <rect x="180" y="100" width="240" height="78" rx="6" stroke="#EABE7B" stroke-width="2.5" fill="rgba(234,190,123,0.08)"/>
    <!-- arrow: line + chevron -->
    <path stroke="rgba(255,255,255,0.65)" stroke-width="2.2" d="M 300 178 L 300 210"/>
    <path stroke="rgba(255,255,255,0.65)" stroke-width="2.2" d="M 293 202 L 300 212 L 307 202"/>
  </g>

  <!-- Text labels — keep OUTSIDE the filter group so they stay crisp -->
  <g text-anchor="middle" fill="#ffffff">
    <text x="300" y="144" font-size="24">LABEL</text>
  </g>
</svg>
```

### Style guide (lock these — every diagram should match)

| Element | Value |
|---|---|
| Font | `'Patrick Hand', 'Comic Sans MS', cursive` — single weight only, never use `font-weight: 700` (faux-bold looks bad) |
| Normal stroke | `#EABE7B` (theme `--color-1`), 2.5px width |
| Alert stroke | `#FF6B47` (orange-red), 3.5px — reserved for terminal / bad-outcome boxes |
| Box fill | `rgba(234,190,123,0.08)` normal · `rgba(255,107,71,0.13)` alert (keep alpha low so text reads) |
| Arrow stroke | `rgba(255,255,255,0.65)` neutral · `rgba(255,107,71,0.85)` when feeding into a terminal |
| Title font | size 28, white, centered |
| Subtitle font | size 17, `rgba(255,255,255,0.55)`, centered |
| Box label | size 22–24, white, centered |
| Annotation | size 14–17, `rgba(255,255,255,0.6)`, italic for asides |

### Sizing rules

- Blog content area is **740px wide max** → use `viewBox="0 0 720 ..."`
- Total height **under ~750** to avoid excessive scrolling; split into two diagrams if you need more
- The shortcode CSS handles responsive scaling on mobile automatically

### Layout patterns (pick the one that fits)

| Pattern | Use for | Example |
|---|---|---|
| **Linear flow** | Lifecycles, sequences, pipelines | `pii-lifecycle.svg` |
| **Branching (1→N)** | Decision trees, taxonomies, fan-outs | `four-predicate-patterns.svg`, `proving-systems-tree.svg` |
| **Convergence (N→1)** | Pipelines with multiple inputs | `xylem-flow.svg` |
| **Side-by-side** | Before/after, this vs that | `traditional-vs-zkp.svg` |

**Prefer branching layouts (1-to-4 fan) over pure linear when content supports it** — denser, more interesting visually.

### Why this setup

- **Inline SVG via shortcode** — page-level Patrick Hand font + CSS apply directly; avoids the CORS / external-fetch issues of `<img src="*.svg">`
- **Filter-based wobble** — `feTurbulence` + `feDisplacementMap` add organic imperfection without hand-coding wobbly paths. Apply the filter only to shape `<g>`s, never to text groups
- **Theme-locked palette** — every diagram automatically matches the site without hex-code drift
- **Edits are diff-able** — SVG is text, so git history is meaningful

### Updating a diagram

1. Edit the `.svg` directly (any text editor, or re-export from a sketch tool)
2. `hugo server` to preview
3. `git add` · `commit` · `push`

---

## File reference

| Concern | File |
|---|---|
| Blog post markdown | `content/blog/<slug>.md` |
| Post archetype (template) | `archetypes/blog.md` |
| Diagram SVGs | `static/images/blog/diagrams/<name>.svg` |
| Sketch shortcode | `layouts/shortcodes/sketch.html` |
| Diagram CSS | `.sketch-diagram` block in `static/css/custom.css` |
| Patrick Hand font load | `<link>` in `layouts/partials/head.html` |
| Blog listing page | `layouts/blog/list.html` |
| Blog single page | `layouts/blog/single.html` |
| Blog post styles | `.blog-post*` in `static/css/custom.css` |
| Hugo config | `hugo.toml` |
