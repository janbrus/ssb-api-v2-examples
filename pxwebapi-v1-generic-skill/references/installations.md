# Known PxWebApi v1 installations

**49 installations**, merged from the two catalogues shipped with the R package `pxweb`
(rOpenGov) and probed live on **2026-08-31**. 43 responded; all 43 also answered `?config`.

## How this relates to the table in `SKILL.md`

The two lists answer different questions and are verified to different depths.

| | `SKILL.md` table (7) | This file (49) |
|---|---|---|
| Base URL given | through DATABASEID — ready to POST against | to the LANGUAGE level only |
| `?query=` support | probed and recorded | **not probed** |
| Verification depth | hierarchy walked, queries run | root endpoint answered JSON, `?config` read |

Use `SKILL.md` when you are about to query. Use this file to answer *"does this agency run
PxWebApi v1, and what are its limits?"* — then walk the hierarchy from the base URL as
described in Step 1.

An entry here is **not** promotable to the `SKILL.md` table without probing `?query=` support
and finding its DATABASEID.

## Reading the tables

| Column | Meaning |
|---|---|
| **Status** | `OK` — the root endpoint returned HTTP 200 with a body that parses as JSON, on 2026-08-31. `400` / `404` — the host answered but the endpoint did not serve the request. `down` — no connection. |
| **Lang** | Codes that fill `[lang]`. |
| **Calls** | `maxCalls`/`timeWindow` from `?config` — e.g. `300/60 s` is 300 requests per 60 seconds. |
| **Values / cells** | `maxValues` / `maxCells` from `?config`. These are **independent ceilings**: `maxValues` counts selected values across variables, `maxCells` counts the product. A query naming tens of thousands of individual codes can fail well inside the cell budget. |
| **URL template** | `[version]` takes `v1` (SSB takes `v0`); `[lang]` takes a language code. |

Example: Statistics Finland in English is
`https://statfin.stat.fi/PXWeb/api/v1/en`.

**`maxCells` is not always present.** Five installations return a `?config` payload without
it — see "What the probing showed". A missing cell limit is not an unlimited one; discover
it by bisection if it matters.

Markers: **‡** present only in the CRAN 0.17.0 catalogue, not in the development one ·
**†** the installation reports `1000000 calls per 1000000 seconds`, which is a "not in use"
sentinel rather than a real limit · **\*** URL corrected against the catalogue after probing

Status reflects the **root endpoint at probe time**. It does not mean the whole API works,
and it says nothing about `?query=` support.

## Nordic countries and autonomous areas (26 of 29 responding)

### Norway (1)

| Status | Organisation | Lang | Calls | Values / cells | URL template |
|---|---|---|---|---|---|
| **OK** | Statistics Norway ‡ (v0) \* | `en`, `no` | 300/60 s | 50 k / 800 k | `https://data.ssb.no/api/[version]/[lang]` |

### Sweden (14)

