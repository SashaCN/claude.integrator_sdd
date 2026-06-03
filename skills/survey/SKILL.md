---
name: survey
model: inherit
effort: medium
agents: [explorer]
description: >
  Use to establish the repo's architecture map the rest of the pipeline reads. Two modes: on an
  EXISTING codebase it scans once and persists what's there; on an EMPTY/greenfield repo it runs a
  short, level-adaptive foundation session — picks the stack / folder structure / data approach /
  conventions WITH you (defaults-heavy), fixes them as the foundation + foundational ADRs, and emits
  a scaffold tasks.json that implement materializes into a real skeleton. Triggers on "survey the
  codebase", "map the architecture", "set up a new project", "bootstrap the foundation",
  "/sdd:survey", "вивчи кодову базу", "карта архітектури", "новий проєкт", "заклади фундамент".
  Output: docs/architecture-map.md (+ docs/adr/ + scaffold tasks.json on greenfield). Records
  reflects_commit for staleness; reads, never overwrites, an authored architecture doc. Also
  reconciles the repo's durable docs (docs/architecture/, docs/ecosystem/, docs/integrations/)
  against the same scan — flags where a doc has drifted from the code (code wins) and bootstraps a
  missing doc / README index in the existing taxonomy (propose→approve, left unstaged, never committed).
---

# Skill: survey

The pipeline's anchor on architecture. It produces `docs/architecture-map.md` — the single source of "what the system is" that `specify` (constraints), `design` (matches against it), `data-model`, and `implement` all read instead of re-discovering the code. It runs in one of **two modes**, auto-detected:

- **Brownfield** (the repo has source) → scan it once and persist the **current** architecture.
- **Greenfield** (empty / near-empty repo) → run a short, **level-adaptive foundation session**: pick the stack / structure / data approach / conventions *with* the user (defaults-heavy), fix them as the **foundation** + foundational ADRs, and emit a **scaffold `tasks.json`** that `implement` turns into a real skeleton. Greenfield detail → [`./references/foundation.md`](./references/foundation.md).

Repo-level utility (one map serves every feature). The scan is delegated to [`explorer`](../../agents/explorer.md); question phrasing → [`../_shared/ask-style.md`](../_shared/ask-style.md); depth → [`../_shared/size-matrix.md`](../_shared/size-matrix.md).

> **Paths.** Every `docs/…` path below resolves under `docs_root` (`.claude/sdd.local.md`; default `docs`). The durable reference docs the map reconciles with are the committed `docs/architecture/` · `docs/ecosystem/` · `docs/integrations/<Op>/`; `docs/architecture-map.md` itself is gitignored → [`../_shared/paths.md`](../_shared/paths.md).

## Owner

Architect / Tech Lead — they own the architecture (brownfield: confirm it reflects reality; greenfield: decide the foundation).

## Inputs

- (Optional) a path/scope hint (default: repo root).
- (Read, never overwrite) an authored architecture doc if present (**`AGENTS.md`** — the primary curated entry point in this fork, then `CLAUDE.md`, `docs/architecture.md`, `ARCHITECTURE.md`, `docs/adr/`) — the **authored source of truth** the map reconciles with, never clobbers. `architecture-map.md` is the *derived cache* below it (gitignored, regenerable, carries the volatile `file:line` precedents `AGENTS.md` deliberately omits); on any conflict the curated `AGENTS.md` wins and the map is re-derived.
- The repo's **durable docs** — `docs/architecture/*`, `docs/ecosystem/*`, the relevant `docs/integrations/<Op>/*`, `docs/standards/*` — are part of that authoritative set: the map **reconciles with them, never overwrites** (the same rule as an in-repo authored doc). → [`../_shared/paths.md`](../_shared/paths.md).

## Protocol

1. **Detect mode + freshness.** If `docs/architecture-map.md` exists and is fresh (its `reflects_commit` ≈ current HEAD) → «map is fresh (reflects `<commit>`). Reuse or refresh?»; STOP on reuse. Else decide the mode: **brownfield** if the repo has source (modules/packages beyond config), else **greenfield** (empty or only scaffolding like a bare `go.mod` / `package.json`).

### Brownfield path (existing code)

