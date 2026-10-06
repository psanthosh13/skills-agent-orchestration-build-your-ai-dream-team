# Project Pulse implementation plan

## Summary

Project Pulse is Mona's lightweight static dashboard that lets contributors quickly see which projects are active, who owns each one, current status, recent activity, and priority/risk level, inside a polished, accessible, card-based UI. It is a small static web app with no build step and no backend:

- `app/index.html` — page structure, title "Project Pulse", a `.dashboard` container, and project cards rendered from data.
- `app/styles.css` — visual design: `.dashboard` and `.project-card` selectors, status badges, rounded corners (`border-radius`), shadows (`box-shadow`), responsive layout, and accessible contrast/typography.
- `app/project-data.json` — static data source with a top-level `projects` array; each project has `name`, `owner`, `status`, `recentActivity`, and `priority`.
- `.vscode/launch.json` — a VS Code launch configuration named **Run Project Pulse Dashboard** that serves `app/` with `python3 -m http.server 5500` and uses `serverReadyAction` to open `http://localhost:%s/index.html`, so the dashboard frontend opens directly instead of a directory listing.

The repository currently has an empty `app/` directory and no `docs/project-pulse-plan.md`, `.vscode/launch.json`, or app files yet — all four target files are to be created from scratch. Work is carried out by four custom Copilot CLI agents (Orchestrator, Planner, Designer, Coder) defined in `.github/agents/`, with the learner driving all git operations; no agent stages, commits, or pushes.

## Implementation steps

1. **Confirm/author the data schema** — Define `app/project-data.json` with a top-level `"projects"` array; each entry has `name`, `owner`, `status`, `recentActivity`, `priority` (and optionally a short contributor-friendly `summary` field, since the brief mentions a summary but the required-fields list in the brief/workflow checks only names the five fields above — treat `summary` as optional/stretch, not required).
2. **Build the HTML structure** — Create `app/index.html` with an exact `<title>`/heading text "Project Pulse", a `<link rel="stylesheet" href="styles.css">`, a `.dashboard` root element, a `<script>` (inline or `app.js`-free, fetch-based) that loads `project-data.json` and renders one `.project-card` element per project, displaying `status`, `recentActivity`, and `priority` as visible text/badges.
3. **Style the dashboard** — Create `app/styles.css` implementing `.dashboard` (responsive grid/flex layout) and `.project-card` (with `border-radius`, `box-shadow`, spacing, status-badge colors by priority/status, and responsive breakpoints), plus accessible color contrast and focus states.
4. **Create the run/debug configuration** — Create `.vscode/launch.json` (strict JSON, no comments) with a configuration named exactly **Run Project Pulse Dashboard**, `cwd` set to `${workspaceFolder}/app`, command `python3 -m http.server 5500`, and `serverReadyAction` pattern that opens `http://localhost:%s/index.html`.
5. **Integrate and validate** — Confirm the HTML fetches `project-data.json` correctly (relative path, no CORS issues when served via `http.server`), confirm CSS classes match what HTML emits, confirm JSON is valid and matches the field names used by the rendering script, and manually run the launch configuration to see the dashboard (not a directory listing).
6. **Learner commits** — The learner (not the agents) stages, commits, and pushes `app/index.html`, `app/styles.css`, `app/project-data.json`, and `.vscode/launch.json`.

## File assignments

