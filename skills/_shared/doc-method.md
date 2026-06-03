# Doc method — how durable docs in `docs/` are written (the repo's Markdown convention)

> **Reference-only.** Not a skill. This file is the **single source** for *how* a durable doc under
> `docs/` is written — its format, its name, how it links, where it slots in the taxonomy. `doc-writer`
> enforces it; `document` / `survey` / `specify` read it. The method lives here, in the plugin, so one
> edit reaches every doc the pipeline writes. Where a doc *lives* (which folder, committed vs gitignored)
> is [`./paths.md`](./paths.md)'s job; this file is purely the **writing method**.

## TL;DR (короткий вступ українською)

Довговічні доки — **чистий Markdown** у стилі вже наявних доків репо: відносні посилання
`[текст](../шлях.md)`, ASCII-діаграми + таблиці, **kebab-case** імена файлів, **EN-only**. Слотяться в
**наявну таксономію** під `docs/`: `architecture/`, `ecosystem/`, `integrations/<Op>/`, `standards/`,
`adr/`. README кожної папки — **індекс-навігатор** (список доків + один рядок опису). **Жодного**
Obsidian: ні `[[wikilinks]]`, ні YAML-frontmatter-схеми, ні `type`-vocab, ні `vault:`-рефів. Метод
матчить **найближчий сусідній doc** у цільовій папці; де doc живе — [`./paths.md`](./paths.md).

## The format — clean Markdown, matching the repo

Durable docs are plain GitHub-flavoured Markdown, written to **match the docs already in `docs/`**.
The conventions, observed from the existing tree:

- **Headings** structure the doc (`#` title, `##` sections, `###` sub-sections).
- **Relative Markdown links** between docs: `[API Endpoints](api-endpoints.md)`,
  `[inter-service communication](../ecosystem/inter-service-communication.md)`. **Never** Obsidian
  `[[wikilinks]]`.
- **ASCII / code-fence diagrams** for architecture and flows (box-and-arrow, request lifecycle) — see
  `docs/ecosystem/inter-service-communication.md` and `docs/integrations/hypha/README.md` for the
  house style. **Mermaid** is acceptable where the existing sibling uses it (it renders on GitHub) —
  match the neighbour.
