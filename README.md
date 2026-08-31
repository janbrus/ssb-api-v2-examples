# PxWebApi 2 eksempler



**Skills for SSB PxWebApi v2** (AI mot Statistikkbanken)

- [ssb-pxwebapi-v2-skill/](ssb-pxwebapi-v2-skill/) — En Skill som lærer Claude og andre AI-verktøy å bruke PxWebApi v2 mot SSB. Jeg har prøvd å tappe min kunnskap om dette til et AI-lesbart format.
- Kan f.eks. brukes med MCP-server. Fungerer for flere enn Claude. Denne er bedre, og mye mer utfyllende enn Try sin. Se [ssb-pxwebapi-v2-skill/README.md](ssb-pxwebapi-v2-skill/README.md) for installasjon.
- [ssb-chart-skill/](ssb-chart-skill/) — En skill for figurer og datavisualisering. Er ment å brukes sammen med ssb-pxwebapi-v2 skill. Se [ssb-chart-skill/README.md](ssb-chart-skill/README.md) for installasjon.

**Generic skill for PxWebApi v2**

[pxwebapi-v2-generic-skill/](pxwebapi-v2-generic-skill/) — BETA: A Claude (AI) Skill to use PxWebApi v2 towards PxWebApi v2 installations using JSON-stat2. Can be used in Claude and Claude Code, e.g. with MCP. Probably works for more than Claude. See [pxwebapi-v2-generic-skill/README.md](pxwebapi-v2-generic-skill/README.md) for installation.

**Generic skill for PxWebApi v1**
[pxwebapi-v1-generic-skill/](pxwebapi-v1-generic-skill/) — BETA: A Claude (AI) Skill to use PxWebApi towards PxWebApi version 1 installations in differnt countries.


 [**Hva er nytt i PxWebApi versjon 2**](nytt_i_v2.md) 


**Jupyter notebooks**

Til en viss grad viser rekkefølgen økende kompleksitet

[eks1_doi_csv_nor](eks1_doi_csv_nor.ipynb) viser hvordan hente en enkel tabell, detaljomsetningsindeksen, med de nye parametrene i http GET

[kt-v2-csv-nor](kt-v2-csv-nor.ipynb)Hent Konjunkturtendensene som CSV. Lag figurer og en stor tabell med prognoser markert i blått.

[laks_v2_nor](laks_nor.ipynb) viser hvordan henter datasett som JSON-stat2 med både http GET og POST.

[text-code](text-code-api2-nor.ipynb) - Få Kode og Tekst i JSON-stat2 og Pandas - eksempel med HS-varekoder i månedlig Utenrikshandel

[komm-nr-id](komm-nr-id-nor.ipynb) - Hvordan vise **både** kommunenummer/-kode og kommunenavn i en dataframe, dvs. vise kode og tekst i JSON-stat2

[get_many_default_tables](get_many_default_tables.ipynb) Fra API-søk til tabell, hent forhåndsvalgt uttrekk for mange tabeller.

**Javascript eksempler**

[kpi_js_v2](kpi_js_v2.html) - Enkel KPI-figur med Highcharts, nytt basisår 2025


-----

[Lag dynamisk URL](https://nesa.no/ssb/forenkle_url.html) Endre fra statisk til dynamisk tid i API v2 URL.

-----

## In English

**Generic skill for PxWebApi v2**

[pxwebapi-v2-generic-skill/](pxwebapi-v2-generic-skill/) — BETA: A Claude (AI) Skill to use PxWebApi v2 towards PxWebApi v2 installations using JSON-stat v2. Can be used in Claude and Claude Code, e.g. with MCP. Probably works for more than Claude. See [pxwebapi-v2-generic-skill/README.md](pxwebapi-v2-generic-skill/README.md) for installation.


[**What's new in PxWebApi version 2**](new_in_v2.md)

[eks1_doi_csv_eng](eks1_doi_csv_eng.ipynb) how to get a simple dataset, Index of Retail Sales, using the new parameters in http GET

[laks_v2_eng](laks_eng.ipynb) shows how to get a datasett as JSON-stat2, using both http GET and POST.

[text-code-eng](text-code-api2-eng.ipynb) - Get both code and text in JSON-stat2 and Pandas - example with HS codes for goods from monthly foreign trade statistics


