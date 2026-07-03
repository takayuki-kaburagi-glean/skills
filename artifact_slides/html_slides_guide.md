# HTML Slides Style Guide

Build an Interactive Slides (HTML) deck: a single self-contained `.html` artifact that presents one responsive 16:9 slide at a time and exports one slide per PDF page.

Read `artifact_html/SKILL.md` and `html_artifact_guide` first for general HTML artifact rules, visual design, sandbox details, GleanBridge behavior, and generic PDF export. This file adds slide-specific guidance.

This guide has two parts. **Required structure** is a strict contract — follow it exactly. **Design latitude** is optional creative room — keep it simple by default and only reach for extras when they clearly help the content.

## Required structure

These rules are not optional. They keep slides from overflowing, overlapping, or breaking PDF export. Use the CSS/JS blocks below as your starting scaffold: keep the stage sizing, carousel, controls, fullscreen, and print recipe as-is, and change only the brand tokens and your slide content. Spend your effort on content and layout, not on rebuilding the chassis.

Make the very first line of the generated `.html` file an editor-warning comment, so a future agent editing the deck doesn't silently break the chassis (stage sizing, carousel, controls, and the `@media print` recipe):

```html
<!-- Read the html_slides_guide instructions before modifying this file. The stage sizing, carousel, controls, and @media print recipe are a strict contract — changing them breaks slide fit and PDF export. -->
```

### Stage and sizing

Use a single-slide stage carousel. One shared stage size drives the deck, the track, and every slide. Size all in-slide content (type, spacing, radii, icons, shapes, charts) in pure `cqw` / `cqi` from the slide container so everything scales together as the window shrinks. Do not use `vw` / `vh`, fixed pixels, or a separate scaled 1280px box. Avoid px floors inside slides — a `clamp()` with a px minimum or a px `min-width`/`min-height` on text and shapes makes them stop shrinking at small windows while the rest of the slide keeps scaling (the `min-width: 0` / `min-height: 0` shrink-enabler on grid children is fine and different). For text line length use a character measure (`max-width: NNch`) or none — never `cqw`, which produces a tiny measure that wraps headings into a tall column and overflows.

```css
:root {
  --slide-w: 1280px;
  --slide-h: 720px;
  --stage-w: min(var(--slide-w), 100vw);
  --stage-h: calc(var(--stage-w) * 9 / 16);
}

body { margin: 0; overflow: hidden; }
.deck-viewport { position: fixed; inset: 0; display: grid; place-items: center; overflow: hidden; }
.deck { width: var(--stage-w); height: var(--stage-h); overflow: hidden; }
.deck-track {
  height: 100%;
  display: flex;
  transform: translateX(calc(var(--i, 0) * -1 * var(--stage-w)));
  transition: transform 260ms ease;
}
.slide {
  flex: 0 0 var(--stage-w);
  width: var(--stage-w);
  height: var(--stage-h);
  container-type: inline-size;
  position: relative;
  display: grid;
  overflow: clip;
  contain: layout paint;
}
.slide > * { min-width: 0; min-height: 0; }
```

The screen view is the **same slide composition scaled down**, never a reflowed document. Do not add media/container queries that collapse a two-column slide into one column or turn a 4-up grid into 2-up. Pick the layout at authoring time; smaller widths shrink it proportionally.

Consider the following HTML skeleton as reference:

```html
<div class="deck-viewport">
  <div class="deck">
    <div class="deck-track">
      <section class="slide layout-two-col">
        <div class="slide-body">
          <header class="slide-head"> ... eyebrow / title / brand mark ... </header>
          <main class="slide-main"> ... slide content ... </main>
          <footer class="slide-foot"> ... source / page number ... </footer>
        </div>
      </section>
      <!-- more <section class="slide ..."> here -->
    </div>
  </div>
</div>
```

### Slide body contract

`.slide-body` has **exactly three direct children**: a header, `.slide-main`, and a footer — matching the three grid rows. Put the eyebrow/title in the header; never add a 4th direct child (a stray title or note block becomes an extra row and overflows).

