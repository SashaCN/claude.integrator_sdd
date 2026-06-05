---
name: fix
model: inherit
effort: medium
agents: [explorer, devils-advocate, test-author, implementer, reviewer, doc-writer]
description: >
  Use for a SMALL, self-contained change that doesn't warrant the full backbone — a bug fix, a copy
  or config tweak, a rename, a tiny behavior adjustment. The express lane: you describe the task in
  prose, `fix` locates the code and sizes it, validates it's genuinely small and unambiguous (asking
  you when it isn't), implements it test-first, runs an independent review, syncs any durable doc the
  change touched, and proposes ONE `[FIX-{JIRA_KEY}]` commit. Triggers on "fix {task}", "small change",
  "quick fix", "hotfix", "tweak {X}", "/sdd:fix {task}", "виправ {задача}", "дрібна зміна", "невелика
  правка", "швидкий фікс", "підправ {X}", "хотфікс". It is gated on SIZE, not on a prior artifact:
  the moment the change proves M+ (touches an API/cross-service contract, needs a DB migration, spans
  ≥2 modules, or is a breaking change for consumers) it REFUSES and routes you to `/sdd:specify` +
  the backbone — `fix` never quietly does a big change. Reuses the plugin's agents (explorer ·
  devils-advocate · test-author · implementer · reviewer · doc-writer); writes no spec/tasks process
  trail. Depth-tunable validity check (--depth=easy|medium|hard); TDD by default with a trivial-change
  escape (pure config/docs/rename, or `tdd: false`).
---

# Skill: fix

The **express lane** for small changes. The backbone (`specify → … → ship`) is the right path for a feature; for a one-line bug fix or a config tweak it is pure overhead. `fix` collapses *validate → implement → review → document → propose-commit* into a single pass, reusing the same agents the backbone uses — so a small change still gets a test, an independent review, and a synced doc, just without the spec/design/tasks artifacts it doesn't need.

It is deliberately the plugin's one **un-gated-on-artifacts** skill: there is no `spec.md` to require. In its place stands a **size tripwire** — `fix` refuses to do anything that isn't genuinely small and sends you to the backbone instead. Everything is stack-agnostic: the test / lint / vet commands are **detected**, never hard-coded (the same cascade `implement` uses).

> **Paths.** Every `docs/…` path below resolves under `docs_root` (`.claude/sdd.local.md`; default `docs`) → [`../_shared/paths.md`](../_shared/paths.md). `fix` writes **no** process trail (no `spec.md` / `tasks.json`); the only files it may write outside code are the durable `docs/` docs the change made stale (via `doc-writer`, left unstaged).

## Owner

Whoever needs a small change shipped — the dev drives; `fix` runs the lane. The four agents ship with the plugin: [`explorer`](../../agents/explorer.md) (locate + size), [`devils-advocate`](../../agents/devils-advocate.md) (deep ambiguity hunt), [`test-author`](../../agents/test-author.md) + [`implementer`](../../agents/implementer.md) (TDD), [`reviewer`](../../agents/reviewer.md) (independent review), [`doc-writer`](../../agents/doc-writer.md) (durable-doc sync). Dispatch each as `sdd:<name>` → [`../_shared/agent-roster.md`](../_shared/agent-roster.md).

## Inputs

