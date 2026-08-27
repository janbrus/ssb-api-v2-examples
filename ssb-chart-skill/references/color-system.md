# SSB Fargesystem — Komplett referanse

Alle farger i SSBs designsystem for datavisualisering, med verdier i relevante formater.

Basert på SSBs Plotly Template og designsystem.

**Skillet mellom data-blekk og chrome avgjør rendringsmål.** Selve fargeverdiene under er rendringsmål-uavhengige *data* — det er hva du bruker dem på som avgjør (se SKILL.md → «Rendringsmål»):

- **Data-blekk** — linjer, søyler, punkter, kartflater, heatmap-celler. Alltid SSB-paletten, hardkodet som hex, i **begge rendringsmål**. Canvas-biblioteker kan ikke lese CSS-variabler, så hardkodet hex er riktig her, ikke et unntak.
- **Chrome** — akser, rutenett, bakgrunn, tekst, tooltip, fonter, kort. SSB-verdiene under (`#274247`, `#C3DCDC`, `#FFFFFF`, `#F0F8F9`) gjelder **kun frittstående leveranse**. I en inline chat-widget styres chrome av vertens tokens, ellers brytes mørk modus.

De ferdige tema-objektene nederst (Recharts, Chart.js, matplotlib) blander begge deler i ett objekt. Bruk dem derfor ikke rått i widget — se delingen i hver seksjon.

---

## Primærfarger (identitet)

**Chrome-farger — gjelder kun frittstående leveranse.** Alle fem brukes til tekst, akser, rammer og bakgrunner. I en inline chat-widget erstattes de av vertens tokens; skriv dem ikke inn i widget-DOM-en. Unntak: brukt på selve dataserien er utheving data-blekk og gjelder begge rendringsmål — `#00824D` (SSB Grønn 4) som fokusfarge, `#C3DCDC` (SSB Mørk 2) som nedtoning. Merk at `#C3DCDC` er lys: mot mørk vert-bakgrunn forsvinner nedtonede serier, så nedtone heller med opasitet på seriefargen. Se SKILL.md → «Utheving».

### SSB Mørk 5
Tekst, akser, rammer, header-bakgrunn.

- Hex: `#274247`
- RGB: 39, 66, 71
- CSS: `--ssb-dark-5: #274247;`
- JS: `ssbDark5: '#274247'`
- Python: `ssb_dark_5 = '#274247'`
- openpyxl: `PatternFill(start_color='274247', fill_type='solid')`

### SSB Grønn 4
Primær aksent.

- Hex: `#00824D`
- RGB: 0, 130, 77
- CSS: `--ssb-green-4: #00824D;`
- JS: `ssbGreen4: '#00824D'`
- Python: `ssb_green_4 = '#00824D'`
- openpyxl: `PatternFill(start_color='00824D', fill_type='solid')`

### SSB Hvit
Bakgrunn.

- Hex: `#FFFFFF`
- CSS: `--ssb-white: #FFFFFF;`

### SSB Mørk 1
Lys bakgrunn, zebra-rader (alternativ).

- Hex: `#F0F8F9`
- RGB: 240, 248, 249
- CSS: `--ssb-dark-1: #F0F8F9;`
- JS: `ssbDark1: '#F0F8F9'`
- Python: `ssb_dark_1 = '#F0F8F9'`
- openpyxl: `PatternFill(start_color='F0F8F9', fill_type='solid')`

### SSB Mørk 2
Nedtonet elementer, rutenett.

- Hex: `#C3DCDC`
- RGB: 195, 220, 220
- CSS: `--ssb-dark-2: #C3DCDC;`
- JS: `ssbDark2: '#C3DCDC'`
- Python: `ssb_dark_2 = '#C3DCDC'`

---

## Kategorisk palett

**Data-blekk — gjelder begge rendringsmål.** Dette er den delen av fargesystemet som alltid er SSB-styrt, også i en inline chat-widget der alt annet følger verten.

Bruk i denne rekkefølgen. For N serier, bruk rang 1 til N.

### 1. SSB Grønn (primær dataserie)
- Hex: `#1A9D49`
- RGB: 26, 157, 73
- CSS: `--ssb-cat-1: #1A9D49;`
- JS: `ssbGreen: '#1A9D49'`
- Python: `ssb_green = '#1A9D49'`
- openpyxl: `PatternFill(start_color='1A9D49', fill_type='solid')`

