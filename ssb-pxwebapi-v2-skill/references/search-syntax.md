# Søkesyntaks for /tables?query=

API-et søker i tabelltitler, variabler og variabelverdier (case-insensitivt). Søkemotoren er **Lucene** (via Lucene.Net i PxWebApi), så `query`-parameteren tolkes etter [Lucene Query Parser-syntaks](https://lucene.apache.org/core/2_9_4/queryparsersyntax.html) — feltbegrensning (`felt:verdi`), wildcards (`*`, `?`), fuzzy (`~N`), nærhet (`"…"~N`), intervaller (`[a TO b]`) og boolske operatorer (`AND`, `OR`, `NOT`) virker derfor som i andre Lucene-baserte søk (Elasticsearch, Solr). Følgende mønstre er bekreftet i PxWebApi v2:

## Feltbegrensning

- `title:barn` — begrens søket til tittelfeltet
- `updated:20250908*` — søk etter oppdateringsdato
- `updated:[20250908 TO 20250912*]` — datointervall

## Mønstermatching

- `anlegg*` — trunkering, matcher alt som starter med "anlegg"
- `konsumpris~1` — fuzzy søk, `~N` tillater N tegns avvik
- `"varenummer hs" ~5` — nærhetssøk, finner ordene innen 5 ord fra hverandre

## Boolske operatorer

- `trend AND anlegg*` — begge må matche
- `title:foretak AND title:(F)` — fylkesnivå-tabeller om foretak
- Standard mellom ord er OR; bruk AND/NOT eksplisitt

## Synonymer og termer

Bruk norske fagtermer:

- "konsumprisindeks" (ikke "KPI")
- "sysselsatte" (ikke "jobber")
- "folkemengde" (ikke "befolkning")

Vurder synonymer: "folkemengde" ≈ "befolkning" ≈ "innbyggere".

## Tidsmessig avgrensning

- `pastDays=N` — kun tabeller oppdatert siste N dager
- `includeDiscontinued=false` (default) — skjuler avsluttede serier; sett `true` for historiske data
