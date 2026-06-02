# Diagramvalg — Beslutningsmatrise

Detaljerte regler for valg og utforming av diagramtyper for SSB-data.

---

## Linjediagram

**Når:** Trend over tid, 3+ tidsperioder.

**Akser:**
- Tid på x-aksen, verdi på y-aksen
- Y-aksen starter alltid på 0 for absolutte tall
- For indekser (KPI, boligpris) kan y-aksen starte på en annen verdi, men marker basisverdi tydelig

**Serier:**
- Maks 5 linjer per diagram (strengere enn den generelle maks-6–7-regelen i SKILL.md, fordi overlappende linjer blir uleselige raskere enn separate søyler). Bruk small multiples hvis du trenger å vise flere.
- For interaktive visninger der brukeren selv velger serier: vis advarsel ved overskridelse, men render likevel.
- Bruk utheving-paletten for fokusserie (SSB Grønn for nøkkelserie, grå for resten)
- Linjestil: 2px solid
- Markører bare ved <10 datapunkter

**SSB-spesifikt:**
- Vis basisår for indekser med annotasjon (f.eks. "2015=100")
- Marker prognoser/framskrivninger med stiplet linje
- Marker brudd i tidsserier (f.eks. kommunesammenslåinger) med vertikal stiplet linje og annotasjon

**Vanlige feil:**
- For mange linjer — bruk small multiples i stedet
- Manglende markering av basisår for indekser
- Avkortet y-akse som overdriver trender

---

## Horisontale søyler

**Når:** Sammenligning av kategorier, rangering.

**Regler:**
- Sorter alltid etter verdi (høyest → lavest eller lavest → høyest)
- Y-aksen starter alltid på 0
- Avstand: 40% gap mellom søyler
- Direkte merking med verdier på enden av hver søyle
- Maks 15–20 søyler per diagram

**SSB-spesifikt:**
- For kommunesammenligninger: vis topp/bunn 10 i stedet for alle 400
- Marker landsgjennomsnitt med vertikal referanselinje i `#909090`
- Bruk SSB Grønn for enkelt-serie, kategorisk palett for grupperte søyler

**Vanlige feil:**
- Usorterte søyler — gjør det vanskelig å sammenligne
- For mange søyler — grupper eller filtrer
- Vertikale søyler med lange kategori-labels — bruk horisontale i stedet

---

## Grupperte / stablede søyler

**Når:** Sammenligne kategorier med underkategorier.

**Grupperte:**
- Maks 4 grupper per kategori
- Bruk kategorisk palett i rekkefølge
- Plasser legend over diagrammet

**Stablede:**
- Maks 5 segmenter per stabel
- Plasser det viktigste segmentet nederst
- Vis totalen på toppen av stabelen
- Bruk 100% stablet for del-av-helhet over tid

---

## Ringdiagram (donut)

**Når:** Del av helhet, maks 5 segmenter.

**Regler:**
- Hull: 50% av radius
- Sortering: størst segment starter kl. 12, med klokken
- Samle alt under 5% i "Andre" (SSB Grå)
- Direkte merking med prosent OG absolutt tall

**SSB-spesifikt:**
- Vis total i midten av ringen
- Maks 5 segmenter — grupper resten

**Vanlige feil:**
- For mange segmenter (>6) — vanskelig å lese
- Manglende "Andre"-kategori for små segmenter
- Bruker kakediagram i stedet for ringdiagram

---

## Kart (choropleth)

**Når:** Geografisk variasjon på kommune- eller fylkesnivå.

**Forutsetning:** `data.role.geo` finnes i responsen. **Hvis `role.geo` mangler**, ikke prøv å rendre kart — dataene er typisk allerede aggregerte til hele Norge. Vis dem som linje (tidsserie), horisontal søyle (rangering) eller scorecard (KPI) i stedet. Ikke spør brukeren om geografi når API-et ikke har levert det.

**Regler:**
- Bruk sekvensielt grønn fra `#ECFEED` til `#274247`
- 5 kvantilgrupper er standard
- Tooltip med navn, verdi og rang
- Inkluder legend med verdiskala

**SSB-spesifikt:**
- Bruk gjeldende kommunegrenser (2024-standard)
- Vis landsgjennomsnitt i legend som referanse
- For kommunekart: vurder om fylkesnivå gir bedre oversikt
- Svalbard og Jan Mayen: inkluder kun hvis relevant data finnes

**Vanlige feil:**
- Feil kommunegrenser (bruk alltid gjeldende grenser)
- For mange fargetrinn — 5 er nok for de fleste kart
- Manglende referanseverdi (landsgjennomsnitt)

---

## Scorecard

**Når:** Enkelttall, KPI, nøkkelmetrikk.

**Regler:**
- Stort tall: 32px+ bold
- Undertekst med kontekst: periode, enhet
- Vis endring med ▲/▼ og semantisk farge (grønn/rosa)
- Bruk kortformat for store tall: 5.6 mill., 720k

**SSB-spesifikt:**
- Vis alltid sammenligningsperiode: "5 609 000 innbyggere (+1.2% fra 2024)"
- For indekser: vis både indeksverdi og endring

---

## Spredningsdiagram (scatter)

**Når:** Korrelasjon mellom to variabler.

**Regler:**
- Aldri bruk dobbel y-akse — bruk scatter i stedet
- Merk akser tydelig med enheter
- Bruk farge for tredje dimensjon (maks 5 kategorier)
- Inkluder trend-/regresjonslinje kun hvis statistisk meningsfull

---

## Histogram

**Når:** Fordeling av verdier.

**Regler:**
- Ingen gap mellom søyler (kontinuerlig fordeling)
- Merk x-aksen med intervallgrenser
- Inkluder gjennomsnitt/median som vertikal referanselinje

---

## Small multiples

**Når:** Samme mål for mange kategorier (f.eks. befolkningsutvikling per fylke).

**Regler:**
- Alle deldiagrammer har samme skala
- Bruk en enkel diagramtype (linje eller søyle)
- Maks 3×4 grid (12 deldiagrammer)
- Konsistent farge (SSB Grønn for alle)
- Tittel per deldiagram med kategorinavn
