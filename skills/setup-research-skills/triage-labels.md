# Triage labels

The research skills move issues through four states. This file maps each canonical state to the actual label string used in this repo's issue tracker.

| Canonical state    | Label in our tracker | Meaning                                                                       |
| ------------------ | -------------------- | ----------------------------------------------------------------------------- |
| `needs-info`       | `needs-info`         | Grilling required — methodology or spec gap                                   |
| `ready`            | `ready`              | Methodology locked, spec written, dependencies closed — pickable             |
| `awaiting-review`  | `awaiting-review`    | Agent or human has finished the work; user needs to inspect the result       |
| `done`             | `done`               | Closed                                                                        |

When a skill mentions a state (e.g. "mark the issue as `awaiting-review`"), use the corresponding label string from the right-hand column.

Edit the right-hand column to match the vocabulary you actually use in GitHub.

## A note on `awaiting-review`

`awaiting-review` is a **hint, not a gate**. The user inspects every result regardless of how the agent labels it. The skills' job is to surface "this looks off" — wrong-signed coefficients, divergent MCMC chains, balance failures — in issue or PR comments. The state machine does not gate on the agent's own judgment.
