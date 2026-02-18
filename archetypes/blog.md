---
title: "{{ replace .File.ContentBaseName "-" " " | title }}"
date: {{ .Date }}
draft: false
summary: "One-line description of this post for card previews."
tags:
  - "Tag1"
  - "Tag2"
---

Your content here. Write in standard markdown.

## Section heading

Regular paragraph text.

### Subsection

- Bullet points work
- So do **bold** and *italic*

```python
# Code blocks render with syntax highlighting
def example():
    return "hello"
```

> Blockquotes look like this.

| Column A | Column B |
|----------|----------|
| Tables   | work too |
