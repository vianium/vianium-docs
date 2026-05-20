# ADR-0003: Keep the product name Vianigram

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-05-18 |
| Decider | Angel Careaga |

## Context

The Vianium organization standardizes on the `vianium-*` prefix for
repo and library naming. A consistent prefix would suggest renaming the
Telegram client product from "Vianigram" to "vianium-gram."

However, "Vianigram" already has an internal identity: project files
(`Vianigram.sln`, `Vianigram.Core.*`), namespaces (`Vianigram.*`),
brand assets (logo, brand color `#00A0F0`), and product-level
documentation.

## Decision

The Telegram client product retains the name **Vianigram**. Its repo
under the Vianium organization is `github.com/vianium/vianigram`
(without the `vianium-` prefix).

Internal artifacts are not renamed:

- `Vianigram.sln`
- C# namespaces `Vianigram.*`
- Project files `Vianigram.Core.*`, `Vianigram.App.*`
- The internal managed kernel project `Vianigram.Kernel` (distinct
  from the native `vianium-kernel`)

Other Vianium repos continue to use the `vianium-*` prefix.

## Consequences

### Positive

- Preserves a working build with minimal change. Renaming a `.sln`
  plus every `.csproj` and `.vcxproj` and namespace would be a
  multi-week task with high regression risk.
- Vianigram already has brand identity (logo, color, naming
  recognition). The Vianium prefix would dilute this.
- The naming break is visible (a non-`vianium-*` repo in the org)
  precisely where users care: at the product level. Visitors reading
  the org page recognize it as a finished product, not a library.

### Negative

- Slight naming inconsistency in the org. Mitigated by:
  - Documenting the decision here.
  - Putting the rationale in the org profile and Vianigram README.
- The managed `Vianigram.Kernel` (C#) and the native `vianium-kernel`
  (C++) share a base name. Mitigated by:
  - Different file extensions (`.csproj` vs `.vcxproj`).
  - Different repos (`vianigram` vs `vianium-kernel`).
  - An explicit "Naming" section in the Vianigram README.

## Alternatives considered

- **Rename product to `vianium-gram`.** Rejected for the cost-versus-
  benefit calculation above.
- **Rename only the repo, keep internal artifacts as Vianigram.**
  Rejected: creates a confusing mismatch between the repo URL and the
  code inside.

## References

- [Architecture Overview](../architecture-overview.md)
- ADR-0001: Multi-repo over monorepo