### 2. SSB Blå (sekundær)
- Hex: `#1D9DE2`
- RGB: 29, 157, 226
- CSS: `--ssb-cat-2: #1D9DE2;`
- JS: `ssbBlue: '#1D9DE2'`
- Python: `ssb_blue = '#1D9DE2'`
- openpyxl: `PatternFill(start_color='1D9DE2', fill_type='solid')`

### 3. SSB Gull (tertiær)
- Hex: `#C78800`
- RGB: 199, 136, 0
- CSS: `--ssb-cat-3: #C78800;`
- JS: `ssbGold: '#C78800'`
- Python: `ssb_gold = '#C78800'`
- openpyxl: `PatternFill(start_color='C78800', fill_type='solid')`

### 4. SSB Rosa
- Hex: `#C775A7`
- RGB: 199, 117, 167
- CSS: `--ssb-cat-4: #C775A7;`
- JS: `ssbPink: '#C775A7'`
- Python: `ssb_pink = '#C775A7'`
- openpyxl: `PatternFill(start_color='C775A7', fill_type='solid')`

### 5. SSB Mørk Grønn
- Hex: `#075745`
- RGB: 7, 87, 69
- CSS: `--ssb-cat-5: #075745;`
- JS: `ssbDarkGreen: '#075745'`
- Python: `ssb_dark_green = '#075745'`
- openpyxl: `PatternFill(start_color='075745', fill_type='solid')`

### 6. SSB Mørk Blå
- Hex: `#0F2080`
- RGB: 15, 32, 128
- CSS: `--ssb-cat-6: #0F2080;`
- JS: `ssbDarkBlue: '#0F2080'`
- Python: `ssb_dark_blue = '#0F2080'`
- openpyxl: `PatternFill(start_color='0F2080', fill_type='solid')`

### 7. SSB Mørk Rosa
- Hex: `#A3136C`
- RGB: 163, 19, 108
- CSS: `--ssb-cat-7: #A3136C;`
- JS: `ssbDarkPink: '#A3136C'`
- Python: `ssb_dark_pink = '#A3136C'`
- openpyxl: `PatternFill(start_color='A3136C', fill_type='solid')`

### 8. SSB Mørk Brun
- Hex: `#471F00`
- RGB: 71, 31, 0
- CSS: `--ssb-cat-8: #471F00;`
- JS: `ssbDarkBrown: '#471F00'`
- Python: `ssb_dark_brown = '#471F00'`
- openpyxl: `PatternFill(start_color='471F00', fill_type='solid')`

### 9. SSB Grå ("Annet")
- Hex: `#909090`
- RGB: 144, 144, 144
- CSS: `--ssb-cat-9: #909090;`
- JS: `ssbGray: '#909090'`
- Python: `ssb_gray = '#909090'`
- openpyxl: `PatternFill(start_color='909090', fill_type='solid')`

---

## Palett som arrays/lister

**Gjelder begge rendringsmål** (CSS-variant kun der verten tillater egne custom properties — i widget er hardkodet hex fra JS-arrayet det trygge valget).

### JavaScript / TypeScript
```js
const SSB_CATEGORICAL = [
  '#1A9D49', '#1D9DE2', '#C78800', '#C775A7',
  '#075745', '#0F2080', '#A3136C', '#471F00', '#909090'
];
```

### Python
```python
SSB_CATEGORICAL = [
    '#1A9D49', '#1D9DE2', '#C78800', '#C775A7',
    '#075745', '#0F2080', '#A3136C', '#471F00', '#909090'
]
```

### CSS Custom Properties
```css
:root {
  --ssb-cat-1: #1A9D49;
  --ssb-cat-2: #1D9DE2;
  --ssb-cat-3: #C78800;
  --ssb-cat-4: #C775A7;
  --ssb-cat-5: #075745;
  --ssb-cat-6: #0F2080;
  --ssb-cat-7: #A3136C;
  --ssb-cat-8: #471F00;
  --ssb-cat-9: #909090;
}
```

---

## Sekvensielt (grønn rampe)

**Som data-blekk (choropleth-flater, heatmap-celler): begge rendringsmål** — rampen koder da en verdi og er like SSB-styrt som en seriefarge. **Som chrome (kort-bakgrunner, seksjonstoning): kun frittstående leveranse**, jf. SKILL.md → «Rendringsmål». Merk at trinn 1 (`#ECFEED`) er nær hvit: i widget med mørk modus må du sjekke at lyse trinn fortsatt har ≥3:1 mot vertens bakgrunn, og eventuelt starte rampen på trinn 2.

