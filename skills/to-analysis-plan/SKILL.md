---
name: to-analysis-plan
description: "Synthesise the current conversation, `CONTEXT.md`, and ADRs into a numbered analysis-plan document at `docs/plans/NNNN-slug.md`. Versioned-snapshot model — every invocation writes a new file; successors reference the prior plan and what changed. Does NOT interview the user; if context is thin, surfaces gaps as a first-class \"Open questions\" section and points back to `grill-with-docs-research`. Use when the user wants to lock a plan after grilling, draft a successor plan after a referee report / data refresh / identification pivot, or get a snapshot of the current state of thinking."
---

<what-to-do>

Synthesise what's already in conversation context + `CONTEXT.md` + ADRs into a single analysis-plan markdown file at `docs/plans/NNNN-slug.md`. Do NOT interview the user. If something is missing, capture it as an Open question, don't ask.

One light check-in before writing the body: show the proposed outline (headings + one-sentence bullets) and the proposed slug; wait for thumbs-up or edits, then write.

After saving the file, offer once to also create a top-level GitHub epic issue referencing the plan. Don't push.

</what-to-do>

<supporting-info>

## Research context awareness

Same single-context layout as the rest of the suite. One paper, one repo:

```
/
├── CONTEXT.md                # glossary / variables / population
├── docs/
│   ├── adr/                  # methodology decisions, numbered; append-only once Accepted
│   │   ├── 0000-template.md
│   │   ├── 0001-identification-strategy.md
│   │   └── 0002-priors.md
│   └── plans/                # analysis plans, numbered, versioned snapshots
│       ├── 0001-initial-analysis-plan.md
│       └── 0002-post-referee-revision.md
└── R/ (or analysis/, src/, etc.)
```

The plan document is a **snapshot of the current state of thinking**, not a living spec. Each invocation writes a new numbered file. Old plans stay on disk as the audit trail; the latest plan is the working draft until it gets a successor.

## Process

### 1. Read context

Before drafting:

- `CONTEXT.md` — paper terminology (glossary), code/data ↔ paper-term map (variables), sample inclusion / exclusion (population). Drives vocabulary in the plan.
- `docs/adr/*.md` — methodology decisions touching the work. Plans cite ADRs by number rather than restating them.
- Conversation history — what's been grilled, what's been decided, what's still open.
- `_targets.R`, files in `R/` — to understand the pipeline as it exists. Read code, names, comments first; column names and cleaning rules are usually checkable there. But if examining the data would make the plan *materially better* — settling an open question, checking a feasibility assumption before it's written into a step — then open it. Light inspection (distributions, missingness, value ranges), not the planned estimation.

If `CONTEXT.md` or `docs/adr/` is missing, proceed silently. Don't gate. The absence surfaces as an Open question in the draft.

### 2. Detect prior plans

Glob `docs/plans/NNNN-*.md`. If any exist:

- The highest-numbered file is the predecessor.
- The new plan's number is predecessor + 1.
- The new plan **must** begin with a successor preamble (format in [PLAN-FORMAT.md](./PLAN-FORMAT.md)) — a `Supersedes:` line and a one-paragraph "What changed" naming the trigger and substantive deltas.

If `docs/plans/` doesn't exist, the new plan is `0001` and the directory gets created lazily on write. No preamble.

### 3. Infer project type and stage

From conversation signals + ADRs + code, infer:

- **Project type** — causal / descriptive / methods / measurement / replication / exploratory-data-dive. Drives **shape** (per [PLAN-FORMAT.md](./PLAN-FORMAT.md)).
- **Project stage** — exploratory vs. confirmatory / pre-registration-bound / late-stage. Drives **depth**.

Default depth is **sketch-level** (~1–3 pages, narrative, primary spec or central artifact named precisely, robustness left as categories rather than enumerated variations). Switch to **spec-level** (~5–10 pages, enumerated robustness, exact controls / clustering / weighting, named transformations) only when conversation signals call for it — preparing a registered report, replication package, or co-author handoff.

When in doubt, sketch.

### 4. Synthesise, do not interview

Draft from what's already in context. The skill before this one (`grill-with-docs-research`) is for interviewing. If the context is thin, the plan will be thin, and every unresolved decision lands in a top-of-document **Open questions** section that names the gap and points back to `grill-with-docs-research`. Don't fabricate. Don't ask. Don't refuse.

Concretely:

