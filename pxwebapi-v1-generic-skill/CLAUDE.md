# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This is **not** an application codebase. It is a Claude Code **Skill** package that teaches Claude
how to query *any* PxWebApi v1 installation (the PxWeb 1.0 API). There is no build, no tests, no
runtime — changes here are documentation/prompt edits that get loaded when the skill triggers.

The defining constraint of v1, and the thing every page here has to keep in view: **data is
POST-only**, metadata is thin, and there is no codelist endpoint and no saved-query API. Limits
*are* discoverable, but through `?config` as a **query parameter** on the language level — the
`/config` path returns 400, and an early draft of this skill wrongly concluded from that that v1
had no config endpoint at all.

v1 is not "v2 with fewer features" — the request shape is genuinely different, and the gaps in
discovery drive most of the guidance.

Like its sibling `generic-pxweb-v2-skill`, this skill is **vendor-neutral**. Where a behaviour is
agency-specific (SSB's `K-`/`F-` aggregation prefixes, SCB's `TotSA` totals), say so explicitly
rather than presenting it as universal.

## Layout

- `SKILL.md` — the skill entrypoint. Frontmatter (`name`, `description`) controls auto-triggering;
  the body is the operational guide. Keep frontmatter `name` aligned with the folder name
  (`generic-pxweb-v1-skill`) — Claude Code triggers by folder name in practice, but the frontmatter
  `name` is used when the skill is packaged and uploaded to claude.ai.
- `references/` — deeper material loaded on demand:
  - `query-syntax.md` — the `{"query": […], "response": {…}}` body, the item/all/top/agg/vs
    filters, elimination rules, discovering aggregations, worked examples, cURL
  - `api-details.md` — URL structure, hierarchy navigation, `?query=` search syntax, output
    formats, PX-JSON, limits
  - `json-stat2.md` — json-stat2 Dataset structure, row-major indexing, `role`-first analysis,
    units, status codes (vendor-neutral; also applies to Eurostat and the World Bank)
  - `troubleshooting.md` — HTTP codes and the three distinct v1 error payloads
  - `installations.md` — the broad inventory: 49 known v1 installations with status, languages
    and `?config` limits, grouped by country
  - `px-files-and-classifications.md` — file-based vs relational installations, `.vs`/`.agg` file
    structure, and the PX keywords behind the API's metadata (`ELIMINATION`, `AGGREGALLOWED`, …)
  - `v1-vs-v2.md` — endpoint/body/filter translation both directions, and using v2 to fill v1's
    discovery gaps

## Editing guidance

- The **Data integrity — the base rule** section near the top of `SKILL.md` outranks everything else in the skill: never state a number not fetched from the API in that conversation, never blend other sources, never interpolate. Don't weaken, shorten or relocate it. It mirrors the same section in `generic-pxweb-v2-skill`, `ssb-pxwebapi-v2` (Norwegian) and `scb-pxwebapi-v2` (Swedish); a change to the rule belongs in all four. The v1 version deliberately differs on two points: the `status` bullet notes that symbols are per-PX-file (`DATASYMBOL1`–`6`), and the API-failure bullet notes that several v1 installations return a bare `Bad Request`, which makes a failed call easy to mistake for an empty result. There is no `discontinued` bullet — v1 has no such flag.
- **Versioning:** `metadata.version` in `SKILL.md` frontmatter plus a `CHANGELOG.md` entry on every content change (semver; stay below 1.0.0 while the skill has no published distribution). A version that isn't bumped lies about copies being current.
- **Verify against a live installation before changing a factual claim.** Everything currently in
  these files was reproduced with `curl` against SSB and/or SCB on 2026-08-28. Where a claim
  contradicts published documentation, the file says so and states which one is right — keep that
  habit, and keep the verification date accurate when you re-check.
