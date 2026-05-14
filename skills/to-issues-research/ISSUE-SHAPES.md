# Issue shapes

Writing-style reference for the issues produced by `to-issues-research`. Consult this whenever the skill is decomposing a plan or composing an issue body. The issue tracker contract — CLI conventions and the one required field — lives in `docs/agents/issue-tracker.md`; this file layers the research-flavored conventions on top.

## Where issues live + the one required field

Issues are GitHub Issues. The only mandatory body field is **`Depends on:`** referencing the blocking issue numbers — that's what captures the task DAG without using GitHub's sub-issues feature. If an issue has no prerequisites, omit the field rather than writing `Depends on: none`. Everything else in the body is flexible and adapts to the work.

```markdown
## Add cleaning step for treatment indicator
Depends on: #3 (raw data load), #5 (codebook ADR)

[rest of body]
```

See `docs/agents/issue-tracker.md` for the full CLI cheatsheet.

## Label vocabulary recap

Two flavours, used differently:

- **State labels** (one per work-unit issue) — `needs-info` → `ready` → `awaiting-review` → `done`. Move with the issue as it progresses. New issues land in `needs-info` or `ready`.
- **Kind labels** (categorical, stay with the issue for life) — `epic` (plan container; doesn't get a state label) and `autonomous-ok` (regular work that falls within the agent's autonomous scope; coexists with a state label).

`to-issues-research` applies the state label at creation, and applies `autonomous-ok` to issues whose category is plumbing / estimation / sweep. methodology-gap / validation / deliverable issues do not get `autonomous-ok`. Use the canonical strings recorded in `docs/agents/triage-labels.md` (the user may have overridden them at setup time).

## Sizing

**One issue ≈ one session.** Each issue should be plausibly completable in a single agent (or human) session. Prefer many small, dependent issues over a few large omnibus ones. Splitting issues is cheap; recovering from a half-finished giant one is not.

Signals an issue is too big:

- Touches several pipeline nodes that aren't tightly coupled.
- Combines task categories (e.g. plumbing **and** estimation **and** a sweep).
- Has many distinct deliverables.
- Body would naturally want multiple sub-headings of unrelated work.

When the skill spots one of these, propose a split using one of these three strategies. Each example is a one-paragraph sketch — not a template.

### Pipeline boundary — plumbing → estimation

A "build the analytic file and run the primary spec on it" issue is two issues. The first issue is the plumbing: `tar_target(analytic_file)` materialising the cleaned, joined, sample-restricted dataset, with acceptance criteria on row counts, ID uniqueness, and join coverage. The second issue is the estimation, with `Depends on: #<plumbing>`, citing the relevant identification ADR and naming the output target (`tar_target(primary_att)`). Splitting cleanly separates "did the data come in right?" from "did the estimator produce a sensible coefficient?" — and lets the second issue be picked up after the first lands in `awaiting-review`.

### Spec boundary — primary → robustness

"Run the primary spec plus the four sensitivities the plan calls for" is two issues. The first issue is just the primary spec — locked, single output, easy to review. The second issue is the robustness sweep, with `Depends on: #<primary>`, listing the variations exactly as the plan enumerates them. This keeps the primary result reviewable on its own terms before the sweep arrives and lets the sweep be revisited without re-running the primary.

### Variation boundary — sweep → N variant issues

When each variation in a sweep is non-trivial (touches its own data prep, has its own diagnostics, or takes meaningful compute), split the sweep into one issue per variant, each `Depends on: #<primary>`. Put the distinguishing detail in the title (*Sweep: cluster level — industry-year*; *Sweep: cluster level — state-year*). When variations are truly mechanical (same code, swap one argument), collapse into a single sweep issue.

**Lower bound: don't atomise to triviality.** An issue should be worth filing — not "rename one variable" or "fix a typo in a target name." The sizing rule is an upper bound on complexity, not a mandate to minimise.

## The six research task categories

Use these to bucket work as you decompose the plan.

- **plumbing.** Data load, cleaning, joining, harmonisation, sample restriction, anything that produces or modifies a `tar_target(...)` node carrying data rather than results. Acceptance criteria are usually about the produced object: row counts, ID uniqueness, join coverage, distributional sanity. Default state: `ready` once the spec is in the plan and the source data is known. Default `autonomous-ok`: **yes**.

- **estimation.** A *named* specification on *named* inputs producing a *named* output. The plan + ADRs together should pin down which estimator, which covariates, which clustering, which sample. The issue body cites those ADRs by number rather than re-deciding methodology. Default state: `ready` when the spec is locked. Default `autonomous-ok`: **yes**.

- **sweep.** Pre-specified variations of a primary estimation. Robustness, sensitivity, alternative samples, alternative clusterings, alternative priors. The plan enumerates them; the issue body lists them; the agent runs them. Default state: `ready`. Default `autonomous-ok`: **yes**.

- **methodology-gap.** Work blocked on a methodology decision that hasn't been made. The plan flagged the gap; until it's resolved (via `grill-with-docs-research` → ADR) the work can't be specified. Default state: `needs-info`. Default `autonomous-ok`: **no** — methodology exploration is outside the agent's autonomous scope (per PLAN.md §3).

- **validation.** Sanity checks, balance checks, MCMC convergence diagnostics, distributional checks against prior data refreshes, ID-uniqueness audits. The work itself is mechanical, but the *judgment* about whether a result is acceptable belongs to the user. Default state: `ready`. Default `autonomous-ok`: **no** — flagged as user-driven because the framing of "acceptable" is a methodological judgment.