| File | Owner | Notes |
|---|---|---|
| `app/project-data.json` | **Coder** (data shape chosen by Coder; content informed by Designer's labeling/vocabulary needs, e.g. status/priority value sets used for badge styling) | Must exist before HTML/JS can render real cards; Designer needs sample values early to design badge variants |
| `app/index.html` | **Coder** (structure/markup/fetch logic), with **Designer** reviewing/adjusting markup for semantic HTML, ARIA labels, heading hierarchy, and card markup needed for styling hooks | Coder owns the `<script>` fetch/render logic; Designer may propose/request specific class names or data attributes for styling and accessibility (e.g., `aria-label`, `role="status"` on badges) |
| `app/styles.css` | **Designer** (full ownership of visual design) | Must include `.dashboard` and `.project-card` selectors, `border-radius`, `box-shadow`, responsive layout, status/priority badge treatments, and accessible contrast |
| `.vscode/launch.json` | **Coder** (sole owner) | Strict JSON, no comments; name must be exactly "Run Project Pulse Dashboard"; serves `app/`; opens `index.html` via `serverReadyAction` |

## Designer responsibilities

- Own all of `app/styles.css`: layout system (`.dashboard` grid/flex), `.project-card` component styling, spacing scale, typography, color palette, and responsive breakpoints.
- Define status and priority badge visual language (color, shape, icon/text) that is distinguishable without relying on color alone (accessibility requirement — add text labels, not just color).
- Specify/request semantic HTML and ARIA attributes in `app/index.html` (e.g., `<main class="dashboard">`, `<section class="project-card" aria-label="...">`, heading levels, `role="status"` or `aria-live` regions if status is dynamic) — Designer proposes, Coder implements the actual markup edits to avoid two people editing the same file concurrently.
- Verify color contrast (WCAG AA) for text on badges/cards, verify keyboard focus visibility, and verify the layout is usable at narrow viewport widths (mobile-first or at least responsive down to ~375px).
- Ensure the first-paint view "clearly looks like a Project Pulse dashboard frontend," not a bare unstyled page.
- Report design decisions and any markup change requests back to the Orchestrator/Coder rather than editing `index.html` directly, to keep file ownership unambiguous.

## Coder responsibilities

- Own `app/index.html` markup: document structure, exact title/heading text "Project Pulse", `<link>` to `styles.css`, script that `fetch`es `app/project-data.json`, parses the `projects` array, and renders one `.project-card` element per project showing `name`, `owner`, `status`, `recentActivity`, and `priority`.
- Own `app/project-data.json`: top-level `"projects"` key, consistent field names/types across all entries, enough sample projects (e.g., 4–6) to demonstrate varied statuses/priorities for Designer's badge styling.
- Own `.vscode/launch.json`: strict JSON (no comments), configuration named **Run Project Pulse Dashboard**, `cwd` = `${workspaceFolder}/app`, `command`/`program` = `python3 -m http.server 5500` (or equivalent `preLaunchTask`/`type":"node-terminal"` setup consistent with VS Code's documented `serverReadyAction` pattern), `serverReadyAction.uriFormat` = `http://localhost:%s/index.html`.
- Implement all JS behavior: fetch error handling (e.g., if `project-data.json` fails to load, show a visible error state rather than a blank dashboard), and apply whatever class names/data attributes the Designer specifies for styling hooks.
- Validate JSON syntax, validate the page renders cards (not raw JSON) when opened through the launch configuration, and validate the launch configuration opens `index.html` and not a directory index.

## Dependencies between steps

1. `app/project-data.json` schema (step 1) must exist/stabilize **before** `app/index.html`'s render logic is finalized (step 2), since the script's field names depend on the JSON shape.
2. `app/index.html` markup/class names (step 2) must exist **before** `app/styles.css` can target real selectors like `.project-card` meaningfully, although Designer can start a design system/mockup in parallel using the class-name contract agreed up front (`.dashboard`, `.project-card`).
3. `.vscode/launch.json` (step 4) is independent of the HTML/CSS/JSON content — it only needs to know the final directory is `app/` and the entry file is `index.html` — so it can be authored in parallel with steps 1–3, but final validation (step 5) requires all four files to exist together.
4. End-to-end validation (step 5) depends on steps 1–4 all being complete.
5. Commit/push (step 6) depends on validation passing.

## Parallel work decisions

**Can run in parallel (no file conflicts):**
- Coder drafting `app/project-data.json` sample data **and** Coder/Designer agreeing on the class-name/ARIA contract for `app/index.html` (a short written contract, not simultaneous edits to the same file).
- Designer building out `app/styles.css` against an agreed-upon class-name contract **while** Coder finalizes `app/index.html` fetch/render logic — as long as both reference the same already-agreed selector names (`.dashboard`, `.project-card`) to avoid rework.
- Coder authoring `.vscode/launch.json` at any time in parallel with Designer's CSS work and the HTML/data work, since it has no content dependency on the other three files beyond their final directory/filenames.

**Must run sequentially (same file, avoid conflicting edits):**
- Only one agent should edit `app/index.html` at a time. Recommended order: Coder writes the initial markup/fetch skeleton → Designer reviews and requests specific class/ARIA changes → Coder applies them. Do not have Designer and Coder edit `index.html` simultaneously.
- Final integration/validation (step 5) must happen only after all files are in their current, agreed-upon state — run it last, sequentially, after parallel work converges.
- Commit/push (step 6) is strictly last, after validation passes.

## Validation expectations

- **JSON validity:** `python3 -m json.tool app/project-data.json` and `python3 -m json.tool .vscode/launch.json` both succeed (launch.json must be strict JSON with no comments).
- **Data shape:** `app/project-data.json` has a top-level `projects` array; every project object includes `name`, `owner`, `status`, `recentActivity`, and `priority`.
- **Markup checks:** `app/index.html` contains the exact text "Project Pulse"; contains a `<link ... href="styles.css">` (or equivalent) reference; contains a `<script>` or fetch call referencing `project-data.json`; contains elements with class `project-card`; renders `status`, `recentActivity`, and `priority` values somewhere in the visible DOM per card.
- **Style checks:** `app/styles.css` contains `.dashboard` and `.project-card` selectors and includes both `border-radius` and `box-shadow` declarations; responsive rules (e.g., media queries or CSS grid `auto-fit`/`minmax`) are present.
- **Launch config checks:** `.vscode/launch.json` includes a configuration literally named `Run Project Pulse Dashboard`; its working directory serves from `app/`; it opens `http://localhost:%s/index.html` via `serverReadyAction` (confirmed via the repo's own `scripts/validate-exercise.sh` keyphrase checks, which look for exactly these strings).
- **Manual run:** In VS Code, open **Run and Debug**, select **Run Project Pulse Dashboard**, press play; confirm the browser opens `app/index.html` (styled dashboard with cards) — not a bare file/directory listing — then stop the server afterward.
- **Accessibility spot-check:** Tab through the page to confirm cards/interactive elements are keyboard-reachable; confirm status/priority are conveyed with text, not only color; confirm contrast is readable.
- **No regressions to template structure:** `scripts/validate-exercise.sh` in this repo encodes many of the above checks (file existence, keyphrases like `.dashboard`, `.project-card`, `border-radius`, `box-shadow`, `index.html`, `Run Project Pulse Dashboard`, JSON parse of `launch.json`) — running it locally after implementation is a good final gate, understanding it will also fail on earlier steps until files 1–4 all exist (by design, since this is the per-step grading script for the exercise).

## Edge cases to handle

- **Fetch over `file://` protocol:** Opening `index.html` directly by double-click (not through the launch configuration's HTTP server) will likely fail `fetch()` for `project-data.json` due to CORS/file restrictions in some browsers — document that the dashboard must be viewed via the **Run Project Pulse Dashboard** launch configuration (or any local HTTP server), not by opening the file directly.
- **Malformed or missing JSON:** If `project-data.json` is empty, malformed, or missing the `projects` key, `index.html`'s script should show a visible, readable error/empty state instead of a blank page or uncaught exception.
- **Inconsistent field casing/typos:** Field names must match exactly (`recentActivity`, not `recent_activity` or `RecentActivity`) between JSON and the render script — a single source-of-truth schema should be agreed before both Coder and Designer build against it.
- **Unknown/unmapped status or priority values:** CSS badge styling should have a sensible default style for any status/priority value not explicitly enumerated (e.g., a neutral/gray fallback badge) so new data doesn't render unstyled.
- **Port conflicts:** Port 5500 in `launch.json` may already be in use in the Codespace; note this as a known risk (the learner may need to stop a previous server before relaunching).
- **Duplicate/variable project counts:** Rendering logic must not hardcode a fixed number of cards; it should map over however many projects exist in the JSON array.
- **Long text overflow:** Long `name`/`recentActivity`/`owner` strings should wrap or truncate gracefully in `.project-card` rather than breaking the card layout.
- **No JavaScript fallback:** Since this is a learning exercise, a `<noscript>` message is a nice-to-have but not required; at minimum avoid silent total blank-page failure.

## Open questions

- Should `app/project-data.json` include the brief's mentioned "short contributor-friendly summary" field even though it isn't in the required five-field list used by validation? (Recommendation: include it as an optional extra field so the brief's full intent is honored without breaking required-field checks.)
- Should the HTML/JS be split into a separate `app/app.js`/`app/script.js`, or should the fetch/render script remain inline in `app/index.html`? Both satisfy the brief; inline keeps the file count minimal and matches the brief's listed three app files exactly (no additional JS file is explicitly required), but a separate file is cleaner structurally. Recommend inline `<script>` in `index.html` to avoid introducing an unassigned file.
- Exact `launch.json` configuration `type`/`request`/`preLaunchTask` mechanics for triggering `python3 -m http.server 5500` plus `serverReadyAction` — VS Code's built-in debug config types don't have a single first-class "serve a static folder and open a URL" type; this typically requires either a Node.js/`pwa-node` task that shells out to `python3 -m http.server`, or a `"type": "node-terminal"`/`"type": "chrome"` combo with `serverReadyAction`. The Coder should confirm the exact working configuration in this VS Code/Codespace version before finalizing, since `serverReadyAction` historically requires a debug session that stays alive (e.g., wrapping the python server launch in a Node task) — test this concretely rather than assuming any one JSON shape works out of the box.
- Should status/priority be free-form strings or constrained to an enum (e.g., `status: "On Track" | "At Risk" | "Blocked"`, `priority: "High" | "Medium" | "Low"`)? Constraining them makes badge styling simpler and more deterministic — recommend agreeing on a fixed small enum before Designer finalizes badge CSS.
- How many sample projects are "enough" to demonstrate the dashboard convincingly without being excessive for a learning exercise? Recommend 4–6.
