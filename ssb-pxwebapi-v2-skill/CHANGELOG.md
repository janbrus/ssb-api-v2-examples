# Endringslogg — ssb-pxwebapi-v2

Gjeldende versjon står i `SKILL.md`-frontmatter under `metadata.version`.
Har din kopi ingen `metadata.version`, er den fra før 2026-06-12 — last ned ny.

## 1.4.1 — 2026-08-30

Faktarettelser og utdypninger, alle verifisert mot live API 2026-08-30. Ingen endring i arbeidsflyten.

* **Obligatoriske noter dokumentert** (`extension.noteMandatory`, nøklet på note-indeks). Satt på 14700 `{"1": true}`, 14710 `{"0": true}` og 03013 `{"2": true, "3": true}` — altså KPI-tabellene skillen bruker mest. Noten på 14700/14710 varsler at referanseåret ble 2025=100 f.o.m. 2026, og at **endringstall beregnet fra disse seriene kan avvike fra publiserte endringstall**. Steg 5 ber modellen regne ut nettopp slike endringstall, så noten er nå presentasjonsplikt tre steder: nytt kulepunkt i Dataintegritet, i Steg 3-feltlista, og i Steg 5 ved siden av `status`-regelen. Semantikken (inkl. `category.note` per verdi, styrt av `categoryNoteMandatory`) i `references/json-stat2.md`
* **Rettet: `elimination` i data-responser.** Feltet svarer på ulike spørsmål i metadata og data. I metadata er det kontrakten («kan variabelen utelates?»); i en data-respons beskriver det uttrekket du fikk — `true` bare når verdimengden fortsatt inneholder elimineringsverdien. Les aldri eliminerbarhet av et datauttrekk
* **Rettet: `eliminationValueCode` brukes av SSB, men bare i data-responser.** Sveip over 50 tabeller: feltet finnes på null dimensjoner i metadata. Det dukker opp i en data-respons som inkluderer totalen (07459 `Region` → `"0"`). SSB bruker altså begge PX-formene — `Region` har totalkoden `0` = «Hele landet», `Kjonn` har ingen totalkode og summeres på flyet. Metadata skiller dem ikke; se etter «Hele landet»/«I alt» i `category.label` i stedet
* **`outputValues[Var]` nedgradert fra nødvendig til ufarlig-men-ikke-bærende.** Seks tester ga identiske data for `aggregated`, `single` og utelatt parameter (07459 + `agg_KommSummer`/`K-0301` → `[717710, 724290, 728714]`; 07459 + `agg_KommFylker`/`*` → 18 fylkeskoder). **`outputValues[Region]=nonsense` gir HTTP 200 og samme data** — verdien valideres ikke, så en skrivefeil er usynlig. Det er kodelisten som aggregerer. Rettet i `SKILL.md`, `references/codelists-and-filters.md`, `evals/eval-scenarios.md` (scenario 3 krever ikke lenger parameteren som nøkkelvalg), `CLAUDE.md` og `README.md`. **Gjelder ikke SCB — ikke verifisert der**; `scb-pxwebapi-v2` er urørt på dette punktet
* **`extension.px.aggregallowed` dokumentert.** 8 av 12 stikkprøvde tabeller har `false`, inkludert 14700, 14710, 03013 og 07221. Verifisert nyanse: `false` er et tolkningssignal, ikke en teknisk sperre — 14700 tilbyr sju kodelister og svarer 200 på dem
* **Rettet: dataset-`extension` inneholder ikke `firstPeriod`/`lastPeriod`, og det finnes ikke noe `nextUpdate`-felt.** Faktiske nøkler hos SSB: `px`, `contact`, samt `noteMandatory` og `discontinued` når satt. `firstPeriod`/`lastPeriod`/`timeUnit` ligger på `/tables`-treffet og `GET /tables/{id}`. Rettet i `references/json-stat2.md` og Steg 3
* **Kodeliste-proveniens registreres ikke i responsen.** `codelist[Region]=agg_Fylker2024` kommer tilbake med `Region.extension = {"elimination": false, "show": "code_value"}` — ingenting navngir grupperingen. `agg_KommFylker` og `agg_KommSummer` gir tall som ser like ut og ikke er det. Steg 5 krever nå at kodeliste og utelatte dimensjoner oppgis eksplisitt
* **400-feil: `detail` finnes som regel ikke — `title` er diagnosefeltet.** Verifiserte former: `Non-existent variable`, `Non-existent value`, `Missing selection for mandantory variable` (sic), `Too many cells selected` (eneste som også setter `detail`, og da bare som gjentakelse av `title`). `type` er en ren streng, ikke en URI, så RFC 7807-formuleringen i `references/troubleshooting.md` er myknet opp. Formen er identisk hos SCB
* **`/config`-eksempelet i `references/troubleshooting.md` oppdatert til full 12-felts respons** — manglet `apiVersion` (2.3.2), `license`, `sourceReferences`, `features` og `parquet` i `dataFormats`. `sourceReferences` gir SSBs egen kildehenvisningsstreng per språk («Kilde: Statistisk sentralbyrå»)
* **`timeUnit` er eneste kilde til tidsfrekvens** — json-stat2 bærer den ikke. Relevant når et uttrekk sendes videre til `ssb-chart-skill`, som må vite om aksen er måned eller kvartal
* **SSB kjører fortsatt v1 på `https://data.ssb.no/api/v0/`** — nevnt i én seksjon under base-URL-en, med ruting til `generic-pxweb-v1-skill`. Grunnen er at brukere kommer med gamle POST-bodyer i v1-form fra skript og Power BI
* **R-pakken `PxWebApiData` støtter begge API-versjonene** (1.9.0, 2026-02-02) — egen vignett for hver, v2 via snake_case-grensesnittet `api_data()`/`get_api_data()`/`query_url()` + `meta_frames()`/`meta_code_list()`/`meta_data()` med `_1`/`_2`/`_12`-varianter, v1 via camelCase `ApiData()`. Dokumentert i SKILL.md og README. En `PxWebApiData`-oppskrift er altså **ikke** et signal om at brukeren er på v1 — sjekk base-URL-en i stedet
* Søskenskill: `scb-pxwebapi-v2` fikk i samme omgang egne endringer (integritetsseksjon, defaultselection, relative tidsfiltre, `extension`-rettelse) — se dens changelog 0.10.0. `outputValues`-funnet er **ikke** portet dit, siden det ikke er verifisert mot SCB

