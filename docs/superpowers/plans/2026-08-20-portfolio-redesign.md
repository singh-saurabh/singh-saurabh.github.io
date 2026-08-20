# Portfolio Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rebuild the portfolio site from Jekyll to Astro with a clean, blog-forward design featuring dark mode, tag filtering, and excellent reading typography.

**Architecture:** Static site built with Astro content collections (Markdown/MDX). Tailwind CSS v4 via Vite plugin for utility classes. CSS custom properties for the design token system (light/dark). Vanilla JS for tag filter and theme toggle. GitHub Actions deploys to GitHub Pages.

**Tech Stack:** Astro 5+, Tailwind CSS v4, Shiki (syntax highlighting), fontsource (self-hosted fonts)

**Spec:** `docs/superpowers/specs/2026-08-20-portfolio-redesign-design.md`

## Global Constraints

- Node.js >= 18
- Package manager: npm
- Deploy target: GitHub Pages at `https://singh-saurabh.github.io`
- All work on `redesign` branch, merged to `master` only when complete
- No external CDN dependencies — fonts self-hosted, no analytics, no third-party scripts
- All color combinations must pass WCAG AA contrast (4.5:1 body, 3:1 large text)
- Zero JS shipped except: theme toggle script (inline in `<head>`), tag filter script (inline on homepage)
- Content max-width: 640px centered

---

### Task 1: Project Foundation — Scaffold, Styles, Layout, Shared Components

**Files:**
- Create: `astro.config.mjs`
- Create: `package.json` (via `npm create astro`)
- Create: `.nvmrc`
- Create: `src/styles/global.css`
- Create: `src/layouts/BaseLayout.astro`
- Create: `src/components/Header.astro`
- Create: `src/components/Footer.astro`
- Create: `src/components/ThemeToggle.astro`
- Create: `src/components/SEO.astro`
- Create: `src/pages/index.astro` (minimal placeholder)

**Interfaces:**
- Consumes: nothing (first task)
- Produces:
  - `BaseLayout.astro` — accepts props `{title: string, description?: string, ogType?: string}`
  - `Header.astro` — no props
  - `Footer.astro` — no props
  - `ThemeToggle.astro` — no props
  - `SEO.astro` — accepts props `{title: string, description?: string, ogType?: string, canonicalUrl?: string}`
  - `global.css` — all CSS custom properties, Tailwind config, font declarations

- [ ] **Step 1: Create the `redesign` branch**

```bash
git checkout -b redesign
```

- [ ] **Step 2: Scaffold the Astro project**

Run from the repo root. Astro's scaffolding will ask questions — choose "Empty" template, "Yes" to TypeScript (strict), "Yes" to install dependencies.

```bash
npm create astro@latest . -- --template minimal
```

If the repo already has files, Astro may warn — accept overwriting. The old Jekyll files will be cleaned up in Task 5.

- [ ] **Step 3: Install dependencies**

```bash
npm install tailwindcss @tailwindcss/vite @astrojs/mdx @fontsource-variable/plus-jakarta-sans @fontsource-variable/lora @fontsource/jetbrains-mono
```

- [ ] **Step 4: Create `.nvmrc`**

```
22
```

- [ ] **Step 5: Configure `astro.config.mjs`**

```js
import { defineConfig } from 'astro/config';
import mdx from '@astrojs/mdx';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  site: 'https://singh-saurabh.github.io',
  integrations: [mdx()],
  vite: {
    plugins: [tailwindcss()],
  },
  markdown: {
    shikiConfig: {
      theme: 'catppuccin-mocha',
    },
  },
});
```

- [ ] **Step 6: Create `src/styles/global.css`**

This file contains the Tailwind import, dark mode custom variant, all CSS custom properties, font imports, and base styles.

```css
@import "tailwindcss";
@import "@fontsource-variable/plus-jakarta-sans";
@import "@fontsource-variable/lora";
@import "@fontsource/jetbrains-mono";

@custom-variant dark (&:where([data-theme="dark"], [data-theme="dark"] *));

:root {
  --bg: #FAFAF8;
  --text: #1C1917;
  --text-secondary: #78716C;
  --accent: #2563EB;
  --accent-hover: #1D4ED8;
  --border: #E7E5E4;
  --tag-bg: #F5F5F4;
  --tag-text: #57534E;
  --tag-active-bg: #1C1917;
  --tag-active-text: #FAFAF8;
  --code-bg: #1E1E2E;
  --code-text: #CDD6F4;

  --font-sans: 'Plus Jakarta Sans Variable', system-ui, -apple-system, sans-serif;
  --font-serif: 'Lora Variable', Georgia, 'Times New Roman', serif;
  --font-mono: 'JetBrains Mono', 'Fira Code', monospace;
}

[data-theme="dark"] {
  --bg: #09090B;
  --text: #FAFAFA;
  --text-secondary: #A1A1AA;
  --accent: #60A5FA;
  --accent-hover: #3B82F6;
  --border: #27272A;
  --tag-bg: #27272A;
  --tag-text: #A1A1AA;
  --tag-active-bg: #FAFAFA;
  --tag-active-text: #09090B;
  --code-bg: #1E1E2E;
  --code-text: #CDD6F4;
}

@theme {
  --font-sans: var(--font-sans);
  --font-serif: var(--font-serif);
  --font-mono: var(--font-mono);
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

body {
  font-family: var(--font-sans);
  background-color: var(--bg);
  color: var(--text);
  transition: background-color 0.15s, color 0.15s;
}

a {
  color: inherit;
  text-decoration: none;
}

.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}

.sr-only:focus {
  position: fixed;
  top: 0;
  left: 0;
  width: auto;
  height: auto;
  padding: 12px 24px;
  margin: 0;
  overflow: visible;
  clip: auto;
  white-space: normal;
  background: var(--accent);
  color: white;
  font-size: 14px;
  font-weight: 600;
  z-index: 9999;
  border-radius: 0 0 8px 0;
}

:focus-visible {
  outline: 2px solid var(--accent);
  outline-offset: 2px;
}

@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

- [ ] **Step 7: Create `src/components/SEO.astro`**

```astro
---
interface Props {
  title: string;
  description?: string;
  ogType?: string;
  canonicalUrl?: string;
}

