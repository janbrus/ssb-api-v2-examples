# API details and json-stat2

Reference for the json-stat2 response format.

---

## json-stat2 Dataset structure

Both metadata (`/tables/{id}/metadata`) and data (`/tables/{id}/data`) are returned as json-stat2 Dataset:

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
- **`value`** — Flat array with all data values. Index is computed from `id`, `size`, and `dimension.{var}.category.index`.
- **`dimension`** — Detailed info per variable: codes (`category.index`), labels (`category.label`), units (`category.unit`), metadata (`extension`)
- **`role`** — Which variables have role as `time`, `geo`, or `metric`
- **`status`** — Marks special values. Key is index in the value array. Common symbols: `"."` (not applicable), `".."` (data not available), `":"` (confidential). Exact symbols may vary by agency.
- **`extension`** — `firstPeriod`, `lastPeriod`, `discontinued`, contacts, and PX-specific metadata

---

## Number formatting

The API always returns decimal point (`.`) regardless of language setting. When presenting data to users, reformat according to their locale — the API output should not be shown directly as-is.

---

## Checking API configuration

Use `GET {base_url}/config` to see current limits:

```json
{
  "maxDataCells": 150000,
  "maxCallsPerTimeWindow": 30,
  "timeWindow": 10,
  "defaultLanguage": "sv",
  "languages": [{"id": "sv", "label": "svenska"}, {"id": "en", "label": "English"}],
  "defaultDataFormat": "json-stat2",
  "dataFormats": ["px", "json-stat2", "csv", "xlsx", "html", "json-px"]
}
```
(Example shows SCB configuration — values vary between installations.)

Values vary between installations — never hardcode them.
