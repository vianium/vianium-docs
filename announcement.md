# Introducing Vianium — open-source foundation for Windows Phone 8.1

> 2026-05-19 · by Angel Careaga
> Status: draft — to be published when the org goes live

After a year of writing native libraries and apps for **Windows Phone 8.1**
in isolation, I'm releasing the underlying work as the
[**Vianium**](https://github.com/vianium) open-source ecosystem.

Twenty-six repositories. Four tiers. Two licenses. One person.

---

## What this is

Vianium is a ground-up rewrite of the modern web + protocol stack
targeting the `v120_wp81` toolset (Visual Studio 2015 SP3 + Windows
Phone 8.1 SDK). It is the **last platform Microsoft made for which a
modern client of Telegram, YouTube Music, or a current TLS 1.3 web
browser does not exist** — until now.

| Tier | Repos | License | Description |
|---|---|---|---|
| 1 — Foundation | 10 | Apache 2.0 | `vianium-kernel` (allocator, Result, base types), `vianium-crypto` (AES, SHA, BigNum, ECDH), `vianium-tls` (TLS 1.3 Mozilla Modern), `vianium-net` (sockets, DNS, DoH), `vianium-http` (HTTP/1.1, HTTP/2, HPACK), `vianium-grpc` (managed gRPC over HTTP/2), `vianium-managed-kernel` (managed analogue of `vianium-kernel`), `vianium-audio` (WASAPI capture/playback, PCM resampling), `vianium-fs` (Win32 filesystem engine with WinRT projection), `vianium-icons` (Fluent UI System Icons font + catalog) |
| 2 — Domain | 8 | Apache 2.0 | `vianium-mtproto` (Telegram MTProto 2.0 + TL), `vianium-mtproxy` (obfuscated transport), `vianium-voip` (RTP/SRTP, Opus, libtgvoip wrapper), `vianium-media` (image + audio + video decoders, FFmpeg LGPL), `vianium-image-palette` (image palette extraction), `vianium-innertube` (YouTube Innertube API), `vianium-store` (Firestore SDK + samples), `vianium-genai` (provider-neutral AI chat client — Gemini, OpenAI) |
| 3 — Products | 6 | PolyForm Noncommercial 1.0.0 | `vianigram` (Telegram client), `vianium-browser` (web browser), `vianium-music` (YouTube Music client), `vianium-localsend` (LocalSend protocol v2 client), `vianium-chat` (multi-provider AI chat client), `vianium-explorer` (file explorer for WP8.1) |
| Meta | 2 | Apache 2.0 | `vianium-docs` (architecture, ADRs, getting-started), `.github` (org profile + default community files) |

---

## Why publish now

Three reasons:

1. **The platform has no successor.** Every other modern OS has a fully
   maintained ecosystem. WP 8.1 (and the embedded WinRT-based stack
   shared with Surface RT, IoT Core, and several industrial devices)
   does not. Releasing this work is the only way to keep that
   ecosystem alive.

2. **The libraries are reusable.** TLS 1.3, HTTP/2, MTProto, Innertube,
   gRPC over HTTP/2 — these aren't WP-specific. The same C++ + WinRT
   pattern compiles on Windows 10 Mobile, Windows 10/11 (Universal),
   and the embedded WinRT family. Apache 2.0 means anyone, anywhere,
   for any reason — including in commercial products.

3. **One person can't maintain it alone forever.** Open source is the
   succession plan.

---

## How the licensing works

The **foundation and domain libraries** (18 repos) are **Apache 2.0**.
You can include them in any product, commercial or otherwise, with
attribution. Use them, build with them, sell with them.

The **products** (6 repos) are **PolyForm Noncommercial 1.0.0**.
You can read the source, modify it, distribute modifications, and use
it for personal, educational, or non-commercial purposes. Commercial
use of a product (e.g. selling a Telegram client built on `vianigram`)
requires a separate commercial license — contact
`hello@angelcareaga.com`.

Both licenses are OSI-aware (PolyForm-NC is source-available, not
strictly OSI-Open-Source, by design).

See [licensing-policy.md](licensing-policy.md) for the full rationale.

---

## How to start

For the impatient:

```bash
# Pick the layer you care about.
git clone https://github.com/vianium/vianium-kernel.git
git clone https://github.com/vianium/vianium-crypto.git

# Open the .vcxproj in Visual Studio 2015 (Update 3) with the
# WP 8.1 SDK installed. Build configurations: Debug|ARM, Release|ARM,
# Debug|Win32, Release|Win32.
```

For the curious:

- [vianium-docs](https://github.com/vianium/vianium-docs) — start with
  `getting-started.md`, then the four ADRs and the architecture overview.
- [`.github`](https://github.com/vianium/.github) — the org profile,
  default `FUNDING.yml`, default community files.

---

## How to help

The fastest way to help is by **trying the code** and **filing issues**
when something breaks on your device.

- Open issues in the specific repo where the breakage is, using the
  bug-report template.
- For protocol disagreements (MTProto frame layout, TLS 1.3 record
  ordering), include packet captures.
- For UI bugs in the products, screenshots help.

If `vianium-mtproto` makes your Telegram client work on a Windows Phone
that hasn't received an update in 7 years, that's the most important
contribution you can make.

To sponsor the work directly, see
[donations.md](donations.md) — the GitHub Sponsors and Buy Me a Coffee
buttons are wired into every repo's `.github/FUNDING.yml`.

---

## What comes next

In rough order:

1. **Stabilisation.** v0.1.0 across all 26 repos, then sweep for
   field-reported breakage.
2. **Documentation.** ADRs landed; tutorials missing. The
   [vianium-docs](https://github.com/vianium/vianium-docs) tree will
   grow first.
3. **CI.** Lint-level CI lands in every repo with the public release.
   Full native-build CI requires a self-hosted WP 8.1 runner; that's
   a 2026-Q3 item.
4. **Successor platforms.** Windows 10/11 Universal targets via the
   same code, plus an embedded WinRT manifest profile, are explicitly
   in scope. Linux/macOS ports are NOT planned — the C++/CX coupling
   to WinRT is the whole point.

---

## Acknowledgements

A few names without which this would not exist:

- **The TDLib team** (Aliaksei Levin, Arseny Smirnov) — `vianium-mtproto`
  cross-checks against TDLib's reference for every wire-format decision.
- **The libtgvoip authors** (Grishka, Telegram FZ-LLC) —
  `vianium-voip` vendors libtgvoip under the Unlicense.
- **The Mozilla Modern TLS team** — `vianium-tls` follows the Modern
  ciphersuite list as the canonical configuration baseline.
- **The FFmpeg team** — `vianium-media` vendors the LGPL build of
  FFmpeg for the audio/video codec surface.

And anyone whose Windows Phone is still working after seven years —
this is for you.

---

**Angel Careaga** · [angelcareaga.com](https://angelcareaga.com) · [@AngelCareaga](https://github.com/AngelCareaga) · `hello@angelcareaga.com`
