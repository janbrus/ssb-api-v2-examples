# Feilsøking for PxWebApi v2

Vanlige feilscenarier og løsninger, basert på API-ets Problem-respons (RFC 7807).

---

## HTTP-feilkoder

### 400 Bad Request

Ugyldig forespørsel. Sjekk `detail`-feltet i Problem-responsen.

**Vanlige årsaker:**

- **Ukjent variabelkode** — `variableCode` i selection matcher ikke metadata. Variabelkoder er case-sensitive.
- **Ugyldig verdikode** — Koden finnes ikke i tabellen. Sjekk `category.index` i metadata.
- **Feil tidsformat** — Bruker `"2024"` i en månedlig tabell (skal være `"2024M01"`).
- **For mange celler** — Resultatet overstiger `maxDataCells` fra `/config`.
- **Manglende obligatorisk variabel** — Variabel med `elimination: false` mangler fra selection.
- **Ugyldig kodeliste-ID** — Kodelisten finnes ikke for denne variabelen.
- **Blanding av filteruttrykk og koder** — `top()`, `from()`, `range()` skal brukes alene i valueCodes.

**Løsning:** Hent metadata på nytt, sammenlign variabelkoder og verdikoder nøyaktig.

### 403 Forbidden

Forespørselen er forstått men nektet.

**Vanlige årsaker:**

- Tabellen er merket som ikke tilgjengelig via API
- Lagret spørring tilhører en annen bruker/sesjon

### 404 Not Found

Ressursen finnes ikke.

**Vanlige årsaker:**

- Feil tabell-ID (skal være et tall som streng, f.eks. `"07459"`)
- Tabellen er fjernet og erstattet av en ny — søk etter temaet
- Feil kodeliste-ID
- Feil saved query-ID

**Løsning:** Bruk `GET /tables?query=...` for å finne riktig ID.

### 429 Too Many Requests

Rate-limiting. Du har sendt for mange forespørsler.

**Løsning:** Vent og prøv igjen. Sjekk `/config` for `maxCallsPerTimeWindow` og `timeWindow` (i sekunder).

---

## Vanlige problemer

### For mange celler

**Symptom:** 400-feil med melding om at resultatet overstiger cellegrensen.

**Beregning:** Antall celler = produktet av antall verdier per variabel. Eksempel:
- 400 kommuner × 2 kjønn × 100 aldre × 40 år = 3 200 000 celler

**Løsning (i prioritert rekkefølge):**

1. Begrens Tid: `top(5)` i stedet for alle år
2. Begrens Region: velg spesifikke kommuner/fylker, eller bruk kodeliste for fylkesnivå
3. Bruk kodeliste for aldersaggregering: `agg_FemAarigGruppering`
4. Filtrer på ContentsCode: velg kun den måleenheten du trenger
5. Utelat variabler med `elimination: true`

**Tips:** Hent defaultselection først — den er designet for å holde seg innenfor cellegrensen.

### Tomme eller uventede søkeresultater

- Prøv andre søkeord eller synonymer ("folkemengde" vs "befolkning" vs "innbyggere")
- SSB bruker fagtermer: "konsumprisindeks" ikke "KPI", "sysselsatte" ikke "ansatte"
- Bruk `includeDiscontinued=true` hvis du trenger historiske serier
- Bruk `pastDays=30` for å finne nylig oppdaterte tabeller
- Paginering: sjekk `page.totalPages` — det kan finnes flere resultatsider

### Manglende kommuner

Kommunesammenslåinger i 2020 endret mange kommunekoder. For eksempel ble gamle Oppegård (0217) og Ski (0213) til nye Nordre Follo (3020).

**Løsning:**

- Sjekk metadata for gyldige kommunekoder i tabellen
- Bruk kodeliste for "sammenslåtte kommuner" for konsistente tidsserier
- Bruk fylkesaggregering med `agg_KommFylker` for å unngå kommuneproblemer

### NULL-verdier i data

Normalt. Ikke alle kombinasjoner har data. Spesielt vanlig for detaljerte nedbrytninger (kommune × alder × kjønn × næring).

Sjekk `status`-objektet i json-stat2-responsen. SSBs gjeldende standardtegn (fra 2021):
- `"."` = ikke mulig å oppgi tall (kategorien var ikke i bruk)
- `".."` = tallgrunnlag mangler (ikke innkommet eller for usikre til å publiseres)
- `":"` = vises ikke av konfidensialitetshensyn (for å unngå identifisering)

I eldre tabeller (før 2021) kan du også finne: `"..."` = oppgave mangler foreløpig, `"-"` = null, `"*"` = foreløpig tall.

Se https://www.ssb.no/diverse/standardtegn-i-tabeller (engelsk: https://www.ssb.no/en/diverse/standardtegn-i-tabeller)

### Feil tall eller uventede enheter

- Sjekk `ContentsCode` i metadata — tabellen kan ha flere målevarianter
- Sjekk `category.unit` for enhet (f.eks. "personer", "prosent", "1000 kr", "indeks")
- Sjekk om verdiene er indeksert (KPI: 2025=100)
- Sjekk `extension.measuringType` (Stock, Flow, Average)
- Sjekk `extension.priceType` (Current = løpende priser, Fixed = faste priser)
- Sjekk `extension.adjustment` (sesongjustert, arbeidsdagskorrigert)

### Data ser "feil ut" over tid

- Kommunesammenslåinger i 2020 bryter tidsserier på kommunenivå
- Næringsklassifisering (NACE) kan endre seg mellom revisjoner
- KPI-basisår endres periodisk (nå 2025=100)
- Nasjonalregnskapet revideres (foreløpige → endelige tall)

---

## Sjekk API-konfigurasjon

Bruk `GET /config` for å se gjeldende grenser:

```json
{
  "maxDataCells": 800000,
  "maxCallsPerTimeWindow": 30,
  "timeWindow": 10,
  "defaultLanguage": "no",
  "languages": [{"id": "no", "label": "norsk (bokmål)"}, {"id": "en", "label": "English"}],
  "defaultDataFormat": "json-stat2",
  "dataFormats": ["px", "json-stat2", "csv", "xlsx", "html", "json-px"]
}
```

Verdiene kan endre seg — hardkod dem ikke.
