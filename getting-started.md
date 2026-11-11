# Getting Started

This guide explains how to clone and build
[vianium-localsend](https://github.com/vianium/vianium-localsend) for
development. It is a standalone app with no external library
dependencies, so no sibling repositories are required.

## Cloning

```powershell
git clone https://github.com/vianium/vianium-localsend.git
```

Optionally clone the docs and org templates alongside it:

```powershell
git clone https://github.com/vianium/vianium-docs.git
git clone https://github.com/vianium/.github.git
```

## Prerequisites

- Windows 10 or 11.
- **Visual Studio 2015 (Update 3)** with the Windows Phone 8.1 SDK
  extension installed.
- MSBuild 14.0 (ships with VS 2015).
- Developer Command Prompt for VS 2015.

## Building LocalSend

```cmd
cd vianium-localsend
MSBuild Vianium.LocalSend.sln /p:Configuration=Debug /p:Platform=x86
```

See the repository's own `README.md` for run, deploy and smoke-test
instructions.

## Common operations

- Sign off a commit:

  ```sh
  git commit -s -m "fix: my change"
  ```

## Reporting issues

- Bugs and feature requests: open an issue in the relevant repo.
- Questions and discussions: use the Discussions tab on
  [vianium-docs](https://github.com/vianium/vianium-docs/discussions).
- Security: see
  [SECURITY.md](https://github.com/vianium/.github/blob/main/SECURITY.md).
