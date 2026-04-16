---
name: pxwebapi-v2
description: >
  Access official statistics from any PxWebApi v2 installation. Use when querying
  statistical databases from agencies like Statistics Sweden (SCB), Statistics Norway
  (SSB), or any other institution running PxWebApi v2. Trigger on "Swedish statistics",
  "SCB data", "Nordic statistics", "PxWeb", "PxWebApi", "statistical database",
  "Statistikdatabasen", or similar. Covers table search, metadata, data queries,
  codelists, and saved queries.
---

# PxWebApi v2 — Generic Skill

This skill guides you through using PxWebApi v2 to search, explore, and retrieve official statistics. PxWebApi v2 is developed by Statistics Sweden (SCB) and used by multiple national statistical institutes.

## Known PxWebApi v2 installations

| Agency | Country | Base URL | Languages |
|---|---|---|---|
| Statistics Norway (SSB) | Norway | `https://data.ssb.no/api/pxwebapi/v2` | no, en |
| Statistics Sweden (SCB) | Sweden | `https://statistikdatabasen.scb.se/api/v2` | sv, en |

More agencies are expected to migrate to v2. Check `/config` at the base URL to verify a v2 installation. Do not use v2 endpoints against agencies still running v1 — the API structure is different.

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

---

## Workflow

Follow these steps in order. Never skip the metadata step.

### Step 1: Identify the installation

Determine which PxWebApi v2 installation the user needs. If unclear, ask. Then set the base URL accordingly.

If you are unsure about the installation's limits or available formats, check:
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
- The search covers table titles, variables, and variable values (case-insensitive)
- `title:` prefix restricts search to the title field: `title:population`
- Truncation: `population*` matches anything starting with "population"
- Boolean operators: `trade AND fish*`
- Fuzzy search: `~N` after a word allows N character deviations
- Proximity search: `"foreign trade" ~5` finds words within 5 words of each other
- Date search: `updated:20250908*` or `updated:[20250908 TO 20250912*]`
- Check `lastPeriod` and `timeUnit` in results

Present the 3–5 most relevant hits with table ID, title, last period, time frequency, and `discontinued` status.

### Step 3: Explore metadata

Use `GET {base_url}/tables/{id}/metadata` to understand the table structure.

Metadata is returned in json-stat2 format. Focus on:

- **`id` array** — Variable names
- **`size` array** — Number of values per variable
- **`dimension` object** — Detailed info per variable: codes (`category.index`), labels (`category.label`), units (`category.unit`), elimination flag (`extension.elimination`), and available codelists (`extension.codelists`)
- **`role` object** — Which variables have role as `time`, `geo`, or `metric`

**Key rules:**
- Variables with `elimination: true` can be omitted — they are summed automatically
- Variables with `elimination: false` MUST be included in the query. Time and `ContentsCode` are never eliminable.
- `ContentsCode` tells you what is measured — check `category.unit` for unit and decimals

**Codelists:** Group values into higher aggregation levels. Two types: Aggregation (`agg_` prefix) maps many-to-one; Valueset (`vs_` prefix) shows an alternative set of values. You can fetch metadata with a codelist pre-activated: `GET /tables/{id}/metadata?codelist[Region]=agg_KommFylker`. See `references/codelists-and-filters.md` for details.

**Default selection:** Use `GET {base_url}/tables/{id}/defaultselection` as a starting point for large tables.

### Step 4: Build and run query

PxWebApi v2 supports **both GET and POST** for data retrieval. You can also use the agency's web interface to build queries graphically — look for a "Save" or "API query" option to get ready-made GET URLs and POST bodies.

#### POST (recommended for complex queries)

```
POST {base_url}/tables/{id}/data?outputFormat=json-stat2
Content-Type: application/json

{
  "selection": [
    { "variableCode": "Region", "valueCodes": ["0301"] },
    { "variableCode": "ContentsCode", "valueCodes": ["Personer1"] },
    { "variableCode": "Tid", "valueCodes": ["top(5)"] }
  ]
}
```
(Example uses SSB table 07459 — codes are installation-specific.)

#### GET (simpler queries, shareable URLs)

```
GET {base_url}/tables/07459/data?valueCodes[Region]=0301&valueCodes[ContentsCode]=Personer1&valueCodes[Tid]=top(5)&outputFormat=json-stat2
```

#### Filter expressions in valueCodes

Key patterns: `top(N)` = last N values, `from(value)` = from and including, `range(from,to)` = interval, `*` = all values. Wildcards `*` and `?` can be used for pattern matching (e.g. `46*` = all codes starting with "46"). See `references/codelists-and-filters.md` for complete syntax, including `outputValues` for aggregation control.

