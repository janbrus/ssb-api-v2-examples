---
name: scb-pxweb-v2
description: >
  Svensk offentlig statistik från SCB via PxWebApi v2. Använd ALLTID när någon frågar om
  svenska siffror, statistik, befolkning, KPI, inflation, arbetslöshet, löner, priser,
  BNP, ekonomi, handel, export, import, utbildning, hälsa, bostäder, kommun- eller
  länsdata, eller nämner SCB, Statistiska centralbyrån eller Statistikdatabasen.
  Trigger på "hitta siffror på", "hur många bor i", "KPI senaste året", "befolkningsökning",
  "prisindex", "bostadspriser" och liknande. Also trigger on "Swedish statistics",
  "population of Sweden", "Statistics Sweden", "SCB data", "Sweden GDP",
  "inflation in Sweden", "housing prices Sweden" or similar. Använd denna framför
  websökning när svaret finns i svensk offentlig statistik. Täcker kodlistor,
  sparade frågor och outputformat (json-stat2, csv, xlsx).
---

# SCB PxWebApi v2 — Komplett guide

Denna skill guidar dig genom korrekt användning av SCB:s PxWebApi v2 för att söka, utforska och hämta svensk offentlig statistik. API:ets bas-URL är:

```
https://statistikdatabasen.scb.se/api/v2
```

## API-översikt

PxWebApi v2 har följande endpoints:

| Endpoint                        | Metod      | Syfte                                        |
| ------------------------------- | ---------- | -------------------------------------------- |
| `/tables`                       | GET        | Sök och lista tabeller                       |
| `/tables/{id}`                  | GET        | Hämta info om en tabell                      |
| `/tables/{id}/metadata`         | GET        | Hämta metadata (variabler, koder, kodlistor) |
| `/tables/{id}/defaultselection` | GET        | Hämta tabellens förvalda selektion           |
| `/tables/{id}/data`             | GET / POST | Hämta data med filter                        |
| `/codelists/{id}`               | GET        | Slå upp en kodlista                          |
| `/savedqueries`                 | POST       | Skapa en sparad fråga                        |
| `/savedqueries/{id}`            | GET        | Hämta en sparad fråga                        |
| `/savedqueries/{id}/data`       | GET        | Kör en sparad fråga och hämta data           |
| `/savedqueries/{id}/selection`  | GET        | Hämta selektionen för en sparad fråga        |
| `/config`                       | GET        | API-konfiguration (gränser, format, språk)   |

Alla endpoints accepterar `lang`-parameter (`sv`, `en`). Standard är `sv`.

---

## Språk / Language

SCB:s API stödjer svenska (`lang=sv`) och engelska (`lang=en`). Tabelltitlar, variabelnamn och värdetexter finns på båda språken.

**Språkval:**

- Om användaren skriver på **svenska**: svara på svenska och använd `lang=sv` i API-anrop
- Om användaren skriver på **engelska**: svara på engelska och använd `lang=en`
- Talformat: svenska använder mellanslag som tusentalsavgränsare och komma som decimaltecken (1 234,5); engelska använder komma och punkt (1,234.5)
- OBS: API:et returnerar alltid decimalpunkt oavsett språk — formatera om vid presentation
- Källhänvisning på svenska: "Källa: SCB, tabell {id}" / på engelska: "Source: Statistics Sweden, table {id}"

---

## Arbetsflöde / Workflow

Följ stegen i ordning. Hoppa aldrig över metadata-steget.

### Steg 1: Förstå behovet

Klargör innan du anropar något:

- **Fenomen** — Vad mäts? (befolkning, priser, sysselsättning, handel, utbildning, hälsa)
- **Geografi** — Hela Sverige, län, kommun, församling?
- **Tidsperiod** — Senaste året, senaste 10 åren, bestämt intervall?
- **Nedbrytning** — Kön, ålder, näringsgren, utbildningsnivå?

Om användaren är vag, ställ **en** följdfråga — inte flera.

### Steg 2: Sök efter tabell