- **deliverable.** Produce a specific table, figure, or output file with whatever formatting and labelling the plan describes. The judgment about whether the artifact reads well and serves its purpose stays with the user. Default state: `ready`. Default `autonomous-ok`: **no**.

The defaults are starting points. During the check-in the user can flip `autonomous-ok` either way per issue.

## Shape sketches

These exist to show the *flexibility* of issue bodies, not to template them. Pick a shape that fits the work.

### Sketch A — estimation issue (autonomous-ok)

```markdown
## Primary specification: ATT on earnings two years post-exit

Depends on: #14 (analytic file built), #5 (compliance-weighted sample ADR)

**Plan:** [docs/plans/0001-initial-analysis-plan.md](docs/plans/0001-initial-analysis-plan.md), §Primary specification.

Run the Callaway–Sant'Anna estimator per ADR-0003 on the analytic file built in #14, restricted to compliers per ADR-0005. Cluster SEs at state-year per ADR-0006. Outcome is `earn2` (= **outcome** per CONTEXT.md). Write the point estimate, SE, and event-study coefficients into `tar_target(primary_att)`.

**Acceptance**
- [ ] `targets::tar_make(primary_att)` runs clean.
- [ ] Output has the expected three components (ATT, SE, event-study coefs).
- [ ] Sign is consistent with the predicted direction in the plan; flag if not.
```

### Sketch B — sweep variant issue (autonomous-ok)

```markdown
## Sweep: cluster level — industry-year

Depends on: #17 (primary ATT)

**Plan:** [docs/plans/0001-initial-analysis-plan.md](docs/plans/0001-initial-analysis-plan.md), §Robustness — clustering.

Re-run the primary specification from #17, replacing the state-year clustering with industry-year. All other choices identical. Produce a single ATT + SE in `tar_target(sweep_cluster_industry_year)` and add the row to the robustness table per the plan.

**Acceptance**
- [ ] Output target built without errors.
- [ ] Comparison row appended to the robustness table.
- [ ] If SE moves by more than a factor of 2, flag in a comment for the user to inspect.
```

### Sketch C — methodology-gap issue (no autonomous-ok)

```markdown
## Decide: should anticipation effects bound the pre-period?

Depends on: (none — blocks #17, #18)

**Plan:** [docs/plans/0001-initial-analysis-plan.md](docs/plans/0001-initial-analysis-plan.md), §Open questions, high-priority entry.

The plan flagged anticipation effects as an unresolved high-priority gap. Resolving it shapes how the pre-period is bounded in the primary spec (#17) and the event-study figure (downstream).

**Resolution path:** Run `grill-with-docs-research` on the anticipation question. The outcome should be a new ADR (likely supersedes or amends ADR-0003) capturing the decision. After the ADR lands, re-run `to-analysis-plan` to produce a successor plan, then re-decompose with `to-issues-research`.

(State: `needs-info`. No `autonomous-ok` — methodology exploration is HITL.)
```

## Open-questions → issues conversion

When the plan's "Open questions" section is walked item-by-item, an entry the user opts to materialise becomes a `needs-info` issue with:

- A title that names the gap (not the priority — the priority lives in the plan and isn't reproduced on the issue).
- A `**Plan:**` line pointing back to the originating plan + section, like the sketches above.
- A `**Resolution path:**` block that names `grill-with-docs-research` as the next move (or another resolution where appropriate — *await co-author input*, *decide via probe NNNN*).
- No `autonomous-ok` label.
- `Depends on:` only when the gap is actually blocked on another known unit; "this question depends on a methodology decision" is *not* a dependency — it's the issue itself.

Entries the user leaves in the plan stay in the plan. They'll resurface in any successor plan unless resolved or explicitly dropped.

## Title and citation conventions

- **Titles use `CONTEXT.md` glossary terms.** "ATT on earnings two years post-exit," not "ate2 on earn2." Reserve code identifiers for the body.
- **Cite ADRs by number** (`per ADR-0003`). Don't restate the decision.
- **Reference the plan** with `**Plan:** [docs/plans/NNNN-slug.md](docs/plans/NNNN-slug.md), §<section>` near the top of the body.
- **For split sets**, put the distinguishing detail in the title. *Sweep: cluster level — state-year*, not *Sweep #4*.

## What does NOT belong in an issue

- **Methodology decisions.** Identification strategy, estimator, prior, sample restriction with a methodological reason. → ADR. The issue cites it.
- **Results or interpretation.** Numbers go into `tar_target` outputs or paper drafts, not issue bodies.
- **Variable redefinitions.** Paper-term ↔ code-identifier mappings. → `CONTEXT.md` Variables.
- **Long preambles re-explaining the plan.** The plan is the source of truth; the issue links to it.
- **A re-statement of an ADR's decision.** → cite the ADR by number.

If you find yourself writing any of the above into an issue, stop. The issue is a thin overlay on the plan + ADRs + `CONTEXT.md`, not a duplication of them.

---

*Skill: [`SKILL.md`](./SKILL.md). Related: `docs/agents/issue-tracker.md` (CLI + Depends-on contract), `docs/agents/triage-labels.md` (label vocabulary), `to-analysis-plan/PLAN-FORMAT.md` (the shape the plan arrives in), `grill-with-docs-research` (resolution path for `needs-info` issues).*