#### Output formats

| Format | Value | Use |
|---|---|---|
| json-stat2 | `json-stat2` | Default, machine-readable, rich metadata |
| CSV | `csv` | Simple tabular format |
| Excel | `xlsx` | For end users |
| HTML | `html` | Table for display |
| PX | `px` | Traditional PX format |
| JSON-PX | `json-px` | JSON variant of PX |

**Note:** Not all formats may be available at every installation. Check `/config` for `dataFormats`.

**OutputFormatParams** (can be combined): `UseCodes`, `UseTexts`, `UseCodesAndTexts`, `IncludeTitle`, `SeparatorTab` / `SeparatorSpace` / `SeparatorSemicolon`.

**`heading` and `stub`:** Control pivoting with `heading` (variables in columns) and `stub` (variables in rows) parameters. Useful for csv/html/xlsx output.

**Important limits:**
- Check `/config` for `maxDataCells` — this varies between installations (e.g. SSB: 800,000, SCB: 150,000)
- Rate limiting: `/config` shows `maxCallsPerTimeWindow` and `timeWindow`
- GET URLs cannot exceed ~2,100 characters — use POST for complex queries
- The API always returns decimal point regardless of language — reformat for presentation as needed
- Start narrow — it's easier to expand than to handle too much data

### Step 5: Present results

- Display data in a clean markdown table
- Always include source attribution with agency name and table ID
- Explain what the numbers mean in context — in the user's language
- The API returns decimal point regardless of language — adapt number formatting to the user's locale when presenting
- Present units clearly (count, percent, index, currency)
- Offer to visualize the data or download in another format

### Step 6: Saved queries (optional)

To create a shareable, reusable query:

```
POST {base_url}/savedqueries
Content-Type: application/json

{
  "tableId": "07459",
  "language": "en",
  "selection": {
    "selection": [
      { "variableCode": "Region", "valueCodes": ["0301"] },
      { "variableCode": "ContentsCode", "valueCodes": ["Personer1"] },
      { "variableCode": "Tid", "valueCodes": ["top(5)"] }
    ]
  },
  "outputFormat": "json-stat2"
}
```
(Example uses SSB table 07459 — adapt table ID and codes for your installation.)

Useful for reports that are updated regularly — `top(N)` always returns the latest periods.

---

## Response format

Both metadata and data are returned as **json-stat2** by default. See `references/api-details.md` for the Dataset structure and status codes.

---

## Rules

**Always:**
- Search first — never guess table IDs
- Check metadata before building a query
- Check `/config` when unsure about limits or available formats
- Filter on time to keep datasets manageable
- Show source with agency name and table ID
- Present units clearly
- Check the `elimination` flag — only omit variables that allow it

**Never:**
- Fetch data without filters from large tables
- Assume table IDs, variable codes, or codelists are the same across installations
- Mix codes from different codelists
- Present data without units
- Ignore the `status` field — it may indicate missing or confidential values

---

## Examples

### Population of Oslo (SSB, Norway)

```
1. GET https://data.ssb.no/api/pxwebapi/v2/tables?query=folkemengde
2. GET /tables/07459/metadata
3. POST /tables/07459/data
   { "selection": [
       { "variableCode": "Region", "valueCodes": ["0301"] },
       { "variableCode": "ContentsCode", "valueCodes": ["Personer1"] },
       { "variableCode": "Tid", "valueCodes": ["top(1)"] }
   ]}
→ "As of 1 January 2026, Oslo had 728,714 inhabitants (Source: Statistics Norway, table 07459)"
```

### All counties using codelist (SSB, Norway)

```
1. GET /tables/07459/metadata?codelist[Region]=agg_KommFylker
   → Region now shows county codes with F- prefix (F-03, F-11, F-46 etc.)
2. POST /tables/07459/data
   { "selection": [
       { "variableCode": "Region", "codelist": "agg_KommFylker",
         "valueCodes": ["*"] },
       { "variableCode": "ContentsCode", "valueCodes": ["Personer1"] },
       { "variableCode": "Tid", "valueCodes": ["top(1)"] }
   ]}
→ Population per county. The codelist limits * to county codes only.
```

Note: These examples use SSB-specific table IDs and codes. Other installations will have different IDs and codes — always check metadata.

---

## Licensing

Data licensing varies by agency. Check each agency's terms:
- SSB: Creative Commons CC BY 4.0
- SCB: CC0 (public domain)

---

## Troubleshooting

See `references/troubleshooting.md` for common errors and solutions.

---

## Fallback

If the API is not available, refer the user to the agency's web-based statistical database.