Använd `GET /tables` med `query`-parameter.

**Sökparametrar:**

| Parameter             | Typ    | Beskrivning                                        |
| --------------------- | ------ | -------------------------------------------------- |
| `query`               | string | Fritextsökord                                      |
| `pastDays`            | int    | Begränsa till tabeller uppdaterade senaste N dagar |
| `includeDiscontinued` | bool   | Inkludera avslutade serier (default: false)        |
| `pageNumber`          | int    | Sidnummer för paginering                           |
| `pageSize`            | int    | Antal träffar per sida                             |

**Tips för bra sökningar:**

- Använd svenska fackord: "konsumentprisindex" (inte "KPI"), "sysselsatta" (inte "jobb"), "folkmängd" (inte "befolkning")
- Överväg synonymer: "folkmängd" ≈ "befolkning" ≈ "invånare"
- Sökningen letar i tabelltitlar, variabler och variabelvärden (case-insensitivt)
- `title:`-prefix begränsar till titelfältet: `title:barn`
- Fuzzy-sökning: `~N` efter ett ord tillåter N teckens avvikelse (t.ex. `konsumentpris~1`)
- Närhetssökning: `"varunummer hs" ~5` hittar orden inom 5 ord från varandra
- Trunkering: `anlägg*` matchar allt som börjar med "anlägg"
- Booleska operatorer: `trend AND anlägg*`
- Sök efter uppdateringsdatum: `updated:20250908*` eller `updated:[20250908 TO 20250912*]`
- Använd `pastDays` för att hitta nyligen uppdaterade tabeller
- Kontrollera `lastPeriod` och `timeUnit` i resultaten
- Avslutade tabeller har `discontinued: true` — undvik dessa om inte historiska data behövs

Presentera de 3–5 mest relevanta träffarna med tabell-ID, titel, senaste period, tidsfrekvens och `discontinued`-status. Rekommendera den mest passande.

Responsstrukturen för varje träff inkluderar: `id`, `label`, `description`, `updated`, `firstPeriod`, `lastPeriod`, `timeUnit` (Annual/Quarterly/Monthly/Weekly), `variableNames`, `discontinued`, `subjectCode`, och `paths` (ämnesplacering i SCB:s hierarki).

**Hitta kommunkoder:**

När användaren nämner en kommun du inte kan koden till — wildcards i valueCodes matchar bara *koder*, inte kommunnamn. Strategi:

1. Sök efter kommunnamnet i API:et — sökningen letar i variabelvärden och bekräftar att kommunen finns
2. Hämta metadata för en relevant tabell (t.ex. BefolkningNy) — scanna `category.label` i Region-dimensionen för kommunnamnet och hitta koden
3. Alternativt: använd wildcard på länskoden (t.ex. `01*` för Stockholms län) och identifiera kommunen i resultaten
4. SCB:s standardkoder för kommuner/län följer SKR:s 4-siffriga kommunkoder (de två första = länskod)

### Steg 3: Utforska metadata

Använd `GET /tables/{id}/metadata` för att förstå tabellens struktur.

Metadata returneras i json-stat2-format (Dataset-schema) — se `references/json-stat2.md` för full struktur, row-major-indexering och status-koder. Fokusera på:

- **`id`-array** — Variabelnamnen (t.ex. `["Region", "Kon", "Alder", "ContentsCode", "Tid"]`)
- **`size`-array** — Antal värden per variabel
- **`dimension`-objekt** — Detaljerad info per variabel:
  - `category.index` — Koderna och ordningen
  - `category.label` — Läsbara namn
  - `category.unit` — Enhet och decimaler (på ContentsCode)
  - `extension.elimination` — Om variabeln kan uteslutas från query (true = summeras automatiskt)
  - `extension.eliminationValueCode` — Vilken kod som används vid eliminering
  - `extension.codelists` — Tillgängliga kodlistor för variabeln
