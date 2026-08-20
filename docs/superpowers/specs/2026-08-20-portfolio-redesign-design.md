# Portfolio Redesign: Full Astro Rewrite

## Goals

- **Technical credibility**: Establish Saurabh as someone who builds interesting things in infrastructure, AI platforms, and trading systems
- **Personal brand hub**: Professional presence + writing + projects, expressing who he is
- **Blog-forward**: The blog is the primary content; the site exists to serve it

## Pages

Four pages total:

1. **Homepage** (`/`) — Name, one-line bio, inline tag filter, chronological post list
2. **Blog Post** (`/blog/:slug`) — Individual post with excellent reading typography
3. **About** (`/about`) — Short personal bio (3-4 paragraphs), photo, social links
4. **404** (`/404`) — Branded "page not found" with navigation back to homepage

No dedicated resume/experience page. No portfolio/projects page. LinkedIn covers the full professional background.

## Tech Stack

- **Framework**: Astro with content collections (Markdown and MDX files for blog posts)
- **Styling**: Tailwind CSS v4, dark mode via CSS custom properties (see Dark Mode section)
- **Deployment**: GitHub Pages via Astro's static adapter + GitHub Actions
- **URL**: `singh-saurabh.github.io` (unchanged)
- **Node.js**: >= 18, with `.nvmrc` for consistency
- **Package manager**: npm

### Why Astro

- Built for content sites, outputs static HTML by default
- First-class Markdown/MDX support with content collections
- Islands architecture for interactive MDX components (minimal JS shipped)
- Modern DX, fast builds, easy deployment

### MDX Support

Blog posts can use either `.md` (plain Markdown) or `.mdx` (Markdown + JSX components). MDX enables interactive content within posts — embedded visualizations, live demos, animated diagrams, toggleable UI, etc.

- Install `@astrojs/mdx` integration
- Content collection accepts both `.md` and `.mdx` files
- Interactive components use Astro's `client:*` directives for hydration:
  - `client:visible` — hydrate when the component scrolls into view (default for most embeds)
  - `client:load` — hydrate immediately on page load (for above-the-fold interactivity)
  - `client:idle` — hydrate when the browser is idle
- Components live in `src/components/interactive/` and are imported directly in MDX files
- Posts without interactive elements stay as plain `.md` — no overhead

Example usage in a post:

```mdx
---
title: "Visualizing Container Layer Loading"
---

Here's how Fastpull lazily loads layers:

import LayerLoadingDemo from '../../components/interactive/LayerLoadingDemo.jsx'

<LayerLoadingDemo client:visible />
```

### Project Structure

```
src/
  components/
    Header.astro
    Footer.astro
    TagFilter.astro       # includes inline <script> for filtering
    PostList.astro
    ThemeToggle.astro
    interactive/          # MDX-embeddable components
  content/
    blog/                 # .md and .mdx posts
  content.config.ts       # Zod schema for content collections
  layouts/
    BaseLayout.astro      # Shared HTML shell, head, fonts, theme script
    PostLayout.astro      # Extends BaseLayout, adds prose styling
  pages/
    index.astro
    about.astro
    404.astro
    blog/
      [...slug].astro
  styles/
    global.css            # Tailwind imports, CSS custom properties, prose styles
public/
  fonts/                  # Self-hosted font files
  images/
    posts/                # Blog post images
  favicon.ico
```

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

### Font Loading

Fonts are **self-hosted** in `public/fonts/` — no Google Fonts CDN dependency. This eliminates third-party requests and GDPR concerns.

- Use variable font files where available (Plus Jakarta Sans and Lora both have variable versions)
- `@font-face` declarations with `font-display: swap` to avoid flash of invisible text
- Preload the primary body font (Lora regular) via `<link rel="preload">` for fastest first paint

### Colors

**Light mode (default):**

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

All color combinations meet WCAG AA contrast (4.5:1 for body text, 3:1 for large text).

### Code Blocks

- Dark background (`#1E1E2E`) in both light and dark mode
- Catppuccin Mocha-inspired syntax highlighting
- Shiki built-in to Astro, using `catppuccin-mocha` theme
- 14px JetBrains Mono, line-height 1.7
- 12px border-radius, 24px padding
- Subtle `rgba(255,255,255,0.06)` border

### Dark Mode

**Mechanism:** CSS custom properties on `:root`, switched via `[data-theme="dark"]` selector. Tailwind v4 dark variant configured to match:

```css
@custom-variant dark (&:where([data-theme="dark"], [data-theme="dark"] *));
```

This means `dark:bg-zinc-950` in Tailwind utilities responds to the same `data-theme` attribute that the CSS custom properties use. One mechanism, two systems in sync.

**Implementation:**
- Inline `<script>` in `<head>` (before body renders) reads `localStorage('theme')` or falls back to `prefers-color-scheme`, sets `data-theme` on `<html>`
- Manual toggle in header cycles between light/dark, persists to `localStorage`
- No flash of wrong theme on page load

### Responsive Design

- **Content area**: 640px max-width with 24px horizontal padding (shrinks to 16px below 480px)
- **Header**: Stays single row — the nav is only 3 items (blog, about, toggle) so it fits even at 320px
- **Tag filter pills**: `flex-wrap` so they wrap to a second row on narrow screens
- **Post body font**: Stays 18px on mobile (already sized for comfortable reading)
- **Post title**: Scales down slightly on mobile via `clamp(24px, 5vw, 34px)`
- **Code blocks**: `overflow-x: auto` for horizontal scrolling; full-bleed (negative margins) on mobile to maximize code width

