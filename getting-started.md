# Getting Started with Vianium

This guide explains how to clone the Vianium ecosystem locally and build
it for development.

## Workspace layout

All Vianium repos are designed to live as siblings under a common
workspace directory. Integration between repos uses relative paths in
`.vcxproj` / `.sln` files.

```text
workspace/
  .github/                  meta — org profile + default community files
  vianium-docs/             meta — architecture, ADRs, donations, this guide

  vianium-kernel/           Tier 1 foundation (Apache 2.0)
  vianium-managed-kernel/
  vianium-crypto/
  vianium-tls/
  vianium-net/
  vianium-http/
  vianium-grpc/
  vianium-audio/
  vianium-fs/
  vianium-icons/

  vianium-mtproto/          Tier 2 domain (Apache 2.0)
  vianium-mtproxy/
  vianium-voip/
  vianium-media/
  vianium-image-palette/
  vianium-innertube/
  vianium-store/
  vianium-genai/

  vianium-browser/          Tier 3 product (PolyForm-Noncommercial-1.0.0)
  vianigram/
  vianium-music/
  vianium-localsend/
  vianium-chat/
  vianium-explorer/
```

Twenty-six repos total: 10 foundation + 8 domain + 6 products + 2 meta.

You can clone only the repos you need to work on, plus their transitive
dependencies. The
[architecture overview](architecture-overview.md) dependency graph
shows what each repo depends on.

## Cloning

Pick or create a workspace directory, then clone the repos you need.

```powershell
$workspace = "D:\Projects\2026\VianiumProject"
New-Item -ItemType Directory -Force -Path $workspace | Out-Null
Set-Location $workspace

# Foundation (Tier 1, Apache 2.0)
git clone https://github.com/vianium/vianium-kernel.git
git clone https://github.com/vianium/vianium-managed-kernel.git
git clone https://github.com/vianium/vianium-crypto.git
git clone https://github.com/vianium/vianium-tls.git
git clone https://github.com/vianium/vianium-net.git
git clone https://github.com/vianium/vianium-http.git
git clone https://github.com/vianium/vianium-grpc.git
git clone https://github.com/vianium/vianium-audio.git
git clone https://github.com/vianium/vianium-fs.git
git clone https://github.com/vianium/vianium-icons.git

# Domain (Tier 2, Apache 2.0) — only the ones you need
git clone https://github.com/vianium/vianium-mtproto.git
git clone https://github.com/vianium/vianium-mtproxy.git
git clone https://github.com/vianium/vianium-voip.git
git clone https://github.com/vianium/vianium-media.git
git clone https://github.com/vianium/vianium-image-palette.git
git clone https://github.com/vianium/vianium-innertube.git
git clone https://github.com/vianium/vianium-store.git
git clone https://github.com/vianium/vianium-genai.git

# Products (Tier 3, PolyForm Noncommercial 1.0.0) — pick what you want
git clone https://github.com/vianium/vianium-browser.git
git clone https://github.com/vianium/vianigram.git
git clone https://github.com/vianium/vianium-music.git
git clone https://github.com/vianium/vianium-localsend.git
git clone https://github.com/vianium/vianium-chat.git
git clone https://github.com/vianium/vianium-explorer.git

# Docs
git clone https://github.com/vianium/vianium-docs.git
```

## Prerequisites

- Windows 10 or 11.
- **Visual Studio 2015 (Update 3)** with the Windows Phone 8.1 SDK
  extension installed.
- MSBuild 14.0 (ships with VS 2015).
- The **`v120_wp81`** platform toolset (selected automatically when
  the WP 8.1 SDK is installed).
- Developer Command Prompt for VS 2015.

For libraries (Tier 1, Tier 2), other Visual Studio versions may work
as long as MSBuild 14.0 and the v120_wp81 platform toolset are
available. Visual Studio 2013 historically worked for an earlier
Pivora-prefixed iteration of the codebase but is **not** the supported
target — current sources assume the C++11 / C++/CX surface the
v120_wp81 + VS 2015 SP3 combination provides.

## Building a single library

Each library has its own minimal `.sln` for standalone builds.

```cmd
cd vianium-kernel
MSBuild Vianium.Core.Kernel.sln /p:Configuration=Debug /p:Platform=x86
```

## Building a product

Products have their own `.sln` that references sibling repos via
relative paths. The sibling repos must be present in the same workspace
directory.

For Vianigram:

```cmd
cd vianigram
build-validate.cmd Debug x86
```

The build script validates that all required sibling repos exist before
invoking MSBuild.

## Common operations

- Add SPDX headers to new source files:

  ```powershell
  .\tools\Add-SpdxHeaders.ps1 -Path .\src -License Apache-2.0
  ```

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
