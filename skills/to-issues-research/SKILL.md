---
name: to-issues-research
description: Decompose an analysis plan at `docs/plans/NNNN-slug.md` into a DAG of GitHub Issues using the `Depends on: #N` convention from `docs/agents/issue-tracker.md`, sized so each issue is plausibly a single session. Publishes in dependency order, applies state labels and the `autonomous-ok` kind label where appropriate, and populates the matching epic's `Children:` placeholder (or offers once to create an epic). Surfaces the plan's Open questions item-by-item and converts the ones the user opts in to `needs-info` issues. Use when the user wants to turn a locked analysis plan into pickable work, populate an epic with child-issue references, or convert specific open questions into actionable tickets.
---

<what-to-do>

Read the latest `docs/plans/NNNN-slug.md` (or one the user names), decompose it into a DAG of GitHub Issues using `Depends on:` per `docs/agents/issue-tracker.md`, and publish in dependency order. Size issues so each is plausibly completable in one session — split long or multi-category tasks and wire them with `Depends on:`. One light check-in on the proposed decomposition before publishing; then walk the plan's "Open questions" section item-by-item to decide which become `needs-info` issues. After publishing, populate the matching epic's `Children:` placeholder, or offer once to create an epic if none exists.

Do NOT interview the user on methodology or invent decisions that should live in an ADR.

</what-to-do>

<supporting-info>

## Research context awareness

Same single-context layout as the rest of the suite. One paper, one repo:

```
/
├── CONTEXT.md                # glossary / variables / population
├── docs/
│   ├── adr/                  # methodology decisions, numbered; append-only once Accepted
│   ├── plans/                # analysis plans, numbered, versioned snapshots
│   └── agents/               # this skill's contracts (written by setup-research-skills)
│       ├── issue-tracker.md  # gh CLI conventions + Depends-on field
│       └── triage-labels.md  # state + kind label vocabulary
└── R/ (or analysis/, src/, etc.)
```

The plan is the input. The issues + the updated epic are the output. Decomposition follows task-type (plumbing / estimation / sweep / methodology-gap / validation / deliverable), not a vertical schema-API-UI slice.

## Process

### 1. Read context

Before drafting:

- The target plan. By default, the highest-numbered `docs/plans/NNNN-*.md`; if the user named a specific plan, use that one.
- `CONTEXT.md` — vocabulary for issue titles and bodies (glossary, variables, population).
- `docs/adr/*.md` — methodology decisions to cite by number rather than restate.
- `docs/agents/issue-tracker.md` and `docs/agents/triage-labels.md` — the contract this skill writes against. Pull label strings from `triage-labels.md` rather than assuming defaults.
- Open issues to avoid duplicates: `gh issue list --state open --json number,title,labels --jq '[.[] | {number, title, labels: [.labels[].name]}]'`.
- The matching epic, if it exists. Find via `gh issue list --label epic --state all --json number,title,body --jq '[.[] | {number, title, body}]'` and pick the one whose body references this plan file path.

**Verify the labels exist on the remote.** Run `gh label list --json name --jq '[.[].name]'` and confirm `autonomous-ok` and the `epic` label are present (per the canonical strings in `docs/agents/triage-labels.md`). If either is missing, do NOT auto-create — tell the user "The `<name>` label isn't on the remote yet. Re-run `setup-research-skills` to add it." Labels are setup's domain.

If no GitHub remote, stop and say so.

### 2. Classify and size tasks

Walk the plan's deliverables + approach. Bucket each unit of work into one of the six research task categories (see [ISSUE-SHAPES.md](./ISSUE-SHAPES.md)):

- **plumbing** — data load, cleaning, transformation, schema; touches `tar_target(...)`
- **estimation** — a named specification on named inputs producing a named output
- **sweep** — pre-specified variations of a primary spec (robustness, sensitivity)
- **methodology-gap** — work blocked on a methodology decision that hasn't been made
- **validation** — sanity checks, balance checks, MCMC diagnostics, distributional checks
- **deliverable** — produce a specific table, figure, or output file

