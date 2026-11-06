# TKK Collage Maker

> **Sinds november 2026 wordt `index.html` gebouwd vanuit de repo `kindkidnils/tkk-merk`.** Pas de tool
> daar aan (`toepassingen/tools/collage-maker/bron.html`) en bouw opnieuw met
> `python3 toepassingen/tools/bouw_tool.py collage-maker <deze map>`. Kribbels, statements, kleuren en de
> vormgeving komen uit het merksysteem. De mappen `kribbels/`, `krabbels/` en `manifest.json` hier worden
> niet meer gebruikt en blijven alleen staan voor oude links.

## Doel
Interne tool voor The Kind Kids om snel een vierkante, zwart-witte fotocollage te maken met een TKK-"kribbel" of "krabbel" (handgetekende illustratie/tekst) erbovenop in een merkkleur. Vervangt het handmatig samenstellen van zo'n visual in Figma/Photoshop voor eenvoudige social- of contentposts.

## Pipeline in het kort
1. **Foto** — gebruiker upload een foto (client-side via `FileReader`, geen upload naar een server). De foto wordt automatisch grijswaarden (`filter: grayscale(100%)`), geschaald zodat ze de vierkante canvas vult, en is daarna vrij te verslepen/schalen.
2. **Illustratie** — een SVG ("kribbel" = tekening, "krabbel" = handgeschreven tekst) wordt gekozen uit de vooraf gedefinieerde sets in `manifest.json` (ingelezen bij page load) of zelf geüpload. Geüploade SVG's gaan door `sanitizeSvg()` (verwijdert `<script>`, `<foreignObject>`, `on*`-attributen en `javascript:`-URI's) en `normalizeSvg()` (herschrijft harde `fill:#hex` in `<style>`-blokken naar `fill: currentColor` zodat de kleurenkiezer werkt).
3. **Kleur** — een vaste TKK-kleur wordt toegepast via CSS `color` op de SVG-overlay (die `currentColor` gebruikt).
4. **Aanpassen** — schaal/rotatie van de illustratie, positie van foto en illustratie via drag (pointer events op `#photo-layer` / `#svg-overlay`).
5. **Formaat + export** — gebruiker kiest digitaal (1080px, 72dpi) of print (3240px, 300dpi); de exportknop in de topnav tekent foto + illustratie op een `<canvas>` op exportresolutie (schaalfactor = exportpx / previewpx) en downloadt als PNG via `canvas.toDataURL`.

## Bewust niet (v1)
- Geen backend/opslag — alles blijft client-side, geen accounts, geen historiek van collages.
- Geen ondersteuning voor meerdere foto's of illustraties tegelijk in één collage — één foto, één illustratie per export.
- Geen kleurenkiezer buiten de vaste TKK-palette (bewust beperkt tot merkkleuren).
- Geen crop-tool voor de foto — schalen/verslepen binnen de vierkante canvas is voldoende voor het beoogde gebruik.

## State shape
```js
const photo = {
  src: null,      // data-URL van de geüploade foto
  name: '',       // bestandsnaam, gebruikt als alt-tekst
  naturalW: 0, naturalH: 0,
  x: 0, y: 0,     // positie t.o.v. canvas-wrapper (px, preview-schaal)
  scale: 100,     // % t.o.v. natuurlijke grootte
};
const krib = {
  content: null,  // gesaniteerde/genormaliseerde SVG-markup
  x: 60, y: 60, scale: 60, rotate: 0,
  color: '#000000',
};
let exportFormat = 'digitaal'; // 'digitaal' (1080px) of 'print' (3240px)
```

## Deployment
Repo: [`kindkidnils/tkk-collage-maker`](https://github.com/kindkidnils/tkk-collage-maker) — gehost via GitHub Pages op de `main`-branch, root. Enkelvoudig HTML-bestand (`index.html`), geen build step; SVG-assets in `kribbels/` en `krabbels/`, gerefereerd via `manifest.json`.

Gebouwd volgens de TKK-tool-template (spoor A, single-file vanilla). Bij een volgende update: her-lees de template-skill (`tkk-tool-template`) voor de actuele visuele spec voordat je structuur/CSS aanpast.
