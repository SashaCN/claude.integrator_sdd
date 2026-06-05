# Stage handoff — what every skill prints when it finishes (the output contract)

> **Reference-only.** Not a skill. **Every** skill ends by emitting the handoff block defined here —
> as its **last output**. The format lives only in this file; each
> skill keeps a one-line pointer and supplies its own *What I did* / *Review* / *next command*. This
> exists because a bare «Next: …» line is hard to act on — the user can't tell what changed, which
> files to open, or what to run next without scrolling back.

## TL;DR (короткий вступ українською)

Кожен крок (skill) наприкінці **завжди** друкує однаковий хендоф-блок із трьох секцій:

1. **What I did** — що стадія зробила (які файли записала). Коміт пропонує лише `implement`; стадії планування не комітять (їхні артефакти поза git — див. *Commit policy*).
2. **Review before continuing** — посилання на файли, які стадія створила/змінила і які треба
   глянути на цьому геті (реальні `docs/features/<slug>/…` шляхи — клікабельні/копіювані).
3. **Run next** — спершу `/clear` (обов'язково для forward-переходу — наступна стадія перечитує
   все з диска), потім наступна команда `/sdd:<next> <slug>` у **fenced-блоці** (копіюється в один
   клік) + альтернатива-пропуск, якщо вона є.

Це прибирає головний біль: «погано виводить, незручно копіювати і перевіряти».

---

## Commit policy (this fork)

Everything lives **in the code repo**, in two layers (full model → [`./paths.md`](./paths.md)). The **process
trail** — `docs/features/<slug>/` (spec, sad, tasks, `_review`), `docs/architecture-map.md` —
is **gitignored** (it stays in the working tree but out of git history; the dev references a set via its Jira
ticket — ai-operating-policy §5). The **durable docs** — `docs/architecture/`, `docs/ecosystem/`,
`docs/integrations/<Op>/`, `docs/standards/`, **`docs/adr/`** (ADRs), and **`docs/roadmap.md`** — are **committed**. Either way, **the
AI never commits** (§1) — it writes files and the dev commits:

> **Where these paths resolve.** Every `docs/…` path in this file (and everywhere in the plugin) is named relative to
> `docs_root` from `.claude/sdd.local.md` — default `docs` (in-repo). The single resolution rule lives in
> [`./paths.md`](./paths.md); the review-file paths in the canonical-sequence table below are the `docs_root`-relative form.

- **No stage runs a git mutation.** `specify` · `clarify` · `glossary` · `design` · `sequences` ·
  `data-model` · `api` · `tasks` · `plan-tests` · `classify-size` · `decide-adr` · `roadmap` · `survey` ·
  `document` **write their files and stop** — they never `git add` / `git commit`. *What I did* lists the
  files written, **left unstaged** for the dev (ai-operating-policy §1).