- `<task>` — a prose description of the small change. **Required** — if absent, ask for it (don't guess).
- (Optional) `<slug>` — a feature slug, only to scope a durable-doc update or derive the branch ticket. `fix` needs **no** upstream artifact — that's the point.
- (Optional) `--depth=easy|medium|hard` — how hard to probe the task for ambiguity (default: `interview_depth` from settings, else `medium` = light inline).
- Settings: `.claude/sdd.local.md` (gate commands, `tdd`) — auto-created with documented defaults if absent, exactly as `implement` does → [`../implement/references/settings.md`](../implement/references/settings.md).
- **No upstream-artifact gate.** The gate is on **size** (step 2), not on a prior doc.

## Protocol

1. **Settings + detect commands.** Read `.claude/sdd.local.md`; if absent, auto-create it with the documented defaults and patch `.gitignore` (the same template `specify` / `implement` write) → [`../implement/references/settings.md`](../implement/references/settings.md). Then run the stack-agnostic detection cascade (settings override → Makefile → package scripts → language manifests → Docker probe) to resolve unit / integration / lint / vet commands → [`../implement/references/command-detection.md`](../implement/references/command-detection.md). Print what was detected.

2. **Locate & size — the tripwire.** Dispatch [`sdd:explorer`](../../agents/explorer.md) (docs-first: read the repo's `docs/` + `AGENTS.md` before code) to **locate** the code the task touches and **size** the change against the three signals in [`../_shared/size-matrix.md`](../_shared/size-matrix.md) — time-to-merge / new module·API·migration / breaking changes. Classify **XS / S / M+** and report the files with `file:line` citations.
   - **TRIPWIRE — refuse and route to the backbone.** If the change is **M+** by *any* signal — it touches an **API / cross-service contract**, needs a **DB migration**, spans **≥2 modules**, or is a **breaking change for consumers** — `fix` **stops**: «This isn't a small change ({the dominant signal}). Run `/sdd:specify <slug>` and walk the backbone — `fix` won't do a change this size.» It does **not** implement. A genuine **hotfix** that is large stays a human-led backbone change.
   - **Borderline S/M** → one `AskUserQuestion` ([`../_shared/ask-style.md`](../_shared/ask-style.md)) to confirm; **default to bailing** (conservative — a wrongly-fast big change is the failure mode this lane must avoid).
   - The located files + the confirmed size are this skill's gate result — record them.

3. **Validate + clarify (depth-tunable).** Confirm the task is **valid** (the described change makes sense against what `explorer` found — not already done, not contradicting the code, doable as a small change) and **unambiguous**.
   - **easy / medium (default — light inline):** if anything is under-specified (what exactly, which call-site, the expected behavior, the edge case), ask via `AskUserQuestion` ([`../_shared/ask-style.md`](../_shared/ask-style.md)). No ambiguity → proceed, and **state the assumptions ledger** (every reversible call you made) so the dev can veto.
   - **hard:** dispatch [`sdd:devils-advocate`](../../agents/devils-advocate.md) for a clean-context ambiguity hunt over the task + the located code; **resolve** each finding with `AskUserQuestion` or proceed with the assumption stated.
   - **Invalid task → stop, don't fabricate.** If the behavior already exists, the request contradicts the code, or it can't be done as a small change, report that with the `file:line` evidence and stop — never invent work to look busy.

4. **Implement.** One change — single-thread; no `tasks.json`, no DAG, no team/workflow modes (those are `implement`'s, for a multi-task feature). Pick the **test policy** for *this* change:
   - **Has observable logic / behavior change → TDD (default).** Dispatch [`sdd:test-author`](../../agents/test-author.md) for the **RED** step (write the failing test first, run it, **classify the first run** — GOOD red / BAD red / false-pass / NON-red — and quote the failing line), then [`sdd:implementer`](../../agents/implementer.md) for **GREEN → REFACTOR → GATE**. Same cycle as [`../implement/references/tdd-loop.md`](../implement/references/tdd-loop.md). Persistent red → escalate per [`../implement/references/escalation.md`](../implement/references/escalation.md) (stronger model → retry → split → **ask a human** → rollback). **Never weaken a test to make it pass** — if the asserted behavior is wrong, fix the understanding, not the test.
   - **Pure no-logic change, OR `tdd: false` in settings → test optional.** A config value, copy/text, a rename, formatting, a docs-only edit has nothing behavioral to assert. Implement directly; add a test only if one *cheaply* applies, else **state why no test was warranted**. (This is the trivial-change escape — and it honors the `tdd: false` setting; say loudly when the safety net is off.)
   - **Gate after the change:** unit green + (integration if available, else NON-red recorded) + lint + vet clean, per the detected commands.

5. **Independent review (recommended).** Dispatch [`sdd:reviewer`](../../agents/reviewer.md) over the working diff — correctness + the project's documented standards ([`../_shared/coding-standards.md`](../_shared/coding-standards.md), the repo's `docs/standards/` + `AGENTS.md`), repository-code-wins, severity-tagged (CRITICAL / IMPORTANT / MINOR), **cite-or-drop**. For a one-file XS this is a single quick pass. **On a CRITICAL / IMPORTANT finding → loop back to step 4** (no `/clear` — you're iterating) and re-review the changed surface. `fix` does **not** self-certify.

6. **Document (auto-hook — usually a no-op).** Run the `document` logic → [`../document/SKILL.md`](../document/SKILL.md): decide whether the change makes a **durable** doc (`docs/architecture/`, `docs/ecosystem/`, `docs/integrations/<Op>/`, `docs/standards/`) **wrong or incomplete**; if so, dispatch [`sdd:doc-writer`](../../agents/doc-writer.md) to **update** it (or create-on-gap), recording a **cross-boundary contract** in this repo's `docs/ecosystem/` (no fan-out → [`../_shared/paths.md`](../_shared/paths.md)). **Proposed + approved + left unstaged** (`auto_commit: off`, ai-operating-policy §1). A trivial change (rename / format / typo / config with no behavior shift) or no `docs/` tree → explicit **no-op** (the heuristic exists to skip noise).

7. **Propose one commit + hand off.** **Propose** a single commit in the team format `[FIX-{JIRA_KEY}] <imperative summary>` — derive `{JIRA_KEY}` from the branch (`feature/{JIRA_KEY}-…`); if it can't be derived, ask, else use the no-ticket `[FIX]` form. **Never auto-commit, never push, never `gh`** (`auto_commit: off`; §1 — no git mutation, §2 — no remote). No `SDD-*` trailers, no AI mention. Then **emit the stage-handoff block** per [`../_shared/handoff.md`](../_shared/handoff.md) (terminal variant) — *What I did* (the confirmed size, the change, the test result — TDD or why test-optional, the review result, the doc result, the proposed commit) + *Review* (the working diff + any durable `docs/` doc updated, left unstaged) + *Run next* = **Done**: the dev runs the commit; or, if review looped, the re-run note.

## Definition of Done

- The change was **sized** (step 2) and confirmed XS/S — or the tripwire fired and routed to `/sdd:specify` with the dominant M+ signal named, and **nothing was implemented**.
- The task was validated and any ambiguity resolved (inline, or via `devils-advocate` on `--depth=hard`) — or an invalid task was reported and stopped.
- The change is implemented test-first (TDD) — or the trivial-change / `tdd: false` escape was taken **and the reason stated**; the gate (unit + integration-if-available + lint + vet) is clean.
- An independent `reviewer` pass ran (its findings resolved or the loop-back taken); `fix` did not self-certify.
- The durable docs were checked against the change and updated / created (proposed + unstaged) — or an explicit no-op reason was stated.
- **One** `[FIX-{JIRA_KEY}]` commit is **proposed** (never auto-run); the handoff block is the last output.

## Anti-patterns

- **Doing a big change on the fast lane.** The tripwire is not advisory — an API/contract change, a migration, a multi-module or breaking change goes to the backbone, full stop.
- **Skipping the size check** and diving straight into code — you can't know it's small until `explorer` has sized it.
- **Inventing work for an invalid task.** If the behavior already exists or contradicts the code, say so and stop.
- **Code before the test** for a behavioral change (the trivial-escape is *only* for no-logic changes), and **weakening a test** to force green — fix the understanding instead.
- **Self-certifying.** Skipping the `reviewer` pass because "it's small" — small changes break things too.
- **Auto-committing / pushing / opening a PR.** `fix` *proposes* the `[FIX]` commit; git and the remote are the dev's (§1/§2).
- **Writing a spec/tasks trail.** `fix` is artifact-light by design — if a change needs a spec, it wasn't a `fix`.

## References

- [`../_shared/size-matrix.md`](../_shared/size-matrix.md) — the XS/S/M+ signals the tripwire reads.
- [`../_shared/ask-style.md`](../_shared/ask-style.md) — phrasing for the clarify / borderline-size questions.
- [`../_shared/agent-roster.md`](../_shared/agent-roster.md) — the `sdd:<name>` dispatch + model/effort policy.
- [`../implement/references/command-detection.md`](../implement/references/command-detection.md) · [`../implement/references/tdd-loop.md`](../implement/references/tdd-loop.md) · [`../implement/references/escalation.md`](../implement/references/escalation.md) · [`../implement/references/settings.md`](../implement/references/settings.md) — the detection cascade, the TDD cycle, escalation, and the settings file `fix` reuses.
- [`../document/SKILL.md`](../document/SKILL.md) — the living-doc logic the step-6 auto-hook runs.
- [`../_shared/handoff.md`](../_shared/handoff.md) — the closing handoff block (terminal variant).

## Example invocation

> **User:** «/sdd:fix the retry backoff cap is 30s, it should be 60s in BillingClient»
> **Skill:** settings read, commands detected (`go test` / `golangci-lint` / `go vet`). `explorer` → the cap lives in `internal/billing/client.go:88`, single module, no contract/migration/breaking-change → **size S**. Validate: unambiguous (one constant, clear new value) → no questions, assumptions ledger printed. Behavioral change → **TDD**: `test-author` writes a test asserting a 60s cap (GOOD red, quoted), `implementer` flips the constant → GREEN, gate clean. `reviewer` → no findings. `document` → no durable doc references the cap → **no-op**. Propose `[FIX-1234] Raise BillingClient retry backoff cap to 60s` → handoff block (Done — dev commits).