const {
  title,
  description = 'Saurabh Singh — Founding engineer at TensorFuse, writing about infrastructure, AI platforms, and trading systems.',
  ogType = 'website',
  canonicalUrl,
} = Astro.props;

const siteTitle = title === 'Saurabh Singh' ? title : `${title} | Saurabh Singh`;
const canonical = canonicalUrl || Astro.url.href;
---

<title>{siteTitle}</title>
<meta name="description" content={description} />
<link rel="canonical" href={canonical} />

<meta property="og:title" content={siteTitle} />
<meta property="og:description" content={description} />
<meta property="og:type" content={ogType} />
<meta property="og:url" content={canonical} />

<meta name="twitter:card" content="summary" />
<meta name="twitter:title" content={siteTitle} />
<meta name="twitter:description" content={description} />
```

- [ ] **Step 8: Create `src/components/ThemeToggle.astro`**

```astro
---
---

<button
  id="theme-toggle"
  type="button"
  aria-label="Switch to dark mode"
  style="width:32px;height:32px;border-radius:8px;display:flex;align-items:center;justify-content:center;background:none;border:none;color:var(--text-secondary);cursor:pointer;transition:background 0.15s,color 0.15s;"
>
  <svg id="sun-icon" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
    <circle cx="12" cy="12" r="5"></circle>
    <line x1="12" y1="1" x2="12" y2="3"></line>
    <line x1="12" y1="21" x2="12" y2="23"></line>
    <line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line>
    <line x1="18.36" y1="18.36" x2="19.78" y2="19.78"></line>
    <line x1="1" y1="12" x2="3" y2="12"></line>
    <line x1="21" y1="12" x2="23" y2="12"></line>
    <line x1="4.22" y1="19.78" x2="5.64" y2="18.36"></line>
    <line x1="18.36" y1="5.64" x2="19.78" y2="4.22"></line>
  </svg>
  <svg id="moon-icon" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" style="display:none;">
    <path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path>
  </svg>
</button>

<script>
  function updateToggle() {
    const theme = document.documentElement.getAttribute('data-theme');
    const sunIcon = document.getElementById('sun-icon');
    const moonIcon = document.getElementById('moon-icon');
    const btn = document.getElementById('theme-toggle');
    if (theme === 'dark') {
      sunIcon!.style.display = 'none';
      moonIcon!.style.display = 'block';
      btn!.setAttribute('aria-label', 'Switch to light mode');
    } else {
      sunIcon!.style.display = 'block';
      moonIcon!.style.display = 'none';
      btn!.setAttribute('aria-label', 'Switch to dark mode');
    }
  }

  document.getElementById('theme-toggle')!.addEventListener('click', () => {
    const current = document.documentElement.getAttribute('data-theme');
    const next = current === 'dark' ? 'light' : 'dark';
    document.documentElement.setAttribute('data-theme', next);
    localStorage.setItem('theme', next);
    updateToggle();
  });

  updateToggle();

  document.addEventListener('astro:after-swap', updateToggle);
</script>
```

- [ ] **Step 9: Create `src/components/Header.astro`**

```astro
---
import ThemeToggle from './ThemeToggle.astro';

const pathname = Astro.url.pathname;
---

<header style="padding:32px 0;">
  <div style="max-width:640px;margin:0 auto;padding:0 24px;display:flex;align-items:center;justify-content:space-between;">
    <a
      href="/"
      style="font-weight:700;font-size:15px;letter-spacing:-0.01em;color:var(--text);text-decoration:none;"
    >
      saurabh singh
    </a>
    <nav style="display:flex;align-items:center;gap:24px;" aria-label="Main navigation">
      <a
        href="/"
        style={`font-size:14px;font-weight:500;text-decoration:none;transition:color 0.15s;color:${pathname === '/' ? 'var(--text)' : 'var(--text-secondary)'};`}
      >
        blog
      </a>
      <a
        href="/about"
        style={`font-size:14px;font-weight:500;text-decoration:none;transition:color 0.15s;color:${pathname === '/about' || pathname === '/about/' ? 'var(--text)' : 'var(--text-secondary)'};`}
      >
        about
      </a>
      <ThemeToggle />
    </nav>
  </div>
</header>
```

- [ ] **Step 10: Create `src/components/Footer.astro`**

```astro
---
---

