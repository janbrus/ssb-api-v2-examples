# Kodelister og filtersyntaks

Komplett referanse for kodelister og filteruttrykk i PxWebApi v2.

---

## Finne kommunekoder

Når brukeren nevner en kommune du ikke kjenner koden til — wildcards i `valueCodes` matcher bare *koder*, ikke kommunenavn. Strategi:

1. Søk etter kommunenavnet med `/tables?query=…` — API-et søker i variabelverdier og bekrefter at kommunen finnes
2. Hent metadata for en relevant tabell (f.eks. 07459) — scan `category.label` i Region-dimensjonen for kommunenavnet og finn koden
3. Alternativt: bruk wildcard på fylkeskoden (f.eks. `34*` for Innlandet, `345?` for å snevre inn) og identifiser kommunen i resultatene
4. Klass API har komplett kommuneklassifikasjon: `https://data.ssb.no/api/klass/v1/classifications/131.json`

---

## Kodelister

### To typer kodelister

| Type | Prefix | Beskrivelse | Eksempel |
|---|---|---|---|
| **Aggregation** | `agg_` | Slår sammen verdier til høyere nivå | `agg_KommFylker` (kommuner → fylker) |
| **Valueset** | `vs_` | Viser et alternativt verdisett | `vs_Fylker2024` (kun fylkeskoder) |

Forskjellen: En **aggregering** mapper mange-til-én (flere kommuner → ett fylke). Et **valueset** er bare et annet utvalg av verdier (f.eks. kun fylkeskoder i stedet for alle regioner).

### Finne tilgjengelige kodelister

Kodelister er listet i metadata under `dimension.{variabel}.extension.codelists`:

```json
{
  "extension": {
    "codelists": [
      {
        "id": "agg_KommFylker",
        "label": "Fylke (2024)",
        "type": "Aggregation",
        "links": [{ "rel": "metadata", "href": "/codelists/agg_KommFylker" }]
      },
      {
        "id": "vs_Fylker2024",
        "label": "Fylke (2024)",
        "type": "Valueset",
        "links": [{ "rel": "metadata", "href": "/codelists/vs_Fylker2024" }]
      }
    ]
  }
}
```

### Bruke kodeliste i metadata-oppslag

Hent metadata med kodeliste ferdig aktivert:
```
GET /tables/07459/metadata?codelist[Region]=agg_KommFylker
```
Da viser Region-variabelen aggregerte koder (fylkene) i stedet for alle kommuner.

### Bruke kodeliste i data-query

