---
name: ssb-pxwebapi-v2
description: >
  Norsk offentlig statistikk fra SSB via PxWebApi v2. Bruk ALLTID når noen spør om
  norske tall, statistikk, befolkning, KPI, inflasjon, arbeidsledighet, lønn, priser,
  BNP, økonomi, handel, eksport, import, utdanning, helse, bolig, kommune- eller
  fylkesdata, eller nevner SSB, Statistisk sentralbyrå eller Statistikkbanken.
  Trigger på "finn tall på", "hvor mange bor i", "KPI siste år", "befolkningsvekst",
  "prisindeks", "boligpriser" og lignende. Also trigger on "Norwegian statistics",
  "population of Norway", "Statistics Norway", "SSB data", "Norway GDP",
  "inflation in Norway", "housing prices Norway" or similar. Bruk denne fremfor
  websøk når svaret finnes i norsk offentlig statistikk. Dekker kodelister,
  lagrede spørringer og outputformater (json-stat2, csv, xlsx).
---

# SSB PxWebApi v2 — Komplett guide

Denne skillen guider deg gjennom riktig bruk av SSBs PxWebApi v2 for å søke, utforske og hente norsk offentlig statistikk. API-ets base-URL er:

```
https://data.ssb.no/api/pxwebapi/v2
```

## API-oversikt

PxWebApi v2 har disse endepunktene:

| Endepunkt | Metode | Formål |
|---|---|---|
| `/tables` | GET | Søk og list tabeller |
| `/tables/{id}` | GET | Hent info om én tabell |
| `/tables/{id}/metadata` | GET | Hent metadata (variabler, koder, kodelister) |
| `/tables/{id}/defaultselection` | GET | Hent tabellens forhåndsvalgte seleksjon |
| `/tables/{id}/data` | GET / POST | Hent data med filtre |
| `/codelists/{id}` | GET | Slå opp en kodeliste |
| `/savedqueries` | POST | Opprett en lagret spørring |
| `/savedqueries/{id}` | GET | Hent en lagret spørring |
| `/savedqueries/{id}/data` | GET | Kjør en lagret spørring og hent data |
| `/savedqueries/{id}/selection` | GET | Hent seleksjonen til en lagret spørring |
| `/config` | GET | API-konfigurasjon (grenser, formater, språk) |

Alle endepunkter aksepterer `lang`-parameter (`no`, `en`). Standard er `no`.

---

## Språk / Language

SSBs API støtter norsk (`lang=no`) og engelsk (`lang=en`). Alle tabelltitler, variabelnavn og verditekster finnes på begge språk.

**Språkvalg:**
- Hvis brukeren skriver på **norsk**: svar på norsk og bruk `lang=no` i API-kall
- Hvis brukeren skriver på **engelsk**: svar på engelsk og bruk `lang=en` i API-kall
- Tallformat tilpasses brukerens språk: norsk bruker mellomrom som tusenskilletegn og komma som desimalskille (1 234,5); engelsk bruker komma og punktum (1,234.5)
- NB: API-et returnerer alltid desimal punktum uavhengig av språk — formater om ved presentasjon
- Kildehenvisning på norsk: "Kilde: SSB, tabell {id}" / på engelsk: "Source: Statistics Norway, table {id}"

---

## Arbeidsflyt / Workflow

Følg disse stegene i rekkefølge. Hopp aldri over metadata-steget.

### Steg 1: Forstå behovet

Avklar før du kaller noe:

- **Fenomen** — Hva måles? (befolkning, priser, sysselsetting, handel, utdanning, helse)
- **Geografi** — Hele Norge, fylke, kommune, bydel?
- **Tidsperiode** — Siste år, siste 10 år, bestemt intervall?
- **Nedbrytning** — Kjønn, alder, næring, utdanningsnivå?

Hvis brukeren er vag, still **ett** oppfølgingsspørsmål — ikke flere.

### Steg 2: Søk etter tabell

Bruk `GET /tables` med `query`-parameter.

**Søkeparametre:**

| Parameter | Type | Beskrivelse |
|---|---|---|
| `query` | string | Fritekst-søkeord |
| `pastDays` | int | Begrens til tabeller oppdatert siste N dager |
| `includeDiscontinued` | bool | Inkluder avsluttede serier (default: false) |
| `pageNumber` | int | Sidenummer for paginering |
| `pageSize` | int | Antall treff per side |

