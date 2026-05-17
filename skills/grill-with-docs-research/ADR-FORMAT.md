# ADR format

Writing-style reference for methodology ADRs in a research repo. Consult this every time `grill-with-docs-research` proposes a new ADR. The seed template (the empty starting file) lives in [`setup-research-skills/adr-template.md`](../setup-research-skills/adr-template.md); this file describes when and how to use it.

## Location and numbering

ADRs live in `docs/adr/` with sequential numbering: `0001-slug.md`, `0002-slug.md`, etc.

- `0000-template.md` is the seed (copied in by `setup-research-skills` from `adr-template.md`). Don't overwrite it; start real ADRs at `0001`.
- Numbering: scan `docs/adr/` for the highest existing real ADR (skipping `0000-template.md`) and increment by one.
- Slug: kebab-case, descriptive enough to scan in a list. `0003-exclude-pre-2008.md` over `0003-sample.md`.
- Create `docs/adr/` lazily — only when the first ADR is accepted.

## Template

The seed already defines the structure:

```markdown
# ADR-NNNN: <decision title>

- **Status:** Proposed | Accepted | Superseded by ADR-NNNN | Reversed by ADR-NNNN
- **Date:** YYYY-MM-DD

## Context
## Decision
## Alternatives considered
## Consequences
```

Only **Status** and **Date** are required. Everything else is optional — an ADR can be a single paragraph in the **Decision** section. The value is in recording *that* a methodology decision was made and *why*; not in filling out sections.

Use the sections when they earn their place:

- **Context** — only if the forcing constraint (data feature, design constraint, prior literature) isn't already obvious from the project.
- **Alternatives considered** — only if the rejected options are worth remembering, especially the ones a future reader (or referee) will want to suggest again.
- **Consequences** — only if there are non-obvious downstream effects: robustness checks this triggers, ADRs this makes reachable or unreachable, what the choice commits you to.

## Status lifecycle

An ADR carries one of:

- `Proposed` — drafted during grilling; nothing builds on it yet. **Edit it in place freely.** There's nothing to audit until something depends on it, so revise the body as your thinking moves.
- `Accepted` — the working decision; the plan, issues, or code now build on it. **From here the record is append-only:** don't edit the body — supersede it.
- `Superseded by ADR-NNNN` — a later ADR replaced this decision with a different one.
- `Reversed by ADR-NNNN` — a later ADR went back to an approach this one had moved away from. Use this instead of *Superseded* when you're reverting rather than advancing; the distinction is itself worth recording in a research trail.

The `Proposed → Accepted` transition is the lock, and it is **yours to flip, not the agent's**. The natural triggers are the points where you actually commit: an issue or the analysis plan starts depending on the decision, you file a pre-registration, you hit an analysis freeze, or you make first contact with outcome data. The skill may *offer* to mark an ADR `Accepted` when it sees work beginning to depend on it; it never flips it for you. New ADRs are born `Proposed` — don't default a fresh one to `Accepted` unless the user says it's locked.

**Revisiting an Accepted decision is normal research, not a failure.** You learned something — that is the process working. The append-only rule is not there to discourage changing your mind; it is there so that when you do, the trail shows the change was principled and not outcome-driven. Recording the pivot *protects* you. To revise an Accepted decision, write a new ADR that supersedes (or reverses) it and flip the old one's **Status** line to point at the new number. A supersede ADR can be four lines — what changed, what you learned, what it affects, `Supersedes ADR-NNNN`. Don't edit the superseded body: its value is being a faithful record of what you believed then, and why you moved.

## When to offer an ADR

All three of these must be true:

1. **Hard to reverse** — the cost of changing your mind later is meaningful. The choice changes the findings, or invalidates work already done, or commits you to a specific robustness test.
2. **Surprising without context** — a future reader (collaborator, referee, future-you) will look at the code or the table and wonder "why on earth did they do it this way?"
3. **The result of a real trade-off** — there were genuine alternatives and you picked one for specific reasons.

If a decision is easy to reverse, skip it — you'll just reverse it. If it's not surprising, nobody will wonder why. If there was no real alternative, there's nothing to record beyond "we did the obvious thing."

## What qualifies (research edition)

- **Identification strategy.** "We use DiD with two-way fixed effects." "We use IV with shift-share instruments." The structural assumption that turns observed differences into causal claims.
- **Estimator choice.** "We use Callaway–Sant'Anna over TWFE because of staggered adoption and heterogeneous effects." "We use doubly-robust estimation to guard against propensity-model misspecification."
- **Prior choices.** "Weakly-informative N(0, 2.5) on standardised coefficients." "Half-normal(0, 1) on group-level scales." Priors that materially influence inference and that a referee will want justified.
- **Sample restrictions with non-trivial reasons.** "Exclude pre-2008 cohorts because the income measure changes definition there." Restrictions that change the population being estimated and that a future reader would query.
- **Missing-data handling.** "Listwise deletion; sensitivity via multiple imputation in ADR-0007." "MICE with predictive mean matching." The choice is consequential and reversible only at the cost of re-running everything downstream.
- **Weighting / clustering level.** "Cluster SEs at the state-year level." "Survey weights from the public-use file applied throughout." Inferential choices that a referee will probe.
- **Outcome construction.** "Use inverse-hyperbolic-sine over log because of zero earnings." "Winsorise the top 1% by year." Transformations that change what the coefficient is estimating.
- **Deliberate deviations from the field's standard.** "We use OLS where the literature defaults to logit because the marginal-effect comparison is easier and the conditional distribution is approximately normal." Anything where a reasonable referee would assume you'd done the other thing.
- **Constraints not visible in the code.** "IRB restricts sub-group analysis below n = 50." "DUA forbids merging dataset X with dataset Y." Constraints that explain why the obvious thing wasn't done.
- **Rejected alternatives a future reader will want to suggest again.** "We considered synthetic control; rejected because pre-period is too short to fit the donor weights." Recording the rejection stops the same suggestion in six months.

If you're unsure whether a decision qualifies, run the three-criteria check explicitly. If two of three feel forced, skip the ADR.

---

*Seed template: [`setup-research-skills/adr-template.md`](../setup-research-skills/adr-template.md). Agent contract (how consumer skills read this file): `docs/agents/research-docs.md`.*
