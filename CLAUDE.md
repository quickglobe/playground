# Claude Code Guide

This repo is a personal playground for small, self-contained projects and experiments, hosted on GitHub Pages. Each project lives in its own folder under `main`, and the root `index.html` is a browsable project index.

## Project conventions

- Each experiment or project gets its own folder (e.g. `my-project/index.html`)
- Add a card for each new project to the root `index.html`
- Projects are primarily HTML/CSS/JavaScript — single files or small multi-file setups
- No build step unless the project specifically calls for it
- Keep things simple and self-contained

## Favicons and bookmark icons

The site is served from a project subpath (`https://quickglobe.github.io/playground/`), so **always use relative icon paths** (`favicon.svg`, not `/favicon.svg`). Absolute paths beginning with `/` resolve to the domain root, where nothing is served, and every icon silently 404s.

The root index has a full icon set (`favicon.svg`, `favicon.ico`, sized PNGs, `apple-touch-icon.png`, `site.webmanifest`, `browserconfig.xml`). Each project folder gets its own project-specific icon set so tabs and home-screen bookmarks are distinguishable.

To add icons for a new project:

1. Hand-author a `favicon.svg` in the project folder using a `512x512` viewBox and the rounded-square style of the root icon (`rx="115"`), themed to match the project's colors.
2. Generate the raster variants from that SVG (the repo has `cairosvg` and `pillow` available via pip):
   - `favicon-16x16.png`, `favicon-32x32.png`
   - `apple-touch-icon.png` (180x180) — flatten onto the project's solid background color so iOS's rounded mask has no transparent/black corners
   - `favicon.ico` — multi-resolution (16/32/48), rendered from the SVG at 256 and saved with `sizes=[(16,16),(32,32),(48,48)]`
3. Add the link tags to the project's `<head>` with relative `href`s, plus a `theme-color` meta matching the project background:

   ```html
   <link rel="icon" type="image/svg+xml" href="favicon.svg">
   <link rel="icon" type="image/x-icon" href="favicon.ico">
   <link rel="icon" type="image/png" sizes="32x32" href="favicon-32x32.png">
   <link rel="icon" type="image/png" sizes="16x16" href="favicon-16x16.png">
   <link rel="apple-touch-icon" sizes="180x180" href="apple-touch-icon.png">
   ```

Browsers cache favicons aggressively; a hard refresh (or re-adding the home-screen bookmark on iOS) may be needed to see updates.

## Previewing changes and pull requests

Sessions often run in the cloud and are driven from a phone, so the only "preview" is the deployed GitHub Pages site — and that only reflects `main`. To avoid a merge / test / fix / merge-again loop:

- **Keep the PR open until the change is confirmed working.** Push iterative fixes as new commits to the same branch rather than opening a fresh PR each round. Merge only once it is verified.
- **Preview a branch without merging via raw.githack.** For a self-contained file, open `https://raw.githack.com/quickglobe/playground/<branch>/<project>/index.html` on the device. It serves over HTTPS with correct content types, so the page renders and HTTPS-only APIs (geolocation, etc.) work. Use this to check on a real device before merging.
  - **Make the link clickable/tappable.** Put the bare githack URL on its own line (no surrounding markdown link text, no parentheses, no trailing punctuation), so the terminal/phone client auto-links it. Do not hand the user the PR (`github.com/.../pull/N`) URL as the preview — that opens the diff view, not the rendered page; the `raw.githack.com` URL is the preview.
  - githack caches aggressively, so if a preview looks stale, append a cache-busting query (`?v=2`, bump the number each time).
- **Some things only reproduce on a real device** (e.g. iOS Safari status bar / toolbar tinting), so a device check via the githack link is the last step before merge — desktop/headless browsers will not show them.
- For changes that do not depend on device chrome (layout, JS logic, canvas), they can be verified headlessly in the session before the user ever looks.

## Rendering grids on canvas

Square-grid canvases (Game of Life, pixel editors, the landing-page previews) keep
hitting the same family of bugs: a row/column of half-covered cells at one edge, a
gap between the grid and its frame, blurry lines, and cells bleeding over the grid
lines. Fixing one naively reintroduces another, so apply all of these together.

- **Never round the cell count up.** `cols = Math.ceil(W / cell)` makes the grid
  wider than the canvas, so the last row/column is clipped mid-cell. Use the largest
  whole number of cells that *fits*: `Math.floor` (or `Math.round` if you intend to
  stretch — see below).
- **Decide who owns the leftover sub-cell remainder**, because `W`/`H` is almost never
  an exact multiple of the cell size:
  - *If the container can be resized to the grid* (a standalone app like
    `game-of-life/`), size it to `cols*cell (+ border)` in JS. The grid is then flush
    with the frame — no gap, no overlap. This is the cleanest option.
  - *If the container is fixed* (a preview tile that must fill a card), don't centre
    the integer grid — that leaves a visible background margin (a gap) on the right and
    bottom. Instead stretch the cells to fill edge-to-edge: pick the cell count nearest
    the target (`Math.round(W/cell)`) and snap every boundary to a whole pixel with
    `colX = i => Math.round(i*W/cols)` (same for rows). Cells come out 11-13px for a
    ~12px target — near-square and unnoticeable. The first boundary is `0` and the last
    is exactly `W`/`H`, so it fills with no gap and no overflow.
- **Render at CSS resolution + `image-rendering: pixelated`, not at device resolution.**
  Scaling the context by `devicePixelRatio` pushes the `+0.5` crisp-line offsets onto
  fractional device pixels under a 2x/3x display, so the browser antialiases every grid
  line into a blurry mess. Set `canvas.width = clientWidth` (no `ctx.scale(dpr, dpr)`)
  and let the pixelated upscale keep cells and lines crisp. (Smooth scenes like the sun
  preview's gradients are the exception — they *want* DPR scaling and antialiasing.)
- **Don't let cells paint over the grid lines.** Drawing full-size cells and then
  stroking lines on top puts each line on the left/top pixel of its neighbour cell, an
  asymmetry that reads as overlap when zoomed. Inset each filled cell by 1px on its
  left/top edge (`colX(x) + (x ? 1 : 0)`); the right/bottom line belongs to the next
  cell, which is inset too, so every interior line keeps a clean 1px gap with cells
  flush on both sides.
- **Draw interior divisions only; let the frame be the outer border.** Lines at
  `c==0`/`c==cols` either double up on the box border or get clipped at the canvas edge.
  Draw lines for `1..cols-1` / `1..rows-1` only and let the container's border (or the
  preview tile's frame) supply the outer edge.
- **Verify the geometry headlessly before the device check.** The invariants above are
  pure arithmetic — assert them in Node across a range of widths (boundaries fill `0..W`
  exactly, no cell exceeds `W`/`H`, no cell overlaps a line pixel) so a device check only
  has to confirm it *looks* right, not whether the maths holds.

## Working style

- Prefer vanilla HTML/CSS/JS over frameworks unless the project is specifically exploring a framework
- No need for tests unless the project is specifically about testing something
- Comments only when the behavior would be non-obvious
- Do not use emoji anywhere in the site — not in HTML, JS, or CSS; use plain text labels instead
