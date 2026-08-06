# Step 5: Verify

## RULES

- This step runs AFTER review patches are applied — it gates on the code as it actually stands now.
- **Best-effort, never blocking on inability.** Run every check that *can* run locally. If a check cannot run here (no test infra, needs remote/credentials/paid services), skip it and record why — never HALT or ask the human just because something can't run. The only things that loop back or halt are *failures of checks that did run* (see Process) and the loop-limit escalation.
- Tests are run, never faked. Never write a passing result you did not observe. Never mark a command green from inference.
- NEVER weaken, skip, or delete a test to make the suite pass. A failing test is a signal, not an obstacle.
- No push, no remote ops, no `git add`.

## INSTRUCTIONS

### Run Spec Verification Commands

If `{spec_file}` has a `## Verification` section with a **Commands** list, run each command exactly as written and capture pass/fail against its stated success criteria. These are the author's own checks — they are the first gate.

If the section has only **Manual checks** (no CLI), perform them by inspection and record the outcome. If the section is absent, skip — do not invent commands. If a listed command can't run here (missing tool, needs remote/credentials), mark it **Blocked** and record why — don't halt.

### Detect Test Infrastructure

Determine how this project runs tests — infer the framework and the available test tiers from project structure, never assume. Inspect (in order of signal strength):

1. **Task runner / scripts** — `package.json` `scripts` (e.g. `test`, `test:integration`, `test:e2e`, `e2e`), `Makefile` targets, `justfile`, `pyproject.toml`/`tox.ini`, `Cargo.toml`, `*.csproj`, `composer.json`, `go.mod`.
2. **Test directories & config** — `test/`, `tests/`, `e2e/`, `integration/`, `cypress/`, `playwright.config.*`, `*.spec.*`, `*_test.*`, `conftest.py`, `jest.config.*`, `vitest.config.*`.
3. **Service dependencies** — `docker-compose*.yml`, `.env.test`, testcontainers usage, fixtures that boot a DB/API. These mark tests that need real infrastructure.

Classify what you find into tiers:

- **unit / smoke** — no external dependencies. Always runnable locally.
- **integration / e2e / api / contract** — exercise real dependencies (DB, HTTP API, message broker, browser, external service). The user explicitly wants these run when the project supports them locally.

### Decide What's Runnable Locally

For each discovered tier, decide whether it can run **here, now, without remote side effects**:

- **Runnable** — the command exists and its dependencies are satisfiable locally (deps installed, or a compose/testcontainer stack can be brought up, or a local/test DB is reachable).
- **Blocked** — needs credentials, a remote/staging environment, paid third-party calls, or infrastructure not present. Do NOT attempt; do NOT fabricate.

When a real-test tier looks runnable but needs a one-time setup (start a compose stack, run migrations, seed fixtures), bring it up if doing so is local-only and reversible, and tear it down afterward. If setup would mutate shared/remote state or you are unsure, treat the tier as **Blocked** — skip it and record why. Do not halt or ask; a tier that can't run locally is simply reported, not a gate.

### Run

Prefer a subagent for the run so heavy output stays out of the main context; the subagent returns a distilled pass/fail summary with failing-test names and the relevant excerpt. Run inline only when subagents are unavailable in the current runtime.

1. Run **unit / smoke** tiers. These are the baseline regression gate.
2. Run every **Runnable** real-test tier (integration / e2e / api). This is the point of this step — do not stop at unit tests when real ones are available.
3. Record each tier: command, tier, result (pass / fail / blocked), and for failures the failing test names plus a short excerpt.
4. If lint / type-check / build are configured and not already covered by the spec Verification commands, run them too — a broken build fails this gate.

### Classify Failures

Deduplicate failures. Classify each using the SAME taxonomy as `[[bmad-snapshot:step-04-review.md]]`, because a test failure has the same possible root causes as a review finding:

- **intent_gap** — the test encodes a behavior the captured intent never resolved. Root cause inside `<frozen-after-approval>`.
- **bad_spec** — the implementation diverged from the spec, or the spec's stated behavior is itself wrong. Root cause outside `<frozen-after-approval>`.
- **patch** — a trivially fixable defect in the change (off-by-one, wrong import, missing await, bad assertion the code should satisfy). Fixable without human input.
- **defer** — a pre-existing failure not caused by this change (already-red test, flaky infra unrelated to the diff). Collect, don't fix here.
- **reject** — noise (environment quirk that isn't a real defect). Drop silently.

Distinguish **a failing test that is correct** (the code is wrong → patch / bad_spec / intent_gap) from **a failing test that is itself wrong** (the test encodes the wrong expectation → that is a bad_spec-class problem in the change, fix the test as part of the patch — never by deleting the assertion to go green).

### Process (cascading, same discipline as review)

Process in cascading order. `intent_gap` and `bad_spec` trigger a loopback — lower findings are moot since code will be re-derived. Before each loopback, read `{spec_file}` frontmatter `verify_loop_iteration` (missing means `0`), increment it by 1, and write it back. If it exceeds 5, HALT and escalate to the human.

- **intent_gap** — Revert the offending code changes. Loop back to the human to resolve the gap. Once resolved, read fully and follow `[[bmad-snapshot:step-02-plan.md]]` to re-run planning → implement → review → verify.
- **bad_spec** — Before reverting: extract KEEP instructions (what worked and must survive re-derivation). Revert the offending changes. Read the `## Spec Change Log` in `{spec_file}`, respect all logged constraints, and append a new entry recording the triggering test failure, what was amended, the known-bad state avoided, and the KEEP instructions. Then read fully and follow `[[bmad-snapshot:step-03-implement.md]]` to re-derive — the review and verify steps will run again.
- **patch** — Fix it now (correct the code, or correct a genuinely-wrong test expectation). Re-run the affected tier to confirm green. These are the only findings that survive loopbacks.
- **defer** — Append one new entry to `{{.implementation_artifacts}}/deferred-work.md` naming the failing test and why it's pre-existing, using this format. Do not modify existing entries or look for duplicates.
  ```markdown
  - source_spec: `{spec_file}`
    summary: <one sentence>
    evidence: <why this is real>
  ```
- **reject** — Drop silently.

After any patch, re-run at least the tiers that failed. The gate is satisfied only when every **Runnable** tier passes (or its sole remaining failures are classified `defer`/`reject` and the human is informed).

### Record Verification Outcome

Update the `## Verification` section of `{spec_file}` (create it if absent) with an outcome summary:

- Each tier run: command, tier, result.
- **Blocked** tiers: list them with the reason they couldn't run locally, so the human knows what still needs CI or a real environment. Never present a blocked tier as passed.
- Patches applied, items deferred.

If the only meaningful tests are **Blocked** and nothing real could run locally, say so plainly — do not imply the change is verified when it isn't.

## NEXT

Read fully and follow `[[bmad-snapshot:step-06-present.md]]`
