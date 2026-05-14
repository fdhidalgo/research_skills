# Analysis plan format

Writing-style reference for analysis plans produced by `to-analysis-plan`. Consult this every time a plan is drafted. The plan is a markdown file at `docs/plans/NNNN-slug.md`; the seed structure is not a fixed template — what every plan must carry is described as a set of roles, not a section list.

## Location and numbering

Plans live in `docs/plans/` with sequential numbering: `0001-slug.md`, `0002-slug.md`, etc.

- Numbering: scan `docs/plans/` for the highest existing `NNNN-*.md`, increment by one. No `0000-template.md` — plans don't use a seed file; the structure flexes per project.
- Slug: kebab-case, topical. Describes the distinctive contribution of *this* plan, not a generic version label. `0001-initial-analysis-plan.md`, `0002-post-referee-revision.md`, `0003-tighten-sample-after-data-refresh.md`.
- Create `docs/plans/` lazily — only when the first plan is written.

## The four accomplishment-principles

Every plan must serve four *roles*. These are not section headings — they're what the document must accomplish, regardless of the headings the model chooses.

1. **Communicate the goal.** What is this project actually about? A substantive research question, a methodological contribution, a data-construction target, a replication scope, a measurement goal. Different project types phrase the goal differently — what matters is that someone landing on the plan cold can name the thing being built.

2. **Communicate the approach.** How will the goal be reached? Identification strategy, method, construction logic, replication protocol, exploratory probes — whichever maps to the goal. Concrete enough that a reader can name the central artifact (the primary regression, the index definition, the model class, the simulation design) without ambiguity.

3. **Name concrete deliverables.** What will exist when this plan is executed? Tables, figures, an index, validation reports, posterior summaries, a method-validation appendix, a replication package, a parameter recovery study. Whatever this project will produce. Even when the list is loose, the deliverables must be *namable*, because they're what `to-issues-research` will decompose into work units.

4. **Surface open work.** What's still mobile? An **Open questions** section names every unresolved decision, points back to `grill-with-docs-research` as the resolution path, and marks each entry with rough priority. This section is the handoff back to grilling and the explicit audit-trail entry for "what was still being decided when this snapshot was taken." Keep it even when empty; an empty Open questions section is itself informative.

The model picks the headings that best carry these roles for the project at hand. A causal paper might have headings like *Research question / Identification / Sample / Primary specification / Robustness / Tables and figures / Open questions*. A measurement paper might have headings like *Construct / Source data / Construction steps / Validation / Comparisons to existing indices / Open questions*. A methods paper might have headings like *Contribution / Estimator / Simulation design / Real-data application / Diagnostics / Open questions*. None of these is the right answer — they're examples of how the four roles get carried by different headings.

## Three structurally-different shape sketches

These exist to demonstrate that *shape itself* flexes by project type — not just content. Use them as reference points, not templates.

### Sketch A — Causal paper with staggered DiD

```markdown
# Analysis Plan 0001: Effect of Program X on Earnings

## Research question
Does enrolment in Program X causally increase earnings 2 years post-exit?
(Estimand: ATT, restricted to compliers per ADR-0002.)

## Identification
Staggered DiD across cohorts entering 2010–2018. Parallel-trends assumption
defensible because <reason>. See ADR-0001 for the full identification story.

## Sample
Workers as defined in CONTEXT.md Population. No new exclusions in this plan.

## Primary specification
Callaway–Sant'Anna estimator (per ADR-0003), clustered SEs at state-year (ADR-0005),
outcome = `earn2` (= **outcome** per CONTEXT.md).

## Robustness
- Sensitivity to anticipation: Sun & Abraham re-estimation.
- Negative controls: pre-program outcomes (placebo).
- Cluster level: state vs. state-year vs. industry-year.
- Imputation: M=5 multiple imputation as sensitivity to ADR-0007 (listwise).

## Tables and figures
- Table 1: descriptive stats by cohort × treatment.
- Table 2: primary ATT estimates with sensitivity columns.
- Figure 1: event-study plot.
- Figure 2: cohort heterogeneity.

## Open questions
- (high) Should anticipation effects bound the pre-period? Grill on this before robustness lock.
- (low) Clustering for the Bayesian sensitivity in ADR-0009: open.
```

### Sketch B — Measurement / index-construction paper

```markdown
# Analysis Plan 0001: Construction of the Subnational Capacity Index

## What we're building
A unit-level (province-year) index of state capacity covering 1990–2020, comparable
across countries, designed to be drop-in for downstream cross-national analyses.

## Source data
- Administrative tax records (per CONTEXT.md Variables `tax_*` series).
- Census enumeration coverage (per ADR-0002 on geographic harmonisation).
- World Bank governance indicators (as auxiliary anchors, not inputs).

## Construction logic
Three-step:
1. Z-score component indicators within country-year.
2. Aggregate via Bayesian factor model (per ADR-0004 on prior choices).
3. Anchor to comparator indices via post-hoc rescaling.

## Validation
- Convergent: correlation with WB indicators in overlapping country-years.
- Discriminant: index should *not* track GDP per capita beyond a sanity threshold.
- Test-retest: rebuild on a held-out 10% sample of provinces; agreement ≥ 0.85.

## Deliverables
- The index dataset (province-year panel).
- A method-validation appendix with the three validation checks above.
- An R package wrapping the construction code.

## Open questions
- (high) Provincial boundary changes 2007–2012 — ADR needed on harmonisation rule.
- (medium) Whether to include a coverage-uncertainty field per cell.
```

