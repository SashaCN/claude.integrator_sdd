# Paths — where process artifacts and durable docs live (one self-sufficient repo)

> **Reference-only.** Not a skill. Defines, in **one place**, how every `docs/…` path the skills name
> resolves, and which paths are **committed** (durable docs — the source of truth) vs **gitignored**
> (the planning trail). This fork writes everything **inside the code repository** — there is no
> separate documentation vault. *How* a durable doc is written is [`./doc-method.md`](./doc-method.md);
> *where* it lives is here.

## TL;DR (українською)

Усе живе **в одному репозиторії коду**, під `docs_root` (default `docs`). Два шари:
**durable docs** (джерело правди — `docs/architecture/`, `docs/ecosystem/`, `docs/integrations/<Op>/`,
`docs/standards/`, `docs/adr/`, **`docs/roadmap.md`**) — **комітяться**; **process-trail** (план/таски —
`docs/features/<slug>/`, `docs/architecture-map.md`) — **gitignored** (план не в історії коду, §5
ai-operating-policy). ADR — **комітиться** (`docs/adr/`, довговічне рішення). **`docs/roadmap.md` —
committed durable** (Prime.Integrator комітить його; AGENTS.md трактує як джерело правди «що зроблено / що далі»). Особистий Obsidian-vault
розробника — приватний шар поза репо (`Obsidian/` у `.gitignore`); плагін його **не чіпає**. Нормативи/
метод/конвенції — у **плагіні**. Між сервісами fan-out **немає** — крос-сервісні контракти живуть у
`docs/ecosystem/` **цього ж** репо (принцип «repo self-sufficient»). Як пишеться нота —
[`./doc-method.md`](./doc-method.md).

## The settings key

Lives in `.claude/sdd.local.md` → [`../implement/references/settings.md`](../implement/references/settings.md):

| Key | Default | What it roots |
|-----|---------|---------------|
| `docs_root` | `docs` | The base dir for **every** `docs/…` path the skills name — both the committed durable docs and the gitignored process trail. Default `docs` = in-repo (the only supported layout in this fork). |

There is **no** `vault_root` and **no** `doc_routes` — they belonged to the old separate-vault model
and are gone. One repo, one `docs_root`.

## The target model — two layers under `docs_root`

| Layer | Path (under `docs_root`) | git | Who writes |
|---|---|---|---|
| **Durable docs** (source of truth) | `docs/architecture/`, `docs/ecosystem/`, `docs/integrations/<Op>/`, `docs/standards/`, `docs/adr/`, **`docs/roadmap.md`** | **committed** (by the dev) | `document` / `doc-writer`, `survey` bootstrap; ADR — `design` / `decide-adr`; roadmap — `roadmap` / `specify` / `ship` |
| **Process trail** (plan / tasks) | `docs/features/<slug>/` (spec, sad, tasks, `_review`), `docs/architecture-map.md` | **gitignored** | the backbone skills |
| **Personal Obsidian vault** | outside the repo (`Obsidian/` ignored) | — | **not touched by the plugin** |

This realizes the repo's own `docs/standards/output-discipline.md` (the **repository is the source of
truth**; the dev's personal Obsidian vault is a private layer that is never committed and never a
load-bearing dependency) and `docs/standards/ai-operating-policy.md` §5 (developer-specific plans /
tasks / session notes stay **out of the repository**).

## Rule 1 — every `docs/…` resolves under `docs_root`

Wherever a skill names `docs/<X>`, read/write it at **`{docs_root}/<X>`** — `docs/features/<slug>/…`,
`docs/roadmap.md`, `docs/architecture-map.md`, `docs/architecture/…`, `docs/ecosystem/…`,
`docs/adr/…`. With the default `docs_root: docs` the path is literal.

## Rule 2 — durable docs are committed; the process trail is gitignored

- **Durable docs** (`docs/architecture/`, `docs/ecosystem/`, `docs/integrations/<Op>/`,
  `docs/standards/`, `docs/adr/`, **`docs/roadmap.md`**) are the living source of truth the pipeline
  reads and keeps current. They are **committed by the developer** (the AI proposes the edit and leaves
  the file **unstaged** — ai-operating-policy §1). They follow [`./doc-method.md`](./doc-method.md).
