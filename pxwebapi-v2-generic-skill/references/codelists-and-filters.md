# Codelists and filter syntax

Reference for codelists and filter expressions in PxWebApi v2.

---

## Codelists

### Two types of codelists

| Type | Prefix | Description | Example |
|---|---|---|---|
| **Aggregation** | `agg_` | Maps many values to one (e.g. municipalities → regions) | `agg_RegionLevel` |
| **Valueset** | `vs_` | Shows an alternative set of values | `vs_RegionOnly` |

Codelist IDs and prefixes vary between installations and tables. Always check metadata.

### Finding available codelists

Codelists are listed in metadata under `dimension.{variable}.extension.codelists`.

### Using a codelist in a data query

**POST:**
```json
{
  "selection": [
    {
      "variableCode": "Region",
      "codelist": "agg_RegionLevel",
      "valueCodes": ["*"]
    },
    {
      "variableCode": "ContentsCode",
      "valueCodes": ["Population"]
    },
    {
      "variableCode": "Tid",
      "valueCodes": ["top(5)"]
    }
  ]
}
```

**GET:**
```
GET /tables/{id}/data?valueCodes[Region]=*&codelist[Region]=agg_RegionLevel&valueCodes[ContentsCode]=Population&valueCodes[Tid]=top(5)
```

`ContentsCode` and `Tid` are never eliminable and must always be included.

### Looking up codelist contents

```
GET /codelists/{codelist_id}?lang=en
```

Returns all codes with labels and `valueMap` showing which original codes map to each aggregated code.

### outputValues parameter (with aggregation codelists)

When using an aggregation codelist, `outputValues[variable]` controls what is returned:

| Value | Description |
|---|---|
| `aggregated` | Return aggregated (summed) values |
| `single` | Return individual values from the codelist without summing |

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

### Time formats

The format in valueCodes must match the table's `timeUnit`:

| timeUnit | Format | Example |
|---|---|---|
| Annual | `YYYY` | `"2024"` |
| Monthly | `YYYYMNN` | `"2024M06"` |
| Quarterly | `YYYYKN` or `YYYYQN` | Varies by installation |
| Weekly | `YYYYWNN` | `"2024W01"` |

**Note:** Quarterly format may differ between installations. SSB uses `K` (e.g. `2024K2`), SCB uses `K` as well. Always check metadata for actual time codes.

---

## Important rules

1. **Function filters are used alone** — `top()`, `from()`, `range()` etc. are the sole element in the array
2. **Wildcards can be combined** with explicit codes in the same array
3. **Codes must match metadata** — always use metadata to see valid codes
4. **Time format varies per table** — check `timeUnit`
5. **Codes are strings** — always in quotes, even purely numeric ones
6. **Codelist IDs are case-sensitive** — use the exact ID from metadata
7. **Codelist codes may have prefixes** — aggregation codelists often use prefixes on their codes (check the codelist contents)
