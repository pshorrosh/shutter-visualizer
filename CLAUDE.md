# ShutterViz — CLAUDE.md

Customer-facing shutter visualizer for ShuttersDirectUSA. Customers upload a
window photo, drag a 4-corner overlay to fit their window (dragging a single
corner skews it for perspective), then see a live shutter preview with
pricing, a before/after toggle, and a quote-request flow that posts to
GoHighLevel and sends an email confirmation.

**Single file. No build step.** ⚠️ Not fully zero-dependency: `index.html:7`
loads the EmailJS SDK from a CDN — see **Known TODOs** below.

---

## Running the app

Open `index.html` directly in any modern browser — no server, no install, no npm.

```
open index.html
```

---

## Architecture

Everything lives in one file: `index.html` (999 lines).

| Block | Lines |
|---|---|
| CSS (`<style>`) | 8–134 |
| HTML structure | 136–349 |
| JavaScript (`<script>`) | 350–997 |

The JS is organized with section-marker comments in the form `─── Section Name ───`. Never remove or rename these markers — they are the navigation system.

---

## Key sections and locations

| Section | Lines | Notes |
|---|---|---|
| `─── DOM refs ───` | 351 | Canvas/ctx, upload elements, width/height/select inputs, quote elements, modal |
| `─── State ───` | 373 | `img`, `overlay` (the 4-corner quad), `showShutters`, `action`/`dragStart` (corner-drag state), `shutterColor`, `numPanels`, `EMAIL_TO`/`EMAIL_CC` |
| `─── Upload ───` | 387 | Drag-and-drop + click upload, `loadFile()` downsamples to max 1400px, then centers the starting overlay via `initOverlay()` |
| `─── Overlay (free-form quad) ───` | 425 | `overlayAspect()`, `initOverlay()`, `refreshOverlayAspect()`, `pointInQuad()`, `hitTest()` — the draggable 4-corner quad; appears automatically on upload, no separate "mark corners" step |
| `─── Coordinate helper ───` | 472 | `canvasCoords()` — client px → canvas px |
| `─── Pointer events ───` | 478 | Touch + mouse handlers for dragging the whole overlay or a single corner |
| `─── Before / After toggle ───` | 523 | `btnAfter`/`btnBefore` click handlers — a simple show/hide toggle, not a drag divider |
| `─── Quad geometry helpers ───` | 537 | `lerp2`, `qpt`, `qpath`, `qvgrad`, `expandQuad` — bilinear interpolation across the quad, used by every panel-drawing function |
| `─── Draw ───` | 561 | `drawCanvas()` — draws the photo, clips to the quad (expanded for outside mount), calls the panel drawer per panel, then draws the outline + corner handles |
| `─── Shutter rendering (perspective-aware quad versions) ───` | 607 | `lv()` (lighten/darken a hex color), `drawPlantationPanel()`, `drawArchPanel()`, `drawSunburstPanel()` — draw straight onto the visible canvas via the quad helpers; no off-screen texture or cache |
| `─── Panel count buttons ───` | 746 | `.panel-btn` click handlers set `numPanels`, re-validate, redraw |
| `─── Panel width limit validation ───` | 757 | `maxPanelWidth()` (24″ for louvers ≤2.5″, else 30″), `checkPanelWidthLimit()` — shows `#panelWarn` when exceeded |
| `─── Preferences → redraw ───` | 782 | Color-swatch clicks, shutter/mount/louver `change` listeners — all trigger `drawCanvas()` |
| `─── Measurement guide ───` | 792 | `GUIDE` object + `updateMeasureGuide()` — swaps inside/outside "How to Measure" instructions |
| `─── Measurement inputs ───` | 804 | Width/height `input` listeners — clear errors, re-check panel width, refresh overlay aspect, live-update the quote if visible |
| `─── Estimate ───` | 815 | `estimateBtn` handler validates inputs + panel width, then calls `updateQuote()` ($29/sq ft vs $49/sq ft retail) |
| `─── Reset ───` | 847 | `doReset()` clears photo/overlay/inputs/quote section; wired to both `resetBtn` and `startOverBtn` |
| `─── Quote modal ───` | 863 | `quoteBtn` opens the modal with a summary + price; `closeModal()` on × or overlay click |
| `─── GoHighLevel webhook ───` | 881 | `GHL_WEBHOOK` — live LeadConnector URL |
| `─── EmailJS config ───` | 884 | `EMAILJS_PUBLIC_KEY`/`EMAILJS_SERVICE_ID`/`EMAILJS_TEMPLATE_ID` |
| `─── Submit quote → auto-send via EmailJS ───` | 889 | Validates contact fields, posts the lead to `GHL_WEBHOOK`, sends the EmailJS confirmation, closes the modal, shows a toast |
| `─── Init ───` | 972 | `emailjs.init()`, pre-computes the default quote |
| `─── Toast ───` | 976 | `toast(msg)` — bottom notification |
| `─── Resize ───` | 983 | Debounced (120ms) canvas resize that rescales the overlay's corner coordinates proportionally |

