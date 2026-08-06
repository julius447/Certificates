# Certificates v2 — redesign

Granska: **https://julius447.github.io/Certificates/redesign/**
Baslinjen (1:1-klonen av dagens block) ligger kvar orörd på `/Certificates/`.

Tre bakgrundsförslag växlas med knapparna överst. **Griden, korten, logotyperna
och copyn är identiska i alla tre** — bara bakgrunden skiljer.

---

## Vad som ändrats

| | Före | Efter |
|---|---|---|
| Märken | 6 | **4** (IN, Elsäkerhetsverket, ID06, Trygg-Hansa) |
| Kortstorlek | 113,6 × 90,6 px | **305 × 113 px** — 3,35× ytan |
| Logotypbredd | 44–67 px | **70–161 px** |
| Rubrik | h3 "Certifikat och partners" | **h2** "Därför kan du lita på oss" |
| Brödtext | 143 tecken, 4 rader på mobil | **61 tecken, 2 balanserade rader** |
| Blockets höjd | 354,58 px | **354 px** — oförändrad |
| Gradient | `90deg #090b32 → #5eb1bf` | **identisk** |
| Vågen | 1193×568 klippt till 1193×319 → kantig | **egen, kan aldrig klippas** |

Borttagna: Skatteverket, Naturvårdsverket, Rexel (den senare hade dessutom en
404-länk).

---

## De tre besluten bakom designen

**1. Höjden är låst — så luften växlades mot kortyta.**
Blockets totalhöjd fick inte ändras. Korten behövde ändå bli större. Lösningen
var att ta 46,4 px från sektionens lodräta padding (`--apspace-2xl` 79,2 px →
`--apspace-xl` 56 px) och ge dem till korten, som växte 90 → 113 px. Blocket är
fortfarande 354 px högt. På mobil behålls den gamla paddingen — där fanns inget
höjdproblem att lösa.

**2. Vågen kunde aldrig lagas i filen — felet satt i visningsytan.**
Original-SVG:n är helmjuka Bézier-kurvor, 1193×568. Den visades i en 1193×319-ruta
med `object-fit: cover`, vilket klippte bort ~124 px uppe och nere — alltså exakt
vågkammen. Kvar blev de branta mittsegmenten, som läser som kantiga band. De nya
bakgrunderna är SVG med `preserveAspectRatio="none"` i 100 % × 100 %, så
ingenting kan klippas vid någon skärmbredd.

**3. Optisk normalisering, inte höjdnormalisering.**
De fyra lockuperna har vitt skilda proportioner:

| Logotyp | Proportion | Höjd | Renderas |
|---|---|---|---|
| Installatörsföretagen | 3,83:1 | 42 px | 161 × 42 |
| Elsäkerhetsverket | 1,00:1 | 72 px | 72 × 72 |
| ID06 | 2,52:1 | 50 px | 126 × 50 |
| Trygg-Hansa | 1,68:1 | 58 px | 97 × 58 |

Samma *höjd* åt alla hade gjort den kvadratiska Elsäkerhetsverket-loggan dubbelt
så tung som IN. Samma *bredd* hade gjort tvärtom. Höjden sätts därför per logotyp,
avvägd mot lockupens bredd så att den optiska massan blir jämn — spridningen
mellan största och minsta logotypyta är **26 %** (mot 51 % vid ren
höjdnormalisering). Ett värde per logga i CSS:en, enkelt att finjustera.

---

## [GATE] Logotypfiler — spec till dig

Installatörsföretagens logotyp är hämtad från in.se och är en **äkta vektor**.
De tre andra är **inte** SVG:er trots filändelsen — de är 300 px breda PNG:er
inbäddade i ett SVG-skal. De håller på dagens storlekar och på 2× retina, men
inte på 3×.

När du tar fram de riktiga filerna:

| Logotyp | Visas som | Minsta bredd (3×) |
|---|---|---|
| Installatörsföretagen | 161 × 42 | 483 px — eller behåll SVG |
| Elsäkerhetsverket | 72 × 72 | 216 px |
| ID06 | 126 × 50 | 378 px |
| Trygg-Hansa | 97 × 58 | 291 px |

Krav: transparent bakgrund, **beskuren till bläcket** (ingen inbyggd luft — CSS:en
sköter paddingen, inbyggd luft förstör den optiska normaliseringen), SVG i första
hand, annars WebP.

---

## Öppna punkter före publicering

1. **IN:s logotypregler** `[GAP]` — Installatörsföretagen distribuerar
   medlemslogotyper och användningsvillkor via Mina sidor
   (`in.se/mina-sidor/in-logotyper-och-bildekaler/`). Hämta den officiella
   medlemslogotypen därifrån och stäm av villkoren innan publicering.
2. **Länkarna är kvar oförändrade.** Du sa att vi inte aktivt ska skicka folk
   till andra sajter, så all "kolla själv"-copy är borta — men själva länkarna
   ligger kvar precis som idag. Säg till om du hellre vill ha korten olänkade.
3. **Copyn** — "försäkrat arbete" är medvetet försiktigt formulerat.
   `[GAP]` exakt försäkringsform hos Trygg-Hansa. Får jag den kan raden bli
   skarpare, t.ex. "ansvarsförsäkrade hos Trygg-Hansa".
4. **Val av bakgrund** — A, B eller C.
5. **Organization-schema** (`memberOf`, `sameAs`) är inte byggt än — det är den
   enda av SEO-punkterna från analysen som ger faktiskt värde, och den tas
   lämpligen i samma svep som Bricks-konverteringen.
