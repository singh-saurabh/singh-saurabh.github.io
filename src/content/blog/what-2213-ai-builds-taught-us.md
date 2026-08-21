---
title: "What 2,213 AI Builds Taught Us About Code Generation"
date: 2026-08-21
tags: ["ai", "systems", "infrastructure"]
description: "We analyzed every chat thread on our AI code-gen platform and discovered that most failures aren't reasoning failures — they're variance in hand-written boilerplate. Here's what we did about it."
---

[Whip](https://whip.run) is a mobile platform that turns natural language into mini-apps. You describe what you want, an AI agent writes React/TypeScript, the platform builds it with esbuild and Tailwind, and you get a working app on your phone. It mostly works. When it doesn't, we wanted to know why.

We exported every chat thread from the platform — 2,213 of them — and classified what was actually going wrong. The answer surprised us.

## Getting the data right

Each thread is a JSON object: a title, an app ID, and an array of turns with a role (`user` or `agent`), a failure flag, and a message. Simple enough. 21,633 total turns across the 2,213 threads.

The first thing we noticed: 1,876 of the 11,025 "user" turns weren't user messages at all. They were `BUNDLE FAILED:` messages injected by the build healing loop — the system feeding build errors back to the agent for self-repair. Nearly 17% of what looked like user conversation was actually the platform talking to itself. That contamination had masked the real failure rate for months. Every analysis we'd done by counting "user turns" had been wrong.

After filtering, we classified failures into three clusters: scaffold/toolchain defects, agent behavior defects, and library/whitelist issues. The first cluster was the largest, and the most surprising.

## The counterintuitive finding

Most people assume AI code-generation failures are reasoning failures — the model misunderstands the task, picks the wrong algorithm, can't hold the architecture in context. Some of that is real. But it wasn't the primary failure mode.

The primary failure mode was **variance in hand-written boilerplate**.

Our agent writes a `cn()` utility function on every single generation — a three-line helper that composes `clsx` and `tailwind-merge` for conditional class names. It's the same function every time. But "the same" is the problem: each generation wrote it slightly differently. Sometimes `import { clsx } from 'clsx'`, sometimes `import clsx from 'clsx'`. Sometimes the file was `utils.ts`, sometimes `lib/utils.ts`. Sometimes it imported `tailwind-merge`, sometimes `tw-merge` (which doesn't exist).

These weren't reasoning failures. The model knew what `cn()` was supposed to do. It was failing at being a photocopier. And `cn()` resolution errors — `TS2307: Cannot find module` — were our **number one build-failure class**.

The same pattern showed up everywhere we looked:

**Bridge type collisions.** `Bridge` is the global API that mini-apps use to talk to the native app — data persistence, AI chat, networking, leaderboards. We had its TypeScript types declared in three places: the main system prompt, the `ai-chat` skill, and the `game-development` skill. Each declared a partial interface. An AI game with a leaderboard and chat would load all three and hit `TS2451: Cannot redeclare block-scoped variable 'Bridge'`. The agent was getting type errors from our own contradictory instructions.

**Blank screens.** Generated apps would render to a 0px-height container because there was no CSS reset for `html`, `body`, or `#root`. The model would sometimes include one and sometimes not. When it didn't, users saw a black screen and assumed the app was broken.

**Silent runtime crashes.** When a generated app threw an uncaught error — a null reference, a failed API call — there was no error boundary. The screen went white or black. No message, no stack trace, no way for the user to report what went wrong. No way for the agent to learn what happened on retry.

**Unstyled shadcn components.** The agent frequently used shadcn-style utility classes like `bg-background` and `text-foreground`, which map to CSS custom properties via `hsl(var(--background))`. But it only sometimes remembered to declare those custom properties. When it didn't, the app shipped with invisible text on a transparent background.

Every one of these is a case where the model knows the right answer — it writes it correctly most of the time — but per-generation variance occasionally produces a broken variant. The model was failing at consistency, not competence.

## Measuring blast radius with data, not intuition

When we started fixing these, we made a mistake: we prioritized by how bad each failure *sounded*. Breaking legacy apps felt catastrophic. A subtle type collision felt minor. We were wrong.

We ran a read-only census over `AppVersion.source_files` — the JSON field that stores every generated app's source tree — across all 2,314 production app versions. Instead of reasoning about blast radius, we measured it:

- **The `no_code` guard breakage: 111 versions across 82 apps.** Our scaffold seeding had accidentally made `read_all_files()` always return truthy, which silently disabled the guard that detects when the agent produces no code. This was actively misfiring the day we measured. We'd initially ranked it second priority.
- **Legacy flat-JS app breakage: 0 apps.** The regression we'd called "highest severity" protected an empty set. We kept the guard (it's cheap insurance) but deprioritized the work.
- **Bridge type collisions with existing declarations: 43 apps** out of 2,314 (1.9%). Not nothing, but far less explosive than it sounded.

