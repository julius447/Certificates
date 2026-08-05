# Provenans — var varje byte kommer ifrån

Hämtat från **https://ampy.se/** den **2026-08-05**. Ingenting i den här repon är
skrivet på fri hand: varje regel, varje bild och varje siffra är kopierad från live.

---

## 1. Markup

`index.html` innehåller `<section class="brxe-gcsnlt brxe-section certificates">`
kopierad **tecken för tecken** ur den renderade server-HTML:en på ampy.se
(byte-offset 186 655–190 770 i hämtningen).

**Enda ändringen:** de 7 bildernas `src` pekar på lokala kopior i `assets/img/`
istället för `https://ampy.se/wp-content/uploads/`.

Allt annat är orört, inklusive:

- klassnamnen (`brxe-<id>` + BEM-klasserna)
- `data-interactions` / `data-interaction-id`
- `decoding`, `loading="lazy"`, `fetchpriority`, `width`/`height`
- `alt="Ampy"` på alla sju bilder (så står det på live)
- `&nbsp;` sist i brödtexten
- utropstecknet i brödtexten

---

## 2. CSS — nio lager, i exakt dokumentordning

Numreringen följer varje lagers **byte-position i live-HTML:en**, så kaskaden blir
identisk med ampy.se.

| # | Fil | Källa på live | Byte |
|---|-----|---------------|------|
| 01 | `01-fonts.css` | inline `<style>` (variabel-fonten Outfit) | 4723 |
| 02 | `02-bricks-frontend.css` | `7e2af5844dce.frontend-light-layer.min.css` | 25169 |
| 03 | `03-bricks-color-palettes.css` | inline `<style id="bricks-color-palettes-inline">` | 25832 |
| 04 | `04-global-variables.css` | `775fd0e5c09e.global-variables.min.css` | 26863 |
| 05 | `05-theme-style-ampy.css` | `3c553d32ad28.theme-style-ampy.min.css` | 27025 |
| 06 | `06-bricks-element-styles.css` | `e5b38b4eea25.post-15589.min.css` (endast blockets regler) | 27337 |
| 07 | `07-post-15096-section-rule.css` | `d20f91c9b2b6.post-15096.min.css` (endast `.brxe-section`) | 27487 |
| 08 | `08-certificates-global-classes.css` | inline `<style>` (`.certificates*`-klasserna) | 27637 |
| 09 | `09-core-framework-inline.css` | inline `<style id="core-framework-frontend-inline">` | 40054 |

Filerna 02, 04 och 05 är **oförändrade kopior** av live-filerna. Filerna 06 och 07
är utdrag: bara de regler som faktiskt träffar det här blocket. Att inget saknas i
utdragen är inte en bedömning utan ett mätresultat — se `MEASUREMENTS.md`.

**Utelämnat medvetet:** `59-header-styling.css`, `60-site-css.css` och
`post-15042.min.css` innehåller noll regler som träffar blockets element
(verifierat med full computed-style-diff). `<style id="bricks-style-manager-inline">`
(byte 28301) upprepar färgpaletten med exakt identiska värden — ingen effekt.

---

## 3. Bilder

Sju SVG:er, nedladdade obehandlade från `https://ampy.se/wp-content/uploads/`:

| Fil | Byte | Roll |
|-----|------|------|
| `wlsa.svg` | 58 520 | Elsäkerhetsverket |
| `skatt.svg` | 93 598 | Skatteverket |
| `natu.svg` | 104 524 | Naturvårdsverket |
| `1d.svg` | 38 871 | ID06 |
| `trygg.svg` | 26 731 | Trygg-Hansa |
| `960px-Rexel.svg` | 31 125 | Rexel |
| `partner-section-overlay.svg` | 1 065 | **vågen** (1193×568) |

Länkmålen är oförändrade — inklusive att Rexel-kortet som enda kort saknar
`target="_blank"` och går över `http://` (inte https), precis som på live.

---

## 4. Tre fynd i den befintliga koden

Detta är iakttagelser, **inga ändringar**. Allt nedan är reproducerat som det är.

### 4.1 De deklarerade animationerna körs inte på live

Blocket deklarerar tre Bricks-interaktioner (container `fadeIn` 0.3s, båda blocken
`fadeIn` 1.3s, vågen `fadeInUp`). De **händer aldrig** på ampy.se:

- inga `fadeIn`/`fadeInUp`-keyframes finns i något av sajtens 32 stylesheets
- ingen animate-CSS laddas (0 träffar i `performance.getEntriesByType('resource')`)
- med sektionen 4732 px under vikten är elementen redan `opacity: 1`, utan
  `style`-attribut och utan `data-interaction-hidden-on-load`

Klonen renderar därför statiskt synligt — vilket är exakt vad live gör.

### 4.2 Två döda mobilregler

Båda ligger kvar i klonen eftersom de ligger kvar på live:

- `@media (max-width:480px){.brxe-gcsnlt{background-image:linear-gradient(#090b32,#5eb1bf)}}`
  — den vertikala mobil-gradienten. Slås ut av `.certificates.brxe-section`
  (specificitet 0-2-0 mot 0-1-0). **Mobilen får alltså samma 90°-gradient som desktop.**
- `@media (max-width:480px){.brxe-b1bd66{object-fit:scale-down}}` — slås ut av
  `.certificates__bg-image.brxe-image:not(.tag)` (0-3-0 mot 0-1-0). Vågen är
  `cover` även på mobil. Däremot får `object-position: 200% 100%` genomslag.

### 4.3 `--text-m` finns inte

`@media (max-width:480px)` sätter `font-size: var(--text-m)` på brödtexten, men
`--text-m` är odefinierad på ampy.se. Deklarationen blir ogiltig vid beräkning och
texten ärver body-storleken (`--aptext-sm`, 14,119 px @390). Klonen definierar
**avsiktligt inte** `--text-m`, så beteendet blir identiskt.

### 4.4 Typsnittet: de statiska Outfit-filerna är 404

`05-theme-style-ampy.css` deklarerar nio statiska `@font-face` mot
`…/core-framework/fonts/Outfit-100…900.woff2`. **Alla nio svarar 404 på live**
(`document.fonts` rapporterar dem som `unloaded`). Det som faktiskt renderar är
variabelfonten i lager 01, som deklareras senare och vinner kaskaden.
Klonen self-hostar variabelfonten; de trasiga deklarationerna är kvar i lager 05
eftersom filen är en ordagrann kopia — och de 404:ar likadant på båda hållen.

---

## 5. Verktyg

- HTML/CSS/SVG/font: `curl` mot ampy.se
- Mätning: `getComputedStyle` + `getBoundingClientRect` i Chromium via Browser-panen,
  körd mot live och klon i **samma** webbläsare och **samma** viewport
