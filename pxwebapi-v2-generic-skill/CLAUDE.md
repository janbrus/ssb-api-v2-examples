# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This is **not** an application codebase. It is a Claude Code **Skill** package that teaches Claude how to query *any* PxWebApi v2 installation. Verified installations are listed in `SKILL.md` under "Known PxWebApi v2 installations" — currently SSB (Norway) and SCB (Sweden); more agencies are expected to migrate. There is no build, no tests, no runtime — changes here are documentation/prompt edits that get loaded when the skill triggers.

Unlike its siblings `ssb-pxwebapi-v2` and `scb-pxwebapi-v2`, this skill is **vendor-neutral**: examples should not assume a specific base URL, table ID, or codelist convention. When agency-specific behavior matters (e.g. SSB's `K-`/`F-` aggregation prefixes, SCB's 4-digit SKR codes), describe it as "varies by installation" rather than hardcoding one agency's rules.

## Layout

- `SKILL.md` — the skill entrypoint. Frontmatter (`name`, `description`) controls when the skill auto-triggers; the body is the operational guide. Keep frontmatter `name` aligned with the folder name (`generic-pxweb-v2-skill`) — Claude Code triggers by folder name in practice, but the frontmatter `name` is used when the skill is packaged and uploaded to claude.ai.
- `references/` — deeper reference material loaded on demand:
  - `json-stat2.md` — json-stat2 Dataset structure, row-major indexing, status codes, role analysis principle (vendor-neutral)
  - `api-details.md` — generic API endpoint specifics, parameters, output formats
  - `codelists-and-filters.md` — codelist/filter syntax: `codelist[Var]=…`, `outputValues[Var]=aggregated`, `top(n)`/`from(x)`/`range(a,b)`, wildcards
  - `troubleshooting.md` — common errors and fixes

## Editing guidance

- The **Data integrity — the base rule** section at the top of `SKILL.md` outranks everything else in the skill: never state a number not fetched from the API in that conversation, never blend other sources, never interpolate. Don't weaken, shorten or relocate it — the rest of the workflow assumes it. It mirrors the same section in `ssb-pxwebapi-v2` (Norwegian), `scb-pxwebapi-v2` (Swedish) and `generic-pxweb-v1-skill`; a change to the rule belongs in all four.
- **Versioning:** `metadata.version` in `SKILL.md` frontmatter plus a `CHANGELOG.md` entry on every content change (semver; stay below 1.0.0 while the skill has no published distribution). A version that isn't bumped lies about copies being current. Unlike `ssb-pxwebapi-v2`, this skill states the version only in frontmatter — there is no prose version line and no repo, so don't add one until it is actually published somewhere.
- Keep the trigger surface broad (Nordic + generic PxWeb keywords). The agency-specific skills (`ssb-pxwebapi-v2`, `scb-pxwebapi-v2`) are preferred when the user clearly targets one country — this skill should fire when the agency is unclear or when the user explicitly mentions a non-Nordic v2 installation.
- The base URL list in `SKILL.md` (Known installations) is the canonical inventory. When a new agency is confirmed on v2, add it there before mentioning it in references.
- Examples should be **schematic** (`/tables/{id}/data?…`) rather than tied to a real table, since the skill spans many installations. When a concrete example is unavoidable, hit the live API and confirm HTTP 200.
- Use `valueCodes` (camelCase) consistently in GET examples — matches the OpenAPI spec and the POST body shape, even though the API is case-insensitive.
- When a behavior is known to differ across installations (e.g. codelist prefixes, default language, time-format granularity), call that out explicitly — the user may apply this skill to an installation we have not tested.

## Related sibling skills

- `ssb-pxwebapi-v2` — Norway-specific, deeper SSB knowledge (Klass/VarDok URNs, `agg_KommFylker`/`agg_KommSummer` with `F-`/`K-` prefixes, common-tables index)
- `scb-pxwebapi-v2` — Sweden-specific, sparser
- `generic-pxweb-v1-skill` — the same vendor-neutral shape for **PxWebApi v1**, which most agencies outside SSB/SCB are still on (Finland, Iceland, Faroe Islands, Greenland, Estonia, verified 2026-08-30)

The v2 skills share the PxWebApi v2 shape — when fixing a bug in one, check whether the others need the same fix. Vendor-neutral improvements (json-stat2 semantics, filter syntax, status codes) belong here; agency-specific ones belong in the country skill.

**The v1 skill follows a rule in both directions:** the **response** side is shared — json-stat2 semantics, `extension.px`, status codes, elimination *semantics*, classification structure — so a fix to any of that probably belongs in both. The **request** side does not transfer at all: v1 is POST-only with a `{"query": […], "response": {…}}` body and `item`/`all`/`top`/`agg:`/`vs:` filters, v2 is GET-or-POST with `valueCodes[…]`, `codelist[…]` and `top(n)`/`from(x)`. Never copy request syntax across, in either direction. Where v1 and v2 genuinely differ in behaviour (v2 returns 400 for a missing mandatory variable where v1 returns all values; the cell limit is 400 in v2 and 403 in v1), say so explicitly — those are the errors people carry between the two.
