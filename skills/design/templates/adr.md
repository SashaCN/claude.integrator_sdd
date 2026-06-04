<!-- Format: MADR (Markdown Any Decision Record). -->
<!-- Spawned by `design` when a decision crosses the blast-radius gate (references/blast-radius.md),
     and also written by `decide-adr` for post-hoc decisions.
     Written to the COMMITTED docs/adr/ (globally numbered), in clean Markdown — no wikilinks. -->


---
status: Accepted                                # Proposed → Accepted → Superseded by NNNN. `design` writes Accepted directly.
owner: "<decision owner — usually Architect>"   # who is accountable for the decision
reviewers: []                                   # who reviewed before merge (usually Tech Lead, + Security if relevant)
updated_at: "<YYYY-MM-DD>"                       # last update (for Superseded — the date of replacement)
feature_size: "<from .size: XS/S/M/L/XL>"
ticket: "<tracker ticket that triggered the feature>"
---

# NNNN — <title in decision-form: the chosen solution as a noun phrase, e.g. "Sliding-window counter for rate limiting">

<!-- IMPORTANT: the title names the DECISION (the chosen solution), not the problem. -->
<!-- Same rule as the filename: NNNN-time-sortable-ids.md ✓ vs NNNN-id-strategy.md ✗ -->
<!-- ✓ "Typed-blocks table for content storage"    -->
<!-- ✗ "Content storage strategy"                  -->

- **Status:** Accepted
- **Date:** <YYYY-MM-DD>
- **Deciders:** <names — usually the Architect + the user during the Socratic walk>

## Context

<2–4 sentences: what is happening, why this decision must be made now. Pull from sad.md §3 (Context)
+ the section that triggered this ADR.>

## Decision drivers

<bullets — the quality goals / constraints that pushed the choice. Each bullet comes from spec §6 NFR,
or §2 SAD Constraints, or a §1 top-3 quality goal. Don't invent drivers — this filters out pet decisions.>

- <e.g. latency target from spec §6 NFR>
- <e.g. multi-tenant isolation requirement from spec §6.1>
- <e.g. an existing capability already in the stack — no new infra cost>

## Considered options

<List ALL options presented in the AskUserQuestion, including the rejected ones. One line each.
Do NOT add a strawman — an option an existing constraint already excludes (a critic F6 hit).>

1. **<Option A>** — <one sentence>.
2. **<Option B>** — <one sentence>.
3. **<Option C>** — <one sentence>.

## Decision outcome

**Chosen:** Option <letter or name>. <1–2 sentences — why this won over the alternatives, citing the
decision drivers above.>

## Consequences

**Positive**
- <e.g. the dominant quality goal is met naturally>
- <e.g. reuses an existing capability — no new infra>

**Negative**
- <e.g. more memory/space per item than the simpler option>
- <e.g. the logic is a little harder to reason about>

**Neutral**
- <e.g. switching to the alternative later is possible but needs a data backfill>

<!-- An honest consequence log fills Negative and Neutral too, not only Positive. -->

## Links

<!-- Without this section an ADR is an orphan. This ADR is COMMITTED in docs/adr/; the spec and SAD
     are gitignored process artifacts, so link them via the Jira ticket, not an in-repo path.
       1) the ticket (which feature / user story triggered it) — see frontmatter `ticket`
       2) the SAD section it attaches to (by §N, referenced through the ticket)
       3) sideways to a sibling ADR (relative link, same folder) if together they form one contract -->

- Ticket: <JIRA-KEY> (the spec/SAD live under the gitignored `docs/features/<slug>/` — reach them via the ticket)
- SAD section: §<N> (in the feature's `sad.md`)
- Related ADR: [NNNN-other](NNNN-other.md) <if any>
