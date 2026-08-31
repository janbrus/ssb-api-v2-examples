# Eval-scenarier — ssb-pxwebapi-v2

Typiske brukerspørsmål med fasit (forventet tabell-ID + endepunktsekvens).
Kjøres via `skill-creator` ved større endringer i SKILL.md eller references/.
Vedlikeholder-internt — ikke med i distribusjons-zip eller README-filtreet.

Fasitverdier verifisert mot live API 2026-06-12. Hvis et scenario feiler, sjekk
først om tabellen har endret koder/struktur (kjør `scripts/check_examples.py`)
før du antar at skillen er problemet.

## 1. «Hvor mange bor i Oslo?»

- **Forventet tabell:** 07459
- **Sekvens:** (`GET /tables?query=folkemengde` hvis ukjent) → `GET /tables/07459/metadata` → `POST /tables/07459/data`
- **Nøkkelvalg:** `Region=0301`, `ContentsCode=Personer1`, `Tid=top(1)`
- **Suksess:** Ett tall med årstall og «Kilde: SSB, tabell 07459»

## 2. «KPI siste år»

- **Forventet tabell:** 14700 (IKKE 03013/03014 — begge avsluttet)
- **Sekvens:** metadata → data
- **Nøkkelvalg:** `ContentsCode=KpiIndMnd` (indeks) eller `Tolvmanedersendring` (12-mnd endring), `Tid=top(13)`
- **Suksess:** Månedlig serie med enhet (indeks 2025=100 / prosent), kilde

## 3. «Folketall i Moss over tid»

- **Forventet tabell:** 07459 med kodeliste `agg_KommSummer`
- **Nøkkelvalg:** `codelist[Region]=agg_KommSummer`, `valueCodes[Region]=K-3103` (K-prefiks!). `outputValues[Region]=aggregated` er valgfri — den har ingen observerbar effekt (verifisert 2026-08-30), så scenarioet skal **ikke** kreve den
- **Suksess:** Konsistent tidsserie over kommunesammenslåingene (K-3103 aggregerer 0104/0136/3002/3103); ingen 400-feil pga. manglende prefiks. Svaret oppgir hvilken kodeliste som er brukt — responsen registrerer det ikke selv

## 4. «Arbeidsledighet nå»

- **Forventet tabell:** 13760 (AKU månedlig)
- **Sekvens:** søk («arbeidsledige» / AKU) → metadata → data
- **Nøkkelvalg:** riktig ContentsCode for arbeidsledige; vær eksplisitt om sesongjustert vs. ujustert; `Tid=top(1)`
- **Suksess:** Nyeste måned (AKU har ~2 mnd publiseringsetterslep), enhet tydelig (prosent av arbeidsstyrken vs. antall i 1000)

## 5. «Lakseeksport denne uka»

- **Forventet tabell:** 03024
- **Nøkkelvalg:** `Tid=top(1)` — ukeformat er `ÅÅÅÅUnn` (f.eks. `2026U23`), IKKE `W`
- **Suksess:** Kilopris og tonn for siste uke, ukenummer riktig gjengitt

## 6. «Sammenlign befolkningen i fylkene»

- **Forventet tabell:** 07459 med kodeliste `agg_KommFylker`
- **Nøkkelvalg:** `codelist[Region]=agg_KommFylker`, `valueCodes[Region]=*` (eller F-koder med F-prefiks), `Tid=top(1)`
- **Suksess:** Én rad per fylke, ikke per kommune

## 7. «Eksporter boligprisindeksen til Excel»

- **Forventet tabell:** 07221
- **Nøkkelvalg:** `outputFormat=xlsx`, `outputFormatParams=UseCodesAndTexts`/`IncludeTitle`; `ContentsCode=Boligindeks` (IKKE `KvPris` — finnes ikke i 07221)
- **Suksess:** Gyldig xlsx-fil levert

## 8. «What is the population of Norway?» (engelsk)

- **Forventet tabell:** 07459 med `lang=en`
- **Nøkkelvalg:** `Region=0`, `ContentsCode=Personer1`, `Tid=top(1)`
- **Suksess:** Svar på engelsk, engelsk tallformat (5,550,203), «Source: Statistics Norway, table 07459»

## 9. «Folketallet i Norge i 1875»

- **Forventet atferd:** Ikke pxwebapi-kall som primærsvar — historiske tall fra før Statistikkbanken-perioden skal rutes til `ssb-histstat`-skillen (hvis tilgjengelig), ellers veiledning per Fallback-seksjonen
- **Suksess:** Skillen forsøker ikke å presse spørsmålet inn i moderne tabeller med feil tidsdekning

## 10. «Hva er styringsrenten?»

- **Forventet atferd:** Rutes til `norges-bank-api`-skillen (hvis tilgjengelig) — ikke SSB-tabeller; sentralbankdata finnes ikke i Statistikkbanken
- **Suksess:** Ingen forsøk på å besvare med SSB-uttrekk. Generell regel som også gjelder alle scenariene over: presentasjonen kommenterer kun tall fra det hentede uttrekket — aldri innblandede tall fra andre kilder
