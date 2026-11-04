# Project Pulse final handoff

## Summary

Project Pulse is a static dashboard for Mona's team. It shows one card per project with the project's name, owner, status badge, priority, recent activity, summary, progress and last-updated date. The data comes from `app/project-data.json`. Learners open the dashboard with the VS Code launch configuration **"Run Project Pulse Dashboard"**.

The dashboard was built by a team of four custom agents and committed by the learner as `77c7ab1` "Build the Project Pulse dashboard". The automated checks below all passed. The manual browser, keyboard and contrast checks haven't been done yet.

## Agent roles and what each did

| Agent | Role | What it did |
| --- | --- | --- |
| **Planner** | Research and planning, no code | Wrote `docs/project-pulse-plan.md` with the file assignments and the shared Contracts A–D: A covers the data schema, B the class names, C the runtime behavior and D the launch configuration. |
| **Orchestrator** | Delegation and verification, no implementation | Split the work into phases with explicit file scopes. It ran the Designer and Coder in parallel only where their files didn't overlap. |
| **Designer** | Visual design and accessibility | Built `app/styles.css` in Phase 1. In Phase 3 it reviewed the stylesheet and fixed 6 CSS mismatches. It made no markup requests. |
| **Coder** | Code, data and runnable-app support | Built `app/project-data.json`, `.vscode/launch.json` and `app/index.html` in Phase 1, in that order. |

**Phases:**

1. **Phase 1, build in parallel.** The Designer built `app/styles.css` while the Coder built the data, the launch configuration and the page. Both worked against Contracts A–D.
2. **Phase 2, integration check.** The Orchestrator ran the static checks, contract checks, class coverage check and runtime checks.
3. **Phase 3, design review.** The Designer fixed 6 CSS mismatches in `app/styles.css` only. There were no markup requests, so the Coder had nothing to do in this phase.
4. **Phase 4, final validation.** The Orchestrator re-ran validation. The learner then committed `77c7ab1` "Build the Project Pulse dashboard".

## Files delivered

| File | Owner | Purpose |
| --- | --- | --- |
| `app/index.html` | Coder | The page shell, titled "Project Pulse". It links `styles.css` and includes a `<template>` for each `project-card`. One inline script fetches `project-data.json` with `cache: "no-store"` and renders the cards using `textContent` only. It also shows messages for loading, errors, an empty list and `file://` access. |
| `app/styles.css` | Designer | Design tokens and the `.dashboard` layout. Cards in `.project-card` have `border-radius` and `box-shadow`. Status and priority badges use text labels, not color alone. The grid is responsive through `@media` rules. The stylesheet also covers reduced motion, forced colors and a dark theme. |
| `app/project-data.json` | Coder | A top-level `projects` array with 6 sample projects that use every status and every priority value. |
| `.vscode/launch.json` | Coder | The **"Run Project Pulse Dashboard"** configuration (`node-terminal`). It runs `python3 -m http.server 5500` in `${workspaceFolder}/app` and opens `http://localhost:%s/index.html`. |

## How to run

1. Open **Run and Debug** in VS Code (Ctrl+Shift+D / Cmd+Shift+D).
2. Select **"Run Project Pulse Dashboard"** and start it (F5).
3. The configuration serves `app/` on port 5500. When the terminal prints `Serving HTTP on … port 5500`, it opens `/index.html` in your browser, so you see the dashboard rather than a directory listing.

**Manual alternative:** run `cd app && python3 -m http.server 5500`, then open `http://localhost:5500/index.html`.

## Dashboard validation

These results come from a fresh run on 2026-11-04 against the committed files.