---

## Common tasks

**Change the price per sq ft** → computed independently in three places, all using `* 29` (cost) / `* 49` (retail): `updateQuote()` (line 837), the quote-modal handler (line 866), and the submit handler (line 900). Change all three together — there's no single source of truth here.

**Change available colors** → `.color-swatch` divs, HTML lines 215–220 (`data-color` hex values)

**Change shutter types** → `<select id="shutterSel">`, HTML lines 186–190

**Change louver sizes** → `<select id="louverSel">`, HTML lines 205–209

**Change panel-count options** → `.panel-btn` buttons inside `#panelBtns`, HTML lines 196–199 (buttons, not a `<select>`)

**Change the panel-width limit** → `maxPanelWidth()` (~line 758): 24″ for louvers ≤2.5″, 30″ otherwise

**Tweak how shutters look** → `drawPlantationPanel()` (615), `drawArchPanel()` (691), `drawSunburstPanel()` (711) — draw directly onto the visible canvas using the quad helpers (`qpt`/`qpath`/`qvgrad`, line 537)

**Change the inside/outside measuring instructions** → `GUIDE` object (~line 793)

**GHL webhook / EmailJS notification** → both live, wired in the submit-quote handler (~line 890). `GHL_WEBHOOK` (~line 882) posts the full lead payload (name, contact info, dimensions, shutter type/color/mount, pricing) to GoHighLevel; `EMAILJS_*` config (~line 885) plus `EMAIL_TO`/`EMAIL_CC` (~line 384) send a parallel confirmation email. Change recipients at `EMAIL_TO`/`EMAIL_CC`, not in the handler itself.

**Edit the "what happens next" copy / trust badges** → trust badges at HTML lines 290–294, the 5-step process list + install phone number at lines 325–334

**Responsive layout breakpoint** → CSS line 20 (`@media(max-width:820px)` → single column)

---

## Code style

- **Match the existing style exactly.** Section-marker comments (`─── Name ───`), camelCase variables, descriptive names, the same gradient/shading idiom used by the shutter draw functions.
- No build tooling — the file must stay a single deployable `.html`.
- Do not break mobile/touch support. All pointer interactions use both mouse and touch event handlers.
- Prefer editing in-place. Do not restructure the file layout or rename sections.

---

## Known TODOs

- **Zero-dependency claim is currently false**: `index.html:7` loads the EmailJS SDK from `cdn.jsdelivr.net`, which the quote-confirmation email (Known Sections → EmailJS config / Submit quote) depends on. Don't remove that `<script>` tag as a "zero-dependency cleanup" — that would break lead-confirmation email. If true zero-dependency matters more than the EmailJS confirmation email, that's a product decision for the repo owner, not something to resolve unilaterally.
- **Deployment**: Vercel is already connected via GitHub integration and auto-deploys previews + production on every push (no `vercel.json` committed — it's configured on the Vercel side). Don't add a separate GitHub Pages/Netlify config, or commit a `vercel.json`, without asking first.

---

## Testing

See [`docs/testing.md`](docs/testing.md) for the manual QA checklist — read it before merging any change that touches drawing, pricing, the quote flow, or layout.

---

## Context audit

For periodically auditing everything that loads into an agent's context at
session start (this file, skills, etc.) and trimming what's front-loaded but
rarely relevant — run the `context-audit` skill (`.claude/skills/context-audit/`).
Don't restate that procedure here; it has one home.
