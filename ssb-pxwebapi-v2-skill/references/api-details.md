# API-detaljer og json-stat2

Referanse for json-stat2 responsformat og praktisk driftsinformasjon.

---

## json-stat2 Dataset-struktur

Både metadata (`/tables/{id}/metadata`) og data (`/tables/{id}/data`) returneres som json-stat2 Dataset:

```json
{
  "version": "2.0",
  "class": "dataset",
  "label": "07459: Folkemengde, etter region, kjønn og alder 2024",
  "source": "Statistisk sentralbyrå",
  "updated": "2024-02-22",
  "id": ["Region", "Kjonn", "Alder", "Tid"],
  "size": [1, 1, 1, 5],
  "dimension": { ... },
  "value": [693494, 697010, 709037, 716272, 723803],
  "role": { "time": ["Tid"], "geo": ["Region"], "metric": ["ContentsCode"] },
  "status": { "3": ".." }
}
```

### Nøkkelelementer

- **`id`** — Variabelnavnene i rekkefølge
- **`size`** — Antall verdier per variabel (i samme rekkefølge som `id`)
- **`value`** — Flat array med alle dataverdier. Indeksen beregnes fra `id`, `size` og `dimension.{var}.category.index`.
- **`dimension`** — Detaljert info per variabel med koder (`category.index`), navn (`category.label`), enheter (`category.unit`) og metadata (`extension`)
- **`role`** — Hvilke variabler som har rolle som `time`, `geo` eller `metric`
- **`status`** — Markerer spesielle verdier. Nøkkelen er indeks i value-arrayet:
  - `"."` = ikke mulig å oppgi tall
  - `".."` = tallgrunnlag mangler
  - `":"` = konfidensielt (vises ikke av hensyn til identifisering)
- **`extension`** — `firstPeriod`, `lastPeriod`, `discontinued`, kontaktpersoner, og PX-spesifikk metadata (`subject-code`, `subject-area`, `nextUpdate`)

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
