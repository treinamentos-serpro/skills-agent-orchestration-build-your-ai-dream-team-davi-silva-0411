# Agent team

I am using GitHub Copilot CLI in a Codespace to orchestrate a team of custom agents that will build Mona's Project Pulse dashboard.

| Agent | Target model | Responsibility | Definition |
| --- | --- | --- | --- |
| Orchestrator | Claude Opus 5.5 (copilot) | Breaks down the request, delegates to specialists with explicit file scopes, runs work in parallel only when scopes don't overlap, verifies the integrated result. Does not implement. | `.github/agents/orchestrator.agent.md` |
| Planner | Claude Opus 5.5 (copilot) | Researches the repo and docs, then returns an ordered plan with file assignments, dependencies, edge cases, and validation expectations. Writes no code. | `.github/agents/planner.agent.md` |
| Coder | GPT-5.5 (copilot) | Implements code and logic within its assigned files, plus runnable-app support such as `.vscode/launch.json` (cwd `${workspaceFolder}/app`, opens `index.html`). Validates before reporting. | `.github/agents/coder.agent.md` |
| Designer | Gemini 3.7 Flash (copilot) | Handles UI/UX, accessibility, and visual design: polished dashboard with `.dashboard` and `.project-card` hooks, status badges, priority treatment, responsive layout. | `.github/agents/designer.agent.md` |

All agents leave git operations (stage, commit, push) to me via Copilot CLI prompts.
