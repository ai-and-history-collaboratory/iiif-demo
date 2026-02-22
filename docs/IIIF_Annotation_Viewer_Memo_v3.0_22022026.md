# Building a Bespoke IIIF Annotation Viewer for the Takvîm-i Vekâyi

**Version:** 3.0
**Date:** 22 February 2026
**Author:** Colin Greenstreet & Claude Cowork (Opus 4.6)
**Repository:** [ai-and-history-collaboratory/iiif-demo](https://github.com/ai-and-history-collaboratory/iiif-demo)
**Live viewer:** [viewer.html on GitHub Pages](https://ai-and-history-collaboratory.github.io/iiif-demo/viewer.html)

---

## Executive Summary

This memo documents the design, evaluation, construction, and extension of a bespoke IIIF annotation viewer for *Takvîm-i Vekâyi* (Calendar of Events), the Ottoman Government Gazette, beginning with Issue 1, dated 1 November 1831. The viewer was built to demonstrate AI-assisted Historical Text Recognition (HTR) as part of the "Opening the Ottoman Archive" initiative, for presentation at the AI + History Collaboratory session on 24 February 2026 and for the broader community of Ottoman Turkish scholars.

The core challenge was to display a deep-zoomable historical document image alongside multi-layer scholarly annotations — original Perso-Arabic script, scholarly transliteration, literal English translation, modernised English translation, and a flowing summary — with the ability to toggle each layer independently. After evaluating Mirador, Universal Viewer, and Glycerine Viewer, we concluded that no existing IIIF viewer supports per-annotation-layer toggling. We therefore built a single-file custom viewer using OpenSeadragon, hosted as a static page on GitHub Pages with zero server-side dependencies.

**Version 3.0** records the completion of the multi-page extension. The viewer now covers all eight pages of Issue 1 across four two-page spreads, with full HTR annotations for pages 1–5 and placeholder bounding boxes for pages 6–8. The navigation was redesigned from a two-tier spread+page interface to a single row of P1–P8 page buttons in the sidebar plus a Mirador-style thumbnail strip below the viewer. All navigation follows right-to-left reading order, matching the Ottoman Turkish text direction. The IIIF manifest now declares `viewingDirection: right-to-left`.

The entire pipeline — from source image acquisition through tile generation, HTR transcription, annotation structuring, viewer construction, and iterative refinement — was carried out within Claude Cowork sessions, with the exception of the Stage 1 visual capture (performed in Google Gemini).

---

## Table of Contents

1. [Goal](#1-goal)
2. [Source Materials](#2-source-materials)
3. [The Annotation Challenge](#3-the-annotation-challenge)
4. [Options Evaluated](#4-options-evaluated)
5. [Solution Chosen: Bespoke OpenSeadragon Viewer](#5-solution-chosen-bespoke-openseadragon-viewer)
6. [What Was Built: Single-Page Viewer](#6-what-was-built-single-page-viewer)
7. [Iterative Refinements: Phase 1](#7-iterative-refinements-phase-1)
8. [Commit History](#8-commit-history)
9. [What Was Built: Multi-Page Extension](#9-what-was-built-multi-page-extension)
10. [The Multi-Page Plan vs. What Actually Happened](#10-the-multi-page-plan-vs-what-actually-happened)
11. [A Dynamic Version: Assessment and Decision](#11-a-dynamic-version-assessment-and-decision)
12. [Next Steps](#12-next-steps)

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

### 2.3 IIIF tile pyramids

The viewer uses four IIIF Image API 2.1 Level 0 static tile pyramids, one per two-page spread:

| Spread | Pages | Original resolution | Tiled resolution | Tiles |
|--------|-------|--------------------|--------------------|-------|
| Spread 1 | P1–P2 | 13,075 × 9,129 px | 5,000 × 3,491 px | 256px, 5 scale factors |
| Spread 2 | P3–P4 | 13,077 × 9,138 px | 5,001 × 3,494 px | 256px, 5 scale factors |
| Spread 3 | P5–P6 | 13,100 × 9,158 px | 5,010 × 3,502 px | 256px, 5 scale factors |
| Spread 4 | P7–P8 | 13,081 × 9,165 px | 5,002 × 3,505 px | 256px, 5 scale factors |

All four spreads were extracted from the Wikimedia PDF at approximately 13,075 × 9,130 pixels and downscaled to approximately 5,000 × 3,500 pixels for tiling. All tile pyramids are served from GitHub Pages under `images/takvim-i-vekayi-1831*` subdirectories.

### 2.4 Bounding box coordinates

Page 1 has three regions; pages 2–8 each have two regions (right-hand column and left-hand column), following the gazette's consistent two-column layout. Bounding boxes for pages 1–2 were generated programmatically by image analysis. Bounding boxes for pages 3–8 were estimated from the page 1–2 proportions applied to the spread 2–4 canvas dimensions; these are approximate and may need fine-tuning against the source images.

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
- **RTL support incomplete** — Mirador 3 has an [open issue (#2843)](https://github.com/ProjectMirador/mirador/issues/2843) tracking RTL support. Partial configuration is possible via `theme: { direction: 'rtl' }`, but canvas navigation, sidebar positioning, and thumbnail ordering are not fully RTL-aware.

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

The final architecture is a single HTML file (`viewer.html`, ~345KB including all annotation data for 8 pages) containing:

- **OpenSeadragon v4.1.1** loaded from jsDelivr CDN, providing deep-zoom pan/zoom navigation over IIIF tile pyramids.
- **Embedded annotation JSON** (~270KB) containing per-spread, per-region, per-line, per-layer annotation data plus flowing summary translations for pages 1–5, with placeholder bounding boxes for pages 6–8.
- **Custom CSS** implementing a dark-themed split-screen layout with a collapsible sidebar, P1–P8 page navigation, and a thumbnail strip.
- **Custom JavaScript** managing spread switching, page selection, region selection, layer toggling, overlay positioning, sidebar resize, RTL reading order, and fullscreen behaviour.

The viewer reads its tile sources from IIIF Image API endpoints, switching between four tile pyramids as the user navigates between spreads. The annotation data uses a custom JSON format optimised for the viewer's layer-toggling requirements, structured as `SPREAD_DATA[spreadKey].regions[regionKey].lines[].{script, transliteration, literal, modern}`.

---

## 6. What Was Built: Single-Page Viewer

### 6.1 User interface (initial version)

The initial viewer presented a split-screen layout:

- **Left panel (sidebar):** Title, date, and credits header; five layer-toggle checkboxes (Original Script, Transliteration, Literal English, Modernised English, Summary Translation), each with a colour-coded swatch; three region navigation buttons; and a scrollable annotation panel showing line-by-line content for the selected region.
- **Right panel (viewer):** Deep-zoomable image with three clickable semi-transparent overlay rectangles marking the annotated regions. Includes a mini-map navigator (top-left) and standard zoom controls (top-right).

### 6.2 Interaction features (initial version)

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
├── viewer.html                          ← Bespoke annotation viewer (~306KB, self-contained)
├── index.html                           ← Landing page with Mirador instructions
├── README.md                            ← Repository documentation
├── docs/
│   └── IIIF_Annotation_Viewer_Memo_*    ← This memo (versioned)
├── images/
│   ├── takvim-i-vekayi-1831/            ← IIIF Level 0 tile pyramid (Spread 1, P1–P2)
│   ├── takvim-i-vekayi-1831-spread2/    ← IIIF Level 0 tile pyramid (Spread 2, P3–P4)
│   ├── takvim-i-vekayi-1831-spread3/    ← IIIF Level 0 tile pyramid (Spread 3, P5–P6)
│   └── takvim-i-vekayi-1831-spread4/    ← IIIF Level 0 tile pyramid (Spread 4, P7–P8)
├── manifests/
│   └── demo-manifest.json               ← IIIF Presentation API v3 manifest (viewingDirection: right-to-left)
└── annotations/
    └── annotations-layered.json         ← Structured annotation data (for reference/rebuild)
```

---

## 7. Iterative Refinements: Phase 1

The single-page viewer was built and refined across multiple commits, with each round of user testing driving specific improvements:

| Round | Changes |
|-------|---------|
| Initial build | Four-layer annotation viewer with OpenSeadragon, region overlays, dark theme, layer toggles |
| First refinement | Added right-click zoom out; moved credits from floating badge to inline sidebar header with expandable detail; added Summary Translation as fifth toggleable layer; removed "Open in Mirador" link from info bar; added sidebar collapse/expand toggle |
| Second refinement | Brightened credits text (inline `#666` → `#aaa`, strong `#999` → `#ddd`, detail `#555` → `#999`); added blank line suppression when only summary layer is active; fixed fullscreen to include sidebar (overriding OpenSeadragon's default fullscreen to use `.app` container) |
| Final fix | Added `.summary-section.hidden` CSS rule — the summary section was not responding to the layer toggle because only `.layer-line.hidden` had a `display: none` rule |

---

## 8. Commit History

### Phase 1: Single-page viewer (21 February 2026)

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

### Phase 1.5: Page 2 extension and bounding box refinement (21–22 February 2026)

| Commit | Message |
|--------|---------|
| `a1e19eb` | Fix Script tag to LH side for clean RTL flow |
| `ecca973` | Fix credits: separate visual capture and semantic processing skill files |
| `92cbf08` | Rewrite README for bespoke viewer; add project memo to docs/; fix credits skill file names |
| `efe3571` | Add Page 2 annotations: 105 lines across RH and LH columns |
| `50fa4d4` | Tabbed page nav, collapsible accordions, fix P2 bounding boxes and labels |
| `3f1233c` | Fix P2 boxes, add region indicator bar, fix overlay click navigation |
| `f4c0637` | Fix duplicate const declaration crashing viewer |
| `0e78d63` | Red active overlay, fine-tune bounding boxes, RTL tab order |
| `d07a2aa` | Align masthead and P1 RH right edges, narrow P1 RH column |

### Phase 2: Multi-page extension (22 February 2026)

| Commit | Message |
|--------|---------|
| `1d17a1c` | Add tile pyramids for spreads 2-4 (pages 3-8) |
| `23b8ea0` | Add multi-spread support for pages 1-8 with HTR for P3-P5 |
| `c93a8c7` | Fix tile paths: add missing 0/ rotation directory for spreads 2-4 |
| `b862944` | Redesign navigation: P1-P8 buttons, thumbnail strip, scroll fix |
| `f91c5ec` | Apply RTL reading order to page buttons and thumbnail strip |
| `fa87312` | Add breathing room to initial view and preload info.json files |

---

## 9. What Was Built: Multi-Page Extension

### 9.1 Overview

Between 21 and 22 February 2026, the viewer was extended from a single-page demonstration to a multi-page viewer covering all eight pages of Issue 1 across four two-page spreads. This work was carried out across two Cowork sessions and involved tile generation, annotation data integration, bug fixing, and a complete navigation redesign.

### 9.2 Tile pyramid generation

The Wikimedia source PDF contains four two-page spreads. Each spread was extracted as a separate image and converted into a IIIF Image API 2.1 Level 0 static tile pyramid using the Python `iiif_static` library within Claude Cowork. Spread 1 (P1–P2) had already been generated in Phase 1; spreads 2–4 were generated in Phase 2.

A bug in the tile generation for spreads 2–4 omitted the `0/` rotation directory from the IIIF path structure. The IIIF Image API requires the path format `{region}/{size}/{rotation}/{quality}.{format}`, where the rotation component (`0/` for no rotation) is a mandatory path segment even when no rotation is applied. Spread 1's tiles had the correct structure (`0,0,256,256/256,/0/default.jpg`); spreads 2–4 had the broken structure (`0,0,256,256/256,/default.jpg`). This caused OpenSeadragon to request URLs that returned 404 errors.

The fix involved a Python script that moved all 1,206 `default.jpg` files (402 per spread) into newly created `0/` subdirectories. The diagnosis required comparing the tile directory structures between the working spread and the broken spreads to identify the missing path component.

### 9.3 Spread-based architecture

The viewer uses a **spread-based architecture** rather than a page-based one, reflecting the physical structure of the source document. Each spread is a single high-resolution image containing two pages side by side. The viewer maintains four tile pyramids (one per spread) and switches between them using `viewer.open()` when the user navigates to a page on a different spread.

Key data structures:

- `SPREAD_DATA` — an object keyed by spread identifier (`spread1` through `spread4`), containing the IIIF image service URL, canvas dimensions, page list, and region definitions for each spread.
- `PAGE_SPREAD` — a mapping from page identifiers (`p1` through `p8`) to their parent spread, enabling transparent spread switching when a user clicks a page button.
- `PAGE_REGIONS` — a mapping from page identifiers to their region keys, used to populate the region buttons when the user selects a page.

### 9.4 Annotation data

Full HTR annotations (original script, transliteration, literal English, modernised English, and summary) were integrated for pages 1–5. Pages 3–5 annotations were parsed from the HTR markdown files in the `colin-yournamehere-htr` repository. Pages 6–8 have bounding boxes but no annotation data; selecting a region on these pages displays a placeholder message: "Image available but HTR annotations have not yet been processed for this page."

The total embedded annotation data is approximately 300KB, bringing the viewer HTML file to approximately 345KB. This follows the "embed all pages" architecture (Option A from the original plan in v2.1 Section 9.3), which avoids fetch latency when switching pages and eliminates any risk of CORS or network issues.

### 9.5 Navigation redesign

The initial multi-page navigation used a two-tier interface: four spread tabs (Sp.1–Sp.4) above two dynamic page tabs that changed when the user switched spreads. User testing found this "not intuitive" — the indirection of selecting a spread to see its pages added cognitive load without benefit.

The navigation was redesigned to a single row of eight page buttons (P1–P8) in the sidebar, with spread switching handled automatically behind the scenes. When the user clicks P5, the viewer detects that P5 belongs to spread 3, switches the tile source if necessary, and updates the overlays. The page buttons use a subtle background tint to indicate which pages share the currently loaded spread, providing visual context without requiring the user to think in terms of spreads.

A **thumbnail strip** was added below the main viewer, showing small preview images of all four spreads. This follows the navigation pattern used by Mirador and other IIIF viewers, where thumbnail images provide a visual overview of the document. Clicking a thumbnail switches to that spread and selects its first page. The thumbnail images are served from the IIIF image service's `full/{size}/0/default.jpg` endpoint at reduced resolution (156–312px wide).

### 9.6 Right-to-left reading order

The Takvîm-i Vekâyi is an Ottoman Turkish document written in Perso-Arabic script, which reads right-to-left. Within each two-page spread, the lower-numbered page is on the right (e.g., P1 is the right half of spread 1, P3 is the right half of spread 2). An Ottoman reader would begin at what a Western reader would consider the "back" of the document and read rightward, turning pages leftward to progress.

The IIIF Presentation API v3 supports this natively through the `viewingDirection` property. We added `"viewingDirection": "right-to-left"` to the manifest, and implemented RTL ordering in the bespoke viewer:

- **Page buttons** display P8 on the far left and P1 on the far right, so the reading start point is at the right edge.
- **Thumbnail strip** displays Spread 4 on the far left and Spread 1 on the far right.

This was informed by research into how IIIF viewers handle RTL manuscripts. The IIIF Cookbook includes a [dedicated recipe (0010)](https://iiif.io/api/cookbook/recipe/0010-book-2-viewing-direction/) for viewing direction. Institutions including the Bodleian Library (Oxford), Bibliothèque nationale de France, Princeton University Library, and the National Library of Israel all serve Arabic, Persian, Ottoman, and Hebrew manuscripts through IIIF with RTL viewing direction.

### 9.7 Performance optimisations

Two optimisations were added to improve the user experience when navigating between spreads:

**Info.json preloading.** On page load, the viewer fires `fetch()` requests for all four spreads' `info.json` files, even though only spread 1 is displayed initially. This warms both the browser cache and the GitHub Pages CDN edge cache. The browser cache benefit is per-user; the CDN cache benefit is shared across all users in the same geographic region, since GitHub Pages uses the Fastly CDN.

**Initial view padding.** OpenSeadragon's default behaviour fits the image to fill the viewport edge-to-edge. A 6% padding was added to the initial view so the image sits with a thin border of dark space, providing visual breathing room. This matches the aesthetic of the region-zoom view (which already had padding) and gives the user a clearer sense of the image boundaries.

### 9.8 Other fixes

**Annotation panel scroll-to-top.** When clicking a new region, the annotation panel now scrolls to the top. Previously, if the user had scrolled down within one region's annotations and then clicked a different region, the scroll position was preserved, leaving the new content half-way down — confusing because the user expects to start reading from the beginning of the new region's annotations.

**Placeholder text brightness.** The "not yet processed" placeholder message for pages 6–8 was brightened from `#555` to `#999` for better readability against the dark background.

**Stale selector cleanup.** The navigation redesign left stale references to removed CSS selectors (`.spread-tab`, `.page-tab`) and a removed function (`updatePageTabButtons()`) in the `selectRegion()` function. These were identified by a post-redesign grep and cleaned up before deployment.

---

## 10. The Multi-Page Plan vs. What Actually Happened

Version 2.1 of this memo (Section 9) laid out a plan for extending the viewer to multiple pages. This section compares the plan with what was actually built.

### 10.1 What the plan got right

The plan correctly anticipated:

- **Per-page tile pyramids** in separate subdirectories — this is exactly what was built (one per spread rather than per page, but the principle held).
- **OpenSeadragon's `viewer.open()` for tile source switching** — this worked as expected.
- **Existing region overlay and layer toggle code working unchanged** — confirmed; the annotation rendering and layer visibility logic required no modifications.
- **The architecture choice of embedded annotation data** (Option A) — at ~270KB total, embedding all pages was the right call; no fetch latency, no CORS issues.

### 10.2 What the plan didn't anticipate

- **Spread-based architecture.** The plan assumed per-page tile pyramids. In practice, the Wikimedia source PDF contains two-page spreads, making it natural to tile each spread as a single image. This introduced the spread ↔ page mapping layer.
- **Navigation design complexity.** The plan estimated "50–80 lines of new JavaScript" for a page selector. The actual navigation redesign — from two-tier spread+page tabs to P1–P8 buttons plus thumbnail strip — was substantially more involved and required two iterations.
- **IIIF path structure bug.** The plan did not anticipate the `0/` rotation directory issue, which caused spreads 2–4 to fail silently (404 errors on all tile requests).
- **RTL reading order.** The plan did not address the right-to-left viewing direction, which required research into IIIF standards and implementation in both the viewer and the manifest.
- **Process lessons.** The navigation redesign was delegated to a subagent (see Section 9.8 on stale selectors), which highlighted the importance of post-delegation verification — a broader lesson about collaborative AI-assisted development documented separately in `Cowork-Learnings/Agent_Subagent_Control_Memo_v1.0_22022026.md`.

---

## 11. A Dynamic Version: Assessment and Decision

### 11.1 What "dynamic" would mean

The current viewer is entirely static: HTR transcriptions are processed offline, annotation JSON and viewer HTML are generated in Claude Cowork sessions, and the output is pushed to GitHub Pages. A dynamic version would accept new document uploads, tile them automatically, ingest HTR output, structure annotations, and serve the viewer — without manually editing HTML or pushing to git.

### 11.2 What would need to be built

A dynamic system would require five components:

**Image processing pipeline.** An upload endpoint that accepts a document image, downscales it, generates IIIF Level 0 tiles, and stores them on disk. The Python `iiif_static` library already does the tiling; wrapping it in an API endpoint is straightforward.

**Annotation data store.** A backend that stores per-page, per-region, per-line annotation data. This could be as simple as JSON files on disk (matching the current format) or a lightweight database (SQLite) if search or editing functionality were desired.

**API layer.** A REST API (Flask or FastAPI in Python) serving tile `info.json` descriptors, annotation data, and page metadata to the viewer. The viewer's JavaScript would fetch from this API instead of reading embedded data.

**Viewer modifications.** The frontend viewer would need only modest changes: replace the embedded `SPREAD_DATA` object with `fetch()` calls to the API, and add a document selector that queries available documents dynamically.

**Admin interface.** A simple web form for uploading images and associating HTR markdown files with pages and regions. This is the most labour-intensive component, because it requires UI for managing the document → page → region → annotation hierarchy.

### 11.3 Estimated effort

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

### 11.4 Hosting options

**Local workstation.** Python (Flask/FastAPI) serving both tiles and the API on `localhost`. Functional for personal use but not accessible to collaborators without tunnelling (ngrok or similar), which is fragile and not suitable for a demo.

**Hosted server.** A small VPS (DigitalOcean, Hetzner, or similar — £5–10/month) running the same Python stack behind nginx, with a domain name and HTTPS via Let's Encrypt. This would be accessible to anyone with the URL.

### 11.5 Cowork vs Claude Code

The static viewer was well suited to Claude Cowork: iterative design, file generation, research, and pushing changes through git. A dynamic backend project — with its multi-file codebase, local server processes, test cycles, and deployment scripts — would be better suited to Claude Code, which runs directly in the development environment with full filesystem access and the ability to run and test servers locally.

### 11.6 Decision

The dynamic version is **parked**. The technical assessment confirms it is feasible and within our combined capabilities, but the effort is better invested after the Collaboratory session, when feedback from participants and from the broader IIIF community — including Rainer Simon and others working on annotation tooling — can inform the design. The dynamic version may also benefit from emerging developments in IIIF viewer annotation support, particularly in Glycerine Viewer, which is under active development.

---

## 12. Next Steps

### Immediate (before 24 February 2026)

- **Fine-tune bounding boxes for P3–P8** — current boxes are estimated from P1–P2 proportions; visual comparison against the source images may reveal adjustments needed.
- **Custom domain** — configure a subdomain (e.g. `iiif.ai-and-history.org`) pointing to the GitHub Pages site, for cleaner sharing URLs and Open Graph previews.
- **Version stamp** — add version number and build metadata to the sidebar header.

### Short-term (before 31 March 2026)

- **HTR for pages 6–8** — run the two-stage pipeline on the remaining three pages to complete the full issue.
- **Line-level bounding boxes** — use Gemini 3 Pro Preview to generate per-line bounding box coordinates, enabling click-on-image-to-highlight-annotation interaction (Phase 2, targeted for the 31 March session).
- **NER tables** — display Named Entity Recognition output (persons, places, dates, vessels) as a toggleable panel, with mockup needed for UI/UX review.
- **Second document type** — process HCA 13/71 f.1r (a mid-1650s English maritime deposition from the MarineLives corpus) to demonstrate the same pipeline on a second script and language.

### Medium-term (mid-2026)

- **Dynamic version** — revisit the dynamic architecture (Section 11) in light of Collaboratory feedback and IIIF community developments. Discuss with Rainer Simon and others. If pursued, build in Claude Code rather than Claude Cowork.
- **Video walkthrough** — record a demo walkthrough for asynchronous sharing with collaborators.
- **Summary landing page** — create a page describing the full pipeline from source image to annotated viewer.