**POST:**
```json
{
  "selection": [
    {
      "variableCode": "Region",
      "codelist": "agg_KommFylker",
      "valueCodes": ["F-03", "F-11", "F-46"]
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

**GET:**
```
GET /tables/07459/data?lang=no&valueCodes[ContentsCode]=Personer1&valueCodes[Tid]=top(5)&valueCodes[Region]=F-03,F-11,F-46&codelist[Region]=agg_KommFylker
```

Husk: `ContentsCode` og `Tid` er aldri eliminerbare og må alltid være med i spørringen. Dette er en endring fra API v1 der Tid kunne utelates.

**Uten kodeliste** — du kan også spørre etter enkeltverdier som finnes i en kodeliste uten å angi kodelisten eksplisitt. API-et finner riktig kode automatisk.

### Viktig om kodeprefiks i grupperinger

Aggregeringskodelister bruker prefiks på kodene:
- `agg_KommFylker` bruker **`F-`**-prefiks: `F-03` (Oslo), `F-11` (Rogaland), `F-46` (Vestland)
- `agg_KommSummer` bruker **`K-`**-prefiks: `K-0301` (Oslo), `K-3103` (Moss)

`agg_KommFylker` og `agg_KommSummer` gir begge konsistente tidsserier over kommune- og fylkesendringer. `agg_KommFylker` aggregerer til fylkesnivå, `agg_KommSummer` gir summerte kommunetall.

### Slå opp kodelistens innhold

```
GET /codelists/agg_KommFylker?lang=no
```

Returnerer:
```json
{
  "id": "agg_KommFylker",
  "label": "Fylker 2024, sammenslåtte tidsserier",
  "language": "no",
  "type": "Aggregation",
  "values": [
    { "code": "F-31", "label": "Østfold", "valueMap": ["0101", "3124", "0103", ...] },
    { "code": "F-32", "label": "Akershus", "valueMap": ["3232", "3234", ...] },
    { "code": "F-03", "label": "Oslo - Oslove", "valueMap": ["0399", "0301"] },
    { "code": "F-34", "label": "Innlandet", "valueMap": ["3419", "3420", ...] },
    ...
  ]
}
```

`valueMap` viser hvilke opprinnelige kommunekoder (historiske og gjeldende) som aggregeres inn i fylkeskoden. Labelen `"Fylker 2024, sammenslåtte tidsserier"` betyr at kodelisten gir konsistente tidsserier bakover ved å samle alle historiske kommunekoder under gjeldende fylkesstruktur.

### Vanlige kodelister

| Kodeliste-ID | Variabel | Beskrivelse |
|---|---|---|
| `agg_KommFylker` | Region | Kommuner aggregert til fylker (gjeldende grenser) |
| `agg_KommSummer` | Region | Kommuner summert med gjeldende grenser — gir konsistente tidsserier over kommunesammenslåinger |
| `vs_Fylker2024` | Region | Kun fylkeskoder |
| `agg_FemAarigGruppering` | Alder | 5-årige aldersgrupper (0-4, 5-9, ...) |
| `agg_TiAarigGruppering` | Alder | 10-årige aldersgrupper (0-9, 10-19, ...) |
| `agg_RegHFRHF` | Region | Helseforetaksregioner |
| `agg_Nace17` | NACE | 17 næringsgrupper |
| `vs_CoiCop2018Kpi01` | VareTjenesteGrp | KPI: alle nivåer av varer og tjenester (COICOP 2018) — brukes i tabell 14700 |
| `agg_CoiCop2018Kpi011` | VareTjenesteGrp | KPI: hovedgruppenivå i COICOP 2018 (12 hovedgrupper) — brukes i tabell 14700 |

NB: Kodeliste-IDer varierer mellom tabeller. Sjekk alltid metadata for den aktuelle tabellen.

### outputValues-parameter (ved bruk av grupperinger)

Når du bruker en aggregerings-kodeliste, kan `outputValues[variabel]` styre hva som returneres:

| Verdi | Beskrivelse | Typisk bruk |
|---|---|---|
| `aggregated` | Returner aggregerte (summerte) verdier | `agg_KommSummer` for sammenslåtte kommunetall over tid |
| `single` | Returner enkeltverdier fra kodelisten uten summering | `agg_Fylker2024` for å velge ut kun gjeldende fylker |

Eksempel — sammenslåtte kommunetall for Moss over tid:
```
GET /tables/07459/data?valueCodes[Region]=K-3103&valueCodes[Tid]=*&valueCodes[ContentsCode]=Personer1&codelist[Region]=agg_KommSummer&outputValues[Region]=aggregated
```

---

## Filteruttrykk i valueCodes

Disse uttrykkene kan brukes i `valueCodes`-arrayet (POST) eller som verdier i `valueCodes`-parameteren (GET).

### Funksjonsbaserte filtre

| Uttrykk | Beskrivelse | Eksempel |
|---|---|---|
| `top(N)` | Siste N verdier (nyeste) | `top(5)` → siste 5 perioder |
| `bottom(N)` | Første N verdier (eldste) | `bottom(3)` → 3 eldste perioder |
| `from(verdi)` | Fra og med (inklusivt) | `from(2020)` → 2020 og fremover |
| `to(verdi)` | Til og med (inklusivt) | `to(2022)` → opp til og med 2022 |
| `range(fra,til)` | Intervall (inklusivt begge) | `range(2018,2023)` |

Disse brukes som **eneste element** i valueCodes-arrayet — ikke kombiner med eksplisitte koder.

### Wildcard-filtre

| Uttrykk | Beskrivelse | Eksempel |
|---|---|---|
| `*` | Alle verdier, eller matcher null eller flere tegn | `*` alene = alle verdier; `03*` = alle koder som starter med "03" |
| `?` | Matcher nøyaktig ett tegn | `??` = alle tosifrede koder |

`*` alene i valueCodes betyr "velg alle verdier for denne variabelen". Kombinert med en kodeliste betyr det "alle verdier i kodelisten".

Wildcards kan kombineres med eksplisitte koder i samme valueCodes-array:
```json
{ "variableCode": "Region", "valueCodes": ["0301", "46*"] }
```
→ Oslo + alle kommuner i Vestland fylke.

### Tidsformater

Formatet i valueCodes må matche tabellens `timeUnit`:

| timeUnit | Format | Eksempel |
|---|---|---|
| Annual | `YYYY` | `"2024"` |
| Monthly | `YYYYMNN` | `"2024M06"` (juni 2024) |
| Quarterly | `YYYYKN` | `"2024K2"` (Q2 2024) |
| Weekly | `YYYYUNN` | `"2024U01"` (uke 1, 2024) |

NB: Tidskodene bruker norske bokstaver uavhengig av `lang`-parameter: `K` for kvartal og `U` for uke — ikke Q/W. Eksempel: tabell 03024 (ukentlig lakseeksport) har perioder som `2026U23`.

### Vanlige scenarier

**Tidsfiltre:**

| Behov | valueCodes |
|---|---|
| Siste år | `["top(1)"]` |
| Siste 5 år | `["top(5)"]` |
| 2015–2020 | `["range(2015,2020)"]` |
| Fra 2020 og fremover | `["from(2020)"]` |
| Opp til og med 2022 | `["to(2022)"]` |
| Spesifikke år | `["2018", "2020", "2022"]` |
| Årsendring, siste 13 måneder | `["top(13)"]` |
| Spesifikk måned | `["2024M06"]` |
| Alle verdier | `["*"]` |

**Geografiske filtre:**

| Behov | valueCodes | Kommentar |
|---|---|---|
| Hele landet | `["0"]` | Gjelder 07459 og mange andre tabeller, men er ikke universell — sjekk alltid `category.index` i metadata |
| Oslo kommune | `["0301"]` | |
| Bergen kommune | `["4601"]` | |
| Alle i Vestland | `["46*"]` | Wildcard |
| Alle fylker | `["*"]` med `codelist: "agg_KommFylker"` | Kodelisten begrenser * til fylker |
| Spesifikke fylker | `["F-03","F-11","F-46"]` med `codelist: "agg_KommFylker"` | NB: F-prefiks |

---

## Viktige regler

1. **Funksjonsfiltre brukes alene** — `top()`, `from()`, `range()` osv. er eneste element i arrayet
2. **Wildcards kan kombineres** med eksplisitte koder i samme array
3. **Koder må matche metadata** — bruk metadata for å se gyldige koder
4. **Tidsformat varierer per tabell** — sjekk `timeUnit`
5. **Kodene er strenger** — alltid i anførselstegn, også rent numeriske
6. **Kodeliste-IDer er case-sensitive** — bruk nøyaktig ID fra metadata
