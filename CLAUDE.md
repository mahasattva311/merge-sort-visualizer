# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A standalone, single-file sorting-algorithm explainer and visualizer (`index.html`, Korean UI). Covers bubble, selection, insertion, shell, merge, quick and heap sort. No build tools, dependencies, or package manager — open directly in a browser. `index.html#quick` (etc.) deep-links to an algorithm.

## Running

```
open index.html        # macOS
# or open with any browser
```

## Architecture

Everything lives in `index.html` (CSS in `<style>`, markup, then one IIFE script):

- **Record, then replay.** `record(algo, data)` runs an algorithm to completion and stores a snapshot per step (`arr`, `sorted`, highlight sets `hl.{cmp,act,axc}`, `line`, `msg`, `tags`, `range`, `pivot`, `aux`, running `cmp`/`swp` counters). The player only moves `state.pos` through `state.steps`, so play/pause/step-back/scrubber are trivial.
- **`ALGOS`** — one entry per algorithm: `run(c)` (uses the recorder context `c`: `c.compare`, `c.count`, `c.swap`, `c.write`, `c.note`, `c.mark`, `c.range`, `c.pivot`, `c.aux`), complexity fields, descriptive text (`summary`, `how`, `pros`, `cons`, `use`, `note`) and pseudocode `code[]`. The `line` passed to each recorder call is the index into `code[]` that gets highlighted. Flags `usesAux` / `usesPivot` toggle the aux-array row and pivot legend.
- **Rendering:** `buildStage()` creates bar/tag/aux DOM once per data or algorithm change; `render()` only updates heights and classes for the current step (so CSS transitions animate). `renderMeasure()` records all algorithms on the current data for the comparison chart.
- **Card mode** (`mode === 'cards'`, max `MAX_CARDS` = 20): `record()` also tracks card identity (`px`, `held` per snapshot) so `renderCards()` can move each number card to its new slot along an arc (Web Animations API, duration `state.mdur`); cards held out of the array (insertion key, merge temp) are lifted. Playback rate is capped by `curRate()` so a move finishes before the next step.
- Theme is CSS variables on `:root` with a `prefers-color-scheme: dark` override.
- `window.__sorting` exposes `{ALGOS, record}` for testing: every algorithm's final snapshot should equal the sorted input.

## Adding an algorithm

Add an entry to `ALGOS` with `run(c)` that only mutates the array through the `c.*` helpers (so every change is recorded). Tabs, info panel, comparison table and measurements are generated from the entry automatically.
