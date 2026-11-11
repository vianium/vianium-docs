# ADR-0001: Multi-repo over monorepo

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-05-18 |
| Decider | Angel Careaga |

## Context

Vianium consists of an app (`vianium-localsend`), documentation
(`vianium-docs`) and org templates (`.github`). Reusable components, if
any are ever published, should be addressable on their own.

Two organizational strategies are practical:

- **Monorepo**: a single Git repository containing all libraries and
  products. Atomic cross-cutting changes, single source of truth.
- **Multi-repo**: one Git repository per published unit. Independent
  releases, focused issue tracking per library, lower friction for
  external consumers.

## Decision

Use multi-repo. Each library, each app, and each meta artifact (docs,
org templates) is its own GitHub repository under the `vianium` organization.

Local development assumes all relevant repos are cloned as siblings
under a single workspace directory; project references between repos
use relative paths.

## Consequences

### Positive

- Each library has its own README, license footprint, issue tracker,
  and release cadence. External adopters see a focused project, not a
  monorepo full of unrelated code.
- Apache 2.0 `NOTICE` attribution stays inside a coherent boundary. A
  consumer of one repo does not pull in unrelated code.
- App repos can use a more restrictive license
  (PolyForm-Noncommercial-1.0.0) without leaking into library or docs
  repos.
- Cloning a single library is fast, even with future history.

### Negative

- Cross-cutting changes that touch multiple repos require coordinated
  PRs.
- Versioning is per-repo, so consumers must read each library's
  CHANGELOG to assemble a working set.
- More CI configuration to maintain. Mitigated by sharing workflows
  between repos and using the `.github` repo for org-wide defaults.

## Alternatives considered

- **Monorepo with directory-scoped releases.** Rejected: external
  consumers expect "one repo, one library" on GitHub, and the value of
  seeing each component as its own repo with its own stars, watchers,
  and contributors is significant.
- **Hybrid: single monorepo for libraries, separate product repos.**
  Rejected: the libraries are the parts most likely to gain external
  contributors, and they benefit most from being separately addressable.

## References

- [Vianium Licensing Policy](../licensing-policy.md)
- ADR-0002: Dual licensing