5-trinns gradient for heatmaps, choropleths og intensitetsskalaer.

| Trinn | Hex | RGB |
|---|---|---|
| 1 (lysest) | `#ECFEED` | 236, 254, 237 |
| 2 | `#B6E8B8` | 182, 232, 184 |
| 3 (midt) | `#1A9D49` | 26, 157, 73 |
| 4 | `#075745` | 7, 87, 69 |
| 5 (mørkest) | `#274247` | 39, 66, 71 |

### Python matplotlib colormap
```python
from matplotlib.colors import LinearSegmentedColormap

ssb_sequential = LinearSegmentedColormap.from_list('ssb_green', [
    '#ECFEED', '#B6E8B8', '#1A9D49', '#075745', '#274247'
])
```

### JavaScript array
```js
const SSB_SEQUENTIAL = ['#ECFEED', '#B6E8B8', '#1A9D49', '#075745', '#274247'];
```

---

## Divergerende (negativ–nøytral–positiv)

**Samme regel som den sekvensielle rampen:** data-blekk i begge rendringsmål, chrome kun frittstående. Nøytralpolen `#F0F8F9` er en lys bakgrunnsfarge og forsvinner mot lys vert-bakgrunn — i widget bør nøytral heller være vertens bakgrunnsfarge eller en transparent celle, slik at bare de to polene er SSB-farger.

For avvik fra gjennomsnitt, referanseverdier eller null.

| Pol | Hex | RGB |
|---|---|---|
| Negativ | `#A3136C` | 163, 19, 108 |
| Nøytral | `#F0F8F9` | 240, 248, 249 |
| Positiv | `#1A9D49` | 26, 157, 73 |

### Python matplotlib colormap
```python
from matplotlib.colors import LinearSegmentedColormap

ssb_diverging = LinearSegmentedColormap.from_list('ssb_diverging', [
    '#A3136C', '#F0F8F9', '#1A9D49'
])
```

---

## Recharts tema-objekt

Objektet er delt i to slik at widget-bruk kan plukke bare den delen som alltid er SSB-styrt.

```jsx
// Data-blekk — gjelder BEGGE rendringsmål
const SSB_SERIES = [
  '#1A9D49', '#1D9DE2', '#C78800', '#C775A7',
  '#075745', '#0F2080', '#A3136C', '#471F00', '#909090',
];

// Chrome — KUN frittstående leveranse (lys modus innebygd)
const SSB_CHROME = {
  background: '#FFFFFF',
  text: '#274247',
  grid: '#C3DCDC',
  tooltip: {
    background: '#274247',
    text: '#FFFFFF',
    borderRadius: 4,
  },
  fonts: {
    title: "'Roboto Condensed', Arial, sans-serif",
    body: "'Open Sans', Arial, sans-serif",
  },
};

// Frittstående leveranse: bruk hele temaet
const SSB_THEME = { colors: SSB_SERIES, ...SSB_CHROME };
```

**I inline chat-widget:** bruk `SSB_SERIES` alene og la `SSB_CHROME` ligge. Verten styler container, tekst og bakgrunn selv, og arver mørk modus automatisk. Der en komponent krever en eksplisitt farge, les vertens token i stedet:

```js
const css = getComputedStyle(document.documentElement);
const textColor = css.getPropertyValue('--text-muted').trim();
const gridColor = css.getPropertyValue('--border').trim();
```

Tokennavnene varierer med verten — sjekk hvilke som faktisk finnes før du bruker dem, og ha en nøytral fallback (f.eks. `|| 'currentColor'`) i stedet for å falle tilbake på SSB-hex.

---

## Chart.js konfigurasjon

Felles styling — diagram-type-spesifikke felter (særlig `beginAtZero` på y-aksen) settes per diagram.

**Gjelder frittstående leveranse.** Alt utenom `backgroundColor`-arrayet er chrome med innebygd lys modus. Chart.js tegner på canvas og arver ingenting fra vertens CSS, så i widget må chrome settes eksplisitt fra vertens tokens — se widget-varianten under konfigurasjonen.

