# Architecture knowledge base

Welcome 👋 to the "any decision" knowledge base of nubarchiva.

## Definition and purpose

[Markdown Any Decision Records](https://adr.github.io/madr/) (MADR) is a lean template to capture
any decisions in a structured way. The template originated from capturing architectural decisions
and developed into a template allowing to capture any decisions taken.

An [Architectural Decision (AD)](glossary.md) is a software design choice that addresses a
functional or non-functional requirement that is architecturally significant. This might, for
instance, be a technology choice (e.g., Java vs. JavaScript), a selection of the IDE (e.g., IntelliJ
vs. Eclipse IDE), a choice between a library (e.g., SLF4J vs. java.util.logging), or a decision on
features (e.g., infinite undo vs. limited undo). Any decisions that impact the architecture somehow
are architectural.

There are debates about what is an architecturally-significant decision and which decisions are not.
Since we believe that any (significant) decision should be captured in a structured way, we use the
MADR template to capture any decision.

An (M)ADR is immutable: its status can change (i.e., become deprecated or superseded). That way, you
can become familiar with the whole project's history by reading its decision log chronologically.
Moreover, maintaining this documentation aims at the following:

- 🚀 Improving and speeding up the onboarding of a new team member
- 🔭 Avoiding blind acceptance/reversal of a past decision
  ([Michael Nygard's famous article on ADRs](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions.html))
- 🤝 Formalizing the decision process of the team

## Usage

The typical workflow of an ADR is the following:

![ADR workflow](../assets/adr-workflow.png)

The decision process is entirely collaborative and backed by pull requests.

## More information

- [Architectural glossary terms](glossary.md)
- [ADR GitHub organization](https://adr.github.io/)
- [MADR 3.0.0 – The Markdown Any Decision Records](https://adr.github.io/madr/)
- [The Markdown ADR (MADR) Template Explained and Distilled](https://medium.com/olzzio/the-markdown-adr-madr-template-explained-and-distilled-b67603ec95bb)
