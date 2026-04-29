# venkatesh.dev

Personal portfolio and blog built with [Hugo](https://gohugo.io/), deployed via GitHub Actions to GitHub Pages.

**Live:** https://ni8crawler18.github.io/me

## Structure

```
me/
├── hugo.toml                          # Site config, params, Giscus, Umami
├── content/
│   ├── _index.md                      # Homepage content
│   └── blog/                          # Blog posts (markdown)
├── data/
│   ├── projects.yaml                  # Project cards + modal data
│   └── hackathons.yaml                # Hackathon recognition slides
├── layouts/
│   ├── _default/baseof.html           # Base template shell
│   ├── index.html                     # Homepage (assembles partials)
│   ├── 404.html                       # Custom 404 page
│   ├── blog/
│   │   ├── list.html                  # /blog/ listing
│   │   └── single.html                # /blog/:slug/ post page
│   ├── partials/                      # Section partials (intro, about, works, head, etc.)
│   └── shortcodes/
│       └── sketch.html                # Inlines hand-drawn SVG diagrams into posts
├── static/
│   ├── css/                           # vendor.css, styles.css, custom.css
│   ├── js/                            # plugins.js, main.js
│   └── images/
│       ├── hackathons/                # Recognition slide photos
│       ├── portfolio/                 # Project screenshots
│       └── blog/diagrams/             # Hand-drawn SVG flowcharts for blog posts
├── archetypes/blog.md                 # Blog post template
├── blog_workflow.md                   # ⮕ How to write posts + create diagrams
└── .github/workflows/hugo.yml         # CI/CD deploy to Pages
```

## Quick links

- **Writing a post / adding a diagram:** see [`blog_workflow.md`](./blog_workflow.md)
- **Live site:** https://ni8crawler18.github.io/me
- **Blog index:** https://ni8crawler18.github.io/me/blog/

## Local dev

```bash
hugo server --buildDrafts
# → http://localhost:1313/me/
```

## Publish

Push to `master`. GitHub Actions builds and deploys to Pages within ~60 seconds.
