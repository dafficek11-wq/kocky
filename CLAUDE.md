# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Tamagotchi-style cat game (Czech UI). The **entire app is one self-contained file: `index.html`** — HTML, CSS, and vanilla JS in a single `<script>` IIFE. No build step, no dependencies, no tests, no framework.

## Running / developing

- **Run:** open `index.html` directly in a browser (double-click). There is no dev server, package manager, or build.
- A local preview server cannot be started in this environment — Python, Node, and npm are all unavailable. Verify changes by reading the code and reasoning about it, then ask the user to confirm visually in their browser.

## Deployment

Hosted on **GitHub Pages**: repo `dafficek11-wq/kocky`, live at `https://dafficek11-wq.github.io/kocky/` (source: `main` branch, root). To update the live site, commit and `git push` — Pages rebuilds automatically.

`gh` CLI lives at `C:\Program Files\GitHub CLI\gh.exe` (not on PATH; call by full path) and is authenticated as `dafficek11-wq`.

## Architecture

Everything lives in the `<script>` IIFE at the bottom of `index.html`:

- **`state`** — the single source of truth (name, the four stats, `sleeping`, `appearance`, `lastTick`). Persisted to `localStorage` under key `kocky_save_v1`.
- **Game loop** — `setInterval(tick, 1000)` calls `applyDecay(1)` → `render()` → `save()`. Stats decay over time (rates in `DECAY` / `DECAY_SLEEP`); on load, elapsed offline time is applied via `applyDecay(elapsedSec)`.
- **`render()`** — the only function that writes game state to the DOM (stat bars, mood text, sleep classes). Also calls `applyAppearance()`.
- **Action functions** (`feed`, `play`, `pet`, `clean`, `toggleSleep`) mutate `state`, trigger an animation/effect, then call `after()` (= `render()` + `save()`).

### The cat is pure CSS

The cat in `.cat` is built entirely from positioned `<div>`s (body, belly, paws, tail, head with ears/eyes/nose/whiskers). Animations and expressions are CSS keyframes + state classes (`blink`, `asleep`, `sad`, `jump`, `wiggle`).

### Appearance customization (the main feature)

Customization options are defined as JS objects near the top of the IIFE: `PALETTES`, `EYES`, `PATTERNS`, `EARS`, `TAILS`, `POSES`. The user's choices live in `state.appearance`.

Two functions connect data → DOM:
- **`applyAppearance()`** translates `state.appearance` into the cat: fur colors via CSS variables (`--cat`, `--cat-dark`, `--cat-light`), eye color via `--eye`, and everything else via toggled classes on `.cat` (`tabby`, `rainbow-fur`, `ears-round`, `tail-fluffy`/`tail-short`, `lying`).
- **`buildLookPanel()`** renders the clickable option buttons in the side panel from those objects; clicking writes to `state.appearance`, then `applyAppearance()` + `save()`.

The appearance panel is a persistent `<aside class="card look-card">` shown beside the cat in a responsive flex `.layout` (wraps below on narrow screens).

**To add a new appearance option**, touch these consistently:
1. Add the entry to the relevant options object (`PALETTES`/`EYES`/etc.).
2. Add the CSS (a new palette is data-only; ears/tail/pattern/pose variants need a `.cat.<class> ...` rule).
3. Apply it in `applyAppearance()` (set a CSS var or toggle a class).
4. If it's a new *category* (not a new value in an existing one): add an HTML container with an `id` inside `.look-card`, and a `buildOptRow(...)` call in `buildLookPanel()`.
5. Add the default key in **three places** so old/new saves stay consistent: the initial `state.appearance` literal, the `keepLook` default in `newGame()`, and the normalization `Object.assign` in `init()` (this last one backfills keys missing from older saved games).

Special case: rainbow fur uses a gradient instead of a flat color — its palette entry has `rainbow: true` (toggles `.rainbow-fur`) and a `swatch` gradient string for the picker dot.

## Working with the user

The user is **not technical**. Do not ask them technical questions (tooling, git, config, implementation choices, etc.) — make reasonable decisions yourself and just do the work. When you explain what you did, use plain, non-technical language. Save technical detail for code comments and this file, not for messages to the user.

## Conventions

- All user-facing strings are **Czech**. Keep new UI text in Czech.
- Keep it a single dependency-free file — don't introduce a build, bundler, or external libraries.
