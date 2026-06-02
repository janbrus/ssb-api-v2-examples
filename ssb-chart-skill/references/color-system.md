# SSB Fargesystem — Komplett referanse

Alle farger i SSBs designsystem for datavisualisering, med verdier i relevante formater.

Basert på SSBs Plotly Template og designsystem.

---

## Primærfarger (identitet)

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

```jsx
const SSB_THEME = {
  colors: ['#1A9D49', '#1D9DE2', '#C78800', '#C775A7', '#075745', '#0F2080', '#A3136C', '#471F00', '#909090'],
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
```

---

## Chart.js konfigurasjon

Felles styling — diagram-type-spesifikke felter (særlig `beginAtZero` på y-aksen) settes per diagram.

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
- **Legend-posisjon:** `bottom` for linjediagram (mer plass til selve linjene), `top` eller `right` for søyler.
- **Kildelinje:** bruk `plugins.subtitle` med `position: 'bottom'` og `color: '#909090'` (lever i canvas, følger med ved PNG-eksport). Se `format-guidelines.md` → «Vanilla JS + Chart.js v4».

---

## Python matplotlib theme

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
