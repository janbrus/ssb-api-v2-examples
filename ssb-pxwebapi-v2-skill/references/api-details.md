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
