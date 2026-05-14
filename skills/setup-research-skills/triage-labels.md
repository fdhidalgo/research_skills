# Triage labels

The research skills use two kinds of labels on this repo's issue tracker:

- **State labels** position a work-unit issue in the lightweight four-state pipeline. There's no skill that wraps the transitions — you apply state labels by hand (or with `gh issue edit`), and the agent applies `awaiting-review` when it finishes autonomous work. New issues get their starting state label from `to-issues-research`.
- **Kind labels** mark issues that are *categorically* different or carry a permanent attribute that the state pipeline doesn't capture. Two are defined: `epic` (the container created by `to-analysis-plan` to anchor a numbered plan to the issue tracker) and `autonomous-ok` (applied by `to-issues-research` to work that falls within the agent's autonomous scope).

Kind labels are orthogonal to state. An `epic` isn't in the state pipeline (it's a container, not a work unit); a regular work issue can carry `autonomous-ok` *and* a state label simultaneously.

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

| Canonical kind   | Label in our tracker | Meaning                                                                                                            |
| ---------------- | -------------------- | ------------------------------------------------------------------------------------------------------------------ |
| `epic`           | `epic`               | Top-level container for a numbered analysis plan; not a work item, not in the state pipeline                       |
| `autonomous-ok`  | `autonomous-ok`      | Work falls within the agent's autonomous scope: plumbing, specified estimation, or pre-specified sweep             |

Kind labels are applied at issue creation and stay with the issue for its lifetime — they don't move.

- An `epic` issue is a container and does not also get a state label.
- An `autonomous-ok` issue is a regular work unit and *also* carries a state label; the two are orthogonal. Methodology-gap, validation, and deliverable issues do not get `autonomous-ok`.

## A note on `awaiting-review`

`awaiting-review` is a **hint, not a gate**. The user inspects every result regardless of how the agent labels it. The skills' job is to surface "this looks off" — wrong-signed coefficients, divergent MCMC chains, balance failures — in issue or PR comments. The state machine does not gate on the agent's own judgment.
