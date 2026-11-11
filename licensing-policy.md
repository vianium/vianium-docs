# Vianium Licensing Policy

The Vianium project applies a dual licensing strategy by kind of
repository. This document is the canonical source. Each repo's `LICENSE`
file contains the binding legal text; this document explains the
rationale and operational rules.

## Repository-to-license mapping

| Kind | Repositories | License |
|---|---|---|
| App | `vianium-localsend` | PolyForm-Noncommercial-1.0.0 |
| Meta (docs, org templates) | `vianium-docs`, `.github` | Apache-2.0 |

## Why two licenses

Documentation and templates are licensed Apache 2.0. This maximizes
their reach, and the `NOTICE` requirement guarantees that author
attribution travels with them.

The app is licensed PolyForm Noncommercial 1.0.0. This keeps the source
visible to users, students, researchers, and noncommercial
organizations, but blocks commercial repackaging, paid SaaS, and resale.

Both licenses include a defensive patent grant that terminates a user's
license if they sue over a patent claim covering the code.

## Apache 2.0 (Meta)

What users can do:

- Use the content for any purpose, including commercial.
- Modify and create derivative works.
- Redistribute in source or binary form, with or without changes.

What users must do:

- Include a copy of the Apache License in any redistribution.
- Preserve copyright, patent, trademark, and attribution notices,
  including the `NOTICE` file.
- Mark modified files as modified.

What users may not do:

- Use the author's name or "Vianium" to endorse derivative products
  without permission.

## PolyForm Noncommercial 1.0.0 (App)

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
source. Public-facing descriptions should say "source-available" or
"free for non-commercial use," not "open source."

LocalSend itself is a third-party project; its name and protocol belong
to their respective authors.

## Commercial licensing

If you want to use `vianium-localsend` commercially (resell, embed in a
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

Every source file must begin with an SPDX header.

Apache-2.0 files:

```cpp
// SPDX-License-Identifier: Apache-2.0
// SPDX-FileCopyrightText: 2026 Angel Careaga <hello@angelcareaga.com>
```

PolyForm-Noncommercial files:

```cpp
// SPDX-License-Identifier: PolyForm-Noncommercial-1.0.0
// SPDX-FileCopyrightText: 2026 Angel Careaga <hello@angelcareaga.com>
```
