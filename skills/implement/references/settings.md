# Settings — `.claude/sdd.local.md` (step 2)

The engine is configured per-project by a plugin-settings file with YAML frontmatter. On first run, **lazy-create** it with the defaults below and tell the user where it is; on later runs, read it.

> **Plugin-wide, not implement-only.** Most keys below configure the `implement` engine, but a few are read by **other skills too**. `interview_depth` is read by the Q&A skills (`specify` / `clarify` / `design`) to pre-select the depth dial. The file is **auto-created with documented defaults the first time any skill needs it** — normally `specify` at the start of the backbone — so the rest of the pipeline finds a real file instead of silently falling back. If for any reason it's still missing, a reader falls back to its own default (medium): there is **no hard ordering dependency** on `implement` having run first.

## Auto-create when absent

Created **automatically** the first time a skill needs it — normally `specify` at the start of the backbone (it ensures the file alongside establishing `.size`), or `implement` if you jump straight to it. **Idempotent:** if the file already exists it is read, never overwritten.

1. If `.claude/sdd.local.md` is absent, write it with **the documented frontmatter below, followed by the «What each key does» section as the file's markdown body** — so the file is self-documenting: every key carries its default, its allowed values, and a plain explanation inline, with no need to open the plugin docs.
2. **Patch `.gitignore`** (create it if absent) to include `.claude/*.local.md` + `.worktrees/` (per-developer config). **Then, when `docs_root` is repo-relative** (the default `docs`), also ignore the SDD **process trail** — `docs/features/`, `docs/architecture-map.md` — this fork's per-feature specs / design notes / tasks / reviews + the architecture map, kept **out of git history** so the code repo's history stays code-only (ai-operating-policy §5: developer-specific plans / tasks / notes are not committed). They are **not lost** — they live in the repo working tree, just gitignored; the dev references a set via its Jira ticket. **Do NOT ignore the durable docs** — `docs/architecture/`, `docs/ecosystem/`, `docs/integrations/`, `docs/standards/`, **`docs/adr/`**, and **`docs/roadmap.md`** are the committed source of truth and stay tracked. See the commit policy in [`../../_shared/handoff.md`](../../_shared/handoff.md) and the two-layer model in [`../../_shared/paths.md`](../../_shared/paths.md). (The `.claude/*.local.md` glob already covers `sdd.local.md`; don't add a redundant explicit line.)
3. Tell the user: «Wrote `.claude/sdd.local.md` with documented defaults — edit it to change how the pipeline behaves.»

## The documented frontmatter

<!-- This block is written verbatim to the top of `.claude/sdd.local.md`; the «What each key does»
     section below becomes the file's body. Keep the inline comments — they list the allowed values. -->

```yaml
interview_depth: medium    # easy | medium | hard — plugin-wide default for specify/clarify/design (see _shared/interview-depth.md)
tdd: true                  # enforce red→green→refactor
team_mode: false           # true → agent team via TeamCreate
workflow_mode: auto        # auto → dynamic Workflow; off → never
max_parallel_agents: 3     # integer ≥1 — fan-out cap for team/workflow modes (1 = sequential)
isolation: worktree        # worktree | inplace (parallel>1 ⇒ forces worktree)
stop_on_red: true          # halt on a red that survives escalation, vs drop-and-continue
max_red_retries: 3         # integer ≥1 — RED→GREEN attempts before escalation
gate_lint: true            # true | false — include lint in the per-task gate
gate_vet: true             # true | false — include vet / static-analysis in the per-task gate
require_integration: auto  # auto | always | never (Docker-probed)
auto_commit: off           # off (default — propose ONE commit, never run it) | per_task | per_phase
branch_strategy: feature   # feature | current
docs_root: docs            # base dir for ALL docs/… paths — committed durable docs + gitignored process trail (see _shared/paths.md). 'docs' = in-repo (the only supported layout)
cmd_test_unit: ""          # empty = autodetect → Laravel: php artisan test --compact tests/Unit
cmd_test_integration: ""   # empty = autodetect → Laravel: php artisan test --compact tests/Feature (sqlite :memory:, no Docker)
cmd_lint: ""               # empty = autodetect → Laravel: vendor/bin/pint --dirty --format agent
cmd_vet: ""                # empty = autodetect → Laravel: vendor/bin/phpstan analyse (only if larastan/phpstan installed)
model_test_author: sonnet     # per-role model (see _shared/agent-roster.md); inherit = session model
model_implementer: sonnet
model_reviewer: opus
effort_test_author: medium    # per-role effort; raised to high on escalation
effort_implementer: medium
effort_reviewer: high
```

## What each key does

- **`interview_depth`** — `easy | medium | hard`. The plugin-wide default for the **Q&A skills'** depth dial (`specify` / `clarify` / `design`), which governs how much each skill decides on its own vs. interrogates you (question volume, autonomy, which ideation analyses run, per-diagram confirm vs. proceed). It only **pre-selects** the recommended option in each skill's opening depth question — the user can still override per run, or pass `--depth=` to skip the question. It does **not** affect AC-completeness (that's a floor at every level). Full semantics → [`../../_shared/interview-depth.md`](../../_shared/interview-depth.md). (Not read by the `implement` engine itself.)
- **`tdd`** — when false, RED is skipped and the engine writes code directly (warns; you lose the safety net).
- **`team_mode` / `workflow_mode`** — feed the decision tree (see [`decision-tree.md`](./decision-tree.md)). `team_mode` wins when both could apply.
- **`max_parallel_agents`** — fan-out cap for team/workflow modes. `1` forces sequential.
- **`isolation`** — `worktree` gives each parallel agent its own git worktree under `.worktrees/`; `inplace` edits the checkout directly and **forces parallelism to 1**.
- **`stop_on_red`** — `true`: a red that survives escalation halts the run. `false`: drop that task, auto-block its dependents, continue other branches.
- **`max_red_retries`** — RED→GREEN attempts before escalation (see [`escalation.md`](./escalation.md)).
- **`gate_lint` / `gate_vet`** — include lint / vet in the per-task gate (skipped gracefully if no command is detected — see [`command-detection.md`](./command-detection.md)).
- **`require_integration`** — `auto`: run integration tests if a Docker daemon answers, else mark NON-red; `always`: BLOCK before dispatch if Docker is absent; `never`: skip the integration tier entirely.
- **`auto_commit`** — `off` (default in this fork, and the only value consistent with ai-operating-policy §1): `implement` does the work, then **proposes a single** `[TASK-{JIRA_KEY}]` commit (or `[FIX-{JIRA_KEY}]`) and waits for your go-ahead — it **never runs `git add` / `git commit` itself** (ai-operating-policy §1: the agent never mutates git; modified files are left unstaged for the dev to review and commit). AC traceability lives in `tasks.json`, not in commit trailers. `per_task` / `per_phase` exist for upstream parity but contradict §1 — leave `off`.
- **`branch_strategy`** — `feature`: the proposed commit assumes a feature branch (the dev creates / switches it — the agent never runs `git checkout` / `git branch`, §1); `current`: assume the current branch. Either way the agent only **proposes**; it never switches branches itself.
- **`docs_root`** — base directory for **every** `docs/…` path the skills name — both the **committed durable docs** (`docs/architecture/`, `docs/ecosystem/`, `docs/integrations/<Op>/`, `docs/standards/`, `docs/adr/`, `docs/roadmap.md`) and the **gitignored process trail** (`docs/features/<slug>/…`, `architecture-map.md`). Resolution: `docs/features/<slug>/…` → `{docs_root}/features/<slug>/…`, etc. Default `docs` is the in-repo layout — the only one this fork supports (there is no separate documentation vault). The two-layer model + which paths are committed vs ignored → [`../../_shared/paths.md`](../../_shared/paths.md).
- **`cmd_*`** — explicit command overrides; non-empty values short-circuit detection (the escape hatch for unusual repos).
- **`model_*` / `effort_*`** — per-role model + effort for the three agents, applied when the engine spawns them (it overrides the agent's frontmatter default). Roster defaults + rationale → [`../../_shared/agent-roster.md`](../../_shared/agent-roster.md). Precedence: env var > this setting > agent frontmatter > session.
  - **Env path:** the engine also exports `CLAUDE_CODE_EFFORT_LEVEL` / `CLAUDE_CODE_SUBAGENT_MODEL` for the dispatch when these keys are set — the reliable lever (see [`agent-roster.md`](../../_shared/agent-roster.md) for why frontmatter alone may not suffice).
  - **`.size` scaling:** the engine raises the default effort for **L/XL** features (execution agents → `high`) before dispatch, and keeps the cheap defaults for **XS/S** — a cross-module change is where reasoning depth pays off. It prints the resolved per-role model+effort in the banner.

## Reading semantics

Unknown keys are ignored (forward-compatible). A missing key falls back to the default above. A malformed file → warn and fall back to all-defaults rather than failing the run.
