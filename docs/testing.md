# Manual test checklist

Read this before merging any change that touches drawing, pricing, the quote
flow, or layout — not needed for a pure copy/color tweak. Run it in Chrome or
Safari against `index.html` directly (no server needed).

1. Upload a window photo (drag-and-drop onto the upload zone, or click it).
   The canvas should appear immediately with a centered 4-corner overlay —
   no scrolling required.
2. Drag the overlay by its middle to reposition it; drag an individual
   corner to resize/skew it to fit the window frame (this is how perspective
   is set — there's no separate "mark 4 corners in order" step).
3. Toggle **With Shutters** / **Original** — canvas should redraw instantly
   between the photo and the shutter preview.
4. Cycle shutter type (Plantation / Arch / Sunburst), panel count (1–4),
   louver size (2½″ / 3½″ / 4½″), color (6 swatches), and mount type
   (Inside / Outside). Every change should redraw the canvas live, and
   switching mount type should expand the shutters slightly outward
   ("outside mount" look).
5. Switch mount type and confirm the "How to Measure" guide box swaps its
   instructions between inside- and outside-mount wording.
6. Enter width/height and click **Calculate My Estimate** — verify the price
   ($29/sq ft), retail comparison ($49/sq ft), and savings line all agree
   with the entered dimensions.
7. Pick a combination that exceeds the panel-width limit (e.g. a wide
   window with 1 panel and a 3½″+ louver) and confirm the panel-width
   warning appears with a suggested panel count, and that clicking
   **Calculate My Estimate** is blocked until it's resolved.
8. Click **Request Full Quote** — modal opens with a correct summary
   (size/type/louver/color/mount/price) and the 5-step "what happens next"
   list.
9. Check "I have multiple windows or a complex order" and confirm the
   multi-window note appears in the modal.
10. Fill in name, email, phone, and ZIP and submit. Confirm:
    - the lead lands in GoHighLevel with the full payload (contact info,
      dimensions, shutter config, pricing), and
    - the EmailJS confirmation email arrives at the configured recipients
      (`EMAIL_TO`/`EMAIL_CC` in `index.html`).
11. Click **Start Over** (either the toolbar button or the one under the
    quote box) — confirms the photo, overlay, inputs, and quote section all
    reset to their initial state.
12. Resize the browser to under 820px wide — layout switches from the
    two-column grid to a single column.
13. On a phone or touch device: confirm touch dragging works for
    repositioning the overlay and its corners, and that Width/Height/Zip
    inputs bring up the numeric keyboard.
