---
name: scb-pxwebapi-v2
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
metadata:
  version: "0.10.0"
---

# SCB PxWebApi v2 — Komplett guide

Denna skill guidar dig genom korrekt användning av SCB:s PxWebApi v2 för att söka, utforska och hämta svensk offentlig statistik. API:ets bas-URL är:

```
https://statistikdatabasen.scb.se/api/v2
```

## Dataintegritet — grundregeln

SCB är Sveriges officiella statistikproducent. Förtroendet för siffrorna är själva produkten. Denna regel går före allt annat i skillen:

**Ange aldrig en siffra du inte har hämtat från API:et i denna konversation.**

- **Inga siffror ur minnet.** Har du inte kört frågan har du inte siffran — det gäller även siffror du är säker på. Folkmängd, prisindex och arbetslöshetstal ändras, och träningsdata har ett brytdatum.
- **Inga siffror från andra källor i samma svar.** Hänvisa vidare i stället för att blanda.
- **Fallerar API:et, säg det.** Inga uppskattningar, inget "ungefär". Se Fallback.
- **Ingen interpolering eller framskrivning.** Saknas en period i uttaget saknas den i svaret.
- **Märk dina egna beräkningar.** Tillväxttal, andelar och summor är dina, inte SCB:s — visa vilka hämtade siffror de bygger på, och behåll API:ets decimaler (`category.unit.decimals`).
- **Visa `status`-värden som de är.** Saknade, preliminära och konfidentiella värden hör hemma i tabellen, inte dolda eller ersatta med noll.
- **Kontrollera `discontinued` och `lastPeriod`.** Tabeller avslutas och serien fortsätter ofta i en ny tabell. Använder du en avslutad tabell, säg det och ange sista perioden.

Att inte hitta siffran är ett giltigt svar. Ett ärligt "hittade inte", med förslag på sökord, är bättre än en rimlig siffra som är fel.

---

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

**Vilken endpoint när?**

- Vet inte tabell-ID → `GET /tables?query=…`
- Har tabell-ID, känner inte strukturen → `GET /tables/{id}/metadata`
- Behöver en förvald selektion för stor tabell → `GET /tables/{id}/defaultselection`
- Känner strukturen, ska hämta data → `POST /tables/{id}/data` (eller GET för delbar URL)
- Slå upp en kodlista isolerat → `GET /codelists/{id}`
- Användaren har byggt ett uttag i Statistikdatabasen → kopiera "Spara/API"-URL/POST-body direkt
- Ska återanvända/dela en fråga → `POST /savedqueries`, sedan `GET /savedqueries/{id}/data`
- Kontrollera gränser (maxDataCells, rate limit, licens) → `GET /config`

---

## Verktygsval / Tool selection

API:et kan nås via två kanaler. Använd MCP-verktygen från `pxweb-mcp` (npm: `@jarib/pxweb-mcp`) när de är anslutna *och täcker behovet*; använd annars direkta HTTP-anrop (curl via Bash eller motsvarande — POST kräver verktyg med stöd för request-body).

**OBS:** `pxweb-mcp` pekar som standard mot norska SSB. Mot SCB måste servern startas med `--url https://statistikdatabasen.scb.se/api/v2` — verifiera vilken instans den anslutna servern använder (verktygsbeskrivningarna nämner "Statistics Norway" oavsett konfiguration).

| Verktyg              | Motsvarar                   | Huvudparametrar                                                                  |
| -------------------- | --------------------------- | -------------------------------------------------------------------------------- |
| `search_tables`      | `GET /tables?query=…`       | `query`, `language`, `include_discontinued`                                       |
| `get_table_info`     | `GET /tables/{id}`          | `table_id`, `language`                                                            |
| `fetch_metadata`     | `GET /tables/{id}/metadata` | `table_id`, `language`                                                            |
| `query_table`        | `GET /tables/{id}/data`     | `table_id`, `value_codes`, `code_list`, `output_values`, `output_format`, `language` |
| `get_code_list`      | `GET /codelists/{id}`       | `code_list_id`, `language`                                                        |
| `list_recent_tables` | `GET /tables?pastDays=N`    | `days`, `language`                                                                |

`query_table` mappar objekten till URL-parametrar (`value_codes` → `valueCodes[Var]`, `code_list` → `codelist[Var]`, `output_values` → `outputValues[Var]`) med samma filtersyntax (`top()`, `from()`, wildcards) som GET-kanalen.

**Begränsningar — använd HTTP när:**