| Status | Organisation | Lang | Calls | Values / cells | URL template |
|---|---|---|---|---|---|
| **OK** | Linköping municipality in Sweden | `sv` | 1000/10 s | 1 k / 100 k | `https://statistik.linkoping.se/PXWeb/api/[version]/[lang]` |
| **OK** | Statistics Sweden | `en`, `sv` | 30/10 s | 110 k / 150 k | `https://api.scb.se/OV0104/[version]/doris/[lang]` |
| **OK** | Sundsvall municipality in Sweden | `sv` | 10/10 s | 1 k / — | `https://m02-http-pxwebb.login.sundsvall.se/PXWeb_Ext/api/[version]/[lang]` |
| **OK** | Swedish Agency for Growth Policy Analysis (Tillväxtanalys) | `sv` | 10/10 s | 1 k / 100 k | `https://statistik.tillvaxtanalys.se/PxWeb/api/[version]/[lang]` |
| **OK** | Swedish Board of Student Finance | `sv` | 10/10 s | 1 k / 100 k | `https://statistik.csn.se/PXWeb/api/[version]/[lang]` |
| **OK** | Swedish Energy Agency | `en` | 100/10 s | 1 k / 100 k | `https://pxexternal.energimyndigheten.se/api/[version]/[lang]` |
| **OK** | Swedish University of Agricultural Sciences forest statistics | `en` | 10/10 s | 1 k / 300 k | `https://skogsstatistik.slu.se/api/[version]/[lang]` |
| **OK** | The Public Health Agency of Sweden | `sv` | 1000/10 s | 10 k / 100 k | `https://fohm-app.folkhalsomyndigheten.se/Folkhalsodata/api/[version]/[lang]` |
| **OK** | The Swedish Agricultural Agency | `sv` | 1000/10 s | 1 k / 100 k | `https://statistik.sjv.se/PXWeb/api/[version]/[lang]` |
| **OK** | The Swedish Forest Agency | `en` | 10/10 s | 1 k / 120 k | `https://pxweb.skogsstyrelsen.se/api/[version]/[lang]` |
| **OK** | The Swedish National Institute of Economic Research | `en`, `sv` | 10/10 s | 1 k / 100 k | `https://statistik.konj.se/PXWeb/api/[version]/[lang]` |
| **OK** | The Swedish national institute of economic research, forecast database | `sv` | 10/10 s | 1 k / 100 k | `https://prognos.konj.se/PXWeb/api/[version]/[lang]` |
| **OK** | Vasteras municipality in Sweden | `sv` | 30/1 s | 10 k / — | `https://statistik.vasteras.se/api/[version]/[lang]` |
| **OK** | Vastra Gotaland Region in Sweden | `sv` | 100/10 s | 150 k / 150 k | `https://pxweb2022.vgregion.se/Pxwebb/api/[version]/[lang]` |

### Finland (9)

| Status | Organisation | Lang | Calls | Values / cells | URL template |
|---|---|---|---|---|---|
| **OK** | Finnish Centre for Pensions | `en` | 20/5 s | 1 k / 100 k | `https://tilastot.etk.fi/api/[version]/[lang]/ETK` |
| **404** | Finnish Pension Alliance statistics | `en` | — | — | `https://tilastot.tela.fi/api/[version]/[lang]` |
| **OK** | Finnish Transport Safety Agency | `en`, `sv`, `fi` | 100/10 s | 110 k / 110 k | `https://trafi2.stat.fi/PXWeb/api/[version]/[lang]/` |
| **OK** | Helsingin seudun aluesarjat -tilastotietokanta | `fi` | 1000/10 s | 120 k / 120 k | `https://stat.hel.fi/api/[version]/[lang]` |
| **OK** | LUKE Natural Resources Institute Finland | `en`, `fi`, `sv` | 100/10 s | 10 k / 1 M | `https://statdb.luke.fi/PXWeb/api/[version]/[lang]` |
| **OK** | Statistics Finland | `en`, `fi`, `sv` | 40/60 s | 120 k / 120 k | `https://statfin.stat.fi/PXWeb/api/[version]/[lang]` |
| **down** | Statistics Finland (old version) ‡ | `fi` | — | — | `https://pxwebapi2.stat.fi/PXWeb/api/[version]/[lang]` |
| **OK** | Verohallinto - Finnish Tax Administration | `en`, `fi`, `sv` | 100/10 s | 150 k / 150 k | `https://vero2.stat.fi/PXWeb/api/[version]/[lang]` |
| **OK** | Visit Finland (Rudolf service) | `fi` | 30/10 s | 110 k / 110 k | `https://visitfinland.stat.fi/PXWeb/api/[version]/[lang]` |

### Åland (1)

| Status | Organisation | Lang | Calls | Values / cells | URL template |
|---|---|---|---|---|---|
| **OK** | Statistics Aland | `en`, `sv` | 10/10 s | 1 k / 100 k | `https://pxweb.asub.ax/PXWeb/api/[version]/[lang]/` |

