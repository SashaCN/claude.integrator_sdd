# Agent roster — model / effort policy + the shared agent contract

> **Reference-only.** Not a skill. Skills and the implement engine read this for the model/effort
> matrix, the override precedence, and the contract every spawned agent follows. The canonical
> agent definitions live in `agents/*.md`; this file is the policy that ties them together.

## The roster (model + effort by role)

Model is chosen by the **kind of work**, not by taste — judgment gets the strongest model, execution gets a balanced one, search/scan gets the cheapest. Effort is the reasoning depth that role needs.

| Agent | Kind of work | `model` | `effort` | Tools |
|---|---|---|---|---|
| `explorer` | brownfield scan / search (read-only) | `haiku` | `low` | Read, Grep, Glob, Bash |
| `test-author` | write the failing test (execution) | `sonnet` | `medium` → `high` on escalation | + Write, Edit |
| `implementer` | green + refactor + gate (execution) | `sonnet` | `medium` → `high` on escalation | + Write, Edit |
| `reviewer` | independent review (judgment) | `opus` | `high` | Read, Grep, Glob, Bash |
| `critic` | coherence critique (judgment) | `opus` | `high` | Read, Grep, Glob |
| `devils-advocate` | ambiguity hunt (judgment) | `opus` | `high` | Read, Grep, Glob |
| `researcher` | competitive / adjacent-solution research (ideation) | `sonnet` | `medium` | Read, Grep, Glob, WebSearch, WebFetch |
| `strategist` | generate the 3 strategic approaches (judgment) | `opus` | `high` | Read, Grep, Glob |
| `analyst` | multi-perspective review of approaches (judgment) | `opus` | `high` | Read, Grep, Glob |
| `doc-writer` | create/update durable docs in `docs/` (librarian / execution) | `sonnet` | `medium` | Read, Write, Edit, Grep, Glob, Bash |

Rationale: judgment quality (review, critique, ambiguity, strategy, multi-perspective synthesis) is where a stronger model pays off; execution (write code/tests to a clear spec) is well served by a balanced model and escalates only when it gets stuck; a read-only scan is cheap. The **ideation trio** (`specify` step 3, gated by the depth dial) follows the same logic: `researcher` is gathering-and-citing work (balanced model + web tools), while `strategist` and `analyst` are judgment (generating real alternatives, synthesizing across lenses) and get the strongest model. (Treat model-by-role as a sound principle — the headline "stronger orchestrator + cheaper workers wins by X%" claim from the multi-agent literature did not survive verification, so we lean on role-fit, not a magic ratio.)

## Dispatching (`subagent_type`)

These agents are **plugin-namespaced**. Spawn each with `subagent_type: "sdd:<name>"` — the id Claude Code registers and shows in the available-agents list — **not** the bare name and **not** an `sdd-…` prefix:

`sdd:explorer` · `sdd:test-author` · `sdd:implementer` · `sdd:reviewer` · `sdd:critic` · `sdd:devils-advocate` · `sdd:researcher` · `sdd:strategist` · `sdd:analyst` · `sdd:doc-writer`

So when a skill says «dispatch the `explorer` agent», the call is `subagent_type: "sdd:explorer"`. If the namespaced agent isn't available at runtime, fall back to the general-purpose (or `Explore`) agent the skill names, passing the same prompt.

## Override precedence (highest wins)

```
env var  >  per-invocation (the Agent call)  >  frontmatter  >  session
```

- **`model`** env: `CLAUDE_CODE_SUBAGENT_MODEL`. Values: `haiku|sonnet|opus|inherit|<full-model-id>`.
- **`effort`** env: `CLAUDE_CODE_EFFORT_LEVEL`. Values: `low|medium|high|xhigh|max|<number>` (`xhigh`/`max` only on Opus 4.8 / 4.7).
- Per-project overrides live in `.claude/sdd.local.md` as `model_<role>` / `effort_<role>` keys (see the implement settings).