<footer style="padding:48px 0 40px;border-top:1px solid var(--border);">
  <div style="max-width:640px;margin:0 auto;padding:0 24px;display:flex;justify-content:space-between;align-items:center;">
    <span style="font-size:13px;color:var(--text-secondary);">&copy; 2026 Saurabh Singh</span>
    <div style="display:flex;gap:20px;">
      <a href="https://github.com/singh-saurabh" style="font-size:13px;color:var(--text-secondary);text-decoration:none;transition:color 0.15s;">GitHub</a>
      <a href="https://linkedin.com/in/saur" style="font-size:13px;color:var(--text-secondary);text-decoration:none;transition:color 0.15s;">LinkedIn</a>
      <a href="mailto:saurabh@tensorfuse.io" style="font-size:13px;color:var(--text-secondary);text-decoration:none;transition:color 0.15s;">Email</a>
    </div>
  </div>
</footer>
```

- [ ] **Step 11: Create `src/layouts/BaseLayout.astro`**

```astro
---
import '../styles/global.css';
import Header from '../components/Header.astro';
import Footer from '../components/Footer.astro';
import SEO from '../components/SEO.astro';

interface Props {
  title: string;
  description?: string;
  ogType?: string;
}

const { title, description, ogType } = Astro.props;
---

<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <link rel="icon" type="image/x-icon" href="/favicon.ico" />
    <SEO title={title} description={description} ogType={ogType} />
    <script is:inline>
      (function() {
        const stored = localStorage.getItem('theme');
        if (stored) {
          document.documentElement.setAttribute('data-theme', stored);
        } else if (window.matchMedia('(prefers-color-scheme: dark)').matches) {
          document.documentElement.setAttribute('data-theme', 'dark');
        } else {
          document.documentElement.setAttribute('data-theme', 'light');
        }
      })();
    </script>
  </head>
  <body>
    <a href="#main-content" class="sr-only">Skip to content</a>
    <Header />
    <main id="main-content">
      <slot />
    </main>
    <div style="max-width:640px;margin:0 auto;padding:0 24px;">
      <Footer />
    </div>
  </body>
</html>
```

- [ ] **Step 12: Create a minimal `src/pages/index.astro`**

```astro
---
import BaseLayout from '../layouts/BaseLayout.astro';
---

<BaseLayout title="Saurabh Singh">
  <div style="max-width:640px;margin:0 auto;padding:56px 24px;">
    <h1 style="font-size:36px;font-weight:700;letter-spacing:-0.03em;line-height:1.15;margin-bottom:14px;">
      Saurabh Singh
    </h1>
    <p style="font-family:var(--font-serif);font-size:18px;line-height:1.65;color:var(--text-secondary);">
      Founding engineer at TensorFuse, building AI platforms and developer tools.
    </p>
  </div>
</BaseLayout>
```

- [ ] **Step 13: Verify the foundation works**

```bash
npm run dev
```

Open `http://localhost:4321` in the browser. Verify:
- Page renders with warm off-white background
- Header shows "saurabh singh" on left, "blog" / "about" / sun icon on right
- Clicking the sun icon toggles dark mode (background goes dark, icon changes to moon)
- Refreshing the page preserves the theme choice
- Footer shows copyright and links
- Skip-to-content link appears when pressing Tab

```bash
npm run build
```

Verify: Build completes without errors.

- [ ] **Step 14: Commit**

```bash
git add -A
git commit -m "feat: scaffold Astro project with BaseLayout, dark mode, and shared components"
```

---

### Task 2: Homepage — Content Collection, Post List, Tag Filter

**Files:**
- Create: `src/content.config.ts`
- Create: `src/content/blog/building-a-container-snapshotter.md` (sample post)
- Create: `src/content/blog/why-we-chose-django.md` (sample post)
- Create: `src/content/blog/redis-streams-consumer-groups.md` (sample post)
- Create: `src/components/PostList.astro`
- Create: `src/components/TagFilter.astro`
- Modify: `src/pages/index.astro`

**Interfaces:**
- Consumes: `BaseLayout.astro` (props: `{title, description?, ogType?}`)
- Produces:
  - Content collection `blog` with schema `{title, date, tags, description, slug?, draft?}`
  - `PostList.astro` — accepts prop `{posts: CollectionEntry<'blog'>[]}`
  - `TagFilter.astro` — accepts prop `{tags: string[]}`

- [ ] **Step 1: Create `src/content.config.ts`**

```ts
import { defineCollection } from 'astro:content';
import { glob } from 'astro/loaders';
import { z } from 'astro/zod';

const blog = defineCollection({
  loader: glob({ base: './src/content/blog', pattern: '**/*.{md,mdx}' }),
  schema: z.object({
    title: z.string(),
    date: z.coerce.date(),
    tags: z.array(z.string()),
    description: z.string(),
    draft: z.boolean().optional().default(false),
  }),
});

export const collections = { blog };
```

- [ ] **Step 2: Create sample blog posts**

These are placeholder posts so the homepage has content to display. They'll be removed or replaced with real content later.

`src/content/blog/building-a-container-snapshotter.md`:

```markdown
---
title: "Building a Container Snapshotter for Sub-Second Cold Starts"
date: 2026-08-15
tags: ["infrastructure", "systems"]
description: "How we built Fastpull to achieve sub-second container cold starts using lazy filesystem loading."
---

At TensorFuse, every millisecond of container cold start time directly translates to user-perceived latency. This post walks through how we built Fastpull, our container snapshotter that achieves sub-second cold starts by lazily loading container filesystem layers on demand.

## The Problem with Traditional Container Starts

A typical container start involves pulling all image layers, unpacking them into an overlay filesystem, and then starting the process. For a 4GB PyTorch image with CUDA libraries, it's painfully slow.

The key insight is that most containers don't actually read their entire filesystem at startup.
```

