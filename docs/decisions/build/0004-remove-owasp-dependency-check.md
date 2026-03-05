---
status: "accepted"
date: 2026-03-05
---

# Remove OWASP dependency-check from the build

## Context and Problem Statement

The OWASP dependency-check plugin was configured in the build to detect publicly disclosed
vulnerabilities in project dependencies. However, the NVD (National Vulnerability Database) retired
its v1.1 JSON feeds, and the newer NVD API 2.0 used by the plugin (versions >= 9.0.0) suffers from
persistent instability — the API returns malformed responses that cause NullPointerExceptions in the
client library.

## Decision Drivers

* The NVD API 2.0 is unreliable, causing consistent build failures
* Configuring `failOnError=false` masks failures and provides false confidence
* The plugin adds seconds to every build with zero value in return

## Considered Options

* Keep dependency-check with `failOnError=false`
* Remove dependency-check entirely

## Decision Outcome

Chosen option: "Remove dependency-check entirely" because running a security scanner that always
fails and ignoring the failure is worse than not running it at all. It creates noise and false
confidence.

### Consequences

* Good, because the build is clean, fast, and honest
* Bad, because there is no automated CVE detection in the build pipeline

## More Information

When the project starts incorporating runtime dependencies (during legacy migration), this decision
should be revisited. At that point, evaluate the state of the NVD API and consider alternatives
like [OSS Index](https://ossindex.sonatype.org/) or [Trivy](https://github.com/aquasecurity/trivy).
