# research-skills

A Claude Code plugin: a suite of agent skills for social-science research workflows (R + `targets` primary, Python + `marimo` later).

Adapted from [Matt Pocock's skills](https://github.com/mattpocock/skills) and re-targeted at research artifacts (analysis plans, ADRs, data validation, probes) instead of software-product artifacts (PRDs, features, tests).

## Status

Design complete, implementation in progress. See [`PLAN.md`](PLAN.md) for the full design and [build sequence](PLAN.md#6-build-sequence-q19).

## Skills

Shipped so far:

- [`setup-research-skills`](skills/setup-research-skills/SKILL.md) — one-time per-repo setup. Writes the `## Agent skills` block to `CLAUDE.md`, seeds `docs/agents/*.md`, `CONTEXT.md`, and `docs/adr/0000-template.md`, and creates the GitHub labels (four state + `epic` kind).
- [`grill-with-docs-research`](skills/grill-with-docs-research/SKILL.md) — Socratic grilling session that stress-tests a research plan against the paper's recorded language (`CONTEXT.md` glossary / variables / population) and methodology ADRs. Writes inline updates as terms and decisions crystallise.
- [`to-analysis-plan`](skills/to-analysis-plan/SKILL.md) — synthesises the current conversation + `CONTEXT.md` + ADRs into a numbered analysis-plan file at `docs/plans/NNNN-slug.md`. Versioned snapshots (successors cite predecessors); does NOT interview; surfaces gaps as a first-class "Open questions" section. Offers once to also create a top-level GitHub epic issue.

Coming next (in build order — see [`PLAN.md`](PLAN.md) §6): `to-issues-research`, `triage-research`. The full 15-skill catalog is in `PLAN.md` §4.

## Install

_Installation instructions will land once the first skill ships._

## License

TBD.
