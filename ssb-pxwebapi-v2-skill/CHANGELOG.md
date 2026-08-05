# Endringslogg — ssb-pxwebapi-v2

Gjeldende versjon står i `SKILL.md`-frontmatter under `metadata.version`.
Har din kopi ingen `metadata.version`, er den fra før 2026-06-12 — last ned ny.

## 1.3.1 — 2026-08-05

- Rate limit annonseres nå i HTTP-responsheadere i stedet for i `/config` (headere observert 2026-08-05: `x-ratelimit-limit: 40`, `x-ratelimit-policy: 40;w=60s`, `x-ratelimit-remaining`, `x-ratelimit-resource: SB_API_1MIN`). `maxCallsPerTimeWindow`/`timeWindow` står igjen i `/config`-responsen, men er nullstilt til `0` og ikke lenger i bruk (`0` betyr ikke «ingen grense»). Gjeldende grense er 40 kall per 60 sekunder (tidligere dokumentert som 30/minutt). Ny headertabell i `references/api-details.md`; 429-avsnittet og `/config`-eksempelet i `references/troubleshooting.md` samt endepunktvalg og «Viktige begrensninger» i SKILL.md oppdatert.
- Sjekket søsterskillen: endringen gjelder ikke SCB (per 2026-08-05) — SCBs API viser fortsatt rate limit i `/config`, så `scb-pxwebapi-v2` beholder `/config`-formuleringen uendret.

## 1.3.0 — 2026-07-29

- Dokumentert SSBs nye `link.related` i metadata-responsen (verifisert mot live API 2026-07-29): rot-nivå gir ferdige lenker til statistikksiden (`relation: "statistics-homepage"`) og «Om statistikken» (`relation: "about-statistics"`) med kortnavnet direkte i `extension.metaid` (`KORTNAVN:<kortnavn>`); variabel-nivå gir menneskelesbare Klass-/VarDok-lenker med label (`relation: "definitions"`, `metaid` = URN-en fra `describedby`). Lenker og labels følger `lang`-parameteren. Kun i metadata-responser — data-responser har fortsatt bare `describedby`. (`references/klass-vardok.md` omstrukturert, ny `link`-oppføring i `references/json-stat2.md`, Steg 2/Steg 3 og «Kobling til SSBs metadata-systemer» i SKILL.md oppdatert)
- Dokumentert `describedby`-nøkkelstrukturen: `extension`-nøkkelen er variabelnavn (URN-er for hele variabelen) eller enkeltverdi-kode (URN per verdi — vanlig for `ContentsCode`)
- Sjekket søsterskillen: SCBs API (`statistikdatabasen.scb.se/api/v2`) eksponerer ikke `link.related` per 2026-07-29 — ingen endring i `scb-pxwebapi-v2`

## 1.2.0 — 2026-06-15

- Ny gjennomgående regel for tidsdimensjonen: foretrekk relative tidsfiltre `top(N)`/`from(verdi)` framfor `range(fra,til)` og eksplisitte enkeltverdier — relative filtre fanger automatisk opp nye perioder, så delbare URL-er og lagrede spørringer holder seg oppdaterte (SKILL.md Steg 3/Steg 4 + `references/codelists-and-filters.md`)
- Dokumentert **pyjstat** (https://pypi.org/project/pyjstat/) i `references/json-stat2.md` — Python-bibliotek for å lese/skrive json-stat og konvertere til/fra pandas DataFrame
- Lagt til **PxWebApiData** (https://cran.r-project.org/package=PxWebApiData) i README — R-pakke som henter PxWeb/PxWebApi-data (SSB, SCB m.fl.) inn i R

## 1.1.0 — 2026-06-12

- Versjonering innført (`metadata.version` i SKILL.md-frontmatter + denne loggen)
- Ny `references/mcp-tools.md` og «Verktøyvalg»-seksjon i SKILL.md: mapping til `pxweb-mcp` MCP-verktøyene (verifisert mot v2.0.0), begrensninger og `--url`-forbehold
- Dokumentert defaultselection-atferd: GET uten seleksjonsparametre returnerer defaultselection-data, ikke feil og ikke hele tabellen (Steg 4 + Fallgruver); feilmeldingen siteres nå literalt («mandantory», sic)
- Rettet Excel-eksemplet for tabell 07221: `ContentsCode=KvPris` → `Boligindeks` (`KvPris` finnes ikke i tabellen)
- Slanket Steg 3 — formatdetaljer (`measuringType`, `priceType`, `basePeriod` m.fl.) samlet i `references/json-stat2.md`
- Kryssreferanser: `ssb-chart-skill` (Steg 5), `ssb-histstat` (Fallback) og tredjeparts `norges-bank-api` (Steg 1) — alle med «hvis tilgjengelig»-forbehold
- Ny regel i Steg 5 og Fallgruver: svar kommenterer kun tall fra eget uttrekk — data fra andre kilder hentes aldri inn i samme svar
- «Hele landet»-koden `0` dokumentert som ikke-universell (sjekk `category.index`); `valueCodes` camelCase-fiks
- CI: nytt `scripts/check_examples.py` validerer alle eksempel-URL-er og -spørringer mot live API; nytt `scripts/build_zip.sh` + zip-synksjekk i workflow
- Zip-distribusjonen inneholder nå kun brukervendte filer (SKILL.md, README.md, CHANGELOG.md, references/) — vedlikeholdsfiler (CLAUDE.md, scripts/, CI-oppsett) ligger kun i repoet

Utgaver før 2026-06-12 var uversjonerte. Rettelser (2026-06-11): 

- ukeformat `U` (ikke `W`), 

- nytt parquet-outputformat, 

- json-stat2-eksempel.


