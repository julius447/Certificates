# Certificates v2 — redesign

Granska: **https://julius447.github.io/Certificates/redesign/**
Baslinjen (1:1-klonen av dagens block) ligger kvar orörd på `/Certificates/`.

**Allt är låst** (ägarbeslut 2026-08-06): bakgrund B · Strömfält, rubriken
"Behörigheten bakom varje jobb", brödtexten och dess radbrytning, fyra märken i
2×2. A- och C-bakgrunderna samt copy-alternativen finns i git-historiken
(`178bfa0` respektive `359ed7e`).

---

## Vad som ändrats mot dagens block

| | Före | Efter |
|---|---|---|
| Märken | 6 | **4** (Installatörsföretagen, Elsäkerhetsverket, ID06, Trygg-Hansa) |
| Kortstorlek | 113,6 × 90,6 px | **236 × 113 px** — 2,6× ytan |
| Kortproportion | 1,25:1 | **2,09:1** |
| Synligt logotypbläck | 14–40 px högt | **28–70 px**, optiskt utjämnat |
| Rubrik | h3 "Certifikat och partners" | **h2** "Behörigheten bakom varje jobb" |
| Brödtext | 143 tecken, 4 rader på mobil | **78 tecken, 2 rader**, styrd brytning |
| Blockets höjd | 354,58 px | **354 px** — oförändrad |
| Gradient | `90deg #090b32 → #5eb1bf` | **identisk** |
| Vågen | 1193×568 klippt till 1193×319 → kantig | **egen, kan aldrig klippas** |

Borttagna: Skatteverket, Naturvårdsverket, Rexel (den senare hade dessutom en
404-länk).

---

## De fyra besluten bakom designen

### 1. Höjden är låst — så luften växlades mot kortyta
Blockets totalhöjd fick inte ändras, men korten behövde bli större. 46,4 px togs
från sektionens lodräta padding (`--apspace-2xl` 79,2 px → `--apspace-xl` 56 px)
och gavs till korten, som växte 90 → 113 px. Blocket är fortfarande 354 px högt.
På mobil behålls den gamla paddingen — där fanns inget höjdproblem att lösa.

### 2. Vågen kunde aldrig lagas i filen — felet satt i visningsytan
Original-SVG:n är helmjuka Bézier-kurvor, 1193 × 568. Den visades i en
1193 × 319-ruta med `object-fit: cover`, vilket klippte bort ~124 px uppe och
nere — alltså exakt vågkammen. Kvar blev de branta mittsegmenten, som läser som
kantiga band. De nya bakgrunderna är SVG med `preserveAspectRatio="none"` i
100 % × 100 %, så ingenting kan klippas vid någon skärmbredd.

### 3. Optisk normalisering på bläck, i två steg

**Steg 1 — filerna hade inbyggd luft.** Mätt genom att rendera varje fil till
canvas och läsa alfakanalen:

| Logotyp | Tom yta i filen | Synligt bläck vid dåvarande höjd |
|---|---|---|
| Trygg-Hansa | **68,1 % av höjden** | 17,9 px av 56 px box |
| Elsäkerhetsverket | 17,4 % | 57,8 px av 70 px |
| ID06 | 9,6 % | 43,4 px av 48 px |
| Installatörsföretagen | 0 % | 40 px av 40 px |

Det var hela orsaken till att Trygg-Hansa såg liten ut och Installatörsföretagen
stor — ingen ögonmätning hittar det. Alla fyra viewBox:ar är nu beskurna till
bläckrutan, så CSS-höjden motsvarar synlig höjd.

**Steg 2 — lika optisk MASSA, inte lika yta.** Lika yta gav 2,3 % spridning på
papperet men såg fortfarande ojämnt ut: upplevd tyngd beror också på hur tät och
mörk logotypen är. Trygg-Hansa är fet svart versaltext (medelsvärta 0,61),
Elsäkerhetsverket är tunn text plus en öppen ring (0,43).

| Logotyp | Sann proportion | Desktop | Mobil |
|---|---|---|---|
| Installatörsföretagen | 3,824:1 | 35 px → 134 × 35 | 25 px |
| Elsäkerhetsverket | 1,129:1 | 70 px → 79 × 70 | 56 px |
| ID06 | 2,684:1 | 43 px → 115 × 43 | 30 px |
| Trygg-Hansa | 5,147:1 | 28 px → 144 × 28 | 20 px |

