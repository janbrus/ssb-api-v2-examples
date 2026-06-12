# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This is **not** an application codebase. It is a Claude Code **Skill** package that teaches Claude how to query Statistics Norway (SSB) via the public PxWebApi v2 at `https://data.ssb.no/api/pxwebapi/v2`. There is no build, no tests, no runtime — changes here are documentation/prompt edits that get loaded when the `ssb-pxwebapi-v2` skill triggers.

## Layout

- `SKILL.md` — the skill entrypoint. Frontmatter (`name`, `description`) controls when the skill auto-triggers; the body is the operational guide Claude follows. Keep triggers (Norwegian + English keywords for Norwegian official statistics) in the `description`.
- `references/` — deeper reference material loaded on demand:
  - `json-stat2.md` — json-stat2 Dataset structure, row-major indexing, status codes, `extension` semantics at both dataset and dimension level (vendor-neutral)
  - `api-details.md` — SSB-specific operational info (publishing times, limits, license)
  - `codelists-and-filters.md` — codelist/filter syntax: `codelist[Var]=agg_…`, `outputValues[Var]=aggregated`, `top(n)`/`from(x)`/`range(a,b)`, wildcards. Note prefixes: `agg_KommFylker` uses `F-`, `agg_KommSummer` uses `K-`. Includes KPI/COICOP groupings (`vs_CoiCop2018Kpi01`, `agg_CoiCop2018Kpi011`).
  - `search-syntax.md` — Lucene query parser syntax for `/tables?query=` (PxWebApi uses Lucene.Net under the hood)
  - `klass-vardok.md` — pointers to SSB's Klass (classifications) and VarDok (variable definitions) via URNs in `link.describedby`
  - `output-formats.md` — `json-stat2`, `csv`, `xlsx`, `html`, `px` and parameters (`UseCodesAndTexts`, `IncludeTitle`, `heading`/`stub` pivoting)
  - `common-tables.md` — well-known table IDs (KPI, befolkning, etc.)
  - `troubleshooting.md` — common errors and fixes
  - `mcp-tools.md` — mapping between `pxweb-mcp` MCP tools (`@jarib/pxweb-mcp`) and API endpoints, plus limitations (lossy `search_tables`, no `codelist[Var]` in `fetch_metadata`, no `outputFormatParams`) and the `--url` config caveat. Facts are version-bound — re-verify against the package source when it ships a new major version.

## Editing guidance

- Preserve the bilingual (Norwegian primary, English secondary) trigger surface in `SKILL.md` frontmatter — removing keywords will cause the skill to stop firing for real user queries.
- The API base URL and endpoint table in `SKILL.md` are the canonical contract; if an endpoint is added/changed upstream, update `SKILL.md` first, then cross-check `references/api-details.md`.
- Examples should use real, currently-published SSB table IDs. When adding an example, verify the table still exists via `GET /tables/{id}` before committing — dead IDs mislead future sessions.
- For example URLs/queries, hit the live API and confirm HTTP 200. Codelist prefixes are easy to forget: `agg_KommFylker` requires `F-` codes, `agg_KommSummer` requires `K-` codes (and usually `outputValues[Region]=aggregated`). A 400 response usually means a missing prefix or a missing required variable (`Tid`/`ContentsCode` are never eliminable).
- Use `valueCodes` (camelCase) consistently in GET examples — matches the OpenAPI spec and the POST body shape, even though the API is case-insensitive.
- Keep `references/*.md` focused; `SKILL.md` should stay the overview and defer detail to references rather than duplicating it.
- On every content change: bump `metadata.version` in `SKILL.md` frontmatter (semver: PATCH for fact fixes, MINOR for new content), add a `CHANGELOG.md` entry, and rebuild the zip with `scripts/build_zip.sh` — the zip-sync CI job fails otherwise. A version that isn't bumped lies about copies being current.

## Related sibling skill

A parallel `scb-pxwebapi-v2` skill exists for Sweden's SCB. The two APIs share the PxWebApi v2 shape — when fixing a bug in one, check whether the other needs the same fix.

A third-party `norges-bank-api` skill (github.com/avocodetoast/norges-bank-api-skill, SDMX — not PxWebApi) covers Norges Bank data. SKILL.md Steg 1 routes central-bank questions there; Steg 5 and Fallgruver require answers to comment only on the fetched SSB numbers — never fetch and blend in data from other sources. Keep these references when editing.
