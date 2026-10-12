# agent.md — Swift Strips Organization Chart

Hand-off context for any AI agent or developer continuing this project. Read this first, then `index.html`.

---

## 1. Intent (why this exists)

**Owner:** Satyam, Senior Software Lead, Swift Strips India Private Limited (SLTT Group), Jamshedpur. Aluminium recycling manufacturer.
**Tagline (use verbatim):** *Aluminium recycling for a sustainable society*
**Company name (use in full in headings):** *Swift Strips India Private Limited*

The company's org chart lived as drawn shapes inside an Excel file (`organization_structure_new.xls`; cells empty, 61 shapes + connectors). Goal: replace it with a **top-tier, industry-grade web app** that is:

1. **Editable** on the page (people, roles, phones, reporting lines), with undo/redo.
2. **Exportable** to PDF (vector), plus CSV and JSON backup.
3. **Visually premium**: Apple "Liquid Glass" aesthetic matching the reference video (liquidGL demos), with the **company logo and tricolour palette** (saffron / green / navy).
4. **Deployable as a static site** (GitHub -> Netlify or Render), no build step.

The user is a demanding design reviewer. They asked for the work to be done "as a senior UI/UX designer with 15+ years at Apple". Expect further polish requests; favour restraint, hierarchy and clarity over decoration.

---

## 2. Repository layout

```
index.html            # entire app: CSS + HTML + JS (single file, no framework)
assets/logo.png       # circular, transparent-corner logo (256px). Also inlined as base64 in index.html (const LOGO)
vendor/jspdf.umd.min.js   # jsPDF 2.5.1 (MIT), loaded locally for PDF export
vendor/jspdf-LICENSE.txt
netlify.toml          # publish "." + security headers
render.yaml           # Render static site blueprint
README.md             # deploy + usage
agent.md              # this file
.gitignore
```

No package.json, no bundler. Edit `index.html` directly. Serve locally with `python3 -m http.server 8080`.

Fonts (Google Fonts, with system fallbacks): **Inter** (UI), **JetBrains Mono** (labels, bracket style from video), **Libre Baskerville** (company name, echoes the serif in the logo).

---

## 3. Source data (ground truth from the .xls)

57 people + 3 functions (Security, Safety, Process Management Office). Data lives in the `SEED` array in `index.html`:
`[id, parentId, tier, title, name, phone, opts]`, opts = `{mode:'h'|'v', side:'l'|'r', group:1, dept, gap, lvl}`.

Hierarchy as read from the drawing:

- Managing Director: Jawahar Lal Vig
  - Director: Shruti Vig
  - CEO: Shaurya Vig
    - HR: Ravi Anand -> Security, Safety
    - EA to CEO: Sushmita Mukherjee (7903066993) -> Data Entry Operator Sneha (9973913178)
    - Plant Head: Yogesh Tomar -> Quality Counsellor Sourabh Mishra, 5S Executive Sonu Nishad, **Process Management Office** -> the 7 departments:
      1. Operation & RM Head Santosh Kumar -> Sr. Managers Om Kumar, Rupesh Choudhar -> supervisors/assistant/sorter
      2. HOD Accounts Rajeshwar Pandey -> 2 Sr. Accountants
      3. HOD Store & Purchase Chandan Kumar -> 2 Sr. Supervisors, Store Keeper
      4. HOD Mech. Maintenance Ram Rekha Prasad -> 3 Sr. Technicians, 3 Fitters
      5. HOD Electrical Pratyush Nanda -> Sr. Technician, Fitter, Helper
      6. Incharge Production Satish & Dev Kumar Nag -> Sr. Incharge, supervisors, JCB and forklift drivers
      7. HOD Dispatch Mobin Ansari -> 6 supervisors (one is QA)

### Assumptions I made (flag these if the user disagrees)