- Use paper-terminology from `CONTEXT.md` consistently. If the plan would use a term that conflicts with the glossary, flag it in Open questions rather than silently coining a synonym.
- Cite ADRs by number (`per ADR-0003`) instead of restating their decisions. If a plan-relevant decision has no ADR, *don't* synthesise one inline — note in Open questions that the decision should be promoted to an ADR via grilling.
- Methodology choices, implementation details, variable definitions, and results don't belong in the plan body — they belong in ADRs / code / `CONTEXT.md` / paper drafts respectively. See [PLAN-FORMAT.md](./PLAN-FORMAT.md) for the "what does NOT belong" list.

### 5. Propose outline and slug — one light check-in

Before writing the body, present:

- A proposed slug (kebab-case, topical — describes what's distinctive about *this* plan). For the first plan, `initial-analysis-plan` is the obvious default. For successors, name the distinctive contribution: `post-referee-revision`, `add-mechanism-tests`, `tighten-sample-after-data-refresh`.
- A proposed outline — the headings you'll use plus a one-sentence bullet per heading saying what content will live there.

Wait for thumbs-up or edits, then write. Keep this check-in light — one paste, one yes/no.

This is the only user-facing check-in. The plan body is written without further interruption.

### 6. Write the plan body

Following [PLAN-FORMAT.md](./PLAN-FORMAT.md):

- **No mandatory sections.** Headings flex per project type. The plan must serve four *roles*: communicate the goal, communicate the approach, name concrete deliverables, surface open work. The headings that best carry those roles depend on the project — a measurement paper looks structurally different from a causal-DiD paper looks structurally different from a Bayesian-methods paper.
- **Successor preamble** when applicable (steps 2). Fixed two-line shape: `Supersedes:` line + "What changed" paragraph.
- **Open questions** is the one near-universal section — keep it even when there are no open questions (then it explicitly says "None at time of writing"). This is the handoff back to grilling and the audit-trail entry for "what was still mobile when this snapshot was taken."

See [PLAN-FORMAT.md](./PLAN-FORMAT.md) for the four accomplishment-principles, three structurally-different shape sketches, citation and vocabulary conventions, "Open questions" entry format, and the successor-preamble format.

### 7. Save the file

Write to `docs/plans/NNNN-slug.md`. Create `docs/plans/` lazily if it doesn't exist.

After writing, summarise in chat: file path, the project type / stage you inferred, the count of open questions surfaced, and a one-line recap of the substantive goal.

### 8. Offer GitHub epic registration — once

After saving, ask once:

> Also create a top-level epic issue on GitHub referencing this plan?

If yes:

```bash
gh issue create \
  --title "Epic: <plan title>" \
  --body "<body>" \
  --label "epic"
```

Where `<body>` contains:

1. A link to the plan file: `Plan: [docs/plans/NNNN-slug.md](docs/plans/NNNN-slug.md)`.
2. A 2–3 sentence recap of the substantive goal.
3. If a predecessor epic exists, a line: `Supersedes epic #M.` (Find the predecessor via `gh issue list --label epic --state all --json number,title --jq '...'` and pick the one whose body links to the predecessor plan file.)
4. A literal placeholder line: `Children: (to be populated by to-issues-research)`.

**`epic` label availability.** The `epic` label should have been created by `setup-research-skills`. If `gh label list` shows it's missing, don't auto-create it from here — labels are setup-skill's domain. Tell the user: "The `epic` label isn't on the remote yet. Re-run `setup-research-skills` to add it, or create it manually with `gh label create epic --color c5def5 --description 'Top-level container for an analysis plan'`."

If no GitHub remote, skip this step and say so.

If the user declines the epic, no follow-up — the plan file alone is a complete artifact.

### 9. Mention the next step

After everything is written, mention the natural next step in the chain:

> Next: `to-issues-research` to decompose this plan into the issue DAG. Run when you're ready.

Don't force it.

## Handoff

The plan file is the artifact `to-issues-research` will read to derive the task DAG. It reads the plan semantically — there's no fixed section structure to satisfy. What matters is that the plan clearly communicates the goal, the approach, and the concrete deliverables (whatever shape they take for this project).

The Open questions section is the handoff back to `grill-with-docs-research`. Each entry names a gap; running grilling on those gaps and then re-invoking `to-analysis-plan` produces a successor plan that closes them.

</supporting-info>
