# Stilregler per output-format

SSB-tilpassede regler for hvert leveranseformat.

---

## Tallformat — språk-bundet

Tallformat følger språkvalget (samme regel som i `ssb-pxwebapi-v2`):

- Norsk (`lang=no`): mellomrom som tusenskilletegn, komma som desimaltegn → `1 234,5`
- Engelsk (`lang=en`): komma som tusenskilletegn, punktum som desimaltegn → `1,234.5`

API-et returnerer alltid desimal-punktum uavhengig av språk — formater om ved presentasjon. Bruk `Intl.NumberFormat('nb-NO')` / `Intl.NumberFormat('en-GB')` i JS, eller `format(value, '.0f').replace(',', ' ')`-mønster i Python.

---

## React / HTML (Recharts, D3.js, Chart.js)

**Layout:**
- Dashboard: Scorecard-kort øverst, detaljdiagrammer i 2–3 kolonne grid under
- Diagrammer skalerer med container, minimum 300px bredde
- Responsive breakpoints: 1 kolonne under 768px, 2–3 over

**Tooltip:**
- Bakgrunn: `#274247`
- Tekst: `#FFFFFF`
- Border-radius: 4px
- Padding: 8px 12px

**Hover:**
- 70% opacity på ikke-hovrede elementer
- Smooth transition: 150ms

**Fonter:**
- Import fra Google Fonts: Roboto Condensed + Open Sans
```html
<link href="https://fonts.googleapis.com/css2?family=Open+Sans:wght@400;600;700&family=Roboto+Condensed:wght@700&display=swap" rel="stylesheet">
```

**Kilde-footer:**
```jsx
// Velg språk basert på `lang`-parameteren som ble brukt mot API-et
// (se ssb-pxwebapi-v2-skillens språk-seksjon).
const sourceLine = lang === 'en'
  ? `Source: Statistics Norway, table ${tableId}. Last updated: ${updated}.`
  : `Kilde: SSB, tabell ${tableId}. Sist oppdatert: ${updated}.`;
<p style={{ fontSize: '11px', color: '#909090', marginTop: '8px' }}>{sourceLine}</p>
```

---

## Vanilla JS + Chart.js v4

For apper uten React/Vue/build-step. Forutsetter at `formatNumber(value, decimals)` og en JSON-Stat2-til-datasets-funksjon er tilgjengelig (se `jsonstat-to-chart.md` for sistnevnte).

```js
// Tittel + kildelinje bygget fra JSON-Stat2-responsen
const chartTitle = cleanContents(data.extension?.px?.contents);  // strip "<id>:" + trailing ","
const sourceLine = lang === 'en'
  ? `Source: Statistics Norway, table ${tableId}. Last updated: ${updated}.`
  : `Kilde: SSB, tabell ${tableId}. Sist oppdatert: ${updated}.`;

// Datasets med per-serie decimals (se jsonstat-to-chart.md, steg 6)
const yAxisDecimals = datasets.reduce((m, d) => Math.max(m, d.decimals || 0), 0);

new Chart(canvas, {
  type: 'line',
  data: { labels, datasets },
  options: {
    responsive: true,
    maintainAspectRatio: false,
    interaction: { mode: 'index', intersect: false },
    plugins: {
      title: {
        display: true, text: chartTitle, position: 'top',
        color: '#274247', font: { size: 16, weight: 'bold' },
      },
      subtitle: {  // mandatorisk kildelinje — lever i canvas så den følger med ved PNG-eksport
        display: true, text: sourceLine, position: 'bottom',
        color: '#909090', font: { size: 11 },
      },
      legend: { position: 'bottom', labels: { color: '#274247' } },
      tooltip: {
        callbacks: {
          // ctx.dataset.decimals settes per serie i datasets-byggeren — IKKE hardkod
          label: (ctx) => `${ctx.dataset.label}: ${formatNumber(ctx.parsed.y, ctx.dataset.decimals ?? 0)}`,
        },
      },
    },
    scales: {
      x: { grid: { display: false }, ticks: { color: '#274247' } },
      y: {
        // Linjediagram: beginAtZero avhenger av om aksen kan kuttes
        // (alltid for absolutte tall, fleksibel for indekser).
        // Søylediagram: bruk { beginAtZero: true } uten unntak.
        beginAtZero: false,
        grid: { color: '#C3DCDC' },
        ticks: { color: '#274247', callback: (v) => formatNumber(v, yAxisDecimals) },
      },
    },
  },
});
```

**Status-koder som hull:** `spanGaps: false` på hver dataset slik at `null`-verdier (fra status-koder, se `jsonstat-to-chart.md`) faktisk vises som hull, ikke som rette linjer mellom kjente punkter.

