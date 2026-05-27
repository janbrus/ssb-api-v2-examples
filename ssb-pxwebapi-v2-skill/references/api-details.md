# API-detaljer (PxWebApi v2 hos SSB)

Praktisk driftsinformasjon for SSBs PxWebApi v2. For json-stat2-formatet (Dataset-struktur, indeksering, status-koder) — se `json-stat2.md`.

---

## Praktisk driftsinformasjon

- Nye tall publiseres vanligvis kl. 08.00. Unngå spørringer 07.55–08.15 ved høy belastning.
- Tall som skal revideres kl. 08 vises som 0 eller prikk i tidsrommet 05.00–08.00.
- Metadata oppdateres kl. 05.00 og 11.30 — tabellene er utilgjengelige under oppdatering.
- API-grense: 800 000 celler per uttrekk, 30 spørringer per minutt (per IP-adresse).
- GET-URL kan ikke overstige ca. 2 100 tegn — bruk POST for komplekse spørringer.
- Desimalskilletegn er `.` (punktum) for alle formater unntatt xlsx på norsk (komma).
- Lisens: Creative Commons CC BY 4.0.
- Strukturelle endringer i tabeller dokumenteres på: https://www.ssb.no/statbank/hvordan-bruke-statistikkbanken/endringer-i-statistikkbanktabeller

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
