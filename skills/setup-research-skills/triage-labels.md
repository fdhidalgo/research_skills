# Triage labels

The research skills use two kinds of labels on this repo's issue tracker:

- **State labels** position a work-unit issue in the lightweight four-state pipeline that `triage-research` moves issues through.
- **Kind labels** mark issues that are *categorically* different from regular work units — currently just `epic`, the container created by `to-analysis-plan` to anchor a numbered plan to the issue tracker.

The two kinds don't overlap. An epic isn't in the state pipeline (it's a container, not a work unit); a regular work issue doesn't get a kind label.

## State labels (the triage pipeline)

| Canonical state    | Label in our tracker | Meaning                                                                       |
| ------------------ | -------------------- | ----------------------------------------------------------------------------- |
| `needs-info`       | `needs-info`         | Grilling required — methodology or spec gap                                   |
| `ready`            | `ready`              | Methodology locked, spec written, dependencies closed — pickable             |
| `awaiting-review`  | `awaiting-review`    | Agent or human has finished the work; user needs to inspect the result       |
| `done`             | `done`               | Closed                                                                        |

When a skill mentions a state (e.g. "mark the issue as `awaiting-review`"), use the corresponding label string from the right-hand column.

Edit the right-hand column to match the vocabulary you actually use in GitHub.

## Kind labels

| Canonical kind | Label in our tracker | Meaning                                                                          |
| -------------- | -------------------- | -------------------------------------------------------------------------------- |
| `epic`         | `epic`               | Top-level container for a numbered analysis plan; not a work item, not in the state pipeline |

Kind labels are applied at issue creation and stay with the issue for its lifetime — they don't move. An issue with the `epic` kind label still doesn't get a state label.

## A note on `awaiting-review`

`awaiting-review` is a **hint, not a gate**. The user inspects every result regardless of how the agent labels it. The skills' job is to surface "this looks off" — wrong-signed coefficients, divergent MCMC chains, balance failures — in issue or PR comments. The state machine does not gate on the agent's own judgment.
