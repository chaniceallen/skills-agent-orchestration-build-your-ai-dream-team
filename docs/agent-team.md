# Agent team

This custom agent team will build Mona's Project Pulse dashboard:

- Orchestrator (Claude Opus 4.7): Coordinates Planner, Coder, and Designer; breaks work into phases, assigns explicit file scopes, runs tasks in parallel or sequentially as needed, and verifies the integrated result. Definition: .github/agents/orchestrator.agent.md

- Planner (Claude Opus 4.7): Researches the repository, docs, and dependencies; produces ordered implementation steps, file assignments, dependencies, edge cases, validation criteria, and open questions. Definition: .github/agents/planner.agent.md

- Coder (GPT-5.5): Implements code within files assigned by the Orchestrator. For Project Pulse, provides runnable app support (e.g., create a strict JSON .vscode/launch.json with cwd set to ${workspaceFolder}/app and open index.html), validates changes, and reports risks. Stays within assigned scopes. Definition: .github/agents/coder.agent.md

- Designer (Gemini 3.1 Pro): Handles UI/UX, accessibility, information hierarchy, interaction flow, and visual design. For Project Pulse, produce a polished dashboard with project cards, status badges, clear priority treatment, deterministic CSS hooks (e.g., .dashboard, .project-card), responsive layout, and accessible typography. Definition: .github/agents/designer.agent.md

All git staging, commits, and pushes are controlled by the learner via Copilot CLI prompts; agents should not stage or push changes.
