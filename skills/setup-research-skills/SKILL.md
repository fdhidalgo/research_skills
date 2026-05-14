---
name: setup-research-skills
description: One-time per-repo setup for the research-skills suite. Writes an `## Agent skills` block to `CLAUDE.md`, seeds `docs/agents/*.md`, `CONTEXT.md`, and `docs/adr/0000-template.md`, and creates the GitHub labels (four triage-state labels + the `epic` kind label). Run before first use of `grill-with-docs-research`, `to-analysis-plan`, `to-issues-research`, or `triage-research` — or if those skills appear to be missing context about this repo's issue tracker, triage labels, or research docs.
disable-model-invocation: true
---

# Setup Research Skills

Scaffold the per-repo configuration that the research skills assume:

- **Issue tracker** — GitHub Issues, accessed via the `gh` CLI, with a `Depends on: #N` field in issue bodies for the task DAG.
- **Triage labels** — two flavours: **state labels** (`needs-info`, `ready`, `awaiting-review`, `done`) that move regular issues through the lightweight pipeline, and **kind labels** (currently just `epic`) that mark categorically-different issues like the plan containers created by `to-analysis-plan`.
- **Research docs** — where this repo's `CONTEXT.md` (glossary / variables / population) and `docs/adr/` (methodology decisions) live, and how the other skills should read them.

This is a prompt-driven skill, not a deterministic script. Explore, present what you found, confirm with the user, then write.

## Process

### 1. Explore

Look at the current repo to understand its starting state. Read whatever exists; don't assume:

- `git remote -v` and `.git/config` — is this a GitHub repo? Which one? No remote at all?
- `CLAUDE.md` at the repo root — does it exist? Is there already an `## Agent skills` section?
- `CONTEXT.md` at the repo root — already present? (Don't open it; we won't overwrite it.)
- `docs/adr/` — already populated? Is there already a `0000-template.md`?
- `docs/agents/` — does this skill's prior output already exist?

### 2. Present findings and ask

Summarise what's present and what's missing in two or three short lines. Then walk through the decisions **one at a time** — present, get the user's answer, move on. Don't dump them all at once.

Assume the user does not know what these terms mean. Each decision starts with a short explainer (what it is, why these skills need it). Then show the default and ask if they want to change it.

**Decision A — Triage label vocabulary.**

> Explainer: The research skills use two flavours of labels. **State labels** move regular work issues through four states: `needs-info` (grilling required), `ready` (methodology locked, spec written, deps closed), `awaiting-review` (work finished — you need to inspect), and `done`. **Kind labels** mark categorically-different issues — currently just `epic`, the container created by `to-analysis-plan` to anchor a numbered analysis plan to the issue tracker. State and kind labels don't overlap on the same issue. To apply any of these the skills need GitHub labels matching strings *you've actually configured*. Pick the strings now; they get written to `docs/agents/triage-labels.md` and (optionally) created on GitHub in step 5.

State labels (defaults):

- `needs-info` — grilling required (methodology or spec gap)
- `ready` — methodology locked, spec written, dependencies closed
- `awaiting-review` — agent or human has finished; user needs to inspect
- `done` — closed

Kind labels (defaults):

- `epic` — top-level container for an analysis plan; not a work item, not in the state pipeline

Default: each role's string equals its name. Ask the user if they want to override any. If their repo already uses different label names, map them here so the skills apply the right ones instead of creating duplicates.

Skip any further "decisions": tracker is fixed (GitHub Issues), repo layout is fixed (single-context — one paper, one `CONTEXT.md`, one `docs/adr/`).

### 3. Draft and confirm

Show the user drafts of what you're about to write before writing:

- The `## Agent skills` block to add to `CLAUDE.md`.
- The contents of `docs/agents/issue-tracker.md`, `docs/agents/triage-labels.md`, `docs/agents/research-docs.md`.
- The seed content for `CONTEXT.md` and `docs/adr/0000-template.md` (only if those files don't already exist).

Let them edit before writing. Keep this confirmation light — one paste of all the drafts, one yes/no.

### 4. Write the files

**Pick the file to edit:**

- If `CLAUDE.md` exists, edit it.
- If it doesn't exist, create it. (Unlike Matt's `setup-matt-pocock-skills`, there's no `AGENTS.md` fallback — research skills standardise on `CLAUDE.md`.)

If an `## Agent skills` block already exists in `CLAUDE.md`, **update its contents in place** rather than appending a duplicate. Don't overwrite user edits to the surrounding sections.

The block to upsert:

```markdown
## Agent skills

### Issue tracker

GitHub Issues, accessed via the `gh` CLI. Issue bodies use a `Depends on: #N` field for cross-issue dependencies. See `docs/agents/issue-tracker.md`.

### Triage labels

State labels (four-state pipeline): `needs-info` / `ready` / `awaiting-review` / `done`. Kind labels: `epic` (plan containers, not in the state pipeline). See `docs/agents/triage-labels.md`.

### Research docs

Single-context: `CONTEXT.md` (glossary / variables / population) and `docs/adr/` (methodology decisions, immutable, supersede-don't-edit) at the repo root. See `docs/agents/research-docs.md`.
```

If the user overrode any triage label names in step 2, reflect those strings in the `Triage labels` line.

Then write the three docs files using the seed templates in this skill folder as a starting point:

- [issue-tracker-github.md](./issue-tracker-github.md) → `docs/agents/issue-tracker.md`
- [triage-labels.md](./triage-labels.md) → `docs/agents/triage-labels.md` (apply user overrides to the right-hand column)
- [research-docs.md](./research-docs.md) → `docs/agents/research-docs.md`

And seed two repo-root files **only if they don't already exist** (never overwrite):

- [context-template.md](./context-template.md) → `CONTEXT.md`
- [adr-template.md](./adr-template.md) → `docs/adr/0000-template.md`

If either already exists, leave it alone and tell the user.

### 5. Create GitHub labels

If the repo has a GitHub remote (check `git remote -v`):

1. Run `gh label list --json name --jq '[.[].name]'` and diff against the five confirmed label strings (four state + one kind).
2. Show the user the missing labels in a short list.
3. Ask once: "Create these labels on the remote now?"
4. If yes, run `gh label create <name> --color <hex> --description "<one-liner>"` for each missing label. Suggested colors and descriptions:

   State labels:
   - `needs-info` — color `fbca04` (yellow), description "Grilling required: methodology or spec gap"
   - `ready` — color `0e8a16` (green), description "Methodology locked, spec written, dependencies closed"
   - `awaiting-review` — color `1d76db` (blue), description "Work finished; user needs to inspect"
   - `done` — color `5319e7` (purple), description "Closed"

   Kind labels:
   - `epic` — color `c5def5` (light blue), description "Top-level container for an analysis plan; not a triage state"

5. If any label exists already with the same name, skip it silently — don't error out and don't change its color/description without asking.

If the user overrode label names in step 2, use the overridden strings here too.

If there's no GitHub remote, skip this step and tell the user they can re-run setup once a remote is added, or create the labels manually later.

### 6. Done

Tell the user setup is complete. Mention briefly:

- Which skills will read which files (`grill-with-docs-research` produces `CONTEXT.md` and ADRs; `to-issues-research` reads them; `triage-research` reads `docs/agents/triage-labels.md`).
- That they can edit `docs/agents/*.md` directly later — re-running this skill is only necessary to change the triage label vocabulary or restart from scratch.
- That `CONTEXT.md` and ADRs are intentionally seeded as templates — they get filled in lazily during grilling, not up front.
