# Endringslogg — ssb-pxwebapi-v2

Gjeldende versjon står i `SKILL.md`-frontmatter under `metadata.version`.
Har din kopi ingen `metadata.version`, er den fra før 2026-06-12 — last ned ny.

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


