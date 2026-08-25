# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository overview

This repository contains a single self-contained educational web page, [linear_functions.html](linear_functions.html), that teaches linear functions (`y = mx + b`). There is no build system, package manager, test suite, or server — it is a static HTML file with inline CSS and JavaScript, plus a single CDN dependency on Chart.js (`chart.js@4.4.4`).

## Development workflow

There is no build/lint/test tooling. To work on this file:

- Edit [linear_functions.html](linear_functions.html) directly.
- Open it directly in a browser to preview changes (double-click the file, or use a simple local server if `file://` CDN/script loading is restricted).

## Architecture

Everything lives in one file, structured top-to-bottom as:

1. **`<style>`** — all CSS, including a light theming layer via `:root` custom properties (`--blue`, `--purple`, etc.) and RTL-specific overrides scoped under `[dir="rtl"]`.
2. **Static markup** — header/intro, `#section1` (static worked examples) and `#section2` (interactive playground), plus a fixed-position language toggle button.
3. **`<script>`** — all behavior, in this order:
   - `translations` object (`en` / `he`) — every user-facing string lives here, including function-valued strings (`slopeDesc`, `interceptDesc`) that interpolate current `m`/`b`. `applyLanguage(lang)` is the single function that re-renders all text and chart labels from this table; there is no other source of UI copy.
   - `examples` — the four hardcoded example lines rendered in Section 1, each computed into a table (`computeTable`/`buildTableHTML`) and a Chart.js scatter+line chart.
   - `attachTableHighlight(tableEl, getChart, data)` — shared wiring so hovering/clicking a table cell highlights the matching point on that table's chart. Used by both the Section 1 example cards and the Section 2 interactive chart.
   - Interactive section (`renderInteractive`, the `#okBtn` handler) — reads `m`/`b` from the number inputs, validates them, and rebuilds the table + destroys/recreates the Chart.js instance.
   - Language switching (`applyLanguage`, `#langToggleBtn` handler) — toggles `document.documentElement.dir` for RTL support and re-applies all translated strings and chart legend labels.

Key invariants to preserve when editing:

- `X_AXIS_RANGE` / `Y_AXIS_RANGE` are shared fixed bounds so example charts and the interactive chart stay visually comparable regardless of slope — don't let per-chart data auto-scale the axes.
- Every user-visible string must be added to **both** locales in `translations`, and wired into `applyLanguage` if it's not already covered by an existing chart/table re-render path.
- RTL support is driven purely by `document.documentElement.dir`; table elements force `dir="ltr"` since coordinate tables should stay left-to-right even in the Hebrew layout.
