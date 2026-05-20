# ShutterViz — CLAUDE.md

Customer-facing shutter visualizer for ShuttersDirectUSA. Customers upload a window photo, mark 4 corners, then see photorealistic, perspective-correct previews of shutters with live pricing, before/after compare, save/share, and printable quotes.

**Single file. Zero dependencies. No build step.**

---

## Running the app

Open `index.html` directly in any modern browser — no server, no install, no npm.

```
open index.html
```

---

## Architecture

Everything lives in one file: `index.html`

| Block | Lines |
|---|---|
| CSS (`<style>`) | 7–148 |
| HTML structure | 150–369 |
| JavaScript (`<script>`) | 371–1444 |

The JS is organized with section-marker comments in the form `─── Section Name ───`. Never remove or rename these markers — they are the navigation system.

---

## Key sections and locations

| Section | Lines | Notes |
|---|---|---|
| `─── DOM refs ───` | 372 | All element references, including modal and preview-action refs |
| `─── State ───` | 410 | `img`, `corners`, `vizMode`, `shutterColor`, `sliderX`, `sliderActive`, `GHL_WEBHOOK`, `STORAGE_KEY` |
| `─── Storage ───` | 436 | `saveToStorage()`, `loadFromStorage()`, `hasStored()` — corners stored normalised |
| `─── Upload ───` | 495 | File drop + `loadFile()` — downsamples to max 1400px (line 507). **Bug fix:** display:block runs before sizeCanvas via rAF |
| `─── Coordinate helpers ───` | 548 | `canvasCoords`, `dist`, `nearestCorner`, `nearSlider`, `lerpPt` |
| `─── Pointer events ───` | 571 | Mouse + touch unified into `handleStart/Move/End`; slider-drag wins over corner placement |
| `─── Corner status UI ───` | 635 | Dot indicators, hint text, **undo button** |
| `─── Dimension estimation ───` | 654 | Heuristic px-to-inch mapping; calibration table at lines 666–671 |
| `─── Perspective warp ───` | 680 | `drawTexturedTriangle()` (line 685), `warpToQuad()` (line 707) — bilinear-interpolated 12×12 triangle mesh, no libraries |
| `─── Off-screen shutter render ───` | 742 | `renderShuttersOffscreen()` — rebuilds shutter texture when `shutterDirty=true`. `PIXEL_PER_INCH` at line 752 |
| `─── Draw ───` | 780 | `drawCanvas()` → `drawMarkup()` / `drawPreview()` → `warpToQuad()`; `drawDimensionLabels()` overlays W/H |
| `─── Before/After slider ───` | 911 | `drawSlider()` and toggle-button handler; default ON |
| `─── Shutter rendering ───` | 964 | `lv()` (969), `drawPlantationPanel()` (975), `drawArchPanel()` (1065), `drawSunburstPanel()` (1098) — all draw onto the off-screen canvas |
| `─── Mode toggle ───` | 1144 | Markup / Preview tab buttons; shows the Save/Share/Compare action bar in preview |
| `─── Color picker ───` | 1160 | Swatch selection → `shutterColor`, marks shutter dirty |
| `─── Live quote ───` | 1172 | `updateQuote()` — $29/sq ft at line 1177 |
| `─── Save & Share ───` | 1197 | `buildExportCanvas()` (with branding strip), save-as-PNG, Web Share API with download fallback |
| `─── Print quote ───` | 1276 | `fillPrintSheet()` populates the hidden `#printSheet` and `window.print()` |
| `─── Contact modal ───` | 1307 | `openContactModal()`, validates name + email, posts GHL webhook with contact info |
| `─── Reset ───` | 1373 | `doReset()` — full restart, clears localStorage; `Clear Markers` keeps the photo |
| `─── Toast helper ───` | 1403 | `toast(msg)` — 3.2s bottom notification |
| `─── Window resize ───` | 1410 | Debounced canvas resize (120ms); **scales corners proportionally** to new canvas size |
| `─── Restore-on-load prompt ───` | 1427 | "Continue your last visualization?" banner if localStorage has state |
| `─── Init ───` | 1440 | Initial calls to `updateCornerStatus()`, `updateQuote()` |

