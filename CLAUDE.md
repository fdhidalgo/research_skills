# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A Claude Code **plugin** that ships a suite of agent skills for social-science research workflows (R + `targets` primary, Python + `marimo` later). Adapted from Matt Pocock's skills and re-targeted at research artifacts. Design is complete; the plugin scaffold is in place; skills are being built incrementally per `PLAN.md` §6.

Canonical artifacts:
- **`PLAN.md`** — the design doc *and* the live progress tracker. Captures every decision from the interview (20 numbered questions); §6 tracks which skills have shipped, which is next, and which are not started. Read this first for any implementation task, and update §6 every time a skill lands.
- **`.claude-plugin/plugin.json`** — the plugin manifest. Every shipped skill must be listed here.
- **`skills/`** — where research-flavored skills live, one directory per skill containing `SKILL.md` (+ optional supporting files).
- **`references/`** — shared reference docs callable from multiple skills (R packages, conventions, etc.).
- **`reference/mattpocock_skills/`** — vendored copy of `mattpocock/skills`, **gitignored**. Read-only source material; never edit. The reference for what we're adapting and the starting point for ports.
- **`.agents/skills/`** — working subset of Matt's skills installed locally via `skills.sh` so this workspace itself can be operated on with them. Gitignored. Lock file is `skills-lock.json` (also gitignored).

## Repo layout

```
research-skills/                   # published as a Claude Code plugin
  .claude-plugin/plugin.json       # plugin manifest — every skill must be listed here
  skills/                          # research-flavored skills land here
  references/                      # shared reference docs (R packages, etc.)
  PLAN.md                          # design doc — read first
  README.md                        # public-facing skill index + install
  .gitignore                       # excludes vendored Matt copy + .agents/skills/
  CLAUDE.md                        # this file
  reference/mattpocock_skills/     # gitignored — full vendored upstream (read-only)
  .agents/skills/                  # gitignored — installed Matt skills for working in this repo
  .claude/settings.local.json      # local Claude Code permissions (gitignored by default)
```

## Build sequence and progress tracker

`PLAN.md` §6 is the live tracker for what's shipped, what's next, and what's not started. **Read it at the start of every session** to know where to pick up, and **update it whenever a skill lands** — change the bullet to ✅, add the date, note paths and anything deferred. Don't track progress anywhere else (no separate TODO file, no checklist in this file).

End-to-end thin slice first, then deepen. Four skills to ship, in order:

1. `setup-research-skills`
2. `grill-with-docs-research`
3. `to-analysis-plan`
4. `to-issues-research`

Then run on one real research project and find what's broken before deepening (`validate-data`, `improve-research-pipeline`, `diagnose-research`, the rest).

## Architectural commitments

These came out of the design interview and should be respected by anything we build:

- **One repo per paper/project** is the unit of work. All artifacts are repo-scoped.
- **`CONTEXT.md` vs `docs/adr/`** split: static facts (glossary, variable map, population) vs. numbered methodological decisions. ADRs are append-only **once `Accepted`**; `Proposed` ADRs are edited freely, and revisiting an Accepted one is normal research — done by superseding (or `Reversed by ADR-N`), not editing. The researcher flips the `Proposed → Accepted` lock; the agent may offer but never flips it.
- **GitHub Issues** with a `Depends on: #N` field in the body capture the task DAG. A top-level epic issue (or `PLAN.md`) ties the decomposition together. No use of GitHub's sub-issues feature.
- **Lightweight 4-state triage**: `needs-info` / `ready` / `awaiting-review` / `done`.
- **Agent autonomous scope**: pipeline plumbing, specified estimations, pre-specified robustness/sensitivity sweeps. *Not* methodological exploration.

## Style preferences (from the design interview)

When proposing or building skills for this user:

- **Generic Socratic grilling.** Don't bake in method-specific question banks (no `references/causal-did.md` style add-ons). Trust the model; the user wants open-ended interviewing across diverse tasks.
- **Flexible templates, not rigid ones.** Plans, issues, and other artifacts should adapt to project type (descriptive, causal, Bayesian, methods) and stage (exploratory vs. confirmatory). Optional sections, not mandatory schemas.
- **Lean infrastructure.** The user already declined `setup-pre-commit`, `git-guardrails`, `caveman`, `review`, and the writing skills. Don't propose new infrastructure/safety layers unless they pay for themselves.
- **Human in the loop, agent flags anomalies.** The user inspects every result. Skills that execute analyses should always surface "this looks off" notes in issue/PR comments — wrong sign, divergent MCMC, balance failures — but never gate on their own judgment.
- **R-first**. References (`pointblank`/`assertr`, `testthat`, `renv`, `targets`, `cmdstanr`/`brms`). Python (`pandera`, `pytest`, `uv`, `marimo`) is v2.

## Working in this repo

No build/test/lint commands — skills are plain markdown (`SKILL.md`) following Matt's conventions; no toolchain required to author them.

When implementing a skill:
1. Mirror Matt's structure for that skill (in `reference/mattpocock_skills/skills/<category>/<name>/`) — `SKILL.md`, optional supporting files (`REFERENCE.md`, `EXAMPLES.md`, language-specific reference docs), and `scripts/` only when a deterministic operation needs them.
2. Place the new skill under `skills/<skill-name>/` (flat for now; bucket into category folders only if/when the suite outgrows it).
3. Add the skill's path to `.claude-plugin/plugin.json` and link it from `README.md`.
