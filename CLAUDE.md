# ShutterViz — CLAUDE.md

Customer-facing shutter visualizer for ShuttersDirectUSA. Customers upload (or snap) a window photo, mark 4 corners with magnified precision, then see perspective-correct previews of shutters with live pricing, before/after compare, adjustable louver tilt, save/share, and printable quotes.

**Single file. Zero dependencies. No build step.**

---

## Running the app

Open `index.html` directly in any modern browser — no server, no install, no npm.

```
open index.html
```

---

## Architecture

Everything lives in one file: `index.html` (~1580 lines).

| Block | Lines |
|---|---|
| CSS (`<style>`) | 7–166 |
| HTML structure | 168–392 |
| JavaScript (`<script>`) | 394–1576 |

The JS is organized with section-marker comments in the form `─── Section Name ───`. Never remove or rename these markers — they are the navigation system.

---

## Key sections and locations

| Section | Lines | Notes |
|---|---|---|
| `─── DOM refs ───` | 395 | All element references, including modal, preview-actions, camera, tilt, and loupe |
| `─── State ───` | 439 | `img`, `corners`, `vizMode`, `shutterColor`, `sliderX/Active`, `louverTilt`, `GHL_WEBHOOK`, `STORAGE_KEY` |
| `─── Storage ───` | 469 | `saveToStorage()`, `loadFromStorage()`, `hasStored()` — corners stored normalised, includes `tilt` |
| `─── Upload ───` | 532 | File drop + `loadFile()` (downsamples to max 1400px at line 549) + **camera-capture input** wiring. Bug fix: display:block before sizeCanvas via rAF |
| `─── Coordinate helpers ───` | 590 | `canvasCoords`, `dist`, `nearestCorner`, `nearSlider`, `lerpPt` |
| `─── Magnifier loupe ───` | 613 | `showLoupe(pt)` renders a 3× zoomed circle near touch point with crosshair; `hideLoupe()` on release |
| `─── Louver tilt label ───` | 649 | `updateTiltLabel()` maps the 0–100 slider to text (Fully Closed → Wide Open) |
| `─── Pointer events ───` | 661 | Touch & mouse handlers. Slider drag wins over corners. New placements auto-enter drag mode for fine-tune with loupe |
| `─── Corner status UI ───` | 740 | Dot indicators, hint text, **undo button** |
| `─── Dimension estimation ───` | 759 | Heuristic px-to-inch mapping; calibration table inside `estimateDimensions()` |
| `─── Perspective warp ───` | 785 | `drawTexturedTriangle()`, `warpToQuad(src, quad, c, fast)`. Grid at line 815 (`N = fast ? 6 : 12`) — coarser during interactive drag, crisp on release |
| `─── Off-screen shutter render ───` | 849 | `renderShuttersOffscreen()` — rebuilds shutter texture when `shutterDirty=true`. `PIXEL_PER_INCH` at line 859 |
| `─── Draw ───` | 887 | `drawCanvas(fast)` → `drawMarkup()` / `drawPreview(fast)` → `warpToQuad()`; `drawDimensionLabels()` overlays W/H |
| `─── Before/After slider ───` | 1018 | `drawSlider()`, toggle-button handler, `setSmartSliderPosition()` (places divider opposite the window) |
| `─── Shutter rendering ───` | 1081 | `lv()`, `drawPlantationPanel()` (uses `louverTilt` for `openFrac` at line 1125), `drawArchPanel()`, `drawSunburstPanel()` |
| `─── Mode toggle ───` | 1262 | Markup / Preview tab buttons; shows the Save/Share/Compare action bar in preview |
| `─── Color picker ───` | 1278 | Swatch selection → `shutterColor`, marks shutter dirty |
| `─── Live quote ───` | 1290 | `updateQuote()` — $29/sq ft at line 1295; tilt slider listener also lives here |
| `─── Save & Share ───` | 1326 | `buildExportCanvas()` (with branding strip), save-as-PNG, Web Share API with download fallback |
| `─── Print quote ───` | 1405 | `fillPrintSheet()` populates `#printSheet` and `window.print()` |
| `─── Contact modal ───` | 1436 | `openContactModal()`, validates name + email, posts GHL webhook with contact info |
| `─── Reset ───` | 1502 | `doReset()` clears all state + localStorage; `Clear Markers` keeps the photo |
| `─── Toast helper ───` | 1534 | `toast(msg)` — 3.2s bottom notification |
| `─── Window resize ───` | 1541 | Debounced (120ms); scales corners proportionally to new canvas size |
| `─── Restore-on-load prompt ───` | 1558 | "Continue your last visualization?" banner if localStorage has state |
| `─── Init ───` | 1571 | Initial `updateCornerStatus()`, `updateQuote()`, `updateTiltLabel()` |

