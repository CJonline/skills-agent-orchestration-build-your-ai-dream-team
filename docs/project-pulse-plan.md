# Project Pulse — Implementation Plan

## 1. Summary

Build a small static frontend dashboard ("Project Pulse") for Mona's team inside this repository. Data lives in `app/project-data.json` (top-level `projects` array), is fetched by `app/index.html` at runtime, and rendered as project cards styled by `app/styles.css`. A `.vscode/launch.json` file provides a **Run Project Pulse Dashboard** configuration that serves the `app/` folder and opens `index.html` so learners see the dashboard UI, not a directory listing.

Work is split across two specialists per the agent definitions:

- **Coder (GPT-5.5)** owns structure, logic, data, and tooling: `app/index.html` (markup + fetch/render script), `app/project-data.json`, `.vscode/launch.json`.
- **Designer (Gemini 3.1 Pro)** owns visual/UX layer: `app/styles.css`, plus advisory input on semantic markup and CSS hook names used in `app/index.html`.

Constraint checks from repo research:
- `.github/project-pulse-brief.md` requires the four files listed, top-level `projects` array, and fields `name`, `owner`, `status`, `recentActivity`, `priority`.
- Coder agent doc explicitly permits creating `.vscode/launch.json`, requires strict JSON (no comments), `cwd: ${workspaceFolder}/app`, and opening `index.html`.
- Designer agent doc requires deterministic CSS hooks such as `.dashboard` and `.project-card`, plus badges, spacing, and responsive layout.
- Existing `.vscode/tasks.json` must not be disturbed; only add `launch.json` alongside it.
- `app/` directory currently exists but is empty — no legacy conventions to preserve inside it.
- The brief adds `dueDate` is *not* explicitly required, but the user prompt mentions it; treat it as an optional additional field (see Open Questions).

## 2. Ordered Implementation Steps

1. **Design direction (Designer)** — Produce a short design spec (no code): information hierarchy for cards, badge color mapping for `status` and `priority`, spacing scale, typography, responsive breakpoints, accessibility rules (contrast, focus states, semantic landmarks, aria labels). Define the exact CSS class hooks to be used so Coder's markup and Designer's CSS align. Minimum hooks: `.dashboard`, `.dashboard__header`, `.project-grid`, `.project-card`, `.project-card__title`, `.project-card__meta`, `.status-badge`, `.status-badge--active|--at-risk|--on-hold|--complete`, `.priority-badge`, `.priority-badge--high|--medium|--low`, `.recent-activity`.
2. **Seed data (Coder)** — Create `app/project-data.json` with a top-level `projects` array of ~5–6 realistic sample entries. Each entry includes `name`, `owner`, `status`, `recentActivity`, `priority` (required by brief) plus optional `dueDate` for the extra field mentioned in the prompt. Use a small controlled vocabulary for `status` (`Active`, `At Risk`, `On Hold`, `Complete`) and `priority` (`High`, `Medium`, `Low`) so Designer's badge classes map cleanly.
3. **Markup + rendering logic (Coder)** — Create `app/index.html`: semantic structure (`<header>`, `<main>`, `<section class="dashboard">`, `<ul class="project-grid">`), a heading "Project Pulse", link to `styles.css`, and an inline `<script>` (or small `<script>` block) that:
   - `fetch('./project-data.json')`
   - parses `projects`
   - renders one `<li class="project-card">` per project using the agreed CSS hooks
   - handles empty array (renders friendly empty state)
   - handles fetch/parse failure (renders an accessible error message)
   Use the exact CSS class hooks from Step 1.
4. **Styling (Designer)** — Create `app/styles.css` implementing the Step 1 spec against the actual class hooks used in Step 3: responsive grid (CSS Grid `auto-fill, minmax(...)`), card visuals (rounded corners, subtle shadow, readable spacing), status/priority badge color system with WCAG-AA contrast, focus-visible states, and a mobile-friendly single-column fallback. No JS changes.
5. **Launch configuration (Coder)** — Create `.vscode/launch.json` with a configuration named exactly **Run Project Pulse Dashboard**. Recommended shape: a Node-based static server (e.g., `npx --yes serve` or `npx --yes http-server`) invoked with `cwd: ${workspaceFolder}/app`, plus a `serverReadyAction` that opens `http://localhost:<port>/index.html` in the browser. Strict JSON, no comments. Do not delete or modify `.vscode/tasks.json`. If the codespace/devcontainer has a preferred static server preinstalled, prefer that; otherwise `npx serve -l 5173 .` is a safe default.
6. **Integration verification (Coder, light-touch)** — Open the launch config, confirm the dashboard renders cards from JSON, confirm no console errors. Adjust only within owned files; escalate CSS issues back to Designer.

## 3. File Assignments per Step

| Step | Owner | Files created/modified | Notes |
|---|---|---|---|
| 1 | Designer | *(spec only, no files)* | Advises on hooks used in `app/index.html` but does not edit it. |
| 2 | Coder | `app/project-data.json` | Sole owner. |
| 3 | Coder | `app/index.html` | Sole owner. Must use hooks defined in Step 1. |
| 4 | Designer | `app/styles.css` | Sole owner. Read-only reference to `app/index.html` for hook names. |
| 5 | Coder | `.vscode/launch.json` | Sole owner. Must not touch `.vscode/tasks.json`. |
| 6 | Coder | (verification; may patch `app/index.html` / `app/project-data.json` / `.vscode/launch.json` only) | Designer re-engaged if `app/styles.css` needs changes. |