> **Caveat (verify on your build).** Some Claude Code builds have reported the `effort:` *frontmatter*
> having no observable runtime effect (GitHub claude-code#43083). The field is documented and we set
> it, but treat the **env path** (`CLAUDE_CODE_EFFORT_LEVEL`) as the reliable lever, and the per-role
> `effort_*` settings keys map to it. If a run feels under-reasoned, set the env var.

## Scale with feature size

Default effort/model scale with the feature `.size` (see [`size-matrix.md`](./size-matrix.md)):

- **XS/S** → keep the roster defaults (cheap; the work is small).
- **M** → roster defaults; escalation handles the hard tasks.
- **L/XL** → bump execution effort to `high` and let judgment agents stay `high`; a cross-module change is where reasoning depth pays off.

A skill/engine that knows the size applies this before dispatch and says so in its banner.

## The shared agent contract (every spawned agent)

1. **Clean, isolated context by default.** A spawned agent does **not** see the parent conversation, tool results, system prompt, invoked skills, or files already read — the **only channel is the Agent prompt string**. So the dispatching skill must inline paths, the draft/diff, and decisions explicitly; the agent re-reads upstream artifacts itself. Only the agent's final message returns. This isolation *is* the "fork" for independent review/critique — fresh eyes are the point.
   - **Fork mode** (`CLAUDE_CODE_FORK_SUBAGENT`, experimental) inherits the full conversation + shares the prompt cache. Use it **only** for a live side-task that genuinely needs the running context — never for `reviewer` / `critic` / `devils-advocate`, whose value is independence.
2. **Worker preamble.** When an orchestrator (the implement team/workflow) delegates, it wraps the task: «execute directly, do not spawn sub-agents, use tools directly, report results with absolute file paths». A subagent cannot spawn subagents, so the lead owns fan-out.
3. **Verify before claiming done.** Before saying "done / fixed / passing": IDENTIFY the command that proves it → RUN it → READ the output → only then claim, with the evidence. Words like "should / probably / seems" are a red flag that verification hasn't run.
4. **Cite or drop.** Read-only judgment agents (reviewer/critic/devil's-advocate) emit only cited findings (`file:line` + the artifact/AC clause). An uncited finding is dropped, not shipped.

## Team-agent merge (this fork)

The SDD roster above is the **base**. The team's own agents (symlinked into `~/.claude/agents/`) were reviewed and their distinctive functions folded into the matching SDD agents — SDD agents were **enriched, not replaced** (no dispatch re-pointing; skills still call `sdd:*`):

- **`explorer` ← `codebase-scout`** — gains the **docs-first** discipline: read the repo's `docs/` + the repo `AGENTS.md` *before* code, cite docs alongside `file:line`, and report doc gaps.
- **`reviewer` ← `reviewer`** — gains **severity tags** (CRITICAL / IMPORTANT / MINOR) and reviews against the team's **documented** standards (`docs/standards/`, repo `AGENTS.md`), repository-code-wins.
- **`doc-writer` ← `doc-writer`** — folded in for this fork's **living-documentation** behavior: the `document` stage (and `ship`'s auto-hook) dispatches `sdd:doc-writer` to **update** a durable doc the shipped change made **wrong or incomplete**, **create** one on a gap, or record a **cross-boundary contract** in this repo's `docs/ecosystem/`. It applies the codified method in [`./doc-method.md`](./doc-method.md) (clean Markdown, relative links, the existing `docs/` taxonomy, README index, EN-only). Adapted from the team agent: its external-vault write-boundary becomes the repo's durable `docs/` folders, and it carries an explicit `effort`. It writes only durable docs — never the gitignored process trail under `docs/features/`, never code, never a commit (leaves edits unstaged).

The team's **`debugger`** (production-incident root-cause from logs / job-state / queues) was **not** folded in: SDD's spec→ship pipeline has no incident-investigation stage. Use it directly, outside this pipeline, when an incident comes up.