---

## Common tasks

**Change the price per sq ft** → line 1295 (`sqft * 29` in `updateQuote()`). The contact modal and print sheet derive from `updateQuote`, so this is the single source of truth.

**Change available colors** → HTML lines 297–302 (color swatches with `data-color` hex values)

**Change shutter types** → HTML lines 273–277 (`<select id="shutterSel">`)

**Change louver sizes** → HTML lines 282–286 (`<select id="louverSel">`)

**Change panel-count options** → HTML lines 256–262 (`<select id="panelSel">`)

**Change louver-tilt slider** (range, default) → HTML line 291 (`<input id="tiltInput">`) and JS default at line 442 (`let louverTilt = 0.32`)

**Tweak how shutters look** → `drawPlantationPanel` (~line 1092), `drawArchPanel`, `drawSunburstPanel` — all render onto the off-screen canvas, then `warpToQuad()` (785) maps the texture onto the marked quad.

**Improve perspective warp quality** → grid resolution in `warpToQuad()` line 815 (`N = fast ? 6 : 12`). Bump 12 → 16 for smoother, slower; bump 6 → 8 for crisper drags.

**Change dimension estimation heuristics** → `estimateDimensions()` (~line 765); calibration table inside that function

**Wire up GHL webhook** → line 462, replace `'https://YOUR-GHL-WEBHOOK-URL-HERE'` with the real URL. Payload (name, email, phone, zip, panels, dims, etc.) is built in the modal-submit handler

**Tweak the export image** (saved/shared/printed) → `buildExportCanvas()` (~line 1328) — change branding strip text/size/position there.

**Change print layout** → `#printSheet` HTML and the `@media print` CSS

**Responsive layout breakpoint** → CSS line 21 (`max-width:820px` → single column)

**Magnifier loupe** (size, zoom) → `LOUPE_R` and `LOUPE_ZOOM` constants in State section (line ~447), plus `showLoupe()` at line 615 and the `#loupe` CSS rule

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
3. On a phone, the **Take Photo** button should appear under the drop zone and launch the rear camera.
4. Mark all 4 corners in order: top-left → top-right → bottom-right → bottom-left. While placing/dragging each corner, the **magnifier loupe** appears with a yellow crosshair at the exact placement. Try Undo and Clear.
5. Switch to "See Shutters". Verify shutters follow the **perspective** of the marked quad (not flat-rectangle pasted on top).
6. Drag the BEFORE / AFTER divider left/right; toggle the Compare button to hide/show the slider. The slider should auto-position **opposite the window** on first enter.
7. Drag the **Louver Tilt** slider — see blades open/close in real time, label updates Fully Closed → Wide Open.
8. Cycle through all 3 shutter types, 3 louver sizes, 4 panel counts, 6 colors. Switch between Inside and Outside mount.
9. Click **Save Image** — a PNG with the branding strip should download.
10. Click **Share** — on mobile, the native share sheet should open; on desktop, the image downloads.
11. Click **Get Full Quote** — modal opens, requires name + email, posts to GHL webhook.
12. Click **Print this quote** — print dialog opens with a formatted quote sheet.
13. Reload the page — the "Continue your last visualization?" banner should appear; pick "Yes, restore" — including the tilt slider position.
14. Resize the browser to under 820px wide — layout switches to single column, corners scale proportionally.
15. On mobile: confirm touch events work for corner placement (with loupe), slider drag, tilt drag, and modal interaction. Numeric keyboards should appear on Width/Height/ZIP inputs.
