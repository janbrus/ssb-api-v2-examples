---
name: generic-pxweb-v2-skill
description: >
  Access official statistics from any PxWebApi v2 installation. Use when querying
  statistical databases from national statistical institutes running PxWebApi v2.
  Trigger on "PxWeb", "PxWebApi", "statistical database", "Statistikdatabasen",
  or similar generic references. For Norwegian (SSB) or Swedish (SCB) statistics
  specifically, prefer the dedicated `ssb-pxwebapi-v2` or `scb-pxwebapi-v2` skills
  which include agency-specific examples and table catalogs.
  Covers table search, metadata, data queries, codelists, and saved queries.
metadata:
  version: "0.10.0"
---

# PxWebApi v2 — Generic Skill

This skill guides you through using PxWebApi v2 to search, explore, and retrieve official statistics. PxWebApi v2 is developed by Statistics Sweden (SCB) and used by multiple national statistical institutes.

## Data integrity — the base rule

Official statistics carry an agency's name. A wrong number under that citation damages trust in the agency, not just in the answer. This rule outranks everything else in this skill:

**Never state a number you have not fetched from the API in this conversation.**

- **No numbers from memory.** If you did not run the query, you do not have the number — including numbers you are confident about. Populations, price indices and unemployment rates all move, and training data has a cutoff.
- **No numbers from other sources in the same answer.** Point the user elsewhere rather than blending.
- **If the API fails, say so.** No estimates, no "roughly". See Fallback.
- **No interpolation or projection.** A period missing from the extract is missing from the answer.
- **Mark your own calculations.** Growth rates, shares and sums are yours, not the agency's — show which fetched numbers they rest on, and keep the API's decimals (`category.unit.decimals`).
- **Show `status` values as they are.** Missing, provisional and confidential values belong in the table, not hidden or replaced by zero.
- **Check `discontinued` and `lastPeriod`.** Tables are closed and the series often continues in a new one. If you use a discontinued table, say so and give the last period.

Not finding the number is a valid answer. An honest "not found", with suggested search terms, beats a plausible number that is wrong.

---

## Known PxWebApi v2 installations

| Agency | Country | Base URL | Languages |
|---|---|---|---|
| Statistics Norway (SSB) | Norway | `https://data.ssb.no/api/pxwebapi/v2` | no, en |
| Statistics Sweden (SCB) | Sweden | `https://statistikdatabasen.scb.se/api/v2` | sv, en |

More agencies are expected to migrate to v2. These agencies were checked on 2026-08-30 and are **still v1-only** — no v2 endpoint responds at any of the usual URL patterns. Use `generic-pxweb-v1-skill` for them:

| Agency | Country | v1 base URL |
|---|---|---|
| Statistics Finland | Finland | `https://pxdata.stat.fi/PXWeb/api/v1/{fi\|sv\|en}/StatFin` |
| Statistics Iceland | Iceland | `https://px.hagstofa.is/pxis/api/v1/is/{database}` |
| Statistics Faroe Islands | Faroe Islands | `https://statbank.hagstova.fo/api/v1/{fo\|en}/H2` |
| Statistics Greenland | Greenland | `https://bank.stat.gl/api/v1/{da\|en\|kl}/Greenland` |
| Statistics Estonia | Estonia | `https://andmed.stat.ee/api/v1/{et\|en}/stat` |

Note that SSB and SCB each still run their v1 API alongside v2 (`data.ssb.no/api/v0/`, `api.scb.se/OV0104/v1/doris/`), so a v1 URL from a user's old script is not evidence that the agency lacks v2.

**Which version is this installation?**

1. `GET {base}/config` — a v2 installation returns a JSON object with `apiVersion`, `maxDataCells`, `dataFormats`.
2. If that fails, try v1's config, which is a **query parameter, not a path**: `GET {v1_base}?config`. The v1 path form `/config` returns 400.

**Check that the body parses as JSON — do not trust the status code.** Statistics Greenland answers `GET /api/v2/config` with **HTTP 200**, `Content-Type: text/html`, and the body `CONFIG404;…<br>Filen findes ikke - File not found`. A version check that only looks at the status code concludes "v2 exists" and every subsequent call fails confusingly.

**For SSB or SCB queries, prefer the dedicated sibling skills** (`ssb-pxwebapi-v2`, `scb-pxwebapi-v2`) — they have agency-specific examples, curated table catalogs, and operational details (publishing times, metadata systems) that this generic skill does not cover.

