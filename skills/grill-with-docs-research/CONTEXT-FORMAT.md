# CONTEXT.md format

Writing-style reference for the three sections of a research repo's `CONTEXT.md`. Consult this every time `grill-with-docs-research` resolves a glossary, variable, or population entry. The seed template (the empty starting structure) lives in [`setup-research-skills/context-template.md`](../setup-research-skills/context-template.md); this file describes how to *write into* it.

## Structure

```markdown
# CONTEXT

Static facts about this paper / project.

## Glossary

**treatment**:
Exposure ≥ 1 day to program X.
_Avoid_: enrolment, participation, dose

**outcome**:
Log-earnings two years after program exit.
_Avoid_: earnings, wages, income

## Variables

- `tr` — binary 0/1 indicator (= **treatment**)
- `earn2` — log of W-2 earnings two years post-exit (= **outcome**)
- `state_fips` — state of residence at baseline; used for clustering

## Population

- Workers aged 25–55 at program entry.
- Excludes self-employed at baseline — no W-2 record, so **outcome** is undefined.
- Excludes pre-2008 cohorts — data-quality break, see ADR-0003.

## Flagged ambiguities

- "effect" was used to mean both ATT and ATE — resolved: this paper estimates ATT throughout.
```

## Writing rules — Glossary

Paper-terminology definitions. One entry per concept.

- **Tight definitions.** One sentence max. Define what it *is*, not what it does.
- **Be opinionated.** When multiple words exist for the same concept, pick one and list the others under `_Avoid_`.
- **Flag conflicts explicitly.** If a term has been used ambiguously, resolve it and record the resolution in *Flagged ambiguities*.
- **Only paper-specific terms belong.** General statistics or programming terms (regression, covariate, NA) don't go here even when the project uses them.

## Writing rules — Variables

Two-column code ↔ paper-term map. The strings that appear in data and code, and which paper concept each one corresponds to.

Typographic convention, applied consistently:

- Backticks around code identifiers: `` `earn2` ``.
- **Bold** around paper terms: **outcome**.
- Em-dash separator between identifier and description.
- Trailing `(= **glossary-term**)` cross-reference when the variable directly realises a glossary concept. Omit the cross-reference for utility variables (IDs, weights, cluster keys, fixed-effect indicators) that don't map to a paper concept.

Example: `` `tr` — binary 0/1 indicator (= **treatment**) ``.

One bullet per variable. Keep the description short — one line. If a derivation needs more, it belongs in code, not here.

## Writing rules — Population

Sample inclusion and exclusion rules. Bullet list, one rule per bullet.

- Each bullet states a single inclusion or exclusion and the *reason*.
- When the exclusion is methodologically motivated, cite the ADR: `Excludes pre-2008 cohorts — data-quality break, see ADR-0003.`
- When the inclusion is structural (defining who the paper is about), no ADR needed: `Workers aged 25–55 at program entry.`

If a sample claim arises that doesn't fit any existing bullet, push back during the grilling session — either log it as a new bullet (and an ADR if it has a methodological reason) or drop it.

## Flagged ambiguities

Bottom-of-file section. Optional — only present once there's something to record.

When a term has been used in conflicting ways and you resolve it, log the resolution here. Example:

> "effect" was used to mean both ATT and ATE — resolved: this paper estimates ATT throughout.

The point is to leave a paper trail so the ambiguity doesn't re-emerge in three months when a collaborator picks up where you left off.

## What does NOT belong in CONTEXT.md

- **Methodology choices** (estimator, identification strategy, prior, sample restriction *with a methodological reason*, missing-data rule) → ADR.
- **Implementation details** (which R package, which function, which target node, which file) → code comments.
- **Results or interpretation** ("the effect was 0.12 log-points") → paper draft.

If you find yourself writing any of these into CONTEXT.md, stop. CONTEXT.md is a record of what things *mean*, not what was *done* or what was *found*.

## Lazy creation

If `CONTEXT.md` doesn't exist, create it with the three section headings (Glossary, Variables, Population) and the one-sentence preamble from `setup-research-skills/context-template.md`. Don't pre-populate with examples. Don't add *Flagged ambiguities* until there's something to flag.

---

*Seed template: [`setup-research-skills/context-template.md`](../setup-research-skills/context-template.md). Agent contract (how consumer skills read this file): `docs/agents/research-docs.md`.*
