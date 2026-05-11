# json-stat2 — format reference

json-stat2 is an open format for statistical datasets (https://json-stat.org/). It is used by PxWebApi v2 but also by other statistical providers such as Eurostat and World Bank. This file documents the format itself — agency-specific operational details (publishing times, limits, etc.) belong in `api-details.md`.

---

## Dataset structure

Both metadata (`/tables/{id}/metadata`) and data (`/tables/{id}/data`) are returned as a json-stat2 Dataset:

```json
{
  "version": "2.0",
  "class": "dataset",
  "label": "Table title",
  "source": "Agency name",
  "updated": "2024-02-22",
  "id": ["Region", "ContentsCode", "Tid"],
  "size": [1, 1, 5],
  "dimension": { ... },
  "value": [100, 200, 300, 400, 500],
  "role": { "time": ["Tid"], "geo": ["Region"], "metric": ["ContentsCode"] },
  "status": { "3": ".." }
}
```

### Key elements

- **`id`** — Variable names in order
- **`size`** — Number of values per variable (same order as `id`)
- **`value`** — Flat array with all data values, stored in **row-major order** (last dimension in `id` varies fastest, first varies slowest — same convention as C/NumPy). Index is computed from `id`, `size`, and `dimension.{var}.category.index`: for `size = [s₀, s₁, …, sₙ]` and category indices `(i₀, i₁, …, iₙ)`, flat index = `i₀·(s₁·s₂·…·sₙ) + i₁·(s₂·…·sₙ) + … + iₙ`.
- **`dimension`** — Detailed info per variable: codes (`category.index`), labels (`category.label`), units (`category.unit`), metadata (`extension`)
- **`role`** — Which variables have role as `time`, `geo`, or `metric`. **Start your analysis here:** `role.metric` shows what is measured (in PxWeb installations this is typically the `ContentsCode` variable — check `dimension.{metric}.category.unit` for unit and decimals), `role.time` is the time dimension, `role.geo` is geography. If `role.geo` is absent, the data typically refers to the whole country / total — don't ask the user. Variables present in `id` but not in `role` are breakdown dimensions (sex, age, industry, etc.).
- **`status`** — Marks special values. Key is index in the value array. Common symbols include `"."` (not applicable), `".."` (data not available), `":"` (confidential). Exact symbols vary by agency — check the response.
- **`extension`** — Agency-specific metadata; common fields include `firstPeriod`, `lastPeriod`, `discontinued`, contacts.
