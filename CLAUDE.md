# ShutterViz — CLAUDE.md

Customer-facing shutter visualizer for ShuttersDirectUSA. Customers upload a window photo, mark 4 corners, then see photorealistic previews of shutters with live pricing.

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
| CSS (`<style>`) | 7–93 |
| HTML structure | 95–230 |
| JavaScript (`<script>`) | 232–850 |

The JS is organized with section-marker comments in the form `─── Section Name ───`. Never remove or rename these markers — they are the navigation system.

---

## Key sections and locations

| Section | Lines | Notes |
|---|---|---|
| `─── DOM refs ───` | 233 | All element references |
| `─── State ───` | 250 | `img`, `corners`, `vizMode`, `shutterColor`, `GHL_WEBHOOK` |
| `─── Upload ───` | 267 | File drop + `loadFile()` — downsamples to max 1400px |
| `─── Coordinate helpers ───` | 313 | Canvas pixel ↔ screen pixel math |
| `─── Pointer events ───` | 329 | Mouse + touch unified into `handleStart/Move/End` |
| `─── Corner status UI ───` | 364 | Dot indicators and hint text |
| `─── Dimension estimation ───` | 373 | Heuristic px-to-inch mapping; see calibration table |
| `─── Draw ───` | 407 | `drawCanvas()` dispatches to `drawMarkup()` or `drawPreview()` |
| `─── Shutter rendering ───` | 541 | `drawPlantationPanel`, `drawArchPanel`, `drawSunburstPanel`, `lv()` |
| `─── Mode toggle ───` | 735 | Mark vs. Preview mode buttons |
| `─── Color picker ───` | 747 | Swatch selection → `shutterColor` |
| `─── Live quote ───` | 757 | `updateQuote()` — $29/sq ft at line 762 |
| `─── Reset ───` | 775 | `doReset()` — full restart |
| `─── Get Quote (GHL webhook) ───` | 790 | Submits JSON payload via `fetch` in `no-cors` mode |
| `─── Toast helper ───` | 824 | `toast(msg)` — 3.2s bottom notification |
| `─── Window resize ───` | 831 | Debounced canvas resize (120ms) |

---

## Common tasks

**Change the price per sq ft** → line 762 (`sqft * 29`) and line 795 (`w * h / 144 * 29`)

**Change available colors** → HTML lines 193–199 (color swatches with `data-color` hex values)

**Change shutter types or louver sizes** → HTML lines 174–188 (the two `<select>` elements)

**Tweak how shutters look** → `drawPlantationPanel` (552), `drawArchPanel` (644), `drawSunburstPanel` (681); `lv()` (542) lightens/darkens a hex color

**Change dimension estimation heuristics** → `estimateDimensions()` at line 378, calibration table at lines 391–396

**Wire up GHL webhook** → line 265, replace `'https://YOUR-GHL-WEBHOOK-URL-HERE'` with the real URL. The payload shape is at lines 797–808.

**Responsive layout breakpoint** → CSS line 21 (`max-width:820px` switches to single column)

---

## Code style

- **Match the existing style exactly.** Section-marker comments (`─── Name ───`), camelCase variables, descriptive names.
- No external libraries or CDN imports — must remain zero-dependency.
- No build tooling — the file must stay a single deployable `.html`.
- Do not break mobile/touch support. All pointer interactions use both mouse and touch event handlers.
- Prefer editing in-place. Do not restructure the file layout or rename sections.

---

## Known TODOs

- **GHL webhook** (line 265): URL is a placeholder — `'https://YOUR-GHL-WEBHOOK-URL-HERE'`. Do not treat it as live. Ask before making assumptions about the CRM setup.
- **Deployment**: Method is TBD. Do not add GitHub Pages, Netlify, or any hosting config without asking first.

---

## Testing

1. Open `index.html` in Chrome or Safari.
2. Upload a window photo (drag-and-drop or click).
3. Mark all 4 corners in order: top-left → top-right → bottom-right → bottom-left.
4. Switch to "See Shutters" and cycle through all 3 shutter types, 3 louver sizes, and 6 colors.
5. Verify the quote panel shows a reasonable sq ft and price.
6. Resize the browser to under 820px wide — layout should switch to a single column.
7. On mobile: confirm touch events work for corner placement and dragging.
