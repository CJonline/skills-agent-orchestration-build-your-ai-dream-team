# Agent team

To build Mona's Project Pulse dashboard, I am using a four-agent custom team defined under `.github/agents/` and orchestrated with GitHub Copilot CLI running in a Codespace.

## Orchestrator

- **Model:** Claude Opus 4.7 (copilot)
- **Responsibility:** Breaks the request into tasks, delegates work to the Planner, Coder, and Designer, assigns explicit file scopes, sequences phases based on dependencies/overlap, and reports the final integrated outcome. Does not implement anything itself.
- **Definition:** `.github/agents/orchestrator.agent.md`

## Planner

- **Model:** Claude Opus 4.7 (copilot)
- **Responsibility:** Researches the repository and relevant docs/dependencies, identifies edge cases and risks, and produces an ordered implementation plan with file assignments, dependencies, and parallelizable work for the Orchestrator to schedule. Does not write code.
- **Definition:** `.github/agents/planner.agent.md`

## Coder

- **Model:** GPT-5.5 (copilot)
- **Responsibility:** Implements code within the assigned file scope, including Project Pulse app logic and support files like `.vscode/launch.json` (configured to run from `app/` and open `index.html`). Validates changes before reporting completion.
- **Definition:** `.github/agents/coder.agent.md`

## Designer

- **Model:** Gemini 3.1 Pro (copilot)
- **Responsibility:** Owns UI/UX, accessibility, information architecture, and visual design for the Project Pulse dashboard — polished project cards, status badges, priority treatment, responsive layout, and CSS hooks like `.dashboard` and `.project-card`.
- **Definition:** `.github/agents/designer.agent.md`

All four agents are coordinated through the GitHub Copilot CLI in a Codespace, which acts as the entry point for invoking the Orchestrator and routing work across the team. None of the agents stage, commit, or push changes — all git operations remain under my control via Copilot CLI prompts.
