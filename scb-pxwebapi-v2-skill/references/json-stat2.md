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
- **`role`** — Vilka variabler som har roll som `time`, `geo` eller `metric`. **Börja analysen här:** `role.metric` visar vad som mäts (kontrollera `dimension.{metric}.category.unit` för enhet/decimaler), `role.time` är tidsdimensionen, `role.geo` är geografi. Om `role.geo` saknas gäller data typiskt hela landet/totalen — fråga inte användaren. Variabler som finns i `id` men inte i `role` är nedbrytningsdimensioner.
- **`status`** — Markerar specialvärden. Nyckeln är index i value-arrayen. Vanliga symboler: `"."` (ej tillämpligt), `".."` (uppgift saknas), `":"` (konfidentiellt). Exakta symboler kan variera mellan leverantörer.
- **`extension`** — Leverantörsspecifik metadata, och den finns på **två nivåer** som inte får förväxlas (verifierat mot SCB 2026-08-30):
  - **Dataset-nivå** (`extension` i roten): `px` (PX-filens nyckelord — `decimals`, `heading`/`stub`, `aggregallowed`, `subject-code`), `contact`, samt `noteMandatory` och `discontinued` när de är satta. `firstPeriod`/`lastPeriod` hör **inte** hemma här — de är fält i `/tables`-träffen, inte i datasetet.
  - **Dimensionsnivå** (`dimension.{var}.extension`): `elimination`, `codelists`, `show`, `refperiod`, `measuringType`, `priceType`, `adjustment`, `alternativeText`, `categoryNoteMandatory`. `measuringType`, `priceType` och `adjustment` avgör hur siffran får presenteras (t.ex. fast pris kontra löpande, säsongrensat eller ej) — läs dem innan du beskriver vad talet betyder.
