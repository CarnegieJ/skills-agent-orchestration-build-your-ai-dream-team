# Agent team

This is the custom agent team used to build Mona's Project Pulse dashboard, orchestrated via **GitHub Copilot CLI running inside a Codespace**.

## Agents

### Orchestrator
- **Model:** Claude Opus 4.7 (copilot)
- **Definition:** `.github/agents/orchestrator.agent.md`
- **Responsibility:** Coordinates the Planner, Coder, and Designer agents. Breaks down complex requests into phases, delegates tasks with explicit file scopes, runs independent tasks in parallel, and reports the final integrated outcome.

### Planner
- **Model:** Claude Opus 4.7 (copilot)
- **Definition:** `.github/agents/planner.agent.md`
- **Responsibility:** Researches the codebase, documentation, and dependencies to produce a structured implementation plan. Identifies ordered steps, file assignments, dependencies, parallelizable work, edge cases, and open questions. Does not write code.

### Coder
- **Model:** GPT-5.5 (copilot)
- **Definition:** `.github/agents/coder.agent.md`
- **Responsibility:** Implements code within the file scope assigned by the Orchestrator. Writes clear, testable logic, fixes bugs, and creates support configuration (e.g., `.vscode/launch.json`) needed to run or preview the app.

### Designer
- **Model:** Gemini 3.1 Pro (copilot)
- **Definition:** `.github/agents/designer.agent.md`
- **Responsibility:** Handles UI/UX, accessibility, information hierarchy, and visual design within the assigned file scope. For Project Pulse, produces a polished dashboard with project cards, status badges, priority treatment, and a responsive layout.

## Orchestration note

All agents are orchestrated using **GitHub Copilot CLI in a Codespace**. The CLI acts as the top-level controller, invoking each agent in the appropriate order and passing context between phases.
