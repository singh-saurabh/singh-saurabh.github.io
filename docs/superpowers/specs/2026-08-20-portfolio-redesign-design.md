# Portfolio Redesign: Full Astro Rewrite

## Goals

- **Technical credibility**: Establish Saurabh as someone who builds interesting things in infrastructure, AI platforms, and trading systems
- **Personal brand hub**: Professional presence + writing + projects, expressing who he is
- **Blog-forward**: The blog is the primary content; the site exists to serve it

## Pages

Three pages total:

1. **Homepage** (`/`) — Name, one-line bio, inline tag filter, chronological post list
2. **Blog Post** (`/blog/:slug`) — Individual post with excellent reading typography
3. **About** (`/about`) — Short personal bio (3-4 paragraphs), photo, social links

No dedicated resume/experience page. No portfolio/projects page. LinkedIn covers the full professional background.

## Tech Stack

- **Framework**: Astro with content collections (Markdown files for blog posts)
- **Styling**: Tailwind CSS v4 with `dark:` variants for dark mode
- **Deployment**: GitHub Pages via Astro's static adapter + GitHub Actions
- **URL**: `singh-saurabh.github.io` (unchanged)

### Why Astro

- Built for content sites, outputs static HTML by default
- First-class Markdown/MDX support with content collections
- Islands architecture for the tag filter (minimal JS shipped)
- Modern DX, fast builds, easy deployment

## Design System

### Typography

| Role | Font | Weights |
|------|------|---------|
| Headings / UI | Plus Jakarta Sans | 400, 500, 600, 700 |
| Body / reading | Lora | 400, 400i, 700 |
| Code | JetBrains Mono | 400 |

- Post body: 18px Lora, line-height 1.75
- Content max-width: 640px for optimal line length (~65-75 characters)
- Headings within posts: Plus Jakarta Sans, bold, tight letter-spacing

### Colors

**Light mode:**

| Token | Value | Usage |
|-------|-------|-------|
| `--bg` | `#FAFAF8` | Page background (warm off-white) |
| `--text` | `#1C1917` | Primary text |
| `--text-secondary` | `#78716C` | Dates, metadata, nav links |
| `--accent` | `#2563EB` | Links, hover states |
| `--border` | `#E7E5E4` | Dividers, post separators |
| `--tag-bg` | `#F5F5F4` | Tag pill background |
| `--tag-text` | `#57534E` | Tag pill text |
| `--tag-active-bg` | `#1C1917` | Active tag filter pill bg |
| `--tag-active-text` | `#FAFAF8` | Active tag filter pill text |
| `--code-bg` | `#1E1E2E` | Code block background |
| `--code-text` | `#CDD6F4` | Code block text |

**Dark mode:**

| Token | Value | Usage |
|-------|-------|-------|
| `--bg` | `#09090B` | Page background |
| `--text` | `#FAFAFA` | Primary text |
| `--text-secondary` | `#A1A1AA` | Dates, metadata |
| `--accent` | `#60A5FA` | Links, hover states |
| `--border` | `#27272A` | Dividers |
| `--tag-bg` | `#27272A` | Tag pill background |
| `--tag-text` | `#A1A1AA` | Tag pill text |
| `--tag-active-bg` | `#FAFAFA` | Active tag pill bg |
| `--tag-active-text` | `#09090B` | Active tag pill text |
| `--code-bg` | `#1E1E2E` | Code blocks (unchanged) |

### Code Blocks

- Dark background (`#1E1E2E`) in both light and dark mode
- Catppuccin Mocha-inspired syntax highlighting
- Shiki built-in to Astro, using a dark theme
- 14px JetBrains Mono, line-height 1.7
- 12px border-radius, 24px padding
- Subtle `rgba(255,255,255,0.06)` border

### Dark Mode

- Respects system preference via `prefers-color-scheme`
- Manual toggle in header (sun/moon icon)
- Preference stored in `localStorage`
- Toggle script in `<head>` to prevent flash of wrong theme

## Page Designs

### Header (shared)

- Left: `saurabh singh` (lowercase, 15px, bold)
- Right: `blog` | `about` nav links + dark mode toggle icon
- No border below header — clean separation via whitespace
- 640px max-width content area, centered

