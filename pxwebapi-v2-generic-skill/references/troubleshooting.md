# Troubleshooting PxWebApi v2

Common errors and solutions.

---

## HTTP error codes

### 400 Bad Request

Invalid request. Check the `detail` field in the Problem response.

**Common causes:**
- Unknown variable code — `variableCode` does not match metadata (case-sensitive)
- Invalid value code — code does not exist in the table
- Wrong time format — using `"2024"` in a monthly table (should be `"2024M01"`)
- Too many cells — result exceeds `maxDataCells` from `/config`
- Missing required variable — variable with `elimination: false` is missing from selection
- Invalid codelist ID — codelist does not exist for this variable

**Solution:** Re-fetch metadata and compare variable codes and value codes exactly.

### 403 Forbidden

Request understood but denied. The table may not be available via API.

### 404 Not Found

Resource does not exist. Wrong table ID, codelist ID, or saved query ID.

**Solution:** Use `GET /tables?query=...` to find the correct ID.

### 429 Too Many Requests

Rate-limited. Check `/config` for `maxCallsPerTimeWindow` and `timeWindow`.

---

## Common problems

### Too many cells

Cell count = product of number of values per variable. Solution: limit time with `top(N)`, limit region, use codelists for aggregation, select specific `ContentsCode`, or omit variables with `elimination: true`.

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
