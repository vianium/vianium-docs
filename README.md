# vianium-docs
[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE) [![Vianium](https://img.shields.io/badge/org-vianium-00A0F0.svg)](https://github.com/vianium) [![Issues](https://img.shields.io/github/issues/vianium/vianium-docs.svg)](https://github.com/vianium/vianium-docs/issues)

Cross-repository documentation, governance, and architecture decision
records for the [Vianium](https://github.com/vianium) project.

## Read this if you want to understand...

| ...this | Start with |
|---|---|
| What Vianium is and why | [architecture-overview.md](architecture-overview.md) |
| The repo tiers and dependency flow | [architecture-overview.md](architecture-overview.md) |
| Why we use two licenses | [licensing-policy.md](licensing-policy.md) and [adr/0002-dual-licensing.md](adr/0002-dual-licensing.md) |
| How to contribute across the ecosystem | [contribution-guide.md](contribution-guide.md) |
| How to clone and build the whole ecosystem | [getting-started.md](getting-started.md) |
| Why multi-repo instead of monorepo | [adr/0001-multi-repo.md](adr/0001-multi-repo.md) |
| Why we kept the name Vianigram | [adr/0003-vianigram-name.md](adr/0003-vianigram-name.md) |
| Why C# consumes native TLS via WinRT (not a managed reimplementation) | [adr/0004-native-tls-winrt-projection.md](adr/0004-native-tls-winrt-projection.md) |
| How each component is organized internally (Hexagonal / DDD / flat) | [adr/0005-component-architecture-standard.md](adr/0005-component-architecture-standard.md) |

## Architecture Decision Records

ADRs capture significant design decisions for the ecosystem as a whole.
Per-repo decisions live in each repository's own `docs/adr/` folder when
needed.

- [0001 — Multi-repo over monorepo](adr/0001-multi-repo.md)
- [0002 — Dual licensing: Apache for libraries, PolyForm-NC for products](adr/0002-dual-licensing.md)
- [0003 — Keep the product name Vianigram](adr/0003-vianigram-name.md)
- [0004 — Native TLS via WinRT projection, no managed reimplementation](adr/0004-native-tls-winrt-projection.md)
- [0005 — Component architecture standard: pattern per component category](adr/0005-component-architecture-standard.md)

New ADRs follow the format `NNNN-short-slug.md` and are numbered
sequentially.

## License

This documentation is published under the **Apache License 2.0**. See
[LICENSE](LICENSE) and [NOTICE](NOTICE).

The licenses applied to source code in other Vianium repos are described
in [licensing-policy.md](licensing-policy.md).

## Support this project

Vianium is maintained by [Angel Careaga](https://angelcareaga.com) as a
personal open-source effort. If `vianium-docs` is useful to you, please
consider supporting future work:

- 💖 **[GitHub Sponsors](https://github.com/sponsors/vianium)** — recurring or one-time
- ☕ **[Buy Me a Coffee](https://www.buymeacoffee.com/soyangelcareaga)** — one-time tip, no account needed
- 🌐 **[angelcareaga.com](https://angelcareaga.com)** — contact, consulting

Detailed channels and a transparency page live in
[`SUPPORT.md`](SUPPORT.md) and
[vianium-docs/donations.md](https://github.com/vianium/vianium-docs/blob/main/donations.md).

## Author

**Angel Careaga** ·
[@AngelCareaga](https://github.com/AngelCareaga) ·
[hello@angelcareaga.com](mailto:hello@angelcareaga.com) ·
[angelcareaga.com](https://angelcareaga.com)