Categories plumbing / estimation / sweep default to the **`autonomous-ok`** kind label. methodology-gap / validation / deliverable do not.

**Apply the session-sizing rule.** Each issue should plausibly be completable in a single session. Signals an issue is too big: touches many pipeline nodes, combines categories, has many distinct deliverables, or its body would naturally want multiple unrelated sub-headings. When that happens, propose a split using one of the three splitting strategies in [ISSUE-SHAPES.md](./ISSUE-SHAPES.md) (pipeline boundary / spec boundary / variation boundary) and wire the resulting issues with `Depends on:`. Don't atomise to triviality — the rule is an upper bound, not a floor. The classification + sizing is a proposal; the user adjusts in step 4.

### 3. Build the DAG

Per proposed issue, decide:

- **Title** — short, uses `CONTEXT.md` glossary terms. For split sets, put the distinguishing detail in the title (e.g. *Sweep: cluster level — industry-year* rather than burying it in the body).
- **Category** — one of the six above.
- **Proposed state label** — `ready` if the methodology is locked and the spec is written; `needs-info` if there's a methodology gap or the spec is still open.
- **Proposed `autonomous-ok` flag** — default per category, override when the specifics warrant it.
- **`Depends on:`** — the issue numbers (or, before publishing, placeholder names) of blockers. Captures the DAG.

Do NOT auto-lift Open questions here — they're handled in step 4.

### 4. Quiz the user — one light check-in

Present the proposed decomposition as a numbered list. For each entry show: title, category, state, `autonomous-ok` (yes/no), `Depends on:`. Then ask:

- Does any issue still feel too big for one session? Anything atomised past usefulness?
- Are dependencies correct? Anything to merge or split?
- Classification correctness — categories, states, and `autonomous-ok` flags.

Iterate until the user approves the list.

Then, *separately*, walk the plan's "Open questions" section item by item — priority tag + statement — and ask per-item whether to materialise it as a `needs-info` issue or leave it in the plan. No automatic lifting based on priority.

### 5. Publish in dependency order

For each approved issue, in topological order so blockers exist before dependents:

```bash
gh issue create \
  --title "<title>" \
  --body "<body>" \
  --label "<state>" \
  --label "autonomous-ok"   # only when the flag is yes
```

Substitute real issue numbers into `Depends on:` once the blockers have been created.

`Depends on:` is the only required body field (per `docs/agents/issue-tracker.md`); everything else flexes to the work. See [ISSUE-SHAPES.md](./ISSUE-SHAPES.md) for shape sketches by category.

Materialised Open-questions issues land in `needs-info`, never get `autonomous-ok`, and cite the originating plan section + name `grill-with-docs-research` as the resolution path.

### 6. Populate (or offer to create) the epic

If an epic was found in step 1, replace the `Children: (to be populated by to-issues-research)` placeholder with a checkboxed list of the child-issue numbers, grouped by the phases the user approved in step 4. Use `gh issue edit <epic#> --body-file <path>` so multi-line content survives.

If no epic exists, offer once: *"Also create a top-level epic issue on GitHub referencing this plan?"* If yes, create one with the same body shape `to-analysis-plan` would have written — `Plan:` link, 2–3-sentence recap, then a `Children:` list populated directly from the issues just created. Label `epic`. If the user declines, say so and stop. Don't push.

### 7. Mention the next step

After everything is written:

> The tracker is now the source of truth for the work. Pick issues by hand via `gh issue list --label ready` (and the `Depends on:` graph for what's pickable). Move labels with `gh issue edit` as work progresses; close with `gh issue close` when done.

Don't force it.

## Handoff

The issue tracker is now the source of truth for what work is to be done. State labels (`needs-info` / `ready` / `awaiting-review` / `done`) are a vocabulary you apply by hand or with `gh issue edit`; the `Depends on:` graph encodes pickability. Materialised `needs-info` issues are the handoff back to `grill-with-docs-research`; resolving them and re-running `to-analysis-plan` produces a successor plan whose decomposition this skill can re-run against the open issue list (without duplicating already-published work).

</supporting-info>