Explicit ownership of the four required files:
- `app/index.html` → **Coder**
- `app/styles.css` → **Designer**
- `app/project-data.json` → **Coder**
- `.vscode/launch.json` → **Coder**

## 4. Dependencies Between Steps

- Step 3 (markup) depends on Step 1 (agreed CSS hooks) and Step 2 (JSON schema/field names).
- Step 4 (CSS) depends on Step 1 (design spec) and Step 3 (final class hooks in the DOM).
- Step 5 (launch.json) depends on Step 3 existing (needs `index.html` present to open).
- Step 6 depends on Steps 3, 4, 5.

## 5. Parallel vs Sequential Work

**Can run in parallel:**
- Step 2 (`project-data.json` by Coder) and Step 1 (design spec by Designer) — no file overlap, no data dependency.
- After Step 3 is complete and committed to disk, Step 4 (`styles.css` by Designer) and Step 5 (`.vscode/launch.json` by Coder) can run in parallel — disjoint file scopes and no data dependency.

**Must run sequentially:**
- Step 1 → Step 3 (hooks must be agreed before markup is written).
- Step 2 → Step 3 (schema must exist before render code is written; alternatively field names can be locked in Step 1 and steps run concurrently — see Open Questions).
- Step 3 → Step 4 (Designer styles the actual class hooks Coder emitted).
- Step 3 → Step 5 (launch opens `index.html`; that file must exist).
- Steps 3, 4, 5 → Step 6 (verification).

## 6. Edge Cases to Handle

- Empty `projects` array → render an accessible empty state ("No projects yet").
- `fetch` failure or malformed JSON → render a visible, screen-reader-friendly error region (`role="alert"`), not a silent console error.
- `file://` origin: opening `index.html` directly (double-click) will break `fetch('./project-data.json')` in most browsers due to CORS on `file://`. This is exactly why `.vscode/launch.json` must serve over HTTP. Document this in the launch config's name/comment area (via a `presentation` group label, since JSON comments are disallowed).
- Long project names, long `recentActivity` strings → CSS must handle wrapping/truncation gracefully.
- Unknown `status` or `priority` values in JSON → render a neutral default badge; do not crash.
- Missing optional field (e.g., `dueDate`) → omit the row rather than showing "undefined".
- Port already in use for the static server → pick a fixed uncommon port (e.g., 5173) and mention alternatives in Open Questions.
- Codespaces port forwarding: `serverReadyAction` `openExternally` vs internal simple browser — pick `debugWithChrome`/`openExternally` compatible with the devcontainer.
- Accessibility: color alone must not encode status/priority — pair badges with text labels; ensure focus-visible outlines; sufficient contrast on badge backgrounds.
- Responsive: card grid must collapse to single column below ~480px.

## 7. Validation Expectations

1. **File presence check**: `app/index.html`, `app/styles.css`, `app/project-data.json`, `.vscode/launch.json` all exist. `.vscode/tasks.json` is unchanged.
2. **JSON validity**: `app/project-data.json` parses; top-level key is `projects` (array); each item has `name`, `owner`, `status`, `recentActivity`, `priority`. `.vscode/launch.json` is strict JSON (no comments/trailing commas).
3. **Launch flow**: In VS Code, Run & Debug picker shows **Run Project Pulse Dashboard**. Selecting it starts a static server with `cwd = ${workspaceFolder}/app` and opens `http://localhost:<port>/index.html` (not a directory listing).
4. **Render check**: Dashboard displays a header and one card per project. Each card shows name, owner, status badge, priority badge, and recent activity. No console errors.
5. **Design check**: Cards look like a polished dashboard — rounded corners, spacing, shadows, distinct status/priority colors, responsive grid, visible focus outlines.
6. **Edge case checks**: Temporarily set `projects: []` → empty state appears; temporarily rename JSON file → error state appears with `role="alert"`.
7. **Accessibility spot-check**: Tab through the page; focus is visible; badges have text labels; heading structure is logical (`h1` for page, `h2`/`h3` for cards).

## 8. Open Questions

1. **`dueDate` field**: The brief lists only `name`, `owner`, `status`, `recentActivity`, `priority`. The user prompt additionally mentions "due date." Should `dueDate` be included as an optional field on each project? Current plan: include it optionally. Confirm.
2. **Static server choice**: Preferred approach for `.vscode/launch.json` — Node `npx serve`, `npx http-server`, Python `http.server`, or the VS Code Live Preview extension? Devcontainer capabilities should decide; Node-based `npx serve` is the safest default assumption.
3. **Port**: Any preferred port? Plan assumes `5173`.
4. **Schema lock timing**: Can the JSON field names be finalized during Step 1 (design spec) so Steps 2 and 3 can run in parallel? Current plan sequences 2 → 3 for safety.
5. **Status/priority vocabularies**: Confirm the enumerations (`Active`/`At Risk`/`On Hold`/`Complete` and `High`/`Medium`/`Low`) — these drive both sample data and badge CSS.
6. **Number of sample projects**: Plan assumes 5–6. Confirm.
7. **Framework constraints**: Plan assumes vanilla HTML/CSS/JS (no build step). The brief says "static app," which supports this — confirm no bundler or framework is desired.
