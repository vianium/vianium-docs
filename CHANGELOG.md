# Changelog — vianium-docs

All notable changes to this repository are documented here. The format
follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and
this project adheres to [Semantic Versioning](https://semver.org/).

Unreleased changes are listed under `## [Unreleased]`. Each tagged
release moves the content from there into a dated `## [vX.Y.Z] — YYYY-MM-DD`
heading.

---

## [Unreleased]

### Added
- `architecture-overview.md`: registered three new repositories in the
  ecosystem manifest — `vianium-fs` (Tier 1, C++ + WinRT projection
  filesystem engine), `vianium-icons` (Tier 1, managed Fluent UI System
  Icons catalog with bundled font), and `vianium-explorer` (Tier 3,
  product app that consumes both). Updated the workspace tree, the
  Mermaid dependency graph, and the WinRT-projection list.

### Changed
- Ecosystem-wide repo count bumped from **20 → 26** across the
  canonical references: `licensing-policy.md` (tier tables and
  aggregate), `getting-started.md` (workspace tree and clone block),
  `announcement.md` (intro line, tier table, "in rough order"),
  `community.md` (product list and welcome message), and
  `PUBLISH-CHECKLIST.md` (FUNDING.yml reference).
- `adr/0005-component-architecture-standard.md`: scope updated from
  **18 → 24** code repositories, conformance table extended with the
  six new entries (`vianium-audio` Cat 3, `vianium-genai` Cat 2,
  `vianium-fs` Cat 3, `vianium-icons` Cat 1, `vianium-chat` Cat 4,
  `vianium-explorer` Cat 4), and the per-category repo lists rebuilt
  accordingly. The five known debt items remain unchanged.

### Deprecated
- _Track soon-to-be-removed surfaces._

### Removed
- _Track removed features (only after a deprecation period)._

### Fixed
- _Track bug fixes._

### Security
- _Track security-relevant fixes (CVE references where applicable)._

---

## [v0.1.0] — TBD

Initial public release as part of the [Vianium](https://github.com/vianium)
org migration. License: Apache-2.0. Tier: Meta.

See [`NOTICE`](NOTICE) for copyright attribution and
[`vianium-docs/MIGRATION_PLAN.md`](https://github.com/vianium/vianium-docs/blob/main/MIGRATION_PLAN.md)
for the cross-repo migration narrative.

[Unreleased]: https://github.com/vianium/vianium-docs/compare/v0.1.0...HEAD
[v0.1.0]: https://github.com/vianium/vianium-docs/releases/tag/v0.1.0