- Du behöver `/savedqueries`, `/tables/{id}/defaultselection` eller `/config` — de exponeras inte som verktyg.
- Du behöver svenska texter: verktygens `language`-parameter stödjer endast `no`/`en`. Använd `language: "en"` mot SCB, eller HTTP med `lang=sv`.
- Du söker tabeller och behöver Steg 2-fälten: `search_tables` returnerar endast `id` + titel (utan `lastPeriod`, `timeUnit`, `discontinued`, `variableNames`) och saknar paginering — anropa `get_table_info` per kandidat, eller sök via HTTP.
- Du vill se metadata med en kodlista applicerad: `fetch_metadata` stödjer inte `codelist[Var]`-parametern — använd `get_code_list` i stället.
- Du behöver `outputFormatParams` (`UseCodesAndTexts`, `IncludeTitle`, `heading`/`stub`) — `query_table` exponerar dem inte.

(Verifierat mot `@jarib/pxweb-mcp` v2.0.0, 2026-06-12.)

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
2. Hämta metadata för en relevant tabell (t.ex. TAB638) — scanna `category.label` i Region-dimensionen för kommunnamnet och hitta koden
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
- **`role`-objekt** — Vilka variabler som har roll som `time`, `geo` eller `metric`. **Börja analysen här:**
  - `role.metric` visar vad som mäts. Hos SCB är det `ContentsCode` — kontrollera `category.unit` för enhet och decimaler.
  - `role.time` är tidsdimensionen.
  - `role.geo` är geografi. **Om `role.geo` saknas, anta att data gäller hela Sverige** — fråga inte användaren.
  - Övriga variabler i `id` är nedbrytningsdimensioner (kön, ålder, näringsgren m.m.).
- **`value`-array** — Platt array med alla datavärden i row-major order (se `references/json-stat2.md` för indexformel)

**Viktiga regler om metadata:**

- Variabler med `elimination: true` kan uteslutas — de summeras automatiskt
- Variabler med `elimination: false` MÅSTE inkluderas i query. Tid och ContentsCode är alltid icke-elimineringsbara.
- `ContentsCode` talar om vad som mäts (antal, procent, SEK, index) — kontrollera `category.unit` för enhet och decimaler

**Kodlistor och filtrering:**

Använd `codelist`-parameter i metadata-uppslag eller data-query för att aktivera en kodlista, eller slå upp en kodlista separat med `GET /codelists/{id}`. Aggregeringar (`agg_`) definierar nya aggregatkoder; värdemängder (`vs_`) är delmängder av originalkoder.

**Defaultselection:**

Använd `GET /tables/{id}/defaultselection` för att hämta tabellens förvalda selektion. Användbart som utgångspunkt — särskilt för stora tabeller.

**Ett `GET /data`-anrop utan selektionsparametrar är inget fel, och ger inte hela tabellen** — det returnerar tyst tabellens förvalda selektion. Verifierat 2026-08-30: `TAB638` ger `size [290, 1, 1, 2]`, 580 celler, HTTP 200.

Det farliga är vad som försvinner. Elimineringsbara dimensioner som inte ingår i förvalet summeras bort utan att svaret nämner det, så du får en rimlig datamängd som besvarar en annan fråga än du ställde. Bygg alltid selektionen själv.

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
GET /tables/{id}/data?valueCodes[Region]=0180&valueCodes[ContentsCode]=BE0101N1&valueCodes[Tid]=top(5)&outputFormat=json-stat2
```

OBS: Variabler med `elimination: false` (typiskt `Tid` och `ContentsCode`) måste alltid inkluderas — annars returnerar API:et HTTP 400. Felmeddelandet innehåller en stavfel i API:et och lyder ordagrant `"Missing selection for mandantory variable"` (sic, `mandantory`) — samma stavfel som hos SSB. Citera det som det är, annars matchar inte en sökning i loggar.

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

**För tidsdimensionen: föredra `top(N)`/`from(värde)` framför `range(från,till)` och explicita perioder.** Relativa filter fångar upp nya perioder automatiskt, så delbara URL:er och sparade frågor fortsätter ge aktuella siffror i stället för att frysa på de perioder som råkade vara senast när frågan skrevs. Använd `range()` bara när det slutna intervallet är själva poängen. Detta gäller särskilt `/savedqueries`, vars hela syfte är att köras om senare.

**Viktiga begränsningar:**

- API:et har en övre gräns för antal celler per query. Kontrollera `/config` för `maxDataCells`.
- Rate limiting: `/config` visar `maxCallsPerTimeWindow` och `timeWindow`.
- Börja smalt — lättare att utvidga än att hantera för mycket data.

### Steg 5: Presentera resultaten

- Visa data i en snygg markdown-tabell
- Inkludera **alltid** källhänvisning med **samtliga tabell-ID:n** som använts (lista alla om flera tabeller kombinerats — utelämna ingen):
  - Svenska: **"Källa: SCB, tabell {id}"** (eller "tabellerna {id1}, {id2}, …")
  - Engelska: **"Source: Statistics Sweden, table {id}"** (eller "tables {id1}, {id2}, …")
- Förklara vad siffrorna betyder i kontext — på användarens språk. Kommentera **endast** siffrorna som hämtats i uttaget — hämta inte in eller blanda data från andra källor (Riksbanken, Eurostat, webbsökning) i svaret. Behöver användaren sådana siffror, hänvisa till rätt skill eller källa i stället
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
  "tableId": "TAB638",
  "language": "sv",
  "selection": {
    "selection": [
      { "variableCode": "Region", "valueCodes": ["0180"] },
      { "variableCode": "ContentsCode", "valueCodes": ["BE0101N1"] },
      { "variableCode": "Tid", "valueCodes": ["top(5)"] }
    ]
  },
  "outputFormat": "json-stat2",
  "outputFormatParams": []
}
```

