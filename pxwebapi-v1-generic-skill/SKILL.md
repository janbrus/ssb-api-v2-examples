---
name: generic-pxweb-v1-skill
description: >
  Access official statistics from any PxWebApi v1 installation (the PxWeb 1.0 API).
  Use when querying statistical databases that run the older POST-based PxWeb API —
  base URLs of the form `/api/v1/{lang}/{database}/…`, or SSB's `/api/v0/`.
  Trigger on "PxWebApi v1", "PxWeb v1", "PxWeb API", "PXAPI", "statbank API",
  "Statistikkbanken API", "Statistikdatabasen API", or when a PxWeb URL contains
  `/api/v0/` or `/api/v1/`. Covers hierarchy navigation, table search, metadata,
  the JSON query body (`{"query": […], "response": {…}}`), the item/all/top/agg/vs
  filters, elimination rules, output formats, and error handling.
  For installations already migrated to PxWebApi v2 (SSB `data.ssb.no/api/pxwebapi/v2`,
  SCB `statistikdatabasen.scb.se/api/v2`) prefer `generic-pxweb-v2-skill`,
  `ssb-pxwebapi-v2` or `scb-pxwebapi-v2` instead.
metadata:
  version: "0.10.0"
---

# PxWebApi v1 — Generic Skill

This skill guides you through using **PxWebApi v1** (PxWeb 1.0 API) to navigate, explore, and retrieve official statistics. PxWeb is developed by Statistics Sweden (SCB) and used by statistical agencies worldwide. Many installations are still on v1 and have no migration date.

**The single most important fact:** in v1, **data is retrieved with HTTP POST only.** There is no GET data endpoint. A GET against a table URL returns that table's *metadata*, not its numbers.

## Data integrity — the base rule

Official statistics carry an agency's name. A wrong number under that citation damages trust in the agency, not just in the answer. This rule outranks everything else in this skill:

**Never state a number you have not fetched from the API in this conversation.**

- **No numbers from memory.** If you did not run the query, you do not have the number — including numbers you are confident about. Populations, price indices and unemployment rates all move, and training data has a cutoff.
- **No numbers from other sources in the same answer.** Point the user elsewhere rather than blending.
- **If the API fails, say so.** No estimates, no "roughly". See Fallback. This matters more in v1 than in v2: several installations return a bare `Bad Request` with no diagnostic, so a failed call is easy to mistake for an empty result. **The text is localised** — Spain's judicial statistics answer `Solicitud incorrecta`. Do not test for the string; treat any 4xx without a JSON body as a failed request.
- **No interpolation or projection.** A period missing from the extract is missing from the answer.
- **Mark your own calculations.** Growth rates, shares and sums are yours, not the agency's — show which fetched numbers they rest on, and keep the API's decimals.
- **Show `status` values as they are.** Missing, provisional and confidential values belong in the table, not hidden or replaced by zero. In v1 the symbols are defined per PX file (`DATASYMBOL1`–`6`), so they vary between agencies and even between tables — read them, don't assume `.` and `..`.

Not finding the number is a valid answer. An honest "not found", with suggested search terms, beats a plausible number that is wrong.

## Known PxWebApi v1 installations

All base URLs below were verified live on 2026-08-28.

| Agency | Country | Base URL (through DATABASEID) | `?query=` |
|---|---|---|---|
| Statistics Norway (SSB) | Norway | `https://data.ssb.no/api/v0/{no\|en}/table` | yes |
| Statistics Sweden (SCB) | Sweden | `https://api.scb.se/OV0104/v1/doris/{sv\|en}/ssd` | **no** (400) |
| Statistics Finland | Finland | `https://pxdata.stat.fi/PXWeb/api/v1/{fi\|sv\|en}/StatFin` | yes |
| Statistics Iceland | Iceland | `https://px.hagstofa.is/pxis/api/v1/is/{database}` | **no** (400) |
| Statistics Faroe Islands | Faroe Islands | `https://statbank.hagstova.fo/api/v1/{fo\|en}/H2` | yes |
| Statistics Greenland | Greenland | `https://bank.stat.gl/api/v1/{da\|en\|kl}/Greenland` | yes |
| Statistics Estonia | Estonia | `https://andmed.stat.ee/api/v1/{et\|en}/stat` | yes |