| # | Assumption | Original drawing |
|---|---|---|
| 1 | Ops staff **split 5/5**: Om Kumar gets Avinash Pandey, Farhan, Aryan, Geda Tudu, Sanjay Prasad; Rupesh Choudhar gets Ram Ajore, Santosh Kumar, Makhan, Gollu Mahato, Rohit Kumar | First 5 connected to **both** managers; last 5 had no connector |
| 2 | PMO is a child of **Plant Head** | Both Plant Head and EA to CEO linked to it |
| 3 | **Director placed beside the MD** (side-left) so the CEO stays on the centre axis | Director and CEO were peers under MD |
| 4 | **Department order changed** for symmetry: Accounts, Store, Mech | **Operations (centre)** | Electrical, Production, Dispatch | Original: Ops, Accounts, Store, Mech, Electrical, Production, Dispatch |
| 5 | Spellings kept as in source ("Rupesh Choudhar", "Dev Mukherhee") | Likely typos (Choudhary, Mukherjee); unconfirmed |
| 6 | Duplicate numbers kept: 8051122341 (Dev Mukherhee and Ritesh Pattanayak), 9155239079 (Suraj Kumar and Manash Giri) | Same in source |

Tier colours in the source: magenta = senior roles, green = HODs, steel blue = others. Mapped to: senior = saffron, head = green, team = quiet slate-blue, exec = navy.

---

## 4. Architecture of `index.html`

Order in file: `<style>` (tokens -> ambient -> glass -> components -> media queries), markup, one IIFE `<script>`.

### 4.1 Data model
`S.nodes[]` each: `{id,parent,tier,title,name,phone,mode,side,group,dept,gap,lvl,note,collapsed}`.
- `tier`: `exec | head | senior | team | unit` (unit = function pill with no person).
- `mode`: how this node's children are laid out: `h` row beneath, `v` indented stack with a spine.
- `side`: `''` below, `'l'` / `'r'` beside the parent (support roles).
- `group`: renders a **department panel** around the subtree; `dept` is its label.
- Sibling order = order in the `nodes` array.

### 4.2 Layout engine (`lay()` + `compute()`)
Custom, deterministic, no library. `lay(n)` returns a box `{w,h,rx,...}` bottom-up; `compute()` places boxes top-down and emits `cards`, `wires`, `panels`.
- `h` mode: children side by side, parent centred over first/last child centres.
- `v` mode: children indented (`G.INDENT`), spine from parent to each child's left edge.
- Side stacks (`l`/`r`) hang beside the parent card; if a left stack needs room the whole box shifts.
- **Centre-axis rule:** axis = root card centre. Canvas width = 2 x (max horizontal distance from axis) + padding, so the layout is symmetric and the "Fit" view is centred. Do not break this.
- Wires carry `{pts, w, to, dot}`; `w` is thicker near the top (2.6 / 2.0 / 1.5). `to` is the child id (used for hover-chain highlighting). Corners rounded by `rpath()`.
- `compute(true)` ignores collapsed state (used by PDF "include collapsed teams").
- Sizes per tier in `TIER` (exec 330x112, head 304x94, senior 238x76, team 196x58, unit 276x46). Dept boxes assume all six non-Ops departments have equal width, which is what keeps the row symmetric.

### 4.3 Rendering
Full DOM rebuild on every change (`renderWorld()`): ~60 absolutely positioned `.card` divs + one SVG for wires + `.panel` divs. Inspector (`renderSheet()`) is separate so typing doesn't lose focus (`live()` updates the world only).
Pan/zoom: custom `#world` CSS transform; pointer events (drag, two-finger pinch), Ctrl/Cmd+wheel zoom, wheel pans; `fit()`, `frame()`, `focusCard()` animate with rAF (respects reduced motion).

### 4.4 State, history, persistence
- Undo/redo: JSON snapshots (`pushUndo`, max 120). `mutate()` for structural changes, `live()` for coalesced text typing.
- Persistence: always `localStorage` (`ssi-orgchart-v1`). If the Claude artifact runtime `db` capability exists, also syncs doc `charts/org` (`{json, by, at}`), ignoring own echoes. **On a plain static host only localStorage applies** (per browser). To make edits shared for everyone, change `SEED` in the file, or add a backend (see section 9).
- Theme: `data-theme` on `<html>` overrides `prefers-color-scheme`; remembered in localStorage.

