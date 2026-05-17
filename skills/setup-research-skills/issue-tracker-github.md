# Issue tracker: GitHub

Issues for this repo live as GitHub Issues. Use the `gh` CLI for all operations.

## CLI conventions

- **Create an issue**: `gh issue create --title "..." --body "..."`. Use a heredoc for multi-line bodies.
- **Read an issue**: `gh issue view <number> --comments`.
- **List issues**: `gh issue list --state open --json number,title,body,labels,comments --jq '[.[] | {number, title, body, labels: [.labels[].name], comments: [.comments[].body]}]'`, with `--label` / `--state` filters as needed.
- **Comment on an issue**: `gh issue comment <number> --body "..."`.
- **Apply / remove labels**: `gh issue edit <number> --add-label "..."` / `--remove-label "..."`.
- **Close**: `gh issue close <number> --comment "..."`.

`gh` infers the repo from `git remote -v` when run inside a clone.

## Issue body shape

**No mandatory template.** Issue bodies are flexible — match the shape to the work (a methodology question, a cleaning step, an estimation, a robustness test). One required convention:

### `Depends on:` field

Every issue body that has prerequisites lists them in a `Depends on:` field referencing the blocking issue numbers. This captures the task DAG without using GitHub's sub-issues feature.

```markdown
## Issue #12  Add cleaning step for treatment indicator
Depends on: #3 (raw data load), #5 (codebook ADR)

[work description, acceptance criteria]
```

If an issue has no prerequisites, omit the field rather than writing `Depends on: none`.

## Epic / decomposition issue

A top-level "epic" issue (or `PLAN.md` in the repo) ties the whole decomposition together with links to all child issues. It captures the goal-to-tasks DAG. A Mermaid diagram is optional, not required.

## When a skill says "publish to the issue tracker"

Create a GitHub issue.

## When a skill says "fetch the relevant ticket"

Run `gh issue view <number> --comments`.