`src/content/blog/why-we-chose-django.md`:

```markdown
---
title: "Why We Chose Django for an AI Platform"
date: 2026-07-22
tags: ["infrastructure", "python"]
description: "The reasoning behind choosing Django REST Framework for Brahma's backend."
---

When we started building Brahma, we had a choice to make about the backend framework. This is why we went with Django.
```

`src/content/blog/redis-streams-consumer-groups.md`:

```markdown
---
title: "TIL: Redis Streams Consumer Groups Have Surprising Edge Cases"
date: 2026-05-04
tags: ["infrastructure"]
description: "A few things about Redis Streams consumer groups that caught us off guard in production."
---

Redis Streams consumer groups are powerful but have a few behaviors that aren't obvious from the docs.
```

- [ ] **Step 3: Create `src/components/PostList.astro`**

```astro
---
import type { CollectionEntry } from 'astro:content';

interface Props {
  posts: CollectionEntry<'blog'>[];
}

const { posts } = Astro.props;
---

<div class="post-list" data-post-list>
  {posts.map((post) => (
    <a
      class="post-item"
      href={`/blog/${post.id}/`}
      data-tags={post.data.tags.join(',')}
    >
      <div class="post-title">{post.data.title}</div>
      <div class="post-meta">
        <span class="post-date">
          {post.data.date.toLocaleDateString('en-US', { year: 'numeric', month: 'short', day: 'numeric' })}
        </span>
        <div class="post-tags">
          {post.data.tags.map((tag: string) => (
            <span class="post-tag">{tag}</span>
          ))}
        </div>
      </div>
    </a>
  ))}
</div>

<style>
  .post-list {
    display: flex;
    flex-direction: column;
  }
  .post-item {
    display: block;
    padding: 24px 0;
    border-top: 1px solid var(--border);
    text-decoration: none;
    color: inherit;
  }
  .post-item:last-child {
    border-bottom: 1px solid var(--border);
  }
  .post-title {
    font-size: 17px;
    font-weight: 600;
    letter-spacing: -0.01em;
    line-height: 1.45;
    margin-bottom: 8px;
    transition: color 0.15s;
  }
  .post-item:hover .post-title {
    color: var(--accent);
  }
  .post-meta {
    display: flex;
    align-items: center;
    gap: 12px;
    flex-wrap: wrap;
  }
  .post-date {
    font-size: 13px;
    color: var(--text-secondary);
    font-variant-numeric: tabular-nums;
  }
  .post-tags {
    display: flex;
    gap: 6px;
  }
  .post-tag {
    font-size: 11px;
    font-weight: 500;
    padding: 2px 8px;
    border-radius: 4px;
    background: var(--tag-bg);
    color: var(--tag-text);
    text-transform: lowercase;
    letter-spacing: 0.01em;
  }
</style>
```

- [ ] **Step 4: Create `src/components/TagFilter.astro`**

```astro
---
interface Props {
  tags: string[];
}

const { tags } = Astro.props;
---

<div class="tags-filter" data-tag-filter>
  <button class="tag-pill active" data-tag="all" type="button">all</button>
  {tags.map((tag) => (
    <button class="tag-pill" data-tag={tag} type="button">{tag}</button>
  ))}
</div>

<script>
  function initTagFilter() {
    const filterContainer = document.querySelector('[data-tag-filter]');
    const postList = document.querySelector('[data-post-list]');
    if (!filterContainer || !postList) return;

    const pills = filterContainer.querySelectorAll('.tag-pill');
    const posts = postList.querySelectorAll('.post-item');

    pills.forEach((pill) => {
      pill.addEventListener('click', () => {
        const tag = (pill as HTMLElement).dataset.tag;

        pills.forEach((p) => p.classList.remove('active'));
        pill.classList.add('active');

        posts.forEach((post) => {
          const postTags = (post as HTMLElement).dataset.tags?.split(',') || [];
          if (tag === 'all' || postTags.includes(tag!)) {
            (post as HTMLElement).style.display = '';
          } else {
            (post as HTMLElement).style.display = 'none';
          }
        });
      });
    });
  }

  initTagFilter();
  document.addEventListener('astro:after-swap', initTagFilter);
</script>

<style>
  .tags-filter {
    display: flex;
    flex-wrap: wrap;
    gap: 8px;
    padding: 32px 0;
  }
  .tag-pill {
    font-family: var(--font-sans);
    font-size: 13px;
    font-weight: 500;
    padding: 6px 14px;
    border-radius: 100px;
    background: var(--tag-bg);
    color: var(--tag-text);
    cursor: pointer;
    transition: all 0.15s;
    border: none;
    user-select: none;
  }
  .tag-pill:hover {
    background: var(--border);
    color: var(--text);
  }
  .tag-pill.active {
    background: var(--tag-active-bg);
    color: var(--tag-active-text);
  }
  .tag-pill.active:hover {
    background: var(--tag-active-bg);
    color: var(--tag-active-text);
  }
</style>
```

- [ ] **Step 5: Update `src/pages/index.astro` with full homepage**

