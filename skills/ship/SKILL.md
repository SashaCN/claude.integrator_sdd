---
name: ship
model: inherit
effort: medium
agents: []
description: >
  Use to close the loop after review — verify the feature actually works, write the changelog /
  knowledge-base note, and open the pull request. Triggers on "ship {slug}", "open a PR for {slug}",
  "changelog for {slug}", "prepare {slug} for merge", "/sdd:ship {slug}", "відправ фічу {slug}",
  "створи PR для {slug}", "changelog для {slug}". Re-runs the gate, runs the app/feature to confirm
  the spec's outcomes for real (not just green tests), drafts a changelog + PR body that link spec/
  AC/ADRs, and proposes the PR command for whatever forge the repo uses. Never auto-merges to main.
---

# Skill: ship

The closing step. `review` confirmed the change is correct on paper; `ship` confirms it **works in reality** and packages it for merge. The loop ends here: a reviewed, verified change with a changelog and an open PR — not a merge to main (that stays a human decision).

Forge-agnostic and stack-agnostic: the verification commands are detected the way `implement` detects them; the PR step targets whatever forge the remote points at (GitHub via `gh`, GitLab via `glab`, or copy-paste).

> **Paths.** Every `docs/…` path below resolves under `docs_root` (`.claude/sdd.local.md`; default `docs`). The durable docs the auto-hook updates are the committed `docs/architecture/` · `docs/ecosystem/` · `docs/integrations/<Op>/`; the planning trail (`docs/features/<slug>/`, `docs/roadmap.md`, `docs/architecture-map.md`) is gitignored; ADRs (`docs/adr/`) are committed → [`../_shared/paths.md`](../_shared/paths.md).

## Owner

The implementer (drives) + the reviewer who signed off in `review`.

## Inputs

- `<slug>` — feature slug.
- **Gate (hard refuse):** a `PASS` review record (`docs/features/<slug>/_review/`) or, at minimum, an implemented + gate-green change. No review yet → «run `review <slug>` first».
- Read: `spec.md` (what to claim in the changelog), the feature's Accepted ADRs in `docs/adr/` (decisions worth recording), `tasks.json` (the AC satisfied), and the feature's single commit.

## Protocol

1. **Final verification — does it actually work.** Re-run the detected gate (unit + integration where available + lint + vet). Then **run the feature for real** against its acceptance criteria — not just "tests pass": start the app / hit the endpoint / exercise the flow and observe the spec's outcomes (e.g. the default-on read returns defaults; an invalid value is rejected). If the app can't be run here (no runtime, no Docker), say so explicitly and record what was verified vs deferred — never claim verified-working when only tests compiled.
2. **Write the changelog / KB note.** From [`./templates/changelog.md`](./templates/changelog.md): what changed, why (link spec + the key ADRs), any migration/operational note (e.g. "adds migration 000023 — run it on deploy"), and how to use it. Partner-facing if the change is partner-facing.
3. **Prepare the PR.** The work should be on a feature branch (`feature/{JIRA_KEY}-…`, not the default branch) — if it isn't, **tell the dev to create/switch it**; never run `git checkout`/`git branch` yourself (ai-operating-policy §1). Draft the PR body from [`./templates/pr-body.md`](./templates/pr-body.md): summary, **a link to the Jira ticket**, the AC it satisfies (from `tasks.json`), the test + verification evidence, and any migration/rollback note. **Never mention AI** in the PR title, body, or commits. The **spec/sad/tasks live under the gitignored `docs/features/<slug>/`** — reference them via Jira or a one-line summary, not as in-repo links. **ADRs are different — they are committed in `docs/adr/`**, so link them directly.
4. **Detect the forge + propose the PR command.** Inspect the remote (read-only): `github.com` → propose `gh pr create`; `gitlab.com`/self-hosted GitLab → `glab mr create`; otherwise print the branch + body for manual creation. **Only propose** the command — the agent **never runs `gh`/`glab`, never pushes, never opens a PR** (ai-operating-policy §2: no remote/GitHub interaction); PRs are exclusively the developer's, and you never merge to main.
5. **Update the roadmap.** Move this feature's item to **Shipped** in `docs/roadmap.md` (via `roadmap`) — date + outcome + link to the feature folder + the PR/changelog — and remove it from **Now**. This is the anti-drift hook: delivery itself keeps the roadmap current. (No roadmap yet → skip; it's optional.)
6. **Update the living documentation (auto-hook).** Run the `document` logic → [`../document/SKILL.md`](../document/SKILL.md): classify whether the shipped change makes a durable doc **wrong or incomplete**, then **update** the stale doc(s) or **create-on-ship** a doc for a gap (a significant touched area with none) in the right `docs/` taxonomy folder, and — for a **cross-boundary contract** — record it in **this repo's** `docs/ecosystem/` (no fan-out to sibling repos). All dispatched to `sdd:doc-writer`, **proposed and approved, never silently written, left unstaged** (the same `auto_commit: off` + §1 discipline). No-op when there's no `docs/` tree or the change is trivial. This is the anti-drift hook for the **durable** docs — the sibling of step 5's roadmap update for the **process** docs. → [`../_shared/paths.md`](../_shared/paths.md).
7. **Summary (terminal handoff).** **Emit the stage-handoff block** per [`../_shared/handoff.md`](../_shared/handoff.md) (terminal variant) — *What I did* (verification result: verified-working / what was deferred and why; the roadmap update; the living-doc update result — docs synced/created / no-op reason) + *Review* (the changelog path + the proposed PR command + any `docs/…` docs updated, left unstaged for your commit) + *Run next* = **Done**: the PR command (the dev runs it) — merging to main is your call; there is no `/sdd` successor.

## Definition of Done

- The gate was re-run and the feature was exercised against its AC (or the deferral was stated explicitly with the reason).
- A changelog / KB note exists, linking spec + ADRs.
- A PR body is prepared and the forge-appropriate PR command **proposed** (the dev runs it; `gh`/`glab` never invoked, main untouched, §2).
- The durable docs were checked against the shipped change and either updated / created-on-ship in the right `docs/` taxonomy folder (proposed + approved + left unstaged, with a cross-boundary contract recorded in this repo's `docs/ecosystem/`) or an explicit no-op was stated — the `document` auto-hook ran (it never silently writes the durable docs, never commits).

## Anti-patterns

- **"Tests pass" ≠ "it works".** Run the actual feature against the spec's outcomes; green unit tests don't prove the wired system behaves.
- **Claiming verified when you only compiled.** If the runtime/Docker wasn't available, say what was deferred — don't overstate.
- **Mentioning AI in the PR or commits.** Team rule: no AI mention in titles, descriptions, or commit messages.
- **Auto-merging to main / pushing to a shared remote unasked.** Propose the PR; the merge is the team's call.
- **A changelog that restates the diff.** Say what changed and why (link the spec + ADR), plus the operational note (migrations, flags) — not a file list.
- **Forgetting the migration/rollback note** when the change includes one — the deployer needs it.

## References & template

- [`./templates/changelog.md`](./templates/changelog.md) — changelog / KB-note scaffold.
- [`./templates/pr-body.md`](./templates/pr-body.md) — PR description scaffold.
