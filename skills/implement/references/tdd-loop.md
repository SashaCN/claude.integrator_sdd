# TDD loop — the per-task cycle (step 8)

Every task runs `SELECT → RED → GREEN → REFACTOR → GATE`; the feature is committed **once**, after all tasks pass (see COMMIT). This is the same cycle whether the runner is the sequential agent, a team `implementer`, or a Workflow stage. The RED step is the load-bearing one — skip its discipline and the whole method collapses into "write code, write a test that happens to pass".

## SELECT

Pick the next task whose `deps` are all `done`. In sequential mode that's the topo order; in parallel modes the orchestrator hands it out. Read the task body + its `acs` from `spec.md §5` + the relevant `test-plan.md` rows. Know, before writing anything, what observable outcome the test will assert.

## RED — write the failing test first

1. Write the test(s) for this task's `acs` **before any production code**. Put them where the repo keeps tests for that layer (detected, not assumed).
2. Run the unit command. Capture the output.
3. **Classify the first run** — this is mandatory and must be stated aloud:

   | Class | What it looks like | Action |
   |---|---|---|
   | **GOOD red** | test compiles, runs, fails on an assertion or «not implemented» | proceed to GREEN |
   | **BAD red** | the test itself won't compile / import-errors / references a symbol that the test got wrong | the test is broken, not the code — **fix the test**, re-run, re-classify |
   | **false-pass** | green on the very first run, before any production code | the test is too weak (asserts nothing real) — **strengthen it** until it's GOOD red |
   | **NON-red** | skipped because its dependency is unavailable (e.g. Docker absent for an integration test) | not a pass and not a fail — record NON-red, governed by `require_integration` |

4. **Quote the failing line** (the assertion + expected-vs-actual, or the «undefined: X» line) before writing any production code. This is the proof that the test exercises the right thing.

A task with only a NON-red integration test and no unit coverage cannot be driven by TDD locally — write the unit-level RED too, and let the integration RED land in CI (the proving-run pattern).

## GREEN — minimal code to pass

Write the **least** code that turns the quoted failing assertion green. No speculative generality, no unrelated edits, nothing outside the task's `files_hint`. Re-run the unit command; confirm the previously-quoted failure is now green and nothing else broke.

## REFACTOR — clean while staying green

Tidy names, extract helpers, remove duplication — re-running the unit command after each change. If a refactor goes red and isn't trivially fixable, **revert it**; the task's job is the GREEN, not the cleanup.

## GATE — the task isn't done until this is clean

Run, per the detected commands + settings:

- **unit** — must be green.
- **integration** — green if available; NON-red recorded if Docker is absent under `require_integration: auto`; BLOCK was already enforced for `always`.
- **lint** (if `gate_lint` and a linter resolved) — clean.
- **vet/typecheck** (if `gate_vet` and a command resolved) — clean.

Any hard-gate failure (unit red, or integration red when it ran, or lint/vet errors) → the task is not done. Fix, or escalate (see [`escalation.md`](./escalation.md)).

## COMMIT — one commit for the whole feature

This fork does **not** commit per task. Each task runs `SELECT → RED → GREEN → REFACTOR → GATE` and updates `tasks/tracker.md` → `done`; committing happens **once**, when every task is gate-clean. Stage the feature's code and **propose** a single commit in the team format:

```
[TASK-NOTIF-42] Add Telegram notification type for order events
```

- `[TASK-{JIRA_KEY}]` for a feature, `[FIX-{JIRA_KEY}]` for a bug fix; `[TASK]` / `[FIX]` when there's no ticket. **Imperative mood.** Derive `{JIRA_KEY}` from the feature branch (`feature/{JIRA_KEY}-…`); if it can't be derived, ask the user.
- **Body only when needed** — non-obvious rationale, a breaking change, follow-ups.
- **No `SDD-Task` / `SDD-AC` trailers and no AI mention.** AC traceability lives in `tasks.json`, which `review` reads.
- **Never auto-commit** (`auto_commit: off`) — propose the `git commit` command and wait for the user's explicit go-ahead, per the team's git-safety rule «never commit without explicit request». (`per_task` / `per_phase` restore the upstream cadence if a project opts in.)

In parallel modes the **lead lands all agents' work onto the feature branch** and proposes that single commit once the whole DAG is gate-clean.
