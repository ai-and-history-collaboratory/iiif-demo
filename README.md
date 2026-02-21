# IIIF Demo — AI + History Collaboratory

A bespoke deep-zoom annotation viewer for Ottoman Turkish historical documents, demonstrating AI-assisted Historical Text Recognition (HTR) as part of the [Opening the Ottoman Archive](https://github.com/ottoman-archive/.github/wiki) initiative. Built for the AI + History Collaboratory sessions (February–March 2026) and for the broader community of Ottoman Turkish scholars.

## View the Demo

**[Open the annotation viewer →](https://ai-and-history-collaboratory.github.io/iiif-demo/viewer.html)**

The viewer shows page 1 of *Takvîm-i Vekâyi* (Calendar of Events), the Ottoman Government Gazette, Issue 1 (1 November 1831). Click on a highlighted region of the image to see five independently toggleable annotation layers: original Perso-Arabic script, scholarly transliteration, literal English, modernised English, and a flowing summary translation.

## What This Demonstrates

A high-resolution historical document image is served as deep-zoomable IIIF tiles and paired with multi-layer scholarly annotations — all hosted as static files on GitHub Pages with zero server-side dependencies. The annotations were produced using a two-stage AI-assisted HTR pipeline:

- **Stage 1 — Visual capture** performed in Google Gemini using the [V3-S-Minimal](https://github.com/ottoman-archive/.github/wiki/Visual-Capture) skill file, converting ink patterns to Unicode characters with no semantic interpretation.
- **Stage 2 — Semantic processing** performed by Claude (Opus 4.5/4.6) using the [V3-T-Government-Gazette](https://github.com/ottoman-archive/.github/wiki/Semantic-Processing) skill file, producing transliteration, translation, Named Entity Recognition, and summary.

The source images and HTR transcriptions are in the public repository [`colin-yournamehere-htr`](https://github.com/ottoman-archive/colin-yournamehere-htr).

## Why a Bespoke Viewer?

We evaluated Mirador, Universal Viewer, and Glycerine Viewer and found that no existing IIIF viewer supports per-annotation-layer toggling — the ability to independently show or hide the original script, transliteration, and translations. A bespoke viewer built on [OpenSeadragon](https://openseadragon.github.io/) was the only solution that met all requirements. The full evaluation and design rationale is documented in [`docs/IIIF_Annotation_Viewer_Memo_v2.1_21022026.md`](docs/IIIF_Annotation_Viewer_Memo_v2.1_21022026.md).

The IIIF Presentation API v3 manifest remains available for use in standard IIIF viewers, though without layer toggling.

## Viewer Features

- **Five toggleable annotation layers** — Original Script (RTL), Transliteration, Literal English, Modernised English, Summary Translation
- **Deep zoom** — scroll, click, double-click, pinch; right-click to zoom out
- **Three clickable regions** — Masthead, Right-Hand Column, Left-Hand Column
- **Collapsible sidebar** — resize by dragging, collapse/expand with arrow toggle
- **Fullscreen** — includes sidebar and toggle controls
- **Static hosting** — single HTML file, OpenSeadragon from CDN, no build step

## Document

**Takvîm-i Vekâyi, No. 1, Page 1 (1 November 1831)** — the first page of the first issue of the Ottoman Government Gazette, established by Sultan Mahmud II. Source: [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Takvim-i_Vekayi_-_0001.pdf) (Public Domain). 93 annotated lines across three regions.

## Repository Structure

```
iiif-demo/
├── viewer.html                          ← Bespoke annotation viewer (self-contained)
├── index.html                           ← Landing page
├── README.md
├── docs/
│   └── IIIF_Annotation_Viewer_Memo_v2.1_21022026.md   ← Design rationale and project memo
├── images/
│   └── takvim-i-vekayi-1831/            ← IIIF Level 0 static tile pyramid
│       ├── info.json                    ← IIIF Image API 2.1 descriptor
│       └── [tile directories]           ← 256px tiles at 5 scale factors
├── manifests/
│   └── demo-manifest.json               ← IIIF Presentation API v3 manifest
└── annotations/
    └── annotations-layered.json         ← Structured annotation data (JSON)
```

## Built With

- **[Claude Cowork](https://claude.ai)** (Opus 4.6) — viewer design, annotation structuring, iterative refinement
- **[OpenSeadragon](https://openseadragon.github.io/)** v4.1.1 — deep-zoom tile navigation
- **[iiif_static](https://pypi.org/project/iiif/)** (Python) — IIIF Level 0 tile generation
- **GitHub Pages** — static hosting, no image server required

## Learn More

- [Opening the Ottoman Archive wiki](https://github.com/ottoman-archive/.github/wiki) — project overview
- [Visual Capture](https://github.com/ottoman-archive/.github/wiki/Visual-Capture) — Stage 1 HTR with Gemini
- [Semantic Processing](https://github.com/ottoman-archive/.github/wiki/Semantic-Processing) — Stage 2 HTR with Claude
- [Project memo](docs/IIIF_Annotation_Viewer_Memo_v2.1_21022026.md) — full design rationale, viewer evaluation, and next steps