**Important:** Each installation has its own limits (cell count, rate limiting), table IDs, variable codes, and codelists. Always check `/config` and metadata for the specific installation you are querying.

## API endpoints

| Endpoint | Method | Purpose |
|---|---|---|
| `/tables` | GET | Search and list tables |
| `/tables/{id}` | GET | Get info about a single table |
| `/tables/{id}/metadata` | GET | Get metadata (variables, codes, codelists) |
| `/tables/{id}/defaultselection` | GET | Get the table's default selection |
| `/tables/{id}/data` | GET / POST | Get data with filters |
| `/codelists/{id}` | GET | Look up a codelist |
| `/savedqueries` | POST | Create a saved query |
| `/savedqueries/{id}` | GET | Get a saved query |
| `/savedqueries/{id}/data` | GET | Run a saved query and get data |
| `/savedqueries/{id}/selection` | GET | Get the selection of a saved query |
| `/config` | GET | API configuration (limits, formats, languages) |

All endpoints accept the `lang` parameter. Supported languages depend on the installation.

**Which endpoint when?**

- Unknown installation capabilities → `GET {base_url}/config`
- Don't know the table ID → `GET {base_url}/tables?query=…`
- Have table ID, don't know structure → `GET {base_url}/tables/{id}/metadata`
- Need a starting selection for a large table → `GET {base_url}/tables/{id}/defaultselection`
- Know structure, want data → `POST {base_url}/tables/{id}/data` (or GET for a shareable URL)
- Look up a codelist in isolation → `GET {base_url}/codelists/{id}`
- User built a selection in the agency's web UI → copy the "Save" / API URL / POST body directly
- Want a reusable/shareable query → `POST {base_url}/savedqueries`, then `GET {base_url}/savedqueries/{id}/data`

---

## Workflow

Follow these steps in order. Never skip the metadata step.

### Step 1: Identify the installation

Determine which PxWebApi v2 installation the user needs. If unclear, ask. Then set the base URL accordingly.

Always start by checking the configuration:
```
GET {base_url}/config
```
This returns `maxDataCells`, `maxCallsPerTimeWindow`, `timeWindow`, supported languages, and available output formats.

### Step 2: Search for tables

Use `GET {base_url}/tables` with the `query` parameter.

**Search parameters:**

| Parameter | Type | Description |
|---|---|---|
| `query` | string | Free-text search keywords |
| `pastDays` | int | Limit to tables updated in the last N days |
| `includeDiscontinued` | bool | Include discontinued series (default: false) |
| `pageNumber` | int | Page number for pagination |
| `pageSize` | int | Number of results per page |

**Search tips:**
- Use the local language terms of the agency for best results
- `title:` prefix restricts search to the title field
- Truncation: `population*` matches anything starting with "population"
- Boolean operators: `trade AND fish*`
- Check `lastPeriod` and `timeUnit` in results

**Screen candidates from the search hit, before spending a metadata call each.** A `/tables` hit carries more than the title:

| Field | Use |
|---|---|
| `variableNames` | The table's variable labels. Lets you reject "wrong breakdown" candidates without fetching metadata at all — the single biggest call saver when several tables match |
| `firstPeriod` / `lastPeriod` | Coverage. These live here, **not** in the json-stat2 dataset |
| `timeUnit` | `Annual`, `Quarterly`, `Monthly`, `Weekly`, `Other`. The **only** place frequency is published — json-stat2 has no field for it |
| `paths` | Where the table sits in the agency's subject hierarchy, as full breadcrumb arrays. Useful for finding sibling tables and for explaining provenance |
| `discontinued` | Whether the series has stopped |
| `source`, `subjectCode`, `updated`, `category` | Attribution and freshness |

Present the 3–5 most relevant hits with table ID, title, last period, time frequency, and `discontinued` status.

### Step 3: Explore metadata

Use `GET {base_url}/tables/{id}/metadata` to understand the table structure.

Metadata is returned in json-stat2 format. Focus on:

- **`id` array** — Variable names
- **`size` array** — Number of values per variable
- **`dimension` object** — Detailed info per variable: codes (`category.index`), labels (`category.label`), units (`category.unit`), elimination flag (`extension.elimination`), and available codelists (`extension.codelists`)
- **`role` object** — Which variables have role as `time`, `geo` or `metric`. **Start the analysis here:**
  - `role.metric` shows what is measured. Often named `ContentsCode` in Nordic installations, but check `role.metric` for the actual name — and `category.unit` for unit and decimals.
  - `role.time` is the time dimension.
  - `role.geo` is geography. **If `role.geo` is missing, assume the data covers the installation's entire country or area** — do not ask the user.
  - Remaining variables in `id` are breakdown dimensions (sex, age, industry, etc.).

