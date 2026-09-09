# BMAD Workflows

Four paths through the BMM module by task complexity. **Spec-Driven** is the fork-default lean/autonomous path for solo + iterative work; the original **Full** path stays available for team handoffs and large multi-epic projects.

> **Note:** BMad Method upstream defines two paths — Build (the official Phase 4 implementation loop) and the full phased BMM pipeline. The "Medium" path below is a practical composition of anytime tools for tasks that need research but not a full PRD/architecture pipeline. The **Spec-Driven** path rides upstream's `bmad-spec` → `stories.yaml` → `bmad-build-auto` pipeline (this fork's default for lean/autonomous solo work).
>
> A jump-to **[Skill & Command Reference](#skill--command-reference)** listing every skill with a short description is at the bottom.
>
> **Install (v6.12+ layout):** `npx skills add Dynokostya/BMAD-METHOD` — pick the skills and coding tool, include `bmad` — then ask the `bmad` skill to run `bmad setup`. Update with `npx skills update`, then `bmad doctor`.

---

## Fast — Build

For small changes, brownfield additions, utilities. Skips planning entirely.

| Step     | Command       | Description                                                       |
| -------- | ------------- | ----------------------------------------------------------------- |
| 1. Build | `/bmad-build` | Intent → clarified spec → implement → review → verify → present. |

> **Autonomous variant:** `/bmad-build-auto` runs one iteration of an unattended dev loop (plan → implement → verify → review → commit) with no checkpoints — for hands-off execution of an already-clear spec.

---

## Medium — Research + Build

For features that need investigation but not the full PRD/architecture pipeline. Composed from BMad anytime tools.

| Step           | Required | Command             | Description                                                              |
| -------------- | -------- | ------------------- | ------------------------------------------------------------------------ |
| 1. Research    | optional | `/bmad-deep-recon`  | Decision-grade research (technical, domain, market, competitive, …).    |
| 2. Build       | yes      | `/bmad-build`       | Build the change end-to-end from clarified intent.                       |
| 3. Code Review | optional | `/bmad-code-review` | Adversarial multi-lens review of the resulting diff.                     |

---

## Spec-Driven — Spec + Autonomous Dev (fork-default)

**Recommended for solo + iterative work.** Distill intent into a machine-readable spec, break it into an execution-ordered story list, then implement each story — unattended with `build-auto`, or interactively with `build`. Rides upstream's spec-kernel pipeline instead of the Epics → Stories → Sprint cycle.

| Step                              | Required     | Command                             | Description                                                                                                     |
| --------------------------------- | ------------ | ----------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| 1. Research / Brief               | optional     | `/bmad-deep-recon`, `/bmad-product-brief` | Ground the idea first if it's fuzzy.                                                                        |
| 2. **Distill to Spec**            | **required** | `/bmad-spec`                        | Any intent → `SPEC.md` kernel (Why / Capabilities / Constraints / Non-goals / Success signal) + companions.      |
| 3. **Story Breakdown**            | **required** | `/bmad-spec` ("break into stories") | Emit `stories.yaml` — an execution-ordered story list beside `SPEC.md`.                                          |
| 4. Architecture                   | optional     | `/bmad-architecture`                | Add an invariants spine when independently-built parts risk diverging.                                           |
| 5. **Autonomous Dev** (per story) | **required** | `/bmad-build-auto`                  | One unattended plan→implement→verify→review iteration per story; writes a terminal status to the story's spec.   |
| — Interactive alternative         | —            | `/bmad-build`                       | Ship a single change interactively instead of running the auto loop (also takes a spec folder + story id).      |
| 6. Code Review                    | optional     | `/bmad-code-review`                 | Adversarial multi-lens review + triage.                                                                          |

**Cycle:** `bmad-spec` → `SPEC.md` + `stories.yaml` → walk stories top-to-bottom; dispatch each to `bmad-build-auto` (spec folder + story id). Each run plans, implements, verifies, reviews, and writes a terminal status to `stories/<id>-*.md`. Set `spec_checkpoint` / `done_checkpoint` per story in `stories.yaml` for human pauses.

**Orchestration:** `bmad-build-auto` is a **single-story worker** — drive the loop with the `bmad-loop` module (fully unattended), or interactively: *"implement stories 1–N, one `bmad-build-auto` subagent each."*

**Why this path:** the story list is pure execution order and `SPEC.md` is the single machine contract every downstream skill consumes. Each `build-auto` run reads one story, not an epic tree — minimal garbage context, and it can run hands-off.

---

## Full — Complete BMM Pipeline (upstream-default)

For major features, new services, architectural changes, multi-epic work, team handoffs. **All upstream skills, all phases.** Use this when you need the team-coordination affordances of stories and sprint plans.

| Phase | Step                             | Required     | Command                          | Description                                                                       |
| ----- | -------------------------------- | ------------ | -------------------------------- | ---------------------------------------------------------------------------------- |
| plan  | Brainstorm Project               | optional     | `/bmad-brainstorming`            | Facilitated ideation through diverse techniques.                                    |
| plan  | Research                         | optional     | `/bmad-deep-recon`               | Market / domain / technical / competitive research (consolidated).                  |
| plan  | Create Brief                     | optional     | `/bmad-product-brief`            | Nail down the product idea in a brief.                                              |
| plan  | PRFAQ Challenge                  | optional     | `/bmad-prfaq`                    | Working-Backwards stress test — alternative to the brief.                           |
| plan  | **Create / Edit / Validate PRD** | **required** | `/bmad-prd`                      | Facilitated PRD: create, update, or validate.                                       |
| plan  | Create UX                        | optional     | `/bmad-ux`                       | Plan UX patterns and design specifications.                                         |
| plan  | **Create Architecture**          | **required** | `/bmad-architecture`             | The architecture spine of invariants.                                               |
| plan  | **Create Epics & Stories**       | **required** | `/bmad-create-epics-and-stories` | Break requirements into epics and user stories.                                     |
| plan  | **Sprint Planning**              | **required** | `/bmad-sprint-planning`          | Readiness gate (PASS/CONCERNS/FAIL) + sprint status tracking (absorbs the old readiness check). |
| ship  | **Build** (per story)            | **required** | `/bmad-build`                    | Implement each story: clarify → plan → implement → review → verify → present.       |
| ship  | QA Automation Test               | optional     | `/bmad-qa-generate-e2e-tests`    | Generate API/E2E tests for implemented features.                                    |
| ship  | Code Review                      | optional     | `/bmad-code-review`              | Extra adversarial review layer after Build's built-in review.                       |
| ship  | Retrospective                    | optional     | `/bmad-retrospective`            | Evidence-based epic review at epic end.                                             |
| ship  | Correct Course                   | as needed    | `/bmad-correct-course`           | Navigate significant mid-sprint changes.                                            |

**Story cycle:** Sprint Planning → **Build** per story (give it the epic + story id; it syncs `sprint-status.yaml`) → (QA) → (CR) → next story, or Retrospective at epic end.

> Upstream deprecated `create-story` / `dev-story` — Build now owns story execution end-to-end (it compiles epic context and tracks sprint status itself).

**Tip:** For validation workflows (Validate PRD, Code Review), use a different high-quality LLM for independent verification.

---

## Spec-Driven vs. Full — when to pick which

| Decision factor                 | Spec-Driven                              | Full                                     |
| ------------------------------- | ---------------------------------------- | ----------------------------------------- |
| Solo developer or small team?   | ✓ solo or 1–2 people                     | team handoffs, multiple roles             |
| Unattended / autonomous dev?    | ✓ `build-auto` loop over `stories.yaml`  | interactive `build` per story             |
| Formal sprint coordination?     | no                                       | ✓ sprint-status.yaml + epics + stories    |
| Multi-epic project (5+ epics)?  | works                                    | ✓ designed for this                       |
| Minimum ceremony / dev context? | ✓ `SPEC.md` + `stories.yaml` only        | full PRD → epics → stories → sprint       |
| First time using BMad?          | start here                               | switch later if needed                    |

You can move between them. A Spec-Driven project graduates to Full by running `/bmad-create-epics-and-stories` from the SPEC/PRD. A Full project goes lean by feeding its PRD/epics to `/bmad-spec`, which re-distills them into `SPEC.md` + `stories.yaml`.

---

## Anytime Tools

Available regardless of phase.

| Name                 | Command                      | Description                                                                                  |
| -------------------- | ---------------------------- | --------------------------------------------------------------------------------------------- |
| Project Context      | `/bmad-project-context`      | Curate the verified context AI agents load: kernel + knowledge bundle (replaces document-project and generate-project-context). |
| Deep Recon           | `/bmad-deep-recon`           | Decision-grade research: market, domain, technical, competitive, user-voice, academic.        |
| Spec (kernel)        | `/bmad-spec`                 | Distill any intent into the SPEC kernel machine contract.                                     |
| Correct Course       | `/bmad-correct-course`       | Navigate significant changes during execution.                                                |
| Review               | `/bmad-review`               | One skill, many lenses: adversarial critique, edge cases, verification gaps, doc structure, prose copy-edit. |
| Walkthrough          | `/bmad-walkthrough`          | Human-in-the-loop walkthrough of a change/commit/PR (formerly checkpoint-preview).            |
| Forge Idea           | `/bmad-forge-idea`           | Pressure-test an idea via persona interrogation until it hardens.                             |
| Party Mode           | `/bmad-party-mode`           | Multi-agent roundtable for diverse perspectives.                                              |
| Advanced Elicitation | `/bmad-advanced-elicitation` | Push the LLM to reconsider and refine its recent output.                                      |
| Customize BMad       | `/bmad-customize`            | Author customization overrides for installed skills/agents (writes `_bmad/custom/`).          |
| Help / Setup         | `/bmad`                      | Hub: answer BMad questions, recommend the next skill; runs `bmad setup` / `update` / `doctor`. |

**Agents (personas):** load and converse — `/bmad-agent-analyst` (Mary), `/bmad-agent-pm` (John), `/bmad-agent-ux-designer` (Sally), `/bmad-agent-architect` (Winston), `/bmad-agent-dev` (Amelia). The tech-writer persona was retired upstream; use `/bmad-review` structure/prose lenses for editorial passes.

---

## Fork changes

This fork adds **behavior, not extra skills** — its changes live inside the existing upstream skills:

- **Platform Overrides on every skill.** Injected through the customization system: each skill's `customize.toml` ships an `activation_steps_prepend` entry (rendered into the workflow on activation) — agent personas included — and the three skills without that hook (`bmad` hub, customize, advanced-elicitation) carry a `### Platform Overrides` block in SKILL.md. The override text is one file, `skills/bmad/scripts/platform-overrides.md`; `bmad setup` / `bmad doctor` install it to `_bmad/scripts/` with the shared scripts, and skills load it from `{project-root}/_bmad/scripts/platform-overrides.md`. Sections are gated on tool availability, not on the product: AskUserQuestion for all questions, TaskCreate / update_plan tracking, Agent / spawn_agent delegation apply wherever those tools exist and are ignored silently elsewhere.
- **`bmad-build`** — a mandatory real-test **verification gate** (`step-05-verify`, present becomes step 6) after review: detects test infrastructure, tiers tests (unit/smoke vs integration/e2e/api), runs every locally-runnable real tier, and reports blocked tiers honestly. The one-shot route gains the same pass. Plus **structural scope discipline** (task / code-block / heading / Design-Notes signals) in place of a token cap.
- **`bmad-build-auto`** — the implementation verify step runs the same real-test tiers (integration/e2e/api when locally runnable), recording blocked tiers instead of halting; spec templates nudge authors to list real-test commands.

The earlier `*-backlog` skill set was retired in favor of the Spec-Driven path above (upstream's `bmad-spec` → `stories.yaml` → `bmad-build-auto`).

---

## Skill & Command Reference

Every installed skill/command with a short description, grouped by upstream module (`method` / `toolbox`, v6.12+). Codes are the legacy help-menu shortcuts; the `bmad` hub routes by skill name.

### Method — Plan

| Command                          | Code | Description                                                          |
| -------------------------------- | ---- | -------------------------------------------------------------------- |
| `/bmad-product-brief`            | CB   | Nail down your product idea in a brief.                              |
| `/bmad-prfaq`                    | WB   | Working-Backwards PRFAQ challenge to stress-test a concept.          |
| `/bmad-prd`                      | PRD  | Facilitated PRD — create, update, or validate.                       |
| `/bmad-ux`                       | CU   | Plan UX patterns and design specifications.                          |
| `/bmad-architecture`             | CA   | Produce the architecture spine of invariants.                        |
| `/bmad-create-epics-and-stories` | CE   | Break requirements into epics and user stories.                      |
| `/bmad-sprint-planning`          | SP   | Readiness gate + sprint status tracking (also `SS` status action).   |
| `/bmad-project-context`          | PC   | Verified kernel + knowledge bundle context system.                   |
| `/bmad-spec`                     | SPC  | Distill any intent into a SPEC.md contract + companions.             |

### Method — Ship

| Command                       | Code | Description                                                              |
| ----------------------------- | ---- | ------------------------------------------------------------------------ |
| `/bmad-build`                 | BD   | Official Phase 4 loop: clarify → plan → implement → review → verify → present. |
| `/bmad-build-auto`            | —    | One iteration of an unattended (autonomous) dev loop, per story.          |
| `/bmad-code-review`           | CR   | Ad hoc adversarial review of any code change.                             |
| `/bmad-walkthrough`           | CK   | Guided walkthrough of a change/commit/PR (formerly checkpoint-preview).    |
| `/bmad-qa-generate-e2e-tests` | QA   | Generate automated API/E2E tests for implemented code.                    |
| `/bmad-retrospective`         | ER   | Evidence-based epic review at epic end.                                   |
| `/bmad-correct-course`        | CC   | Navigate significant changes during execution.                            |

### Toolbox — Anytime

| Command                      | Code | Description                                                              |
| ---------------------------- | ---- | ------------------------------------------------------------------------ |
| `/bmad`                      | BH   | Hub: answer BMad questions, recommend next skill(s); `bmad setup` / `update` / `doctor`. |
| `/bmad-customize`            | BC   | Author/update customization overrides for skills and agents.              |
| `/bmad-brainstorming`        | BSP  | Core brainstorming — early ideation or when stuck.                        |
| `/bmad-party-mode`           | PM   | Orchestrate multi-agent roundtable discussions.                           |
| `/bmad-advanced-elicitation` | AE   | Push the LLM to reconsider, refine, and improve recent output.            |
| `/bmad-review`               | RV   | Multi-lens review: adversarial, edge cases, verification gaps, structure, prose. |
| `/bmad-forge-idea`           | FI   | Pressure-test an idea until it hardens or dies cheaply.                   |
| `/bmad-deep-recon`           | RS   | Decision-grade research three ways, with claim verification.              |

### Agents (personas)

| Command                   | Persona                              |
| ------------------------- | ------------------------------------ |
| `/bmad-agent-analyst`     | Mary — strategic business analyst    |
| `/bmad-agent-pm`          | John — product manager               |
| `/bmad-agent-ux-designer` | Sally — UX designer                  |
| `/bmad-agent-architect`   | Winston — system architect           |
| `/bmad-agent-dev`         | Amelia — senior software engineer    |

> **Removed upstream (v6.12+):** the v6 shim skills (`/bmad-quick-dev`, `/bmad-dev-auto`, `/bmad-create-story`, `/bmad-dev-story`, `/bmad-sprint-status`, `/bmad-create-prd` / `/bmad-edit-prd` / `/bmad-validate-prd`, `/bmad-create-architecture`, `/bmad-*-research`, `/bmad-document-project`, `/bmad-editorial-review`) no longer exist — call the successor skills named above directly. `/bmad-checkpoint-preview` became `/bmad-walkthrough`; `/bmad-help` became the `/bmad` hub (help + setup / update / doctor). Retired outright: `/bmad-index-docs`, `/bmad-shard-doc`, `/bmad-check-implementation-readiness` (absorbed by sprint-planning), and the tech-writer agent.
