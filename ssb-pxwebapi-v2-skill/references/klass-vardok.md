# Klass og VarDok — SSBs metadata-systemer

json-stat2-metadata fra `/tables/{id}/metadata` inneholder URN-er under `link.describedby` som kobler variabler til SSBs metadata-systemer. Det finnes to typer.

## Klassifikasjoner (Klass)

URN-er på formen `"urn:ssb:classification:klass:131"` peker til SSBs system for klassifikasjoner og kodelister. Tallet til slutt er klassifikasjons-ID. Omskrives til Klass API:

- `https://data.ssb.no/api/klass/v1/classifications/131.json`

Eksempel: `"urn:ssb:classification:klass:691"` → `https://data.ssb.no/api/klass/v1/classifications/691.json`

Klass er nyttig for:

- Fullstendige kodeverk med historikk
- Korrespondansetabeller (gammel→ny kommunestruktur)
- Gyldighetsperioder for koder

Komplett kommuneklassifikasjon ligger på ID 131.

## Variabeldefinisjoner (VarDok)

URN-er på formen `"urn:ssb:conceptvariable:vardok:3380"` peker til SSBs variabeldefinisjoner. Tallet til slutt er variabel-ID. Omskrives til:

- Norsk: `https://www.ssb.no/a/metadata/conceptvariable/vardok/3380/nb`
- Engelsk: `https://www.ssb.no/a/metadata/conceptvariable/vardok/3380/en`

VarDok gir definisjoner, avgrensninger og bakgrunnsinformasjon for statistiske begreper.
