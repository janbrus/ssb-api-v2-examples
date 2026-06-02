# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This is **not** an application codebase. It is a Claude Code **Skill** package that teaches Claude how to present Statistics Norway (SSB) data in SSB's official visual style — charts, tables, dashboards. There is no build, no tests, no runtime; changes here are documentation/prompt edits that get loaded when the `ssb-chart` skill triggers.

The skill governs **presentation only**. It assumes a JSON-Stat2 response already exists (fetched via the `ssb-pxwebapi-v2` skill) and turns it into a correct, SSB-styled visualization. It deliberately does **not** restate the data format — it points to the data skill instead (see "Editing guidance").

## Layout

- `SKILL.md` — the skill entrypoint. Frontmatter (`name: ssb-chart`, `description`) controls auto-triggering; the body is the operational guide. The body holds the six core principles, a quick-reference palette + typography table, the chart-selection cheat sheet, text/labelling rules (declarative titles, mandatory source line), table formatting, the data-storytelling arc, and the pre-delivery checklist.
- `references/` — deeper material loaded on demand:
  - `chart-selection.md` — per-chart-type decision matrix (line, horizontal bar, grouped/stacked bar, donut, choropleth, scorecard, scatter, histogram, small multiples): when to use, rules, SSB specifics, common mistakes.
  - `color-system.md` — the complete colour spec. Every palette colour in hex/RGB/CSS/JS/Python/openpyxl, plus ready-made theme objects for Recharts, Chart.js, and matplotlib. This is the canonical source for the hex values.
  - `format-guidelines.md` — style rules per delivery format: React/HTML, vanilla JS + Chart.js v4, PowerPoint, Excel, matplotlib, markdown tables. Includes the language-bound number-format rule and the source-footer snippets.
  - `jsonstat-to-chart.md` — the step-by-step recipe from a JSON-Stat2 response to chart datasets: axis selection, `category.index` ordering, series labels, status-codes-as-gaps, per-metric decimals.

## Editing guidance

- **Preserve the bilingual trigger surface** in `SKILL.md` frontmatter. The `description` carries Norwegian (primary) and English trigger phrases; removing them will cause the skill to stop firing. Keep the "Bruk IKKE denne skillen for andre datakilder" boundary — it stops the skill over-triggering on non-SSB visualizations.
- **This is a companion skill, not standalone.** It points to `ssb-pxwebapi-v2`'s `references/json-stat2.md` for the data format (row-major value array, `role.time`/`role.metric`/`role.geo`, `category.index`, `category.unit.decimals`, status codes, `extension.px.contents`). Do **not** duplicate those details here — keep the pointer. If the data skill's format docs move, update the cross-reference rather than inlining the content.
- **Palette hex values are canonical and must stay in sync.** The same colours appear in `SKILL.md`'s quick table, in `references/color-system.md`'s per-colour entries, and in the language-specific arrays/theme objects (JS, Python, Recharts, Chart.js, matplotlib). Change a value in one place → change it everywhere. `color-system.md` is the source of truth; it is based on SSB's official design system / Plotly template.
- **Don't "fix" the line-chart limit.** Line charts cap at **5 series** while the general categorical rule is 6–7. This is intentional (overlapping lines become unreadable faster than separate bars) and is explained in `chart-selection.md` and SKILL.md. Leave the asymmetry; don't normalise it.
- **Statistical-integrity rules are invariants — never soften them.** Bars (and absolute-value lines) start the y-axis at 0; indices may start elsewhere but must annotate the base year. Missing data (status codes `"."`, `".."`, `":"`) renders as visual gaps (`null` + `spanGaps: false`), never interpolated, dropped, or carried-forward. These are load-bearing; edits that weaken them defeat the skill's purpose.
- **Decimals come from the data, never hardcoded.** The recipe reads precision per ContentsCode from `category.unit[code].decimals` with `extension.px.decimals` as fallback. Keep the worked examples (`Personer1` → 0, `KpiIndMnd` → 1, valuta → 4) — they show why a hardcoded "2 decimals" is wrong.
- **Keep SKILL.md the overview.** Defer detail to `references/` rather than duplicating; SKILL.md should stay a navigable summary with pointers.
- **The source line is mandatory and bilingual.** Every visualization shows `Kilde: SSB, tabell {id}. Sist oppdatert: {dato}.` (or the English form), language chosen from the API's `lang` parameter. When combining tables, list every table ID. Keep this consistent across `SKILL.md` and all format snippets in `format-guidelines.md`.

## Deployment note

This skill has two homes: the version-controlled source in the `ssb-api-v2-examples` GitHub repo, and the deployed copy under `~/.claude/skills/ssb-chart-skill`. When you edit one, mirror the change to the other (and re-pack the ZIP if distributing). The two can drift — diff `SKILL.md` before assuming they match.

## Related sibling skills

- **`ssb-pxwebapi-v2`** — the upstream data-fetching skill. This chart skill is its presentation companion; they are designed to be loaded together. Owns the JSON-Stat2 format docs.
- **`scb-pxwebapi-v2`** (Sweden) and **`generic-pxweb-v2-skill`** (any PxWebApi v2 installation) — parallel data skills. There is no SCB/generic equivalent of this chart skill; the SSB palette and "Kilde: SSB"/"Statistics Norway" source line are SSB-specific by design. If a Swedish styling skill is ever added, the structure here is the template, but the colours and source attribution must change.
