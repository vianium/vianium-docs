# Contributing to Vianium

This guide applies across all Vianium repositories. Per-repo
`CONTRIBUTING.md` files may add repo-specific rules but never weaken the
ones below.

## Before contributing

1. Read [licensing-policy.md](licensing-policy.md) to understand which
   license applies to the repo you want to contribute to.
2. By submitting a contribution, you agree it will be licensed under the
   same license as the receiving repository.

## Developer Certificate of Origin (DCO)

All contributions require DCO sign-off. The DCO is a lightweight
alternative to a contributor license agreement: by signing off, you
certify that you wrote the code or have the right to submit it under the
project license.

Use the `-s` flag when committing:

```sh
git commit -s -m "your message"
```

This adds a trailer like:

```text
Signed-off-by: Your Name <your.email@example.com>
```

The name and email must match your real identity (no pseudonyms). Full
DCO text: <https://developercertificate.org/>.

## Workflow

1. Open an issue first for non-trivial changes. For large features,
   prefer a Discussion in `vianium-docs`.
2. Fork the repository.
3. Create a topic branch: `feat/short-description`,
   `fix/short-description`, or `docs/short-description`.
4. Make your changes following the repo's existing style (see
   `.editorconfig`).
5. Add or update tests where applicable.
6. Commit with `git commit -s`.
7. Push to your fork and open a pull request against `main`.
8. Fill in the PR template completely.

## Pull request requirements

- All commits signed off (DCO).
- CI green.
- At least one approving review from a maintainer.
- No merge conflicts with `main`.
- License headers preserved (SPDX comments at the top of every source
  file).
- `NOTICE` and `LICENSE` files preserved if you moved or renamed files.

## Coding style

### C++ (native code, Tier 1 and Tier 2)

- C++11/14 only. Do not introduce C++17+ features unless the toolchain
  changes.
- Avoid exceptions in performance-critical paths. Use `Result<T>` from
  `vianium-kernel`.
- Avoid the C++ Standard Library in performance-critical paths where it
  would force allocations. Prefer arena-allocated structures.

### C# (managed code, Tier 3 product code targeting WP8.1)

- C# 6 features only. No `record`, no nullable reference types, no
  `init`.

### PowerShell (tooling)

- Make scripts idempotent. Re-running should be a no-op when the system
  is already in the desired state.
- Support `-WhatIf` for any destructive operation.

### Line endings

- LF for source files.
- CRLF for `.sln` and `.vcxproj` files.

Enforced by `.gitattributes`.

## Third-party code

If your contribution adds third-party code:

1. Add an entry to `THIRD_PARTY_NOTICES.md`.
2. Preserve the upstream license under `LICENSES/`.
3. Confirm license compatibility with the receiving repo's license.

## Communication

- Issues: bug reports and feature requests.
- Discussions (in `vianium-docs`): questions and design conversations.
- Pull requests: code changes.

Be respectful. See `CODE_OF_CONDUCT.md`.