Two of these do not follow the pattern the others suggest:

- **Iceland splits its content across six databases** — `Atvinnuvegir`, `Efnahagur`, `Ibuar`,
  `Samfelag`, `Sogulegar`, `Umhverfi` — so there is no single base URL; GET
  `https://px.hagstofa.is/pxis/api/v1/is` to list them. English is served by a **different
  APINAME**: `pxen` in place of `pxis` (`https://px.hagstofa.is/pxen/api/v1/en`), not by swapping
  the language segment.
- **SSB's `table` is a DATABASEID, not a literal path segment** — `GET https://data.ssb.no/api/v0/no`
  returns `[{"dbid": "table", "text": "Statistikkbanken etter emne"}]`. It just happens to read
  like a REST resource.
- **Statistics Finland hosts eleven databases**, of which `StatFin` is only the main one; the rest
  include `StatFin_Passiivi` (discontinued series), `Kuntien_avainluvut` (municipal key figures),
  `Hyvinvointialueet` (wellbeing services counties) and `SDG`. GET
  `https://pxdata.stat.fi/PXWeb/api/v1/en` to list them. A table missing from `StatFin` is often
  in `StatFin_Passiivi`.

The seven above are verified end to end: base URL through DATABASEID, hierarchy walked, `?query=` support probed. **`references/installations.md` holds the full inventory of 49 known v1 installations** — status, languages and `?config` limits, all probed 2026-08-31 — but only to the LANGUAGE level, without `?query=` support. Look there when the user names an agency not in this table; then walk the hierarchy from its base URL as in Step 1.

Neither list is exhaustive — many national and regional agencies run PxWeb. If an installation appears in neither, probe it rather than assuming it does not exist.

**Note:** SSB serves PxWebApi **1.0 at `/api/v0/`** — the `v0` is the URL's API-version segment, not a beta marker. SSB also runs v2 in parallel at `https://data.ssb.no/api/pxwebapi/v2`; so does SCB. When an installation offers both, prefer v2 (`generic-pxweb-v2-skill`) unless the user specifically wants v1.

---

## How v1 differs from v2

If you know PxWebApi v2, read this table before doing anything else. The **response** format is nearly identical (both return json-stat2); the **request** shape and the **discovery** endpoints are completely different.

| | **v1** | **v2** |
|---|---|---|
| Data retrieval | **POST only** | GET or POST |
| Metadata | `GET {table_url}` (same URL you POST to) | `GET /tables/{id}/metadata` |
| Request body key | `{"query": [ … ], "response": {"format": …}}` | `{"selection": [ … ]}` |
| Selection object | `{"code": …, "selection": {"filter": …, "values": […]}}` | `{"variableCode": …, "valueCodes": […]}` |
| Output format | `"response": {"format": "json-stat2"}` in the body | `?outputFormat=json-stat2` in the query string |
| Filters | `item`, `all`, `top`, `agg:X`, `vs:X` | `top(n)`, `bottom(n)`, `from(x)`, `to(x)`, `range(a,b)`, `*`, `?` |
| Codelists / aggregations | **not in metadata** — must be known in advance | `extension.codelists` + `GET /codelists/{id}` |
| Table discovery | hierarchy navigation; `?query=` on some installations | `GET /tables?query=…` with pagination |
| Limits | `GET …/{LANGUAGE}/?config` (query parameter) | `GET /config` (path segment) |
| Saved queries | none | `/savedqueries` |
| Units, decimals, `role` in metadata | no (present in the json-stat2 *data* response only) | yes |

See `references/v1-vs-v2.md` for a migration guide in both directions.

---

## Workflow

Follow these steps in order. Never skip the metadata step — v1 metadata is thin, and variable codes are wildly installation-specific.

### Step 1: Identify the installation and its base URL

The v1 URL is assembled from fixed parts:

```
{host}/{APINAME}/{APIVERSION}/{LANGUAGE}/{DATABASEID}/{LEVEL1}/…/{LEVELN}/{TABLEID}
```

For example `https://api.scb.se/OV0104/v1/doris/sv/ssd/BE/BE0101/BE0101A/BefolkManadCKM`, where `OV0104` is APINAME, `v1` is APIVERSION, `doris` is a routing segment, `sv` is LANGUAGE and `ssd` is DATABASEID.

