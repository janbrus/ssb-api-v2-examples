# API-detaljer (PxWebApi v2 hos SSB)

Praktisk driftsinformasjon for SSBs PxWebApi v2. For json-stat2-formatet (Dataset-struktur, indeksering, status-koder) — se `json-stat2.md`.

---

## Praktisk driftsinformasjon

- Nye tall publiseres vanligvis kl. 08.00. Unngå spørringer 07.55–08.15 ved høy belastning.
- Tall som skal revideres kl. 08 vises som 0 eller prikk i tidsrommet 05.00–08.00.
- Metadata oppdateres kl. 05.00 og 11.30 — tabellene er utilgjengelige under oppdatering.
- API-grense: 800 000 celler per uttrekk. Rate limit: 40 spørringer per minutt (per 2026-08-05) — les gjeldende verdi fra `x-ratelimit-*`-responsheaderne (se under), ikke fra `/config`.
- GET-URL kan ikke overstige ca. 2 100 tegn — bruk POST for komplekse spørringer.
- Desimalskilletegn er `.` (punktum) for alle formater unntatt xlsx på norsk (komma).
- Lisens: Creative Commons CC BY 4.0.
- Strukturelle endringer i tabeller dokumenteres på: https://www.ssb.no/statbank/hvordan-bruke-statistikkbanken/endringer-i-statistikkbanktabeller

---

## Rate limit-headere

Rate limiting annonseres i HTTP-responsheadere. Feltene `maxCallsPerTimeWindow`/`timeWindow` står fortsatt i `/config`-responsen, men er nullstilt til `0` og ikke lenger i bruk — `0` betyr **ikke** «ingen grense». Eksempel fra en respons:

| Header                  | Eksempelverdi | Betydning                                        |
| ----------------------- | ------------- | ------------------------------------------------ |
| `x-ratelimit-limit`     | `40`          | Maks antall kall i tidsvinduet                   |
| `x-ratelimit-policy`    | `40;w=60s`    | Grense + vinduslengde: 40 kall per 60 sekunder   |
| `x-ratelimit-remaining` | `39`          | Gjenstående kall i inneværende vindu             |
| `x-ratelimit-resource`  | `SB_API_1MIN` | Grense-ressursen kallet telles mot               |

Sjekk `x-ratelimit-remaining` før batch-uttrekk med mange kall. Ved `429 Too Many Requests`: vent til vinduet nullstilles (inntil 60 sekunder med gjeldende policy) før nytt forsøk — se `troubleshooting.md`. Verdiene kan endres av SSB — les headerne, hardkod dem ikke.

---

## RSS-feeds

Komplement til `/tables?pastDays=…` når du vil overvåke publiseringer uten å polle API-et:

| Feed                                                | Innhold                                                          |
| --------------------------------------------------- | ---------------------------------------------------------------- |
| `https://www.ssb.no/rss/statbank`                   | Siste 5 dagers oppdaterte tabeller i Statistikkbanken            |
| `https://www.ssb.no/rss/statbank/{kortnavn}`        | Siste 90 dagers oppdaterte tabeller for ett statistikk-kortnavn  |
| `https://www.ssb.no/rss/statkal`                    | Kommende statistikkpubliseringer (publiseringskalender)          |

`{kortnavn}` er statistikkens kortnavn — samme som 3. nivå i `paths` fra `/tables`-respons, siste ledd i `ssb.no/<kortnavn>`-URL-er, og verdien av `<ssbrss:shortname>`-elementet i RSS-itemene (f.eks. `arblonn`, `kpi`).

Bruksområder:

- `statkal`-feeden: når brukeren spør "når kommer neste tall for X?"
- `statbank/{kortnavn}`-feeden: overvåk en spesifikk statistikk uten å bygge polling-logikk
- Rot-feeden: vis hva som er publisert nylig på tvers av temaer
