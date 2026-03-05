---
status: "accepted"
date: 2026-03-05
---

# Require Java 11 as minimum JDK version

## Context and Problem Statement

nubarchiva v3 (OSS) was initially configured to support Java 8 as the minimum version, inheriting
this constraint from nubarchiva v2 (legacy). This forced maintaining dual configurations (JDK 8 and
JDK 11+ profiles) for tooling like Spotless and prevented using modern versions of plugins like
OWASP dependency-check (>= 9.0.0 requires Java 11+).

## Decision Drivers

* Java 8 reached End of Life in March 2022
* nubarchiva v2 (legacy) migration to Java 11 is imminent
* Key build plugins no longer support Java 8
* Dual JDK profiles add unnecessary complexity to the build

## Considered Options

* Keep Java 8 as a minimum
* Require Java 11 as a minimum

## Decision Outcome

Chosen option: "Require Java 11 as a minimum" because Java 8 is EOL, the legacy project is
migrating to 11 anyway, and it eliminates build complexity.

### Consequences

* Good, because it simplifies the build (single Spotless/Palantir config instead of two profiles)
* Good, because it enables `maven.compiler.release` for stricter compilation
* Good, because it unblocks the use of modern plugin versions
* Good, because CI now tests against 11, 17, and 21
