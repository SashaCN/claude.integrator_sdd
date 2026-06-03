---
name: document
model: inherit
effort: medium
agents: [doc-writer]
description: >
  Use to keep the team's durable documentation true to the shipped reality — the living-documentation
  step. After a feature ships (or ad-hoc), it checks whether the change is significant enough to make a
  durable doc under `docs/` wrong or incomplete, and if so dispatches doc-writer to update — or, on a
  gap, create — the doc(s) in the existing `docs/` taxonomy (architecture / ecosystem / integrations /
  standards).
  Triggers on "document {slug}", "update the docs for {slug}", "sync docs {slug}", "/sdd:document {slug}",
  "оновити доку для {slug}", "задокументувати {slug}", "синхронізувати доку". Runs the significant-change
  heuristic against the feature's diff + spec/sad/adr, locates the affected doc (docs-first in the
  taxonomy, explorer fallback), documents a cross-boundary contract in this repo's `docs/ecosystem/`,
  and PROPOSES the update — it never silently writes, and leaves the edit unstaged. `ship` invokes this
  same logic automatically at completion; this skill is the standalone entry. No-op when `docs/` is
  absent or the change is trivial.
---

# Skill: document

The living-documentation step. The pipeline already writes its **process** artifacts (spec / sad /
tasks / review) under the gitignored `docs/features/<slug>/`; this skill keeps the **durable**
source-of-truth docs — the committed `docs/architecture/`, `docs/ecosystem/`, `docs/integrations/<Op>/`
(and, rarely, `docs/standards/`) — from drifting out of sync with what actually shipped. It is this
fork's analog of the team `end` skill's doc-update step — minus the chat-log (the
`docs/features/<slug>/` artifacts + the handoff trail are the build narrative; there is no separate
chat to log).

> **Paths.** All `docs/…` paths resolve under `docs_root` (`.claude/sdd.local.md`; default `docs`) →
> [`../_shared/paths.md`](../_shared/paths.md). Durable docs are **committed**; the process trail is
> gitignored. With `docs/` absent (a bare greenfield repo with no docs tree yet) this skill creates the
> folder on a genuine gap, else no-ops.

**Auto + standalone.** `ship` runs this logic automatically at feature completion (the anti-drift
hook). You also call it directly — `/sdd:document <slug>` — to sync docs ad-hoc without a full ship.

## Owner

The implementer who shipped the change (drives) + the doc owner for the touched component (the durable
doc is theirs). The skill detects and drafts; the human approves and commits the doc write (the AI
never commits — ai-operating-policy §1).

## Inputs

- `<slug>` — feature slug.
- **Gate (no-op, not refuse):** a `docs/` tree. Present → proceed. Absent and the change is trivial →
  say «no docs tree and nothing significant to capture» and emit the handoff; on a genuine significant
  gap in a greenfield repo, create the relevant `docs/<area>/` folder rather than inventing a vault.
- The **shipped change** — the feature's diff (`git show` / `git diff`, read-only) and the files it
  touched.
- The **WHY** — `docs/features/<slug>/spec.md` (§1 Context, §5 AC), `sad.md`, Accepted `docs/adr/` — so
  the doc update records the decision, not just the diff.
- The **durable docs** under `docs/` that cover the touched component / process.

## Protocol

1. **Resolve root.** Read `docs_root` from `.claude/sdd.local.md` → [`../_shared/paths.md`](../_shared/paths.md). Locate the `docs/` tree.
2. **Gather the change + the WHY.** Read the feature's diff and the touched files; read `spec.md` / `sad.md` / Accepted `docs/adr/` for the rationale. This is the WHAT (diff) + WHY (artifacts) the doc must reflect.
3. **Significant-change classification.** Decide whether the change warrants a durable-doc update — the heuristic from the team `end` skill:
   - **Significant** (a doc may now be wrong): new logic or a notable change to existing behavior (new class / method / job / mechanism, a different contract, a different data flow); a new mediator / provider / integration; a new process, interaction rule, or convention; an architecture or integration-point change; a notable public-API / contract / config change; a conscious non-obvious decision (the kind an ADR records).
   - **Not significant** (leave docs alone): a trivial bugfix with no behavior change, an internal rename, formatting, a typo, a test-only change with no logic change.
   - **Nothing significant → no-op:** emit the handoff («no significant change — docs already reflect reality») and stop.
