# Troubleshooting PxWebApi v2

Common errors and solutions.

---

## HTTP error codes

### 400 Bad Request

Invalid request. **Diagnose from `title`, not from `detail`** — `detail` is usually absent. The payload is a small JSON object with `type`, `title` and `status`; `type` is a plain string, not a URI, so treat it as a category label rather than as an RFC 7807 problem type.

The observed `title` values and what each one means:

| `title` | Cause | Fix |
|---|---|---|
| `Non-existent variable` | A `variableCode` / `valueCodes[…]` key is not in the table | Re-fetch metadata; compare names exactly (case-sensitive) |
| `Non-existent value` | The variable exists, the value code does not | Check `dimension.{var}.category.index`; watch for codelist prefixes |
| `Missing selection for mandantory variable` | A variable with `elimination: false` was omitted | Add it. Note the API's spelling, `mandantory` (sic) — quote it as-is or a log search won't match |
| `Too many cells selected` | Result exceeds `maxDataCells` from `/config` | Narrow the selection — see "Too many cells" below. **This is the one case that also sets `detail`** |

Useful positive finding: this payload shape is **identical across installations** (verified on SSB and SCB, 2026-08-30). Unlike PxWebApi v1 — where several installations return a bare `Bad Request` with no diagnostic and you have to bisect the selection to find the offending part — a v2 400 tells you what went wrong. Read the `title` before changing anything.

**Other causes that surface as one of the titles above:**
- Wrong time format — using `"2024"` in a monthly table (should be `"2024M01"`). Check `timeUnit` on the `/tables` hit; it is not in the json-stat2 document
- Invalid codelist ID — codelist does not exist for this variable
- Missing `OutputFormatParams` in `POST /savedqueries` — the field is required in the request body even when empty. Send `"outputFormatParams": []` if you don't need any. Symptom: `400 — "The OutputFormatParams field is required."`

**Solution:** Re-fetch metadata and compare variable codes and value codes exactly.

### 403 Forbidden

Request understood but denied. The table may not be available via API. **Note:** exceeding the cell limit is a **400** in v2, not a 403 — if you are porting knowledge from PxWebApi v1, where the cell-limit response *is* 403, that mapping does not carry over.

### 404 Not Found

Resource does not exist. Wrong table ID, codelist ID, or saved query ID.

**Solution:** Use `GET /tables?query=...` to find the correct ID.

### 429 Too Many Requests

Rate-limited. Check **both** `/config` (`maxCallsPerTimeWindow`, `timeWindow`) and the `x-ratelimit-*` response headers — an installation may announce the limit in either place, and `0` in `/config` means "not in use", not "unlimited". See `api-details.md`.

---

## Common problems

### Too many cells

Cell count = product of number of values per variable. Solution: limit time with `top(N)`, limit region, use codelists for aggregation, select a specific metric value (the `role.metric` variable — often `ContentsCode` in Nordic installations), or omit variables with `elimination: true`.

**Tip:** Fetch `defaultselection` first — it is designed to stay within the cell limit.

### Empty or unexpected search results

- Try alternative keywords or synonyms in the agency's language
- Use `includeDiscontinued=true` for historical series
- Check pagination: `page.totalPages` may indicate more result pages

### Special values in data

Check the `status` object in the json-stat2 response. Common symbols:
- `"."` = not applicable
- `".."` = data not available
- `":"` = confidential

Exact symbols may vary by agency. Check the agency's documentation for their standard symbols.

### Data looks wrong over time

- Administrative boundary changes (municipality mergers, regional reforms) can break time series — use aggregation codelists for consistent series
- Classification schemes (NACE, ISCED) may change between revisions
- Index base years change periodically
- National accounts are revised (preliminary → final figures)
