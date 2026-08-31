# Claude Skill: PxWebApi v2 (Generic)

A [Claude Skill](https://support.claude.com/en/articles/12512180-use-skills-in-claude) for accessing official statistics from any PxWebApi v2 installation. 

This skill is a reduced, vendor-neutral version derived from the Statistics Norway skill `ssb-pxwebapi-v2` (see link at the bottom of this README).

## What is PxWebApi v2?

PxWebApi v2 is a REST API for statistical databases, developed by Statistics Sweden (SCB) and used by national statistical institutes across the Nordics and beyond. It provides a standardized way to search, explore, and retrieve official statistics.

Known v2 installations:

- **Statistics Norway (SSB):** `https://data.ssb.no/api/pxwebapi/v2`
- **Statistics Sweden (SCB):** `https://statistikdatabasen.scb.se/api/v2`

## What this skill does

- Guides Claude through the correct workflow: search → metadata → query → present
- Covers all PxWebApi v2 endpoints (tables, metadata, codelists, saved queries, config)
- Handles codelists and aggregations
- Works with any PxWebApi v2 installation — just specify the base URL
- Fully in English for international use
- Output format follows the open [json-stat2](https://json-stat.org/) spec, also used by Eurostat and World Bank

## What this skill does NOT include

- Country-specific table lists (no `common-tables.md` — table IDs differ per installation)
- Country-specific codelist IDs or regional codes
- Country-specific metadata conventions (URN links to classification systems)

For a comprehensive SSB-specific skill with curated table lists, codelist documentation, and Norwegian/English support, see [ssb-pxwebapi-v2](https://github.com/janbrus/ssb-api-v2-examples/tree/main/claude-skill).

## File structure

```
generic-pxweb-v2-skill/
├── SKILL.md                       # Main skill entrypoint (loaded on trigger)
├── README.md                      # This file
└── references/                    # Loaded on demand
    ├── json-stat2.md              # json-stat2 format spec (Dataset, row-major indexing, status codes — also applies to Eurostat, World Bank)
    ├── api-details.md             # PxWebApi-specific configuration (/config endpoint)
    ├── codelists-and-filters.md   # Codelists, filters, aggregations, valueCodes syntax
    └── troubleshooting.md         # Common errors and fixes
```

## Installation

### Claude.ai

1. Download or create a ZIP of this folder
2. Go to **Settings > Features > Skills**
3. Upload the ZIP file

### Claude Code

```bash
cp -r generic-pxweb-v2-skill ~/.claude/skills/generic-pxweb-v2-skill
```

## MCP servers

For Claude to call the API directly, you need an MCP server:

- **@jarib/pxweb-mcp** (https://www.npmjs.com/package/@jarib/pxweb-mcp) — open source, works with any PxWeb v2 installation
- Or build your own with FastMCP or similar

## License

PxWebApi v2 is open source: https://github.com/PxTools/PxWebApi. Data licensing depends on the individual agency.
