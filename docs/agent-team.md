# Agent team

To build Mona's Project Pulse dashboard, I am using a team of four custom agents, orchestrated with GitHub Copilot CLI running in a Codespace. The Orchestrator breaks the work into phases, delegates to the Planner, Coder, and Designer, and reports the integrated result — none of the agents stage, commit, or push changes themselves.

| Agent | Model | Responsibility | Definition location |
|---|---|---|---|
| Orchestrator | Claude Opus 4.7 (copilot) | Coordinates Planner, Coder, and Designer; breaks requests into phases, assigns file scopes, runs non-overlapping work in parallel, and reports the final outcome. | `.github/agents/orchestrator.agent.md` |
| Planner | Claude Opus 4.7 (copilot) | Researches the repository and dependencies, then produces an implementation plan with ordered steps, file assignments, dependencies, parallelizable work, edge cases, and open questions. Does not write code. | `.github/agents/planner.agent.md` |
| Coder | GPT-5.5 (copilot) | Implements code-oriented tasks within the assigned file scope, including Project Pulse support files such as `.vscode/launch.json`, with clear structure and testable behavior. | `.github/agents/coder.agent.md` |
| Designer | Gemini 3.1 Pro (copilot) | Owns UI/UX, accessibility, and visual design for Project Pulse, producing a polished dashboard with project cards, status badges, and responsive layout within the assigned scope. | `.github/agents/designer.agent.md` |

All coordination happens through GitHub Copilot CLI in a Codespace, which invokes these agents to plan, implement, and style the dashboard end-to-end.
