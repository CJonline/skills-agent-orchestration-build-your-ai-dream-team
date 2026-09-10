# Project Pulse — Final Handoff

## Overview

Project Pulse is a static, dependency-free dashboard for Mona's team. It renders one card per project from a local JSON data file, styled as a polished, responsive UI and launched from VS Code via a one-click debug configuration.

This handoff summarizes the delivered artifacts, how the agent team produced them, how the dashboard was validated, and what a human owner needs to know to take it forward.

## Agent team

The work was coordinated by four custom agents defined under `.github/agents/` and driven through GitHub Copilot CLI in a Codespace:

- **Orchestrator** — Broke the request into phases, assigned disjoint file scopes to the specialists, ran parallelizable work concurrently, and verified the integrated result. Did not write code.
- **Planner** — Researched the repository, the Project Pulse brief, and the specialist agent docs; produced `docs/project-pulse-plan.md` with ordered steps, file assignments, dependencies, edge cases, and validation expectations.
- **Designer** — Owned the visual and accessibility layer. Delivered `app/styles.css` with a tokenized design system, polished card treatment (border-radius, layered box-shadow, hover lift), responsive CSS Grid, WCAG-AA badge palettes, `:focus-visible` outlines, and `prefers-reduced-motion` support.
- **Coder** — Owned structure, data, and tooling. Delivered `app/index.html` (semantic markup + vanilla-JS fetch/render with empty and error states), `app/project-data.json` (top-level `projects` array with the required fields), and `.vscode/launch.json` (the launch configuration described below).

Git operations were kept out of the agents' hands and driven by the human owner via Copilot CLI prompts.

## Delivered artifacts

- `app/index.html` — Semantic dashboard shell. Sets `<title>Project Pulse</title>`, links `styles.css`, fetches `./project-data.json`, and renders one `<li class="project-card">` per project. Every card surfaces the project's `status`, `recentActivity`, and `priority` (plus name, owner, and optional due date). Rendering uses `textContent` to avoid XSS. Empty and error states are accessible (`role="alert"` on the error region).
- `app/styles.css` — Vanilla CSS design system. Includes the required `.dashboard` and `.project-card` selectors, the shared badge hooks (`.status-badge`, `.priority-badge`, and their `--active`/`--at-risk`/`--on-hold`/`--complete` and `--high`/`--medium`/`--low` modifiers), a responsive `repeat(auto-fill, minmax(280px, 1fr))` grid that collapses to a single column under 480px, and accessibility affordances (focus outlines, reduced-motion handling, non-color badge cues).
- `app/project-data.json` — Top-level `"projects"` array with six realistic sample entries. Each entry includes `name`, `owner`, `status`, `recentActivity`, and `priority`; every entry also carries an optional `dueDate` in ISO format. Values use the controlled vocabularies the badge system expects (`Active` / `At Risk` / `On Hold` / `Complete`; `High` / `Medium` / `Low`).
- `.vscode/launch.json` — Strict JSON (no comments, no trailing commas). Contains a single configuration named exactly **Run Project Pulse Dashboard**, which serves the `app/` directory with `python3 -m http.server 5500` (`cwd` set to `${workspaceFolder}/app`) and uses `serverReadyAction` to open `http://localhost:%s/index.html` in a browser as soon as the server prints its ready line — so the learner lands on the dashboard, not a directory listing.

## validation

Performed against the checklist in `docs/project-pulse-plan.md`:

- **File presence.** `app/index.html`, `app/styles.css`, `app/project-data.json`, and `.vscode/launch.json` all exist. `.vscode/tasks.json` was not modified.
- **Data contract.** `app/project-data.json` parses as strict JSON. The top-level key is `projects` (array). Every entry contains `name`, `owner`, `status`, `recentActivity`, and `priority`, and the sample data covers all status and priority values the badge system supports.
- **HTML contract.** `app/index.html` uses the exact title `Project Pulse`, links `styles.css`, fetches `project-data.json`, uses the class name `project-card` for each card, and visibly renders each project's `status`, `recentActivity`, and `priority`. User-supplied strings are inserted with `textContent`.
- **CSS contract.** `app/styles.css` defines both the `.dashboard` and `.project-card` selectors and gives cards `border-radius`, layered `box-shadow`, and a responsive grid layout with a single-column fallback on narrow viewports.
- **Launch contract.** `.vscode/launch.json` is strict JSON, contains the configuration named exactly `Run Project Pulse Dashboard`, invokes `python3 -m http.server 5500`, sets `cwd` to `${workspaceFolder}/app`, and its `serverReadyAction.uriFormat` opens `http://localhost:%s/index.html` — so the dashboard frontend loads directly.
- **Edge cases considered.** Empty `projects` arrays render an accessible empty state; fetch or parse failures render a visible `role="alert"` error region; unknown status or priority values fall through to a neutral badge instead of crashing; long strings wrap gracefully; the `prefers-reduced-motion` media query disables hover animation.

## handoff notes for the next owner

- **Run it locally.** Open the repository in VS Code (or a Codespace), pick **Run Project Pulse Dashboard** in the Run & Debug panel, and the browser will open on `http://localhost:5500/index.html`. Do not open `app/index.html` directly from the filesystem — `fetch('./project-data.json')` is blocked on `file://` origins in most browsers, which is exactly why the launch configuration is provided.
- **Edit the data.** To add, remove, or update a project, edit `app/project-data.json`. Keep `status` within `Active` / `At Risk` / `On Hold` / `Complete` and `priority` within `High` / `Medium` / `Low` so the badge styling matches; unknown values still render safely but fall back to a neutral badge.
- **Extend the UI.** New card fields should be added by the Coder role in `app/index.html` (markup + render logic) and styled by the Designer role in `app/styles.css` using the existing token system in `:root`. Keep the CSS hook contract (`.dashboard`, `.project-card`, `.project-card__title`, `.project-card__meta`, `.status-badge`, `.priority-badge`, `.recent-activity`, `.dashboard__empty`, `.dashboard__error`) so the two files stay in sync.
- **Port conflicts.** If port 5500 is already in use, update the port in `.vscode/launch.json` (`runtimeArgs`) — the `serverReadyAction.pattern` reads the port from the server's own output, so the browser URL will follow automatically.
- **Accessibility.** The design deliberately pairs color with text and shape (badge dots, priority star, uppercase treatment) so it does not rely on color alone. Preserve this when adding new states.
- **Out of scope.** No build step, bundler, framework, or network fonts were introduced. Keep the app vanilla unless there is a strong reason to add tooling; that keeps the launch story a single `python3 -m http.server 5500`.

Once this handoff is reviewed, the delivered branch can be merged and Mona's team can start driving Project Pulse from real data.
