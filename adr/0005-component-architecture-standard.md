# ADR-0005: Component architecture standard — pattern per component category

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-05-19 |
| Decider | Angel Careaga |
| Supersedes | (none) |
| Affects | All 18 code repositories in the Vianium org |

## Context

The Vianium org holds 18 code repositories across three tiers (7
foundation, 7 domain, 4 products). An audit of their *internal*
organization found the architectural pattern varies repo to repo:

- Some are **flat** — files grouped by technical family, no layering
  (`vianium-crypto`, `vianium-kernel`, `vianium-grpc`).
- Some are **DDD-layered** — a `domain/` folder but no `application/`
  or `ports/` (`vianium-tls`, `vianium-http`, `vianium-mtproxy`).
- Some are **full Hexagonal + DDD** — `domain/` + `application/` +
  `infrastructure/` + `ports/{inbound,outbound}/` (`vianium-net`,
  `vianium-mtproto`, `vianium-voip`, and the three larger products).
- Some are **core + platform adapter** — a pure `core/` algorithm
  wrapped by a thin `winrt/` adapter (`vianium-media`,
  `vianium-image-palette`, `vianium-innertube`).

Without a written standard, two problems compound over time:

1. New code is written to whatever pattern the author last saw, so
   the drift widens.
2. Every reviewer/maintainer must re-learn each repo's layout before
   they can navigate it — there is no "you already know where things
   are" guarantee across the org.

The naive fix — mandate one pattern (full Hexagonal + DDD) everywhere
— is **wrong**. A library of AES/SHA primitives has no business
domain; a `domain/` + `ports/` skeleton over it is pure ceremony that
makes the code *harder* to read, not easier. The right fix recognizes
that the 18 repos are not all the same *kind* of component.

## Decision

The architecture standard is defined **per component category**, not
as a single pattern for all repos. Every repo is classified into
exactly one of four categories, and each category has one mandated
internal pattern.

### Category 1 — Technical primitive

Libraries with **no business domain**: algorithms, runtime services,
pure infrastructure. There is nothing to "model" — only correct,
fast implementations of well-defined operations.

- **Pattern: Flat.** Source grouped by technical family. No
  `domain/` / `application/` / `ports/` layers — they would be noise.
- **Repos:** `vianium-kernel`, `vianium-crypto`,
  `vianium-managed-kernel`, `vianium-grpc`.

### Category 2 — Protocol / domain library

Libraries that **model concepts** — a session, a handshake, a
policy, a framed message, a transport. They have invariants, value
objects, and use cases worth isolating from I/O.

- **Pattern: Hexagonal + DDD.** Required folders under the source
  root:
  - `domain/` — entities, `value_objects/`, `errors/`
  - `application/` — use cases / handlers / orchestration
  - `infrastructure/` — adapters (sockets, OS APIs, codecs)
  - `ports/inbound/` + `ports/outbound/` — interface boundaries
  - `api/v1/` — the public, versioned surface
- **Repos:** `vianium-tls`, `vianium-http`, `vianium-net`,
  `vianium-mtproto`, `vianium-mtproxy`, `vianium-voip`,
  `vianium-store`.

### Category 3 — Algorithm + platform adapter

Native libraries that are **one computational core** plus a thin
projection so managed code can call it. They have a clear inside
(the algorithm) and outside (the platform), but not enough domain
surface to warrant the full Category 2 layering.

- **Pattern: `core/` + adapter.** A platform-free `core/` (the
  algorithm) plus one adapter folder per platform surface
  (`winrt/`, `mft/`). This *is* a Hexagonal split — `core/` is the
  hexagon, the adapters are the driving/driven sides — expressed
  without ceremony.
- **Repos:** `vianium-media`, `vianium-image-palette`,
  `vianium-innertube`.

### Category 4 — Product application

User-facing apps with **many business contexts** (account, chats,
playback, bookmarks, …).

- **Pattern: Hexagonal + strategic DDD.** One project per bounded
  context under `Core/`. Each context project is internally
  Category 2: `Domain/`, `Application/`, `Ports/{Inbound,Outbound}/`,
  `Infrastructure/`, `Composition/`. The shell app and tests live
  under `Clients/`.
- **Repos:** `vianium-browser`, `vianigram`, `vianium-music`,
  `vianium-localsend`.

### Conformance table