### Iceland (2)

| Status | Organisation | Lang | Calls | Values / cells | URL template |
|---|---|---|---|---|---|
| **down** | Icelandic Centre for Retail Studies ‡ | `en`, `is` | — | — | `http://px.rsv.is/PXWeb/api/[version]/[lang]` |
| **OK** | Statistics Iceland | `en`, `is` | 100/10 s | 5 k / 100 k | `https://px.hagstofa.is/px[lang]/api/[version]/[lang]` |

### Greenland (1)

| Status | Organisation | Lang | Calls | Values / cells | URL template |
|---|---|---|---|---|---|
| **OK** | Statbank Greenland | `en`, `kl`, `da` | 10000/10 s | 1 M / 2 M | `https://bank.stat.gl/api/[version]/[lang]` |

### Faroe Islands (1)

| Status | Organisation | Lang | Calls | Values / cells | URL template |
|---|---|---|---|---|---|
| **OK** | Statistics Faroe Islands | `en`, `fo` | none † | 8 M / 8 M | `https://statbank.hagstova.fo:443/api/[version]/[lang]` |

## Rest of Europe (12 of 13 responding)

### Croatia (1)

| Status | Organisation | Lang | Calls | Values / cells | URL template |
|---|---|---|---|---|---|
| **OK** | Croatian Bureau of Statistics | `en` | 10/10 s | 100 k / 100 k | `https://web.dzs.hr/PXWeb/api/[version]/[lang]` |

### Estonia (2)

| Status | Organisation | Lang | Calls | Values / cells | URL template |
|---|---|---|---|---|---|
| **OK** | Estonia - official statistics | `en`, `et` | 1000/10 s | 25 M / 25 M | `https://andmed.stat.ee/api/[version]/[lang]` |
| **OK** | Estonian Health Statistics and Health Research Database | `en` | 100/5 s | 10 k / 1 M | `https://statistika.tai.ee/api/[version]/[lang]` |

### Georgia (1)

| Status | Organisation | Lang | Calls | Values / cells | URL template |
|---|---|---|---|---|---|
| **OK** | Geostat Statistics Database \* | `en` | 30/1 s | 10 k / — | `https://pc-axis.geostat.ge/PXweb/api/[version]/[lang]/Database` |

### Kosovo (1)

| Status | Organisation | Lang | Calls | Values / cells | URL template |
|---|---|---|---|---|---|
| **OK** | Statistics Kosovo | `en` | 100000/10 s | 1 k / 100 k | `https://askdata.rks-gov.net/api/[version]/[lang]/` |

### Latvia (2)

| Status | Organisation | Lang | Calls | Values / cells | URL template |
|---|---|---|---|---|---|
| **OK** | Latvia - official statistics | `en`, `lv` | 100/10 s | 4 k / 100 k | `https://data.stat.gov.lv/api/[version]/[lang]/` |
| **OK** | Latvian Health Statistics Database | `en` | 10/10 s | 1 k / 100 k | `https://statistika.spkc.gov.lv/api/[version]/[lang]/Health` |

### Liechtenstein (1)

| Status | Organisation | Lang | Calls | Values / cells | URL template |
|---|---|---|---|---|---|
| **OK** | Statistics Liechtenstein | `en` | 10/10 s | 2 k / — | `https://etab.llv.li/PXWeb/api/[version]/[lang]/eTab` |

### Moldova (1)

| Status | Organisation | Lang | Calls | Values / cells | URL template |
|---|---|---|---|---|---|
| **OK** | Statistics Moldova | `en`, `ro` | 10/10 s | 1 k / 100 k | `https://statbank.statistica.md/pxweb/api/[version]/[lang]/` |

### North Macedonia (1)

