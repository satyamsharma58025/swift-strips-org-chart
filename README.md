# Swift Strips India – Organization Chart

Interactive, editable organization chart with a liquid-glass UI and vector PDF export.
Plain static site: no build step, no dependencies to install.

## Deploy

**Netlify:** push to GitHub → *Add new site → Import from Git* → pick the repo.
Build command: *(empty)* · Publish directory: `.`

**Render:** *New → Static Site* → pick the repo.
Build command: *(empty)* · Publish directory: `.` (or use `render.yaml` as a Blueprint).

## Run locally
    python3 -m http.server 8080   # then open http://localhost:8080

## Use
- Drag to pan, Ctrl/⌘ + scroll or pinch to zoom, **F** fits the screen, **/** focuses search.
- **Edit** → click a card to change name, phone, role, style, reporting line; add/delete/reorder; undo/redo.
- **Export** → PDF (A4/A3/A2), CSV, JSON backup/import, reset.
- Edits are saved in the browser (localStorage). Use JSON backup/import to move edits between devices,
  or edit the `SEED` array in `index.html` to change the default chart for everyone.

Fonts (Inter, JetBrains Mono, Libre Baskerville) load from Google Fonts and fall back to system fonts offline. jsPDF 2.5.1 is bundled in `vendor/` (MIT). Logo: `assets/logo.png`.
