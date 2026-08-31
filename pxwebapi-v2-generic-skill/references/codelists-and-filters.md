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

### Where codelists come from, and why mixing them fails

The two prefixes are not two flavours of the same thing — they sit at different levels, and knowing this explains most codelist errors.

Underneath PxWeb, a **valueset** is a named list of value codes for a variable, and an **aggregation** is a set of groups built on top of one specific valueset. In file-based installations these are literally files (`.VS` and `.AGG`) beside the table; in relational installations (SSB, SCB) the database manages them. Either way the relationship is the same:

> **An aggregation is defined on a valueset, not on a variable.**

That single fact is the reason for the "never mix codelists" rule. `agg_X` is valid only while the variable is expressed through the valueset `agg_X` belongs to. Combine an aggregation from one valueset with codes from another and you get a `400`, on a query that otherwise looks well-formed.

Two more consequences worth carrying:

- **Codelist names are not namespaced by table.** Several tables can share one aggregation, and one variable can offer several. This is why copying a codelist name from another table's query so often fails — always read `extension.codelists` for the table in hand.
- **Hierarchical code prefixes are meaningful.** A hierarchical valueset defines its levels by character position, so a code like `01.111` decomposes as `01` → `01.1` → `01.11` → `01.111`. That is why a wildcard such as `01*` lines up cleanly with a classification level instead of matching arbitrarily.

Whether an installation makes aggregation meaningful at all is recorded separately, in `extension.px.aggregallowed` on the dataset — see `json-stat2.md`.

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

Variables with `role: time` and `role: metric` are typically never eliminable and must always be included — verify per installation via the `extension.elimination` flag in metadata. In Nordic installations these are usually named `Tid` and `ContentsCode`.

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

**For the time dimension, prefer `top(N)` and `from(value)` over `range(from,to)` and explicit period codes.** Relative filters pick up new periods automatically, so a shareable GET URL or a saved query keeps returning current data instead of freezing on the periods that happened to be latest when it was written. `range(2018,2023)` is right only when the closed interval is the point — a fixed reporting period, or a comparison against a specific baseline. This matters most for `/savedqueries`, whose whole purpose is to be re-run later.

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
