# Strukturkart — ssb.no/a/histstat/

Komplett toppnivå-katalog over SSBs digitaliserte historiske publikasjoner.
Alle URLer er relative til `https://www.ssb.no` med mindre annet er oppgitt.

## Hovedinnganger

- **`/a/histstat/publikasjoner/` — HUBEN.** SSBs autoritative, `curl`-bare publikasjonsindeks (~190 lenker) organisert *Etter emne* (1828–1976 + 1977–1996) og *Etter serie* (serier + periodika). Dette er kilden resten av denne fila annoterer — start her. Emnesidene og `ereg77-96.html` ligger som *relative* lenker under denne katalogen, dvs. `/a/histstat/publikasjoner/histemne-NN.html` (se under).
- `/a/histstat/` — landingsside med navigasjon (peker videre til huben)

## Historisk statistikk (oversiktspublikasjon)

Anbefalt rekkefølge: **1978 → 1968 → 1994** (1978 og 1968 er de beste; 1994 supplerer med nyere tall).

| Utgave | URL |
|---|---|
| 1994 | `/a/histstat/hs1994.html` |
| 1978 | `/a/histstat/hs1978/` |
| 1968 | `/a/histstat/hs1968.pdf` |
| 1958 | `/a/histstat/hs1958.pdf` |
| 1948 | `/a/histstat/hs1948.pdf` |
| 1926 | `/a/histstat/hs1926.pdf` |
| 1914 | `/a/histstat/hs1914.pdf` |
| 1875 | `/a/histstat/aarbok/annuaire_1875.pdf` |

## Bro til Statistikkbanken

`/a/histstat/statbank-histu.html` — **Tidsserier med startår før 1980 i Statistikkbanken**.
Lister tabeller i Statistikkbanken som strekker seg tilbake før 1980, organisert etter emne (Befolkning, Arbeid, Bank og finans, Bygg/bolig/eiendom, Energi/industri, Helse, Inntekt, m.fl.). Bruk denne for å finne live-tall i Statistikkbanken som kan hentes med `ssb-pxwebapi-v2` i stedet for / i tillegg til PDF.

## Andre oversiktspublikasjoner

- **Statistisk årbok** (1880–2013): `/a/histstat/aarbok/`
- **Folketellinger** (1769–2001): `/a/folketellinger/main.html` (ekte per-telling-indeks `fob1769…fob1990.html`). **NB:** landings-URL `/a/folketellinger/` er bare et frameset-skall (~1,3 KB); per-telling-filene har ikke-opplagte år (`fob1866.html`, ikke `fob1865`), så pek alltid på `main.html`.
- **Landbrukstellinger** (1907–1999): `/a/histstat/landbrukstellinger.html`
- **Tabeller, figurer, artikler etter emne:** `/a/histstat/tabart.html`
- **Analyser/studier:** `/a/histstat/analyser.html`

## Bibliografier (PDF — autoritative)

- 1828–1976: `/a/histstat/fortegnelse.pdf` (~3,2 MB)
- 1977–1996: `/a/publikasjoner/publikasjonsoversikt_1977-1996.pdf` (kanonisk sti; `/publikasjoner/…` 301-redirecter hit)

## Emnebibliografi 1828–1976 (25 emner + oversikt)

URL-mønster: `/a/histstat/publikasjoner/histemne-XX.html` (under huben — **ikke** `/a/histstat/histemne-XX.html`)

| Kode | Emne (hubens etikett) |
|---|---|
| `histemne.html` | Oversikt- og samlepublikasjoner |
| 01 | Miljø. Geografiske forhold |
| 02 | Befolkning. Helseforhold |
| 03 | Arbeidsmarked |
| 04 | Nasjonalregnskap |
| 05 | Jordbruk. Skogbruk. Jakt |
| 06 | Fiske |
| 07 | Bergverksdrift. Industri. Kraftforsyning. Bygg- og anlegg |
| 08 | Utenrikshandel |
| 09 | Innenrikshandel |
| 10 | Sjøtransport |
| 11 | Samferdsel |
| 12 | Offentlige finanser |
| 13 | Penger og kreditt |
| 14 | Priser |
| 15 | Lønninger |
| 16 | Inntekt og formue |
| 17 | Forbruk |
| 18 | Bolig og boforhold |
| 19 | Sosiale forhold |
| 20 | Rettsforhold |
| 21 | Utdanning |
| 22 | Kultur |
| 23 | Ferie og friluftsliv |
| 24 | Valg |
| 25 | Økonomisk og statistisk teori og analyse |

> **NB:** Bruk hub-stien `/a/histstat/publikasjoner/histemne-NN.html` — den er ekte HTML med PDF-lenker, så `curl … | grep` fungerer. Rot-stien `/a/histstat/histemne-NN.html` er en tom skall-side (0 PDF-lenker), og WebFetch hjelper ikke. Supplér med `fortegnelse.pdf` og serieindeksene under.

## Emnebibliografi 1977–1996 (12 hovedgrupper + regional)

