# CONTEXT

Static facts about this paper / project. Freely edit as the project evolves. Methodological decisions go in `docs/adr/`, not here — this file is for definitions and scope, not choices.

## Glossary

Paper-terminology definitions. The vocabulary the paper uses for its concepts, in the paper's own words.

- *(example — replace)* `treatment` — exposure ≥1 day to program X
- *(example — replace)* `outcome` — log(earnings) one year after exposure

## Variables

Code / data column ↔ paper term map. The strings that appear in datasets and code, and which paper concept each one corresponds to.

- *(example — replace)* `tr` — binary treatment indicator (= `treatment`)
- *(example — replace)* `earn1` — earnings one year post (= `outcome`)

## Population

Who is in and out of the analysis sample, and why.

- *(example — replace)* Workers aged 25–55 in sample S.
- *(example — replace)* Excludes self-employed (no earnings record); excludes pre-2008 (data-quality break — see ADR-0003).
