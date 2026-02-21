# Building a Bespoke IIIF Annotation Viewer for the Takvîm-i Vekâyi

**Version:** 2.1
**Date:** 21 February 2026
**Author:** Colin Greenstreet & Claude Cowork (Opus 4.6)
**Repository:** [ai-and-history-collaboratory/iiif-demo](https://github.com/ai-and-history-collaboratory/iiif-demo)
**Live viewer:** [viewer.html on GitHub Pages](https://ai-and-history-collaboratory.github.io/iiif-demo/viewer.html)

---

## Executive Summary

This memo documents the design, evaluation, construction, and planned extension of a bespoke IIIF annotation viewer for *Takvîm-i Vekâyi* (Calendar of Events), the Ottoman Government Gazette, beginning with Issue 1, dated 1 November 1831. The viewer was built to demonstrate AI-assisted Historical Text Recognition (HTR) as part of the "Opening the Ottoman Archive" initiative, for presentation at the AI + History Collaboratory session on 24 February 2026 and for the broader community of Ottoman Turkish scholars.

The core challenge was to display a deep-zoomable historical document image alongside multi-layer scholarly annotations — original Perso-Arabic script, scholarly transliteration, literal English translation, modernised English translation, and a flowing summary — with the ability to toggle each layer independently. After evaluating Mirador, Universal Viewer, and Glycerine Viewer, we concluded that no existing IIIF viewer supports per-annotation-layer toggling. We therefore built a single-file custom viewer using OpenSeadragon, hosted as a static page on GitHub Pages with zero server-side dependencies.

The viewer currently shows Page 1 of Issue 1 with 93 annotated lines across three regions. The immediate next step is to extend the viewer to cover all five pages of Issue 1 for which HTR transcriptions exist, adding a page selector and per-page tile pyramids. Looking further ahead, we assessed what it would take to build a dynamic version of this system — one that could accept new document uploads and serve annotations without manual HTML generation — and concluded that while technically feasible (~26–38 hours of collaborative development), this is best parked until mid-2026 for discussion with the IIIF community, including Rainer Simon and others working on annotation tooling.

The entire pipeline — from source image acquisition through tile generation, HTR transcription, annotation structuring, viewer construction, and iterative refinement — was carried out within Claude Cowork sessions, with the exception of the Stage 1 visual capture (performed in Google Gemini).

---

## Table of Contents

1. [Goal](#1-goal)
2. [Source Materials](#2-source-materials)
3. [The Annotation Challenge](#3-the-annotation-challenge)
4. [Options Evaluated](#4-options-evaluated)
5. [Solution Chosen: Bespoke OpenSeadragon Viewer](#5-solution-chosen-bespoke-openseadragon-viewer)
6. [What Was Built](#6-what-was-built)
7. [Iterative Refinements](#7-iterative-refinements)
8. [Commit History](#8-commit-history)
9. [Extending to Multiple Pages](#9-extending-to-multiple-pages)
10. [A Dynamic Version: Assessment and Decision](#10-a-dynamic-version-assessment-and-decision)
11. [Next Steps](#11-next-steps)

---

## 1. Goal

To create a publicly accessible, shareable demonstration showing how AI-assisted HTR can make Ottoman Turkish manuscripts readable to non-specialist audiences. Specifically:

- Serve high-resolution historical document images as deep-zoomable IIIF tiles, requiring no image server infrastructure.
- Overlay multi-layer scholarly annotations (original script, transliteration, literal translation, modernised translation, and summary) on identified regions of each page.
- Allow viewers to toggle annotation layers independently, so that (for example) a reader interested only in the modernised English can suppress the other layers.
- Support multiple pages within a single viewer, enabling a reader to navigate through a complete document.
- Host everything as static files on GitHub Pages, suitable for sharing via LinkedIn and presenting at the AI + History Collaboratory session on 24 February 2026 and to the broader community of Ottoman Turkish scholars.

---

## 2. Source Materials

### 2.1 The document

**Takvîm-i Vekâyi** (تقویم وقایع, "Calendar of Events") was the first official gazette of the Ottoman Empire, established by Sultan Mahmud II in 1831. Issue 1 was published on 1 November 1831 (25 Jumâdâ al-Âkhir 1247 AH). The source images are from [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Takvim-i_Vekayi_-_0001.pdf) (Public Domain).

### 2.2 Source images and HTR transcriptions

The source materials reside in the public repository [`colin-yournamehere-htr`](https://github.com/ottoman-archive/colin-yournamehere-htr) within the `ottoman-archive` GitHub organisation — a shared demo repository for Colin Greenstreet and YourNameHere historians. The `ottoman-archive` organisation contains both public repositories and private repositories for collaborators; readers can learn more about the initiative at the [Opening the Ottoman Archive wiki](https://github.com/ottoman-archive/.github/wiki). The `document-images/Takvim-i-Vekayi-issue-1/` directory contains cropped region images for all eight pages of Issue 1. The `semantic-processing-output/Takvim-i-Vekayi-issue-1/` directory contains HTR markdown files for pages 1–5.

| Page | Regions | Document images | HTR markdown files |
|------|---------|----------------|--------------------|
| P1 | Masthead, RH Column, LH Column | ✓ | ✓ |
| P2 | RH Column, LH Column | ✓ | ✓ |
| P3 | RH Column, LH Column | ✓ | ✓ |
| P4 | RH Column, LH Column | ✓ | ✓ |
| P5 | RH Column, LH Column | ✓ | ✓ |
| P6–P8 | RH Column, LH Column | ✓ | Not yet processed |

Each HTR markdown file was produced using the project's two-stage pipeline: Stage 1 [visual capture](https://github.com/ottoman-archive/.github/wiki/Visual-Capture) was performed in Google Gemini 3 Pro Preview (using the V3-S-Minimal-Takvîm-i-Vekâyi skill file at extreme visual-grounding settings), and Stage 2 [semantic processing](https://github.com/ottoman-archive/.github/wiki/Semantic-Processing) — diplomatic transliteration, literal translation, modernised translation, Named Entity Recognition, and summary — was performed by Claude Opus 4.5/4.6 (using the V3-T-Government-Gazette skill file). Each line in the markdown files carries four parallel fields: original Perso-Arabic script, scholarly transliteration, literal English, and modernised English.

### 2.3 IIIF tile pyramid (Page 1)

The Page 1 source image (original resolution: 13,075 × 9,129 pixels) was downscaled to 5,000 × 3,491 pixels and converted into a IIIF Image API 2.1 Level 0 static tile pyramid (256×256px tiles at five scale factors) using the Python `iiif_static` library within Claude Cowork. The tiles and `info.json` descriptor are served from GitHub Pages at:

```
https://ai-and-history-collaboratory.github.io/iiif-demo/images/takvim-i-vekayi-1831/
```

Tile pyramids for pages 2–5 will follow the same structure under separate subdirectories.

### 2.4 Bounding box coordinates (Page 1)

Three rectangular regions were identified programmatically by image analysis of the source page:

| Region | Pixel coordinates (x, y, w, h) |
|--------|---------------------------------|
| Masthead | 2431, 0, 2568, 698 |
| Right-Hand Column | 3461, 703, 1538, 2797 |
| Left-Hand Column | 2431, 703, 1010, 2797 |

Bounding boxes for pages 2–5 will be generated using the same approach. Because the gazette uses a consistent two-column layout, the RH and LH column coordinates are expected to be broadly similar across pages, with adjustments for page-specific variations in column width and vertical extent.

---

## 3. The Annotation Challenge

IIIF provides excellent standards for serving images (Image API) and describing them (Presentation API), and the W3C Web Annotation model used by IIIF Presentation API v3 supports rich annotation bodies. The challenge was not getting annotations *into* a manifest — it was getting them *out* in a usable form.

The project required something that no standard IIIF viewer currently offers: the ability to treat annotation layers as independently toggleable overlays, so that a reader can (for example) view only the original script and the modernised English side by side, or suppress everything except the flowing summary translation. This is conceptually analogous to layer visibility in a GIS or Photoshop — but the IIIF annotation model does not distinguish between "layers" within a single AnnotationPage.

---

## 4. Options Evaluated

### 4.1 Mirador

[Mirador](https://projectmirador.org/) (v3) is the most widely used IIIF viewer for scholarly annotation work. We tested it extensively:

- **Annotation display works** — Mirador renders annotations from AnnotationPages inlined in the manifest, including HTML bodies.
- **HTML sanitisation is aggressive** — Mirador strips `<table>`, `<style>`, and most CSS from annotation bodies, making it impossible to present structured multi-layer content with visual differentiation.
- **No layer toggling** — Mirador's "Layers" panel controls image layers (i.e. multiple `Choice` bodies on a painting annotation, analogous to Photoshop layers). It does not provide any mechanism to filter or toggle annotation content by type, motivation, or any other property.
- **External AnnotationPages not fetched** — despite the IIIF specification allowing external AnnotationPage references, Mirador does not fetch them; annotations must be inlined.

**Verdict:** Mirador is excellent for displaying a single annotation body per region but cannot support the multi-layer toggling this project requires.

### 4.2 Universal Viewer

[Universal Viewer](https://universalviewer.io/) (UV) is the other major IIIF viewer, widely deployed by cultural heritage institutions:

- **Minimal annotation UI** — UV displays annotations in a side panel but with very limited formatting.
- **No filtering or toggling** — UV provides no mechanism to filter annotations by motivation, collection membership, or any custom property.
- **No layer concept** — unlike Mirador, UV does not even have a layers panel for image layers.

**Verdict:** UV offers less annotation functionality than Mirador and is not suitable for this use case.

### 4.3 Glycerine Viewer

[Glycerine Viewer](https://glycerine.io/) is a newer IIIF viewer developed by Digirati with a more comprehensive annotation feature set:

- **Annotation collection filtering** — Glycerine supports the `partOf` property on annotations, allowing them to be grouped into named collections that can be toggled independently.
- **Richer rendering** — better support for HTML annotation bodies.

**Verdict:** Glycerine was the most promising existing viewer, but its `partOf`-based filtering would require restructuring annotations into separate collections per layer — a non-standard pattern that would not interoperate cleanly with other viewers. Additionally, Glycerine is less mature and less widely known than Mirador, reducing its value as a demonstration tool.

### 4.4 Custom viewer (chosen)

Building a bespoke viewer using [OpenSeadragon](https://openseadragon.github.io/) for deep-zoom tile navigation, with custom JavaScript for annotation rendering and layer toggling:

- **Full control over annotation presentation** — no HTML sanitisation, no reliance on viewer-specific annotation handling.
- **Per-layer toggling** — straightforward to implement with CSS class visibility and checkbox state.
- **Single-file architecture** — all HTML, CSS, JavaScript, and annotation data in one self-contained file, with only OpenSeadragon loaded from CDN.
- **Zero server dependencies** — works as a static file on GitHub Pages.
- **Shareable** — a single URL that works in any modern browser.

**Verdict:** A custom viewer was the only option that met all requirements. The trade-off — losing interoperability with the broader IIIF viewer ecosystem — was acceptable for a demonstration project.

---

## 5. Solution Chosen: Bespoke OpenSeadragon Viewer

The final architecture is a single HTML file (`viewer.html`, ~76KB) containing:

- **OpenSeadragon v4.1.1** loaded from jsDelivr CDN, providing deep-zoom pan/zoom navigation over the IIIF tile pyramid.
- **Embedded annotation JSON** (~60KB) containing per-region, per-line, per-layer annotation data plus flowing summary translations, generated by parsing the three HTR markdown files.
- **Custom CSS** implementing a dark-themed split-screen layout with a collapsible sidebar.
- **Custom JavaScript** managing region selection, layer toggling, overlay positioning, sidebar resize, and fullscreen behaviour.

The viewer reads its tile source from the same IIIF Image API endpoint used by the Mirador manifest, so the underlying image infrastructure is shared. The annotation data is structurally independent from the IIIF manifest — it uses a simpler JSON format optimised for the viewer's layer-toggling requirements.

---

## 6. What Was Built

### 6.1 User interface

The viewer presents a split-screen layout:

- **Left panel (sidebar):** Title, date, and credits header; five layer-toggle checkboxes (Original Script, Transliteration, Literal English, Modernised English, Summary Translation), each with a colour-coded swatch; three region navigation buttons; and a scrollable annotation panel showing line-by-line content for the selected region.
- **Right panel (viewer):** Deep-zoomable image with three clickable semi-transparent overlay rectangles marking the annotated regions. Includes a mini-map navigator (top-left) and standard zoom controls (top-right).

### 6.2 Interaction features

| Feature | Implementation |
|---------|----------------|
| Layer toggling | Five checkboxes controlling CSS visibility of `data-layer` attributed elements |
| Region selection | Click overlay rectangles on the image or use sidebar buttons; viewer auto-zooms to the selected region |
| Deep zoom | Scroll wheel, click-to-zoom, double-click, pinch (touch) |
| Right-click zoom out | `contextmenu` event intercepted, calls `viewport.zoomBy(0.667)` |
| Sidebar resize | Drag handle between sidebar and viewer |
| Sidebar collapse/expand | Arrow toggle button, works in both normal and fullscreen modes |
| Fullscreen | OpenSeadragon's fullscreen button, overridden to put the entire app (sidebar + viewer) into browser fullscreen via `requestFullscreen()` on the `.app` container |
| RTL script rendering | `direction: rtl` and `text-align: right` on original script lines; layer tags float left for consistent positioning |
| Blank line suppression | When all four line-level layers are hidden (e.g. only Summary is active), individual line blocks are hidden via an `all-hidden` CSS class |
| Credits | Inline text in sidebar header with expandable detail panel |
| Open Graph meta tags | Title, description, and preview image for LinkedIn sharing |

### 6.3 Files in the repository

```
iiif-demo/
├── viewer.html                          ← Bespoke annotation viewer (76KB, self-contained)
├── index.html                           ← Landing page with Mirador instructions
├── README.md                            ← Repository documentation
├── images/
│   └── takvim-i-vekayi-1831/            ← IIIF Level 0 static tile pyramid (Page 1)
│       ├── info.json                    ← IIIF Image API descriptor
│       └── [tile directories]           ← 256px tiles at 5 scale factors
├── manifests/
│   └── demo-manifest.json               ← IIIF Presentation API v3 manifest (for Mirador)
└── annotations/
    └── annotations-layered.json         ← Structured annotation data (for reference/rebuild)
```

---

## 7. Iterative Refinements

The viewer was built and refined across multiple commits, with each round of user testing driving specific improvements:

| Round | Changes |
|-------|---------|
| Initial build | Four-layer annotation viewer with OpenSeadragon, region overlays, dark theme, layer toggles |
| First refinement | Added right-click zoom out; moved credits from floating badge to inline sidebar header with expandable detail; added Summary Translation as fifth toggleable layer; removed "Open in Mirador" link from info bar; added sidebar collapse/expand toggle |
| Second refinement | Brightened credits text (inline `#666` → `#aaa`, strong `#999` → `#ddd`, detail `#555` → `#999`); added blank line suppression when only summary layer is active; fixed fullscreen to include sidebar (overriding OpenSeadragon's default fullscreen to use `.app` container) |
| Final fix | Added `.summary-section.hidden` CSS rule — the summary section was not responding to the layer toggle because only `.layer-line.hidden` had a `display: none` rule |

---

## 8. Commit History

| Commit | Message |
|--------|---------|
| `0e3a774` | Add IIIF static tiles and manifest for Takvim-i Vekayi No. 1 |
| `f40d05a` | Add HTR annotations with four-layer translations for Page 1 |
| `bcf0ae0` | Inline annotations into manifest for Mirador compatibility |
| `9751ca5` | Rebuild annotations: full line-by-line content, improved bounding boxes, simple HTML |
| `542b13d` | Add custom annotation viewer with four-layer toggle |
| `1fcc891` | Add custom annotation viewer with four-layer toggle, credits badge, smooth zoom |
| `d1c84dc` | Viewer refinements: right-click zoom out, sidebar credits, summary layer toggle, sidebar collapse |
| `7fe76da` | Fix: summary visibility toggle, brighter credits, fullscreen sidebar |

---

## 9. Extending to Multiple Pages

### 9.1 What exists

HTR transcription files and document images exist for pages 1–5 of Issue 1 in the private `ottoman-archive` GitHub organisation. Pages 6–8 have document images but no HTR output yet. Page 1 alone has a masthead region; pages 2–8 follow a consistent two-column layout (Right-Hand Column + Left-Hand Column).

### 9.2 What needs to happen

Extending the viewer from one page to five requires three categories of work:

**Per-page tile generation.** Each page needs its own IIIF Level 0 tile pyramid, generated from the full-page source image using the same `iiif_static` pipeline used for Page 1. Each tile pyramid lives in its own subdirectory (e.g. `images/takvim-i-vekayi-1831-p2/`). At roughly 15–20MB per page, five pages would total ~75–100MB — well within GitHub Pages' 1GB repository limit.

**Per-page annotation data.** Each page's HTR markdown files need to be parsed into the same structured JSON format used for Page 1 (regions → lines → layers, plus summary). For pages 2–5, which lack a masthead, each page will have two regions rather than three. Bounding box coordinates will be generated programmatically for each page, adjusted for page-specific layout variations.

**Viewer modifications.** The viewer needs a page selector in the sidebar (a dropdown or row of page buttons), and logic to swap tile sources and annotation data when the user selects a different page. OpenSeadragon supports tile source swapping natively via `viewer.open()`. The existing region overlay, layer toggle, and sidebar code will work unchanged — they already operate on whatever data object is current.

Estimated modification: roughly 50–80 lines of new JavaScript (page selector, tile source swapping, annotation data switching) and 20 lines of CSS for the page selector UI. The existing viewer architecture was designed with this extension in mind.

### 9.3 Architecture choice: embedded vs external annotation data

With one page, embedding the annotation JSON directly in the HTML was the simplest approach (~60KB of inline data). With five pages, the total annotation data will be roughly 250–350KB. Two options:

**Option A: Embed all pages.** Keep everything in a single HTML file. Total file size would be ~350–450KB — large but still fast to load. Advantage: zero network requests after initial page load; no CORS or fetch issues. Disadvantage: every page's data loads whether the user views it or not.

**Option B: External JSON per page.** Move annotation data to separate JSON files (e.g. `annotations/p1.json`, `annotations/p2.json`) loaded on demand via `fetch()`. Advantage: faster initial load; cleaner separation of concerns. Disadvantage: adds fetch latency when switching pages; requires CORS headers (not an issue on GitHub Pages, which serves with permissive headers).

For five pages, either option is viable. Option A is simpler and avoids any risk of fetch failures. If the viewer later expands to the full eight pages or to multiple issues, Option B becomes the better architecture.

---

## 10. A Dynamic Version: Assessment and Decision

### 10.1 What "dynamic" would mean

The current viewer is entirely static: HTR transcriptions are processed offline, annotation JSON and viewer HTML are generated in Claude Cowork sessions, and the output is pushed to GitHub Pages. A dynamic version would accept new document uploads, tile them automatically, ingest HTR output, structure annotations, and serve the viewer — without manually editing HTML or pushing to git.

### 10.2 What would need to be built

A dynamic system would require five components:

**Image processing pipeline.** An upload endpoint that accepts a document image, downscales it, generates IIIF Level 0 tiles, and stores them on disk. The Python `iiif_static` library already does the tiling; wrapping it in an API endpoint is straightforward.

**Annotation data store.** A backend that stores per-page, per-region, per-line annotation data. This could be as simple as JSON files on disk (matching the current format) or a lightweight database (SQLite) if search or editing functionality were desired.

**API layer.** A REST API (Flask or FastAPI in Python) serving tile `info.json` descriptors, annotation data, and page metadata to the viewer. The viewer's JavaScript would fetch from this API instead of reading embedded data.

**Viewer modifications.** The frontend viewer would need only modest changes: replace the embedded `DATA` object with `fetch()` calls to the API, and add a page/document selector that queries available documents dynamically.

**Admin interface.** A simple web form for uploading images and associating HTR markdown files with pages and regions. This is the most labour-intensive component, because it requires UI for managing the document → page → region → annotation hierarchy.

### 10.3 Estimated effort

Based on the working pace established across our Cowork sessions — including the characteristic pattern of iterative build-test-refine cycles — the estimated effort for a minimum viable dynamic version is:

| Component | Estimated hours |
|-----------|-----------------|
| Tile generation service (auto-tile on upload) | 4–6 |
| Annotation data API (CRUD for pages, regions, layers) | 6–8 |
| Viewer modifications (fetch from API, document/page selector) | 3–4 |
| Admin upload interface (simple web form) | 4–6 |
| Deployment to VPS with domain and HTTPS | 3–4 |
| Testing, debugging, and iteration | 6–10 |
| **Total** | **~26–38** |

Those are collaborative hours — sessions where we are actively working together. They include the back-and-forth of testing, finding issues, and fixing them, which accounts for roughly 40% of the total based on our experience to date.

### 10.4 Hosting options

**Local workstation.** Python (Flask/FastAPI) serving both tiles and the API on `localhost`. Functional for personal use but not accessible to collaborators without tunnelling (ngrok or similar), which is fragile and not suitable for a demo.

**Hosted server.** A small VPS (DigitalOcean, Hetzner, or similar — £5–10/month) running the same Python stack behind nginx, with a domain name and HTTPS via Let's Encrypt. This would be accessible to anyone with the URL.

### 10.5 Cowork vs Claude Code

The static viewer was well suited to Claude Cowork: iterative design, file generation, research, and pushing changes through git. A dynamic backend project — with its multi-file codebase, local server processes, test cycles, and deployment scripts — would be better suited to Claude Code, which runs directly in the development environment with full filesystem access and the ability to run and test servers locally.

### 10.6 Decision

The dynamic version is **parked**. The technical assessment confirms it is feasible and within our combined capabilities, but the effort is better invested after the Collaboratory session, when feedback from participants and from the broader IIIF community — including Rainer Simon and others working on annotation tooling — can inform the design. The dynamic version may also benefit from emerging developments in IIIF viewer annotation support, particularly in Glycerine Viewer, which is under active development.

In the meantime, we are extending the static viewer to cover all five available pages of Issue 1, creating a multi-page demo for the AI + History Collaboratory on 24 February 2026 and for sharing with the broader community of Ottoman Turkish scholars.

---

## 11. Next Steps

### Immediate (before 24 February 2026)

- **Multi-page viewer** — extend the bespoke viewer to support pages 1–5 of Issue 1, with a page selector, per-page tile pyramids, and per-page annotation data.
- **Tile generation for pages 2–5** — generate IIIF Level 0 tile pyramids for the remaining four pages using the same pipeline as Page 1.
- **Bounding box generation for pages 2–5** — programmatic region identification for each page's two-column layout.
- **Custom domain** — configure a subdomain (e.g. `iiif.ai-and-history.org`) pointing to the GitHub Pages site, for cleaner sharing URLs and Open Graph previews.

### Short-term (before 31 March 2026)

- **Line-level bounding boxes** — use Gemini 3 Pro Preview to generate per-line bounding box coordinates, enabling click-on-image-to-highlight-annotation interaction (Phase 2, targeted for the 31 March session).
- **NER tables** — display Named Entity Recognition output (persons, places, dates, vessels) as a toggleable panel, with mockup needed for UI/UX review.
- **Second document type** — process HCA 13/71 f.1r (a mid-1650s English maritime deposition from the MarineLives corpus) to demonstrate the same pipeline on a second script and language.

### Medium-term (mid-2026)

- **Dynamic version** — revisit the dynamic architecture (Section 10) in light of Collaboratory feedback and IIIF community developments. Discuss with Rainer Simon and others. If pursued, build in Claude Code rather than Claude Cowork.
- **Video walkthrough** — record a demo walkthrough for asynchronous sharing with collaborators.
- **Summary landing page** — create a page describing the full pipeline from source image to annotated viewer.
- **Pages 6–8** — run HTR pipeline on the remaining three pages of Issue 1 to complete the full issue.
