# API details (PxWebApi v2)

For the json-stat2 response format (Dataset structure, indexing, status codes) — see `json-stat2.md`. This file covers PxWebApi-specific configuration.

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

Values vary between installations — never hardcode them.
