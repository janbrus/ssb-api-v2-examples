# Ändringslogg — scb-pxwebapi-v2

Gällande version står i `SKILL.md`-frontmatter under `metadata.version`.
Saknar din kopia `metadata.version` är den från före 2026-06-12 — hämta ny.
Versioner under 1.0.0 markerar beta-status (se README).

## 0.10.0 — 2026-08-30 (beta)

Allt verifierat mot live API 2026-08-30.

- **Ny toppsektion «Dataintegritet — grundregeln»**, placerad före API-översikten: **ange aldrig en siffra du inte har hämtat från API:et i denna konversation.** Sju punkter — inga siffror ur minnet, inga andra källor i samma svar, säg ifrån när API:et fallerar, ingen interpolering, märk egna beräkningar och behåll API:ets decimaler, visa `status`-värden som de är, kontrollera `discontinued`/`lastPeriod`. Regeln fanns tidigare bara i `ssb-pxwebapi-v2`; den är lika giltig här och är nu speglad i alla fyra PxWeb-skills. Rangordningen är inskriven i `CLAUDE.md` så att sektionen inte flyttas eller urvattnas vid nästa redigering
- **Rättat citat av felmeddelandet:** API:et returnerar `"Missing selection for mandantory variable"` — med SCB:s egen stavfel `mandantory`, samma som hos SSB. Skillen citerade tidigare `"mandatory"`, vilket inte matchar en sökning i loggar
- **Defaultselection-beteende dokumenterat:** ett `GET /data` utan selektionsparametrar är inget fel och ger inte hela tabellen — det returnerar tyst förvalet (`TAB638` → `size [290, 1, 1, 2]`, 580 celler, HTTP 200). Det farliga är att elimineringsbara dimensioner utanför förvalet summeras bort utan att svaret nämner det
- **Ny genomgående regel för tidsdimensionen:** föredra `top(N)`/`from(värde)` framför `range(från,till)` och explicita perioder — relativa filter fångar nya perioder automatiskt, så delbara URL:er och sparade frågor håller sig aktuella. Gäller särskilt `/savedqueries`
- **Rättat `extension`-punkten i `references/json-stat2.md`:** den listade `/tables`-fälten `firstPeriod`/`lastPeriod`/`discontinued` som dataset-`extension`. Ersatt med den faktiska tvånivåstrukturen — dataset-nivå (`px`, `contact`, `noteMandatory`, `discontinued`) och dimensionsnivå (`elimination`, `codelists`, `show`, `refperiod`, `measuringType`, `priceType`, `adjustment`, `alternativeText`, `categoryNoteMandatory`). `firstPeriod`/`lastPeriod` hör hemma i `/tables`-träffen, inte i datasetet
- Systerskill: `ssb-pxwebapi-v2` 1.4.1 släpptes samtidigt. Dess `outputValues`-fynd (parametern är inte bärande hos SSB) är **inte** portat hit — det är inte verifierat mot SCB, och parametern nämns här bara i MCP-mappningen

## 0.9.0 — 2026-06-12 (beta)

- Versionering införd (`metadata.version` i SKILL.md-frontmatter + denna logg)
- Ny sektion «Verktygsval / Tool selection»: pxweb-mcp-verktygen med begränsningar — bl.a. stödjer `language`-parametern endast `no`/`en` (använd HTTP med `lang=sv` för svenska texter) och servern måste startas med `--url https://statistikdatabasen.scb.se/api/v2` (default är norska SSB)
- Ny regel i Steg 5 och Fallgropar: svar kommenterar endast siffror från det egna uttaget — data från andra källor hämtas aldrig in i samma svar
- ZIP-distribution: endast användarvända filer paketeras (SKILL.md, README.md, CHANGELOG.md, references/) — CLAUDE.md är repo-intern; se paketeringskommandot i README

Utgåvor före 2026-06-12 saknade versionsnummer.
