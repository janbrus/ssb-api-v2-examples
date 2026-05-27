# json-stat2 — formatreferanse

json-stat2 er et åpent format for statistiske datasett (https://json-stat.org/). Det brukes av PxWebApi v2, men også av andre statistikkleverandører som Eurostat og World Bank. Denne referansen dokumenterer formatet selv — leverandørspesifikke detaljer (publiseringstider, grenser osv.) ligger i `api-details.md`.

---

## Dataset-struktur

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
- **`value`** — Flat array med alle dataverdier lagret i **row-major order** (siste dimensjon i `id` varierer raskest, første varierer saktest — samme konvensjon som C/NumPy). Indeksen beregnes fra `id`, `size` og `dimension.{var}.category.index`: for `size = [s₀, s₁, …, sₙ]` og kategori-indekser `(i₀, i₁, …, iₙ)` er flat-indeksen `i₀·(s₁·s₂·…·sₙ) + i₁·(s₂·…·sₙ) + … + iₙ`.
- **`dimension`** — Detaljert info per variabel med koder (`category.index`), navn (`category.label`), enheter (`category.unit`) og metadata (`extension`)
- **`role`** — Hvilke variabler som har rolle som `time`, `geo` eller `metric`. **Start analyse her:** `role.metric` viser hva som måles (sjekk `dimension.{metric}.category.unit` for enhet/desimaler), `role.time` er tidsdimensjonen, `role.geo` er geografi. Hvis `role.geo` mangler, gjelder dataene typisk hele landet/totalen — ikke spør brukeren. Variabler som er i `id` men ikke i `role` er nedbrytningsdimensjoner.
- **`status`** — Markerer spesielle verdier. Nøkkelen er indeks i value-arrayet. SSB bruker:
  - `"."` = ikke mulig å oppgi tall
  - `".."` = tallgrunnlag mangler
  - `":"` = konfidensielt (vises ikke av hensyn til identifisering)
  - Andre leverandører kan bruke andre symboler — sjekk responsen.
- **`extension`** — Leverandørspesifikk metadata. I json-stat2 kan `extension` forekomme på **to nivåer**:
  - **Dataset/rot-nivå** (`<root>.extension`) — metadata om hele tabellen. Hos SSB: `firstPeriod`, `lastPeriod`, `discontinued`, `nextUpdate`, `contact`, og PX-metadata under `extension.px` med `subject-code`, `subject-area`, `decimals`, `heading`/`stub` (default-pivotering) og `contents` — en kort tabelltittel (f.eks. "07459: Befolkning,") som er nyttig som utgangspunkt for å bygge en presentasjonstittel utvidet med valgte variabler og tidsperiode.
  - **Variabel-nivå** (`dimension.{var}.extension`) — metadata om den enkelte dimensjonen. Hos SSB: `elimination` (kan variabelen utelates?), `eliminationValueCode`, `show`, `codelists` (tilgjengelige `agg_`/`vs_`-kodelister). For statistikkvariabelen (`ContentsCode`) i tillegg: `measuringType` (Stock/Flow/Average), `priceType` (Current/Fixed/NotApplicable), `adjustment` (sesongjustering), `refperiod` (referansetidspunkt) og `alternativeText` — alle indeksert per ContentsCode-verdi.