| Repo | Category | Conforms today? |
|---|---|---|
| vianium-kernel | 1 Technical primitive | ✅ |
| vianium-crypto | 1 Technical primitive | ✅ |
| vianium-managed-kernel | 1 Technical primitive | ✅ |
| vianium-grpc | 1 Technical primitive | ✅ |
| vianium-net | 2 Protocol/domain | ✅ |
| vianium-mtproto | 2 Protocol/domain | ✅ |
| vianium-voip | 2 Protocol/domain | ✅ |
| vianium-tls | 2 Protocol/domain | ⚠️ debt — has `domain/`, missing `application/` + `ports/` |
| vianium-http | 2 Protocol/domain | ⚠️ debt — has `domain/`, missing `application/` + `ports/` |
| vianium-mtproxy | 2 Protocol/domain | ⚠️ debt — has `domain/` + `infrastructure/`, missing `ports/` + `application/` |
| vianium-store | 2 Protocol/domain | ⚠️ debt — flat-by-technical (`Auth/`, `Cache/`, `Grpc/`, …) |
| vianium-media | 3 Algorithm + adapter | ✅ |
| vianium-image-palette | 3 Algorithm + adapter | ✅ |
| vianium-innertube | 3 Algorithm + adapter | ✅ |
| vianium-browser | 4 Product app | ✅ |
| vianigram | 4 Product app | ✅ |
| vianium-music | 4 Product app | ✅ |
| vianium-localsend | 4 Product app | ⚠️ debt — single flat project, no bounded contexts |

13 of 18 repos conform today. The 5 marked ⚠️ are **known,
documented technical debt** — see "Technical debt" below.

## Consequences

### Positive

- A new contributor learns four layouts, not eighteen. Opening any
  repo, they know which of the four patterns to expect from its tier
  and kind.
- New code has an unambiguous home. Code review can cite this ADR
  instead of relitigating structure each PR.
- The standard records *why* a flat repo is flat — it is a deliberate
  classification, not an unfinished migration. Future audits will not
  re-flag `vianium-crypto` as "messy".

### Negative

- Five repos are now formally non-conforming. That is honest
  bookkeeping, not new breakage — but it is visible debt the org
  carries until paid.

### Neutral

- The non-conforming repos (`vianium-tls`, `vianium-http`,
  `vianium-mtproxy`, `vianium-store`, `vianium-localsend`) **build and
  function correctly today**. Their refactor is deferred to after the
  v0.1.0 publication. Refactoring working code under release pressure
  trades real risk for a benefit (maintainability) that is not
  time-sensitive.

## Alternatives considered

- **Mandate Hexagonal + DDD for every repo.** Rejected: imposes
  ports/adapters ceremony on primitive libraries (crypto, kernel)
  that have no domain to model. Makes simple code harder to read.
- **Leave it unwritten — let each repo keep its current shape.**
  Rejected: guarantees continued drift; every new file is a coin
  flip on layout.
- **Refactor all five outliers before v0.1.0.** Rejected: `tls`,
  `http`, `mtproxy`, `store` already compile and pass their tests;
  `localsend` is a small app. Restructuring working code immediately
  before a release is risk with no release-blocking payoff.

## Technical debt — refactors deferred to post-v0.1.0

Tracked so the deferral is a decision, not an oversight:

1. **`vianium-tls`** — add `application/` (handshake orchestration use
   cases) and `ports/{inbound,outbound}/` (the transport + policy
   interfaces). The `domain/` layer already exists.
2. **`vianium-http`** — same as tls: lift the request/response
   pipeline into `application/`, formalize transport `ports/`.
3. **`vianium-mtproxy`** — add `ports/{inbound,outbound}/`
   (`IMtProxyTransport`, `IMtProxyCodec`) and an `application/`
   orchestration layer; `domain/value_objects/` + `infrastructure/`
   already exist.
4. **`vianium-store`** — regroup `Auth/`, `Cache/`, `Grpc/`, `Http/`,
   `Json/`, `Protobuf/` into `domain/` + `application/` +
   `infrastructure/` + `ports/`. Largest of the five; schedule first.
5. **`vianium-localsend`** — split the single project into bounded
   contexts under `Core/` (e.g. Discovery, Transfer, Pairing), each
   Category-2 internally. Smallest domain of the four products, so a
   2–3 context split is enough; do not over-split.

Each item is independent and can be scheduled separately. None blocks
v0.1.0.

## References

- ADR-0001: Multi-repo over monorepo
- ADR-0004: Native TLS via WinRT projection
- [Vianium Architecture Overview](../architecture-overview.md)