Ett värde per logotyp i CSS:en — enkelt att finjustera när de riktiga filerna
kommer.

### 4. Kortproportionen styrs via bredden, inte höjden
Blockhöjden är låst, så korthöjden (113 px) kan inte ändras. Märkesgriden fick
därför ett tak på 492 px: **236 × 113 = 2,09:1**. Överskottet går till
textkolumnen.

---

## Verifiering

Svept över **23 skärmbredder** (1920 → 360) i headless Chromium. Vid varje bredd
kontrolleras blockhöjd, kortstorlek, antal rubrikrader, brödtextens radbrytning
och om någon logotyp klipps:

- Blockhöjden är **exakt 354 px** på 1280–1920 px.
- Den låsta radbrytningen håller i **samtliga 23 bredder** — rad 1 alltid längre
  än rad 2, aldrig brytning mitt i "behörig elektriker".
- **Noll klippta logotyper** i hela spannet.
- Rubriken är en rad överallt utom vid 360 px, där den bryts balanserat i två.

Tre defekter hittades och stängdes under svepet:

| Defekt | Åtgärd |
|---|---|
| Trygg-Hansa klipptes 155 → 126 px i bandet 781–905 px | brytpunkten höjd 780 → 900 px |
| Kort på 344 × 96 px (3,6:1) runt 780 px | tak återinfört i mobilgrenen |
| Den låsta radbrytningen föll vid 375/360 px | sidopadding 18 px under 389 px |

**390 px renderar oförändrat** — ditt godkännande står kvar.

Verktyget som mätte bläckytorna ligger i `tools/ink-measure.html`. Kör det mot
dina nya logotypfiler för att kontrollera att de inte har inbyggd luft.

---

## [GATE] Logotypfiler — spec till dig

Installatörsföretagens logotyp är hämtad från in.se och är en **äkta vektor**.
De tre andra är **inte** SVG:er trots filändelsen — de är 300 px breda PNG:er
inbäddade i ett SVG-skal.

| Logotyp | Visas som | Minsta källbredd (3×) |
|---|---|---|
| Installatörsföretagen | 134 × 35 | äkta vektor idag — behåll |
| Elsäkerhetsverket | 79 × 70 | 240 px |
| ID06 | 115 × 43 | 345 px |
| Trygg-Hansa | 144 × 28 | 435 px |

Krav: transparent bakgrund, **beskuren till bläcket** (ingen inbyggd luft —
CSS:en sköter paddingen, och inbyggd luft förstör den optiska normaliseringen),
SVG i första hand, annars WebP.

**Två filer har konkreta fel som bör åtgärdas i samma svep:**

1. **ID06 bär en främmande ram.** I den inbäddade PNG:n ligger en blek orange
   rundad rektangel runt "iD"-märket, plus en ljus cirkelkontur. Den ingår inte
   i den officiella logotypen och syns tydligt på mobil. Den går inte att
   beskära bort utan att skäras itu, så höjden kompenserar i stället. Med en ren
   fil ska ID06:s `--lh` sänkas 43 → ~41 px (mobil 30 → 29).
2. **Trygg-Hansa skalas upp på telefon.** Källbilden har ~295 px användbar bredd
   och renderas i 144 px — räcker på 2× men inte på 3×, där den blir märkbart
   mjukare än de andra tre.

---

## Öppna punkter före publicering

1. **IN:s logotypregler** `[GAP]` — Installatörsföretagen distribuerar
   medlemslogotyper och användningsvillkor via Mina sidor
   (`in.se/mina-sidor/in-logotyper-och-bildekaler/`). Hämta den officiella
   medlemslogotypen därifrån och stäm av villkoren innan publicering.
2. **Länkarna är kvar oförändrade.** Du sa att vi inte aktivt ska skicka folk
   till andra sajter, så all "kolla själv"-copy är borta — men själva länkarna
   ligger kvar precis som idag. Säg till om du hellre vill ha korten olänkade.
3. **Elsäkerhetsverket väger strukturellt lättast.** Vid 70 px ligger den på
   86 % av kortets innerhöjd — taket. Resterande skillnad går inte att stänga
   utan att ändra korthöjden, som är låst. Den är alltså jämnare än förut, men
   inte matematiskt jämnstor.
4. **Organization-schema** (`memberOf`, `sameAs`) är inte byggt än — den enda av
   SEO-punkterna från analysen som ger faktiskt värde. Tas lämpligen i samma
   svep som Bricks-konverteringen.
