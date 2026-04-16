# Codelists and filter syntax

Reference for codelists and filter expressions in PxWebApi v2.

---

## Codelists

### Two types of codelists

| Type | Prefix | Description |
|---|---|---|
| **Aggregation** | `agg_` | Maps many values to one (e.g. municipalities → regions) |
| **Valueset** | `vs_` | Shows an alternative set of values (e.g. only county codes) |

Codelist IDs vary between installations and tables. Always check metadata.

Examples from SSB (Norway) — other installations will have different IDs:
- `agg_KommFylker` — municipalities aggregated to counties (F- prefix on codes)
- `agg_KommSummer` — municipalities summed with current boundaries (K- prefix on codes)
- `vs_Fylker2024` — only county codes
- `agg_FemAarigGruppering` — 5-year age groups

### Finding available codelists

Codelists are listed in metadata under `dimension.{variable}.extension.codelists`.

### Fetching metadata with a codelist pre-activated

You can fetch metadata with a codelist already applied:
```
GET /tables/{id}/metadata?codelist[Region]=agg_KommFylker
```
The variable then shows the aggregated codes instead of the originals. Useful for discovering which codes to use in your query.

### Using a codelist in a data query

**POST** (example from SSB table 07459):
```json
{
  "selection": [
    {
      "variableCode": "Region",
      "codelist": "agg_KommFylker",
      "valueCodes": ["*"]
    },
    {
      "variableCode": "ContentsCode",
      "valueCodes": ["Personer1"]
    },
    {
      "variableCode": "Tid",
      "valueCodes": ["top(5)"]
    }
  ]
}
```

**GET** (same query):
```
GET /tables/07459/data?valueCodes[Region]=*&codelist[Region]=agg_KommFylker&valueCodes[ContentsCode]=Personer1&valueCodes[Tid]=top(5)
```

`ContentsCode` and time are never eliminable and must always be included.

### Looking up codelist contents

```
GET /codelists/{codelist_id}?lang=en
```

Returns all codes with labels and `valueMap` showing which original codes map to each aggregated code.

### outputValues parameter (with aggregation codelists)

When using an aggregation codelist, `outputValues[variable]` controls what is returned:

| Value | Description | Use case |
|---|---|---|
| `aggregated` | Return aggregated (summed) values | Consistent time series over boundary changes |
| `single` | Return individual values from the codelist without summing | Select a subset of values |

### Codes in aggregation codelists may have prefixes

Aggregation codelists often add prefixes to their codes. For example, in SSB:
- `agg_KommFylker` uses `F-` prefix: `F-03` (Oslo), `F-11` (Rogaland)
- `agg_KommSummer` uses `K-` prefix: `K-0301` (Oslo), `K-3103` (Moss)

Always check the codelist contents to see the actual codes.

---

## Filter expressions in valueCodes

### Function-based filters

| Expression | Description | Example |
|---|---|---|
| `top(N)` | Last N values (newest) | `top(5)` → last 5 periods |
| `bottom(N)` | First N values (oldest) | `bottom(3)` → 3 oldest periods |
| `from(value)` | From and including (inclusive) | `from(2020)` → 2020 onwards |
| `to(value)` | Up to and including (inclusive) | `to(2022)` → up to 2022 |
| `range(from,to)` | Interval (inclusive both ends) | `range(2018,2023)` |

These are used as the **sole element** in the valueCodes array — do not combine with explicit codes.

### Wildcard filters

| Expression | Description | Example |
|---|---|---|
| `*` | All values, or matches zero or more characters | `*` alone = all values; `03*` = codes starting with "03" |
| `?` | Matches exactly one character | `??` = all two-digit codes |

`*` alone in valueCodes means "select all values for this variable". Combined with a codelist, it means "all values in the codelist".

Wildcards can be combined with explicit codes in the same valueCodes array:
```json
{ "variableCode": "Region", "valueCodes": ["0301", "46*"] }
```
(SSB example: Oslo + all municipalities in Vestland county)

### Time formats

The format in valueCodes must match the table's `timeUnit`:

| timeUnit | Format | Example |
|---|---|---|
| Annual | `YYYY` | `"2024"` |
| Monthly | `YYYYMNN` | `"2024M06"` |
| Quarterly | `YYYYKN` | `"2024K2"` |
| Weekly | `YYYYWNN` | `"2024W01"` |

Both SSB and SCB use `K` for quarterly data. Always check metadata for actual time codes at other installations.

---

## Important rules

1. **Function filters are used alone** — `top()`, `from()`, `range()` etc. are the sole element in the array
2. **Wildcards can be combined** with explicit codes in the same array
3. **Codes must match metadata** — always use metadata to see valid codes
4. **Time format varies per table** — check `timeUnit`
5. **Codes are strings** — always in quotes, even purely numeric ones
6. **Codelist IDs are case-sensitive** — use the exact ID from metadata
7. **Codelist codes may have prefixes** — check the codelist contents before querying