```astro
---
import BaseLayout from '../layouts/BaseLayout.astro';
import PostList from '../components/PostList.astro';
import TagFilter from '../components/TagFilter.astro';
import { getCollection } from 'astro:content';

const posts = (await getCollection('blog', ({ data }) => !data.draft))
  .sort((a, b) => b.data.date.valueOf() - a.data.date.valueOf());

const allTags = [...new Set(posts.flatMap((post) => post.data.tags))].sort();
---

<BaseLayout title="Saurabh Singh">
  <div style="max-width:640px;margin:0 auto;padding:0 24px;">
    <div style="padding:56px 0 48px;">
      <h1 style="font-size:36px;font-weight:700;letter-spacing:-0.03em;line-height:1.15;margin-bottom:14px;">
        Saurabh Singh
      </h1>
      <p style="font-family:var(--font-serif);font-size:18px;line-height:1.65;color:var(--text-secondary);max-width:520px;">
        Founding engineer at <a href="https://tensorfuse.ai" style="color:var(--accent);text-decoration:underline;text-decoration-color:rgba(37,99,235,0.3);text-underline-offset:3px;">TensorFuse</a>, building AI platforms and developer tools. Previously infrastructure at Quantbox Trading. IIT Roorkee CS '21.
      </p>
    </div>

    <div style="height:1px;background:var(--border);"></div>

    <TagFilter tags={allTags} />

    <PostList posts={posts} />
  </div>
</BaseLayout>
```

- [ ] **Step 6: Verify the homepage**

```bash
npm run dev
```

Open `http://localhost:4321`. Verify:
- Intro section shows name and bio with TensorFuse linked in accent blue
- Divider line separates intro from tag filter
- Tag pills show "all" (active/inverted) + tags from posts
- Post list shows 3 sample posts sorted by date (newest first)
- Each post shows title, formatted date, and tag pills
- Clicking a tag pill filters the post list (only matching posts shown)
- Clicking "all" resets the filter
- Hovering a post title turns it accent blue
- Dark mode toggle still works
- Page is responsive — tag pills wrap on narrow screens

```bash
npm run build
```

Verify: Build completes without errors.

- [ ] **Step 7: Commit**

```bash
git add -A
git commit -m "feat: add homepage with content collection, post list, and tag filtering"
```

---

### Task 3: Blog Post Page — Dynamic Route, Prose Styling

**Files:**
- Create: `src/layouts/PostLayout.astro`
- Create: `src/pages/blog/[...slug].astro`
- Modify: `src/styles/global.css` (add prose styles)

**Interfaces:**
- Consumes: `BaseLayout.astro` (props: `{title, description?, ogType?}`), content collection `blog`
- Produces: `PostLayout.astro` — renders a full blog post with styled prose

- [ ] **Step 1: Add prose styles to `src/styles/global.css`**

Append these styles to the end of `global.css`:

```css
/* Prose styles for blog posts */
.prose {
  padding-bottom: 64px;
}

.prose p {
  font-family: var(--font-serif);
  font-size: 18px;
  line-height: 1.75;
  color: var(--text);
  margin-bottom: 24px;
  text-wrap: pretty;
}

.prose h2 {
  font-family: var(--font-sans);
  font-size: 24px;
  font-weight: 700;
  letter-spacing: -0.02em;
  line-height: 1.3;
  margin-top: 48px;
  margin-bottom: 20px;
  color: var(--text);
}

.prose h3 {
  font-family: var(--font-sans);
  font-size: 20px;
  font-weight: 600;
  letter-spacing: -0.01em;
  line-height: 1.4;
  margin-top: 36px;
  margin-bottom: 16px;
  color: var(--text);
}

.prose a {
  color: var(--accent);
  text-decoration: underline;
  text-decoration-color: rgba(37, 99, 235, 0.3);
  text-underline-offset: 3px;
  transition: text-decoration-color 0.15s;
}

[data-theme="dark"] .prose a {
  text-decoration-color: rgba(96, 165, 250, 0.3);
}

.prose a:hover {
  text-decoration-color: var(--accent);
}

.prose code {
  font-family: var(--font-mono);
  font-size: 0.875em;
  background: var(--tag-bg);
  padding: 2px 6px;
  border-radius: 4px;
  color: var(--text);
}

.prose pre {
  background: var(--code-bg) !important;
  border-radius: 12px;
  padding: 24px;
  margin: 28px 0;
  overflow-x: auto;
  border: 1px solid rgba(255, 255, 255, 0.06);
}

@media (max-width: 688px) {
  .prose pre {
    margin-left: -24px;
    margin-right: -24px;
    border-radius: 0;
  }
}

.prose pre code {
  font-family: var(--font-mono);
  font-size: 14px;
  line-height: 1.7;
  background: none;
  padding: 0;
  color: var(--code-text);
  border-radius: 0;
}

.prose blockquote {
  border-left: 3px solid var(--border);
  padding-left: 20px;
  margin: 28px 0;
}

.prose blockquote p {
  font-style: italic;
  color: var(--text-secondary);
}

.prose ul,
.prose ol {
  font-family: var(--font-serif);
  font-size: 18px;
  line-height: 1.75;
  margin-bottom: 24px;
  padding-left: 24px;
}

.prose li {
  margin-bottom: 8px;
}

.prose li strong {
  font-family: var(--font-sans);
}

.prose hr {
  border: none;
  height: 1px;
  background: var(--border);
  margin: 48px 0;
}

.prose img {
  max-width: 100%;
  height: auto;
  border-radius: 8px;
  margin: 28px 0;
}
```