OBS: Både `outputFormat` *och* `outputFormatParams` är obligatoriska i savedqueries-bodyn — sätt `outputFormatParams: []` om du inte behöver några. Inkludera även alla icke-elimineringsbara variabler (`Tid`, `ContentsCode`).

Returnerar ett ID. Data hämtas med `GET /savedqueries/{id}/data`.

Användbart för rapporter som uppdateras regelbundet — `top(N)` ger alltid de senaste perioderna.

---

## Responsformat

Både metadata och data returneras som **json-stat2** som standard (Dataset-schema).

---

## Fallgropar — gör aldrig

- Hämta data utan filter från stora tabeller
- Anta att kommunkoder är stabila över tid (kommunsammanslagningar förekommer)
- Blanda koder från olika kodlistor
- Presentera data utan enhet
- Komplettera SCB-siffror med data hämtade från andra källor i samma svar — kommentera endast det hämtade uttaget och hänvisa vidare i stället
- Ignorera `status`-fältet — det kan indikera saknade eller konfidentiella värden

---

## Licens

SCB:s statistik publiceras under **Creative Commons CC0 1.0** (public domain — fri att använda, även kommersiellt, utan attributionskrav). Den exakta licens-URL:en exponeras via `GET /config` i fältet `license`. Behåll ändå källhänvisning ("Källa: SCB, tabell {id}") som god praxis och för spårbarhet.

---

## Exempel

### "Hur många bor i Stockholm?"

```
1. GET /tables?query=folkmängd
2. GET /tables/TAB638/metadata
3. POST /tables/TAB638/data
   { "selection": [
       { "variableCode": "Region", "valueCodes": ["0180"] },
       { "variableCode": "ContentsCode", "valueCodes": ["BE0101N1"] },
       { "variableCode": "Tid", "valueCodes": ["top(1)"] }
   ]}
→ "Per 31 december 2025 hade Stockholm N invånare (Källa: SCB, tabell TAB638)"
```

### "KPI senaste 5 åren, månadsvis"

```
1. GET /tables?query=konsumentprisindex+totalt
2. GET /tables/TAB6596/metadata
   → ContentsCode: "00000808" (KPI fastställda tal, 2020=100),
     "00000804" (Årsförändring i procent)
3. POST /tables/TAB6596/data
   { "selection": [
       { "variableCode": "ContentsCode", "valueCodes": ["00000808"] },
       { "variableCode": "Tid", "valueCodes": ["top(60)"] }
   ]}
→ Tabell med månatlig KPI (2020=100) senaste 60 månaderna, med källa.
  TAB6596 är den löpande totala KPI-serien; TAB5737 är äldre serie (1980=100, uppdateras ej efter 2025M12).
```

### "What is the population of Sweden?" (English query)

```
1. GET /tables?query=population&lang=en
2. GET /tables/TAB638/metadata?lang=en
3. POST /tables/TAB638/data?lang=en
   { "selection": [
       { "variableCode": "Region", "valueCodes": ["00"] },
       { "variableCode": "ContentsCode", "valueCodes": ["BE0101N1"] },
       { "variableCode": "Tid", "valueCodes": ["top(1)"] }
   ]}
→ "As of 31 December 2025, Sweden had N inhabitants (Source: Statistics Sweden, table TAB638)"
```

---

## Fallback

Om API:et inte är tillgängligt:

1. Hänvisa till SCB:s Statistikdatabas: https://www.statistikdatabasen.scb.se
2. Föreslå relevanta sökord baserat på frågan
3. Ge vägledning för manuell uppslagning (tabellnamn, variabler att leta efter)

**Tips:** I Statistikdatabasen kan du bygga upp ett uttag grafiskt och sedan exportera som API-query. Användbart för att verifiera koder och filter.
