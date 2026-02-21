# IIIF Collaborative Annotation Demo: Planning Memo
## Liiive Deployment for AI + History Collaboratory

---

## Document Version Control

| Version | Date | Author | Summary of Changes |
|---------|------|--------|-------------------|
| v1.0 | 2026-02-21 | Claude Opus 4.6/Colin | Initial memo: captures full planning conversation covering goal, options analysis, deployment decisions, and implementation plan |
| v1.1 | 2026-02-21 | Claude Opus 4.6/Colin | Two-phase timeline established: Phase 1 (IIIF infrastructure demo, 24 February) and Phase 2 (liiive collaborative annotation demo, 31 March); updated executive summary, use case, and implementation plan; added Phase Timeline section; resolved open questions on scheduling |

**Current Version:** v1.1  
**Last Updated:** 2026-02-21  
**Status:** Implementation-ready briefing document

---

**Prepared by:** Claude Opus 4.6  
**Prepared for:** Colin Greenstreet  
**Purpose:** Document the planning analysis for deploying a IIIF collaborative annotation demo using liiive; serve as briefing document for Claude Cowork to execute the static tile generation and manifest creation

---

## 1. Executive Summary

This memo documents a planning conversation on 21 February 2026 regarding the deployment of **liiive** — an open-source, real-time collaborative IIIF annotation tool — as a demonstration tool for the AI + History Collaboratory.

The project follows a **two-phase timeline**:

- **Phase 1 (24 February 2026):** Demo the IIIF infrastructure layer — two manuscript images served as deep-zoomable IIIF resources from GitHub Pages, displayed in a standard IIIF viewer (Mirador or Universal Viewer). This demonstrates that the Collaboratory can host and serve IIIF content independently.
- **Phase 2 (31 March 2026):** Demo the collaborative annotation layer — the same IIIF images loaded into liiive.now, demonstrating real-time multi-user annotation tools, live cursors, and region-based markup of manuscript content.

The agreed solution is a **lightweight static IIIF deployment on GitHub Pages**, hosted under the ai-and-history-collaboratory GitHub organisation. Two test images will be prepared: a handwritten English legal deposition (HCA 13/71 f.1r) and the first page of the Ottoman Government Gazette (*Takvim-i Vekayi*, 1831).

The repository has been named **`iiif-demo`** and will be created under the ai-and-history-collaboratory GitHub organisation. Phase 1 preparation takes place over the weekend of 22–23 February, with testing on Monday 23 February ahead of the Collaboratory Zoom session on Tuesday 24 February.

---

## 2. Background: What Is Liiive?