The lesson: census beats intuition. Run the query before you prioritize the fix.

## The fix: stop letting the agent write boilerplate

The core insight is simple: **when an AI agent fails repeatedly at the same boilerplate, the fix isn't a better prompt — it's removing the boilerplate from the agent's responsibility entirely.**

We split the fix across two ownership boundaries.

### Platform-seeded scaffold (agent doesn't write it, agent doesn't touch it)

Five changes, two mechanisms:

**Seeded at job start** (the agent can read these files, but the platform wrote them):

1. **`src/utils.ts`** — the canonical `cn()` implementation, seeded once. The bundler force-installs `clsx` and `tailwind-merge` whenever this file is present, so even if the agent never imports `cn()`, the type checker doesn't fail on an orphaned file. The prompt changed from "write a cn() helper" to "cn() helper (provided — import it, add other helpers elsewhere)."

2. **`src/bridge.d.ts`** — a 190-line type declaration covering all four Bridge surfaces (`data`, `chat`, `net`, `leaderboard`), tagged with a `// @whip-bridge-types v1` marker. On refine, the platform recognizes its own marker and refreshes the file to the current version. If an app already has its own Bridge declaration (43 apps did), seeding is skipped. A new `add_bridge_types` agent tool lets the agent install the platform types when it hits a `TS2304: Cannot find name 'Bridge'` error, instead of hand-writing a declaration to work around it.

