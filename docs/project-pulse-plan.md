# Project Pulse implementation plan

## Summary

Project Pulse is a small static dashboard for Mona's team. It shows one card per project, and each card has the project's name, owner, status, recent activity, priority, a short summary for contributors and progress. The card data comes from `app/project-data.json`. Learners open the dashboard from a VS Code launch configuration named **Run Project Pulse Dashboard**. That configuration runs a static server from `app/` and opens `index.html`, so the learner sees the dashboard instead of a directory listing.

**Research findings that shape this plan:**

- **`.github/project-pulse-brief.md`** lists the required files and fields. It requires a top-level `projects` array, and each project needs `name`, `owner`, `status`, `recentActivity` and `priority`. The brief also asks for a short summary for contributors, plus status badges and readable spacing.
- **`.github/steps/3-step.md`** contains the build prompt. It fixes these values:
  - The page title is exactly "Project Pulse".
  - Each card uses the class `project-card`.
  - The stylesheet has `.dashboard` and `.project-card` selectors and uses `border-radius`, `box-shadow` and a responsive layout.
  - The launch command is `python3 -m http.server 5500`.
  - `serverReadyAction` opens `http://localhost:%s/index.html`.
- **`.github/workflows/3-step.yml`** grades Step 3 by checking that files exist and contain certain phrases. The checks are case-insensitive. Every row in "File assignments" below must be satisfied.
- **`.github/workflows/4-step.yml`** grades a later file, `docs/final-handoff.md`. That file is out of scope here, but the validation notes from this plan should feed into it.
- **`app/` is empty** and **`.vscode/` contains only `tasks.json`**, so every file in this plan is new. Nothing has to be merged.
- **Agent rules** (`.github/agents/*.agent.md`):
  - The Orchestrator delegates with explicit file scopes. It runs agents in parallel only when their files don't overlap and neither needs the other's output.
  - The Coder owns code, logic and runnable-app support, including `.vscode/launch.json` as strict JSON.
  - The Designer owns visual design and accessibility.
  - No agent stages, commits or pushes. The learner does all git operations.

## File assignments