- **Process trail** (`docs/features/<slug>/`, `docs/architecture-map.md`) is the per-feature planning
  artifact set. It lives **inside the repo working tree** for convenience but is **gitignored** so it
  never enters the code history (ai-operating-policy §5). The dev references a spec/tasks set via its
  Jira ticket, not via an in-repo link.
- **ADR is the exception inside the trail's neighbourhood:** an architecture decision record is a
  **durable** artifact and is **committed** in `docs/adr/` — it is the long-lived record of *why*, and
  `output-discipline` names ADRs as repository source of truth.
- **ADR ↔ `design-decisions.md` (this fork's reconciliation).** The repo's own `output-discipline.md`
  mandates ADRs in `docs/adr/ADR-XXXX-*.md` **and** the repo keeps a human-facing decisions log at
  `docs/ecosystem/design-decisions.md` (cited by `AGENTS.md`). They are **not** two competing systems:
  the **full MADR record** is the file in `docs/adr/` (unchanged — `design` / `decide-adr` own it); the
  log in `docs/ecosystem/design-decisions.md` is the **one-line index** — title + `Status: Decided` +
  a link to the ADR. When a skill writes an ADR it **also appends/updates the matching one-line row** in
  `design-decisions.md`. ⚠️ Prime.Integrator currently has **no `docs/adr/`** — the dev must create it
  (its own standard requires it); until then the skills create it on first ADR.
- **`docs/roadmap.md` is durable, not trail.** Unlike the upstream model, this fork **commits** the
  roadmap: Prime.Integrator tracks it and `AGENTS.md` cites it as the source of truth for "what's done /
  what's next". `roadmap` / `specify` / `ship` edit it and leave it **unstaged** for the dev to commit.

## Rule 3 — cross-service contracts live in this repo's `docs/ecosystem/`

There is **no fan-out** to sibling repositories. A contract or event that crosses a service boundary is
documented **here**, in `docs/ecosystem/` (e.g. `ecosystem/events-catalog.md`,
`ecosystem/inter-service-communication.md`) — the repo stays **self-sufficient**: a new developer
clones it and finds every doc, with no access to any other repo or vault (`output-discipline` Hard
rule 1). The AI **never** edits another team's repository (ai-operating-policy §1–§2).

## Rule 4 — the process trail is provenance, not a read-source

`specify` / `survey` treat the **durable docs** as source-of-truth constraints, **not** the process
trail. A durable doc may mention the ticket that drove it, but the dependency runs doc→ticket, never
the reverse. Process artifacts are kept in the working tree (gitignored) and pruned by the dev — the
pipeline never auto-deletes them.

## Rule 5 — normatives live in the plugin, not in `docs/`

Rules / method / templates / conventions are **agent-facing** and live in the plugin:
[`./doc-method.md`](./doc-method.md) (the writing method) and the shared coding standards. They are
**not** durable docs in `docs/` — `docs/` holds project **knowledge** (architecture / ecosystem /
integrations / standards / ADRs). (The repo's own `docs/standards/` are the *project's* cross-cutting
standards, duplicated per repo by the team — distinct from this plugin's internal method files.)

## `.gitignore` interaction

The settings auto-create patches the repo `.gitignore` (when `docs_root` is repo-relative — the
default) to ignore the **process trail** only: `docs/features/`, `docs/architecture-map.md` — **plus**
`.claude/*.local.md` + `.worktrees/`. It must **not** ignore the durable docs: `docs/architecture/`,
`docs/ecosystem/`, `docs/integrations/`, `docs/standards/`, **`docs/adr/`**, and **`docs/roadmap.md`**
stay tracked. Detail → [`../implement/references/settings.md`](../implement/references/settings.md).

## Discipline

- **Skills keep naming `docs/…`** — the indirection lives here, not smeared across every skill.
- **Durable docs committed, process trail gitignored, ADR committed.** The dev commits; the AI proposes
  and leaves edits unstaged (never `git add` / `git commit` — ai-operating-policy §1).
- **One self-sufficient repo.** Cross-service contracts are documented in this repo's
  `docs/ecosystem/`; the plugin never writes to or fans out to another repository.
- **The process trail never feeds the read contract** — durable docs are the source of truth.
- **Nothing normative in `docs/`** — the writing method / plugin conventions are the plugin's (Rule 5).
- **The personal Obsidian vault is private and untouched** — it sits outside the repo (`Obsidian/`
  ignored); the plugin neither reads nor writes it.
