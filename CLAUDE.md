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

## equations.html

A bilingual (en/he), 7th-grade equation-solving practice page, following the same single-file/no-build convention as the other pages in this repo.

- `PROBLEM_SETS` — four difficulty categories (`oneStep`/`twoStep`/`bothSides`/`distribution`), each a curated list of problems. Every problem is `{ initial, steps: [{op, distractors?, result}], answer }`; the **last** step in `steps` always fully isolates x (`result` is `"x = <answer>"`).
- Step flow: `renderStepOptions()` shows shuffled multiple-choice buttons (the correct `op` plus distractors — auto-generated via `numericDistractors()` for plain `add/sub/mul/div` steps, hand-curated in the data for `addVar/subVar/distribute` steps) for the current step. A wrong pick (`handleStepPick`) disables only that button so the student can retry; `useStepHint()` eliminates one remaining wrong option, once per step. After the last step is answered correctly, `renderFinalAnswerUI()` swaps in a free-text "what is x?" input, checked by `checkFinalAnswer()`.
- `translations` follows the same `en`/`he` + `applyLanguage(lang)` pattern as [triangle_proofs.html](triangle_proofs.html), including function-valued operation-label translators (`opAdd`, `opSubVar`, etc.) since the multiple-choice button text is data-driven from `op.type`/`op.value`. Equation strings themselves aren't translated — they're rendered in a `.math` element to stay LTR under RTL, same as the other pages. Hebrew op-label strings wrap embedded math tokens (numbers, `x`) in Unicode isolate marks (`⁦…⁩`) to avoid the RTL bidi bugs fixed in triangle_proofs.html's given/prove boxes.
- `applyLanguage` re-renders the equation/step-counter and relabels (not rebuilds) the current step's option buttons via `relabelStepOptions()`, so toggling language mid-problem doesn't reset progress, hints used, or a already-solved final answer.
- Problems are a fixed curated set (not randomly generated); `goToNextProblem()` cycles through the current difficulty tab's list, wrapping around.