### 4.5 Export
- **PDF:** `pdfChart(format, includeCollapsed)` draws vector primitives with jsPDF directly (not a screenshot): header with logo + serif company name + tagline, legend, department panels, wires, cards, footer. A4/A3/A2 landscape, scaled to fit. Uses `helvetica` and `times` built-ins.
- **CSV:** ID, Name, Role, Phone, Reports to, Department, Level.
- **JSON:** `{app:'swift-strips-orgchart', v:1, nodes}`; import validates tiers and exactly one root.
- `saveFile()` tries the artifact `downloads` capability first, then falls back to a normal `<a download>` (this is what works on Netlify/Render).

---

## 5. Design system

### 5.1 References
- **Reference video** (liquidGL / "Apple Liquid Glass on your website"): pale lavender-white ground; black pill buttons; monospace `[BRACKETED]` labels; hairline grid; clear glass that refracts vivid iridescent blue/cyan/pink/peach; iOS-style tab bar dock with a solid active pill; deep navy/violet in dark.
- **Logo:** circular badge; saffron arc, green arc, navy arc and flame; serif wordmark "Swift Strips India / Private Limited".

### 5.2 Principles (keep)
1. **Seniority = elevation.** Executives are largest, most tinted, deepest shadow, 21px bold name. Team cards are small, near-transparent, flat, 13px regular. Reporting lines are heavier near the top.
2. **Colour is information, not decoration.** Only exec (navy), head (green), senior (saffron) carry colour. Team = quiet slate. One contained fluid glow sits behind the chart centre; no rainbow ground.
3. **Centre-oriented.** Everything on one vertical axis; support roles beside it; symmetric department row.
4. **Brand plate is opaque** (white in light, navy `#10123F` in dark) so logo and name are never washed out by glass.
5. Glass = translucent gradient + `backdrop-filter` blur/saturate + chromatic edge (`inset 1.4px` pink/cyan) + specular rim (`::after` masked gradient). Header/dock/zoom also get an SVG turbulence displacement (`#lg`) in Chromium only; other browsers fall back to plain blur.
6. Dark theme must stay **deep and neutral** (`--bg0:#05051A`, low `--orb`, `--sat:1.2`). Earlier versions were too electric/saturated and the user rejected them.

### 5.3 Tokens (CSS variables on `:root`; dark values under both `@media (prefers-color-scheme:dark)` guarded by `:root:not([data-theme="light"])` and `:root[data-theme="dark"]`)
Surface: `--bg0 --bg1 --ink --muted --faint`. Glass: `--g-a --g-b --g-edge --g-hi --g-lo --g-glow --g-shadow --panel --panel-edge --blur --sat`. Chroma: `--chroma-r --chroma-b`. Plate: `--plate --plate-edge`. Buttons: `--btn --btn-ink` (black pill in light, white in dark). Accent: `--accent` (saffron; selection ring, hover-chain wires). Tiers: `--t-exec --t-head --t-senior --t-team --t-unit`. Fluid: `--f1a..e`, `--orb`. Type: `--mono --serif`.
When adding any colour, define it light and dark in all three places.

### 5.4 Interactions implemented
Search (name/role/phone/dept; Enter cycles matches, expands collapsed ancestors; `/` focuses, Esc clears) · hover or select highlights reporting chain · collapse/expand per team (+N badge) · department dock (active pill) · zoom controls, Fit (`F`) · Edit mode (inspector: title, name, phone, style, placement, reports-to, team layout, department panel, notes, add / move earlier-later / delete with confirm) · undo/redo (Ctrl/Cmd+Z, Shift+Z) · breadcrumbs of reporting chain in inspector · phone copy button · export menu · light/dark toggle · mobile layout (header wraps, bottom-sheet inspector, starts framed on leadership).

---

## 6. Deploy

- **Netlify:** import repo, build command empty, publish dir `.` (`netlify.toml` provided).
- **Render:** New -> Static Site, build empty, publish `.` (or Blueprint via `render.yaml`).
- The repo should be **private** (contains staff phone numbers). Suggested name: `swift-strips-org-chart`.
- No secrets, no env vars, no server.

