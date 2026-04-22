# Claude Skill: SSB PxWebApi v2

En [Claude Skill](https://support.claude.com/en/articles/12512180-use-skills-in-claude) som lærer AI verktøy som Claude å søke, utforske og hente data fra SSBs Statistikkbank via PxWebApi v2.

## Hva skillen gjør

- Guider AI-assisteneten gjennom riktig arbeidsflyt: søk → metadata → query → presenter
- Dekker alle endepunkter i PxWebApi v2 (tabeller, metadata, kodelister, lagrede spørringer, config)
- Håndterer kodelister og aggregeringer (fylker, kommunesammenslåinger, aldersgrupper)
- Støtter norsk og engelsk
- Inkluderer kurert liste over ~60 mye brukte tabeller
- Refererer til SSBs Klass- og VarDok-systemer via URN-er i metadata

## Filstruktur

```
claude-skill/
├── SKILL.md                              # Hovedinstruksjoner og arbeidsflyt
└── references/
    ├── api-details.md                    # json-stat2-format og driftsinformasjon
    ├── codelists-and-filters.md          # Kodelister, filtersyntaks, outputValues
    ├── common-tables.md                  # Kurert liste over vanlige tabeller
    └── troubleshooting.md               # Feilsøking og standardtegn
```

## Installasjon

### For AI-plattformer som støtter skills/prompts


1. Last ned ZIP-filen: [ssb-pxwebapi-v2-skill.zip](../ssb-pxwebapi-v2-skill.zip) (eller pakk denne mappen som ZIP selv)
2. Gå til **Settings > Features > Skills** i Claude.ai
3. Last opp ZIP-filen

### Claude Code

Kopier mappen til din globale eller prosjektspesifikke skills-katalog:

```bash
# Globalt (tilgjengelig i alle prosjekter)
cp -r claude-skill ~/.claude/skills/ssb-pxwebapi-v2

# Per prosjekt
cp -r claude-skill .claude/skills/ssb-pxwebapi-v2
```

### Andre

Følg plattformens dokumentasjon for å legge til tilpassede instruksjoner eller "skills".


## Bruk sammen med MCP-server eller API-klient

Skillen er ren kunnskap — den gir AI-assistenten *veiledning* for hvordan PxWebApi v2 fungerer. For at Claude faktisk skal kunne *kalle* API-et, trenger du også verktøy. Alternativer:


- **@jarib/pxweb-mcp** (https://www.npmjs.com/package/@jarib/pxweb-mcp) — open source MCP-server for PxWeb-APIer, fungerer med SSB og andre statistikkbyråer som bruker PxWeb V2
- **TRYs MCP-server** (https://tools.try.no/ssb-mcp) — ferdig hosted MCP-server med `ssb_search`, `ssb_get_data` m.fl.
- **Egen MCP-server** — bygg din egen med FastMCP eller lignende
- **Direkte API-kall** — skillen beskriver endepunktene slik at Claude kan konstruere korrekte URL-er

## Lisens

Skillen er laget som et hjelpemiddel for bruk av SSBs åpne API. Data fra SSB er lisensiert under [CC BY 4.0](https://www.ssb.no/diverse/lisens).

## Visualisering

TRY har laget en egen skill for visualisering av SSB-data i SSBs offisielle stil (farger, typografi, diagramtyper): [ssb-dataviz-skill.zip](https://tools.try.no/ssb-mcp/ssb-dataviz-skill.zip). Den er designet for å brukes sammen med denne API-skillen — last opp begge i Claude.ai for komplett arbeidsflyt fra datahenting til ferdig graf. Dataviz-skillen styrer visualisering, ikke datahenting — den fungerer uavhengig av hvilken MCP-server som brukes.
