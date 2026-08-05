# Mätprotokoll — beviset för 1:1

Metod: samma Chromium, samma viewport, live (`https://ampy.se/`) i en flik och
klonen (`http://localhost:8796/`) i en annan. För varje element hämtas **hela**
den beräknade stilen (`getComputedStyle`, alla egenskaper) plus `getBoundingClientRect`
relativt sektionens överkant. De två dumparna diffas byte för byte.

20 element mäts: sektionen, containern, båda blocken, rubriken, brödtexten,
grid-behållaren, alla 6 kort, alla 6 logotyper och vågen.

---

## Fullständig diff — varje beräknad egenskap

| Viewport | Egenskaper jämförda | Skillnader | Geometriboxar | Felaktiga boxar |
|----------|--------------------:|-----------:|--------------:|----------------:|
| **1440 × 900** | 9 160 | **0** | 20 | **0** |
| **390 × 844** | 9 160 | **0** | 20 | **0** |

Det är alltså inte ett urval av egenskaper som stämmer — det är samtliga
9 160 beräknade CSS-egenskaper per breakpoint, inklusive färger, skuggor, filter,
transformer, `object-fit`, `grid-template-*`, `font-*` och `-webkit-*`.

## Breddsvep — hash över geometri + 25 nyckelegenskaper

| Bredd | Live | Klon | Sektionshöjd |
|-------|------|------|--------------|
| 1920 | `identisk sträng` | `identisk sträng` | 354,58 px |
| 1440 | full diff, 0 fel | full diff, 0 fel | 354,58 px |
| 1024 | `98578178` | `98578178` | 321,66 px |
| 780 *(breakpoint)* | `6cb66a4c` | `6cb66a4c` | 337,27 px |
| 480 *(breakpoint)* | `c435111a` | `c435111a` | 428,74 px |
| 390 | full diff, 0 fel | full diff, 0 fel | 410,53 px |
| 360 | `a4fbe783` | `a4fbe783` | 428,36 px |

Sju bredder, inklusive båda mediefråge-gränserna (780 px och 480 px). Ingen avvikelse.

---

## Referensvärden @ 1440 × 900

| Element | x, y (från sektionen) | b × h | Noteringar |
|---------|----------------------|-------|------------|
| section | 0, 0 | 1440 × 354,58 | padding 79,2 / 56 px, marginal 36 px |
| container | 80, 79,20 | 1280 × 196,19 | gap 19,8 / 28 px |
| textblock | 80, 100,60 | 626 × 153,38 | |
| rubrik | 80, 100,60 | 321,33 × 41,59 | 32 px / 400 / lh 41,6 |
| brödtext | 80, 162,20 | 469,50 × 91,78 | 18 px / 300 / max-width 75 % |
| grid | 734, 79,20 | 370,78 × 196,19 | 3 × 113,594 px, 2 × 90,594 px, gap 15 px |
| kort (var) | — | 113,59 × 90,59 | padding 19,8 px, radie **20 px**, vit |
| vågen | 247, 35,46 | 1193 × 319,12 | absolut, höger 0 / botten 0, max-height 90 % |

Kort 1 är 88,59 px högt (inte 90,59) — det har `align-self: center` och en logotyp
som är 2 px lägre än radens höjd. Så ser det ut på live också.

## Referensvärden @ 390 × 844

| Element | x, y | b × h | Noteringar |
|---------|------|-------|------------|
| section | 0, 0 | 390 × 410,53 | padding 34,761 / 27,297 px |
| container | 27,30, 34,76 | 335,41 × 341,02 | radbryts till två rader |
| textblock | 27,30, 34,76 | 335,41 × 114,39 | centrerad |
| rubrik | 71,81, 34,76 | 246,38 × 31,90 | 24,537 px |
| brödtext | 27,30, 77,17 | 335,41 × 71,98 | **14,119 px** — ärvd, se PROVENANCE §4.3 |
| grid | 27,30, 183,95 | 332,06 × 170,38 | 3 × 100,688 px, 2 × 77,688 px |
| kort (var) | — | 100,69 × 77,69 | radie 16,338 px |
| vågen | 39, 41,05 | 351 × 369,48 | object-position 200 % 100 % |

---

## Två avvikelser som hittades och stängdes

Båda upptäcktes av diffen, inte av ögat:

1. **Rubriken 36 px istället för 32 px.** `05-theme-style-ampy.css` sätter
   `--aptext-xl: clamp(2rem, calc(1.67vw + 1.47rem), 3.6rem)`. På live skrivs den
   över av `<style id="core-framework-frontend-inline">` längre ned i dokumentet.
   Åtgärd: lager 09 lades till. Samma block styr även `--apradius-l`, så kortens
   hörnradie gick samtidigt från 16 px till korrekta 20 px.

2. **De vita korten försvann helt.** `--white` definieras i
   `<style id="bricks-color-palettes-inline">`, som saknades. Utan den blir
   `.certificates__div.brxe-div{background-color:var(--white)}` ogiltig vid
   beräkning — och den lägre `.brxe-56f1bc{background-color:#fff}` träder **inte**
   in som reserv, eftersom var()-fallerandet sker efter kaskaden. Korten blev
   genomskinliga. Åtgärd: lager 03 lades till.

---

## Vad som inte kunde verifieras visuellt

Skärmdump av **live-sidan** gick inte att ta: Browser-panens pixelfångst returnerar
en tom ljus ruta för ampy.se (renderaren hänger på den tunga sidan — ett
`scroll`-anrop timeoutade också efter 30 s). JavaScript-mätningen mot samma flik
fungerade hela tiden, så siffrorna ovan är hämtade ur den verkliga live-renderingen.

Klonen är skärmdumpad och ser rätt ut på både desktop och mobil. Den visuella
jämförelsen mot live vilar alltså på computed-style- och geometribeviset, inte på
en pixeljämförelse av två skärmdumpar. **Titta gärna själv på båda sidorna innan
du godkänner** — det är den enda kontrollen som inte gick att utföra här.
