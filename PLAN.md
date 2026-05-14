# Research Skills Plan

**Status:** Design complete; implementation not yet started.
**Created:** 2026-05-14
**Origin:** Adaptation of [Matt Pocock's agent skills](reference/mattpocock_skills/) (28 skills) for social-science research workflows.

This document records every decision made during the design interview so a future session can implement without re-deriving the design. Each section cites the interview question (Q1–Q20) it came from.

---

## 1. Why this exists

Matt Pocock's skill suite delivers engineering discipline for AI-assisted coding (TDD, diagnose, grill-with-docs, to-prd → to-issues → triage, etc.). It hard-assumes a TypeScript/Node stack, a software-product mental model (PRDs, features, vertical slices through schema → API → UI), and software-product failure modes (tests pass/fail).

Social-science research has different artifacts (analysis plans, data dictionaries, methodology decisions, tables, figures), a different unit of work (estimations, robustness sweeps, validation checks), and different failure modes (wrong-signed coefficients, non-convergent MCMC, omitted confounders, p-hacking risk). Direct use of Matt's skills mostly works but leaves the biggest research-specific leverage on the floor.

This plan defines a **15-skill research-flavored suite** that keeps Matt's framework spine (grill → plan → issues → autonomous slice) and adapts it for an R + `targets` (primary) and Python + `marimo` (later) stack doing regression / causal / Bayesian work.

## 2. Goals (Q1)

In priority order:
1. **The plan→issues→autonomous workflow** — extensive grilling produces a plan; plan slices into GitHub Issues; user works issues one at a time (some autonomously, some hands-on).
2. **Methodological discipline** — force structured commitments on identification strategy, priors, sensitivity, sample restrictions, so the agent's speed doesn't override care.
3. **Engineering hygiene for analysis code** — treat pipelines as real software (testable, navigable, documented).
4. **Project memory & handoff** — capture *why* behind decisions so future-you, collaborators, and agents can pick up.

Writing skills are **out of scope** for this iteration.

## 3. Core architectural decisions

### Repo unit (Q2)
- **One git repo per paper/project.**
- All artifacts (CLAUDE.md, CONTEXT.md, ADRs, issues) are repo-scoped. No shared-program repo.

### Memory artifacts (Q5)
Clean split between *static facts* and *immutable decisions*:

```
repo/
  CONTEXT.md          # static facts, freely edited
    glossary:         # paper-terminology definitions
      treatment   — exposure ≥1 day to program X
      outcome     — log(earnings_t+1)
    variables:        # code/data column ↔ paper term map
      tr          — binary treatment indicator (= treated)
      earn1       — earnings one year post (= outcome)
    population:       # who's in / out
      workers aged 25–55 in sample S

  docs/adr/           # numbered, immutable, supersede-don't-edit
    0000-template.md
    0001-did-twfe.md       — Why DiD with two-way FE
    0002-priors.md         — Weakly-informative N(0, 2.5) on coefs
    0003-sample.md         — Exclude pre-2008 (data quality)
    0004-missing.md        — Listwise deletion; sensitivity in 0005
```

ADRs record methodology choices (identification strategy, estimator, prior choice, sample cuts, missing-data handling). They are citable from issues, the plan, and code comments.

### Issue tracker (Q8)
**GitHub Issues only.** Private repos OK. `gh` CLI for the agent's access.

### Issue structure (Q9, Q10)
- **No fixed template.** Issues are flexible in shape.
- **Mandatory `Depends on:` field** in issue body listing the issue numbers that block this one.
- **Top-level epic issue (or PLAN.md)** ties the whole decomposition together with links to all child issues. Captures the goal-to-tasks decomposition as a dependency DAG.
- Mermaid diagram of the DAG is *optional*, not required.
- **Sub-issues feature is NOT used** (user unfamiliar with it; `Depends on:` field is sufficient).
- Example:

```markdown
## Issue #12  Add cleaning step for treatment indicator
Depends on: #3 (raw data load), #5 (codebook ADR)
Touches targets: tar_target(treatment_clean)

[work description, acceptance criteria]
```

### Autonomous scope (Q3)
What the agent may execute autonomously from an issue:
- ✅ Pipeline plumbing & data work
- ✅ Specified estimations (named spec, named data, named output)
- ✅ Robustness & sensitivity sweeps (pre-specified variations)
- ❌ Methodological exploration (identification strategy choice, MCMC re-parameterization, etc.) — stays with the user

### Triage states (Q13)
Lightweight, four states:
- `needs-info` — grilling required (methodology or spec gap)
- `ready` — methodology locked, spec written, deps closed
- `awaiting-review` — agent has finished; user needs to inspect
- `done` — closed

**Agent behavior:** always inspect every result. Agent's job is to surface "this looks off" in issue/PR comments — wrong sign, divergent chains, balance failures — *not* to gate on its own state machine. The user inspects everything regardless.

### Validation focus (Q12)
By far the biggest priority is **data validation & sanity checks**, not code-level tests. Synthetic-recovery and inference-diagnostics (MCMC convergence etc.) stay ad-hoc; not skill-scaffolded for v1.

## 4. The 15-skill suite

### Planning chain

#### 1. `grill-with-docs-research` *(adapted from Matt's `grill-with-docs`)*
- Relentless Socratic grilling, one question at a time.
- **Stays generic** — no method-specific question banks (Q11). Tasks are too diverse (data-cleaning plan vs. analysis plan vs. methodology choice).
- Updates `CONTEXT.md` (glossary, variables, population) and `docs/adr/*.md` **inline as decisions crystallize**.
- Differs from Matt: writes to research artifacts, not software PRDs.

#### 2. `to-analysis-plan` *(adapted from Matt's `to-prd`)*
- Synthesizes current conversation into a single plan doc at any moment.
- **No fixed template** (Q6) — structure emerges from project type and stage (exploratory ≠ confirmatory; descriptive ≠ causal ≠ Bayesian-methods).
- Output is a flexible markdown doc the user can hand off, edit, register, or ignore.

#### 3. `to-issues-research` *(adapted from Matt's `to-issues`)*
- Reads the analysis plan + CONTEXT.md + ADRs.
- Decomposes goal into discrete, partly-parallelizable tasks (the dependency DAG).
- Writes each task as a GitHub Issue with `Depends on:` field referencing other issue numbers.
- Writes/updates the top-level epic issue (or PLAN.md) tying everything together.

#### 4. `triage-research` *(adapted from Matt's `triage`)*
- Moves issues through the lightweight 4-state machine.
- Confirms an issue is "ready": methodology locked, spec written, dependencies closed, acceptance criteria present.
- Lighter than Matt's; no agent-brief artifact required.

### Execution & quality

#### 5. `validate-data` *(NEW — the priority skill)*
- Heavy-lift skill for data validation and sanity checks.
- For R-first build: scaffolds checks via `pointblank` and/or `assertr`. Inline checks inside `tar_target()` nodes.
- Catches: out-of-range values, missingness changes after data refresh, ID uniqueness violations, broken joins, distributional drift, balance failures.
- Surfaces failures loudly; never silently passes a stale dataset.

#### 6. `tdd-research` *(adapted from Matt's `tdd`)*
- Lightweight. Code-test cases only (helper functions, cleaning utilities). Uses `testthat` (R).
- Skip integration-style or full red-green-refactor discipline.
- Not a priority — code tests are "occasional" in user's workflow.

#### 7. `diagnose-research` *(adapted from Matt's `diagnose`)* — **two-track**
- **Track A (code/pipeline bug):** Matt's reproduce → minimise → hypothesise → instrument → fix → regression-test loop, adapted for `targets` failures and (later) marimo cell failures.
- **Track B (analytical surprise):** code runs cleanly, result is wrong-signed / non-convergent / fails-balance. Different flow: revisit assumptions → check data → re-examine spec → judge whether result is real or a methodology gap.
- Skill asks user (or detects) which track at entry.

#### 8. `improve-research-pipeline` *(adapted from Matt's `improve-codebase-architecture`)*
- **Two combined audits** (Q16):
  - **Pipeline-shape audit:** reads `_targets.R`. Surfaces shallow steps, duplicated logic across nodes, oversized monolithic steps, I/O mixed into computation, hidden state.
  - **Methodology-risk scan:** flags garden-of-forking-paths patterns — multiple alternative specs reachable from the same target, spec choices buried in conditionals instead of ADRs, results-dependent branching, hardcoded sample restrictions not recorded in CONTEXT.md.
- Concrete refactor proposals tied to specific targets nodes.
- Likely the most distinctively valuable skill in the suite.

### Exploration

#### 9. `probe` *(NEW)* — quick exploratory probes for ideas not yet committed to the pipeline
- For ideas the user wants to test before earning a place in `_targets.R`: "what if I drop post-2008 data?", "is there enough variation to identify Y?", "does this prior break convergence?", "does this finding hold under a quick robustness check?"
- Distinct from `prototype-research`: probes are about a *substantive question*, not engineering prototyping of pipeline-bound code. They live **outside** the targets DAG by design.
- **Lifecycle:**
  1. One round of grilling to crisp the question/hypothesis.
  2. Skill creates `probes/NNNN-slug/` with a `scratch.R` (or `scratch.py`/`scratch.qmd`) for the user to explore. Scratch code is gitignored.
  3. User explores at their own pace.
  4. On return, skill captures a **verdict**: `pursue` / `discard` / `inconclusive`.
  5. Skill writes `probes/NNNN-slug/VERDICT.md` (always committed): question, what was tried, findings, verdict, links to follow-up artifacts.
  6. If `pursue`: skill helps draft the downstream artifact (a new ADR, a new issue, an addition to the analysis plan, or a new pipeline step).
  7. If `discard` / `inconclusive`: VERDICT.md stands as a record so future-you searches probes before re-exploring the same dead end.

```
probes/
  0001-post2008-sensitivity/
    scratch.R         # gitignored
    VERDICT.md        # committed
  0002-prior-shrinkage/
    scratch.py        # gitignored
    VERDICT.md        # committed
```

### Generics kept as-is

#### 10. `zoom-out` — generic; no changes.
#### 11. `handoff` — generic; only retarget at research artifacts (plan, ADRs, open issues).
#### 12. `write-a-skill` — meta; unchanged. Needed to author the rest of the suite.
#### 13. `prototype-research` *(adapted from Matt's `prototype`)*
- Replace Matt's logic-vs-UI branches with **exploratory-vs-spec-locked branches**:
  - **Exploratory:** throwaway marimo notebook / `scratch.R` to interrogate data before any pipeline commitment.
  - **Spec-locked:** Matt's CLI-style prototype, used to stress-test a state machine or computation before embedding it in the pipeline.

### Infrastructure

#### 14. `setup-research-skills` *(NEW)*
- One-time per-repo configurator.
- Writes/updates `CLAUDE.md` with the agent skills block.
- Creates `CONTEXT.md` from template.
- Creates `docs/adr/0000-template.md`.
- Creates GitHub triage labels (`needs-info`, `ready`, `awaiting-review`).
- Re-targeted version of Matt's `setup-matt-pocock-skills`.

#### 15. `bootstrap-research-repo` *(NEW)*
- Fresh-repo initialization. Calls `setup-research-skills` as a sub-step.
- Creates:
  - `targets` skeleton (`_targets.R`, `R/` directory)
  - `data/` (raw, derived, processed subdirs) with `.gitkeep` and `.gitignore` patterns to keep large/raw data out of git
  - `renv` lockfile initialization
  - Research-flavored `.gitignore` (rendered outputs, sensitive data patterns, etc.)
  - Skeleton README

## 5. What was dropped, and why

| Matt skill | Reason dropped |
|---|---|
| `caveman` | Compression mode adds a state to remember; low priority. |
| `edit-article` | Writing de-prioritized. |
| `obsidian-vault` | Specific to Pocock's personal vault path. |
| `migrate-to-shoehorn` | TypeScript-specific. |
| `scaffold-exercises` | AI Hero course-specific. |
| `setup-pre-commit` | Adds friction; format manually or via editor on save. |
| `git-guardrails` | User opted out; trust the model + own review. |
| `review` | User opted out. |
| `writing-beats` | Writing de-prioritized. |
| `writing-fragments` | Writing de-prioritized. |
| `writing-shape` | Writing de-prioritized. |
| All deprecated (`design-an-interface`, `qa`, `request-refactor-plan`, `ubiquitous-language`) | Already deprecated in Matt's repo. |

Also **not** built in v1 (Q18 candidates declined):
- `archive-replication-package` — useful but periodic; revisit later.
- `pre-register-analysis` — useful only for projects that get pre-registered; revisit later.
- Synthetic-recovery / MCMC-diagnostic dedicated skills — stay ad-hoc.

## 6. Build sequence (Q19)

**End-to-end thin slice first.** Build the smallest viable chain end-to-end before deepening any single skill.

1. `setup-research-skills` (need it to bootstrap any work)
2. `grill-with-docs-research`
3. `to-analysis-plan`
4. `to-issues-research`
5. `triage-research`
6. **Run on one real research project. Find what's broken.**
7. Then deepen, in rough order of leverage:
   - `validate-data`
   - `probe`
   - `improve-research-pipeline`
   - `diagnose-research`
   - `bootstrap-research-repo`
   - `prototype-research`
   - `tdd-research`
   - `zoom-out`, `handoff`, `write-a-skill` (mostly drop-in from Matt)

## 7. Language strategy (Q20)

**R-first; Python is v2.**

For the initial build:
- All skills written R-flavored.
- `references/` docs focus on R: `targets`, `pointblank`/`assertr`, `testthat`, `renv`, `tidyverse`, `cmdstanr`/`brms`.
- No marimo reference doc yet.
- No language-detection logic.

**Python v2** later will add:
- Language detection (presence of `_targets.R` vs marimo `.py`, `renv.lock` vs `uv.lock`).
- Parallel reference docs (`references/python-*.md`): `pandera`, `pytest`, `uv`, `marimo`.
- A dedicated `references/marimo.md` because marimo's reactive cell model is novel enough that the model needs scaffolding to use it well.

## 8. Plugin structure (implementation choice — confirm at build time)

Recommendation (not explicitly decided, but a sensible default):
- Single plugin, mirroring Matt's `.claude-plugin/` + `skills/` + `references/` + `scripts/` layout.
- Distributable independently if useful.

## 9. Open questions for the build session

These weren't decided in the interview and should be revisited at implementation time:

1. **Plugin packaging.** One plugin vs. multiple? Hosted where? Versioning scheme?
2. **CLAUDE.md template** for research repos: what conventions, package preferences, style notes does it bake in?
3. **Naming convention for ADRs.** Numbered `0001-name.md` (Matt's pattern) or topic-grouped?
4. **GitHub label vocabulary** beyond the 4 triage states (do we add `methodology`, `plumbing`, `sweep` labels to mirror issue type?).
5. **How does `to-issues-research` interact with `targets`?** Should it read `_targets.R` to derive dependencies, or stay decoupled and rely on the user/grilling to specify `Depends on:`?
6. **What does an "epic" issue look like exactly?** A GitHub Issue with checkboxes for all child issues, or a separate `PLAN.md` in the repo, or both?
7. **HITL vs AFK marking on issues.** Matt's framework has this; the user opted for lighter states. Do we still want a per-issue label to flag "agent can run this end-to-end" vs. "user-in-loop required"?
8. **Anomaly-flagging convention.** When agent finishes an issue and the result looks off, what's the exact convention? Comment with a specific prefix (`⚠️ ANOMALY:`)? A separate `result-flagged` label even though the state machine doesn't have one?

## 10. References

- [Matt Pocock's skills repo (vendored)](reference/mattpocock_skills/) — primary source material.
- The interview that produced this plan: 21 questions covering goals (Q1), repo unit (Q2), autonomous scope (Q3), plan artifact (Q4), memory artifacts (Q5), PRD template (Q6), planning skill shape (Q7), tracker (Q8), issue templates (Q9), dependencies (Q10), methodology grilling (Q11), validation kinds (Q12), triage states (Q13), diagnose shape (Q14), disposition review (Q15), architecture skill (Q16), pre-commit/guardrails (Q17), new skills (Q18), build sequence (Q19), language strategy (Q20), probe record lifecycle (Q21).