```css
.slide-body {
  position: absolute;
  inset: 5cqw;
  display: grid;
  grid-template-rows: auto minmax(0, 1fr) auto;
  gap: 2cqw;
  min-height: 0;
  overflow-wrap: anywhere;
}
.slide-main { min-width: 0; min-height: 0; display: grid; gap: 1.5cqw; align-content: center; }
.layout-two-col  .slide-main { grid-template-columns: minmax(0, 1.2fr) minmax(0, 1fr); }
.layout-kpi-grid .slide-main { grid-template-columns: repeat(4, minmax(0, 1fr)); }
.layout-chart    .slide-main,
.layout-table    .slide-main { grid-template-rows: minmax(0, 1fr); }
h1, h2 { text-wrap: balance; }
```

- Keep content minimal per slide: one idea, one primary visual/table, a few supporting points. Prefer one extra slide over one crowded slide.
- Grid children must fit their track. Do not use `align-items: end`, large `min-height`, negative margins, or absolute positioning to push panels into the headline or chrome.
- If content does not fit, split or shorten it. Do not shrink text to unreadable sizes and do not reflow.
- Before returning, do a manual fit pass slide by slide and simplify anything close to the edge.

### Controls and fullscreen

Keep all slides in the DOM and move the track; do not use `display: none` for inactive slides. Provide one compact control bar outside `.deck` with Previous, Next, slide count, and Fullscreen. Support ArrowLeft/ArrowRight, PageUp/PageDown, Home/End, `f` for fullscreen, and `Esc` to exit. Fullscreen targets the viewport.

```css
.deck-viewport:fullscreen {
  background: #000;
  --stage-w: min(100vw, calc(100vh * 16 / 9));
  --stage-h: min(100vh, calc(100vw * 9 / 16));
}
.deck-viewport:fullscreen .deck { box-shadow: none; }
```

Use this navigation + export wiring as-is; it is chassis, not something to redesign per deck:

```js
const viewport = document.querySelector('.deck-viewport');
const track = document.querySelector('.deck-track');
const slides = [...document.querySelectorAll('.slide')];
let i = 0;
const setActive = n => {
  i = Math.max(0, Math.min(slides.length - 1, n));
  track.style.setProperty('--i', i);
  document.querySelector('#counter')?.replaceChildren(`${i + 1} / ${slides.length}`);
};
const toggleFullscreen = () => document.fullscreenElement
  ? document.exitFullscreen()
  : viewport?.requestFullscreen?.();
addEventListener('keydown', e => {
  if (e.key === 'ArrowRight' || e.key === 'PageDown') setActive(i + 1);
  else if (e.key === 'ArrowLeft' || e.key === 'PageUp') setActive(i - 1);
  else if (e.key === 'Home') setActive(0);
  else if (e.key === 'End') setActive(slides.length - 1);
  else if (e.key === 'f' || e.key === 'F') toggleFullscreen();
});
setActive(0);

// PDF export: register the host menu action, then print on the action.
GleanBridge?.postMessage?.({ actionId: 'export-pdf', type: 'glean-add-menu', metadata: { label: 'Export PDF', icon: 'export' } });
GleanBridge?.onMessage?.('action', async d => {
  if (d.actionId !== 'export-pdf') return;
  if (document.fullscreenElement) await document.exitFullscreen();
  await document.fonts.ready;
  window.print();
});
```

Wire the prev/next/fullscreen buttons to `setActive` and `toggleFullscreen`.

### Reuse one small system

Define styles once and reuse them. Do not invent a new grid system, min-height scheme, or decorative treatment per slide — that is the main cause of overlaps and inconsistency. Write `<style>` as a tiny design system before slide markup:

```css
/* 1. Tokens */  :root { --brand-primary; --ink; --ink-muted; --line; --surface; --type-display; --type-body; ... }
/* 2. Chrome */  .slide, .brand-mark, .pgnum, .eyebrow { ... }
/* 3. Layouts */ .title-slide, .section-divider, .layout-two-col, .layout-kpi-grid, .data-table { ... }
```

Slides compose these classes. Use inline `style=` only for genuinely one-off values.

