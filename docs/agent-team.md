# Agent team

We will use a four-agent GitHub Copilot CLI team in a Codespace to orchestrate Mona's Project Pulse dashboard work.

- Orchestrator — Model: Claude Opus 4.7 (copilot). Responsibility: coordinate the full workflow, break the request into phases, delegate work to specialist agents, and verify the final result before handoff. Definition: `.github/agents/orchestrator.agent.md`
- Planner — Model: Claude Opus 4.7 (copilot). Responsibility: research the repository, identify dependencies and edge cases, and produce an implementation plan with file ownership, sequencing, and validation expectations. Definition: `.github/agents/planner.agent.md`
- Designer — Model: Gemini 3.1 Pro (copilot). Responsibility: shape the dashboard experience with UX, accessibility, information hierarchy, and visual design for a polished Project Pulse frontend. Definition: `.github/agents/designer.agent.md`
- Coder — Model: GPT-5.5 (copilot). Responsibility: implement the app code and support files in the assigned scope, including Project Pulse UI logic and any required launch configuration. Definition: `.github/agents/coder.agent.md`

The flow is: the GitHub Copilot CLI Orchestrator in the Codespace asks the Planner for a plan, then delegates design and implementation work to the Designer and Coder, and finally validates that the dashboard is cohesive and ready to run. This keeps the work modular while ensuring Mona's Project Pulse dashboard is built as a coordinated team effort.