| File | Owner | Purpose | Strings the Step 3 grader requires (case-insensitive) |
| --- | --- | --- | --- |
| `app/index.html` | **Coder** (structure, accessibility attributes and the class names agreed with the Designer, all listed below) | Page shell, links to the stylesheet, loads the data, renders the cards | `Project Pulse`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity`, `priority` |
| `app/styles.css` | **Designer** | All visual design: layout, cards, badges, priority treatment, responsive behavior | `.dashboard`, `.project-card`, `border-radius`, `box-shadow` |
| `app/project-data.json` | **Coder** | Sample project data with a top-level `projects` array | `projects`, `name`, `owner`, `status`, `recentActivity`, `priority`; must parse as JSON |
| `.vscode/launch.json` | **Coder** | The **Run Project Pulse Dashboard** launch configuration | `Run Project Pulse Dashboard`, `index.html`; must parse as strict JSON |

No file has two owners. The Designer does not edit `app/index.html`. If the Designer wants markup changes, they go to the Coder as a request (see Phase 3).

## Shared contracts (decided up front so agents can work in parallel)

These contracts are fixed in this plan. Because both agents build against them, the Designer and Coder can work at the same time without reading each other's files.

### Contract A: data schema for `app/project-data.json`

- The top level is an object with one key, `projects`, whose value is an array of 5–6 projects.
- **Required fields** on every project, all strings:
  - `name`
  - `owner`
  - `status`
  - `recentActivity`: a one-sentence description of the latest activity
  - `priority`
- **Optional fields**, rendered when present:
  - `id`: a slug string
  - `summary`: a short string for contributors
  - `progress`: an integer from 0 to 100
  - `lastUpdated`: an ISO date in `YYYY-MM-DD` format
- **`status` values:** exactly one of `On Track`, `At Risk`, `Blocked`, `Completed`. The sample data should use all four.
- **`priority` values:** exactly one of `High`, `Medium`, `Low`. The sample data should use all three.
- **Example shape, for reference only:**

  `{ "projects": [ { "id": "docs-refresh", "name": "Docs Refresh", "owner": "Mona", "status": "On Track", "priority": "High", "recentActivity": "Merged the new contributor guide", "summary": "…", "progress": 70, "lastUpdated": "2026-11-02" } ] }`

### Contract B: class names shared by `app/index.html` and `app/styles.css`

**Page structure:**

- `body` contains `<main class="dashboard">`.
- `.dashboard-header` holds the `h1` with the text "Project Pulse" and a subtitle, `.dashboard-subtitle`.
- `.dashboard-summary` holds optional count tiles, each a `.summary-stat`, showing the number of projects per status.
- `.dashboard-message` is the loading, error and empty-state area. It has `role="status"` and `aria-live="polite"`.
- `.project-grid` is the card container. Use a `<section>` with an `aria-label`, or a list.

**Each card is `<article class="project-card">` with these attributes:**

- `data-status`: the status as a slug (`on-track`, `at-risk`, `blocked` or `completed`)
- `data-priority`: the priority as a slug (`high`, `medium` or `low`)

**Inside each card:**

- `.project-card__header` contains:
  - the project name, an `h2` with class `.project-name`
  - a `.status-badge` plus one modifier: `.status-badge--on-track`, `.status-badge--at-risk`, `.status-badge--blocked`, `.status-badge--completed` or `.status-badge--unknown`
- `.project-owner`
- `.priority-badge` plus one modifier: `.priority-badge--high`, `.priority-badge--medium`, `.priority-badge--low` or `.priority-badge--unknown`. It shows a visible text label such as "High priority", not color alone.
- `.project-activity`: the label "Recent activity" followed by the `recentActivity` text
- `.project-summary`: optional
- `.project-progress`: a native `<progress>` element or an element with `role="progressbar"`. It must have an accessible label and a visible percentage.
- `.project-updated`: optional, a `<time datetime>` element

### Contract C: runtime behavior of `app/index.html`

- Use one inline `<script>` and no extra JS file, because the scope is exactly these four files.
- **Loading:** call `fetch("project-data.json", { cache: "no-store" })`.
- **Validation:** check that `projects` is an array. Render cards with `textContent` only, never `innerHTML`, so data can't inject markup.
- **Static markers:** include a `<template id="project-card-template">` containing `<article class="project-card">`. This keeps the card markup and field names literally present in the HTML. It also gives the Designer a fixed structure to style.
- **Messages:**
  - A `<noscript>` message for browsers without JavaScript.
  - A friendly error in `.dashboard-message` if the fetch fails. It tells the user to open the page with **Run Project Pulse Dashboard** rather than as a local `file://` page.

### Contract D: `.vscode/launch.json`

The file is strict JSON with no comments and no trailing commas: `"version": "0.2.0"` and one configuration.