- **`extension`-objekt (rot)** — `firstPeriod`, `lastPeriod`, `discontinued`, `contact`
- **`role`-objekt** — Vilka variabler som har roll som `time`, `geo`, eller `metric`
- **`value`-array** — Platt array med alla datavärden i row-major order (se `references/json-stat2.md` för indexformel)

**Viktiga regler om metadata:**

- Variabler med `elimination: true` kan uteslutas — de summeras automatiskt
- Variabler med `elimination: false` MÅSTE inkluderas i query. Tid och ContentsCode är alltid icke-elimineringsbara.
- `ContentsCode` talar om vad som mäts (antal, procent, SEK, index) — kontrollera `category.unit` för enhet och decimaler

**Kodlistor och filtrering:**

Använd `codelist`-parameter i metadata-uppslag eller data-query för att aktivera en kodlista, eller slå upp en kodlista separat med `GET /codelists/{id}`. Aggregeringar (`agg_`) definierar nya aggregatkoder; värdemängder (`vs_`) är delmängder av originalkoder.

**Defaultselection:**

Använd `GET /tables/{id}/defaultselection` för att hämta tabellens förvalda selektion. Användbart som utgångspunkt — särskilt för stora tabeller.

### Steg 4: Bygg och kör query

PxWebApi v2 stödjer **både GET och POST** för datahämtning. SCB:s Statistikdatabas (https://www.statistikdatabasen.scb.se) fungerar som grafisk frågebyggare.

#### POST (rekommenderat för komplexa frågor)

```
POST /tables/{id}/data?outputFormat=json-stat2
Content-Type: application/json

{
  "selection": [
    {
      "variableCode": "Region",
      "valueCodes": ["0180"],
      "codelist": null
    },
    {
      "variableCode": "ContentsCode",
      "valueCodes": ["BE0101N1"]
    },
    {
      "variableCode": "Tid",
      "valueCodes": ["top(5)"]
    }
  ]
}
```

Variabler med `elimination: true` kan uteslutas från `selection`-arrayen.

#### GET (enklare frågor, delbara URL:er)

```
GET /tables/{id}/data?valuecodes[Region]=0180&valuecodes[Tid]=top(5)&outputFormat=json-stat2
```

#### Outputformat

| Format     | `outputFormat`-värde | Användning                           |
| ---------- | -------------------- | ------------------------------------ |
| json-stat2 | `json-stat2`         | Standard, maskinläsbar, rik metadata |
| CSV        | `csv`                | Enkelt tabulärt format               |
| Excel      | `xlsx`               | För slutanvändare                    |
| HTML       | `html`               | Tabell för visning                   |
| PX         | `px`                 | Traditionellt PX-format              |
| JSON-PX    | `json-px`            | JSON-variant av PX                   |

**OutputFormatParams** (kan kombineras):

- `UseCodes` / `UseTexts` / `UseCodesAndTexts`
- `IncludeTitle`
- `SeparatorTab` / `SeparatorSpace` / `SeparatorSemicolon`

#### heading och stub

Styr pivoteringen med `heading` (kolumner) och `stub` (rader) — listor av variabelnamn.

#### Filteruttryck i valueCodes

Viktigaste mönster: `top(N)` = senaste N värdena, `from(värde)` = från och med, `range(från,till)` = intervall, `*` = alla värden. Wildcards `*` och `?` för mönstermatchning (t.ex. `01*` = alla kommuner i Stockholms län).

**Viktiga begränsningar:**

- API:et har en övre gräns för antal celler per query. Kontrollera `/config` för `maxDataCells`.
- Rate limiting: `/config` visar `maxCallsPerTimeWindow` och `timeWindow`.
- Börja smalt — lättare att utvidga än att hantera för mycket data.

### Steg 5: Presentera resultaten

- Visa data i en snygg markdown-tabell
- Inkludera **alltid** källhänvisning med **samtliga tabell-ID:n** som använts (lista alla om flera tabeller kombinerats — utelämna ingen):
  - Svenska: **"Källa: SCB, tabell {id}"** (eller "tabellerna {id1}, {id2}, …")
  - Engelska: **"Source: Statistics Sweden, table {id}"** (eller "tables {id1}, {id2}, …")
- Förklara vad siffrorna betyder i kontext — på användarens språk
- Talformat: svenska = mellanslag + komma (1 234,5); engelska = komma + punkt (1,234.5)
- Presentera enheter tydligt (antal/count, procent/percent, index, SEK)
- Erbjud att visualisera data
- Erbjud nedladdning i annat format (csv, xlsx)

### Steg 6: Sparade frågor (valfritt)

För att skapa en delbar, återanvändbar fråga:

```
POST /savedqueries
Content-Type: application/json

{
  "tableId": "BefolkningNy",
  "language": "sv",
  "selection": {
    "selection": [
      { "variableCode": "Region", "valueCodes": ["0180"] },
      { "variableCode": "Tid", "valueCodes": ["top(5)"] }
    ]
  },
  "outputFormat": "json-stat2"
}
```

Returnerar ett ID. Data hämtas med `GET /savedqueries/{id}/data`.

Användbart för rapporter som uppdateras regelbundet — `top(N)` ger alltid de senaste perioderna.

---

## Responsformat

Både metadata och data returneras som **json-stat2** som standard (Dataset-schema).

---

## Regler

**Gör alltid:**

- Sök först — gissa aldrig tabell-ID:n
- Kontrollera metadata innan du bygger en query
- Filtrera på tid för att hålla datamängden hanterbar
- Visa källa med tabellnummer
- Använd svenska som standardspråk om inte användaren ber om engelska
- Presentera enheter tydligt
- Kontrollera `elimination`-flaggan — uteslut bara variabler som tillåter det

**Gör aldrig:**

- Hämta data utan filter från stora tabeller
- Anta att kommunkoder är stabila över tid (kommunsammanslagningar förekommer)
- Blanda koder från olika kodlistor
- Presentera data utan enhet
- Ignorera `status`-fältet — det kan indikera saknade eller konfidentiella värden

---

## Exempel

### "Hur många bor i Stockholm?"

```
1. GET /tables?query=folkmängd
2. GET /tables/BefolkningNy/metadata
3. POST /tables/BefolkningNy/data
   { "selection": [
       { "variableCode": "Region", "valueCodes": ["0180"] },
       { "variableCode": "ContentsCode", "valueCodes": ["BE0101N1"] },
       { "variableCode": "Tid", "valueCodes": ["top(1)"] }
   ]}
→ "Per 31 december 2025 hade Stockholm N invånare (Källa: SCB, tabell BefolkningNy)"
```

### "KPI senaste 5 åren, månadsvis"

```
1. GET /tables?query=konsumentprisindex
2. GET /tables/{id}/metadata
3. POST /tables/{id}/data
   { "selection": [
       { "variableCode": "ContentsCode", "valueCodes": ["..."] },
       { "variableCode": "Tid", "valueCodes": ["top(60)"] }
   ]}
```

### "What is the population of Sweden?" (English query)

```
1. GET /tables?query=population&lang=en
2. GET /tables/BefolkningNy/metadata?lang=en
3. POST /tables/BefolkningNy/data?lang=en
   { "selection": [
       { "variableCode": "Region", "valueCodes": ["00"] },
       { "variableCode": "ContentsCode", "valueCodes": ["BE0101N1"] },
       { "variableCode": "Tid", "valueCodes": ["top(1)"] }
   ]}
→ "As of 31 December 2025, Sweden had N inhabitants (Source: Statistics Sweden, table BefolkningNy)"
```

---

## Fallback

Om API:et inte är tillgängligt:

1. Hänvisa till SCB:s Statistikdatabas: https://www.statistikdatabasen.scb.se
2. Föreslå relevanta sökord baserat på frågan
3. Ge vägledning för manuell uppslagning (tabellnamn, variabler att leta efter)

**Tips:** I Statistikdatabasen kan du bygga upp ett uttag grafiskt och sedan exportera som API-query. Användbart för att verifiera koder och filter.
