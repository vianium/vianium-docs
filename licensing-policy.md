# Vianium Licensing Policy

The Vianium project uses a dual licensing strategy applied by repository
tier. This document is the canonical source. Each repo's `LICENSE` file
contains the binding legal text; this document explains the rationale and
operational rules.

## Tier-to-license mapping

| Tier | Purpose | Repositories | License |
|---|---|---|---|
| 1 | Foundation libraries | `vianium-kernel`, `vianium-managed-kernel`, `vianium-crypto`, `vianium-tls`, `vianium-net`, `vianium-http`, `vianium-grpc`, `vianium-audio`, `vianium-fs`, `vianium-icons` | Apache-2.0 |
| 2 | Domain protocols and libraries | `vianium-mtproto`, `vianium-mtproxy`, `vianium-voip`, `vianium-media`, `vianium-image-palette`, `vianium-innertube`, `vianium-store`, `vianium-genai` | Apache-2.0 |
| 3 | Products | `vianium-browser`, `vianigram`, `vianium-music`, `vianium-localsend`, `vianium-chat`, `vianium-explorer` | PolyForm-Noncommercial-1.0.0 |
| Meta | Org metadata, docs | `.github`, `vianium-docs` | Apache-2.0 |

**Aggregate:** 26 repositories — 18 Apache-2.0 (10 foundation + 8 domain), 6 PolyForm-Noncommercial-1.0.0 (products), 2 Apache-2.0 meta.

## Why two licenses

Foundation and protocol libraries (Tier 1+2) are licensed Apache 2.0.
This maximizes their reach: any developer, including those building
commercial software, can adopt them. Apache 2.0's `NOTICE` requirement
guarantees that author attribution travels with the code wherever it
goes. The goal is wide adoption that propagates attribution.

Products (Tier 3) are licensed PolyForm Noncommercial 1.0.0. This keeps
the source visible to users, students, researchers, and noncommercial
organizations, but blocks commercial repackaging, paid SaaS, and resale.
The goal is broad noncommercial access without losing control of the
finished product.

Both licenses include a defensive patent grant that terminates a user's
license if they sue over a patent claim covering the code.

## Apache 2.0 (Tier 1, Tier 2, Meta)

What users can do:

- Use the code for any purpose, including commercial.
- Modify and create derivative works.
- Redistribute in source or binary form, with or without changes.
- Sublicense and incorporate into proprietary software.

What users must do:

- Include a copy of the Apache License in any redistribution.
- Preserve copyright, patent, trademark, and attribution notices,
  including the `NOTICE` file.
- Mark modified files as modified.

What users may not do:

- Use the author's name or "Vianium" to endorse derivative products
  without permission.

## PolyForm Noncommercial 1.0.0 (Tier 3)

What users can do:

- Use the software for any noncommercial purpose: personal use,
  research, study, hobbies, charity, education, government.
- Modify the software.
- Redistribute under the same license to other noncommercial users.

What users may not do:

- Use the software for commercial purposes. This includes selling it,
  offering it as a paid service, including it in a commercial product,
  or using it internally in a for-profit company.
- Remove or alter license or copyright notices.

Important: PolyForm Noncommercial is source-available, not OSI open
source. Public-facing descriptions of Tier 3 products should say
"source-available" or "free for non-commercial use," not "open source."

## Commercial licensing

If you want to use a Tier 3 product commercially (resell, embed in a
commercial product, deploy inside a for-profit organization), contact
[hello@angelcareaga.com](mailto:hello@angelcareaga.com) for a separate
commercial license.

## Required files per repository

Every Vianium repository must include:

- `LICENSE` — full text of the assigned license.
- `NOTICE` — attribution.
- `AUTHORS.md` — founding author and contributors.
- `README.md`
- `CONTRIBUTING.md`
- `CODE_OF_CONDUCT.md`
- `SECURITY.md`
- `.gitignore`
- `.gitattributes`
- `.editorconfig`

Repos with vendored third-party code must also include:

- `THIRD_PARTY_NOTICES.md` — catalog with license per upstream component.
- `LICENSES/` — full text of each upstream license.

## SPDX headers

Every source file must begin with an SPDX header. The
`Bootstrap-VianiumRepo.ps1` and `Add-SpdxHeaders.ps1` scripts apply
these automatically.

Tier 1 and Tier 2 source files:

```cpp
// SPDX-License-Identifier: Apache-2.0
// SPDX-FileCopyrightText: 2026 Angel Careaga <hello@angelcareaga.com>
```

Tier 3 source files:

```cpp
// SPDX-License-Identifier: PolyForm-Noncommercial-1.0.0
// SPDX-FileCopyrightText: 2026 Angel Careaga <hello@angelcareaga.com>
```
