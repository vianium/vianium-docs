# ADR-0002: Dual licensing — Apache 2.0 for libraries, PolyForm-NC for products

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-05-18 |
| Decider | Angel Careaga |

## Context

Vianium publishes both reusable infrastructure (libraries) and
standalone end-user products. The author's goals:

1. Maximize reach and attribution of the libraries; reputation as a
   systems software author is a primary outcome.
2. Allow users to use the products freely for personal, educational,
   and research purposes.
3. Prevent commercial repackaging or resale of the products without
   permission.

A single license cannot achieve all three:

- Apache 2.0 maximizes reach but allows commercial resale of products.
- GPL family discourages library adoption in proprietary software.
- A pure noncommercial license blocks library adoption by the
  commercial software ecosystem.
- A "source available" custom license is hard to evaluate for adopters
  and easy to misrepresent.

## Decision

Apply two licenses by repository tier:

| Tier | License | Rationale |
|---|---|---|
| Tier 1 — Foundation | Apache 2.0 | Maximize reach and attribution. |
| Tier 2 — Domain protocols | Apache 2.0 | Same as Tier 1. |
| Tier 3 — Products | PolyForm Noncommercial 1.0.0 | Free for noncommercial use, blocks commercial resale. |
| Meta | Apache 2.0 | Permissive for docs and templates. |

## Consequences

### Positive

- Apache 2.0 on libraries makes them adoptable by commercial users
  without legal review pain, which is the path of maximum attribution
  propagation. The `NOTICE` clause guarantees attribution travels with
  the code.
- PolyForm-NC on products preserves the option to license commercially
  in the future without breaking promises to noncommercial users.
- The PolyForm license is professionally drafted (by Heather Meeker),
  so it does not require defending a homemade license against
  scrutiny.

### Negative

- PolyForm Noncommercial is not OSI open source. Public-facing copy on
  Tier 3 products must say "source-available" or "free for
  non-commercial use," not "open source." This is a small but
  persistent communication cost.
- Cross-tier code reuse requires care: a Tier 3 product can consume
  Tier 1 libraries, but a Tier 1 library cannot pull in Tier 3 code
  (that would contaminate a permissive library with a noncommercial
  restriction).

### Neutral

- Commercial licensing for Tier 3 products is available on request.

## Alternatives considered

- **Apache 2.0 across all tiers.** Rejected: removes the author's
  ability to prevent commercial resale of the finished products.
- **PolyForm Noncommercial across all tiers.** Rejected: severely
  limits library adoption by commercial users, which is exactly the
  population whose attribution we want to capture.
- **GPL or AGPL for products.** Rejected: copyleft is poorly
  understood by many product users, and GPL/AGPL still permits resale.
  It does not solve the commercial-resale concern.
- **Custom "Vianium License."** Rejected: hand-written licenses are
  rarely enforceable and almost never trusted by adopters.

## References

- [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0)
- [PolyForm Noncommercial 1.0.0](https://polyformproject.org/licenses/noncommercial/1.0.0/)
- [Vianium Licensing Policy](../licensing-policy.md)
- ADR-0001: Multi-repo over monorepo