### Homepage

1. **Intro section**: Name (36px, bold) + one-line bio in Lora serif (18px, secondary color). Bio mentions TensorFuse (linked), Quantbox Trading, IIT Roorkee.
2. **Divider**: 1px border
3. **Tag filter**: Row of pill buttons — "all" (active/inverted), plus tags derived from posts. Clicking filters the list inline. Implemented as a small Preact/vanilla JS island.
4. **Post list**: Each post is a clickable row with:
   - Title (17px, semibold, turns accent blue on hover)
   - Date (13px, secondary color, tabular nums) + tag pills (11px, lowercase)
   - Separated by 1px top borders
5. **Footer**: Copyright + GitHub/LinkedIn/Email links

### Blog Post

1. **Post header**: Title (34px, bold, tight tracking) + date + tag pills
2. **Prose body** in Lora serif with:
   - `text-wrap: pretty`
   - `h2`: 24px Plus Jakarta Sans bold
   - `h3`: 20px Plus Jakarta Sans semibold
   - Inline `code`: light gray background, 4px radius
   - Code blocks: dark themed (see above)
   - Blockquotes: 3px left border, italic, muted color
   - Lists: proper indentation, 8px item spacing
   - Images: full content width, 8px radius
   - `hr`: 1px border, 48px vertical margin
3. **Back link**: Arrow icon + "Back to all posts" below content
4. **Footer**: Same as homepage

### About

1. **Photo**: 120px circle (placeholder for now — user will add their photo)
2. **Title**: "About" (36px)
3. **Bio**: 3-4 paragraphs in Lora serif covering current role, background, what he writes about, personal interests
4. **Social links**: GitHub, LinkedIn, Email with icons, separated by a top border
5. **Footer**: Same as homepage

## Blog Content Structure

Posts are Markdown files in `src/content/blog/` with this frontmatter:

```yaml
---
title: "Building a Container Snapshotter for Sub-Second Cold Starts"
date: 2026-08-15
tags: ["infrastructure", "systems"]
description: "How we built Fastpull to achieve sub-second container cold starts using lazy filesystem loading."
---
```

The existing Jekyll post (`_posts/2019-3-8-hackathon-with-tf-lite.md`) will be migrated to this format with its images moved to `public/images/posts/`.

## Interactive Behavior

### Tag Filter

- Default state: "all" selected (inverted pill)
- Clicking a tag: deselects "all", filters post list to show only posts with that tag
- Clicking "all": resets filter, shows all posts
- Multiple tag selection: not supported (single tag or all)
- Implemented as a vanilla JS island — no framework needed for this

### Dark Mode Toggle

- Script in `<head>` (before body renders) to read `localStorage` or `prefers-color-scheme` and set a `data-theme` attribute on `<html>`
- Clicking toggle cycles between light/dark and persists to `localStorage`
- No flash of wrong theme on page load

## Migration Plan

From the current Jekyll site, we carry over:

- **One blog post**: `_posts/2019-3-8-hackathon-with-tf-lite.md` (converted to Astro frontmatter format)
- **Post images**: `assets/img/posts/tflite_1/` (moved to `public/images/posts/tflite_1/`)
- **Nothing else**: All placeholder portfolio entries, stock photos, Type-on-Strap theme files, and the custom landing page are discarded

## Deployment

- GitHub Actions workflow: on push to `master`, build with Astro, deploy to GitHub Pages
- No Vercel/Netlify dependency — stays on GitHub infrastructure
- CNAME file if custom domain is desired later

## What We're NOT Building

- RSS feed
- Portfolio/projects page
- Resume/experience section
- Comments system (Disqus or otherwise)
- Analytics (can be added later)
- Search functionality
- Gallery page
- Pagination (post list is short enough to show all)

## Design Mockups

Visual mockups are in Claude Design project `cd8f040d-799a-4768-9210-a580300a9a27`:
- `Homepage.dc.html` — Blog-forward homepage
- `Blog Post.dc.html` — Full reading experience with code blocks
- `About.dc.html` — Short bio page