Do not create slide-number-specific layout CSS like `.s2-body`, `.s3-cols`, or `.slide-7-grid`. Use the shared `.slide-body`, `.slide-main`, and a small set of semantic layout classes instead. Slide-specific CSS should be limited to accents such as background color or one image placement.

### Print stylesheet

Export wiring lives in the navigation block above; see `html_artifact_guide` for GleanBridge details and ECharts-in-print handling. The print stylesheet makes each slide exactly `1280px x 720px`, one slide per page, animations off, `print-color-adjust: exact`, controls hidden. Break *between* slides only — a `break-after: page` on the last slide adds a trailing blank page.

```css
@media print {
  @page { size: 1280px 720px; margin: 0; }
  html, body { width: var(--slide-w); margin: 0; overflow: visible; background: #fff; }
  .deck-viewport { position: static; display: block; overflow: visible; }
  .deck { width: auto; height: auto; transform: none; overflow: visible; box-shadow: none; }
  .deck-track { display: block; transform: none; transition: none; }
  *, *::before, *::after { animation: none !important; transition: none !important; }
  .slide {
    width: var(--slide-w);
    height: var(--slide-h);
    aspect-ratio: auto;
    break-inside: avoid;
    print-color-adjust: exact;
    -webkit-print-color-adjust: exact;
  }
  .slide:not(:last-child) { break-after: page; }  /* no break after the last slide */
  img, svg { max-width: none; }
  .controls { display: none !important; }
}
```

## Design latitude

The chassis is fixed, but the inside of `.slide-body` is your creative space. Design slide layouts and visuals freely there; just keep using the shared stage, fit primitives, and a small reused set of classes rather than a new system per slide.

This part is optional. Default to a simple, clean deck. Do not add features just because they are listed here; reach for them only when the content clearly benefits.