- [ ] **Step 2: Create `src/layouts/PostLayout.astro`**

```astro
---
import BaseLayout from './BaseLayout.astro';

interface Props {
  title: string;
  date: Date;
  tags: string[];
  description: string;
}

const { title, date, tags, description } = Astro.props;

const formattedDate = date.toLocaleDateString('en-US', {
  year: 'numeric',
  month: 'long',
  day: 'numeric',
});
---

<BaseLayout title={title} description={description} ogType="article">
  <article style="max-width:640px;margin:0 auto;padding:0 24px;">
    <div style="padding:56px 0 40px;">
      <h1 style={`font-size:clamp(24px, 5vw, 34px);font-weight:700;letter-spacing:-0.03em;line-height:1.2;margin-bottom:16px;`}>
        {title}
      </h1>
      <div style="display:flex;align-items:center;gap:12px;flex-wrap:wrap;">
        <span style="font-size:14px;color:var(--text-secondary);font-variant-numeric:tabular-nums;">
          {formattedDate}
        </span>
        <span style="color:var(--border);font-size:14px;">&middot;</span>
        <div style="display:flex;gap:6px;">
          {tags.map((tag) => (
            <span style="font-size:12px;font-weight:500;padding:3px 10px;border-radius:4px;background:var(--tag-bg);color:var(--tag-text);text-transform:lowercase;">
              {tag}
            </span>
          ))}
        </div>
      </div>
    </div>

    <div class="prose">
      <slot />
    </div>

    <a
      href="/"
      style="display:inline-flex;align-items:center;gap:6px;font-size:14px;font-weight:500;color:var(--text-secondary);text-decoration:none;transition:color 0.15s;padding-bottom:64px;"
    >
      <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
        <line x1="19" y1="12" x2="5" y2="12"></line>
        <polyline points="12 19 5 12 12 5"></polyline>
      </svg>
      Back to all posts
    </a>
  </article>
</BaseLayout>
```

- [ ] **Step 3: Create `src/pages/blog/[...slug].astro`**

```astro
---
import { getCollection, render } from 'astro:content';
import PostLayout from '../../layouts/PostLayout.astro';

export async function getStaticPaths() {
  const posts = await getCollection('blog', ({ data }) => !data.draft);
  return posts.map((post) => ({
    params: { slug: post.id },
    props: { post },
  }));
}

const { post } = Astro.props;
const { Content } = await render(post);
---

<PostLayout
  title={post.data.title}
  date={post.data.date}
  tags={post.data.tags}
  description={post.data.description}
>
  <Content />
</PostLayout>
```

- [ ] **Step 4: Verify the blog post page**

```bash
npm run dev
```

Open `http://localhost:4321`. Click the first post ("Building a Container Snapshotter..."). Verify:
- URL is `/blog/building-a-container-snapshotter/`
- Post title is large and bold with tight tracking
- Date and tag pills appear below the title
- Body text is Lora serif, 18px, with comfortable line height
- `h2` headings are Plus Jakarta Sans, bold, with good spacing
- Code blocks have dark background (`#1E1E2E`) with syntax highlighting
- "Back to all posts" link appears at the bottom with an arrow icon
- Dark mode works on the post page
- On mobile viewport (resize to ~375px): title scales down, code blocks go full-bleed

```bash
npm run build
```

Verify: Build completes. Check that `/blog/building-a-container-snapshotter/index.html` exists in `dist/`.

- [ ] **Step 5: Commit**

```bash
git add -A
git commit -m "feat: add blog post page with prose styling and dark code blocks"
```

---

### Task 4: About Page and 404 Page

**Files:**
- Create: `src/pages/about.astro`
- Create: `src/pages/404.astro`

**Interfaces:**
- Consumes: `BaseLayout.astro` (props: `{title, description?, ogType?}`)
- Produces: Two static pages at `/about` and `/404`

- [ ] **Step 1: Create `src/pages/about.astro`**