---

## Common tasks

**Change the price per sq ft** → line 1177 (`sqft * 29` in `updateQuote()`). The contact-modal submission and print sheet derive from `updateQuote`, so this is the single source of truth.

**Change available colors** → HTML lines 274–279 (color swatches with `data-color` hex values)

**Change shutter types** → HTML lines 255–259 (the shutter `<select>`)

**Change louver sizes** → HTML lines 264–268

**Change panel-count options** → HTML lines 238–244

**Tweak how shutters look** → `drawPlantationPanel` (975), `drawArchPanel` (1065), `drawSunburstPanel` (1098). They render onto an off-screen canvas (size scales with `PIXEL_PER_INCH` at line 752); the result is warped onto the marked quad by `warpToQuad()` (707).

**Improve perspective warp quality** → grid resolution in `warpToQuad()` at line 709 (`NX = 12, NY = 12`). Higher = smoother but slower; 12×12 = 288 triangles per draw.

**Change dimension estimation heuristics** → `estimateDimensions()` at line 660, calibration table at lines 666–671

**Wire up GHL webhook** → line 429, replace `'https://YOUR-GHL-WEBHOOK-URL-HERE'` with the real URL. The payload (now including `name`, `email`, `phone`, `zip`, `panels`) is built in the modal-submit handler around line 1340.

**Tweak the export image** (saved/shared/printed) → `buildExportCanvas()` at line 1199 — change the branding strip text/size/position there.

**Change print layout** → `#printSheet` HTML at lines 348–369 and the `@media print` CSS at line 121.

**Responsive layout breakpoint** → CSS line 21 (`max-width:820px` switches to single column)

---

## Code style

- **Match the existing style exactly.** Section-marker comments (`─── Name ───`), camelCase variables, descriptive names, the same gradient/shading idiom used by the shutter draw functions.
- No external libraries or CDN imports — must remain zero-dependency.
- No build tooling — the file must stay a single deployable `.html`.
- Do not break mobile/touch support. All pointer interactions use both mouse and touch event handlers.
- The off-screen shutter canvas is cached — set `shutterDirty = true` (or call `markShutterDirty()`) whenever shutter style/color/dimensions/panels change so the cache rebuilds.
- Corners stored as canvas pixels at runtime; **normalised to [0,1]** for localStorage so they survive screen-width changes. Convert back on load via `c.x * canvas.width`.
- Prefer editing in-place. Do not restructure the file layout or rename sections.

---

## Known TODOs

- **GHL webhook** (line 429): URL is a placeholder — `'https://YOUR-GHL-WEBHOOK-URL-HERE'`. Do not treat it as live. The contact-modal submission depends on it.
- **Deployment**: Method is TBD. Do not add GitHub Pages, Netlify, or any hosting config without asking first.

---

## Testing

1. Open `index.html` in Chrome or Safari.
2. Upload a window photo (drag-and-drop or click). **Photo must render immediately** — no scrolling required.
3. Mark all 4 corners in order: top-left → top-right → bottom-right → bottom-left. Try Undo and Clear.
4. Switch to "See Shutters". Verify shutters follow the perspective of the marked quad (not flat-rectangle pasted on top).
5. Drag the BEFORE / AFTER divider left/right; toggle the Compare button to hide/show the slider.
6. Cycle through all 3 shutter types, 3 louver sizes, 4 panel counts, 6 colors. Switch between Inside and Outside mount.
7. Click **Save Image** — a PNG with the branding strip should download.
8. Click **Share** — on mobile, the native share sheet should open; on desktop, the image downloads.
9. Click **Get Full Quote** — modal opens, requires name + email, posts to GHL webhook.
10. Click **Print this quote** — print dialog opens with a formatted quote sheet.
11. Reload the page — the "Continue your last visualization?" banner should appear; pick "Yes, restore" to verify state survives.
12. Resize the browser to under 820px wide — layout switches to single column, corners scale proportionally.
13. On mobile: confirm touch events work for corner placement, slider drag, and modal interaction.