| Status | Organisation | Lang | Calls | Values / cells | URL template |
|---|---|---|---|---|---|
| **OK** | State Statistical Office of the Republic of Macedonia | `en`, `mk` | 10/10 s | 1 k / 100 k | `https://makstat.stat.gov.mk/PXWeb/api/[version]/[lang]` |

### Slovenia (1)

| Status | Organisation | Lang | Calls | Values / cells | URL template |
|---|---|---|---|---|---|
| **OK** | SiStat Database | `sl` | 500/10 s | 50 M / 50 M | `https://pxweb.stat.si/SiStatData/api/[version]/[lang]` |

### Spain (1)

| Status | Organisation | Lang | Calls | Values / cells | URL template |
|---|---|---|---|---|---|
| **400** | Judicial statistics, Spain | `es` | — | — | `https://www6.poderjudicial.es/PxWeb-20252-v1/api/[version]/[lang]` |

### Switzerland (1)

| Status | Organisation | Lang | Calls | Values / cells | URL template |
|---|---|---|---|---|---|
| **OK** | Statistics Switzerland | `en`, `de`, `fr` | 50/15 s | 5 k / 100 k | `https://www.pxweb.bfs.admin.ch/api/[version]/[lang]` |

## Outside Europe (1 of 1 responding)

### Philippines (1)

| Status | Organisation | Lang | Calls | Values / cells | URL template |
|---|---|---|---|---|---|
| **OK** | Philippine Statistics Authority OpenSTAT | `en` | 10/10 s | 1 k / — | `https://openstat.psa.gov.ph/PXWeb/api/[version]/[lang]` |

## International and cross-border (4 of 6 responding)

| Status | Organisation | Lang | Calls | Values / cells | URL template |
|---|---|---|---|---|---|
| **OK** | Generations and Gender Contextual Database | `en` | 10/10 s | 1 k / 100 k | `https://px.web.ined.fr/GGP/api/[version]/[lang]` |
| **OK** | International Renewable Energy Agency | `en` | 10/10 s | 1 k / 100 k | `https://pxweb.irena.org/api/[version]/[lang]` |
| **400** | Nordic Health and Welfare Statistics | `en` | — | — | `https://pxweb.nhwstat.org/Prod/api/[version]/[lang]` |
| **OK** | Nordic Statistics Database | `en` | 100/10 s | 1 M / 1 M | `https://pxweb.nordicstatistics.org/api/[version]/[lang]/` |
| **404** | Portail statistique de la Grande Région | `fr`, `de` | — | — | `https://www.grande-region.lu/pxweb/api/[version]/[lang]` |
| **OK** | United Nations Economic Commission for Europe | `en` | 10/10 s | 100 k / 100 k | `https://w3.unece.org/PXWeb2015/api/[version]/[lang]/` |

---

## What the probing showed

### `?config` answered on every installation that was up

All 43 responding installations served `?config` — with the caveat that the exact form
varies. Both `…/{lang}/?config` and `…/{lang}?config` worked everywhere; Estonia is
documented in `SKILL.md` as needing the form without the trailing slash. For the four
templates carrying a database segment after `[lang]` (`/ETK`, `/Health`, `/Database`,
`/eTab`), `?config` answers identically at the language level and after the segment.

This is the strongest available argument for the Step 1 rule: **always start with
`?config`**. It is not a nice-to-have that some installations happen to support.

### Two catalogue URLs are wrong

| Installation | Catalogue says | Actually works |
|---|---|---|
| `pc-axis.geostat.ge` | `http://…` — answers **404** | `https://…` — answers JSON |
| `data.ssb.no` | `http://…` | redirects to `https://…` |

Both are corrected in the tables above and marked `\*`. Geostat is the instructive case:
the same path over `https` returns JSON while `http` returns 404, so a probe that followed
the catalogue faithfully would have reported the installation dead.

### `maxCells` is missing from five `?config` payloads

