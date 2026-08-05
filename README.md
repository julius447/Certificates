# Certificates — 1:1-klon av ampy.se

Exakt reproduktion av tillitsblocket **"Certifikat och partners"** på ampy.se,
hämtat 2026-08-05. Baslinje inför designarbetet på desktop-lådorna.

**Ingenting är omdesignat.** Detta är enbart en klon.

```
öppna index.html  —  eller kör:  python3 -m http.server 8796
```

---

## Är det verkligen 1:1?

Ja, och det är mätt — inte påstått:

| Viewport | Beräknade CSS-egenskaper jämförda | Skillnader |
|----------|----------------------------------:|-----------:|
| 1440 × 900 | 9 160 | **0** |
| 390 × 844 | 9 160 | **0** |

Plus geometrin (x, y, bredd, höjd) för alla 20 element — **0 avvikelser** — och ett
svep över 7 bredder (1920 / 1440 / 1024 / 780 / 480 / 390 / 360), inklusive båda
mediefråge-gränserna. Alla identiska.

Metod och alla siffror: [`docs/MEASUREMENTS.md`](docs/MEASUREMENTS.md).

---

## Innehåll

```
index.html                          klonen (sektionens markup ordagrant från live)
assets/css/01–09                    nio CSS-lager i live-sajtens dokumentordning
assets/img/*.svg                    6 logotyper + vågen (partner-section-overlay)
assets/fonts/Outfit-Variable…woff2  typsnittet som faktiskt renderar på ampy.se
bricks/certificates.json            Bricks-exporten, oförändrad
docs/PROVENANCE.md                  var varje byte kommer ifrån
docs/MEASUREMENTS.md                mätprotokollet
```

Det som gör designen: 90°-gradienten `#090b32 → #5eb1bf`, vågen
(`partner-section-overlay.svg`, absolut placerad nere till höger, höjd 90 %),
rubriken i Outfit 400, de sex vita korten i 3 × 2-grid med 15 px gap och 20 px
hörnradie.

---

## Fyra saker jag hittade i den befintliga koden

Inget av detta är åtgärdat — allt är reproducerat som det är. Detaljer i
[`docs/PROVENANCE.md §4`](docs/PROVENANCE.md).

1. **De tre fade-in-animationerna körs aldrig på live.** Bricks-interaktionerna är
   deklarerade, men animate-CSS:en laddas aldrig — inga `fadeIn`-keyframes finns i
   någon av sajtens 32 stylesheets. Blocket dyker upp statiskt.
2. **Mobilens vertikala gradient är död kod.** Den slås ut av `.certificates.brxe-section`
   på specificitet. Mobilen kör samma 90°-gradient som desktop.
3. **`--text-m` finns inte.** Mobilregeln för brödtextens storlek är ogiltig, så
   texten ärver body-storleken istället.
4. **De statiska Outfit-filerna 404:ar.** Alla nio `Outfit-100…900.woff2` saknas på
   servern; det är variabelfonten som räddar renderingen.

---

## Kvar att kontrollera (ägargrind)

Skärmdump av **live-sidan** gick inte att ta i den här miljön — Browser-panens
pixelfångst returnerar en tom ruta för ampy.se (renderaren hänger på den tunga
sidan). JS-mätningen mot samma flik fungerade, så siffrorna kommer från den
verkliga live-renderingen, men den sista visuella jämförelsen sida-vid-sida är inte
gjord. **Öppna klonen bredvid ampy.se och ge klartecken innan vi går vidare.**
