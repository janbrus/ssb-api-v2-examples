# Claude Skill: SCB PxWebApi v2 - BETA

En [Claude Skill](https://support.claude.com/en/articles/12512180-use-skills-in-claude) som lär AI-verktyg som Claude att söka, utforska och hämta data från SCB:s Statistikdatabas via PxWebApi v2.

## Vad skillen gör

- Guidar AI-assistenten genom rätt arbetsflöde: sök → metadata → query → presentera
- Täcker alla endpoints i PxWebApi v2 (tabeller, metadata, kodlistor, sparade frågor, config)
- Hanterar kodlistor och aggregeringar
- På svenska och engelska

## Filstruktur

```
scb-pxwebapi-v2/
├── SKILL.md                  # Huvudinstruktioner och arbetsflöde
├── README.md                 # Denna fil
├── CHANGELOG.md              # Ändringslogg — gällande version står i SKILL.md-frontmatter (metadata.version)
└── references/
    └── json-stat2.md         # json-stat2 formatspecifikation (Dataset, row-major, statuskoder — gäller även Eurostat, World Bank)
```

## Installation

### Claude.ai

1. Packa de användarvända filerna som ZIP — endast `SKILL.md`, `README.md`, `CHANGELOG.md` och `references/` (utelämna `CLAUDE.md`, den är repo-intern):

   ```bash
   zip -r scb-pxwebapi-v2-skill.zip scb-pxwebapi-v2/SKILL.md scb-pxwebapi-v2/README.md scb-pxwebapi-v2/CHANGELOG.md scb-pxwebapi-v2/references/
   ```

2. Gå till **Settings > Features > Skills**
3. Ladda upp ZIP-filen

### Claude Code

```bash
cp -r scb-pxwebapi-v2 ~/.claude/skills/scb-pxwebapi-v2
```

## MCP-servrar

För att Claude faktiskt ska kunna *anropa* API:et behövs ett verktyg:

- **@jarib/pxweb-mcp** (https://www.npmjs.com/package/@jarib/pxweb-mcp) — open source, fungerar med PxWeb-installationer inklusive SCB
- Egen MCP-server med FastMCP eller liknande

## Licens

Data från SCB är öppen statistik. Se SCB:s användarvillkor för detaljer.