**Tips for gode søk:**

- Bruk norske fagtermer: "konsumprisindeks" (ikke "KPI"), "sysselsatte" (ikke "jobber"), "folkemengde" (ikke "befolkning")
- Vurder synonymer: "folkemengde" ≈ "befolkning" ≈ "innbyggere"
- Søket leter i tabelltitler, variabler og variabelverdier (case-insensitivt)
- `title:`-prefix begrenser søket til tittelfeltet: `title:barn`
- Fuzzy søk: `~N` etter et ord tillater N tegns avvik (f.eks. `konsumpris~1`)
- Nærhetssøk: `"varenummer hs" ~5` finner ordene innen 5 ord fra hverandre
- Trunkering: `anlegg*` matcher alt som starter med "anlegg"
- Boolske operatorer: `trend AND anlegg*`
- Søk etter oppdateringsdato: `updated:20250908*` eller `updated:[20250908 TO 20250912*]`
- Kombiner med regionalt nivå: `title:foretak AND title:(F)` for fylkesnivå
- Bruk `pastDays` for å finne nylig oppdaterte tabeller
- Sjekk `lastPeriod` og `timeUnit` i resultatene
- Avsluttede tabeller har `discontinued: true` — unngå disse med mindre historiske data trengs

Presenter de 3–5 mest relevante treffene med tabell-ID, tittel, siste periode, tidsfrekvens og `discontinued`-status. Anbefal den mest passende.

Respons-strukturen for hvert treff inkluderer: `id`, `label`, `description`, `updated`, `firstPeriod`, `lastPeriod`, `timeUnit` (Annual/Quarterly/Monthly/Weekly), `variableNames`, `discontinued`, `subjectCode`, og `paths` (emneplassering i SSBs hierarki).

Se `references/common-tables.md` for en kurert liste over mye brukte tabeller.

**Finne kommunekoder:**

Når brukeren nevner en kommune du ikke kjenner koden til — wildcards i valueCodes matcher bare *koder*, ikke kommunenavn. Strategi:

1. Søk etter kommunenavnet med `ssb_search` — API-et søker i variabelverdier og bekrefter at kommunen finnes
2. Hent metadata for en relevant tabell (f.eks. 07459) — scan `category.label` i Region-dimensjonen for kommunenavnet og finn koden
3. Alternativt: bruk wildcard på fylkeskoden (f.eks. `34*` for Innlandet, `345?` for å snevre inn) og identifiser kommunen i resultatene
4. Klass API har komplett kommuneklassifikasjon: `https://data.ssb.no/api/klass/v1/classifications/131.json`

### Steg 3: Utforsk metadata

Bruk `GET /tables/{id}/metadata` for å forstå tabellens struktur.

Metadata returneres i json-stat2-format (Dataset-schema). Fokuser på:

- **`id`-array** — Variabelnavnene (f.eks. `["Region", "Kjonn", "Alder", "ContentsCode", "Tid"]`)
- **`size`-array** — Antall verdier per variabel
- **`dimension`-objekt** — Detaljert info per variabel:
  - `category.index` — Kodene og rekkefølgen (f.eks. `{"0301": 0, "1103": 1}`)
  - `category.label` — Lesbare navn (f.eks. `{"0301": "Oslo"}`)
  - `category.unit` — Enhet og desimaler (på ContentsCode)
  - `extension.elimination` — Om variabelen kan utelates fra query (true = summeres automatisk)
  - `extension.eliminationValueCode` — Hvilken kode som brukes ved eliminering
  - `extension.codelists` — Tilgjengelige kodelister for variabelen
- **`extension`-objekt (rot)** — `firstPeriod`, `lastPeriod`, `discontinued`, `contact`, og PX-metadata som `subject-code`, `subject-area`
- **`role`-objekt** — Hvilke variabler som har rolle som `time`, `geo`, eller `metric`
- **`link.describedby`** — Kobling til SSBs Klass (klassifikasjoner) og VarDok (variabeldefinisjoner) via URN-er (se egen seksjon nedenfor)

