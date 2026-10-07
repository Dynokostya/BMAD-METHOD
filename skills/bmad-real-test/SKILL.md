---
name: bmad-real-test
description: 'Tests a change by running the project test suites, going past unit tests to the real tiers (integration, end-to-end, API, contract) whenever they can run locally. Fixes failures the change caused and reports what ran, what passed, and what could not run and why. Use when the user says "run real tests", "run the integration tests", "run the e2e tests", "verify this against the real API", or "test this change for real", or when a build passes its plan file after committing.'
---

If `{project-root}/_bmad/scripts/platform-overrides.md` exists, read it and apply every section whose gating tools exist in your toolset for the rest of this run.

Run the project's tests against a change, going past unit tests to the real tiers (integration, e2e, API, contract) whenever they can run locally, and report exactly what ran, what passed, and what could not run and why. Whoever reads the report decides from it whether the change is safe to ship, so the report must match what actually happened.

## Inputs

- **Plan file** (optional, `{plan_file}`): a build plan. The change under test is everything since its frontmatter `baseline_revision`. Its `## Verification` section may hold a `**Commands:**` list and `**Manual checks**`.
- **Headless mode** (optional): the caller runs unattended. Never ask anything; every decision falls to the conservative choice.
- **No plan:** the change is the uncommitted diff plus any commits the person names. With no diff at all, test the project as it stands.

Before running anything, note whether the working tree is clean.

## Rules

- Run tests, never fake them. Record only results you observed.
- Never weaken, skip, or delete a test to go green.
- Stay local. Never touch staging, production, or any other remote environment, never use real credentials or paid third-party calls, and never mutate shared state. A tier that needs any of these is Blocked.
- A Blocked tier is reported with its reason and is never a reason to stop.
- Write no new test suites. When the change lacks coverage, say so and suggest invoking the `bmad-qa-generate-e2e-tests` skill.

## 1. Find the test tiers

Infer how the project tests from its files; do not assume. Look at:

- Task runners and scripts: `package.json` scripts, `Makefile`, `justfile`, `pyproject.toml`, `tox.ini`, `Cargo.toml`, `go.mod`, `*.csproj`, `composer.json`.
- Test directories and config: `tests/`, `e2e/`, `integration/`, `cypress/`, `playwright.config.*`, `conftest.py`, `jest.config.*`, `vitest.config.*`, `*_test.*`, `*.spec.*`.
- Service dependencies: `docker-compose*.yml`, testcontainers, `.env.test`, fixtures that boot a database or API.

Sort what you find into **unit/smoke** (no external dependencies) and the real tiers, **integration, e2e, api, contract** (a real database, HTTP API, message broker, browser, or external service). Also note configured lint, type-check, and build commands.

## 2. Decide what can run

A tier is Runnable when its command exists and its dependencies can be satisfied on this machine. Local, reversible setup is allowed: start a compose stack or testcontainers, run migrations, load seeds into a local database. When setup would reach beyond this machine, or you are unsure that it would not, the tier is Blocked.

When the change touches an HTTP API, the project has no runnable API test tier, and the server can start locally, add an `api-smoke` tier: start the server, call the changed endpoints (for example with curl), check status codes and response shape, then stop it.

## 3. Run

Prefer one subagent for the runs so heavy output stays out of your context. It returns, per tier, the command, the result, failing test names with a short excerpt, and the reason for any tier it could not bring up. Wait for its result before going on. Without subagents, run inline.

Run in this order:

1. The plan's Verification commands, exactly as written, each judged against its stated success criteria. Perform its manual checks by inspection.
2. Unit/smoke tiers.
3. Every Runnable real tier, then `api-smoke`.
4. Lint, type-check, and build.

Do not run a tier or check again when a plan command already ran it.

## 4. Fix what the change broke

For each failure, decide whether the change caused it.

- **Caused by the change:** fix it in place and re-run the tiers that failed, for at most 3 fix rounds. A test whose expectation is genuinely wrong may have that expectation corrected, never deleted.
- **Pre-existing:** it also fails without the change. Report it as pre-existing and leave it. When the diff does not settle this, run the test without the change, for example in a temporary worktree at the baseline that you remove afterwards.
- **Cannot fix:** still failing after 3 rounds, or you cannot tell whether the test or the change is wrong. Headless: record it. Otherwise ask the person how to proceed.

## 5. Record

Tear down everything you started and delete the temporary files this run created. Leave every other untracked file alone.

The outcome is a table with one row per tier: tier (`plan` for the plan's own checks), command, result (pass, fail, or blocked), and the failing test names with a short excerpt or the Blocked reason. Below it, list fixes made, pre-existing failures, failures left unfixed, and missing coverage. If no real tier ran (only unit/smoke, lint, type-check, or build ran, or nothing did), say so plainly in the first line. Never imply verification that did not happen.

With a plan, write the outcome under `## Real Test Result` in `{plan_file}`, replacing that section if an earlier run wrote one.

## 6. Commit

If the tree was clean when you started, commit your fixes and the plan record in one local Conventional Commit: `fix(<scope>): ...` when code or tests changed, `test(<scope>): ...` when only the record changed. The tree ends clean. If it was dirty, leave your fixes uncommitted and list them. Never push.

## 7. Report

Lead with one sentence on what happened, then the outcome and the commit hash, if you made one.
