# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This is **not** an application codebase. It is a Claude Code **Skill** package that teaches Claude how to query Statistics Sweden (SCB) via the public PxWebApi v2 at `https://statistikdatabasen.scb.se/api/v2`. There is no build, no tests, no runtime — changes here are documentation/prompt edits that get loaded when the `scb-pxwebapi-v2` skill triggers.

## Layout

- `SKILL.md` — the skill entrypoint. Frontmatter (`name`, `description`) controls when the skill auto-triggers; the body is the operational guide Claude follows. Keep triggers (Swedish + English keywords for Swedish official statistics) in the `description`. The frontmatter `name` must match the folder name (`scb-pxwebapi-v2`).
- `references/` — deeper reference material loaded on demand:
  - `json-stat2.md` — json-stat2 Dataset structure, row-major indexing, status codes, role analysis principle (vendor-neutral)

The SCB skill is intentionally sparser than its sibling `ssb-pxwebapi-v2` — operational details (codelist syntax, filter expressions, common tables, troubleshooting) live inline in `SKILL.md` rather than in separate reference files.

## Editing guidance

- Preserve the bilingual (Swedish primary, English secondary) trigger surface in `SKILL.md` frontmatter — removing keywords will cause the skill to stop firing for real user queries.
- The API base URL and endpoint table in `SKILL.md` are the canonical contract; if an endpoint is added/changed upstream, update `SKILL.md`.
- Examples should use real, currently-published SCB table IDs. SCB tables use mixed naming (numeric `TAB638`, sometimes named like `BefolkningNy`) and tables are renamed/retired more often than IDs suggest. When adding an example, hit the live API and confirm HTTP 200 — a 404 "Non-existent table" means the ID has changed.
- For example URLs/queries, hit the live API and confirm HTTP 200. Common 400 causes: missing required variable (`Tid`/`ContentsCode` are never eliminable), wrong codelist prefix, wrong time format for the table's `timeUnit`.
- Use `valueCodes` (camelCase) consistently in GET examples — matches the OpenAPI spec and the POST body shape, even though the API is case-insensitive.
- Region codes for Sweden are 4-digit SKR codes (e.g. `0180` Stockholm); first two digits are the län code (`01*` = all Stockholm-län municipalities). The "Hela riket" total is typically `00`.
- The **Dataintegritet — grundregeln** section at the top of `SKILL.md` outranks everything else in the skill: never state a number not fetched from the API in that conversation, never blend other sources, never interpolate. Don't weaken, shorten or relocate it — the rest of the workflow assumes it. It mirrors the same section in `ssb-pxwebapi-v2` (Norwegian), `generic-pxweb-v2-skill` and `generic-pxweb-v1-skill` (English); a change to the rule belongs in all four.
- Answers must comment only on the numbers fetched in the query — never fetch and blend in data from other sources (Riksbanken, Eurostat, web search); refer the user onward instead. The rule lives in Steg 5 and Fallgropar — keep it when editing (mirrors the same rule in `ssb-pxwebapi-v2`).
- On every content change: bump `metadata.version` in `SKILL.md` frontmatter (semver; stay below 1.0.0 while the skill is marked BETA) and add a `CHANGELOG.md` entry.

## Related sibling skill

A parallel `ssb-pxwebapi-v2` skill exists for Norway's SSB. The two APIs share the PxWebApi v2 shape — when fixing a bug in one, check whether the other needs the same fix. Differences to keep in mind:

- Different base URLs (SCB: `statistikdatabasen.scb.se/api/v2`; SSB: `data.ssb.no/api/pxwebapi/v2`)
- Different default language (SCB: `sv`; SSB: `no`)
- SCB uses 4-digit SKR codes; SSB uses 4-digit kommunekoder + special prefixes (`F-`, `K-`) for aggregation codelists
- SSB has well-documented Klass/VarDok URN systems exposed via `link.describedby`; SCB metadata is sparser