```astro
---
import BaseLayout from '../layouts/BaseLayout.astro';
---

<BaseLayout
  title="About"
  description="Saurabh Singh — Founding engineer at TensorFuse, writing about infrastructure, AI platforms, and trading systems."
>
  <div style="max-width:640px;margin:0 auto;padding:0 24px;">
    <div style="padding:56px 0 80px;">
      <div style="width:120px;height:120px;border-radius:100%;background:var(--tag-bg);margin-bottom:32px;display:flex;align-items:center;justify-content:center;overflow:hidden;">
        <span style="font-family:var(--font-mono);font-size:11px;color:var(--text-secondary);text-align:center;line-height:1.4;padding:8px;">your<br/>photo</span>
      </div>

      <h1 style="font-size:36px;font-weight:700;letter-spacing:-0.03em;line-height:1.15;margin-bottom:40px;">
        About
      </h1>

      <div class="about-body">
        <p>
          I'm Saurabh, a founding engineer at <a href="https://tensorfuse.ai">TensorFuse</a> where I build AI deployment platforms. Before this, I spent four years building low-latency trading infrastructure at Quantbox Trading&mdash;real-time data pipelines, multi-exchange connectivity, and systems that had to work when markets were moving fast.
        </p>
        <p>
          I studied Computer Science at IIT Roorkee, where I picked up a love for systems programming and distributed systems. During college I contributed to CGAL through Google Summer of Code and did research on federated learning at Queen's University Belfast.
        </p>
        <p>
          I write here about the things I'm learning and building&mdash;infrastructure, AI platforms, trading systems, and the occasional deep dive into something that surprised me. The posts range from short TILs to longer technical write-ups.
        </p>
        <p>
          When I'm not writing code, you'll usually find me reading, tinkering with side projects, or thinking about what makes distributed systems fail in interesting ways.
        </p>
      </div>

      <div style="display:flex;gap:24px;margin-top:40px;padding-top:32px;border-top:1px solid var(--border);">
        <a class="social-link" href="https://github.com/singh-saurabh">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="currentColor"><path d="M12 0c-6.626 0-12 5.373-12 12 0 5.302 3.438 9.8 8.207 11.387.599.111.793-.261.793-.577v-2.234c-3.338.726-4.033-1.416-4.033-1.416-.546-1.387-1.333-1.756-1.333-1.756-1.089-.745.083-.729.083-.729 1.205.084 1.839 1.237 1.839 1.237 1.07 1.834 2.807 1.304 3.492.997.107-.775.418-1.305.762-1.604-2.665-.305-5.467-1.334-5.467-5.931 0-1.311.469-2.381 1.236-3.221-.124-.303-.535-1.524.117-3.176 0 0 1.008-.322 3.301 1.23.957-.266 1.983-.399 3.003-.404 1.02.005 2.047.138 3.006.404 2.291-1.552 3.297-1.23 3.297-1.23.653 1.653.242 2.874.118 3.176.77.84 1.235 1.911 1.235 3.221 0 4.609-2.807 5.624-5.479 5.921.43.372.823 1.102.823 2.222v3.293c0 .319.192.694.801.576 4.765-1.589 8.199-6.086 8.199-11.386 0-6.627-5.373-12-12-12z"></path></svg>
          GitHub
        </a>
        <a class="social-link" href="https://linkedin.com/in/saur">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="currentColor"><path d="M20.447 20.452h-3.554v-5.569c0-1.328-.027-3.037-1.852-3.037-1.853 0-2.136 1.445-2.136 2.939v5.667H9.351V9h3.414v1.561h.046c.477-.9 1.637-1.85 3.37-1.85 3.601 0 4.267 2.37 4.267 5.455v6.286zM5.337 7.433c-1.144 0-2.063-.926-2.063-2.065 0-1.138.92-2.063 2.063-2.063 1.14 0 2.064.925 2.064 2.063 0 1.139-.925 2.065-2.064 2.065zm1.782 13.019H3.555V9h3.564v11.452zM22.225 0H1.771C.792 0 0 .774 0 1.729v20.542C0 23.227.792 24 1.771 24h20.451C23.2 24 24 23.227 24 22.271V1.729C24 .774 23.2 0 22.222 0h.003z"></path></svg>
          LinkedIn
        </a>
        <a class="social-link" href="mailto:saurabh@tensorfuse.io">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z"></path><polyline points="22,6 12,13 2,6"></polyline></svg>
          Email
        </a>
      </div>
    </div>
  </div>
</BaseLayout>

<style>
  .about-body p {
    font-family: var(--font-serif);
    font-size: 18px;
    line-height: 1.75;
    color: var(--text);
    margin-bottom: 24px;
    text-wrap: pretty;
  }
  .about-body p:last-child {
    margin-bottom: 0;
  }
  .about-body a {
    color: var(--accent);
    text-decoration: underline;
    text-decoration-color: rgba(37, 99, 235, 0.3);
    text-underline-offset: 3px;
  }
  .about-body a:hover {
    text-decoration-color: var(--accent);
  }
  .social-link {
    display: flex;
    align-items: center;
    gap: 8px;
    font-size: 14px;
    font-weight: 500;
    color: var(--text-secondary);
    text-decoration: none;
    transition: color 0.15s;
  }
  .social-link:hover {
    color: var(--text);
  }
</style>
```

- [ ] **Step 2: Create `src/pages/404.astro`**

```astro
---
import BaseLayout from '../layouts/BaseLayout.astro';
---

<BaseLayout title="Page Not Found">
  <div style="max-width:640px;margin:0 auto;padding:120px 24px;text-align:center;">
    <h1 style="font-size:36px;font-weight:700;letter-spacing:-0.03em;margin-bottom:16px;">
      Page not found
    </h1>
    <p style="font-family:var(--font-serif);font-size:18px;color:var(--text-secondary);margin-bottom:32px;">
      The page you're looking for doesn't exist.
    </p>
    <a
      href="/"
      style="font-size:14px;font-weight:500;color:var(--accent);text-decoration:underline;text-underline-offset:3px;"
    >
      Go back home
    </a>
  </div>
</BaseLayout>
```

- [ ] **Step 3: Verify both pages**

```bash
npm run dev
```

Open `http://localhost:4321/about`. Verify:
- Photo placeholder circle appears
- "About" title is large and bold
- Bio text is Lora serif with good line height
- TensorFuse link is accent blue with subtle underline
- Social links show icons + labels, change color on hover
- "about" nav link in header is highlighted (darker text)
- Dark mode works

Open `http://localhost:4321/404`. Verify:
- "Page not found" message is centered
- "Go back home" link works

