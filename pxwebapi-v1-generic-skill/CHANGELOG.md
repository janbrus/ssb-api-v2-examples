# Changelog — generic-pxweb-v1-skill

The current version is in `SKILL.md` frontmatter under `metadata.version`.
If your copy has no `metadata.version`, it predates 2026-08-30 — get a newer one.
Versions below 1.0.0 mark a skill with no published distribution yet.

## 0.11.0 — 2026-09-01

One new section. It closes a failure mode that is specific to v1 being POST-only: an assistant can
do the whole workflow correctly and only discover at the last step that its tooling cannot retrieve
data at all.

- **New section "Environment check — before Step 1"**, placed after the v1-vs-v2 pointer and before
  the Workflow. It asks one question up front — *can this environment send an HTTP POST with a JSON
  body?* — and names the three cases:
  - **Bash/shell with network access** to the target host → `curl -X POST` works, proceed normally.
  - **A GET-only web-fetch tool** (one restricted to URLs already seen in a search or fetch result)
    → **cannot** retrieve data. GET reaches metadata, the database/table hierarchy and `?config`,
    and stops there. This is not a limitation to work around; in v1 there is no GET data endpoint
    to fall back on.
  - **An MCP tool wrapping HTTP with POST support** → works if connected.

  The GET-only branch is the point of the section. Instead of abandoning the task, do the parts that
  genuinely don't need POST — identify the installation, walk the hierarchy, pin down the exact
  table and its variable codes from metadata (Steps 1–2) — then route to **Fallback** for the
  figures: hand over the table's URL in the agency's web front end and its "API query for this
  table" button, which emits a ready-made POST body the user or a POST-capable tool can run.

  The closing instruction is the data-integrity rule applied to a *tooling* constraint rather than
  an API failure: state plainly that the figures could not be retrieved in this environment, and
  substitute nothing — not memory, not a third-party aggregator. Without it the GET-only case is
  exactly the situation that tempts an assistant into a plausible number.

Deliberately **not** done:

- **Not propagated to the v2 siblings.** In v2 data retrieval is a GET, so a GET-only fetch tool can
  read the data cube there. The failure mode this section prevents does not exist in
  `generic-pxweb-v2-skill`, `ssb-pxwebapi-v2` or `scb-pxwebapi-v2`.
- **No environment detection logic.** The section describes the classes of tooling and lets the
  assistant recognise its own; probing for POST capability against a live agency host would spend a
  request to learn something already known locally.

## 0.10.0 — 2026-08-31

A broad installation inventory, and two corrections that came out of probing it. Every claim
below was verified against live installations on 2026-08-31.

- **New `references/installations.md`** — 49 known v1 installations with status, languages and
  limits, grouped by country. Sourced from the two catalogues shipped with the R package `pxweb`
  (rOpenGov), merged: URLs and coverage from the development version on GitHub (46 entries,
  modern URLs), the three entries that exist only in CRAN 0.17.0 (`data.ssb.no`, `px.rsv.is`,
  `pxwebapi2.stat.fi`), and **every limit read from live `?config`**.

  43 of the 49 responded. The six that did not are listed with their exact failure.

  This does **not** replace the seven-row table in `SKILL.md`. That table stays the operational
  list — base URL through DATABASEID, hierarchy walked, `?query=` support probed. The new file
  is verified only to the LANGUAGE level. `CLAUDE.md` now records the two levels and the rule
  that an entry cannot be promoted without probing `?query=` and finding its DATABASEID.

- **The bare error text is localised — a correction to how the existing rule reads.** The skill
  already said several installations return a bare `Bad Request` with no diagnostic. Probing
  found Spain's judicial statistics answering **`Solicitud incorrecta`**. Anything that tests for
  the literal string `Bad Request` misclassifies that response, and a failed call can then read
  as an empty result. `SKILL.md`'s integrity bullet and `references/troubleshooting.md` now state
  the structural rule instead: **any 4xx without a JSON body is a failed request.** The 400-body
  table grew from four installations to six.