If you do not know an installation's base URL, walk it down from the top: a GET on `.../{APIVERSION}/{LANGUAGE}` lists the databases, and a GET on `.../{LANGUAGE}/{DATABASEID}` lists the first level. Every level returns a JSON array of nodes.

**Always start with `?config`.** v1 exposes its limits as a **query parameter on the LANGUAGE level**, not as a `/config` path segment (`/config` as a path returns 400 — that is why it is easy to conclude the endpoint does not exist):

```
GET {host}/{APINAME}/{APIVERSION}/{LANGUAGE}/?config
→ {"maxValues": 120000, "maxCells": 120000, "maxCalls": 40, "timeWindow": 60, "CORS": true}
```

| Field | Meaning |
|---|---|
| `maxCells` | Maximum cells per query — the product of all selected value counts |
| `maxValues` | Maximum **selected values** per query, counted across variables — a separate ceiling |
| `maxCalls` / `timeWindow` | Rate limit: `maxCalls` requests per `timeWindow` seconds |
| `CORS` | Whether browser clients can call the API directly |

Verified on all seven installations listed above (Estonia needs the URL without a trailing slash: `.../et?config`), and on **43 of 43** reachable installations in the wider sweep of 2026-08-31 — `?config` is universal in v1, not a feature some installations happen to offer.

`maxValues` and `maxCells` are independent — SSB allows 800,000 cells but only 50,000 selected values, so a query naming tens of thousands of individual codes can fail well inside the cell budget.

**`maxCells` may be absent.** Five of the 43 return the other four fields without it. Do not read that as "no cell limit" — the 403 cell-limit response still applies; you just have to find the ceiling by bisection. See `references/installations.md`.

**Do not trust a limit from anywhere but `?config`.** Agencies' published figures and third-party catalogues disagree with it routinely: of 26 installations listed in the R package `pxweb`, 10 have the wrong call limit and 10 the wrong value limit. SSB is the clean example — its own documentation says 30 calls/60 s, `?config` says 300.

### Step 2: Find the table

Two routes. Use search where it exists; otherwise navigate.

**a) Navigate the hierarchy (works everywhere).** GET any level to list its children:

```
GET {base_url}/be
→ [{"id":"be01","type":"l","text":"Befolkning"}, …]
```

Each node has `id`, `text` and `type`:

| `type` | Meaning |
|---|---|
| `l` | a sub-level — append its `id` to the URL and GET again |
| `t` | a **table** — this is a POST target |
| `h` | a heading (display only, not navigable) |

Table nodes usually also carry `updated`. On several installations the table `id` includes a **`.px` extension** that is part of the URL (Finland `11ra.px`, Faroe `fo_sogtol.px`, Greenland `BEXSTA.px`, Estonia `RL101.PX`) — SSB and SCB do not use it. Always use the `id` exactly as returned.

That extension is a tell. **Most PxWeb installations store each table as a flat PX file on disk; only SSB and SCB run off a relational database.** File-based installations expose the `.px` in the URL and keep their aggregations and valuesets as separate `.vs`/`.agg` files that the API never exposes. See `references/px-files-and-classifications.md`.

**b) Search with `?query=` (installation-dependent).** Where supported, append `?query=` to any level URL to search that subtree:

```
GET {base_url}/?query=title:population*
```

Search is Lucene-based (Apache Lucene.NET query syntax), case-insensitive, and space means AND. It searches titles *and* variable value texts by default. Hits come back as `{"id", "path", "title", "score", "published"}` — note the `path`, which you must append to the base URL to build the POST target.

**Search is not universally available.** SCB and Statistics Iceland return `400 Bad Request` for `?query=`. Probe once; if it 400s, navigate the hierarchy instead.

See `references/api-details.md` for the full search syntax (field prefixes, truncation, proximity, date searches, URL-encoding).

**Beware:** the same table can appear under several subject paths, so a search may return apparent duplicates with identical `id` and `title` but different `path`. Any of them works as a POST target.

### Step 3: Read the table's metadata

```
GET {table_url}
```

The same URL you will POST to. The response is deliberately minimal:

```json
{
  "title": "07459: Befolkning, etter region, kjønn, alder, statistikkvariabel og år",
  "variables": [
    { "code": "Region", "text": "region",
      "values": ["0", "31", "3101", …], "valueTexts": ["Hele landet", "Østfold", …],
      "elimination": true },
    { "code": "ContentsCode", "text": "statistikkvariabel",
      "values": ["Personer1"], "valueTexts": ["Personer"] },
    { "code": "Tid", "text": "år",
      "values": ["1986", …, "2026"], "valueTexts": ["1986", …, "2026"],
      "time": true }
  ]
}
```

Each variable object has `code` and `text` (both required), plus optional `elimination` and `time` — **when absent, both default to `false`**. `values` and `valueTexts` are positionally aligned: `values[i]` is the code you put in a query, `valueTexts[i]` is its human label. At most one variable may have `time: true`.

**`time: true` tells you nothing about the frequency, and does not guarantee the codes are dates.** The PX file declares a time scale (`TLIST(A1|H1|Q1|M1|W1)`) but v1 discards it, and json-stat2 has no field for it either. Infer the frequency from `values` — `2026` annual, `2025M12` monthly, `2020K3` quarterly at SSB/SCB — and never construct codes from a pattern you have not seen in that table. Statistics Greenland has a `time: true` variable whose codes are `0`–`50` with the years only in `valueTexts`; there, `item` needs `"50"`, not `"2025"`. See `references/query-syntax.md`.

**Never assume variable names.** The Nordic convention `Region` / `ContentsCode` / `Tid` holds at SSB and SCB but nowhere near universally — Statistics Finland uses `alue_23_20260101`, `contentscode` and `timeperiod_y` in the same role, and the geography code even embeds a classification date that changes between table versions. Statistics Greenland uses lowercase English words, **including codes that contain spaces** (`place of birth`). Read `variables[].code` every time and copy it verbatim into `"code"`.

**Totals may be explicit values, eliminable, or both.** Greenland's `BEXST8.px` gives `age` an explicit `-1` = "Total" *and* `elimination: true`; selecting `-1` and omitting the variable return the same figure (verified). SSB's `Kjonn` in table 07459 has neither total in `values` nor any way to get one except omission. Check `valueTexts` for a "Total"/"I alt"/"Hele landet" entry before assuming which route you need.

**What v1 metadata does *not* tell you** — plan around these gaps:

