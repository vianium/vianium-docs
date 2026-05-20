# ADR-0004: Native TLS via WinRT projection, no managed reimplementation

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-05-18 |
| Decider | Angel Careaga |
| Supersedes | (none) |
| Affects | `vianium-tls`, `vianium-grpc`, `vianium-store` |

## Context

Two TLS implementations existed at the start of the Vianium ecosystem
migration:

1. **Native C++ `Vianium.Core.Tls`** (from VianiumBrowser): TLS 1.3
   Mozilla Modern policy, pinning, OCSP, CT. Used by the native
   browser and Telegram code paths.
2. **Managed C# `PivoraTLS`** (from PivoraStore): TLS 1.2 with ALPN
   "h2" negotiation, ECDHE_ECDSA_AES_128_GCM_SHA256, P-256 ECDH using
   software BigInteger. Used by the Firestore SDK's gRPC transport.

The two implementations exist for historical reasons: the C# SDK was
built before the native TLS stack was extracted as a sibling library,
and writing a managed TLS implementation avoided P/Invoke complexity at
the time.

Maintaining two TLS stacks creates cost:

- Two codebases to audit, patch for vulnerabilities, and keep aligned
  with policy (Mozilla Modern, deprecated cipher removal).
- Divergence risk: the managed implementation supports TLS 1.2 only;
  the native one supports TLS 1.3. Consumers using the managed stack
  miss out on modern protocol features.
- Confusing tier story: shipping both `vianium-tls` and
  `vianium-managed-tls` as Tier 1 libraries forces every consumer to
  choose between near-duplicates.

## Decision

The native `vianium-tls` library is the single TLS implementation in
the Vianium ecosystem. The managed implementation
(`PivoraTLS` legacy) is **discarded** and not migrated to a
`vianium-managed-tls` repository.

C# consumers (specifically `vianium-grpc` and through it
`vianium-store`) reach the native TLS implementation through a Windows
Runtime Component (`C++/CX`) projection that lives inside the
`vianium-tls` repository as `Vianium.Core.Tls.WinRT.vcxproj`.

The projection exposes a managed-friendly API surface (`TlsStream`,
`TlsHandshake`, `AlpnProtocols`) that mirrors what `PivoraTLS`
exposed today, so the migration of dependent C# code is a transport-
adapter swap rather than a full rewrite.

## Consequences

### Positive

- One TLS implementation to audit, patch, and document. Security
  research and protocol updates land in one place.
- C# consumers transparently benefit from TLS 1.3, advanced cipher
  suites, OCSP stapling, Certificate Transparency, and pinning — all
  features the native stack has, the managed one did not.
- Clearer tier story: `vianium-tls` is Tier 1 and serves both native
  and managed consumers. No parallel `vianium-managed-tls`.
- The WinRT projection pattern, once proven on TLS, applies cleanly to
  future native libraries that need managed exposure
  (`vianium-image-palette`, `vianium-media`, `vianium-innertube`).

### Negative

- The migration from `PivoraTLS` to `vianium-tls` WinRT requires
  refactor work on `vianium-grpc` and `vianium-store` before
  publication. Estimated cost: ~2 days of focused engineering.
  Accepted as the cost of a robust v0.1.0.
- WinRT projection adds a small per-call marshalling cost across the
  managed/native boundary. Expected to be negligible for TLS-rate
  operations (handshake, occasional rekeying); the bulk data path
  uses `IBuffer` which is zero-copy on WP8.1.
- The projection layer becomes an additional API surface to keep
  stable across releases. Mitigated by versioning the projection
  alongside the native API.

### Neutral

- `PivoraRPC` (gRPC framing in C#) is retained as `vianium-grpc`
  because gRPC framing is a thin layer over HTTP/2 streams and is
  comfortable in managed code. Only its TLS dependency switches from
  `PivoraTLS` to the `vianium-tls` WinRT projection.

## Alternatives considered

- **Keep `PivoraTLS` as `vianium-managed-tls`.** Rejected: doubles
  maintenance cost and creates a tier of duplicates. Discoverable as
  a strategic mistake in 12+ months.
- **Migrate the entire transport stack to native (gRPC framing in
  C++ too).** Rejected for v0.1.0: weeks of refactor work in
  `vianium-http` to add gRPC framing support, with no functional
  benefit over a thin managed gRPC layer.
- **P/Invoke from C# directly into native TLS.** Rejected: WinRT
  projection is the supported pattern on WP8.1, is type-safe, and
  generates `.winmd` metadata the C# compiler consumes natively.
  P/Invoke would require manual marshalling and hand-written
  signatures.
- **Defer the managed-TLS removal to v0.2.0** and ship v0.1.0 with
  `PivoraTLS` embedded. Rejected: contradicts the architecture
  declared in `vianium-docs` and creates a confusing first impression
  for early adopters.

## Implementation notes

The WinRT projection lives inside the `vianium-tls` repository:

```text
vianium-tls/
  src/                              (native C++ TLS engine, unchanged)
  winrt/
    Vianium.Core.Tls.WinRT.vcxproj  (C++/CX component)
    TlsStream.h, TlsStream.cpp      (projects to managed TlsStream)
    TlsClient.h, TlsClient.cpp      (projects to managed TlsClient)
    AlpnProtocols.h, AlpnProtocols.cpp
  Vianium.Core.Tls.vcxproj          (native lib)
```

Consumers reference both projects in their `.csproj`:

```xml
<ProjectReference Include="..\vianium-tls\winrt\Vianium.Core.Tls.WinRT.vcxproj" />
```

Migration cost on `vianium-store`:

1. Replace `using PivoraTLS;` with `using Vianium.Core.Tls;`.
2. Swap `PivoraTLS.TlsStream` for `Vianium.Core.Tls.TlsStream` in the
   gRPC transport adapter.
3. Re-validate handshake, ALPN "h2", end-to-end Firestore flows
   (auth, queries, listeners, batched writes, transactions).

## References

- [Vianium Architecture Overview](../architecture-overview.md) §
  "Native and managed interop: WinRT projection"
- ADR-0001: Multi-repo over monorepo
- ADR-0002: Dual licensing
