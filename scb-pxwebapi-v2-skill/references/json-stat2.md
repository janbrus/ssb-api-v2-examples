# json-stat2 — formatreferens

json-stat2 är ett öppet format för statistiska dataset (https://json-stat.org/). Det används av PxWebApi v2 men också av andra leverantörer som Eurostat och World Bank. Den här filen dokumenterar formatet — leverantörsspecifika detaljer (publiceringstider, gränser m.m.) hör hemma i agentens egen referens.

---

## Dataset-struktur

Både metadata (`/tables/{id}/metadata`) och data (`/tables/{id}/data`) returneras som json-stat2 Dataset:

```json
{
  "version": "2.0",
  "class": "dataset",
  "label": "Tabelltitel",
  "source": "Statistikmyndigheten SCB",
  "updated": "2024-02-22",
  "id": ["Region", "Kon", "Alder", "ContentsCode", "Tid"],
  "size": [1, 1, 1, 1, 5],
  "dimension": { ... },
  "value": [100, 200, 300, 400, 500],
  "role": { "time": ["Tid"], "geo": ["Region"], "metric": ["ContentsCode"] },
  "status": { "3": ".." }
}
```

### Nyckelelement

- **`id`** — Variabelnamnen i ordning
- **`size`** — Antal värden per variabel (samma ordning som `id`)
- **`value`** — Platt array med alla datavärden, lagrad i **row-major order** (sista dimensionen i `id` varierar snabbast, första varierar långsammast — samma konvention som C/NumPy). För `size = [s₀, s₁, …, sₙ]` och kategoriindex `(i₀, i₁, …, iₙ)` är plattindex = `i₀·(s₁·s₂·…·sₙ) + i₁·(s₂·…·sₙ) + … + iₙ`.
- **`dimension`** — Detaljerad info per variabel: koder (`category.index`), etiketter (`category.label`), enheter (`category.unit`), metadata (`extension`)
- **`role`** — Vilka variabler som har roll som `time`, `geo` eller `metric`
- **`status`** — Markerar specialvärden. Nyckeln är index i value-arrayen. Vanliga symboler: `"."` (ej tillämpligt), `".."` (uppgift saknas), `":"` (konfidentiellt). Exakta symboler kan variera mellan leverantörer.
- **`extension`** — Leverantörsspecifik metadata; vanliga fält är `firstPeriod`, `lastPeriod`, `discontinued`, kontakter.
