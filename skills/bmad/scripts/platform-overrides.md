# Platform Overrides

Behavioral overrides for BMad skills. They are gated on the tools your harness exposes, not on which product you run in: apply every section whose gating tools exist in your toolset and ignore the rest silently. Applied sections cover ALL subsequent steps of the current workflow run and layer on top of the workflow's own instructions.

`bmad setup` installs this file from the `bmad` hub skill's scripts tree; skills load it from `{project-root}/_bmad/scripts/platform-overrides.md`.

---

## structured_ask — when `AskUserQuestion` exists

- NEVER ask questions as inline plain text in your response
- ALL questions to the user MUST use the AskUserQuestion tool — this includes HALT checkpoints, clarification requests, numbered option menus, and any point requiring user input before proceeding
- When options involve visual artifacts (code patterns, configs, layouts, mappings, architecture choices, file structures), use the `preview` field on each option to show inline comparisons so the user can see the difference without imagining it
- When asking about related but independent decisions, batch up to 4 questions in a single AskUserQuestion call
- If a step says "HALT and ask" or "ask the user", that means: use AskUserQuestion, not plain text

## task_tracking — when `TaskCreate` and `TaskUpdate` exist

- When a workflow's tasks become known (from a spec, story file, or mental plan), register each work item via TaskCreate with `pending` status
- Set each task to `in_progress` when starting work, `completed` when done
- This supplements (not replaces) any existing checkbox or frontmatter tracking the workflow already does
- For step-file workflows, register one task per major step or work item — not one per micro-action

## plan_tracking — when `update_plan` exists

- When a workflow's tasks become known (from a spec, story file, or mental plan), register the full task list via `update_plan`
- Update the plan as you progress — mark items done, add items discovered during execution
- This supplements (not replaces) any existing checkbox or frontmatter tracking the workflow already does
- For step-file workflows, register one entry per major step or work item — not one per micro-action

## subagent_delegation — when the `Agent` tool exists

When a workflow says "isolate exploration in sub-agents", "hand to a sub-agent/task", "delegate to sub-agents", or "sub-agents/tasks where available", spawn an Agent. Do not interpret these directives as optional or fall back to inline work — this override removes the fallback.

- For codebase investigation, context gathering, or read-only research, spawn the `Explore` subagent_type. It returns distilled summaries.
- For implementation, file writes, and code review, spawn `general-purpose` or, when one is configured, a specialized writer agent matching the file type: `python-code-writer`, `typescript-code-writer`, `react-code-writer`, `csharp-code-writer`, `swift-code-writer`, `flutter-code-writer`, `prompt-writer`, `diagrammer`. Specialized writers apply project-level coding standards automatically.
- For independent investigations or tasks, spawn concurrent Agents in a single message — multiple Agent tool calls in one response — rather than sequentially.
- Brief each Agent like a colleague without session context: goal, target paths, acceptance criteria, project conventions, and the contract for what to return. Subagents lack the conversation history.
- Instruct spawned agents to return distilled summaries. Raw exploration output should not flood the main context.

Skip Agent spawning when: the work is a single-file edit visible in current context, a typo or rename, or completable in one direct tool call.

## agent_delegation — when `spawn_agent` exists

- When a workflow says "delegate to sub-agents", "isolate exploration in sub-agents", or "use sub-agents/tasks where available", use `spawn_agent` with the appropriate built-in role:
  - `explorer` — read-only codebase investigation, context gathering, evidence collection
  - `worker` — implementation, file writes, code changes
- When multiple independent investigation tasks exist, spawn concurrent `explorer` agents rather than sequential calls
- For batch parallel tasks across similar items, use `spawn_agents_on_csv`
- Instruct spawned agents to return distilled summaries — do not let raw exploration output flood the main context
