# Research docs

How the research skills should consume this repo's research documentation when exploring or proposing work.

## Before working on anything, read these

- **`CONTEXT.md`** at the repo root — paper terminology (glossary), code/data ↔ paper-term map (variables), and who is in / out of the analysis sample (population).
- **`docs/adr/*.md`** — methodology decisions touching the area you're about to work in (identification strategy, estimator, prior choices, sample restrictions, missing-data handling, etc.).

If any of these files don't exist, **proceed silently**. Don't flag their absence; don't suggest creating them upfront. The producer skill (`grill-with-docs-research`) creates them lazily as terms and decisions actually get resolved.

## File layout

Research repos are single-context — one paper / project per repo.

```
/
├── CONTEXT.md
├── docs/adr/
│   ├── 0000-template.md
│   ├── 0001-identification-strategy.md
│   └── 0002-priors.md
└── R/ (or analysis/, src/, etc.)
```

## Use the glossary's vocabulary

When your output names a paper concept (in an issue title, an analysis plan, a hypothesis, a variable name in a new pipeline step, a comment in code), use the term as defined in `CONTEXT.md`. Don't drift to synonyms the glossary explicitly avoids.

If the concept you need isn't in the glossary yet, that's a signal — either you're inventing language the paper doesn't use (reconsider) or there's a real gap (note it for `grill-with-docs-research`).

## Flag ADR conflicts

If your output contradicts an existing methodology ADR, surface it explicitly rather than silently overriding:

> _Contradicts ADR-0007 (two-way fixed-effects DiD) — but worth reopening because…_

If the ADR is still **Proposed**, resolve the conflict by revising it in place. If it is **Accepted**, don't edit it — propose a new ADR that supersedes (or reverses) the old one. Either way, surface the conflict; never silently override an Accepted decision.