**Viktige regler om metadata:**

- Variabler med `elimination: true` kan utelates — de summeres automatisk (enten til en eliminasjonsverdi/sum, eller samtlige verdier aggregeres til én)
- Variabler med `elimination: false` MÅ inkluderes i query. Tid og ContentsCode er alltid ikke-eliminerbare.
- `ContentsCode` forteller hva som måles (antall, prosent, NOK, indeks) — sjekk `category.unit` for enhet og desimaler
- Per statistikkvariabel kan `extension` inneholde: `measuringType` (Stock/Flow/Average), `priceType` (Current/Fixed), `adjustment` (sesongjustering), og `basePeriod`

**Kodelister og filtrering:**

Se `references/codelists-and-filters.md` for komplett referanse over kodelister (aggregeringer `agg_` og verdimengder `vs_`), filteruttrykk (`top()`, `from()`, `range()`, wildcards), og tidsformater.

Kort oppsummert: Bruk `codelist`-parameter i metadata-oppslag eller data-query for å aktivere en kodeliste, eller slå opp en kodeliste separat med `GET /codelists/{id}`.

**Defaultselection:**

Bruk `GET /tables/{id}/defaultselection` for å hente tabellens forhåndsvalgte seleksjon. Nyttig som utgangspunkt — spesielt for store tabeller der du ikke vet hvilke verdier du bør velge. Returnerer en liste `VariableSelection`-objekter med `variableCode`, `codelist` og `valueCodes`.

### Steg 4: Bygg og kjør query