### Accessibility

- Semantic HTML landmarks: `<nav>`, `<main>`, `<article>`, `<footer>`
- Skip-to-content link (visually hidden, visible on focus)
- ARIA label on theme toggle button ("Switch to dark mode" / "Switch to light mode")
- Visible focus rings on all interactive elements (2px offset, accent color)
- `prefers-reduced-motion: reduce` — disable any transitions/animations
- All color combinations pass WCAG AA contrast ratios

### SEO & Meta Tags

- **Title template**: `{Post Title} | Saurabh Singh` (homepage: just `Saurabh Singh`)
- **Open Graph tags**: `og:title`, `og:description`, `og:type` (`article` for posts, `website` for pages), `og:url`
- **Canonical URLs** on every page
- **Meta description** from post `description` field or page-level description
- No default social sharing image for now (can be added per-post via frontmatter later)
- Implemented via a `<SEO>` component included in `BaseLayout.astro`

## Page Designs

### Header (shared)

- Left: `saurabh singh` (lowercase, 15px, bold)
- Right: `blog` | `about` nav links + dark mode toggle icon
- No border below header — clean separation via whitespace
- 640px max-width content area, centered

### Homepage

1. **Intro section**: Name (36px, bold) + one-line bio in Lora serif (18px, secondary color). Bio mentions TensorFuse (linked), Quantbox Trading, IIT Roorkee.
2. **Divider**: 1px border
3. **Tag filter**: Row of pill buttons — "all" (active/inverted), plus tags derived from posts. Clicking filters the list inline.
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
   - Images: full content width, 8px radius, use Astro `<Image>` component for optimization
   - `hr`: 1px border, 48px vertical margin
3. **Back link**: Arrow icon + "Back to all posts" below content
4. **Footer**: Same as homepage

### About

1. **Photo**: 120px circle (placeholder for now — user will add their photo)
2. **Title**: "About" (36px)
3. **Bio**: 3-4 paragraphs in Lora serif covering current role, background, what he writes about, personal interests
4. **Social links**: GitHub (`singh-saurabh`), LinkedIn (`linkedin.com/in/saur`), Email (`saurabh@tensorfuse.io`) with icons, separated by a top border
5. **Footer**: Same as homepage

### 404

- Same header/footer as all pages
- Centered message: "Page not found" with a link back to homepage
- Minimal — just enough to not feel broken

## Blog Content Structure

### Content Collection Schema

```typescript
const blog = defineCollection({
  type: 'content',
  schema: z.object({
    title: z.string(),
    date: z.coerce.date(),
    tags: z.array(z.string()),
    description: z.string(),
    slug: z.string().optional(),   // overrides filename-derived slug
    draft: z.boolean().optional().default(false),
  }),
});
```

### URL Slugs

Slugs are derived from filenames by default (stripping any date prefix). The optional `slug` frontmatter field overrides this. Filenames follow Astro convention (no date prefix): `container-snapshotter.md`, not `2026-08-15-container-snapshotter.md`.

### Frontmatter Example

```yaml
---
title: "Building a Container Snapshotter for Sub-Second Cold Starts"
date: 2026-08-15
tags: ["infrastructure", "systems"]
description: "How we built Fastpull to achieve sub-second container cold starts using lazy filesystem loading."
---
```

## Interactive Behavior

### Tag Filter

- Default state: "all" selected (inverted pill)
- Clicking a tag: deselects "all", filters post list to show only posts with that tag
- Clicking "all": resets filter, shows all posts
- Multiple tag selection: not supported (single tag or all)
- Implemented as vanilla JS in an inline `<script>` tag within the Astro component (not a framework island — too simple to need one)

### Dark Mode Toggle

- Inline `<script>` in `<head>` (before body renders) to read `localStorage` or `prefers-color-scheme` and set `data-theme` attribute on `<html>`
- Clicking toggle cycles between light/dark and persists to `localStorage`
- No flash of wrong theme on page load

## Migration Plan

Develop on a **feature branch** (`redesign`). Only merge to `master` when the full Astro site is ready and tested — the switch is atomic, and the live site stays up during development.

From the current Jekyll site, we carry over:

- **One blog post**: `_posts/2019-3-8-hackathon-with-tf-lite.md` — converted to Astro frontmatter format, renamed to `hackathon-with-tf-lite.md` (no date prefix). Rewrite image paths from `assets/img/posts/tflite_1/` to `/images/posts/tflite_1/`. Convert raw HTML (`<h2>`, `<i>` tags) to proper Markdown.
- **Post images**: `assets/img/posts/tflite_1/` — moved to `public/images/posts/tflite_1/`
- **Favicon**: `assets/favicon.ico` — moved to `public/favicon.ico`
- **Nothing else**: All placeholder portfolio entries, stock photos, Type-on-Strap theme files, and the custom landing page are discarded

### Cleanup

- Remove `.travis.yml` and disable Travis CI integration
- Remove Jekyll-specific files: `Gemfile`, `Gemfile.lock`, `_config.yml`, `type-on-strap.gemspec`
- Remove old theme directories: `_layouts/`, `_includes/`, `_sass/`, `_posts/`, `_drafts/`, `_portfolio/`, `pages/`
- Remove old assets: `assets/` directory (after extracting favicon and post images)

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
