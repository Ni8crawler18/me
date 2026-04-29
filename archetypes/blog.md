---
title: "{{ replace .File.ContentBaseName "-" " " | title }}"
date: {{ .Date }}
draft: false
summary: "One-line description (120-160 chars). Used in the blog listing card AND OG/Twitter meta."
tags:
  - "Tag1"
  - "Tag2"
---

Opening hook — the contrarian claim a thoughtful reader would push back on.

## Section heading

Regular paragraph text. Mix paragraphs with diagrams, code, and tables — avoid more than ~4 paragraphs in a row.

### Subsection

- Bullet points work
- So do **bold** and *italic*

```python
def example():
    return "hello"
```

> Blockquotes get an orange left border.

| Column A | Column B |
|----------|----------|
| Tables   | work too |

### Adding a hand-drawn diagram

Drop the SVG into `static/images/blog/diagrams/<name>.svg`, then reference it:

{{</* sketch "diagram-name" "Optional caption shown as italic figcaption." */>}}

See `blog_workflow.md` at the repo root for the SVG starter template, the locked color palette, and layout patterns.