```bash
npm run build
```

Verify: Build succeeds. `dist/about/index.html` and `dist/404.html` exist.

- [ ] **Step 4: Commit**

```bash
git add -A
git commit -m "feat: add about page and 404 page"
```

---

### Task 5: Content Migration, Deployment, and Jekyll Cleanup

**Files:**
- Create: `src/content/blog/hackathon-with-tf-lite.md` (migrated from Jekyll)
- Create: `public/images/posts/tflite_1/` (moved from `assets/img/posts/tflite_1/`)
- Create: `public/favicon.ico` (moved from `assets/favicon.ico`)
- Create: `.github/workflows/deploy.yml`
- Delete: `.travis.yml`, `Gemfile`, `Gemfile.lock`, `_config.yml`, `type-on-strap.gemspec`
- Delete: `_layouts/`, `_includes/`, `_sass/`, `_posts/`, `_drafts/`, `_portfolio/`, `pages/`
- Delete: `assets/` (after extracting needed files)
- Delete: `index.html` (old Jekyll homepage)

**Interfaces:**
- Consumes: Content collection `blog`, `BaseLayout.astro`, `PostLayout.astro`, all pages
- Produces: A deployable site with real content and CI/CD pipeline

- [ ] **Step 1: Copy assets that we're keeping**

```bash
mkdir -p public/images/posts/tflite_1
cp assets/img/posts/tflite_1/* public/images/posts/tflite_1/
cp assets/favicon.ico public/favicon.ico
```

- [ ] **Step 2: Migrate the TFLite blog post**

Read the existing post at `_posts/2019-3-8-hackathon-with-tf-lite.md` and create a new file at `src/content/blog/hackathon-with-tf-lite.md` with:
- Astro-style frontmatter (replace `layout`, `feature-img`, `thumbnail` with `date`, `tags`, `description`)
- Convert all `<h2>` HTML tags to `##` Markdown headings
- Convert `<i>` tags to `*italic*` Markdown
- Rewrite all image paths from `assets/img/posts/tflite_1/` to `/images/posts/tflite_1/`
- Keep the Markdown content (tables, links, paragraphs) as-is

The frontmatter should be:

```yaml
---
title: "Tuberculosis Detector with TFLite"
date: 2019-03-08
tags: ["ml"]
description: "Building a tuberculosis detector with a modified VGG-16 model and deploying it on Android with TFLite at the PanIIT hackathon."
---
```

Every image reference like `![alt](assets/img/posts/tflite_1/image4.png)` becomes `![alt](/images/posts/tflite_1/image4.png)`.

Every `<h2>Title</h2>` becomes `## Title`.

Every `<i>text</i>` becomes `*text*`.

- [ ] **Step 3: Remove the sample placeholder posts**

Delete the three sample posts created in Task 2 — the migrated real post replaces them.

```bash
rm src/content/blog/building-a-container-snapshotter.md
rm src/content/blog/why-we-chose-django.md
rm src/content/blog/redis-streams-consumer-groups.md
```

- [ ] **Step 4: Create `.github/workflows/deploy.yml`**

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [master]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout your repository using git
        uses: actions/checkout@v4
      - name: Install, build, and upload your site
        uses: withastro/action@v3

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

- [ ] **Step 5: Remove all Jekyll files**

```bash
rm -f .travis.yml Gemfile Gemfile.lock _config.yml type-on-strap.gemspec index.html
rm -rf _layouts _includes _sass _posts _drafts _portfolio pages
rm -rf assets
rm -rf _site .jekyll-cache
```

Keep these files: `CLAUDE.md`, `.gitignore`, `LICENSE`, `README.md`, `docs/`, `src/`, `public/`, `astro.config.mjs`, `package.json`, `package-lock.json`, `tsconfig.json`, `.nvmrc`, `.github/`.

- [ ] **Step 6: Update `.gitignore`**

Ensure `.gitignore` includes Astro-specific entries:

```
node_modules/
dist/
.astro/
.DS_Store
```

- [ ] **Step 7: Verify full build**

```bash
npm run build
```

Verify the build succeeds and the `dist/` directory contains:
- `index.html` (homepage)
- `about/index.html`
- `404.html`
- `blog/hackathon-with-tf-lite/index.html`
- `favicon.ico`
- `images/posts/tflite_1/` (all images)

- [ ] **Step 8: Verify locally**

```bash
npm run preview
```

Open `http://localhost:4321`. Walk through every page:
- Homepage: intro, tag filter (only "ml" tag now), one post listed
- Click the post: full reading experience with images rendering from `/images/posts/tflite_1/`
- About page: bio, social links
- Navigate to a nonexistent URL: 404 page
- Toggle dark mode on each page
- Resize to mobile width on each page

- [ ] **Step 9: Commit**

```bash
git add -A
git commit -m "feat: migrate content, add GitHub Actions deployment, remove Jekyll"
```

- [ ] **Step 10: Merge to master and deploy**

When satisfied everything works:

```bash
git checkout master
git merge redesign
git push origin master
```

The GitHub Actions workflow will trigger automatically. Go to the repository's Settings > Pages and ensure the source is set to "GitHub Actions" (not "Deploy from a branch").

Verify the live site at `https://singh-saurabh.github.io` once the action completes.

- [ ] **Step 11: Clean up branch**

```bash
git branch -d redesign
```