- **One commit per feature, proposed by `implement`.** `implement` does the work, then **proposes** a
  **single** commit (not per task) in the team format `[TASK-{JIRA_KEY}] <imperative>` (or `[FIX-{JIRA_KEY}]`;
  `[TASK]` / `[FIX]` when there's no ticket) and waits for the dev to run it. **No `SDD-Task` / `SDD-AC`
  trailers and no AI mention** — AC traceability lives in `tasks.json`, which `review` reads.
- **Never auto-commit, never push, never `gh`.** `implement` *proposes* the commit (`auto_commit: off`); `ship`
  *proposes* the PR command — the AI runs neither (ai-operating-policy §1: no commits/git mutations; §2: no
  remote/GitHub interaction). PRs and merges are exclusively the developer's.

### Doc-stage write boundary (the durable `docs/`)

The **doc stages** — `document` (+ `ship`'s auto-hook) and `survey`'s durable-docs bootstrap, all via `doc-writer` — write the **durable `docs/`** docs, never the code. Their boundary (the one [`./paths.md`](./paths.md) Rule 4 points back to here for):

- **Write only inside the durable `docs/` folders** (`architecture/`, `ecosystem/`, `integrations/<Op>/`, `standards/`) — never code, never the gitignored process trail (`docs/features/`, `architecture-map.md`), never `docs/roadmap.md` (that's the `roadmap` skill's domain), never a sibling repo.
- **Never commit.** The AI writes the doc and **leaves it unstaged** (§1); the dev commits. (ADRs in `docs/adr/` are committed too, but authored by `design` / `decide-adr`, not `doc-writer`.)
- **Never delete a doc to "replace" it.** Update the stale section in place — see [`./doc-method.md`](./doc-method.md).
- **Cross-boundary contracts are canon here**, in this repo's `docs/ecosystem/` — no fan-out to another repo (paths.md Rule 3).
- **Propose → approve.** The doc write is always proposed and approved, never silent (the `auto_commit: off` parallel).

## The block (sectioned format)

```md
## ✅ <skill> — <slug>

**What I did**
- <1–3 bullets: the artifact(s) produced/changed; for `implement`, the single proposed code commit>

**Review before continuing**
- `docs/features/<slug>/<file>` — <what to check here>
- `docs/features/<slug>/<file2>` — <…>

**Run next**
1. `/clear` — mandatory (fresh context; the next stage re-reads its inputs from disk)
2. then run:
   ```
   /sdd:<next> <slug>
   ```
   ↳ or `/sdd:<alt> <slug>` to <skip condition>   ← only when a real skip exists
```

Rules for filling it:

- **Always emit it** as the final output, once per run (for `implement`, after the commit is proposed). Never end a
  skill on a bare «Next: X».
- **What I did** — concrete and self-contained: name the files written (and, for `implement`, the proposed
  commit message), so the user doesn't scroll up to reconstruct it.
- **State the size used.** *What I did* names the `feature_size` the stage worked at — «size M (from
  `.size`)»; if the stage had to **default** because `.size` was missing, say so loudly — «size M
  (default — no `.size`; run `/sdd:classify-size <slug>`)» — so a missing size surfaces at this gate,
  not three stages later. (`specify` establishes `.size` at the start, so this should be rare.)
- **Review before continuing** — list **every artifact this stage wrote or changed**, each as a real
  `docs/features/<slug>/…` path (or repo-root path like `docs/architecture-map.md`) plus a one-liner
  on what to eyeball. This *is* the per-gate review checklist.
- **Run next** — the next command in **`/sdd:<name> <slug>`** form inside a fenced code block (so the
  user copies it in one click). `/clear` is step 1 and **mandatory** for a forward backbone handoff.
  Add a `↳ or …` skip-alternative **only** when one genuinely exists (see the table).
- Keep the `<slug>` substituted with the real slug — never leave the literal `<slug>` in the printed
  block.

## Variants

- **Backbone forward handoff** (`survey → … → review → ship`): `/clear` mandatory + the next stage.
- **Loop-back** (`review → implement` on `CHANGES REQUESTED`): **no `/clear`** — you stay in context
  to iterate; *Run next* = `/sdd:implement <slug>` (fix), then re-review the changed surface.
- **Terminal** (`ship`): there is no `/sdd` successor. *Run next* becomes **Done** — the PR command/URL
  + «merging to main is your call»; still print *What I did* + *Review* (the changelog + PR).
- **Express lane** (`fix`): the small-change lane runs validate→implement→review→document in one pass,
  off the backbone. Terminal — no `/sdd` successor: *Run next* = **Done** with the proposed
  `[FIX-{JIRA_KEY}]` commit (the dev commits). The one exception is a review loop-back — a CRITICAL/
  IMPORTANT `reviewer` finding → fix the surface and re-review **with no `/clear`** (you're iterating),
  like the `review → implement` loop-back. If the size tripwire fires, there is no handoff block — the
  skill refuses and routes to `/sdd:specify <slug>`.
- **Utility** (`classify-size`, `glossary`, `decide-adr`, `roadmap`, `document`): called ad-hoc, not a gate.
  `/clear` is **optional** (recommend it only if the context is large); *Run next* = «resume your
  backbone stage», naming the likely one (e.g. `/sdd:design <slug>`). Print *What I did* + *Review*
  (the one file it wrote). `document` is also invoked **automatically by `ship`** (the living-doc hook),
  so as a gate it usually no-ops; standalone it syncs/creates the durable `docs/` docs ad-hoc (cross-boundary contracts recorded in this repo's `docs/ecosystem/`).

## Canonical sequence (stage → review-files → next)

| Stage | Review before continuing (files written) | Run next |
|---|---|---|
| `survey` | `docs/architecture-map.md` (+ scaffold `tasks.json` on greenfield) | `/sdd:specify <slug>` |
| `specify` | `docs/features/<slug>/spec.md` | `/sdd:clarify <slug>` |
| `clarify` | `docs/features/<slug>/spec.md` (tightened) | `/sdd:glossary <slug>` ↳ or `/sdd:design <slug>` |
| `design` | `sad.md` (C4 §3/§5 + `target_surfaces`) + `docs/adr/` (committed) | `/sdd:sequences <slug>` |
| `sequences` | `sad.md` §6 (flows) | `/sdd:data-model <slug>` |
| `data-model` | `data-model.md` + staged `migrations/` | `/sdd:api <slug>` |
| `api` | `contracts/openapi.yaml` (+ `events.md`, `api-sync-report.md`) | `/sdd:tasks <slug>` |
| `tasks` | `tasks/` + `tasks.json` | `/sdd:plan-tests <slug>` ↳ then `/sdd:implement <slug>` |
| `plan-tests` | `test-plan.md` (or `spec.md` `## Test plan` for XS/S) | `/sdd:implement <slug>` |
| `implement` | the committed diff (code + tests) + `tasks/tracker.md` | `/sdd:review <slug>` |
| `review` | `_review/review-<date>.md` | `/sdd:ship <slug>` (PASS) · `/sdd:implement <slug>` (CHANGES, no `/clear`) |
| `ship` | `CHANGELOG` + the proposed PR command (+ any durable `docs/` docs synced/created by the auto-hook, left unstaged) | **Done** — PR command (the dev runs it); merge is your call |
| `classify-size` | `.size` | resume — e.g. `/sdd:specify <slug>` |
| `glossary` | `CONTEXT.md` | resume — e.g. `/sdd:design <slug>` |
| `decide-adr` | `docs/adr/NNNN-<title>.md` (committed) | resume — `/sdd:tasks <slug>` or `/sdd:plan-tests <slug>` |
| `roadmap` | `docs/roadmap.md` | resume your backbone stage |
| `document` | the durable `docs/` docs synced/created (cross-boundary → `docs/ecosystem/`), or the no-op reason | resume — usually `/sdd:ship <slug>`, else done |
| `fix` (express lane) | the working diff (code + tests) + any durable `docs/` doc synced | **Done** — propose `[FIX-{JIRA_KEY}]` commit (the dev commits); loop back to `fix` on a review finding |

## Discipline

- **The block is the last thing printed — every run, no exceptions.** A skill that ends on prose
  without it has regressed.
- **Real paths, not descriptions.** «the SAD» is not reviewable; `docs/features/<slug>/sad.md` is.
- **The next command is copy-ready** — `/sdd:<name> <slug>` in a fenced block, slug substituted.
- **`/clear` only where it's correct** — mandatory on a forward backbone handoff, omitted on a
  loop-back (you're iterating), optional after a utility.
- **Format canonical here** — a skill that hand-rolls its own block shape has duplicated the contract.

## Where each skill calls this

Every skill's final protocol step ends with: «emit the **stage-handoff block** per
[`handoff.md`](./handoff.md)» + its own next command from the table above. The format + variants live
here; the skill supplies only the run-specific content.
