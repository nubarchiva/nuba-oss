---
status: accepted
date: 2023-06-02
---

# Use Markdown Any Decision Records 3.0.0

## Context and Problem Statement

We want to record any decisions made in this project independent of whether decisions concern the
architecture ("architectural decision record"), the code, or other fields.

Which format, conventions, and structure should these records follow?

## Decision Outcome

[MADR 3.0.0 – The Markdown Any Decision Records](https://adr.github.io/madr/) is a lean template to
capture decisions in a structured way. The template originated from capturing architectural
decisions and developed into a template allowing to capture any decisions taken. “MADR” stands for
**M**arkdown and **A**rchitectural **D**ecision **R**ecords. Since MADR 3.0.0, “Architectural” was
replaced by “Any”.

### Create a new ADR

Because currently (2023-06-11) there is no tooling supporting MADR 3.0.0, we use the manual approach

1. Copy [`docs/decisions/templates/adr-template.md`](../../templates/adr-template.md) to
   `docs/decisions/NNNN-title-with-dashes.md`, where `NNNN` indicates the next number in sequence.
2. Edit `NNNN-title-with-dashes.md`.

The filenames follow the pattern `NNNN-title-with-dashes.md`, where

- `NNNN` is a consecutive number, and we assume that there won’t be more than 9,999 ADRs in one
  repository.
- The title is stored using dashes and lowercase.
- The suffix is `.md` because it is a [Markdown](https://github.github.com/gfm/) file.

Decisions are placed in the subfolder `decisions/` to keep them close to the documentation but also
separate the decisions from other documentation.

### Markdown Style

ADRs are written using Markdown. Since Markdown allows many styles, the following specifications are
considered authoritative in cases of ambiguity:

- [CommonMark](https://spec.commonmark.org/current/)
- [GitHub Flavored Markdown Spec](https://github.github.com/gfm/)

### ADR first line (heading title)

The first line in an ADR should be consistent with the markdown file name. It should not contain its
ADR number in front of the line.

EXAMPLES

- Bad: `# 2. Do not use numbers in headings`
- Good: `# Do not use numbers in headings`

### Usage of categories

nubarchiva MADR logs are categorized ADRs by defining subdirectories and putting the ADRs into these
folders. One level of subfolders is allowed, not more.

EXAMPLES

- `docs/decisions/foundational/0001-use-markdown-any-decision-records.md`
- `docs/decisions/foundational/0001-flexible-properties-selection.md`

### Guidelines to Achieve Sustainable Decisions

The following list can serve as guidelines and assessments for
[achieving sustainable decisions](https://www.infoq.com/articles/sustainable-architectural-design-decisions/):

1. Use a lean/minimalistic approach for the initial decision documentation.
2. Prioritize and capture all critical decisions that are relevant enough for documenting and
   understanding the target architecture.
3. Detail the significant decisions with full-blown templates only after the initial work has been
   done (when the decision-makers are content with the architectural choices made and confident that
   these decisions don’t have to be revised any time soon).
4. Use the lean/minimalistic versions, from item 1, as a short version of documented decisions with
   the right granularity level to provide an overview of the complex and trivial or apparent
   choices.
5. Wherever possible, use existing architectural knowledge, either from guidance models or from
   other sources. Review and extend such knowledge and fit it into the specific decision context.
6. Ensure that traceability links between decisions, requirements, and architectural designs/code
   are established. Provide automated consistency checking to ensure the traceability links are in
   sync after a change. Limit the number of dependencies between decisions and other software
   artifacts.
7. Apply the guidelines for the justifications consequently and forcefully—they are the most crucial
   part of the decision documentation because they give the rationale.
