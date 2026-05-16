---
name: grill-with-docs-research
description: Socratic grilling session that stress-tests a research plan against the paper's recorded language (`CONTEXT.md` — glossary, variables, population) and methodology ADRs (`docs/adr/*.md`), updating both inline as decisions crystallise. Use when the user wants to grill a research plan, hypothesis, estimator choice, sample restriction, identification strategy, prior, or other methodological commitment.
---

<what-to-do>

Interview me relentlessly about every aspect of this plan until we reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

Ask the questions one at a time, waiting for feedback on each question before continuing.

If a question can be answered by exploring the codebase, explore the codebase instead.

</what-to-do>

<supporting-info>

## Research context awareness

This is a research repo: one paper, one project, one git repo. The documentation layout is single-context:

```
/
├── CONTEXT.md         # static facts: glossary, variables, population
├── docs/
│   └── adr/           # methodology decisions, numbered; append-only once Accepted
│       ├── 0000-template.md
│       ├── 0001-identification-strategy.md
│       └── 0002-priors.md
└── R/ (or analysis/, src/, etc.)
```

No `CONTEXT-MAP.md`. No per-context subdirectories. One paper per repo.

During exploration, also read what's already there:

- `CONTEXT.md` — the glossary (paper terms), variables (code/data column ↔ paper-term map), and population (who's in/out of the sample).
- `docs/adr/*.md` — methodology decisions touching the area being discussed.
- `_targets.R`, files in `R/` — the pipeline as it exists. Read code, names, comments.
- Raw data — reach for the code and `CONTEXT.md` first; column names and cleaning rules are usually checkable from the code that names and transforms them. But if actually examining the data would make this a *much more useful* session — resolving an ambiguity the code can't settle, surfacing an anomaly worth grilling, or grounding an edge-case scenario in real values — then open it. Light inspection (distributions, missingness, value ranges, cross-tabs), not the planned estimation.

### Lazy file creation

Create files only when there's something to write:

- If `CONTEXT.md` doesn't exist, create it the first time a glossary, variable, or population entry is resolved. Use the three-section structure (Glossary / Variables / Population) seeded by `setup-research-skills` in [`context-template.md`](../setup-research-skills/context-template.md).
- If `docs/adr/` doesn't exist, create it (and copy `0000-template.md` from [`adr-template.md`](../setup-research-skills/adr-template.md)) the first time an ADR is accepted.
- If `CLAUDE.md` lacks an `## Agent skills` section, mention it once at the start of the session — "`setup-research-skills` would have wired this up; continuing without it" — and proceed. Don't gate.

## During the session

### Challenge against CONTEXT.md

CONTEXT.md fails in three different ways. Watch all three.

#### Glossary drift

When the user uses a paper term that conflicts with the existing glossary entry, call it out immediately.

> Your glossary defines `treatment` as exposure ≥ 1 day to program X. You just said "treated for at least a week" — which is it?

#### Variable ↔ concept mismatch

When the user names a code or data column whose mapping in the Variables section doesn't match the paper concept they're invoking, call it out.

> You said `earn1` is the outcome. Variables maps `earn1` to log-earnings one year post — but the comparison in your plan is two years post. Mismatch — which is right?

#### Population scope drift

When the user makes a sample claim that contradicts (or quietly extends) the Population section, call it out.

> Population says workers 25–55 in sample S. You just added a "must have full prior earnings history" restriction. Either log it as a new exclusion or revise Population — don't leave it implicit.

### Sharpen fuzzy language

When the user uses vague or overloaded terms, propose a precise canonical term.

> You're saying "the effect" — do you mean ATT, ATE, or LATE? Those are different estimands and the choice changes both the spec and the interpretation.

### Discuss concrete scenarios

When methodology relationships are being discussed, stress-test them with single units or observations. Invent scenarios that probe edge cases and force precision.

> Take a worker who enrols in the program, exits after one day, never receives the treatment dose — does your `treatment` indicator fire? Does this person enter the analysis sample?

### Cross-reference with code

When the user states how something works in the pipeline, check whether the code agrees. R-flavored: read `_targets.R`, files in `R/`, prior ADRs. If you find a contradiction, surface it.

> The code in `R/clean_treatment.R` drops anyone with `tr == NA`. You just said missingness gets imputed in cleaning. Which is right?

Code and `CONTEXT.md` are the default reference here — column names and cleaning rules are checkable from the code that *names* and *transforms* them, no file-opening required. But when reading the code leaves the contradiction unresolved, or seeing the actual distribution would change which question matters, open the data and look. Inspect to sharpen the questions; don't run the planned analyses.

### Update CONTEXT.md inline

When a glossary, variable, or population entry is resolved, update `CONTEXT.md` right there. Don't batch them up — capture them as they happen. Use the format in [CONTEXT-FORMAT.md](./CONTEXT-FORMAT.md).

`CONTEXT.md` is for static facts only — definitions and scope. Do not treat it as a spec, a scratch pad, or a record of methodology choices:

- **Methodology choices** (identification strategy, estimator, prior, sample restriction *with a reason*, missing-data rule, etc.) → ADR.
- **Implementation details** (which package, which function, which file) → code comments.
- **Result interpretations** → paper draft.

### Offer ADRs sparingly

Only offer to create an ADR when all three are true:

1. **Hard to reverse** — the cost of changing your mind later is meaningful (the choice changes the findings, or invalidates work already done).
2. **Surprising without context** — a future reader (collaborator, referee, future-you) will wonder "why did they do it this way?"
3. **The result of a real trade-off** — there were genuine alternatives and you picked one for specific reasons.

If any of the three is missing, skip the ADR. Examples of what qualifies — and the format — are in [ADR-FORMAT.md](./ADR-FORMAT.md).

## Handoff

When enough has been grilled to draft a coherent plan, the natural next step is `to-analysis-plan`, which synthesises the conversation into a markdown plan document. Mention it; don't force it.

</supporting-info>
