# API details (PxWebApi v2)

For the json-stat2 response format (Dataset structure, indexing, status codes) — see `json-stat2.md`. This file covers PxWebApi-specific configuration.

---

## Checking API configuration

Use `GET {base_url}/config` to see current limits. The response shape is the same across installations, but the concrete values are installation-specific. Full response, verified 2026-08-30 (this one from SSB — yours will differ):

```json
{
  "apiVersion": "2.3.2",
  "appVersion": "2.5.0+build.30",
  "languages": [{"id": "no", "label": "Norsk"}, {"id": "en", "label": "English"}],
  "defaultLanguage": "no",
  "maxDataCells": 800000,
  "maxCallsPerTimeWindow": 0,
  "timeWindow": 0,
  "license": "https://www.ssb.no/en/diverse/lisens",
  "sourceReferences": [
    {"language": "en", "text": "Source: Statistics Norway"},
    {"language": "no", "text": "Kilde: Statistisk sentralbyrå"}
  ],
  "defaultDataFormat": "json-stat2",
  "dataFormats": ["json-stat2", "csv", "px", "xlsx", "html", "json-px", "parquet"],
  "features": [{"id": "CORS", "params": [{"key": "enabled", "value": "True"}]}]
}
```

Three things are worth acting on:

- **`sourceReferences` gives you the agency's own citation string, per language.** Use it verbatim in the source line instead of inventing a wording — it is what the agency wants to be cited as.
- **`maxDataCells` varies by more than 5×** between the two known installations (SSB 800 000, SCB 150 000). Never carry a limit across installations.
- **`dataFormats` is per installation.** SSB serves `parquet`; SCB does not. Read the list before offering a format.

Fields to expect (values vary — never hardcode):

- `apiVersion` / `appVersion` — the PxWebApi contract version and the deployed build. Both installations run API 2.3.2 on different app versions
- `license` — the licence the data is published under
- `sourceReferences` — the agency's citation string per language
- `features` — optional capabilities, e.g. CORS

- `maxDataCells` — upper bound on cells per query
- `maxCallsPerTimeWindow` / `timeWindow` — rate-limit window in seconds. **`0` means "not in use", not "unlimited"** — see below
- `defaultLanguage` / `languages` — language IDs accepted by `lang=` parameter
- `defaultDataFormat` / `dataFormats` — formats accepted by `outputFormat=` parameter

---

## Rate limiting — check both places

An installation may announce its rate limit in `/config`, in HTTP response headers, or in one and not the other. The two known installations do it in **opposite** ways (verified 2026-08-30):

| Installation | `/config` | Response headers |
|---|---|---|
| SSB (Norway) | `maxCallsPerTimeWindow: 0`, `timeWindow: 0` | `x-ratelimit-limit: 40`, `x-ratelimit-policy: 40;w=60s`, `x-ratelimit-remaining`, `x-ratelimit-resource` |
| SCB (Sweden) | `maxCallsPerTimeWindow: 30`, `timeWindow: 10` | none |

So `/config` alone is not enough. Read the headers on your first real call, and treat a `0` in `/config` as "this installation announces the limit elsewhere" — never as "no limit".

| Header | Example | Meaning |
|---|---|---|
| `x-ratelimit-limit` | `40` | Max calls in the window |
| `x-ratelimit-policy` | `40;w=60s` | Limit plus window length: 40 calls per 60 seconds |
| `x-ratelimit-remaining` | `38` | Calls left in the current window |
| `x-ratelimit-resource` | `SB_API_1MIN` | The limit bucket the call counts against |

Check `x-ratelimit-remaining` before a batch of calls. On `429 Too Many Requests`, wait for the window to reset before retrying.