**Embedded i en eksisterende app:** Hvis appen allerede har sin egen typografi (header, knapper), behold appens fonter i diagrammet i stedet for å overstyre med Roboto Condensed/Open Sans. SSB-paletten, kildelinjen og beginAtZero-reglene er fortsatt obligatoriske; fontvalg er fleksibelt.

---

## PowerPoint (.pptx)

**Slide-layout:**
- Tittel: 20pt Roboto Condensed Bold, `#274247`, øverst til venstre
- Diagram fyller ~70% av slide-høyde
- Kilde: bottom-left, 10pt, `#909090`
- Bakgrunn: hvit (#FFFFFF)

**Diagrammer:**
- Solid fills — ingen gradienter i søyler
- Bruk SSB kategorisk palett i rekkefølge
- Fjern unødvendige rutenett og rammer
- Direkte merking fremfor legend der mulig

**Tekst:**
- Maksimalt 3 bullet points per slide
- Bruk tall fra diagrammet i teksten for å forsterke budskapet

---

## Excel (.xlsx)

**Generelt:**
- Fjern default Excel rutenett og rammer
- Fryse øverste rad (header)

**Header-rad:**
- Bakgrunn: `#274247`
- Tekst: hvit (#FFFFFF), bold
- Font: Calibri 11pt (Excel-standard, nært Open Sans)

**Datarad:**
- Annenhver rad: `#ECFEED` bakgrunn
- Tekst: `#162327`
- Font: Calibri 11pt

**Tallformat:**
- Norsk standard: mellomrom som tusenskilletegn, komma som desimaltegn
- Format-streng: `#\u00A0##0` (non-breaking space)
- Prosent: `0,0 %`
- Indeks: `0,0`

**Betinget formatering:**
- Heatmaps: sekvensielt grønt (#ECFEED → #274247)
- Positiv/negativ: `#1A9D49` med ▲ / `#A3136C` med ▼
- Databars: `#1A9D49`

**Kildeinfo:**
- Plasser i rad under dataene, slått sammen
- Font: 9pt kursiv, `#909090`
- Tekst (følg `lang`-valget fra API-kallet):
  - Norsk: `Kilde: SSB, tabell {id}. Sist oppdatert: {dato}.`
  - English: `Source: Statistics Norway, table {id}. Last updated: {date}.`

---

## Python / Matplotlib

**Tema:**
```python
apply_ssb_theme()  # Se color-system.md for implementasjon
```

**Kilde** (velg språk basert på `lang`-parameteren som ble brukt mot API-et):
```python
if lang == 'en':
    source_line = f'Source: Statistics Norway, table {table_id}. Last updated: {updated}.'
else:
    source_line = f'Kilde: SSB, tabell {table_id}. Sist oppdatert: {updated}.'
fig.text(0.01, 0.01, source_line, fontsize=9, color='#909090', style='italic')
```

**Figur:**
- `figsize=(10, 6)` som standard
- `tight_layout()` alltid
- `dpi=150` for skjerm, `dpi=300` for print
- Sett `plt.rcParams['font.sans-serif']` for Roboto/Open Sans med Arial som fallback

**Akser:**
- Fjern topp- og høyre-spines: `ax.spines['top'].set_visible(False)`
- Rutenett: stiplet `#C3DCDC`, linewidth 0.5
- Akselinjer: `#274247`, linewidth 0.8

---

## Markdown-tabeller

**Formatering:**
- Høyrejuster tall-kolonner med `---:` i header-separator
- Bruk tusenskilletegn (mellomrom): `5 609 000`
- Desimaltall med komma: `3,2 %`
- Bruk forkortelser for store tall: `720 k`, `1,2 mill.`, `3,4 mrd.`

**Lange tabeller (>20 rader):**
- Vis topp 10 og bunn 10 med `...` mellom
- Inkluder total/gjennomsnitt som siste rad
- Eller: vis kun topp/bunn med note om fullstendige data

**Kilde** (velg språk basert på `lang`-parameteren som ble brukt mot API-et):
```markdown
*Kilde: SSB, tabell 07459. Sist oppdatert: 2026-02-25.*
*Source: Statistics Norway, table 07459. Last updated: 2026-02-25.*
```

**Eksempel:**
```markdown
| Fylke | Innbyggere | Endring |
|---|---:|---:|
| Oslo | 728 714 | +1,3 % |
| Viken | 1 278 432 | +0,8 % |
| ... | | |
| **Norge** | **5 609 000** | **+0,7 %** |

*Kilde: SSB, tabell 07459. Sist oppdatert: 2026-02-25.*
```