2. **Read authored docs first.** The repo's **`AGENTS.md`** (the curated source of truth in this fork — orientation, domain, folder map, hard constraints) is the **primary authoritative input**, then any other hand-maintained architecture doc / root `CLAUDE.md` / `docs/adr/` → reconcile with them, never overwrite. The generated `docs/architecture-map.md` is the *cache* of this truth, not a peer: where the scan disagrees with `AGENTS.md`, flag it (V2 drift, code wins) but never clobber the curated file. **The repo's durable docs (`docs/architecture/*`, `docs/ecosystem/*`, the relevant `docs/integrations/<Op>/*`, `docs/standards/*`) are part of this authoritative set** — read and reconcile, never clobber (→ [`../_shared/paths.md`](../_shared/paths.md)).
3. **Scan via explorer.** Dispatch the [`explorer`](../../agents/explorer.md) agent — `subagent_type: "sdd:explorer"` (`haiku`/`low`, clean-isolated per [`../_shared/agent-roster.md`](../_shared/agent-roster.md)): «Report (a) language + frameworks + versions, (b) top-level module layout + per-module layers, (c) layering / wiring conventions, (d) datastores + access, (e) inter-module comms, (f) cross-cutting conventions (errors, IDs, tests, migrations) with one cited example each, (g) 2–3 representative features as precedents, (h) **if a frontend exists** — the component library / design system, design tokens (colors/spacing/typography), styling approach (Tailwind / CSS-modules / styled-components / …), shared UI primitives, and a representative screen/component as the UI precedent to reuse.» Large repo → fan out per subtree. (Fallback `subagent_type: "Explore"`.)
4. **Synthesize + stamp + validate + write.** Fill [`./templates/architecture-map.md`](./templates/architecture-map.md) (C4 of what exists, module inventory, cited conventions, datastores, **the Frontend / UI foundation if a frontend exists**, precedent guide, constraints) with real `file:line` anchors. Record `updated_at` + `reflects_commit: <short HEAD>`. **Validate the C4 Mermaid per [`../_shared/mermaid-check.md`](../_shared/mermaid-check.md)** (render-parse with `mmdc` if available, else the structural lint; fix before committing). Write `docs/architecture-map.md` (record `reflects_commit`). Then **emit the stage-handoff block** per [`../_shared/handoff.md`](../_shared/handoff.md) — *What I did* + *Review* (`docs/architecture-map.md`, + scaffold `tasks.json` on greenfield) + *Run next* (`/clear`, then `/sdd:specify <slug>`).

### Greenfield path (empty repo) → [`./references/foundation.md`](./references/foundation.md)

G2. **Calibrate to the person.** One opening `AskUserQuestion` to gauge how the user wants to engage — «pick good defaults, I'll confirm» / «walk me through each choice with explanations» / «let me choose each piece, keep it terse». This sets the dialogue's depth + phrasing (junior → defaults + glossed explanations per [`../_shared/ask-style.md`](../_shared/ask-style.md); senior → terser, more control). Not a product brief.
G3. **Intent (short).** 1–3 questions: what the project is + the kind of capabilities it'll have (e.g. «HTTP API» / «CLI» / «web app»). Enough to choose an architecture — deliberately NOT the feature briefing (that's `specify`, per feature).
G4. **Pick the foundation, defaults-heavy.** At the calibrated depth, choose: stack (language/framework/datastore), architectural style (e.g. hexagonal modules), folder/module structure, data/persistence approach (migration tool, ID strategy), core conventions (errors, tests, CI). Recommend a coherent default set; the user confirms or adjusts. Choice menus + defaults → [`./references/foundation.md`](./references/foundation.md).
G5. **Fix the foundation.** Write `docs/architecture-map.md` as the **established foundation** (mark `mode: greenfield-bootstrap`; the C4 is the *target* baseline) + spawn **foundational ADRs** in `docs/adr/` for the irreversible picks (stack, module style, persistence). Record `reflects_commit`. **Validate the C4 Mermaid per [`../_shared/mermaid-check.md`](../_shared/mermaid-check.md)** before committing.
G6. **Emit the scaffold + hand off.** Write a scaffold `tasks.json` (the skeleton: folder/module structure, a baseline module, the test harness, migration tooling, CI, a `CLAUDE.md`/rules doc) per the contract in [`./references/foundation.md`](./references/foundation.md). Each task's DoD anchors on the **skeleton smoke test** — «the project builds + boots + the empty test suite runs + the migration tool runs». Propose: «foundation fixed — run `implement` to materialize the skeleton» (the wave-of-the-hand hand-off). No commit here — `docs/architecture-map.md` is gitignored process trail and the foundational ADRs in `docs/adr/` are committed durable docs, both **left unstaged** for the dev (§1); `implement` then **proposes** the materialized skeleton code as one `[TASK-{JIRA_KEY}]` commit.

### Durable-docs reconciliation & bootstrap (the in-repo `docs/`)

The map above is the architecture sensor; this step keeps the repo's **durable docs** honest against the same scan. All writes here are **`docs/`-only, propose→approve, left unstaged** (the dev commits — §1). Method + taxonomy → [`../_shared/doc-method.md`](../_shared/doc-method.md); the two-layer model → [`../_shared/paths.md`](../_shared/paths.md).

