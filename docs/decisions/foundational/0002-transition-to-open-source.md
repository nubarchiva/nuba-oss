---
status: accepted
date: 2023-06-02
---

# nubarchiva transition to an open-source tool

## Context and Problem Statement

* Current nubarchiva customers have full access to its source code.
* It has been decided that nubarchiva will become a fully open-source solution.
* It's inviable to maintain two different products: commercial and open-source.

## Decision Outcome

* The current version of nubarchiva will only go as far as version 2.
* The open-source version will number starting at 3.
* In version 2, no significant features will be developed in favor of OSS version 3.
* To ensure a smooth transition, gradually migrate the functionalities from nubarchiva version 2 to
  version 3. It is important to note that copying the code between versions should be avoided.
  Instead, the code migrated to version 3 can be utilized in version 2.

### Consequences

* Bad, having "live" both versions can constrain the decision-making process in the open-source
  version 3, such as Java JDK requirements, deployment options, etc.