4. **Locate the affected doc — docs-first in the taxonomy, then explorer.** For each significant change, find the durable doc that covers the touched code. **Search the taxonomy first** — the folder that owns the subject ([`../_shared/doc-method.md`](../_shared/doc-method.md): a mediator/integration change → `docs/integrations/<Op>/`; an internal architecture change → `docs/architecture/`; a cross-service contract → `docs/ecosystem/`) and that folder's README index. **Fall back to [`explorer`](../../agents/explorer.md)** (`subagent_type: "sdd:explorer"`, docs-first discipline) when the right doc isn't obvious, asking it to return the **exact `docs/…` path(s)** — or report a **gap** (a touched area with no doc). → [`../_shared/agent-roster.md`](../_shared/agent-roster.md).
5. **Decide per doc — update / leave / create (the gap → create-on-ship tree).** A doc is touched only if the shipped change makes it **wrong or incomplete** — a stale claim, or a missing component/flow/rule the change introduced. Three outcomes: **update** (doc exists + now wrong/incomplete → fix the stale section in place); **leave** (doc exists + still accurate → untouched); **create-on-ship** (a **gap** — a significant touched area with no doc → create the doc now in the right taxonomy folder, to `doc-method`, and add it to the folder README index — not merely report it). List, per doc, the outcome + what the update/creation should say.
6. **Propose, then dispatch doc-writer (never silently write).** Present the planned updates — which `docs/…` files, what changes, and why (cite the shipped change) — and **wait for the user's go-ahead** (ai-operating-policy: confirmation before changes; the durable docs are committed for the whole team). This mirrors `implement`'s `auto_commit: off` — the stage is automatic, the **write** is approved, and the **commit is the dev's** (the AI leaves the edit unstaged, §1). On go-ahead, dispatch [`doc-writer`](../../agents/doc-writer.md) (`subagent_type: "sdd:doc-writer"`, clean-isolated) per doc — inline the change, the WHY, the exact `docs/…` path, and what to update/create; it writes only under `docs/` in clean Markdown with relative links, matching the nearest sibling, and updates the folder README index. For a **cross-boundary contract** change, the canonical record is **this repo's** `docs/ecosystem/` (the events catalog / inter-service-communication doc) — doc-writer updates it here; there is **no fan-out** to sibling repos. (If `sdd:doc-writer` is unavailable, fall back to a `general-purpose` Agent with the same brief + the boundary rules.)
7. **Handoff (utility variant).** **Emit the stage-handoff block** per [`../_shared/handoff.md`](../_shared/handoff.md) — *What I did* (the docs updated / created, or the no-op reason) + *Review* (each `docs/…` file touched, with the one-line change, **left unstaged for your commit**) + *Run next* (resume — back to `/sdd:ship <slug>` if called mid-ship, else «done»). `/clear` optional (recommend only if context is large).

## Definition of Done

- The significant-change classification ran against the **actual** shipped diff + the spec/sad/adr WHY (not a guess).
- Every durable doc the change made stale/incomplete was updated (or created on a gap) by `doc-writer` under `docs/`, **after** the proposed update was approved — clean Markdown, relative links, matching siblings, folder README index updated; unverifiable claims under the doc-writer's «Unverified».
- OR an explicit, stated no-op: no `docs/` tree + trivial change, or no significant change, or the docs already reflect reality.
- A **gap** (a touched area with no doc) was **created on ship** in the right taxonomy folder — not merely reported; a **cross-boundary contract** change was recorded in this repo's `docs/ecosystem/`.
- No code and no `docs/features/<slug>/` process artifact was touched — only durable `docs/` docs. The edits are left **unstaged** (no `git add` / `git commit`).

## Anti-patterns

- **Documenting a trivial change.** A rename / format / typo / test-only change is noise — the heuristic exists to skip it.
- **Silently writing the durable docs.** The write is proposed and approved; never push doc changes unasked (the auto-commit-off parallel), and never stage or commit them (§1).
- **Touching the process artifacts** under `docs/features/<slug>/` (`spec.md`, `sad.md`, `tasks.json`, `_review/`) or `docs/architecture-map.md` — those are the pipeline skills' gitignored output, not this stage's. `docs/roadmap.md` is committed but is the `roadmap` skill's domain — don't touch it here either. This stage updates only the **durable** committed knowledge docs (architecture / ecosystem / integrations / standards).
- **Creating a duplicate** instead of updating the stale section of an existing doc — `doc-writer` updates in place.
- **Writing unverified claims as fact** — anything not confirmed against code goes under «Unverified», never buried in prose.
- **Inventing a separate vault or a new top-level folder** — durable docs slot into the existing taxonomy (`architecture/` / `ecosystem/` / `integrations/<Op>/`); there is no external vault.
- **Reporting a gap instead of creating the doc** — a significant touched area with no doc is created on ship (gap → create-on-ship), not deferred.
- **Fanning a contract out to another repo** — cross-boundary contracts are canon **here**, in `docs/ecosystem/`; the repo is self-sufficient.

## References

- [`../_shared/paths.md`](../_shared/paths.md) — `docs_root` resolution + the committed-vs-gitignored two-layer model.
- [`../_shared/agent-roster.md`](../_shared/agent-roster.md) — the `doc-writer` (and `explorer`) contract + dispatch ids.
- [`../../agents/doc-writer.md`](../../agents/doc-writer.md) — the librarian that writes `docs/`.
- [`../ship/SKILL.md`](../ship/SKILL.md) — the auto-hook that invokes this logic at completion.
- [`../_shared/doc-method.md`](../_shared/doc-method.md) — the doc method the created/updated docs follow.
- [`../survey/SKILL.md`](../survey/SKILL.md) — reconciles the durable docs against the code + bootstraps missing docs.
