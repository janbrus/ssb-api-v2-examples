# Changelog — generic-pxweb-v2-skill

The current version is in `SKILL.md` frontmatter under `metadata.version`.
If your copy has no `metadata.version`, it predates 2026-08-30 — get a newer one.
Versions below 1.0.0 mark a skill with no published distribution yet.

## 0.10.0 — 2026-08-30

Response-side material ported from the v1 skill's research, plus the corrections that probing turned up. All claims verified against SSB and SCB on 2026-08-30.

- **`references/json-stat2.md` rewritten (35 → ~180 lines).** It was a stub — byte-for-byte the same stub as `scb-pxwebapi-v2`'s, only translated — while the skill is meant to be the vendor-neutral home for exactly this material. New or corrected:
  - **Corrected the `extension` bullet.** It listed `/tables` fields (`firstPeriod`, `lastPeriod`, `discontinued`) as dataset `extension`. Replaced with the real two-level structure: dataset level (`px`, `contact`, `noteMandatory`, `discontinued`) and dimension level (`elimination`, `eliminationValueCode`, `codelists`, `show`, `refperiod`, `measuringType`, `priceType`, `adjustment`, `basePeriod`, `alternativeText`, `noteMandatory`, `categoryNoteMandatory`)
  - **New `extension.px` section** — `aggregallowed` (the only place the "can this table be summed at all" answer appears), `decimals`, `heading`/`stub`, `subject-code`
  - **New dimension-`extension` section**, highlighting `basePeriod` and `measuringType`/`priceType`/`adjustment` — these decide how a number may honestly be described, and were not mentioned anywhere before
  - **New trap: eliminability is readable only from the metadata response.** In a *data* response `extension.elimination` describes the extract you received — `true` only when the returned value set still contains the elimination value — not the table's contract. The inverse holds for `eliminationValueCode`: absent from metadata across a 50-table sweep, but emitted in a data response that includes the total. Both PX elimination forms (`ELIMINATION=YES` vs `ELIMINATION("<value>")`) look identical in metadata
  - **New "What json-stat2 cannot express"** — time periodicity (PX's TLIST; v2's only carrier is `timeUnit` on `/tables`, *outside* the document), eliminated dimensions, codelist provenance
  - **Added `note` and `category.note`** with the `noteMandatory` / `categoryNoteMandatory` semantics, `link.describedby`, and the `status` / `DATASYMBOL1`–`6` explanation
  - **New tooling section** — pyjstat, rjstat, `PxWebApiData` (supports **both** API versions since 1.9.0: v2 via the snake_case `api_data()`/`query_url()`/`meta_data()` interface, v1 via `ApiData()`), json-stat Toolkit CLI, the explorer
- **`references/troubleshooting.md`:** the 400 guidance said "check the `detail` field". Verified wrong — `detail` is set only for `Too many cells selected`, and then only as a copy of `title`. Replaced with a `title`-keyed table of the four observed values, plus the positive finding that the payload shape is **identical across installations**, so the bisection procedure the v1 skill needs is unnecessary here. Also noted that the cell limit is a **400** in v2, not v1's 403
- **`references/api-details.md`:** replaced the 6-field `/config` example with the verified 12-field response. Three points called out — `sourceReferences` gives the agency's own citation string per language, `maxDataCells` spans 5× between the two known installations, and `dataFormats` is per installation (SSB serves `parquet`, SCB does not)
- **`SKILL.md`:**
  - Installation table now lists the five agencies **verified still v1-only** (Finland, Iceland, Faroe Islands, Greenland, Estonia) and routes them to `generic-pxweb-v1-skill`, plus a "which version is this?" recipe. **Check that the body parses as JSON, not the status code** — Statistics Greenland answers `GET /api/v2/config` with HTTP 200, `text/html`, and `CONFIG404;…File not found` in the body
  - Step 3 elimination rules rewritten: both PX forms spelled out, `eliminationValueCode` named as the discriminator that metadata does not carry, and the note that omitting a non-eliminable variable is a 400 in v2 where **v1 returns all the values instead** — the sharpest difference between the two APIs
  - Step 2 gained a table of the `/tables` hit fields, led by `variableNames` (screen candidates without a metadata call each) and `timeUnit` (the only place frequency is published)
  - Step 5 now requires saying which dimensions were collapsed and which codelist was used, since the response records neither; mandatory notes added; `parquet` added to the format table
- **`references/codelists-and-filters.md`:** new section on where codelists come from — an aggregation is defined **on a valueset, not on a variable**, which is the actual reason the "never mix codelists" trap exists. Also covers non-namespaced codelist names and why hierarchical prefixes line up with wildcard filters
- **`CLAUDE.md`:** `generic-pxweb-v1-skill` added to the sibling list, with the rule the v1 skill already states in the other direction — the response side is shared, the request side does not transfer at all

Deliberately **not** carried over: a `pageSize=500 → HTTP 500` observation from the v1 session (not reproducible — 100/200/500/1000 all return 200 today, so writing it down would have created a false rule), and any new v2 installations (none of the five have migrated; the *negative* finding went in as routing instead).

## 0.9.0 — 2026-08-30

Versioning introduced, plus the rules that `ssb-pxwebapi-v2` had and this skill was missing.

- **Versioning introduced** (`metadata.version` in `SKILL.md` frontmatter + this log). A user holding an old copy previously had no way to tell
- **New top section "Data integrity — the base rule"**, placed after the intro and before the installation list: **never state a number you have not fetched from the API in this conversation.** Seven bullets — no numbers from memory, no other sources in the same answer, say so when the API fails, no interpolation, mark your own calculations and keep the API's decimals, show `status` values as they are, check `discontinued`/`lastPeriod`. The rule previously existed only in `ssb-pxwebapi-v2`; the argument is *stronger* here, since this skill can point at any statistical agency. `CLAUDE.md` records that the section outranks the rest, so it does not get relocated or watered down in a later edit
- **Rate limiting: check both `/config` and the response headers.** The two known installations behave in opposite ways — SSB reports `maxCallsPerTimeWindow: 0` in `/config` but sends `x-ratelimit-*` headers (40 per 60 s); SCB reports `30`/`10` in `/config` and sends no headers. The old text said only "check `/config`", which against SSB reads as "no limit". **`0` means "not in use", not "unlimited"**
- **Defaultselection behaviour documented.** A `GET /data` with no selection parameters is not an error and does not return the whole table — it silently returns the default selection (SSB 07459 → `size [360, 1, 1]`, 360 cells, 200; SCB TAB638 → `size [290, 1, 1, 2]`, 580 cells, 200). The dangerous part: 07459 has **five** dimensions and the default selection returns **three** — sex and age were summed away unannounced
- **New rule for the time dimension:** prefer `top(N)`/`from(value)` over `range(from,to)` and explicit periods — relative filters capture new periods automatically, so shareable URLs and saved queries stay current. Matters most for `/savedqueries`
- **Cross-reference to `ssb-chart-skill`** in Step 5, for visualizing SSB data (with the usual "if available in the environment" caveat)