- **Tables** for layer/responsibility maps, env references, connection matrices.
- **No YAML frontmatter** on a durable doc — the existing docs open straight on a `#` title. (The
  gitignored process artifacts — `spec.md`, `sad.md`, ADRs — keep their own frontmatter; that is the
  process trail's format, not this. ADRs follow MADR — see `design`/`decide-adr`.)

There is **no controlled `type` vocabulary, no `status`/`project`/`tags` block, no `source:` router
field, no tombstone/`superseded_by`** — those were the old vault model. A durable doc is judged by
whether it is accurate and findable, not by a frontmatter schema.

## Naming

- **kebab-case** `.md` files, matching the existing tree: `api-client-conventions.md`,
  `data-normalization.md`, `inter-service-communication.md`, `events-catalog.md`.
- **`README.md`** is the folder index (see below) — uppercase, as the existing `integrations/<Op>/README.md`.
- **ADRs**: `NNNN-<decision-in-kebab>.md` under `docs/adr/` (4-digit, globally numbered) — e.g.
  `0003-time-sortable-ids.md`. Owned by `design` / `decide-adr`.

## The taxonomy — slot into the existing folders

A new or updated durable doc goes into the folder that already owns its subject. The established
taxonomy under `docs/`:

| Folder | What lives there |
|---|---|
| `docs/architecture/` | This service's internal architecture — layers, conventions, data flow, DB, queues, env, the per-area deep-dives. |
| `docs/ecosystem/` | Cross-service view — how this service talks to the others (events catalog, inter-service communication, the ecosystem overview). **Cross-boundary contracts are documented here, in this repo** — there is no fan-out to sibling repos. |
| `docs/integrations/<Operator>/` | Per-operator integration docs (`README.md` as the index + `api-endpoints.md`, `architecture.md`, `configuration.md`, `normalization.md`, `troubleshooting.md`, …). New operator → a new `<Operator>/` folder modelled on an existing one (e.g. `hypha/`). |
| `docs/standards/` | The team's cross-cutting standards (coding, commit, review, security, testing, output-discipline, ai-operating-policy). These are duplicated per repo by the team — **read them, don't rewrite them** unless the change is genuinely a standards change. |
| `docs/adr/` | Architecture Decision Records (MADR). **Committed.** |

If a change doesn't fit an existing folder, prefer extending the nearest one over inventing a new
top-level folder; a genuinely new area is a deliberate taxonomy decision, not an ad-hoc `mkdir`.

## README as the folder index (the navigator)

A folder's `README.md` is its **map**: a short overview + a linked list of the docs in the folder, each
with a one-line description — exactly like `docs/integrations/hypha/README.md`'s `## Related
Documentation` section. When you add a doc to a folder that has a README index, **add its link there**;
when you create a new `<Operator>/` folder, give it a `README.md` index modelled on the sibling
operator's. The README orients; the knowledge lives in the individual docs.

## Cross-referencing

- **Link with relative Markdown paths**, computed from the linking file's location:
  `[normalization](normalization.md)` (same folder), `[events catalog](../ecosystem/events-catalog.md)`
  (sibling folder). Verify the target file exists before linking.
- **A doc that crosses a service boundary** documents the contract in `docs/ecosystem/` (the events
  catalog / inter-service-communication doc) and links to it — the contract is canon **in this repo**,
  not fanned out elsewhere ([`./paths.md`](./paths.md) Rule 3).
- **Code anchors**: when a doc names a class/file, use the repo-relative path the existing docs use
  (`app/Integrations/Mediators/HyphaMediator.php`) so a reader can jump to it. Code is the authority on
  behaviour; the doc explains the *why* and the *shape*.
- **The reverse `@see` backlink (code → doc)**: a doc that documents a single class pairs with an
  `@see docs/<this-doc>.md` line in that class's PHPDoc, for a two-way **code ⇄ doc** link. That line
  is owned by the **coding standard**
  ([`./coding-standards.md`](./coding-standards.md) → PHPDoc), written by the code author — **not** by
  `doc-writer`, which never edits code. `doc-writer` only **reports** the exact `@see` line it expects.
- **Section links** (`[text](file.md#section-anchor)`) when a specific section is meant.

## Match the nearest neighbour (replaces the old per-type templates)

There is **no template file per doc kind**. To write or update a doc, **read the nearest sibling doc in
the target folder and match it** — its heading structure, depth, diagram style, table shape, and tone.
The existing tree *is* the template:

- A new integration doc → model it on `docs/integrations/hypha/` (`README.md` index + the per-aspect
  docs). For per-operator normalization specifically, the target repo keeps a real scaffold at
  `docs/architecture/templates/operator-normalization-template.md` — copy it to the operator's
  normalization doc and fill it.
- A new architecture deep-dive → match an existing `docs/architecture/*.md`.
- A cross-service doc → match `docs/ecosystem/*.md`.

When a sibling and a general guideline disagree on a local convention, **the sibling wins** — match
what the folder actually does.

## Language — EN-only

Durable docs are **English only** — prose *and* identifiers (the repo's `ai-operating-policy.md` §3).
This plugin reference file keeps a Ukrainian TL;DR by house convention — that exemption is for
**plugin** reference files, not for the repo's `docs/`.

## Discipline

- **The method lives here, once.** A rule about how a doc is written belongs in this file — not copied
  into a doc, not re-stated per skill. A skill keeps a one-line pointer + its own delta.
- **Match the neighbour.** The existing `docs/` tree is the authority on format, naming, and depth;
  read the nearest sibling before writing.
- **Clean Markdown, relative links, kebab-case, EN-only.** No Obsidian wikilinks, no frontmatter
  schema on durable docs, no `vault:` refs.
- **Slot into the taxonomy.** A doc routes to the folder that owns its subject; update the folder
  README index when you add one.
- **Cross-boundary contracts are canon here**, in `docs/ecosystem/` — no fan-out to sibling repos.
- **Repository code wins.** Every claim is grounded in code or a primary source; the unverified goes
  under an explicit "Unverified / open questions", never asserted as fact.

## Where each skill reads this

- **`doc-writer`** — applies this on every write: clean Markdown, relative links, taxonomy slot,
  neighbour-matching, README index update, EN-only.
- **`document`** — on a gap, creates a doc *to this method* (gap → create-on-ship).
- **`survey`** — bootstrap creates missing docs / README indexes to this method; the reconciliation
  pass checks the durable docs against the code (code wins).
- **`specify`** — reads the durable docs as source-of-truth constraints (it does not write them).