- Known contradictions between the official PxWeb 1.0 spec and live behaviour, already documented
  and worth not "correcting" back:
  - Format names are **hyphenated** (`json-stat2`, `json-stat`). The spec's `jsonstat2` /
    `jsonstat` return 400 on both SSB and SCB.
  - **Multiple wildcards in one `all` list work.** The spec says only one is permitted.
  - The cell-limit response is **403**, not 400 or 413 (and at Finland it is an IIS HTML page, not
    a JSON body).
  - The JSON error payloads (`{"error":"Parameter error"}` etc.) are **SSB-only**. SCB, Finland and
    Greenland all return a bare `Bad Request` with no diagnostic.
  - `?config` reports limits that differ from the agencies' own published figures — SSB documents
    30 calls/60 s where `?config` says 300; Finland's help page says 100,000 cells where `?config`
    says 120,000 (confirmed by bisection). `?config` wins.
- **Probe the query-parameter form before declaring an endpoint absent.** v1 puts config on
  `?config`, not `/config`; testing only the path form produced a wrong "v1 has no config endpoint"
  claim that survived several revisions of this skill.
- **The inventory has two levels, and they are verified to different depths.** The table in
  `SKILL.md` is the *operational* list: seven installations with a base URL through DATABASEID,
  the hierarchy walked, and the `?query=` support column probed — search is genuinely absent
  (400) at SCB and Statistics Iceland, and that is load-bearing for the workflow.
  `references/installations.md` is the *broad* inventory: 49 installations from the R package
  `pxweb`, verified only to the LANGUAGE level (root endpoint returns JSON, `?config` read).
  Re-probe before adding a row to either. An entry may not move from the broad list to the
  `SKILL.md` table without probing `?query=` and finding its DATABASEID — that is exactly the
  work the two levels distinguish.
- **`?config` outranks every other source of limits.** Agencies' published figures and
  third-party catalogues are wrong often enough to be unusable: across the 26 installations the
  R catalogue and `?config` both describe, 10 disagree on the call limit and 10 on the value
  limit, some by orders of magnitude (`askdata.rks-gov.net` listed at 10 calls/10 s where
  `?config` reports 100 000). The pattern is that catalogues record what agencies *publish* and
  `?config` reports what the installation *enforces*. Never write a limit into this skill from
  anywhere else. Note also that `maxCells` is **absent** from five of 43 `?config` payloads —
  absence is not "unlimited".
- **The R package `pxweb` ships two catalogues, and they disagree.** The development version on
  GitHub has 46 installations with modern URLs but **no** rate-limit fields; CRAN 0.17.0 has 30
  with older URLs (11 still `http://`), the limit fields, and an SSB entry the development
  version has dropped. `references/installations.md` merges them — URLs and coverage from the
  development catalogue, the three CRAN-only entries marked `‡`, limits from live `?config`.
  Anyone regenerating that file needs to know both exist, and that the limit fields are on their
  way out upstream.
- Examples should stay **schematic** (`POST {table_url}`) except where a concrete, verified call
  makes the point better — the Finland example exists specifically because
  `alue_23_20260101` / `contentscode` / `timeperiod_y` proves that Nordic variable names are not
  universal. Keep at least one non-Nordic example for that reason.
- **Remember which installations are file-based.** Only SSB and SCB are relational; everywhere else
  a table is a `.px` file and its classifications are `.vs`/`.agg` files the API never reads. This
  is the reason the `.px` extension appears in most URLs, and the reason the "query v2 for
  aggregation names" trick helps only at the two installations that need it least. Do not present
  that trick as a general solution.
- Do not import v2 filter syntax (`top(5)`, `from()`, `range()`, `?` masking) into v1 guidance.
  The overlap is smaller than it looks, and `references/v1-vs-v2.md` is where the mapping belongs.
- When a behaviour varies by installation (search support, available formats, aggregation
  prefixes, cell limits), call that out explicitly. The skill will be applied to installations
  nobody has tested.

## Related sibling skills

- `generic-pxweb-v2-skill` — the same shape for PxWebApi v2; vendor-neutral
- `ssb-pxwebapi-v2` — Norway-specific v2 (Klass/VarDok URNs, `agg_KommFylker` / `agg_KommSummer`
  with `F-`/`K-` prefixes, curated table index)
- `scb-pxwebapi-v2` — Sweden-specific v2

These skills share the json-stat2 response format and the elimination semantics, so a fix to
`json-stat2.md` or to the elimination rules probably belongs in more than one of them. The
request-side material does **not** transfer — v1 and v2 differ there completely.