A hosted copy also exists as a Claude artifact (same code, jsPDF from cdnjs, `capabilities: {db, downloads}`). It is private to the owner and is a convenience preview; the repo is the deliverable.

---

## 7. Testing recipe (what was used)

Headless Chromium via `playwright-core` (`executablePath: /opt/pw-browsers/chromium`, `--no-sandbox`). Block external hosts, serve jsPDF locally. Checks run each iteration:
- 60 `.card` elements render; `compute(true)` yields **0 overlapping cards**.
- `window.__ssi.pdf('a3', true)` returns a Blob; rasterise with `pdftoppm` and eyeball it.
- Screenshots at 1440x900 light, 1440x900 dark, 390x844 light; hover (`[data-id="mech"]`), select (`[data-id="mm"]`), edit mode.
- A static-server run: download event for `swift-strips-organization-chart-A3.pdf`, no page errors.
Debug hook: `window.__ssi = {pdf, state, compute}`. Note: in headless runs the sheet's entrance animation can look invisible for ~1s; wait before screenshotting.
Expected console noise offline: failed Google Fonts requests only.

---

## 8. Rules for future edits (do not regress)

1. Keep it a **zero-build static site** unless the user asks otherwise.
2. Preserve the **centre-axis symmetry** and equal dept widths; re-run the overlap check after any size/layout change.
3. Preserve **seniority hierarchy** (sizes, tint, shadow, weight scale).
4. Brand: full company name + tagline as above; logo on an **opaque** plate; tricolour used sparingly (tier colours, tagline stripe).
5. Both themes get equal care; check dark for contrast of the smallest text (team `.ti`, `.ph`).
6. Escape all user text with `esc()` before `innerHTML`.
7. Wrap `localStorage` access in try/catch. Never rely on `alert/confirm/prompt` (use inline two-step buttons).
8. PDF must stay vector and use the same tier colours, logo and header lockup.
9. Don't invent people or numbers. Data changes come from the user.
10. Respect `prefers-reduced-motion`; keep focus rings visible; keep 16px side gutters on mobile.

---

## 9. Likely next refinements (intent-based backlog)

Ordered by probable user value:

1. **Shared persistence** so one edit updates everyone: Supabase / Firebase / Netlify Functions + Blobs, or commit-to-GitHub via API; add simple auth (Netlify Identity or password gate) since data includes phone numbers.
2. **Drag-and-drop re-parenting** and reordering on the canvas (currently done via "Reports to" select and Move earlier/later).
3. **Dual reporting lines** (the two Ops managers sharing staff) as a secondary dotted connector; add `alsoReportsTo`.
4. **Photos/avatars** per person (upload or URL) replacing initials.
5. **Multi-page / poster PDF**, per-department PDF, PNG/SVG export, print stylesheet.
6. **Real refraction** with WebGL (liquidGL itself) for header and cards, with a graceful fallback; keep performance on low-end phones.
7. **Hindi / bilingual** labels (names and roles) and Devanagari font in the PDF (jsPDF built-in fonts are Latin only; embed a TTF).
8. **Vacancy tracking, headcount by department, span-of-control stats**, "as of" snapshots/version history.
9. **Data import from the original Excel** or a Google Sheet (CSV already exports; add CSV import).
10. Accessibility pass: arrow-key navigation across the tree, screen-reader tree semantics (`role="tree"`).
11. Tighten mobile: persistent leadership-first view, bottom-sheet inspector polish.

When the user asks for "more refinement", default to: polish interactions and typography, keep the palette calm, improve legibility at fit zoom, and verify with screenshots in both themes before delivering.

---

## 10. Working style with this user

- Wants finished, deployable output, not explanations. Deliver the zip/file plus a short summary of what changed and any assumption that needs confirming.
- Reviews visually and is blunt about what looks wrong ("colour themes are still so bad"); respond by changing the underlying approach, not by nudging values.
- Gives brand assets directly (logo image, reference video). Treat them as authoritative.
- Related ongoing work (context only): Swift Strips MERN ERP, n8n + WhatsApp automations, Live Jharkhand news portal, freelance automation pitch for Jamshedpur businesses. This chart is a standalone static site and shares no code with those.
