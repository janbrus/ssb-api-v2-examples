# Claude Skill: PxWebApi v2 (Generic) - BETA

A [Claude Skill](https://support.claude.com/en/articles/12512180-use-skills-in-claude) for accessing official statistics from any PxWebApi v2 installation.

## What is PxWebApi v2?

PxWebApi v2 is a REST API for statistical databases, developed by Statistics Sweden (SCB) and used by national statistical institutes across the Nordics and beyond. It provides a standardized way to search, explore, and retrieve official statistics.

Known v2 installations:
- **Statistics Norway (SSB):** `https://data.ssb.no/api/pxwebapi/v2`
- **Statistics Sweden (SCB):** `https://statistikdatabasen.scb.se/api/v2`

## What this skill does

- Guides Claude through the correct workflow: search → metadata → query → present
- Covers all PxWebApi v2 endpoints (tables, metadata, codelists, saved queries, config)
- Handles codelists, aggregations, and filter expressions
- Works with any PxWebApi v2 installation — just specify the base URL
- Fully in English for international use

## What this skill does NOT include

- Country-specific table lists (table IDs differ per installation)
- Country-specific codelist IDs or regional codes (examples use SSB where needed, clearly marked)
- Country-specific metadata conventions (URN links to classification systems)

For a comprehensive SSB-specific skill with curated table lists, codelist documentation, and Norwegian/English support, see [ssb-pxwebapi-v2](https://github.com/janbrus/ssb-api-v2-examples/tree/main/ssb-pxwebapi-v2-skill).

## Installation

### Claude.ai
1. Download or create a ZIP of this folder
2. Go to **Settings > Features > Skills**
3. Upload the ZIP file

### Claude Code
```bash
cp -r pxwebapi-v2-generic ~/.claude/skills/pxwebapi-v2
```

## MCP servers

For Claude to call the API directly, you need an MCP server:
- **@jarib/pxweb-mcp** (https://www.npmjs.com/package/@jarib/pxweb-mcp) — open source, works with any PxWeb installation
- Or build your own with FastMCP or similar

## Licensing

Data licensing varies by agency:
- SSB (Norway): Creative Commons CC BY 4.0
- SCB (Sweden): CC0 (public domain)

PxWebApi v2 is open source: https://github.com/PxTools/PxWebApi
