# Sun Tracker — handoff: pre-location reveal transition

## Task

Smooth out the transition from the pre-geolocation "holding" state to the
located/palette state. Most visible symptom: the bottom toolbar (and the scene
generally) starts dark and fades to the daytime palette once geolocation
resolves, which looks like an abrupt dark-to-light jump.

Everything lives in the single self-contained file `sun-tracker/index.html`.

## Testing

Test on the deployed site (`https://quickglobe.github.io/playground/sun-tracker/`)
on a real iOS device. The OS chrome behaviour (status bar / toolbar tint) cannot
be reproduced in desktop or headless browsers. Use a raw.githack branch link to
preview before merging (see the "Previewing changes" section in the root
`CLAUDE.md`).

## How the reveal currently works

- Before location resolves, `tick()` runs with `lat == null`: it calls
  `drawScene(W, H, null, null, 0, null)` which paints the **default dark night
  scene**, sets the clock/date, and returns early **without calling
  `applyPalette`**. So `<body>` keeps its CSS default `#070710` and `#safe-bottom`
  keeps `#05050b` (both dark).
- On location resolve, the geolocation callback calls `tick()`, which runs
  `revealData()` (removes `.data-hidden` to reveal the stats with an
  opacity/translate transition) and then `updateUI()` -> `applyPalette()`, which
  sets the live colours.

## The iOS 26 chrome-tint solution — do NOT break it (merged in PRs #37–#39)

This was hard-won; keep it intact while reworking the transition.

- The page deliberately never scrolls (`html, body { overflow: hidden }`,
  `touch-action: none`). At `scrollY = 0`, iOS 26 Safari tints the **top status
  bar from the `<body>` background only** — fixed elements are not consulted for
  the top. So `applyPalette` sets `document.body.style.background = skyTop`
  (the sky gradient's top colour) to drive the status bar.
- The **bottom toolbar** is tinted from a single `position: fixed` `#safe-bottom`
  strip. It has no `z-index` (paints behind the canvas, invisible) and
  `height: max(env(safe-area-inset-bottom, 0px), 16px)` — the `max` is required
  because Safari reports the inset as `0` while the floating toolbar is visible,
  which would collapse the strip below the 3px qualifying threshold.
  `applyPalette` sets its `backgroundColor = groundEnd`, where
  `groundEnd = darken(p.bg, 0.32)` (the gradient's bottom colour).
- `theme-color` is ignored by Safari 26 (kept only for older iOS / Android).
- Reference: the algorithm/thresholds are reverse-engineered (no Apple docs);
  see `andesco/safari-color-tinting`, 1ar.io, nasedk.in, jahir.dev.

## Things to consider for the transition work

- `#safe-bottom` has `transition: background-color 1.2s ease`; `<body>` has no
  transition. That asymmetry is part of why the bottom fades while the top snaps
  to the new colour. Coordinate `<body>`, `#safe-bottom`, and the canvas redraw
  so chrome and content move together.
- Decide whether the holding state should already approximate the local-time
  palette — e.g. compute an approximate sun altitude from the device clock before
  geolocation, so there is no big dark-to-light jump — or keep it dark and just
  make the fade graceful and coordinated.
- The default colours `#070710` (`<body>`) and `#05050b` (`#safe-bottom`) are the
  start-of-fade values; revisit them alongside the holding-state design.