- **Default to simple.** A small reused set of layouts and one strong visual idea per slide beats many bespoke layouts. Static slides are fine when they communicate best.
- **Brand.** See the Brand and imagery section — every deck should be visibly on-brand (logo, colors, type, chart palette). This is a baseline, not an optional flourish.
- **Variety.** Vary layouts by composing the reused layout set (statement, two-column, KPI grid, comparison, table, chart, section divider, image, closing) — not by inventing new structures.
- **Charts.** Prefer simple CSS/SVG (bars, line `<polyline>`, donut, sparkline). Use ECharts only for genuinely complex charts. Numeric columns: right-aligned, `font-variant-numeric: tabular-nums`.
- **Diagrams.** For architecture, flow, or multi-node diagrams, use the image generation tool and embed the result. Hand-build CSS/SVG only for trivial 3-5 step flows.
- **Icons & libraries.** Prefer inline `<svg>` icons (copy paths from Lucide/Heroicons/Tabler). You can also pull libraries from `cdnjs`, `cdn.jsdelivr.net`, `unpkg`, `esm.sh`, or `cdn.tailwindcss.com`: an icon set that injects inline SVG from a bundled script (Lucide, Feather, or Font Awesome's SVG+JS build), Tailwind utility CSS, ECharts (SVG renderer), or a lightweight animation lib (animate.css, GSAP). Reach for a library only when it clearly helps; a few inline SVGs usually suffice.
- **Interactivity.** This is HTML's edge over static slides — use it when it helps the audience explore, not as filler. Good patterns: a range slider that recomputes a number/chart, tabs or toggles to switch views, hover/click to reveal detail, expandable `<details>`, an animated counter or bar on slide entry, or a step-through build. Wire state with plain inline JS and keep it lightweight. Anything interactive must still show a correct resting state in print (render the default value/expanded content).
- **Motion.** Subtle entrance fades or a count-up are nice; keep them short, optional, and disabled under `prefers-reduced-motion` and in print.
- **Decoration and TOC.** Optional and usually unnecessary. Keep decoration behind content (`z-index` low, `pointer-events: none`); add a TOC only when the user asks.
- **Section dividers.** For longer decks, a divider every 6-8 content slides helps structure.

## Brand and imagery

Branding is required, not decorative. Before writing markup, gather the brand basics — logo, colors, typography, and tone — and make every deck visibly on-brand, including text-heavy ones. The palette, chrome, and chart colors should all read as the company's.

- **Image generation is the primary path.** Generate logos, hero images, section art, and concept visuals with the image generation tool (use references when you have them) and embed the result. Prefer this over hunting for external images. Regenerate until the asset is clean; don't settle for a broken or low-quality mark.
- **Only embed allowed image sources.** Images render only from a base64 `data:` URI, a `blob:`, or a `*.glean.com` URL — arbitrary web URLs are blocked, so a Glean-search image works only if it is served from a Glean host. The robust path is base64: if a tool returns raw image bytes, embed them as a `data:` URI; otherwise generate the image and embed it. Never hotlink an arbitrary off-domain URL — it will silently fail to load.
- **Reuse base64 assets.** Base64 `data:` URIs are the best way to embed images. When the same image appears across slides, define the data URI once (for example in a JS `const assets = { logo: 'data:image/png;base64,...' }`) and apply it to repeated `<img data-asset="logo">` elements or CSS custom properties for backgrounds instead of pasting the same base64 blob throughout the deck.
- **Use web search for brand info, not for image links.** Web search and Glean Document Reader are for finding brand colors, fonts, and CSS values (e.g. scraped `--brand`/hex/`theme-color`) to drive your palette — not for pulling in arbitrary `<img>` URLs.
- **Logo in chrome.** Show the real logo (generated or from an allowed source); a text wordmark is a fallback, not the default. Ensure contrast on the surface (transparent/inverted as needed); if it can't sit cleanly, generate a title/section background with the logo embedded.
- **Place images correctly.** When image bytes are available locally, use PIL/Pillow to check dimensions and aspect ratio before placing. Put images in a fixed-ratio container, use `object-fit: contain` by default (`cover` only for intentional crops), and never stretch to a mismatched box.

## Footguns

Common ways decks break — avoid all of these.

- Don't reflow slides at small widths (collapsing columns, re-wrapping grids); scale the same composition instead.
- Don't size in-slide content with `vw` / `vh`, fixed pixels, or `transform: scale(...)`; use `cqw` / `cqi` from the slide container.
- Don't pin in-slide text or shapes with px floors (`clamp()` px minimum, `min-width`/`min-height` in px); they stop scaling at small windows. Size them in `cqw` / `cqi`.
- Don't set text `max-width` in `cqw` — use `ch` (e.g. `max-width: 22ch`) or omit it. A `cqw` measure is tiny, so a heading wraps into a tall narrow column that grows the header row and overflows the slide.
- Don't give panels fixed heights or push them into the headline/chrome with `align-items: end`, negative margins, or absolute positioning.
- Don't pack a slide until it clips; split or shorten rather than shrinking text to unreadable sizes.
- Don't invent a new layout or decorative system per slide; reuse one small system.
- Don't create per-slide body/layout classes (`.s2-body`, `.s3-cols`, etc.); use shared `.slide-body` and semantic layout classes.
- Don't `loading="lazy"` slide images or lazy-init charts; load both up front so every slide prints.
- Don't hide critical content behind interaction; tabs, drill-downs, and sliders must show a resting state in print.
- Don't omit `break-inside: avoid`, the `animation: none` print reset, or `print-color-adjust: exact`.
- Don't put `break-after: page` on the last slide; it adds a trailing blank PDF page. Use `.slide:not(:last-child)`.
- Don't stretch images or guess aspect ratio; check dimensions and use a fixed-ratio container with `object-fit: contain`.
- Don't use resources that need a runtime network call or a downloaded font; they silently fail. This means no Iconify or other lazy icon sets, no live-data fetches, no Google Fonts `<link>` or icon glyph fonts (use a system/web-safe stack or a base64-embedded `@font-face`), no `react-icons` (needs a bundler), and no nested iframes (YouTube/Vimeo/Figma embeds).
- Don't skip branding or fall back to a text wordmark without trying to fetch or generate the real logo.

## Final check

- One slide visible at a time, centered even in a narrow window; no horizontal scroll at ~900px width.
- Arrow keys, controls, and fullscreen work.
- Every slide fits without clipping; brand mark and page number on content slides.
- Data slides have readable labels, numeric alignment, and source attribution where relevant.
- PDF export is wired; PDF page count equals slide count.