URL-mønster: `/a/histstat/publikasjoner/ereg77-96.html#KODE` (under huben). Hele `ereg77-96.html` er ~449 KB med ~2300 PDF-lenker; `curl … | grep` direkte på siden fungerer.

| Anker | Emne (hubens etikett) |
|---|---|
| #00 | Generelt |
| #00.01 | Generelt — valg |
| #00.02 | Generelt — levekår |
| #01 | Natur og miljø |
| #02 | Befolkning |
| #03 | Helse, sosiale forhold og kriminalitet |
| #04 | Utdanning |
| #05 | Inntekt og forbruk |
| #06 | Arbeid og lønn |
| #07 | Kultur og fritid |
| #08 | Priser, prisindekser og konjunkturindikatorer |
| #09 | Nasjonalregnskap og utenrikshandel |
| #11 | Finansmarkeder og konkurser |
| #12 | Offentlige finanser |
| #R10 | Næringsvirksomhet |

> **NB:** Ankertekstene over er hentet fra huben (`/a/histstat/publikasjoner/`), som er kilden — tidligere versjon hadde flere koder feilmappet (#06/#08/#09/#11/#12/#R10). Det finnes ingen #10; sekvensen hopper fra #09 til #11, med #R10 som egen regional gruppe.

## Serier (12 + diverse)

URL-mønster: `/a/histstat/<kode>/`. Alle serieindekser er rike, `curl | grep`-bare HTML (i motsetning til rot-emnesidene).

| Kode | Navn | Periode |
|---|---|---|
| `nos` | **NOS — Norges offisielle statistikk** | 1828–2010 |
| `ano` | Arbeidsnotater | 1963–1979 |
| `art` | Artikler | 1957–1986 |
| `dp` | Discussion Papers (forskning) | 1985– |
| `doc` | Documents | 1994–2009 |
| `in` | Interne notater | 1979–1992 |
| `not` | Notater | 1993– |
| `rapp` | Rapporter | 1979– |
| `sos` | **SØS — Samfunnsøkonomiske studier / Sosiale og økonomiske studier** (SSBs forskningsmonografi-serie) | 1954–1996 |
| `ssh` | SSBs håndbøker | 1958–2009 |
| `sagml` | Statistiske analyser (gammel serie) | 1972–1986 |
| `aarsmeld` | Årsmeldinger | 1964– |
| `diverse` | Diverse annet digitalisert materiale og små serier — egen fil under huben: `/a/histstat/publikasjoner/diverse.html` | – |

**Forskningsserier:** `sos` er hovedserien for SSBs forskningsmonografier (1954–1996), publisert under to titler over tid — "Samfunnsøkonomiske studier" og "Sosiale og økonomiske studier" — med felles forkortelse SØS. Senere forskningsutgivelser fordeles på `dp` (Discussion Papers) og `rapp` (Rapporter). Til sammenlikning er `nos` myndighetsrapportering, ikke forskning.

**NOS er klart viktigst.** Indeksen `/a/histstat/nos/` er ~1,1 MB inline HTML med direktelenker til alle NOS-PDF-er — bruk `curl ... | grep` for raske oppslag.

## Periodika (9 stk)

| Kode | Navn | Periode |
|---|---|---|
| `bk` | Bank- og kredittstatistikk | 1977–2004 |
| `es` | Economic survey | 1991–2003 |
| `mu` | Månedsstatistikk over utenrikshandelen | 1913–1999 |
| `nd` | Nye distriktstall | 1974–1989 |
| `rs` | Regionalstatistikk | 1989–2000 |
| `ssp` | Samfunnsspeilet | 1987– |
| `sm` | Statistisk månedshefte / Meddelelser | 1882–1997 |
| `us` | Ukens statistikk / Statistisk ukehefte | 1960–2000 |
| `oa` | Økonomiske analyser | 1982– |

## Filnavn-konvensjoner

| Mønster | Eksempel | Katalog | Beskrivelse |
|---|---|---|---|
| `nos_<serie>_<kode>_<år>.pdf` | `nos_i_a1_1867.pdf` | `/a/histstat/nos/` | NOS, fra Serie I (1860–1881) og senere |
| `hs<år>.pdf` / `hs<år>.html` | `hs1968.pdf` | `/a/histstat/` | Historisk statistikk-utgaver |
| `st_<nr>r_<år>.pdf` | `st_04r_1801-35.pdf` | `/a/histstat/nos/` | Statistiske tabeller (1828–1860, før NOS). **Merk:** ligger sammen med NOS-filene, ikke på `/a/histstat/`-rot |
| `annuaire_<år>.pdf` | `annuaire_1875.pdf` | `/a/histstat/aarbok/` | Tidlig Statistisk årbok (på fransk) |

NOS-serier benyttet i filnavn: `i` (1860–1881), `ii` (1882–1885), `iii` (1885–1900), `iv` (1901–1905), `v`…`xii`, `a`…`d` (frem til 2010).
