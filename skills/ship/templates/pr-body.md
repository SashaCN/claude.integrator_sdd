<!-- PR body scaffold. Forge-agnostic — same body for `gh pr create --body-file` or `glab mr create --description`. -->

## Summary

<1–3 sentences: what this PR ships and why.>

**Jira:** <TASK-KEY>

## Acceptance criteria

<the AC this PR satisfies, each one line — the reviewer checks these against the diff>

- AC-01 — <business outcome> ✓
- AC-0N — <business outcome> ✓

## Design & decisions

The SDD process artifacts (spec, SAD, data-model, API contract, tasks) live under the gitignored
`docs/features/<slug>/` — see Jira `<TASK-KEY>` for linked context. **ADRs are committed** in `docs/adr/`
— link the relevant ones directly. If the reviewer needs the rest, summarize the key decisions here in
1–3 bullets (migration `<NNNN>`, any breaking change, the main design choice).

## Tasks

<key tasks from `tasks.json`, one line each — the whole feature ships as a single commit>

## Verification

- Unit: <result>
- Integration: <result, or "CI — Docker-backed">
- Lint + vet: <result>
- Ran the feature: <what was exercised against the AC, or what was deferred and why>

## Operational notes

- Migration: <run-on-deploy + rollback>, or none.
- Feature flag / config: <any>, or none.