| Key | Value |
| --- | --- |
| `name` | `"Run Project Pulse Dashboard"` |
| `type` | `"node-terminal"` (built into VS Code, so the Python extension isn't needed) |
| `request` | `"launch"` |
| `command` | `"python3 -m http.server 5500"` |
| `cwd` | `"${workspaceFolder}/app"` |
| `serverReadyAction.pattern` | `"Serving HTTP on .* port ([0-9]+)"` (a character class avoids JSON backslash escaping) |
| `serverReadyAction.uriFormat` | `"http://localhost:%s/index.html"` |
| `serverReadyAction.action` | `"openExternally"` |

## Ordered implementation steps

### Phase 0: Plan approval (sequential)

1. **Orchestrator:** Save this plan as `docs/project-pulse-plan.md` (the learner commits it). Confirm Contracts A–D before building. Nothing else starts until the contracts are accepted.

### Phase 1: Build in parallel (Designer and Coder at the same time)

2. **Designer** creates `app/styles.css` against Contract B:
   - **Layout and theme:** design tokens as CSS custom properties (colors, spacing scale, radius, shadow). A `.dashboard` layout with a maximum width and centered content. A header with a clear "Project Pulse" heading.
   - **Card grid:** `.project-grid` uses `display: grid` with `repeat(auto-fill, minmax(280px, 1fr))` and consistent gaps.
   - **Cards:** `.project-card` has `border-radius`, `box-shadow`, padding and a subtle hover or focus-within lift. Respect `prefers-reduced-motion`.
   - **Status badges:** pill-shaped, with one color per status modifier and a neutral `--unknown` style. Text contrast must be at least 4.5:1.
   - **Priority:** a stronger visual weight for High, such as an accent left border on `.project-card[data-priority="high"]` plus a badge. Medium and Low get progressively quieter styles. Priority must never be shown by color alone.
   - **Remaining parts:** styles for progress, owner, activity, summary and the `.dashboard-message` states.
   - **Responsive layout:** one column below about 600px, and at least two columns on wider screens.
   - **Accessibility and readability:** visible focus styles, a system font stack and a readable line-height.
3. **Coder** creates `app/project-data.json` against Contract A: 5–6 realistic projects covering every status and priority value. Then validate it with `python3 -m json.tool`.
4. **Coder** creates `.vscode/launch.json` against Contract D, then validate it with `python3 -m json.tool .vscode/launch.json`.
5. **Coder** creates `app/index.html` against Contracts B and C:
   - **Head:** `<!doctype html>`, `lang="en"`, a viewport meta tag, `<title>Project Pulse</title>`, a visible `<h1>Project Pulse</h1>` and `<link rel="stylesheet" href="styles.css">`.
   - **Body:** the card `<template>`, the loading, error and empty-state message area, and the inline script.
   - **Script:** fetches `project-data.json`, converts status and priority values to class slugs, renders `status`, `recentActivity` and `priority` for every project, and fills the summary counts.

   Within the Coder, steps 3 → 5 run in order (one agent), but none of them depends on the Designer's output.

### Phase 2: Integration check (sequential, after Phase 1)

6. **Orchestrator:**
   - Run every check listed under "Validation expectations".
   - Confirm that every class name used in `app/index.html` has a matching selector in `app/styles.css`, and that no selector targets markup that doesn't exist.
   - Start the server and confirm the page renders.

### Phase 3: Design review and fixes (sequential, after Phase 2)

7. **Designer** reviews the rendered dashboard for hierarchy, contrast, spacing and responsive behavior. The Designer edits only `app/styles.css`. Markup or accessibility issues go to the Orchestrator as a written list.
8. **Coder** applies any markup requests to `app/index.html` only.

   Steps 7 and 8 can run in parallel only if the Designer's changes don't depend on the Coder's markup changes. Otherwise run step 8 first and finish step 7 afterwards.

### Phase 4: Final validation and handoff (sequential)

9. **Orchestrator:**
   - Re-run all validation.
   - Report the files changed, the validation results and any remaining risks.
   - Remind the learner to stage, commit and push the four files with the message "Build the Project Pulse dashboard".
   - Note that these results feed the later `docs/final-handoff.md` (Step 4). It must mention the Orchestrator, Planner, Coder and Designer, all four files, "Run Project Pulse Dashboard", "validation" and "handoff".

## Dependencies

| Step | Depends on | Reason |
| --- | --- | --- |
| 2 (Designer, `app/styles.css`) | Phase 0 (Contract B) | It styles class names agreed in advance, not the actual HTML file |
| 3 (Coder, `app/project-data.json`) | Phase 0 (Contract A) | The schema and allowed values are fixed in the plan |
| 4 (Coder, `.vscode/launch.json`) | Phase 0 (Contract D) | It needs no other file to exist when written. It only works once `app/index.html` exists |
| 5 (Coder, `app/index.html`) | Contracts A, B, C; best after step 3 | It reads the data fields. Writing the sample data first lets the Coder test rendering right away |
| 6 (Integration) | Steps 2–5 | It needs all four files |
| 7–8 (Review and fixes) | Step 6 | The review needs a working, rendered page |
| 9 (Final validation) | Steps 7–8 | It checks the finished result |

There are two data dependencies: `index.html` reads `project-data.json`, and `index.html` and `styles.css` share class names. Both are handled by the contracts above, not by waiting for the other agent.

## Parallel work decisions

- **Parallel in Phase 1:** the Designer (`app/styles.css`) and the Coder (`app/project-data.json`, `.vscode/launch.json`, `app/index.html`).
  - **Why it's safe:** the two agents touch no files in common. The only links between their work (class names and the data schema) are fixed in Contracts A–D, so neither agent waits for the other's output.
- **Sequential within the Coder:** `project-data.json`, then `launch.json`, then `index.html`. It's one agent, and writing the data first lets `index.html` be tested against real data.
- **Sequential, Phase 1 → Phase 2 → Phase 3:** the integration check and the design review both need the combined result. Fixes in Phase 3 may touch files that are linked through class names, so they run after integration.
- **Not parallel:** the Designer and Coder never edit the same file in the same phase. Markup changes the Designer wants always go through the Coder.

## Validation expectations

**Static checks.** These mirror `.github/workflows/3-step.yml` and are all case-insensitive:

- All four files exist.
- `python3 -m json.tool app/project-data.json` and `python3 -m json.tool .vscode/launch.json` both succeed. That means no comments and no trailing commas.
- `app/index.html` contains `Project Pulse`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity` and `priority`.
- `app/styles.css` contains `.dashboard`, `.project-card`, `border-radius` and `box-shadow`.
- `app/project-data.json` contains `projects`, `name`, `owner`, `status`, `recentActivity` and `priority`.
- `.vscode/launch.json` contains `Run Project Pulse Dashboard` and `index.html`.

**Contract checks:**

- Every project has all five required fields as non-empty strings.
- Every `status` and `priority` value is in the allowed list.
- `launch.json` has `cwd` set to `${workspaceFolder}/app`, `command` set to `python3 -m http.server 5500` and `uriFormat` set to `http://localhost:%s/index.html`.

**Runtime checks:**

- From `app/`, run `python3 -m http.server 5500`.
- `curl -s http://localhost:5500/index.html` returns the dashboard HTML (not "Directory listing for").
- `curl -s http://localhost:5500/project-data.json` returns the JSON.
- Stop the server afterwards.

**Manual checks** (by the learner, or the Designer during review):

- Run **Run Project Pulse Dashboard** from Run and Debug. The browser opens `/index.html` and shows the title, the summary and one card per project.
- Every card shows its status badge, priority, recent activity and owner.
- **Layout:** one column on narrow screens and several on wide screens.
- **Keyboard:** focus is visible.
- **Contrast:** the badges pass WCAG AA.

**Do not use** `scripts/validate-exercise.sh` **as a gate.** It is a template-maintainer script, and its "learner answer files are not tracked" check is expected to fail once the learner commits these files.

## Edge cases to handle

- **Opening as a local file:** if `index.html` is opened directly, `fetch` fails. Show a clear message in `.dashboard-message` that tells the user to use **Run Project Pulse Dashboard**.
- **Bad data:** if the JSON is malformed or `projects` is missing or not an array, show an error message rather than a blank page.
- **Empty `projects` array:** show an empty state ("No projects yet").
- **Missing or empty optional fields:** omit those elements. If a required field is missing, show "—" rather than "undefined".
- **Unknown status or priority:** fall back to the `--unknown` modifier and still show the raw text.
- **Variations in case or spacing** (for example `"at risk"` versus `"At Risk"`): normalize to a lowercase, hyphenated slug.
- **Out-of-range `progress`:** values below 0, above 100 or non-numeric are clamped to 0–100, or the progress bar is hidden.
- **Long names or activity text:** wrap them with `overflow-wrap: anywhere` and don't break the layout.
- **Data that looks like HTML:** render with `textContent` only.
- **Browser caching:** the server can cache the JSON while the data is being edited. Use `cache: "no-store"`.
- **Port 5500 in use** (it is also the Live Server extension's default): document this as a known risk. Changing the port would break the prompt's fixed values, so keep 5500 by default.
- **Codespaces:** `openExternally` on `localhost` relies on Codespaces port forwarding. Python's server listens on all interfaces by default, so forwarding works.
- **Readiness pattern:** must match Python's "Serving HTTP on 0.0.0.0 port 5500 (…)" line. Avoid `\d` so the JSON needs no backslash escaping.
- **Reduced motion and color-blind users:** respect `prefers-reduced-motion`. Status and priority always have text labels, never color alone.

## Open questions

1. **Status words:** is the vocabulary (`On Track`, `At Risk`, `Blocked`, `Completed`) acceptable, or does Mona's team use other terms?
2. **Priority versus risk:** should `priority` be the only field, or should there also be a separate `risk` field? The brief says "priority or risk level". This plan uses `priority` only to match the grader.
3. **Optional fields:** should the sample data include `summary`, `progress` and `lastUpdated`? The plan recommends yes. The brief asks for a summary for contributors.
4. **Launch type:** is `node-terminal` acceptable, or does the learner prefer a Python debug configuration (`debugpy`, module `http.server`)? That option needs the Python extension, which `devcontainer.json` doesn't install.
5. **Phase 3 overlap:** should the Designer be allowed to propose markup changes directly, or only through the Orchestrator? This plan says only through the Orchestrator, to keep file ownership single.
6. **Dark mode:** is a `prefers-color-scheme: dark` theme in scope, or should it stay light-only for now?