PxWebApi v2 støtter **både GET og POST** for datahenting. Du kan også bruke Statistikkbanken (https://www.ssb.no/statbank) som grafisk spørringsbygger — velg tabell og verdier, trykk "Lagre" for å få ferdig GET-URL og POST-body.

#### POST (anbefalt for komplekse spørringer)

```
POST /tables/{id}/data?outputFormat=json-stat2
Content-Type: application/json

{
  "selection": [
    {
      "variableCode": "Region",
      "valueCodes": ["0301"],
      "codelist": null
    },
    {
      "variableCode": "ContentsCode",
      "valueCodes": ["Personer1"]
    },
    {
      "variableCode": "Tid",
      "valueCodes": ["top(5)"]
    }
  ]
}
```

Variabler med `elimination: true` kan utelates fra `selection`-arrayet.

#### GET (enklere spørringer, delbare URL-er)

```
GET /tables/{id}/data?valuecodes[Region]=0301&valuecodes[Tid]=top(5)&codelist[Region]=agg_KommFylker&outputFormat=json-stat2
```

GET-varianten er ideell for å lage URL-er som kan deles direkte. Se `references/codelists-and-filters.md` for `outputValues`-parameter ved bruk av grupperinger.

#### Outputformater

| Format | `outputFormat`-verdi | Bruk |
|---|---|---|
| json-stat2 | `json-stat2` | Standard, maskinlesbar, rik metadata |
| CSV | `csv` | Enkelt tabulært format |
| Excel | `xlsx` | For sluttbrukere |
| HTML | `html` | Tabell for visning |
| PX | `px` | Tradisjonelt PX-format |
| JSON-PX | `json-px` | JSON-variant av PX |

NB: `parquet` er definert i API-specen men ikke implementert ennå.

**OutputFormatParams** (kan kombineres):

- `UseCodes` — Bruk koder i output
- `UseTexts` — Bruk tekster
- `UseCodesAndTexts` — Begge deler
- `IncludeTitle` — Inkluder tabelltittel
- `SeparatorTab` / `SeparatorSpace` / `SeparatorSemicolon` — Separator for CSV

#### heading og stub

Du kan styre pivoteringen med `heading` og `stub`-parametre (liste av variabelnavn). `heading` legger variabler i kolonnene, `stub` i radene.

#### Filteruttrykk i valueCodes

Viktigste mønstre: `top(N)` = siste N verdier, `from(verdi)` = fra og med, `range(fra,til)` = intervall, `*` = alle verdier. Wildcards `*` og `?` kan brukes for mønstermatching (f.eks. `46*` = alle kommuner i Vestland). Se `references/codelists-and-filters.md` for komplett syntaks.

**Viktige begrensninger:**

- API-et har en øvre grense for antall celler per spørring. Sjekk `/config` for `maxDataCells` (typisk 800 000 men kan endre seg).
- Rate limiting: `/config` viser `maxCallsPerTimeWindow` og `timeWindow`.
- Start smalt — det er lettere å utvide enn å håndtere for mye data.

### Steg 5: Presenter resultatene

- Vis dataene i en ryddig markdown-tabell
- Inkluder alltid kildehenvisning:
  - Norsk: **"Kilde: SSB, tabell {id}"**
  - Engelsk: **"Source: Statistics Norway, table {id}"**
- Forklar hva tallene betyr i kontekst — på brukerens språk
- Tallformat: norsk = mellomrom + komma (1 234,5); engelsk = komma + punktum (1,234.5)
- Presenter enheter tydelig (antall/count, prosent/percent, indeks/index, NOK)
- Tilby å visualisere dataene
- Tilby å laste ned i annet format (csv, xlsx)

### Steg 6: Lagrede spørringer (valgfritt)

For å opprette en delbar, gjenbrukbar spørring:

```
POST /savedqueries
Content-Type: application/json

{
  "tableId": "07459",
  "language": "no",
  "selection": {
    "selection": [
      { "variableCode": "Region", "valueCodes": ["0301"] },
      { "variableCode": "Tid", "valueCodes": ["top(5)"] }
    ]
  },
  "outputFormat": "json-stat2"
}
```

Returnerer en ID og lenke. Data kan hentes med:
```
GET /savedqueries/{id}/data
```

Nyttig for rapporter som oppdateres jevnlig — `top(N)` gir alltid de nyeste periodene.

---

## Responsformat

Både metadata (`/tables/{id}/metadata`) og data (`/tables/{id}/data`) returneres som **json-stat2** som standard. Se `references/api-details.md` for json-stat2 Dataset-struktur, status-koder, og praktisk driftsinformasjon (publiseringstider, grenser, lisens).

---

## Kobling til SSBs metadata-systemer

json-stat2-metadata inneholder URN-er under `link.describedby` som kobler variabler til SSBs metadata-systemer. Det finnes to typer:

**Klassifikasjoner (Klass):**
URN-er på formen `"urn:ssb:classification:klass:131"` peker til SSBs system for klassifikasjoner og kodelister. Tallet til slutt er klassifikasjons-ID. Omskrives til Klass API:
- `https://data.ssb.no/api/klass/v1/classifications/131.json`

Eksempel: `"urn:ssb:classification:klass:691"` → `https://data.ssb.no/api/klass/v1/classifications/691.json`

Klass er nyttig for fullstendige kodeverk med historikk, korrespondansetabeller (gammel→ny kommunestruktur), og gyldighetsperioder for koder.

**Variabeldefinisjoner (VarDok):**
URN-er på formen `"urn:ssb:conceptvariable:vardok:3380"` peker til SSBs variabeldefinisjoner. Tallet til slutt er variabel-ID. Omskrives til:
- Norsk: `https://www.ssb.no/a/metadata/conceptvariable/vardok/3380/nb`
- Engelsk: `https://www.ssb.no/a/metadata/conceptvariable/vardok/3380/en`

VarDok gir definisjoner, avgrensninger og bakgrunnsinformasjon for statistiske begreper.

---

## Regler

**Gjør alltid:**

- Søk først — aldri gjett på tabell-IDer
- Sjekk metadata før du bygger en query
- Filtrer på tid for å holde datasettet håndterbart
- Vis kilde med tabellnummer
- Bruk norsk som standardspråk med mindre brukeren ber om engelsk
- Presenter enheter tydelig
- Sjekk `elimination`-flagget — utelat kun variabler som tillater det

**Gjør aldri:**

- Hent data uten filtre fra store tabeller
- Anta at kommunekoder er stabile over tid (kommunesammenslåinger i 2020!)
- Bland koder fra forskjellige kodelister
- Presenter data uten enhet
- Ignorer `status`-feltet — det kan indikere manglende eller konfidensielle verdier

---

## Eksempler

### "Hvor mange bor i Oslo?"

```
1. GET /tables?query=folkemengde
2. GET /tables/07459/metadata
3. POST /tables/07459/data
   { "selection": [
       { "variableCode": "Region", "valueCodes": ["0301"] },
       { "variableCode": "ContentsCode", "valueCodes": ["Personer1"] },
       { "variableCode": "Tid", "valueCodes": ["top(1)"] }
   ]}
→ "Per 1. januar 2025 hadde Oslo 723 803 innbyggere (Kilde: SSB, tabell 07459)"
```

### "KPI siste 5 år, månedlig"

```
1. GET /tables?query=konsumprisindeks
2. GET /tables/14700/metadata
   → Sjekk ContentsCode: "KpiIndMnd" (indeks), "KpiMndEnd662" (12-mnd endring)
3. POST /tables/14700/data
   { "selection": [
       { "variableCode": "ContentsCode", "valueCodes": ["KpiIndMnd"] },
       { "variableCode": "Tid", "valueCodes": ["top(60)"] }
   ]}
→ Tabell med månedlig KPI (2025=100), med kilde. NB: Både 03013 og 03014 er avsluttet — bruk 14700.
```

### "Sammenlign befolkning i alle fylker"

```
1. GET /tables/07459/metadata?codelist[Region]=agg_KommFylker
   → Region-variabelen viser nå fylkeskoder med F-prefiks (F-03, F-11, F-46 osv.)
2. POST /tables/07459/data
   { "selection": [
       { "variableCode": "Region", "codelist": "agg_KommFylker",
         "valueCodes": ["*"] },
       { "variableCode": "ContentsCode", "valueCodes": ["Personer1"] },
       { "variableCode": "Tid", "valueCodes": ["top(1)"] }
   ]}
→ Tabell med folketall per fylke. Kodelisten begrenser * til kun fylkeskodene.
```

### "Lag en delbar URL for Oslos befolkning siste 10 år"

Tidsserier med kommunedata bør bruke kodeliste `agg_KommSummer` for å håndtere kommunesammenslåinger og gi konsistente tall over tid.

```
GET /tables/07459/data?valuecodes[Region]=0301&codelist[Region]=agg_KommSummer&valuecodes[ContentsCode]=Personer1&valuecodes[Tid]=top(10)&outputFormat=json-stat2
```

Full URL:
```
https://data.ssb.no/api/pxwebapi/v2/tables/07459/data?valuecodes[Region]=0301&codelist[Region]=agg_KommSummer&valuecodes[ContentsCode]=Personer1&valuecodes[Tid]=top(10)&outputFormat=json-stat2
```

### "Eksporter boligpriser som Excel"

```
POST /tables/07221/data?outputFormat=xlsx&outputFormatParams=UseCodesAndTexts&outputFormatParams=IncludeTitle
{ "selection": [
    { "variableCode": "Boligtype", "valueCodes": ["00"] },
    { "variableCode": "ContentsCode", "valueCodes": ["KvPris"] },
    { "variableCode": "Tid", "valueCodes": ["top(20)"] }
]}
→ Excel-fil med boligprisindeks siste 20 kvartaler
```

### "What is the population of Norway?" (English query)

```
1. GET /tables?query=population&lang=en
2. GET /tables/07459/metadata?lang=en
3. POST /tables/07459/data?lang=en
   { "selection": [
       { "variableCode": "Region", "valueCodes": ["0"] },
       { "variableCode": "ContentsCode", "valueCodes": ["Personer1"] },
       { "variableCode": "Tid", "valueCodes": ["top(1)"] }
   ]}
→ "As of 1 January 2026, Norway had 5 550 203 inhabitants (Source: Statistics Norway, table 07459)"
```

---

## Feilhåndtering

Se `references/troubleshooting.md` for vanlige feil og løsninger.

---

## Fallback

Hvis API-et ikke er tilgjengelig:

1. Henvis til SSBs Statistikkbank: https://www.ssb.no/statbank
2. Foreslå relevante søkeord basert på spørsmålet
3. Gi veiledning for manuelt oppslag (tabellnummer, variabler å se etter)

**Tips:** I Statistikkbanken kan du bygge opp et uttrekk grafisk, deretter trykke "Lagre" for å få ferdig API-spørring som både GET-URL og POST-body. Nyttig for å verifisere koder og filtre.