| # | Check | Result |
| --- | --- | --- |
| 1 | `python3 -m json.tool app/project-data.json` | PASS |
| 2 | `python3 -m json.tool .vscode/launch.json` | PASS |
| 3 | `app/index.html` contains "Project Pulse", "styles.css", "project-data.json", "project-card", "status", "recentActivity" and "priority" (case-insensitive) | PASS (7/7) |
| 4 | `app/styles.css` contains ".dashboard", ".project-card", "border-radius", "box-shadow" and an `@media` rule | PASS (5/5) |
| 5 | `app/project-data.json` contains "projects", "name", "owner", "status", "recentActivity" and "priority" | PASS (6/6) |
| 6 | `.vscode/launch.json` contains "Run Project Pulse Dashboard" and "index.html" | PASS (2/2) |
| 7 | Data contract: all 6 projects have non-empty strings for the 5 required fields. Every status is in On Track/At Risk/Blocked/Completed (all 4 are used) and every priority is in High/Medium/Low (all 3 are used). | PASS |
| 8 | Launch contract: `cwd` = `${workspaceFolder}/app`, `command` = `python3 -m http.server 5500`, `uriFormat` = `http://localhost:%s/index.html` | PASS |
| 9 | Class coverage: all 30 classes have a selector in `app/styles.css`. That's every class in the `index.html` markup plus the status and priority badge modifiers the script builds. | PASS (0 missing) |
| 10 | Runtime: `GET /index.html` returns 200, isn't a directory listing and contains `<title>Project Pulse</title>` | PASS |
| 11 | Runtime: `GET /styles.css` and `GET /project-data.json` return 200 | PASS |
| 12 | The startup line `Serving HTTP on 0.0.0.0 port 5500 (http://0.0.0.0:5500/) ...` matches `Serving HTTP on .* port ([0-9]+)` and captures port 5500. The server was stopped afterwards. | PASS |
| 13 | `node --check` on the extracted inline script (Node v22.15.0) | PASS |

**Not run:**

- **Browser:** the page wasn't rendered in a browser, so the JavaScript has not been run against the real DOM.
- **Launch configuration:** "Run Project Pulse Dashboard" wasn't started from VS Code, so the `serverReadyAction` auto-open hasn't been tested.
- **Visual and layout:** no visual review of the layout, the responsive breakpoints (1 column, then 2, then more) or the dark theme.
- **Keyboard:** visible focus hasn't been checked.
- **Contrast:** no tool has checked color contrast, for example DevTools or axe.
- **Screen reader:** not tested.

## Known risks

- **Port 5500 conflict:** port 5500 is also the Live Server extension's default. If it's in use, the server fails to start and the browser doesn't open. Stop the other server; don't change the port, which is a fixed value from the exercise.
- **Codespaces forwarding:** `openExternally` on `localhost` depends on Codespaces port forwarding. If the browser doesn't open, open port 5500 from the **Ports** panel and add `/index.html` to the URL.
- **Unverified contrast:** the contrast ratios in the `app/styles.css` comments (for example "7:1" and "14:1") are the Designer's estimates. No tool has measured them.
- **Dark mode badges:** in the dark theme, status badges keep their light tinted background with dark text. That keeps the badges' text contrast high, but the light pills stand out strongly on dark cards. The visual balance hasn't been reviewed.

## Learner handoff

**Next steps:**

1. **Browser check.** Start **"Run Project Pulse Dashboard"** and confirm the following:
   - The page shows the title, the summary counts (2 On Track, 2 At Risk, 1 Blocked, 1 Completed) and 6 cards.
   - Every card shows its status, priority, recent activity and owner.
   - The layout goes to 1 column when the window is narrow and to more columns when it's wide.
   - Pressing Tab shows a visible focus indicator.
2. **Contrast check.** Use the DevTools accessibility pane or the axe extension in both light and dark themes. Confirm that the badges and muted text meet WCAG AA (4.5:1).
3. **Commit.** Stage, commit and push `docs/final-handoff.md` yourself; no agent does git operations.

**Open questions from the plan:**

1. **Status words:** do `On Track`, `At Risk`, `Blocked` and `Completed` match the words Mona's team uses?
2. **Priority versus risk:** should there be a separate `risk` field, or is `priority` alone enough?
3. **Optional fields:** should `summary`, `progress` and `lastUpdated` stay in the data? They're included now.
4. **Launch type:** is `node-terminal` acceptable, or should it be a Python (`debugpy`) configuration? That option needs the Python extension.
5. **Phase 3 routing:** should the Designer keep sending markup requests only through the Orchestrator?
6. **Dark mode:** should the `prefers-color-scheme: dark` theme stay in scope? If so, the dark mode badge tradeoff above needs a decision.