V1. **Inventory the durable docs vs the code.** Walk `docs/architecture/`, `docs/ecosystem/`, `docs/integrations/<Op>/` and map each doc to the code area it describes (mediator/integration → `docs/integrations/<Op>/`; internal architecture → `docs/architecture/`; cross-service contract → `docs/ecosystem/`). Stamp the survey's `reflects_commit`. This inventory is what `document` consults (docs-first) before falling back to `explorer`.
V2. **Flag drift (code wins).** For each durable doc, compare its claims against the code it describes. Where they diverge, the **code is authoritative** — flag the doc for `document` / `doc-writer` to reconcile (the map = where, the doc = why/shape, the code = behaviour). Do **not** rewrite prose here; survey is the sensor.
V3. **Map the cross-service surface.** From the code (the authority on what crosses a boundary), list the events/contracts this service produces and consumes, and check they're reflected in this repo's `docs/ecosystem/` (the events catalog / inter-service-communication doc). There is **no fan-out** to sibling repos — cross-boundary contracts are canon here. Flag any missing or stale entry for `document`.
V4. **Check the README indexes + relative links.** For each folder with a `README.md` index, flag docs present on disk but missing from the index (and vice-versa), and report any relative `[text](path.md)` link whose target file is absent. Never auto-fix — report for `doc-writer`.
V5. **Bootstrap a missing doc area.** If a significant code area has **no** durable doc (e.g. a new operator under `app/Integrations/` with nothing in `docs/integrations/`), emit a stub in the right taxonomy folder to [`../_shared/doc-method.md`](../_shared/doc-method.md) — modelled on the nearest sibling (e.g. `docs/integrations/hypha/`), with a `README.md` index and heading skeletons (no invented prose). Propose the stub set before writing; leave it unstaged.

## Definition of Done

- `docs/architecture-map.md` exists with `updated_at` + `reflects_commit`; an authored doc (if any) was reconciled, never overwritten.
- **Brownfield:** C4 of what exists + module inventory + cited conventions + precedent guide, real anchors (no placeholders).
- **Greenfield:** foundation fixed (stack/structure/data/conventions) at the user's calibrated level + foundational ADRs + a scaffold `tasks.json` whose tasks carry the skeleton smoke-test DoD, ready for `implement`.
- **Durable-docs reconciliation:** the `docs/` inventory mapped to code (stamped `reflects_commit`); drift flagged (code wins); the cross-service surface checked against `docs/ecosystem/`; README indexes + relative links checked; a missing doc area bootstrapped with a sibling-modelled stub + README index (proposed before write, left unstaged, never committed).

## Anti-patterns

- **Re-scanning the repo in every downstream skill** — the point is to scan once; others read the map (drift detection is the only re-read, of real domain files).
- **Overwriting a hand-maintained `docs/architecture.md`** — survey writes its own map and reconciles.
- **A map with no `reflects_commit`** — it silently rots; nobody knows it's stale.
- **Greenfield: a full product brief.** The foundation session picks the *architecture*, not the features — the idea/briefing is `specify`'s job, per feature. Keep it to intent + foundation choices.
- **Greenfield: ignoring the person's level.** A junior gets defaults + plain-language explanations; a senior gets control + terseness. One calibration question sets this — don't fire a senior-level wall of choices at a first-timer.
- **Placeholders / guessed layout** — cited or `UNKNOWN`; a fictional map is worse than none.
- **Rewriting doc prose during survey** — survey is the sensor (flags drift); `document` / `doc-writer` reconcile.
- **Auto-fixing a broken relative link or a missing README entry** — report it, never silently mutate the doc here.
- **Fanning a contract out to another repo** — the cross-service surface is checked against this repo's `docs/ecosystem/`; survey never touches a sibling repo.
- **Bootstrapping written-out docs** — bootstrap emits a sibling-modelled skeleton + README index, not finished prose; the dev commits.

## References & template

- [`./references/foundation.md`](./references/foundation.md) — greenfield: the calibration question, level-adaptive depth, the stack/structure/convention choice menus + defaults, foundational-ADR list, and the scaffold `tasks.json` contract.
- [`./templates/architecture-map.md`](./templates/architecture-map.md) — output scaffold (same file for current OR foundation; a `mode:` marker distinguishes).
- [`../_shared/agent-roster.md`](../_shared/agent-roster.md) — the explorer contract.
- [`../_shared/doc-method.md`](../_shared/doc-method.md) — the doc method (clean Markdown, taxonomy, README index) for the reconciliation + bootstrap stubs.
