# json-stat2 — format reference

json-stat2 is an open format for statistical datasets (https://json-stat.org/). PxWebApi v1
returns it when you ask for `"response": {"format": "json-stat2"}`, and the structure is the same
one produced by PxWebApi v2, Eurostat and the World Bank. This file documents the format; v1's
request shape lives in `query-syntax.md`.

Two things are specific to v1 and worth stating up front:

- **Only the data response is json-stat2.** A GET on the table URL returns v1's own thin metadata
  shape (`{"title", "variables": [{"code", "text", "values", "valueTexts", "elimination", "time"}]}`),
  not a json-stat2 Dataset. v2 returns json-stat2 for both.
- **The data response carries metadata that v1's metadata endpoint does not** — `role`, units and
  decimals in particular. When metadata leaves you guessing, a minimal probe query is the answer.

---

## Dataset structure

```json
{
  "version": "2.0",
  "class": "dataset",
  "label": "07459: Befolkning, etter region, statistikkvariabel og år",
  "source": "Statistisk sentralbyrå",
  "updated": "2026-02-25T07:00:00Z",
  "note": ["Free-text notes about the table"],
  "id": ["Region", "ContentsCode", "Tid"],
  "size": [1, 1, 3],
  "dimension": { … },
  "value": [724290, 728714, 731500],
  "role": { "time": ["Tid"], "geo": ["Region"], "metric": ["ContentsCode"] },
  "status": { "3": ".." }
}
```

### Key elements

- **`id`** — variable names in order.
- **`size`** — number of values per variable, same order as `id`.
- **`value`** — flat array of all data values in **row-major order**: the last dimension in `id`
  varies fastest, the first varies slowest (the C/NumPy convention). For `size = [s₀, s₁, …, sₙ]`
  and category indices `(i₀, i₁, …, iₙ)`, the flat index is
  `i₀·(s₁·s₂·…·sₙ) + i₁·(s₂·…·sₙ) + … + iₙ`. Category positions come from
  `dimension.{var}.category.index`.
- **`dimension`** — per variable: codes (`category.index`), labels (`category.label`), units
  (`category.unit`), and `extension`.
- **`role`** — which variables are `time`, `geo` and `metric`. **Start your analysis here.**
- **`status`** — special values, keyed by index into `value`; the value itself is `null`.
- **`note`** — table-level caveats. Worth reading before interpreting a series; PxWeb often
  records boundary changes and coverage gaps here.
- **`extension`** — agency-specific metadata.

Older serializers may omit `"version": "2.0"` (Statistics Faroe Islands currently does). The rest
of the structure is unaffected.

---

## Start the analysis with `role`

- **`role.metric`** — what is being measured. In Nordic installations this is usually the
  `ContentsCode` variable, but **check `role.metric` for the actual name** rather than assuming:
  Statistics Finland calls it `contentscode`, and other installations differ again. Read
  `dimension.{metric}.category.unit` for the unit and the number of decimals.
- **`role.time`** — the time dimension.
- **`role.geo`** — geography. **If `role.geo` is missing, the figures cover the installation's
  whole country or area — do not ask the user.** In v1 this happens routinely, because
  eliminating a geographic variable removes it from the response entirely.
- Variables in `id` but not in `role` are breakdown dimensions — sex, age, industry, and so on.

Because v1's metadata endpoint has no `role`, this is also the reliable way to identify the
metric on an unfamiliar installation: run a `top`-1 probe query and read `role` off the result.

---

## What json-stat2 cannot express

Worth knowing when moving between PX and json-stat, because the loss is silent:

- **Time periodicity.** PX declares it explicitly (`TIMEVAL(...)=TLIST(A1|H1|Q1|M1|W1)`, which also
  guarantees consecutive periods). json-stat2 has no standard field for it — `role.time` names the
  time dimension but says nothing about its frequency. PxWebApi v2 works around this with
  `timeUnit` on the table resource, *outside* the json-stat2 document; v1 does not expose it at all.
  Consumers must infer frequency from the shape of the value codes, and those codes are not
  guaranteed to be dates (see `query-syntax.md`).

  Note that the absence is a PxWeb choice rather than a hard limit of the format: PxWeb already
  ships PX header keywords inside the dataset under `extension.px` (see below), and TLIST simply is
  not among them.
- **The aggregation that produced a figure.** Eliminated dimensions vanish from `id` and
  `dimension` entirely, so a dataset does not record what was summed away.
- **Which codelist or valueset a code came from.** In v1 the `agg:`/`vs:` used in the request
  leaves no trace in the response.

---

## `extension.px` — PX keywords inside the dataset

PxWeb adds a non-standard `extension.px` object at the top level of the **data** response,
carrying part of the source PX file's header. Present in both v1 and v2, verified identical for
SSB table 14710:

```json
"extension": {
  "px": { "tableid": "14710", "matrix": "KpiIndMnd", "decimals": 1,
          "aggregallowed": false, "official-statistics": true,
          "heading": ["ContentsCode", "Tid"], "stub": [],
          "subject-code": "pp", "subject-area": "Priser og prisindekser" } },
"contact": [ … ]
```

Useful fields: `aggregallowed` (whether `agg:` can work on this table at all), `decimals`,
`heading`/`stub` (the table's intended pivot layout), `official-statistics`, and the subject
classification. This is the only place several of them appear — the metadata endpoint has none of
them. It is a PxWeb extension, so do not expect it from other json-stat producers.

---

## Units and decimals

```json
"ContentsCode": {
  "label": "statistikkvariabel",
  "category": {
    "index": { "Personer1": 0 },
    "label": { "Personer1": "Personer" },
    "unit":  { "Personer1": { "base": "antall", "decimals": 0 } }
  },
  "extension": {
    "elimination": false,
    "refperiod":     { "Personer1": "1.1." },
    "measuringType": { "Personer1": "Stock" },
    "priceType":     { "Personer1": "NotApplicable" },
    "adjustment":    { "Personer1": "None" }
  }
}
```

`unit.base` is the unit of measurement and `unit.decimals` how many decimals to display.
`extension` frequently carries the reference period, whether the series is a stock or a flow,
whether prices are current or fixed, and whether the series is seasonally adjusted — all of which
belong in a careful presentation of the numbers, and none of which appear in v1 metadata.

---

## Status codes

`status` maps an index in `value` to a symbol; the corresponding `value` entry is `null`.

| Symbol | Meaning |
|---|---|
| `.` | Not applicable — the category did not exist when the data was collected |
| `..` | Data not available — not yet in the database |
| `:` | Confidential — withheld to avoid identifying a person or business |

Exact symbols vary by agency; check the response and the agency's documentation. Never treat any
of them as zero.

---

## Classification links

Some agencies attach classification URNs:

```json
"link": { "describedby": [ { "extension": { "Region": "urn:ssb:classification:klass:104" } } ] }
```

At SSB the trailing number is a Klass id, resolvable as
`https://www.ssb.no/klass/klassifikasjoner/104` or
`https://data.ssb.no/api/klass/v1/classifications/104`. Variable definitions appear as
`urn:ssb:conceptvariable:vardok:2798` → `https://www.ssb.no/a/metadata/conceptvariable/vardok/2798/nb`
(`/nb`, `/nn` or `/en`).

---

## Tooling

Client libraries exist for JavaScript, Python, R, Java and Julia — see https://json-stat.org/.
Useful in practice:

- **Python** — `pyjstat` converts json-stat into pandas DataFrames.
- **R** — `rjstat`; SSB additionally publishes `PxWebApiData` on CRAN, which wraps v1 query
  construction (`ApiData()`) and works from Power BI as well. It is **not** v1-only: since 1.9.0
  it also covers PxWebApi v2 through a separate snake_case interface (`api_data()`,
  `query_url()`, `meta_data()`, …), with a vignette for each version. A user's `PxWebApiData`
  script is therefore not evidence that the installation is on v1 — check the base URL.
- **Node.js** — the json-stat Toolkit's command-line converters (`jsonstat2csv`, `csv2jsonstat`,
  `jsonstat2arrow`, `sdmx2jsonstat`, …). `jsonstat2csv` gives far better control over separators
  and decimal characters than the API's own CSV output.
- **Browsing** — the json-stat explorer at https://json-stat.com/explorer, plus a JSON viewer
  extension (built into Firefox; jsonview.com for Chromium).
