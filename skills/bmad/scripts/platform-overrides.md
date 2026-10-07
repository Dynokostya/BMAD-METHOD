# Platform Overrides

These overrides adapt BMad workflows to the tools your coding agent provides. Each section is gated on tool names, not on the product you run in: apply every section whose tools exist in your toolset and skip the rest without comment. An applied section covers every later step of the current workflow run and adds to the workflow's own instructions.

Where the running workflow's own text forbids something, the workflow wins over these sections. For example, bmad-build-auto forbids background or detached subagents, and review lens prompts forbid a reviewer from spawning subagents of its own.

## Structured questions

Applies when you have any of these tools, called the question tool below: `AskUserQuestion` (Claude Code), `request_user_input` (Codex), `askQuestions` or `vscode/askQuestions` (GitHub Copilot in VS Code), `ask_user` (GitHub Copilot CLI).

- Ask every question through the question tool, not as plain text in your reply. This covers HALT checkpoints, clarification requests, numbered option menus, and any step that says to ask the user.
- Batch related but independent questions into one call.
- `AskUserQuestion` takes up to 4 questions per call. When the options differ as code, config, or layouts, put each option's version in its `preview` field so the user can compare them.
- In Codex, `request_user_input` works only in Plan mode. In Default mode the tool can still be listed, but the call is rejected with "request_user_input is unavailable in Default mode", so ask in plain text there.
- If a question-tool call is rejected or returns an error, ask in plain text for the rest of the run.

## Plan tracking

Applies when you have any of these tools, called the plan tool below: `update_plan` (Codex), `todos` or `manage_todo_list` (GitHub Copilot in VS Code).

- When the workflow's tasks become known (from a spec, ticket, or story file, or from a plan you made), register the full list with the plan tool.
- Update the list as you progress: mark items done and add items you discover.
- The plan tool adds to the workflow's own checkbox or frontmatter tracking and does not replace it. Keep both current.
- Register one entry per major step or work item, not one per tool call.

## Subagent delegation

Applies when you have any of these tools:

- `Agent` (Claude Code): spawn subagent_type `Explore` for read-only investigation and `general-purpose` for writes and reviews.
- `spawn_agent` with `wait_agent` (Codex): spawn role `explorer` for read-only investigation and `worker` for writes. For batch fan-out over many similar items, use `spawn_agents_on_csv`.
- `runSubagent` or `agent/runSubagent` (GitHub Copilot in VS Code).
- `task` (GitHub Copilot CLI).

Rules:

- When a workflow says to isolate exploration in subagents, delegate to subagents, use subagents or tasks where available, or similar, spawn subagents. These directives are not optional, and doing the work inline does not satisfy them.
- Spawn independent subagents together in one message rather than one after another. With `spawn_agent`, spawn them all first, then collect results with `wait_agent`.
- When the host has a custom agent configured whose description matches the type of file being written, prefer it over a generic writer such as `general-purpose` or `worker`.
- Brief each subagent as a colleague who has not seen this session: the goal, target paths, acceptance criteria, project conventions, and what to return.
- Ask each subagent to return a distilled summary, so raw exploration output stays out of your context.
- Do the work yourself instead of spawning when it is a single-file edit already in your context, a typo or rename, or anything one direct tool call finishes.