```js
const SSB_CHARTJS_DEFAULTS = {
  color: '#274247',
  backgroundColor: [
    '#1A9D49', '#1D9DE2', '#C78800', '#C775A7',
    '#075745', '#0F2080', '#A3136C', '#471F00', '#909090'
  ],
  borderColor: '#274247',
  font: { family: "'Open Sans', Arial, sans-serif", size: 12 },
  plugins: {
    title: { font: { family: "'Roboto Condensed', Arial, sans-serif", size: 20, weight: 'bold' } },
    tooltip: { backgroundColor: '#274247', titleFont: { size: 14 }, bodyFont: { size: 12 } },
    legend: { labels: { font: { size: 12 } } },  // posisjon settes per diagram (se under)
  },
  scales: {
    x: { grid: { color: '#C3DCDC', lineWidth: 0.5 }, ticks: { color: '#274247' } },
    y: { grid: { color: '#C3DCDC', lineWidth: 0.5 }, ticks: { color: '#274247' } },
    // beginAtZero settes per chart-type — se under
  },
};
```

**Per diagram-type:**

- **Søylediagram:** `scales.y.beginAtZero = true` (alltid — SSB-prinsipp 2).
- **Linjediagram (absolutte tall):** `scales.y.beginAtZero = true`. Avkortet y-akse overdriver trender.
- **Linjediagram (indekser, f.eks. KPI med basis 100):** `scales.y.beginAtZero = false` er greit, men marker basisår tydelig med annotasjon.
- **Legend-posisjon:** `top` eller `right` — aldri `bottom`. Under diagrammet kolliderer legend med kildelinjen og blir lest som en del av den (SKILL.md → «Tekst og merking»). Foretrekk uansett direkte merking av seriene, og bruk legend først når direkte merking gir overlapp.
- **Kildelinje (frittstående leveranse):** bruk `plugins.subtitle` med `position: 'bottom'` og `color: '#909090'` (lever i canvas, følger med ved PNG-eksport). Se `format-guidelines.md` → «Vanilla JS + Chart.js v4». **I widget:** `display: false` — tittel og kilde skrives i chat-svarteksten.

**Widget-variant:** behold `backgroundColor`-arrayet uendret og overstyr chrome med vertens tokens:

```js
const css = getComputedStyle(document.documentElement);
const text = css.getPropertyValue('--text-muted').trim() || 'currentColor';
const grid = css.getPropertyValue('--border').trim() || 'rgba(128,128,128,.25)';

const widgetOptions = {
  ...SSB_CHARTJS_DEFAULTS,
  color: text,
  borderColor: grid,
  font: undefined,  // arv vertens typografi, ikke Open Sans
  plugins: {
    title: { display: false },     // tittel hører til chat-teksten
    subtitle: { display: false },  // kildelinjen hører til chat-teksten
    legend: { position: 'top', labels: { color: text } },
    tooltip: {},                   // vertens default, ikke #274247
  },
  scales: {
    x: { grid: { color: grid }, ticks: { color: text } },
    y: { grid: { color: grid }, ticks: { color: text } },
  },
};
```

`beginAtZero`, `spanGaps: false` og desimalhåndtering settes likt i begge rendringsmål — statistisk integritet er ikke rendringsmål-avhengig.

---

## Python matplotlib theme

**Gjelder kun frittstående leveranse.** Temaet brenner hvit bakgrunn og `#274247`-tekst inn i en bildefil (PNG/PDF) som ikke kan følge vertens mørk modus. Matplotlib er derfor ikke et widget-format — trenger du et diagram i chat-svaret, bruk et DOM-/canvas-bibliotek med vertens tokens i stedet.

```python
def apply_ssb_theme():
    import matplotlib.pyplot as plt

    plt.rcParams.update({
        'font.family': 'sans-serif',
        'font.sans-serif': ['Open Sans', 'Arial', 'Helvetica'],
        'axes.titleweight': 'bold',
        'axes.titlesize': 20,
        'axes.labelsize': 14,
        'axes.labelcolor': '#274247',
        'axes.edgecolor': '#274247',
        'axes.facecolor': '#FFFFFF',
        'axes.grid': True,
        'axes.prop_cycle': plt.cycler(color=[
            '#1A9D49', '#1D9DE2', '#C78800', '#C775A7',
            '#075745', '#0F2080', '#A3136C', '#471F00', '#909090'
        ]),
        'grid.color': '#C3DCDC',
        'grid.linewidth': 0.5,
        'grid.linestyle': '--',
        'xtick.color': '#274247',
        'ytick.color': '#274247',
        'xtick.labelsize': 12,
        'ytick.labelsize': 12,
        'figure.facecolor': '#FFFFFF',
        'text.color': '#274247',
        'legend.fontsize': 12,
        'legend.frameon': False,
    })
```