- **`?config` answered on 43 of 43 reachable installations.** It is universal in v1, not a
  feature some installations happen to offer — `SKILL.md` Step 1 now says so with the number
  behind it. Two findings came with it:
  - **`maxCells` is absent from five payloads** (`etab.llv.li`, Sundsvall, `openstat.psa.gov.ph`,
    `pc-axis.geostat.ge`, Västerås). The field table implied it was always present. Absence is
    not "no cell limit" — the 403 still applies; the ceiling has to be found by bisection.
  - **Third-party limits are unreliable.** Of the 26 installations described by both the R
    catalogue and `?config`, **10 disagree on the call limit and 10 on the value limit**, some by
    orders of magnitude (`askdata.rks-gov.net`: catalogue 10 calls/10 s, `?config` 100 000). The
    pattern is that catalogues record what agencies *publish* while `?config` reports what the
    installation *enforces* — SSB is the clean case, documented at 30 calls/60 s and enforcing
    300. `CLAUDE.md` now forbids writing a limit into this skill from any other source.

- **Two catalogue URLs are wrong**, recorded in the new file and corrected in its tables:
  `pc-axis.geostat.ge` answers 404 over `http` and JSON over `https`; `data.ssb.no` redirects
  `http` → `https`. Geostat is the instructive one — a probe that followed the catalogue
  faithfully would have reported a live installation dead.

Deliberately **not** done:

- **No propagation to the sibling skills.** The integrity rule is mirrored in
  `generic-pxweb-v2-skill`, `ssb-pxwebapi-v2` and `scb-pxwebapi-v2`, but the API-failure bullet
  is already marked as deliberately v1-specific. The localisation finding concerns v1 error
  payloads only.
- **`?query=` support was not probed** for the 42 new installations. Without it they cannot be
  promoted to the `SKILL.md` table, and the new file says so.
- **No limits from the R catalogue** anywhere in the skill, including for the six installations
  that did not respond — they have no limit column at all rather than an unverified one.

## 0.9.0 — 2026-08-30

Versioning introduced, plus the data-integrity rule the v2 sibling skills carry.

- **Versioning introduced** (`metadata.version` in `SKILL.md` frontmatter + this log). A user holding an old copy previously had no way to tell
- **New section "Data integrity — the base rule"**, placed after the POST-only opening and before the installation table: **never state a number you have not fetched from the API in this conversation.** Six bullets — no numbers from memory, no other sources in the same answer, say so when the API fails, no interpolation, mark your own calculations, show `status` values as they are. The rule previously existed only in `ssb-pxwebapi-v2`; it has nothing to do with Norway, and the argument is *stronger* in a vendor-neutral skill that can point at any agency

  Two bullets are deliberately v1-specific:
  - The `status` bullet notes that the symbols are defined **per PX file** (`DATASYMBOL1`–`6`), so they vary between agencies and even between tables — don't assume `.` and `..`
  - The API-failure bullet notes that this matters more in v1 than in v2, because several installations (SCB, Finland, Greenland) return a bare `Bad Request` with no diagnostic, which makes a failed call easy to mistake for an empty result

  There is **no** `discontinued` bullet — v1 has no such flag; hierarchy nodes carry only `updated`.

  `CLAUDE.md` records that the section outranks the rest of the skill, and that it is mirrored in `generic-pxweb-v2-skill`, `ssb-pxwebapi-v2` and `scb-pxwebapi-v2` — a change to the rule belongs in all four.
- Sibling skills: `generic-pxweb-v2-skill` 0.10.0 released at the same time. It now carries the response-side material developed here (json-stat2 `extension` semantics, `extension.px`, status codes, the classification model behind `agg_`/`vs_`), and routes v1-only agencies here. The **request** side deliberately does not transfer in either direction

  One correction worth knowing if you work across both: v2 returns **400** when a non-eliminable variable is omitted, where v1 returns **all of that variable's values** instead. And the cell-limit response is **400** in v2, not v1's **403**. These are the two mistakes people carry between the APIs.
- **Clarified that `PxWebApiData` is not a v1-only client.** Since 1.9.0 the R package covers both API versions — v1 through `ApiData()`, v2 through a separate snake_case interface (`api_data()`, `query_url()`, `meta_data()`, …), with a vignette for each. A user arriving with a `PxWebApiData` script is therefore no evidence that the installation is on v1; check the base URL instead
