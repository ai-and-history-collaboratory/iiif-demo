# IIIF Demo — AI + History Collaboratory

A bespoke deep-zoom annotation viewer for Ottoman Turkish historical documents, demonstrating AI-assisted Historical Text Recognition (HTR) as part of the [Opening the Ottoman Archive](https://github.com/ottoman-archive/.github/wiki) initiative. Built for the AI + History Collaboratory sessions (February–March 2026) and for the broader community of Ottoman Turkish scholars.

## View the Demo

**[Open the annotation viewer →](https://ai-and-history-collaboratory.github.io/iiif-demo/viewer.html)**

The viewer shows all eight pages of *Takvîm-i Vekâyi* (Calendar of Events), the Ottoman Government Gazette, Issue 1 (1 November 1831), across four two-page spreads. Navigate between pages using the P1–P8 buttons in the sidebar or the thumbnail strip below the viewer. Click on a highlighted region to see five independently toggleable annotation layers: original Perso-Arabic script, scholarly transliteration, literal English, modernised English, and a flowing summary translation. Full HTR annotations are available for pages 1–5; pages 6–8 have bounding boxes but await processing.

## What This Demonstrates

High-resolution historical document images are served as deep-zoomable IIIF tiles and paired with multi-layer scholarly annotations — all hosted as static files on GitHub Pages with zero server-side dependencies. The annotations were produced using a two-stage AI-assisted HTR pipeline:

- **Stage 1 — Visual capture** performed in Google Gemini using the [V3-S-Minimal](https://github.com/ottoman-archive/.github/wiki/Visual-Capture) skill file, converting ink patterns to Unicode characters with no semantic interpretation.
- **Stage 2 — Semantic processing** performed by Claude (Opus 4.5/4.6) using the [V3-T-Government-Gazette](https://github.com/ottoman-archive/.github/wiki/Semantic-Processing) skill file, producing transliteration, translation, Named Entity Recognition, and summary.

The source images and HTR transcriptions are in the public repository [`colin-yournamehere-htr`](https://github.com/ottoman-archive/colin-yournamehere-htr).

## Why a Bespoke Viewer?

We evaluated Mirador, Universal Viewer, and Glycerine Viewer and found that no existing IIIF viewer supports per-annotation-layer toggling — the ability to independently show or hide the original script, transliteration, and translations. A bespoke viewer built on [OpenSeadragon](https://openseadragon.github.io/) was the only solution that met all requirements. The full evaluation and design rationale is documented in [`docs/IIIF_Annotation_Viewer_Memo_v3.0_22022026.md`](docs/IIIF_Annotation_Viewer_Memo_v3.0_22022026.md).

The IIIF Presentation API v3 manifest (with `viewingDirection: right-to-left`) remains available for use in standard IIIF viewers, though without layer toggling.

## Viewer Features

- **Eight pages across four spreads** — navigate via P1–P8 page buttons or clickable thumbnail strip
- **Five toggleable annotation layers** — Original Script (RTL), Transliteration, Literal English, Modernised English, Summary Translation
- **Deep zoom** — scroll, click, double-click, pinch; right-click to zoom out
- **Right-to-left navigation** — page buttons and thumbnails follow Ottoman reading order (P1 at far right)
- **Clickable annotated regions** — Masthead (P1), plus Right-Hand and Left-Hand columns for each page
- **Collapsible sidebar** — resize by dragging, collapse/expand with arrow toggle
- **Fullscreen** — includes sidebar and toggle controls
- **Static hosting** — single HTML file, OpenSeadragon from CDN, no build step

## Document

**Takvîm-i Vekâyi, No. 1 (1 November 1831)** — the complete first issue of the Ottoman Government Gazette, established by Sultan Mahmud II. Eight pages across four two-page spreads, with full HTR annotations for pages 1–5. Source: [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Takvim-i_Vekayi_-_0001.pdf) (Public Domain).

## Repository Structure

```
iiif-demo/
├── viewer.html                          ← Bespoke annotation viewer (~345KB, self-contained)
├── index.html                           ← Landing page
├── README.md
├── docs/
│   ├── IIIF_Annotation_Viewer_Memo_v3.0_22022026.md   ← Design rationale and project memo (current)
│   └── IIIF_Annotation_Viewer_Memo_v2.1_21022026.md   ← Previous version
├── images/
│   ├── takvim-i-vekayi-1831/            ← IIIF Level 0 tile pyramid (Spread 1, P1–P2)
│   ├── takvim-i-vekayi-1831-spread2/    ← IIIF Level 0 tile pyramid (Spread 2, P3–P4)
│   ├── takvim-i-vekayi-1831-spread3/    ← IIIF Level 0 tile pyramid (Spread 3, P5–P6)
│   └── takvim-i-vekayi-1831-spread4/    ← IIIF Level 0 tile pyramid (Spread 4, P7–P8)
├── manifests/
│   └── demo-manifest.json               ← IIIF Presentation API v3 manifest (RTL viewing direction)
└── annotations/
    └── annotations-layered.json         ← Structured annotation data (JSON)
```

## Built With

- **[Claude Cowork](https://claude.ai)** (Opus 4.6) — viewer design, annotation structuring, navigation design, iterative refinement
- **[OpenSeadragon](https://openseadragon.github.io/)** v4.1.1 — deep-zoom tile navigation
- **[iiif_static](https://pypi.org/project/iiif/)** (Python) — IIIF Level 0 tile generation
- **GitHub Pages** — static hosting, no image server required

## Learn More

- [Opening the Ottoman Archive wiki](https://github.com/ottoman-archive/.github/wiki) — project overview
- [Visual Capture](https://github.com/ottoman-archive/.github/wiki/Visual-Capture) — Stage 1 HTR with Gemini
- [Semantic Processing](https://github.com/ottoman-archive/.github/wiki/Semantic-Processing) — Stage 2 HTR with Claude
- [Project memo](docs/IIIF_Annotation_Viewer_Memo_v3.0_22022026.md) — full design rationale, viewer evaluation, and next steps
