# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev       # local dev server (Astro on localhost:4321)
npm run build     # production build → dist/
npm run preview   # preview the production build locally
```

No test suite or linter is configured.

## Architecture

This is an **Astro v5** personal portfolio site for artist Xiuyuan Shen. Deployed to Cloudflare Pages via `git push origin main` (auto-builds from GitHub). Live at [shenxiuyuan.xyz](https://shenxiuyuan.xyz) (also: scallop-gdu.pages.dev).

### Layout system

Four layouts in `src/layouts/`:

| Layout | Used for | Key traits |
|---|---|---|
| `RawLayout.astro` | Index/listing pages, general pages | Full site header + nav, Space Mono font, dark/paper theme system |
| `DetailLayout.astro` | Individual work/note detail pages | Minimal header (← back link only), loads DiatypeVF variable font, `body.detail` class scopes editorial typography |
| `SoundEntry.astro` | Individual sound entry pages | Wraps `RawLayout`, renders embedded player from frontmatter |
| `Bare.astro` | Utility/dev pages | No theme system, system fonts, plain white |

All global CSS lives inside the layout files as `<style is:global>`. `RawLayout.astro` is the canonical source of the design token CSS variables.

### Theme system

Two themes toggled via `localStorage` and `data-theme` on `<html>`:
- `dark` (default): `#050505` bg, `#d4d4d4` fg, `#00ff41` accent (green)
- `paper`: `#f6f3ea` bg, `#111` fg, `#1a49ff` accent (blue)

CSS variables: `--bg`, `--fg`, `--muted`, `--border`, `--accent`, `--code-bg`, `--container-w`, `--font-body`.

### Content sections and frontmatter

**`/works`** — `src/pages/works/*.md`, sorted by `date` descending. Grid layout with thumbnail.
```yaml
layout: ../../layouts/DetailLayout.astro
title: "Work Title"
date: "2025-01-11"
type: "Hybrid Media Video"
thumbnail: "/images/filename.png"
description: "Short description shown on card."
```

**`/sounds`** — `src/pages/sounds/*.md`. Index uses `Astro.glob` + expandable `<details>` player. Individual pages use `SoundEntry.astro`.
```yaml
layout: ../../layouts/SoundEntry.astro  # or omit for index-only entry
title: "Sound Title"
year: 2024                              # or use date: "2024-01-01"
summary: "One-line description"
embedSrc: "https://bandcamp.com/EmbeddedPlayer/..."
embedHeight: 140                        # default: 140
external: "https://..."                 # optional: link shown as "Open"
readMore: "/notes/related-note"         # optional: link to notes page
order: 10                               # optional: tie-break within same year
```

**`/notes`** — `src/pages/notes/*.md`, sorted by `date` descending. Each note uses `RawLayout` directly via frontmatter `layout`.
```yaml
layout: ../../layouts/RawLayout.astro
title: "Note Title"
date: "2025-01-01"
```

**Draft content** lives in `src/notes-drafts/` (outside `pages/`, so never built as routes).

### Special homepage wordmark

`src/pages/index.astro` uses `layoutVariant="home"` on `RawLayout`, which activates the `glitchmark` class. The wordmark renders as individual `<span class="glyph ...">` elements animated with CSS keyframes (`fungal-mutant`, `fungal-jitter`) using the FungalVF variable font axes `'THCK'` and `'grow'`. The `infected` class marks letters (`a`, `l1`, `l2`) that animate with more extreme variation. All animation logic is in `src/pages/index.astro` as a `<style is:global>` block.

### Fonts

- **Space Mono** — Google Fonts, main body/UI font
- **FungalVF** (`/public/fonts/fungal/FungalVF.woff2`) — variable font used only for the homepage wordmark glitch
- **DiatypeVF** (`/public/fonts/diatype/ABCDiatypeVariable.woff2`) — variable font loaded only by `DetailLayout`, used for reading-optimized body text

### Static assets

Images go in `public/images/` and are referenced in Markdown as `/images/filename.jpg`. Markdown code blocks are syntax-highlighted by Shiki (configured in `astro.config.mjs` with `github-light` / `github-dark-dimmed` themes).
