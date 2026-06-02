# JSON-Stat2 → diagram-config — oppskrift

Hvordan gå fra en JSON-Stat2-respons fra SSBs PxWebApi v2 til et chart-bibliotek-config (Chart.js, Recharts, matplotlib …). Selve dataformatet er dokumentert i `ssb-pxwebapi-v2`-skillens `references/json-stat2.md` — denne filen handler kun om mappingen til chart-data.

---

## Steg 1 — Velg akse-dimensjon

For tidsserier (linjediagram) er x-aksen alltid tidsdimensjonen:

```js
const timeDim = data.role?.time?.[0];  // f.eks. "Tid"
if (!timeDim) {
  // Ingen tidsdimensjon → ikke en tidsserie. Bytt til søyle/horisontal eller scorecard.
}
```

Hvis dataene ikke har `role.time`, ikke prøv å lage en linjediagram-tilfelle ut av en non-time dim. Vurder horisontal søyle (rangering) eller scorecard (enkelttall) i stedet.

---

## Steg 2 — Bygg x-akse-labels i riktig rekkefølge

`dimension[timeDim].category.index` mapper kode → numerisk index. **Sorter etter index-verdien**, ikke alfabetisk:

```js
const timeDimObj = data.dimension[timeDim];
const timeCodes = Object.keys(timeDimObj.category.index)
  .sort((a, b) => timeDimObj.category.index[a] - timeDimObj.category.index[b]);
const xLabels = timeCodes.map(code => timeDimObj.category.label?.[code] || code);
```

**Hvorfor ikke alfabetisk:** `"2020M2"` og `"2020M10"` sorterer feil alfabetisk (M10 før M2). SSB returnerer alltid riktig index — bruk den.

---

## Steg 3 — Bygg datasets (serier) fra øvrige dimensjoner

Hver kombinasjon av non-time dimensjonsverdier blir én serie (linje). Beregn flat-index inn i `data.value`-arrayet med formelen i `json-stat2.md` (row-major, `id` + `size` + `category.index`).

```js
const nonTimeDims = data.id.filter(d => d !== timeDim);

// For hver kombinasjon av non-time verdier (kartesisk produkt):
for (const combo of allCombinations(nonTimeDims, data)) {
  const values = timeCodes.map(timeCode => {
    const timeIdx = timeDimObj.category.index[timeCode];
    // Kombinér combo-indekser med timeIdx i samme rekkefølge som data.id,
    // og beregn flat-index. Se json-stat2.md.
    const flatIdx = calculateFlatIndex(/* … */);
    return readValue(data, flatIdx);  // se Steg 5 for status-håndtering
  });
  datasets.push({ label: /* steg 4 */, data: values });
}
```

---

## Steg 4 — Lag serielabels

Slå sammen `category.label[code]` for hver dim i kombinasjonen med ` · ` (mellomrom-punkt-mellomrom) som joiner:

```js
const labelParts = combo.codes.map((code, i) => {
  const dim = data.dimension[nonTimeDims[i]];
  return dim.category.label?.[code] || code;  // fall tilbake til kode hvis label mangler
});
const seriesLabel = labelParts.join(' · ');
// Eksempel: "Hele landet · Menn · 2 år · Personer"
```

**Én-serie-tilfelle:** Hvis det ikke finnes non-time dimensjoner (eller alle har størrelse 1), blir labelen tom. Fall tilbake til renset `data.extension?.px?.contents` (se SKILL.md → «Bygg tittel fra `extension.px.contents`») slik at legenden ikke står tom.

**Aktive kodelister:** Hvis brukeren har valgt `agg_KommFylker`, vil `dimension[Region].category.label` inneholde fylkesnavn (ikke kommunenavn). Bruk samme oppslag — kodelistene reflekteres i `category.label` allerede.

---

## Steg 5 — Status-koder rendres som hull

`data.status` markerer celler med spesielle koder (`"."`, `".."`, `":"` — se `json-stat2.md` for full liste). I diagrammet skal disse vises som hull, ikke som null/0:

```js
function readValue(data, flatIdx) {
  if (data.status && data.status[String(flatIdx)] != null) return null;
  const v = data.value[flatIdx];
  return (v == null || !Number.isFinite(v)) ? null : v;
}
```

I Chart.js: sett `spanGaps: false` på datasettet slik at `null` faktisk lager hull i stedet for å trekke linjer mellom kjente punkter.

**Hvorfor:** Statistisk integritet. Antall innbyggere kan ikke interpoleres — at vi *ikke vet* tallet er en observasjon i seg selv. Aldri drop, interpoler, eller gjenta nabopunkter for å «fylle» dataene.

---

## Steg 6 — Desimaler per metric

Hardkod aldri antall desimaler. SSB oppgir presisjon per ContentsCode-verdi:

```js
const metricDim = data.role?.metric?.[0];          // f.eks. "ContentsCode"
const metricCode = /* hvilken kode fra combo som tilhører metricDim */;
const decimals = data.dimension[metricDim]?.category.unit?.[metricCode]?.decimals
              ?? data.extension?.px?.decimals       // dataset-default
              ?? 0;                                 // siste utvei
```

Lagre `decimals` på datasettet (custom-property — Chart.js bevarer ukjente felter):

```js
datasets.push({ label, data: values, decimals });
```

Bruk det i tooltip-callback og som y-akse-presisjon:

```js
plugins.tooltip.callbacks.label = (ctx) =>
  `${ctx.dataset.label}: ${formatNumber(ctx.parsed.y, ctx.dataset.decimals)}`;

// Y-akse: hvis flere serier har ulik presisjon, bruk maks av dem
const yAxisDecimals = Math.max(...datasets.map(d => d.decimals));
scales.y.ticks.callback = (v) => formatNumber(v, yAxisDecimals);
```

**Edge case — `agg_`-kodeliste på metric:** En aggregert kode finnes ikke i `unit{}`. Slå opp kodelistens `valueMap` for å finne underliggende original-koder; bruk maks desimaler blant dem (eller første — i praksis deler de presisjon). Dokumenter valget med én linje kommentar.

**Eksempler på faktiske desimaler:**
- `Personer1` (befolkning) → `0`
- `KpiIndMnd` (KPI-indeks) → `1`
- `KvPris` (kvadratmeterpris) → `0`
- Valutakurser → `4`

Hardkoding av 2 desimaler gir tooltip som `Personer1: 31 108,00` — feil, og brudd på SSBs presisjons-konvensjon.

---

## Helhetlig eksempel (Chart.js v4)

Se `format-guidelines.md` → seksjonen «Vanilla JS + Chart.js» for et komplett eksempel som setter alle stegene over sammen med SSB-paletten, kildelinje og tittel-rensing.
