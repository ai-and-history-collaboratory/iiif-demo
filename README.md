# IIIF Demo — AI + History Collaboratory

Static IIIF image tiles served from GitHub Pages, prepared for the AI + History Collaboratory sessions (February–March 2026).

## Quick Start

Open the manifest in [Mirador](https://projectmirador.org/demo):

```
https://ai-and-history-collaboratory.github.io/iiif-demo/manifests/demo-manifest.json
```

## Images

**Takvim-i Vekayi, No. 1 (1 November 1831)** — First page of the first issue of the Ottoman Government Gazette. Source: [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Takvim-i_Vekayi_-_0001.pdf). Public Domain.

## How It Works

Source images are downscaled and converted into IIIF Level 0 static tile pyramids (256×256px tiles at multiple zoom levels). A IIIF Presentation API v3 manifest references the tile endpoints. Any IIIF-compatible viewer (Mirador, Universal Viewer, liiive) can load the manifest and provide deep-zoom browsing — no image server required.

## Structure

```
iiif-demo/
├── images/
│   └── takvim-i-vekayi-1831/   (tile pyramid + info.json)
├── manifests/
│   └── demo-manifest.json       (IIIF Presentation API v3)
├── index.html                    (landing page with instructions)
└── README.md
```

## Created With

Tiles generated using [Claude Cowork](https://claude.ai) and the Python `iiif` library. No image server infrastructure is needed — GitHub Pages serves the static tiles directly.
