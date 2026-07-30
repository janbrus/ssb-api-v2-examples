# Klass og VarDok — SSBs metadata-systemer

Metadata-responsen fra `/tables/{id}/metadata` kobler tabellen og variablene til SSBs metadata-systemer via `link`-objekter, med to relasjonstyper:

- **`link.describedby`** — maskinlesbare URN-er, for oppslag mot Klass API
- **`link.related`** — ferdige, menneskelesbare lenker med label, for henvisning og videre lesning

`link` finnes på både rot-nivå (om statistikken) og variabel-nivå (om den enkelte dimensjonen). Data-responser (`/tables/{id}/data`) inneholder kun `describedby` — `related` finnes bare i metadata-responsen.

## Rot-nivå: statistikkside og «Om statistikken»

`<root>.link.related` gir ferdige lenker til statistikken tabellen tilhører:

```json
"link": {
  "related": [
    {
      "extension": { "relation": "statistics-homepage", "metaid": "KORTNAVN:folkemengde" },
      "href": "https://www.ssb.no/folkemengde",
      "label": "Statistikkside",
      "type": "text/html"
    },
    {
      "extension": { "relation": "about-statistics", "metaid": "KORTNAVN:folkemengde" },
      "href": "https://www.ssb.no/folkemengde#om-statistikken",
      "label": "Definisjoner og forklaringer",
      "type": "text/html"
    }
  ]
}
```

- `relation: "statistics-homepage"` → statistikksiden (`ssb.no/<kortnavn>`)
- `relation: "about-statistics"` → «Om statistikken»-siden med definisjoner og forklaringer
- `extension.metaid` = `KORTNAVN:<kortnavn>` — gir statistikkens kortnavn direkte, uten å utlede det fra `paths` (jf. Steg 2 i SKILL.md; `paths`-utledningen trengs fortsatt i søkefasen, før du har hentet metadata)
- Lenker og labels følger `lang`-parameteren: `lang=en` gir `/en/`-URL-er og engelske labels («Statistics page», «Definitions and explanations»)

## Variabel-nivå: definisjoner per variabel

`dimension.{var}.link.related` gir én lenke per klassifikasjon/variabeldefinisjon, med `relation: "definitions"` og `metaid` lik URN-en fra `describedby`:

```json
"dimension": {
  "Kjonn": {
    "link": {
      "describedby": [
        { "extension": { "Kjonn": "urn:ssb:classification:klass:2" } }
      ],
      "related": [
        {
          "extension": { "relation": "definitions", "metaid": "urn:ssb:classification:klass:2" },
          "href": "https://www.ssb.no/klass/klassifikasjoner/2",
          "label": "Standard for kjønn",
          "type": "text/html"
        }
      ]
    }
  }
}
```

Labelen forteller hva lenken er — Klass-lenker heter «Standard for …» (f.eks. «Standard for kommuneinndeling»), VarDok-lenker «Variabeldefinisjon av …» (f.eks. «Variabeldefinisjon av Månedslønn (kr)»). Bruk disse ferdige lenkene når du henviser brukeren til definisjoner — URN-omskriving trengs bare for maskinlesbare oppslag (under).

I `describedby` er `extension`-nøkkelen enten variabelnavnet (URN-er som gjelder hele variabelen, evt. flere adskilt med mellomrom) eller en enkeltverdi-kode (URN som definerer akkurat den verdien — vanlig for `ContentsCode`, der hver statistikkvariabel kan ha sin egen VarDok-definisjon).

## Klassifikasjoner (Klass)

URN-er på formen `"urn:ssb:classification:klass:131"` peker til SSBs system for klassifikasjoner og kodelister. Tallet til slutt er klassifikasjons-ID. Omskrives til Klass API for maskinlesbart oppslag:

- `https://data.ssb.no/api/klass/v1/classifications/131.json`

Eksempel: `"urn:ssb:classification:klass:691"` → `https://data.ssb.no/api/klass/v1/classifications/691.json`

(Menneskelesbar side: `https://www.ssb.no/klass/klassifikasjoner/131` — samme URL som `link.related` gir ferdig.)

Klass er nyttig for:

- Fullstendige kodeverk med historikk
- Korrespondansetabeller (gammel→ny kommunestruktur)
- Gyldighetsperioder for koder

Komplett kommuneklassifikasjon ligger på ID 131.

## Variabeldefinisjoner (VarDok)

URN-er på formen `"urn:ssb:conceptvariable:vardok:3380"` peker til SSBs variabeldefinisjoner. Tallet til slutt er variabel-ID. Omskrives til:

- Norsk: `https://www.ssb.no/a/metadata/conceptvariable/vardok/3380/nb`
- Engelsk: `https://www.ssb.no/a/metadata/conceptvariable/vardok/3380/en`

(Samme URL-er som `link.related` gir ferdig, med språk etter `lang`-parameteren.)

VarDok gir definisjoner, avgrensninger og bakgrunnsinformasjon for statistiske begreper.