**Injected by the bundler** (the agent never sees these, they're added at build time):

3. **Mobile shell CSS** — `:where(html, body, #root) { height: 100%; margin: 0; } :where(#root) { min-height: 100dvh; }`. Wrapped in `:where()` for zero specificity, so any agent CSS targeting the same elements wins naturally. `100dvh` instead of `100vh` handles the mobile toolbar correctly. No more 0px-height blank screens.

4. **Error boundary** — two layers, because ES module imports hoist. A vanilla JS global handler runs in a `<script>` tag *before* the bundle, catching load-time `SyntaxError`s and module-scope throws that a React boundary inside the bundle would never see. A React `ErrorBoundary` component wraps the app root for render-time errors. Both show a dismissible overlay with the error message and a collapsible stack trace. Both are suppressed under `__WHIP_PREVIEW_HARNESS` so screenshot pipelines don't capture error UI. Benign errors (`ResizeObserver loop`, `AbortError`, `NotAllowedError`) are filtered out.

5. **shadcn tokens** — a Tailwind preset mapping shadcn's color utility classes to `hsl(var(--...))` values, plus default `:root` and `.dark` custom properties. The agent's `tailwind.config.js` is treated as a preset under a platform wrapper, so the agent's `theme.extend` overrides our defaults while our content globs and plugin detection stay correct.

### The Tailwind config wars

That fifth change — shadcn tokens — went through three iterations before it worked, and the journey is worth telling because it illustrates a class of problems specific to AI code generation: **you can't text-splice agent-authored config files with regexes**.

Our initial approach was straightforward: read the agent's `tailwind.config.js`, regex-match the `presets` array (or create one), and inject our shadcn preset. This failed in at least four ways:

- Agents sometimes used `module.exports =`, sometimes `export default`. The regex handled one.
- Agents sometimes stored the config in a named const (`const config = { ... }; module.exports = config`). The regex couldn't follow the indirection.
- Our content glob injection would sometimes land inside a string literal or comment, corrupting the file.
- When the agent already had a `presets` key, we'd create a duplicate, and Tailwind silently took only the last one.

We then built a hand-rolled JavaScript comment stripper to make the regex matching more reliable. Our code reviewer found six of nine code-level issues traced back to "the platform parsed and rewrote agent-authored JavaScript with regexes over a hand-rolled lexer." The fix was to delete the capability entirely. One commit was 257 deletions against 46 insertions.

The final design doesn't parse the agent's config at all. It renames it to `whip-app-tailwind.config.js` and writes a wrapper that `require()`s it as a preset:

```javascript
const __whipApp = require('./whip-app-tailwind.config.js');
module.exports = {
  content: ["./index.html", "./src/**/*.{ts,tsx}"],
  presets: [
    require('./whip-shadcn-preset.js'),
    (__whipApp && __whipApp.default) || __whipApp,
  ],
};
```

The `(__whipApp && __whipApp.default) || __whipApp` unwrap handles both `module.exports` and `export default` configs. The agent's theme extensions still override ours (later preset wins in Tailwind). We never touch the agent's file.

The shadcn CSS tokens had their own bug: we initially wrapped them in `@layer base`, thinking this would give them lower precedence so the agent's `:root` overrides would win. The opposite happened — Tailwind hoists `@layer base` rules to the position of `@tailwind base`, which in our output came *after* the agent's unlayered `:root` block. Layer'd rules beat unlayer'd rules regardless of source order. Our "defaults" were overriding the agent's palette. The fix was to emit the tokens as plain unlayered CSS *ahead* of `@tailwind base`, so normal source-order precedence applies.

## The behavior layer

Not every failure was a toolchain problem. The second cluster was about the agent saying and doing things that were wrong at a higher level.

**55–60% of cross-user leaderboard requests got a fake.** When a user asked for "a leaderboard that shows all players," the agent would build a hand-rolled ranking using `Bridge.data` — our per-instance key-value store. But `Bridge.data` is scoped to a single app instance. It fundamentally cannot aggregate scores across all players. The real `Bridge.leaderboard` API does this, but its types were only documented in the `game-development` skill, which the agent didn't load for apps it didn't classify as games. A step-count tracker with a friend ranking? Not a game. No skill loaded. Fake leaderboard built.

The fix was twofold: widen the skill's routing ("load for ANY app that needs an all-players ranking — whether or not the app is a game") and add an explicit rule to the system prompt: "Do NOT hand-roll one with `Bridge.data`: that data is scoped to a single app instance."

**~500 threads showed the agent overclaiming.** "Fully functional," "production-ready," "everything you need" — after fixing a build error. The agent would escalate confidence on retries: "I'm absolutely sure this time." We added four rules: describe what the user can *do* right now, not what you "architected"; only claim behavior you have reason to believe works; on retries, lower your confidence and say what changed; don't default to "fully functional."

**67% of explicit "simple/minimal" requests got maximalist output.** Our creative-direction prompt literally contained the line: `Not "minimal" or "modern" — those are meaningless.` The agent was following instructions — ours, not the user's. We replaced it with a scope read: utility/tool apps get restraint, expressive apps get the full creative brief, and if the user says "simple," that's a hard constraint.

## The ambient declaration ordering gotcha

One fix we *didn't* make is worth mentioning.

Our code reviewer suggested making `src/bridge.d.ts` unconditionally platform-owned — always overwrite it, even if the app has its own declarations. Cleaner, simpler, no collision logic needed.

We tested it. TypeScript resolves precedence between two ambient `declare const Bridge` declarations by filename alphabetical order. Our platform file at `src/bridge.d.ts` would lose to an app's `src/Bridge.d.ts` (capital B), and win against `src/types.d.ts`. The same feature would silently break or silently work depending on what the agent named its file.

That's worse than a loud type error. We kept the marker-based approach: seed our file with a version marker, refresh it on refine if the marker matches, skip if the app has its own declarations, and give the agent a tool to install ours explicitly when it hits a type error.

## Results

We ran 6 prompts from previously-failing classes against 3 workers on `gemini-3.1-pro-preview`. All 6 completed with `error_code=null`. Zero `clsx`/`cn` resolution errors — the former number one build-failure class.

The Langfuse traces showed 6/6 traces with 0 ERROR-level observations. Residual healing turns were limited to failure classes this work didn't target (a `NodeJS.Timeout` type reference the LSP catches, a browserslist environment warning), and all self-healed.

## What we'd do differently

**Instrument failure categories from day one.** We relied on manual thread reading for months before doing the corpus analysis. The `BUNDLE FAILED` contamination — 17% of "user" turns being system messages — meant every metric we had was wrong. If we'd tagged turns with a `source` field (user vs. system vs. agent) from the start, we'd have caught the boilerplate-variance pattern much earlier.

**Census before you prioritize.** Reasoning about blast radius led us to rank fixes in the wrong order. A single SQL query over `source_files` gave us exact counts. The thing that sounded scariest (breaking legacy apps) affected zero apps. The thing that sounded minor (the `no_code` guard) was silently misfiring on 82 apps.

**Don't parse agent output with regexes.** This seems obvious in retrospect, but when you need to inject one key into a JSON-like config file, a regex feels proportionate. It isn't. Agent-authored code has enough syntactic variance that any regex will eventually hit a shape it doesn't handle. The wrapper/composition pattern — treat the agent's file as an opaque input to your own file — is more robust and usually simpler.

The general principle extends beyond Tailwind configs: when your platform and your AI agent share responsibility for a file or a concept, draw the ownership boundary so that the platform's contribution doesn't depend on parsing the agent's. Seed don't splice. Compose don't rewrite. And when the agent fails at the same boilerplate for the twentieth time, take the boilerplate away.
