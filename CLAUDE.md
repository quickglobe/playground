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

## Working style

- Prefer vanilla HTML/CSS/JS over frameworks unless the project is specifically exploring a framework
- No need for tests unless the project is specifically about testing something
- Comments only when the behavior would be non-obvious
- Do not use emoji anywhere in the site — not in HTML, JS, or CSS; use plain text labels instead
