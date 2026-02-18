# venkatesh.dev

Personal portfolio and blog built with [Hugo](https://gohugo.io/), deployed via GitHub Actions to GitHub Pages.

**Live:** https://ni8crawler18.github.io/me

## Structure

```
me/
├── hugo.toml                  # Site config, params, Giscus, Umami
├── content/
│   ├── _index.md              # Homepage content
│   └── blog/                  # Blog posts (markdown)
├── data/
│   ├── projects.yaml          # Project cards + modal data
│   └── hackathons.yaml        # Hackathon recognition slides
├── layouts/
│   ├── _default/baseof.html   # Base template shell
│   ├── index.html             # Homepage (assembles partials)
│   ├── 404.html               # Custom 404 page
│   ├── blog/
│   │   ├── list.html          # /blog/ listing
│   │   └── single.html        # /blog/:slug/ post page
│   └── partials/              # Section partials (intro, about, works, etc.)
├── static/
│   ├── css/                   # vendor.css, styles.css, custom.css
│   ├── js/                    # plugins.js, main.js
│   └── images/                # Project screenshots, icons
├── archetypes/blog.md         # Blog post template
└── .github/workflows/hugo.yml # CI/CD deploy to Pages
```

## Adding a blog post

```bash
hugo new blog/my-post.md
```

Edit the generated file in `content/blog/`, set `draft: false` when ready, push to `master`.

## Local dev

```bash
hugo server --buildDrafts
```