## 1.4.0 — 2026-08-27

* Ny toppseksjon «Dataintegritet — grunnregelen» i SKILL.md, plassert før API-oversikten: **oppgi aldri et tall som ikke er hentet fra API-et i samme samtale**. Dekker forbud mot tall fra hukommelsen og fra andre kilder, mot interpolering og framskriving, krav om å merke egne beregninger, bevare API-ets desimalpresisjon (`category.unit.decimals`), verifisere variabelkoder mot metadata, sjekke `discontinued`/`lastPeriod` og flagge foreløpige/reviderbare tall. Begrunnelsen er eksplisitt: feil tall med SSB-kildehenvisning skader tilliten til SSB, ikke bare til svaret
* Steg 5 utvidet med tre krav: skill hentede tall fra egne beregninger (vekstrater, andeler, differanser), gjør uttrekket etterprøvbart ved å vise GET-URL eller POST-body, og vis `status`-merkede verdier som de er i stedet for å erstatte dem med tall eller tomme celler. Indekser skal alltid oppgis med referanseperiode
* Fallgruver utvidet med fire integritetspunkter: tall som ikke er hentet, utfylling av hull i tidsserien, gjettede variabelkoder, egen beregning presentert som SSB-tall, og bruk av avsluttet tabell uten å si fra
* Fallback skjerpet: manglende API-tilgang skal meldes eksplisitt — uten tilgang leveres veiledning, ikke statistikk
* Rettet feil ContentsCode for tabell 14700 til `Tolvmanedersendring` (prosent) 
* `parquet` dokumentert i SKILL.md Steg 4 (var kun i `references/output-formats.md`). Parquet bruker alltid koder — `outputFormatParams=UseCodesAndTexts` gir HTTP 400
* `GET /config` bekreftet 2026-08-27: `apiVersion` 2.3.2, `maxDataCells` 800 000, `dataFormats` inkluderer `parquet`
* Repo-adresse lagt inn to steder i SKILL.md: `metadata.source` i frontmatter og en synlig linje under base-URL-en — https://github.com/janbrus/ssb-api-v2-examples/tree/main/ssb-pxwebapi-v2-skill
* Avslutta Tabell 10261 er erstattet med etterfølgeren 14824 «Pasienter, behandlinger og oppholdsdøgn i somatisk spesialisthelsetjeneste, etter kjønn, alder, bosted, aktør og diagnose» (2015-2025) i `references/common-tables.md` 
* Fjernet en avkuttet skilletegnslinje (`* *`) sist i `references/common-tables.md`.


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