### Sketch C — Bayesian methods paper

```markdown
# Analysis Plan 0001: Hierarchical Mixed-Outcomes Estimator for Field Trials

## Contribution
A hierarchical Bayesian estimator that handles mixed continuous / binary / count
outcomes in factorial field trials with partial pooling across treatment arms.

## Estimator
Latent-variable formulation: continuous outcomes modeled directly; binary via
probit on a latent score; counts via negative binomial with shared random
effect at the arm level. Priors per ADR-0001 (regression coefficients) and
ADR-0002 (group-level scales).

## Simulation design
Parameter grid (n_arms × outcome-mix × pooling strength × signal/noise). 200 reps
per cell. Recovery metric: bias, coverage, posterior mean error.

## Real-data application
Re-analysis of Bloom et al. (2024) wellbeing-program trial. Compare estimator
output to the published per-outcome analyses (reported in Table 3 of the paper).

## Diagnostics
- Convergence: R-hat < 1.01 on all chains; ESS > 400 per arm-level parameter.
- Posterior predictive check: distributional fit on each outcome type.
- Prior sensitivity: re-run with the alternate prior set in ADR-0003.

## Deliverables
- The estimator (Stan implementation + R wrapper).
- Simulation results figure + table.
- Real-data re-analysis table.
- Method-paper draft.

## Open questions
- (high) Whether to pre-register the simulation grid before running.
- (medium) Numerical-stability handling for very small signal/noise cells.
```

These three sketches are deliberately different in their headings and emphasis. Pick a shape that fits the project; don't try to force one of these onto a different project type.

## Successor preamble

When a prior plan exists (i.e., this plan's number is ≥ 0002), the file begins with a fixed two-line preamble *before* the H1:

```markdown
> Supersedes: [docs/plans/NNNN-prior-slug.md](NNNN-prior-slug.md)
>
> **What changed**: <one paragraph naming the trigger and the substantive deltas.>

# Analysis Plan NNNN: <title>

<body...>
```

The trigger is concrete — e.g., "Referee 2 asked for a different identification strategy"; "Data refresh in March 2026 surfaced a coding break pre-2008"; "Mechanism evidence from probe 0003 changed the central spec from intent-to-treat to compliance-weighted"; "Co-author requested a registered-report shape for Stage 1 submission." The "what changed" paragraph names the trigger *and* the substantive deltas — not a diff, but a reader-oriented summary of which decisions moved.

The first plan (number 0001) has no preamble.

## Citation and vocabulary conventions

- **Cite ADRs by number.** `per ADR-0003`. Don't restate the ADR's decision in the plan body — the ADR owns it, and the plan should be readable as a thin overlay on the ADR set.
- **Use `CONTEXT.md` vocabulary.** Paper terms from the Glossary, code identifiers from Variables, sample-membership claims consistent with Population. If the plan needs a term that conflicts with the glossary, flag it in Open questions; don't silently coin a synonym.
- **Don't invent ADRs in the plan body.** If a plan-relevant decision has no ADR yet, *note in Open questions* that the decision should be promoted to an ADR via grilling. The plan itself shouldn't be the place a methodology choice first appears.

## "Open questions" entry format

One bullet per gap. Each bullet has:

- **A priority tag.** `(high)`, `(medium)`, `(low)`. High = blocks the plan from being executed at all. Medium = blocks a particular section. Low = nice-to-resolve but plan can ship as-is.
- **A short statement of the gap.** What's not yet decided.
- **A resolution path.** Usually `grill on X` — pointing back to `grill-with-docs-research` with a specific topic. Sometimes `decide via probe NNNN` (for empirical questions that need a quick scratch test). Sometimes `await external input` (a co-author's reply, IRB clarification, data steward question).

Example:

```markdown
## Open questions

- (high) Whether anticipation effects bound the pre-period. Grill on this before
  locking robustness; touches ADR-0003 and possibly forces a new ADR.
- (medium) Cluster level for the Bayesian sensitivity. Grill on this once primary
  spec is locked.
- (low) Whether Table 2 should be cohort-stratified. Decide when drafting tables.
```

Keep the section even when there are no open questions. Write `None at time of writing.` — that's itself an audit-trail entry.

## What does NOT belong in a plan

- **Methodology choices.** Identification strategy, estimator, prior, sample restriction *with a methodological reason*, missing-data rule. → ADR.
- **Implementation details.** Which R package, which function, which `tar_target()` node, which file. → code comments.
- **Variable definitions.** Paper-term ↔ code-identifier mappings. → `CONTEXT.md` Variables.
- **Sample-membership rules.** Who's in, who's out, and why. → `CONTEXT.md` Population (when structural) or ADR (when methodologically motivated).
- **Results or interpretation.** "The ATT is 0.12 log-points." → paper draft.
- **A re-statement of an ADR's decision.** → cite the ADR by number.

If you find yourself writing any of these into the plan, stop. The plan is a thin overlay on the ADRs and `CONTEXT.md`, not a duplication of them.

## Lazy creation

If `docs/plans/` doesn't exist, create it the first time `to-analysis-plan` writes a plan. There is no template seed file — the first plan is `0001-<slug>.md` written directly using the shape sketches above as references.

---

*Skill: [`SKILL.md`](./SKILL.md). Related: `CONTEXT.md` (vocabulary), `docs/adr/` (methodology decisions), `to-issues-research` (downstream consumer of this plan).*
