# Outputformater

Standard er `json-stat2`. Velg annet format med `outputFormat`-parameter på `/tables/{id}/data` (og `/savedqueries/{id}/data`).

| Format     | `outputFormat`-verdi | Bruk                                 |
| ---------- | -------------------- | ------------------------------------ |
| json-stat2 | `json-stat2`         | Standard, maskinlesbar, rik metadata |
| CSV        | `csv`                | Enkelt tabulært format               |
| Excel      | `xlsx`               | For sluttbrukere                     |
| HTML       | `html`               | Tabell for visning                   |
| PX         | `px`                 | Tradisjonelt PX-format               |
| JSON-PX    | `json-px`            | JSON-variant av PX                   |
| Parquet    | `parquet`            | Kolonneformat for dataanalyse (pandas, DuckDB) |

`parquet` returneres som `application/octet-stream`. Sjekk `dataFormats` i `GET /config` for gjeldende formatliste.

## OutputFormatParams (kan kombineres)

- `UseCodes` — Bruk koder i output
- `UseTexts` — Bruk tekster
- `UseCodesAndTexts` — Begge deler
- `IncludeTitle` — Inkluder tabelltittel
- `SeparatorTab` / `SeparatorSpace` / `SeparatorSemicolon` — Separator for CSV

Eksempel:

```
POST /tables/07221/data?outputFormat=xlsx&outputFormatParams=UseCodesAndTexts&outputFormatParams=IncludeTitle
```

## heading og stub

Du kan styre pivoteringen med `heading` og `stub`-parametre (liste av variabelnavn). `heading` legger variabler i kolonnene, `stub` i radene. Brukes typisk for csv/html/xlsx der layout har visuell betydning.