- **No `role`.** You must infer which variable is the metric, the time and the geography. The time variable is the one with `time: true`. The metric is usually the one named like `ContentsCode`/`contentscode`; otherwise it is the variable whose `valueTexts` read as measures ("Persons", "Index", "NOK"). Geography is whatever looks like regions. *The json-stat2 data response does include `role`* — so if you are unsure, run a tiny `top`-1 probe query and read `role` off the result.
- **There may be no metric variable at all.** Many tables outside the Nordic core have no `ContentsCode`-equivalent: the whole table measures one thing, named only in the title. Statistics Greenland's `BEXST8.px` has variables `age`, `place of birth`, `gender`, `time` and nothing else — its data response comes back with `role: {"time": ["time"]}`, no `metric` and no `geo`. When `role.metric` is absent, do not hunt for it: read the measure off the table title and the subject level, and say so explicitly when presenting.
- **No units or decimals.** Also only in the data response (`dimension.{metric}.category.unit`) — and some installations omit `unit` there too.
- **No codelists or aggregations.** Groupings such as five-year age bands or merged-municipality time series exist and are usable via the `agg:` filter, but are invisible here — on file-based installations they are separate `.vs`/`.agg` files that the API never reads. See `references/query-syntax.md` for how to discover them and `references/px-files-and-classifications.md` for how they are structured.
- **No elimination *value*.** When `elimination: true`, the total exists but is often not in `values` — you obtain it by omitting the variable, not by selecting a code. (Some installations do list an explicit total, e.g. SCB's `TotSA`/`TotSa`.)

### Step 4: Build the query and POST it

```
POST {table_url}
Content-Type: application/json

{
  "query": [
    { "code": "Region",       "selection": { "filter": "item", "values": ["0301"] } },
    { "code": "ContentsCode", "selection": { "filter": "item", "values": ["Personer1"] } },
    { "code": "Tid",          "selection": { "filter": "top",  "values": ["3"] } }
  ],
  "response": { "format": "json-stat2" }
}
```

`"query"` is an array of selection objects; `"response"` is optional and **defaults to PX format**, so always set it explicitly.

#### Filters

| `filter` | Meaning | `values` |
|---|---|---|
| `item` | Explicit list of value codes | `["0301", "1103"]` |
| `all` | Wildcard match | `["*"]` = all; `["202*"]` = codes starting with 202 |
| `top` | The N newest (for `time: true`) or first N values | `["5"]` — a single positive integer as a **string** |
| `agg:{name}` | Values come from aggregation `{name}` | `["F-03", "F-11"]` |
| `vs:{name}` | Values come from alternative value set `{name}` | `["01", "02"]` |

Notes verified live against SSB and SCB:

- **Multiple wildcards in one `all` list work** — `{"filter": "all", "values": ["199*", "202*"]}` correctly returns the 1990s plus the 2020s. The official SCB 1.0 specification states only one wildcard is permitted; current builds allow several. Older installations may not — fall back to `item` if you get a 400.
- **`agg:` accepts only explicit codes.** `{"filter": "agg:X", "values": ["*"]}` returns 400. You must enumerate the aggregate codes.
- **Aggregation and valueset names are filenames**, so they may contain spaces and punctuation (`agg:25-years classes`). An aggregation belongs to one valueset, so a name valid on one table is often invalid on another.
- **`?` single-character masking is not supported** in any filter.
- `top` is the tool for **rolling queries** — a stored `top` query keeps returning the newest periods as new data is published, whereas an `item` list of future dates errors out.

#### Elimination — omitting a variable

Leaving a variable out of `"query"` is normal and useful. What you get back depends on its `elimination` flag (rules from the PxWeb 1.0 spec, verified live):

1. `elimination: true` **and** the variable has an elimination value → only that total is returned.
2. `elimination: true` **but** no elimination value → all its values are aggregated into one.
3. `elimination: false` (including when the property is absent) → **all** of the variable's values are returned.

Rule 3 is the one that bites. On SCB's `BefolkManadCKM`, omitting `Alder` (`elimination` absent) returns all **134** age values rather than a total. Omitting a `time` variable returns the entire time series — deliberately used to build queries that never need updating.

An empty query `{"query": [], "response": {…}}` is legal and applies these rules to every variable. On a municipality-level table that typically yields whole-country totals for all periods.

**Eliminated variables disappear from the response entirely** — they are absent from `id` and `dimension`, not present with size 1. The response therefore does not record what was aggregated away, so state it yourself when presenting.

#### Output formats

Set in the body as `"response": {"format": "…"}`.

| Format | Notes |
|---|---|
| `json-stat2` | **Recommended.** json-stat2 v2.0. Rich metadata, logical element order, handles large extracts. |
| `json-stat` | json-stat v1.2. Legacy; random element order, struggles with the largest datasets. |
| `csv3` | Codes only, comma-separated, first row is variable codes + table id. Most robust CSV. |
| `csv2` | Pivoted, human-readable texts, one value per row. |
| `csv` | Legacy, single-header layout. Not recommended. |
| `px` | PC-Axis PX. **The default when `response` is omitted.** |
| `xlsx` | Excel. Avoid for large extracts — prone to timeouts. |
| `json` | PX-JSON: `{"columns": […], "data": [{"key": […], "values": […]}]}`. |
| `sdmx` | SDMX-ML. |

**Format names are hyphenated.** Live SSB and SCB both accept `json-stat2` and `json-stat` and both reject `jsonstat2` and `jsonstat` with `400`. The published PxWeb 1.0 specification and several agency guides give the unhyphenated spellings; they are outdated. Availability also varies — some agencies disable `px`, `json` and `sdmx` even though PxWeb implements them.

CSV conventions differ from agency "ready-made dataset" APIs: in PxWebApi, CSV2/CSV3 use comma as field separator, `.` as decimal separator, and quote text fields. Decimal separator is `.` in every format and language, except Excel output in some locales.

### Step 5: Present results

- Display data in a clean markdown table.
- **Always** cite every table used — e.g. "Source: {Agency}, table {id}". Never omit a source table when combining several.
- State units explicitly. v1 metadata has none, so take them from `dimension.{metric}.category.unit` in the json-stat2 response, or from the metric's `valueTexts`. **Both can be missing** — Statistics Greenland returns no `unit` on any dimension. Then the unit lives only in the table title, and you must name it yourself rather than presenting bare numbers.
- Say which variables you eliminated and what that means ("all ages and both sexes combined") — the response will not show it.
- Check `status` for suppressed or missing values before drawing conclusions.
- Explain the numbers in context, in the user's language.

---

## Limits

Query them with `?config` (Step 1) rather than assuming. The spread across installations is
enormous — verified 2026-08-28:

| Agency | `maxCells` | `maxValues` | Rate limit |
|---|---:|---:|---|
| Statistics Estonia | 25,000,000 | 25,000,000 | 1,000 / 10 s |
| Statistics Faroe Islands | 8,000,000 | 8,000,000 | effectively none |
| Statistics Greenland | 2,000,000 | 1,000,000 | 10,000 / 10 s |
| Statistics Norway (SSB) | 800,000 | 50,000 | 300 / 60 s |
| Statistics Sweden (SCB) | 150,000 | 110,000 | 30 / 10 s |
| **Statistics Finland** | **120,000** | 120,000 | 40 / 60 s |
| Statistics Iceland | 100,000 | **5,000** | 200 / 10 s |

Two traps this table makes visible:

- **A query shaped for one installation can fail on another by two orders of magnitude.** Finland
  and Iceland refuse at roughly a seventh of SSB's ceiling; Estonia would accept thirty times SSB's.
- **`maxValues` can bite before `maxCells`.** Iceland allows 100,000 cells but only 5,000 selected
  values; SSB allows 800,000 cells but only 50,000 values. Selecting `*` on a large variable can
  breach the value ceiling while the cell count still looks safe.

Published agency documentation drifts from these values — SSB's user guide states 30 calls / 60 s
where `?config` reports 300, and Statistics Finland's API page states a 100,000-cell limit where
`?config` reports 120,000 (independently confirmed by bisection: 117,000 accepted, 124,800
refused). **Trust `?config` over the documentation.**

Run large queries **in sequence**, waiting for each response before firing the next. Avoid the
minutes right after a publishing deadline (SSB publishes at 08:00 CET; avoid 07:55–08:15).

---

## Response format

`json-stat2` returns a standard json-stat2 Dataset — the same structure as PxWebApi v2, Eurostat and the World Bank. See `references/json-stat2.md` for the Dataset shape, row-major indexing, `role`, and status codes.

Special-value symbols in `status` (data itself is `null`):

| Symbol | Meaning |
|---|---|
| `.` | Not applicable — category did not exist when data was collected |
| `..` | Data not available |
| `:` | Confidential — withheld to avoid identifying a person or business |

Exact symbols vary by agency; check the response.

---

## Pitfalls — never

- Try to fetch data with GET — v1 has no GET data endpoint; a GET returns metadata.
- Omit `"response"` and then expect JSON — the default is PX.
- Use `jsonstat2`/`jsonstat` as format names — they are rejected; use `json-stat2`/`json-stat`.
- Assume variable codes (`Region`, `ContentsCode`, `Tid`) carry across installations — read metadata every time.
- Assume `?query=` search exists — several installations return 400.
- Assume an omitted variable is summed — if `elimination` is false or absent, you get *all* its values.
- Combine `agg:` with `*`, or use `?` masking — both fail.
- Present data without units, or without saying which dimensions were collapsed.
- Fetch a whole large table without filters — the cell limit rejects it with 403.

---

## Troubleshooting

See `references/troubleshooting.md` for verified HTTP codes, exact error payloads, and how to tell v1's three distinct error messages apart.

---

## PX files behind the API

Most installations are file-based, which is why `.px` appears in their URLs and why their
classifications are undiscoverable. `references/px-files-and-classifications.md` covers the
`.vs`/`.agg` file structure, how filter names derive from filenames, and which PX-file keywords
produce the metadata you see — including `ELIMINATION`, whose two forms are exactly the first two
elimination rules, and `AGGREGALLOWED=NO`, which can forbid aggregation on a table with no
API-visible sign.

---

## Fallback

If the API is unavailable or the query cannot be expressed, refer the user to the agency's web statistical database. Most PxWeb front ends offer an "API query for this table" button that emits a ready-made v1 query body — the fastest way to obtain aggregation names that metadata does not expose.
