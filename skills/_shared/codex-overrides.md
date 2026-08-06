# Codex Platform Overrides

When running on Codex, these behavioral rules override default workflow interaction patterns. They apply to ALL subsequent steps in the current workflow.

**Load condition:** Only load this file when `{project-root}/_bmad/_config/ides/codex.yaml` exists.

---

## plan_tracking — Use update_plan

- When a workflow's tasks become known (from a spec, story file, or mental plan), register the full task list via `update_plan`
- Update the plan as you progress — mark items done, add items discovered during execution
- This supplements (not replaces) any existing checkbox or frontmatter tracking the workflow already does
- For step-file workflows, register one entry per major step or work item — not one per micro-action

## agent_delegation — Use spawn_agent with typed roles

- When a workflow says "delegate to sub-agents", "isolate exploration in sub-agents", or "use sub-agents/tasks where available", use `spawn_agent` with the appropriate built-in role:
  - `explorer` — read-only codebase investigation, context gathering, evidence collection
  - `worker` — implementation, file writes, code changes
- When multiple independent investigation tasks exist, spawn concurrent `explorer` agents rather than sequential calls
- For batch parallel tasks across similar items, use `spawn_agents_on_csv`
- Instruct spawned agents to return distilled summaries — do not let raw exploration output flood the main context