`etab.llv.li`, `m02-http-pxwebb.login.sundsvall.se`, `openstat.psa.gov.ph`,
`pc-axis.geostat.ge` and `statistik.vasteras.se` return `maxValues`, `maxCalls`,
`timeWindow` and `CORS` but no `maxCells`. Do not read the absence as "no cell limit" — the
403 cell-limit response still applies.

`CORS` was `true` on all 43. No installation tested restricts browser clients.

### The six that did not respond

| Installation | Status | Detail |
|---|---|---|
| `pxweb.nhwstat.org` | 400 | Body is exactly `Bad Request` — no diagnostic |
| `www6.poderjudicial.es` | 400 | Body is `Solicitud incorrecta` |
| `tilastot.tela.fi` | 404 | XHTML page, not an API |
| `www.grande-region.lu` | 404 | HTML page in French |
| `px.rsv.is` | down | Connect timeout on every variant |
| `pxwebapi2.stat.fi` | down | Hostname does not resolve |

The four that answered were retried over `http` and `https`, with and without a trailing
slash, and in each advertised language. No variant helped.

`www6.poderjudicial.es` carries its deployment name in the path
(`/PxWeb-20252-v1/`), which reads like "2025, release 2". Such paths change on redeployment,
so that entry will go stale independently of whether the service lives.

## Why the limits here come from `?config`, not from the catalogue

The R catalogue also publishes call and value limits. They disagree with `?config` often
enough to be unusable: of the 26 installations present in both, **10 disagree on the call
limit and 10 on the value limit**. Some are far off — `askdata.rks-gov.net` is listed at 10
calls per 10 s where `?config` reports 100 000; `pxweb.nordicstatistics.org` at 1 000 values
where `?config` reports 1 000 000.

The pattern suggests the catalogue records agencies' **published** figures while `?config`
reports what the installation **enforces**. SSB is the clearest case: the catalogue says 30
calls/60 s, which is exactly SSB's documented figure, while `?config` says 300. This skill
already resolves that conflict in `?config`'s favour, and the catalogue's numbers are
therefore not used here at all.

The catalogue also has no `maxCells` and no `CORS` field, so it could not fill these columns
even where it agrees.

## Sources

Two catalogues ship with `pxweb`, and they differ:

| | Development version (GitHub `master`) | CRAN 0.17.0 |
|---|---|---|
| Installations | 46 | 30 |
| URLs | modern — 1 of 46 is `http://` | older — 11 of 30 are `http://` |
| Rate-limit fields | **absent** | present |
| SSB | removed | present, as `v0` |

This file takes URLs and coverage from the development catalogue, adds the three entries
that exist only in CRAN 0.17.0 (`data.ssb.no`, `px.rsv.is`, `pxwebapi2.stat.fi`, marked
`‡`), and takes every limit from live `?config`.

Note that the rate-limit fields are gone in the development catalogue, so
`pxweb_api_catalogue()` will stop reporting limits at the next CRAN release. That is another
reason to read `?config` rather than any catalogue.

To regenerate the raw list:

```bash
# development catalogue
curl -s https://raw.githubusercontent.com/rOpenGov/pxweb/master/inst/extdata/api.json
```

```r
# locally installed catalogue
jsonlite::fromJSON(system.file("extdata", "api.json", package = "pxweb"))
```

`api.json` carries a `citation` field with organisation and address that the package's
console output does not print; the country grouping above is derived from it.

## Adding an entry

Probe before you add. A row needs a status from a live root request and limits from
`?config` — never a limit copied from a catalogue or from an agency's documentation.

```markdown
| **OK** | Organisation name | `en`, `xx` | 30/10 s | 110 k / 150 k | `https://host.example/PXWeb/api/[version]/[lang]` |
```

Keep `[version]` and `[lang]` as placeholders, copy the path exactly including any trailing
slash, and update the counts in both the country heading and the group heading. If the
installation turns out to run v2, it belongs in `generic-pxweb-v2-skill` instead.
