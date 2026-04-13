# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
nvm use                   # Load Node 22 (required first)
npm install               # Install dependencies
npm run dev               # Dev server at http://localhost:3030 (hot-reloads slides.md and style.css)
npm run build             # Generate static site to dist/
npm run export            # Export PDF (requires dev server already running on port 3030)
node screenshot-slides.js # Screenshot all slides to screenshots/ (requires dev server)
node screenshot-step.js   # Screenshot specific slides at mid-click state
```

For Playwright scripts outside an interactive shell (e.g. CI or scripts), use the explicit nvm path:
```bash
env -i HOME="$HOME" PATH="$HOME/.nvm/versions/node/v22.16.0/bin:/usr/local/bin:/usr/bin:/bin" \
  node screenshot-slides.js
```

## Architecture

This is a [Slidev](https://sli.dev/) presentation for a 2-hour hands-on Python workshop with music-themed exercises. All slide content lives in `slides.md` (single file, `---`-separated slides). Styles are in `style.css` (Atom One Light palette; Bricolage Grotesque / Figtree / Fira Code fonts).

### Custom Vue Components (`components/`)

Slidev auto-imports everything in `components/`. Key components:

- **`CodeBlock.vue`** — the primary way code is displayed. Wraps Shiki for syntax highlighting with clickable line selection and filename tabs. Do **not** use Slidev's built-in fenced code blocks for code slides.
- **`ExerciseTimer.vue`** — 3-minute countdown with ambient music and a "ding" on completion.
- **`FStringViz.vue`, `LoopViz.vue`, `RangeViz.vue`, `EnumerateViz.vue`, `CondViz.vue`, `LoopCondViz.vue`** — step-by-step animated visualizations for each Python concept taught.
- **`CoverScroller.vue`** — live clock + Python snippet on the cover slide.
- **`FileHint.vue`** — badge linking to an exercise file.

### Critical Authoring Rule

**Never leave blank lines inside HTML blocks in `slides.md`.** CommonMark terminates an HTML block on the first blank line, which breaks component rendering. Keep all props and content on adjacent lines within any HTML tag.

Code inside components should live in `<script setup>` template literals, not as separate files.
