# singh-saurabh.github.io

Personal portfolio and blog at [singh-saurabh.github.io](https://singh-saurabh.github.io).

## Stack

- [Astro](https://astro.build) with content collections (Markdown/MDX)
- [Tailwind CSS](https://tailwindcss.com) v4
- Self-hosted fonts (Plus Jakarta Sans, Lora, JetBrains Mono)
- Deployed to GitHub Pages via GitHub Actions

## Development

```bash
npm install
npm run dev       # http://localhost:4321
npm run build     # output to dist/
npm run preview   # preview build locally
```

## Writing posts

Add Markdown files to `src/content/blog/`:

```yaml
---
title: "Your Post Title"
date: 2026-08-20
tags: ["infrastructure", "systems"]
description: "A short description for SEO and the post list."
---
```

For interactive posts, use `.mdx` and import components with `client:visible`.