Liiive ([https://github.com/rainersimon/liiive](https://github.com/rainersimon/liiive)) is an open-source, browser-based tool for real-time collaborative annotation of IIIF images. Key capabilities include:

- **Real-time collaboration:** Multiple users view and annotate simultaneously with live cursors and synchronised annotations
- **Annotation tools:** Rectangle, polygon, circle/ellipse, path drawing, and a "smart scissors" magnetic lasso tool
- **IIIF-native:** Works with any IIIF Presentation API manifest; exports annotations in standard IIIF format
- **Room-based sessions:** Supports both temporary (ad-hoc) and persistent collaboration rooms
- **Derivative manifests:** Permanent rooms publish a IIIF v3 manifest at a stable URL, viewable in external IIIF viewers (Mirador, Universal Viewer, Theseus)

The repository has been forked to the ai-and-history-collaboratory GitHub organisation.

**Licence:** MIT (Supabase backend components under Apache 2.0)

---

## 3. Use Case

**Primary purpose:** Screen-share demonstrations at AI + History Collaboratory Zoom sessions, delivered in two phases.

### Phase 1: IIIF Infrastructure Demo (24 February 2026)

**What the audience sees:** Two manuscript images — one handwritten English, one printed Ottoman Turkish — displayed in a standard IIIF viewer (Mirador or Universal Viewer) with deep zoom, pan, and metadata. The images are served from the Collaboratory's own GitHub Pages infrastructure.

**What this demonstrates:** The Collaboratory can independently host and serve IIIF-compliant image resources without depending on institutional IIIF servers. The interoperability of the IIIF standard means these images can be consumed by any IIIF-compatible viewer or tool.

**Preparation timeline:**
- Weekend 22–23 February: Generate static tiles, create manifest, push to GitHub, enable Pages
- Monday 23 February: Test the full chain (GitHub Pages → IIIF viewer) and troubleshoot
- Tuesday 24 February: Screen-share demo at Collaboratory Zoom session

### Phase 2: Collaborative Annotation Demo (31 March 2026)

**What the audience sees:** The same two IIIF images loaded into liiive.now, with live demonstration of collaborative annotation — cursors, rectangle/polygon region selection, annotation comments, real-time synchronisation.

**What this demonstrates:** How IIIF-based collaborative annotation could support HTR validation workflows. Multiple scholars could annotate regions of a manuscript where HTR output requires expert review, with annotations exportable in standard IIIF format.

**Strategic connection:** The Takvim-i Vekayi image is particularly well-chosen given Özgür Türesay's specialism in the Ottoman Government Gazette and 19th-century political semantics. Demonstrating collaborative annotation on this specific document connects the IIIF tooling directly to the Ottoman HTR methodology and specialist engagement strategy.

**Dependency:** Phase 2 requires Phase 1 infrastructure to be working. The five-week gap between phases provides ample time to resolve any issues encountered during Phase 1 deployment.

---

## 4. Deployment Options Analysed

### 4.1 Public Hosted Service (liiive.now)

**Description:** Use the existing public hosted service at [https://liiive.now](https://liiive.now). No installation required. Paste a IIIF manifest URL, get a shared room, collaborate instantly.

**Pros:** Zero setup time; identical functionality to self-hosted version; suitable for screen-share demo.

**Cons:** No control over infrastructure; dependent on external service availability.

**Assessment:** Viable for the demo use case. The audience will not know or care whether the annotation tool is self-hosted.

### 4.2 Self-Hosted Development Deployment (Local Machine)

**Description:** Run the full liiive stack locally using Docker. The Supabase backend, WebSocket server, and admin UI run in Docker containers; the frontend runs via Node.js with hot reloading.

**Prerequisites:** Docker 24.0+ with Docker Compose 2.x; Node.js 20+ with npm.

**Pros:** Full control; no external dependencies during demo.

**Cons:** First-time setup of an unfamiliar Docker application carries risk of unanticipated issues; only accessible on localhost (participants cannot interact directly unless screen-sharing or using a tunnelling service like ngrok).

**Assessment:** Unnecessary complexity for a screen-share demo. Worth exploring for March 31st as a learning exercise, but not required for the demo itself.

### 4.3 Self-Hosted Production Deployment

**Description:** Fully containerised deployment on a Linux server with Traefik reverse proxy, requiring a registered domain and multiple subdomains (app, WebSocket, API, admin UI).

**Prerequisites:** Linux production server; registered domain; DNS configuration for subdomains.

**Pros:** Full production capability; publicly accessible for true collaborative sessions.

**Cons:** Significant infrastructure project; overkill for a demo.

**Assessment:** Not appropriate for current use case. Could be revisited if the Collaboratory wants persistent collaborative annotation infrastructure.

### 4.4 Cloud-Hosted Deployment (HuggingFace / Google Cloud)

**Description:** Host the liiive stack on a cloud platform.

**HuggingFace Spaces:** Not well-suited. Liiive's architecture requires multiple coordinated services (Supabase backend, WebSocket server, frontend), while HuggingFace Spaces is designed for single-container applications (ML demos, Gradio interfaces). Multi-service Docker Compose orchestration is not its intended use case.

**Google Cloud (Cloud Run or VM):** A more natural fit for multi-container applications, but introduces cost and configuration overhead.

**Assessment:** Worth exploring as a longer-term hosting option after the initial demo, particularly Google Cloud. Not necessary for the immediate use case.

---

## 5. The Image Hosting Question

### 5.1 How IIIF Works

A IIIF manifest is a JSON-LD metadata document. It does **not** contain images. It contains URLs pointing to images served by a IIIF Image API endpoint. When liiive loads a manifest, it follows those URLs to fetch image tiles at the required zoom level.

Creating a manifest is therefore the easy part. The harder question is: **where are the images served from?**

### 5.2 Options for Image Hosting

**Option A: Reference existing IIIF servers.** Many archives expose IIIF endpoints. If the required images are already available via IIIF, the manifest simply references their URLs. No image hosting required.

**Status:** Neither of the two selected images is currently available on a IIIF server:
- HCA 13/71 f.1r is not served via IIIF by The National Archives (Kew)
- The Takvim-i Vekayi first page is not available via IIIF from any identified institutional collection

**Option B: Self-host with a IIIF Image Server.** Deploy a IIIF image server (e.g., Cantaloupe, IIPImage). This is a separate infrastructure project and is explicitly listed in the project portfolio as something the initiative is *not* currently building.

**Option C: Static tile generation on GitHub Pages.** Pre-generate a tile pyramid from each image, host the tiles as static files on GitHub Pages, and write a IIIF manifest pointing to those static URLs. This requires no running server — just files served from a CDN.

### 5.3 Chosen Solution: Option C — Static Tiles on GitHub Pages

This is the lightest-weight approach and keeps everything within infrastructure the Collaboratory already controls.

---

## 6. Chosen Architecture

```
┌─────────────────────────────────────┐
│  ai-and-history-collaboratory/      │
│  iiif-demo (GitHub Repository)      │
│                                     │
│  ├── images/                        │
│  │   ├── hca-13-71-f1r/            │
│  │   │   ├── info.json             │  ← IIIF Image API response
│  │   │   └── [tile directories]    │  ← Pre-generated tile pyramid
│  │   └── takvim-i-vekayi-1831/     │
│  │       ├── info.json             │
│  │       └── [tile directories]    │
│  ├── manifests/                     │
│  │   └── demo-manifest.json        │  ← IIIF Presentation API v3 manifest
│  └── index.html                    │  ← Optional landing page
│                                     │
│  GitHub Pages enabled               │
│  URL: https://ai-and-history-       │
│       collaboratory.github.io/      │
│       iiif-demo/                    │
└─────────────────────────────────────┘
            │
            │  Manifest URL
            ▼
┌─────────────────────────────────────┐
│  liiive.now                         │
│  (Public hosted annotation service) │
│                                     │
│  Paste manifest URL → Go liiive     │
│  → Shared annotation room           │
└─────────────────────────────────────┘
```

### 6.1 How It Works

1. **Source images** (JPEG or PNG, out of copyright) are processed into IIIF-compatible static tile pyramids
2. **Tiles and metadata** are pushed to the `iiif-demo` GitHub repository
3. **GitHub Pages** serves the tiles as static files at a public URL
4. **A IIIF Presentation API v3 manifest** in the same repository references the tile URLs
5. **The manifest URL is pasted into liiive.now**, which loads the images and provides the collaborative annotation environment

### 6.2 Test Images

| Image | Description | Script | Type | Copyright Status |
|-------|-------------|--------|------|-----------------|
| HCA 13/71 f.1r | High Court of Admiralty deposition, first folio recto | English (secretary hand) | Handwritten | Out of copyright |
| Takvim-i Vekayi No. 1 (1831) | First page of the first issue of the Ottoman Government Gazette | Ottoman Turkish | Printed | Out of copyright |

### 6.3 GitHub Repository

- **Organisation:** ai-and-history-collaboratory
- **Repository name:** `iiif-demo`
- **GitHub Pages:** Must be enabled on the repository (Settings → Pages → Source: deploy from branch, select `main` and `/ (root)`)

---

## 7. Implementation Plan: Phase 1 (Weekend 22–23 February)

### 7.0 Prerequisite: GitHub Repository Setup

Before tile generation begins, Colin must:

1. **Create the `iiif-demo` repository** under the ai-and-history-collaboratory GitHub organisation
2. **Enable GitHub Pages** (Settings → Pages → Source: deploy from branch, select `main` and `/ (root)`)
3. **Confirm the base URL** will be: `https://ai-and-history-collaboratory.github.io/iiif-demo/`

This is critical because the `info.json` files and manifest must contain the final GitHub Pages URL where tiles will be served. Cowork needs this URL confirmed before generating the files.

### 7.1 Task Summary

Generate IIIF-compatible static tile pyramids for two manuscript images and create a IIIF Presentation API v3 manifest referencing those tiles. Output a complete folder structure ready to push to GitHub.

### 7.2 Inputs

Colin will provide two source images (available in JPEG and PNG formats):

1. **HCA 13/71 f.1r** — handwritten English legal deposition
2. **Takvim-i Vekayi first page** — printed Ottoman Turkish gazette (1831)

PNG versions are preferred as source material (lossless), but JPEG is acceptable if file sizes are prohibitive.

### 7.3 Tile Generation Process

**Tool:** Use Python with the `iiif_static` library (for images under ~10,000 pixels on the long edge) or `libvips`/`pyvips` (for larger images — faster and more memory-efficient).

**Install:**
```bash
pip install iiif_static
# or for large images:
pip install pyvips
```

**IIIF Image API tile structure:**

Each image must be processed into a directory of tiles following the IIIF Image API 3.0 URL structure:
```
{identifier}/{region}/{size}/{rotation}/{quality}.{format}
```

The tile generator creates:
- Multiple zoom levels (a tile pyramid)
- An `info.json` file containing image dimensions and tile size metadata
- Tile directories organised by the IIIF URL pattern

**Critical:** The `info.json` file must reference the correct base URL where the tiles will be served from GitHub Pages. This URL will be:
```
https://ai-and-history-collaboratory.github.io/iiif-demo/images/{identifier}
```

### 7.4 Manifest Creation

Create a **IIIF Presentation API v3** manifest (JSON-LD) containing:

- Two canvases (one per image)
- Each canvas referencing the corresponding image's `info.json` via its GitHub Pages URL
- Appropriate metadata labels (title, description, source attribution)

The manifest must conform to the IIIF Presentation API 3.0 specification. The manifest URL will be:
```
https://ai-and-history-collaboratory.github.io/iiif-demo/manifests/demo-manifest.json
```

### 7.5 Output Structure

```
iiif-demo/
├── images/
│   ├── hca-13-71-f1r/
│   │   ├── info.json
│   │   └── [tile directories following IIIF Image API URL pattern]
│   └── takvim-i-vekayi-1831/
│       ├── info.json
│       └── [tile directories following IIIF Image API URL pattern]
├── manifests/
│   └── demo-manifest.json
├── index.html          (simple landing page with links to manifest)
└── README.md           (repository documentation)
```

### 7.6 Validation Steps (Phase 1)

1. **Check `info.json` files** point to the correct GitHub Pages base URL
2. **Validate the manifest** against the IIIF Presentation API 3.0 specification (use [https://presentation-validator.iiif.io/](https://presentation-validator.iiif.io/))
3. **Test in a IIIF viewer** — load the manifest URL in Mirador ([https://projectmirador.org/demo](https://projectmirador.org/demo)) or Universal Viewer to confirm deep zoom, pan, and metadata display work correctly. This is the Phase 1 demo deliverable.
4. **Test in liiive.now** by pasting the manifest URL — this validates readiness for the Phase 2 demo on 31 March but is not required for the 24 February session

### 7.7 Constraints

- **GitHub repository size:** Soft limit ~1GB; individual files max 100MB. A single high-resolution manuscript page typically generates 10–100MB of tiles depending on resolution. Two images should be well within limits.
- **GitHub Pages:** Serves static files only; no server-side processing. This is fine for pre-generated static tiles.
- **CORS:** GitHub Pages serves with appropriate CORS headers for cross-origin requests, which liiive requires.

---

## 7A. Implementation Outline: Phase 2 (March — Preparation for 31 March)

Phase 2 builds on the Phase 1 infrastructure. No additional tile generation or manifest work is required unless new images are added. The preparation is primarily about **demo design and testing**.

### 7A.1 Tasks

1. **Test the manifest in liiive.now** — confirm that the Phase 1 manifest loads correctly and that annotation tools function as expected on both images
2. **Create a demo room** — liiive supports both ad-hoc temporary rooms and persistent rooms. Decide which is appropriate for the Collaboratory session
3. **Plan annotation scenarios** — pre-identify regions of each image to annotate during the demo, with prepared commentary connecting to HTR methodology (e.g., annotating the masthead region of the Takvim-i Vekayi and discussing what an HTR pipeline would extract)
4. **Test annotation export** — demonstrate that annotations can be downloaded in standard IIIF format, reinforcing the interoperability message
5. **(Optional) Evaluate liiive self-hosting** — if there is interest in a Collaboratory-hosted instance, use March to explore the Docker-based development deployment on Colin's machine

---

## 8. Open Questions

### Phase 1 (Immediate — Weekend 22–23 February)

1. **Image resolution:** What are the pixel dimensions and file sizes of the two source images (PNG versions)? This determines which tiling tool is optimal and whether the output will fit comfortably within GitHub's limits.

2. **IIIF viewer selection for Phase 1 demo:** Mirador and Universal Viewer are the two standard options for displaying a IIIF manifest. Mirador offers a split-pane workspace view; Universal Viewer is simpler and more focused. Which better suits the Collaboratory audience?

3. **Future expansion:** If the demo is successful, should the `iiif-demo` repository be designed to accommodate additional images over time, or is this a fixed two-image demonstration?

### Phase 2 (March — Preparation for 31 March)

4. **Liiive self-hosting evaluation:** Is there value in standing up a local development deployment of liiive (Docker-based) during March, or is the hosted liiive.now service sufficient for the demo and foreseeable use?

5. **Cloud hosting follow-up:** The conversation identified Google Cloud as a more suitable platform than HuggingFace for hosting the full liiive stack if a self-hosted option is pursued. This is a separate project from the static tile demo.

6. **Annotation workflow design:** For the March 31st demo, it would be effective to pre-plan which regions of the Takvim-i Vekayi page to annotate and what kinds of annotations to demonstrate (region identification, HTR output commentary, palaeographic notes). This connects the tool demo to the Collaboratory's substantive research agenda.

---

## 9. Relationship to Project Portfolio

This demo addresses an area explicitly noted in the [HTR Project Portfolio Overview (v6, 09 January 2026)](HTR_Project_Portfolio_Overview_09012026_v6.md) as outside current scope: "An AtoM instance or IIIF server for hosting images." The static tile approach on GitHub Pages is deliberately minimal — it demonstrates IIIF capability without committing to image hosting infrastructure. The images remain the researcher's own files; GitHub Pages simply makes them accessible via IIIF URLs.

The two-phase approach mirrors the project's broader methodology of building incrementally with empirical validation at each step. Phase 1 establishes the infrastructure; Phase 2 demonstrates application. If the Phase 1 infrastructure proves unreliable or the Phase 2 annotation workflow proves uncompelling, each phase can be evaluated independently.

The liiive annotation layer has potential future relevance for collaborative HTR validation sessions — multiple scholars annotating regions of a manuscript where HTR output requires expert review, with annotations exportable in standard IIIF format for integration with other tools. This is not the immediate use case but connects to the broader expert witness / specialist co-developer engagement model described in the [Ottoman HTR Scholar Outreach Strategy (v1.4, 07 January 2026)](Ottoman_HTR_Scholar_Outreach_Strategy_v1_4_07012026.md).

---

## Bibliography

"liiive: Collaborative Annotation for IIIF Images." GitHub, [https://github.com/rainersimon/liiive](https://github.com/rainersimon/liiive). Accessed 21 Feb. 2026.

"IIIF Image API 3.0." International Image Interoperability Framework, [https://iiif.io/api/image/3.0/](https://iiif.io/api/image/3.0/). Accessed 21 Feb. 2026.

"IIIF Presentation API 3.0." International Image Interoperability Framework, [https://iiif.io/api/presentation/3.0/](https://iiif.io/api/presentation/3.0/). Accessed 21 Feb. 2026.

---

*Memo prepared 21 February 2026*  
*For AI + History Collaboratory project reference and Claude Cowork briefing*