**Key rules:**

- Variables with `elimination: true` can be omitted from the query. **Read this flag from the metadata response only** — in a *data* response the same field describes the extract you received, not the table's contract, and will mislead you (see `references/json-stat2.md`).
- **Omitting an eliminable variable removes it from the response entirely.** It does not come back as a "total" row — the dimension disappears from `id` and `dimension`, and nothing records that it was collapsed. Note what you omitted; the dataset will not.
- PX has **two** elimination forms, and metadata does not distinguish them:
  - *No total value exists* — the API sums the dimension on the fly when you omit it (e.g. a sex dimension whose only categories are "women" and "men").
  - *A predefined total value exists* in the value set (e.g. a region dimension carrying a "whole country" code). Selecting that code and omitting the variable give the same figure.

  `eliminationValueCode` is the discriminator, but it is **absent from metadata** — a 50-table sweep at SSB found it on no dimension. It appears only in a *data* response that includes the total. So: scan `category.label` for a "Total" / "Whole country" entry, or run a probe query, rather than expecting metadata to tell you.
- Variables with `elimination: false` **MUST** be included, or the query fails with `400 — "Missing selection for mandantory variable"` (the API's own spelling; quote it as-is). This is the sharpest v1/v2 difference: **v1 returns all the variable's values instead of failing.** A v1 habit carried into v2 turns a silently-oversized result into a hard error — which is the better behaviour, but it surprises.
- Variables with `role: time` and `role: metric` are typically never eliminable — verify per installation via the `extension.elimination` flag.
- The metric variable (often named `ContentsCode` in Nordic installations, but check `role.metric` for the actual name) tells you what is measured — check `category.unit` for unit and decimals

**Codelists:** Group values into higher aggregation levels. Two types: Aggregation (`agg_` prefix) maps many-to-one; Valueset (`vs_` prefix) shows an alternative set of values. See `references/codelists-and-filters.md` for details.

**Default selection:** Use `GET {base_url}/tables/{id}/defaultselection` as a starting point for large tables.

**A `GET /data` call with no selection parameters is not an error, and does not return the whole table** — it silently returns the table's *default selection*. Verified on both known installations (2026-08-30): SSB table 07459 returns `size [360, 1, 1]` (360 cells, HTTP 200) and SCB table TAB638 returns `size [290, 1, 1, 2]` (580 cells, HTTP 200).

The dangerous part is what disappears. 07459 has **five** dimensions, but the default selection returns **three** — `Kjonn` and `Alder` were eliminated and summed away, and the response says nothing about it. You get a plausible dataset that answers a different question than you asked. Always build the selection yourself.

### Step 4: Build and run query

PxWebApi v2 supports **both GET and POST** for data retrieval. You can also use the agency's web interface to build queries graphically — look for a "Save" or "API query" option to get ready-made GET URLs and POST bodies.

#### POST (recommended for complex queries)

The examples below use `Region`, `ContentsCode` and `Tid` — these are Nordic naming conventions (SSB/SCB). For other installations, substitute the actual variable names from `role.geo`, `role.metric` and `role.time` in the metadata response.

```
POST {base_url}/tables/{id}/data?outputFormat=json-stat2
Content-Type: application/json

{
  "selection": [
    { "variableCode": "Region", "valueCodes": ["01"] },
    { "variableCode": "ContentsCode", "valueCodes": ["Population"] },
    { "variableCode": "Tid", "valueCodes": ["top(5)"] }
  ]
}
```

#### GET (simpler queries, shareable URLs)

```
GET {base_url}/tables/{id}/data?valueCodes[Region]=01&valueCodes[ContentsCode]=Population&valueCodes[Tid]=top(5)&outputFormat=json-stat2
```

#### Filter expressions in valueCodes

Key patterns: `top(N)` = last N values, `from(value)` = from and including, `range(from,to)` = interval, `*` = all values. Wildcards `*` and `?` can be used for pattern matching. See `references/codelists-and-filters.md` for complete syntax.

**For the time dimension, prefer `top(N)`/`from(value)` over `range(from,to)` and explicit periods** — relative filters capture new periods automatically, so shareable URLs and saved queries stay current instead of freezing on whatever was latest when they were written. Use `range()` only when a closed interval is genuinely the point.

#### Output formats

| Format | Value | Use |
|---|---|---|
| json-stat2 | `json-stat2` | Default, machine-readable, rich metadata |
| CSV | `csv` | Simple tabular format |
| Excel | `xlsx` | For end users |
| HTML | `html` | Table for display |
| PX | `px` | Traditional PX format |
| JSON-PX | `json-px` | JSON variant of PX |
| Parquet | `parquet` | Columnar binary, for large extracts into pandas/Arrow |

**Note:** Not all formats are available at every installation — `parquet` in particular is not universal (SSB serves it, SCB does not). Check `/config` for `dataFormats` before offering one.

**OutputFormatParams** (can be combined): `UseCodes`, `UseTexts`, `UseCodesAndTexts`, `IncludeTitle`, `SeparatorTab` / `SeparatorSpace` / `SeparatorSemicolon`.

**Important limits:**
- Check `/config` for `maxDataCells` — varies widely between installations (tens of thousands to hundreds of thousands of cells per query); `/config` is authoritative
- Rate limiting: check **both** `/config` (`maxCallsPerTimeWindow`, `timeWindow`) **and** the `x-ratelimit-*` response headers — installations use one or the other, and `0` in `/config` means "not in use", not "unlimited". See `references/api-details.md`
- GET URLs cannot exceed ~2,100 characters — use POST for complex queries
- Start narrow — it's easier to expand than to handle too much data

### Step 5: Present results

- Display data in a clean markdown table
- **Say which dimensions you collapsed and which codelist you used — the response records neither.** An omitted dimension vanishes from the dataset, and a `codelist[Var]=agg_…` request comes back with no trace of the aggregation that produced the codes. Two different aggregations of the same variable yield datasets that look identical and are not. If the reader cannot reconstruct your query from your answer, the extract is not reproducible
- Show mandatory notes. `extension.noteMandatory` marks entries in the root `note` array that the agency requires be displayed — typically a base-year change, a correction, or a discontinued table with a successor. They travel with the data response, so showing them costs nothing
- **Always** include source attribution listing **every table ID used** — if multiple tables were combined, list all of them (e.g. "Source: {Agency}, tables {id1}, {id2}, …"); never omit a source table
- Explain what the numbers mean in context — in the user's language
- Present units clearly (count, percent, index, currency)
- Offer to visualize the data or download in another format. If the data came from SSB and the `ssb-chart-skill` skill is available in the environment, use it for the visualization — it carries SSB's official palette, typography and source-line rules

### Step 6: Saved queries (optional)

To create a shareable, reusable query:

```
POST {base_url}/savedqueries
Content-Type: application/json

{
  "tableId": "{id}",
  "language": "en",
  "selection": {
    "selection": [
      { "variableCode": "Region", "valueCodes": ["01"] },
      { "variableCode": "ContentsCode", "valueCodes": ["Population"] },
      { "variableCode": "Tid", "valueCodes": ["top(5)"] }
    ]
  },
  "outputFormat": "json-stat2",
  "outputFormatParams": []
}
```

Both `outputFormat` and `outputFormatParams` are required in the savedqueries body — pass `outputFormatParams: []` if you don't need any. Include all non-eliminable variables (with `elimination: false`) in the selection, otherwise the API returns HTTP 400.

Useful for reports that are updated regularly — `top(N)` always returns the latest periods.

---

## Response format

Both metadata and data are returned as **json-stat2** by default. See `references/json-stat2.md` for the Dataset structure, row-major indexing and status codes (format spec — also applies to non-PxWeb providers like Eurostat). See `references/api-details.md` for PxWebApi-specific configuration.

---

## Pitfalls — never

- Fetch data without filters and assume you got the table — you get the *default selection*, HTTP 200, with eliminable dimensions summed away unannounced (see Step 3)
- Assume table IDs, variable codes, or codelists are the same across installations — always re-check `/config` and metadata when switching installation
- Mix codes from different codelists
- Present data without units
- Ignore the `status` field — it may indicate missing or confidential values

---

## Troubleshooting

See `references/troubleshooting.md` for common errors and solutions.

---

## Fallback

If the API is not available, refer the user to the agency's web-based statistical database.
