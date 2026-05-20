# Vianium org — pending work roadmap

> Generated: 2026-05-19
> Source: 10-agent parallel audit + manual cross-check
> Status: snapshot before manual publish
> Owner: Angel Careaga — all items below require human execution

This document is the single source of truth for what is NOT done yet.
Everything documented as completed in `MEMORY.md` / closed tasks is in
the working tree at `D:\Projects\2026\VianiumProject\` and verified by
`tools/Validate-Org.ps1` (0 fail / 0 warn at last run).

Items are categorised by **who** must do them and **when**.

---

## P0 — Blockers that prevent a clean publish

### P0.1 — Verify the build in Visual Studio 2015

**Why:** Agents perform static analysis only. The only way to confirm
the full graph compiles is to open the .sln in VS 2015 (Update 3) with
the WP 8.1 SDK installed.

**Action:**
```cmd
cd D:\Projects\2026\VianiumProject\vianigram
start Vianigram.sln
```
Build configurations to try: `Debug|ARM`, `Release|ARM`, `Debug|x86`,
`Release|x86`. Order: build vianium-kernel first, then crypto, then
tls, then net, then http, then mtproxy, then mtproto, then up the
stack.

**Expected failure modes already triaged (still need a try):**
- WinRT type-resolution mismatch — see P0.2.
- Duplicate ProjectGuid `{8AAE0001-…}` between
  `vianium-managed-kernel\Vianium.Browser.Kernel.csproj` and
  `vianium-browser\Core\Vianium.Browser.Kernel\Vianium.Browser.Kernel.csproj`
  — see P0.3.

If anything else breaks, capture the error + the project that fails
and file an issue in the affected repo.

### P0.2 — Decide on the `Vianium.MTProto` ↔ `Vianigram.MTProto` namespace/filename drift

**Symptom:**
- `vianium-mtproto\Vianium.MTProto.vcxproj` line 23 declares
  `<ProjectName>Vianium.MTProto</ProjectName>` and line 70
  `<TargetName>Vianium.MTProto</TargetName>`, which means the project
  emits `Vianium.MTProto.dll` + `Vianium.MTProto.winmd`.
- The C++/CX source in
  `vianium-mtproto\src\mtproto\api\v1\mtproto_api.h:14` declares
  `namespace Vianigram { namespace MTProto { …` so the projected types
  are `Vianigram.MTProto.MtProtoChannel`, `Vianigram.MTProto.MtProxyRuntime`,
  etc.
- Same divergence for `Vianium.Tl.vcxproj` (TargetName) vs
  `namespace Vianigram::Tl` (source).
- Same divergence for the `Vianigram.Core.Media` and `Vianigram.Core.Voip`
  projects (in the vianigram repo) which emit Vianigram-prefixed
  .winmds matching their type namespaces — this is the canonical
  convention.

**Why it matters:** At the .winmd-reference level there's no breakage
(C# compiler binds by metadata token, not filename). At the AppX
packaging level, Windows finds the activation factory by scanning the
package; it works regardless. So this is **hygiene, not a runtime
bug** — confirmed because Vianigram already runs in development with
the current layout.

**Recommendation:** Rename for consistency. Option A (minimal touch,
preferred):
1. In `vianium-mtproto\Vianium.MTProto.vcxproj`: change
   `<ProjectName>` and `<TargetName>` from `Vianium.MTProto` to
   `Vianigram.MTProto`.
2. In `Vianium.Tl.vcxproj`: same rename to `Vianigram.Tl`.
3. Rename the .vcxproj files on disk to match (`Vianigram.MTProto.vcxproj`,
   `Vianigram.Tl.vcxproj`).
4. Search-and-replace `Vianium.MTProto.vcxproj` → `Vianigram.MTProto.vcxproj`
   and `Vianium.Tl.vcxproj` → `Vianigram.Tl.vcxproj` in:
   - `D:\Projects\2026\VianiumProject\vianigram\Vianigram.sln`
   - `vianigram\Core\Vianigram.Composition\Vianigram.Composition.csproj`
   - `vianigram\Clients\Vianigram.App\Vianigram.App.csproj`
   - `vianigram\Clients\Vianigram.SmokeTests\Vianigram.SmokeTests.csproj`
   - `vianium-mtproto\Vianium.MTProto.vcxproj` itself (SolutionOutputSyncDir + AdditionalDependencies)
5. Update the `Vianium.MtProxy.vcxproj` link-time dependency (if any).

This makes the .winmd filename match the namespace, which is the
WP 8.1 convention used by every Microsoft sample.

### P0.3 — Resolve duplicate `ProjectGuid {8AAE0001-…}`

**Where:**
- `D:\Projects\2026\VianiumProject\vianium-managed-kernel\Vianium.Browser.Kernel.csproj` (canonical)
- `D:\Projects\2026\VianiumProject\vianium-browser\Core\Vianium.Browser.Kernel\Vianium.Browser.Kernel.csproj` (legacy in-tree copy)

**Why it matters:** Two projects with the same `ProjectGuid` in the
same solution confuse MSBuild's incremental build cache (the second
project always re-builds; one of them silently steals the other's
output paths in some configurations).

**Recommendation:** Delete the legacy in-tree copy in vianium-browser
and have vianium-browser's solution + dependent projects reference the
sibling `..\vianium-managed-kernel\Vianium.Browser.Kernel.csproj`
instead. This was the migration intent per `MIGRATION_PLAN.md`
("Tier 1 — managed-kernel extracted from browser").

After the delete:
- Edit `vianium-browser\VianiumBrowser.sln` to update the project path.
- Edit any browser-internal `<ProjectReference>` to point at the sibling.
- Re-run `Validate-Org.ps1` to confirm no other repo references the deleted path.

### P0.4 — Fix `vianium-grpc\Vianium.Grpc.csproj` ProjectReference GUID

**Symptom:** Lines 126–129 declare
`<Project>{C1A2B3D4-E5F6-7A8B-9C0D-E1F2A3B4C5D7}</Project>` for the
managed-kernel reference, but the actual `ProjectGuid` is
`{8AAE0001-0000-0000-0000-000000000001}`.

**Action:** After resolving P0.3, update the declared GUID to whichever
copy survives. Single-line edit.

### P0.5 — Restore or remove `_vendor/telegram-official` reference

**Symptom:** `vianigram\Clients\Vianigram.App\Vianigram.App.csproj`
line 620 has a `<Content Include="..\..\..\_vendor\telegram-official\…\countries.txt">`
that resolves to a directory which does NOT exist in the workspace.

**Action:** Either:
- Drop the `<Content>` entry (the country list is also embedded in
  `Strings\en-US\Resources.resw`), OR
- Restore the Telegram-official submodule/vendor drop at
  `D:\Projects\2026\VianiumProject\_vendor\`.

### P0.6 — `vianigram\Clients\Vianigram.SmokeTests` missing 5 ProjectReference GUIDs

**Symptom:** 5 `<ProjectReference>` entries in `Vianigram.SmokeTests.csproj`
have no `<Project>{guid}</Project>` child:
`Vianigram.Kernel`, `Vianium.Core.Crypto.vcxproj`, `Vianium.Tl.vcxproj`,
`Vianium.MTProto.vcxproj`, `Vianigram.Core.Media.vcxproj`.

**Action:** Add the matching GUID for each. Look up the canonical
`<ProjectGuid>` in the target .csproj/.vcxproj and copy it in.

---

## P1 — High-value cleanups (should do before publish)

### P1.1 — Fix HIGH-severity C++ memory patterns in `vianium-tls`

From the memory audit (agent 8, HIGH-severity findings 1–3):
1. **`vianium-tls\src\tls\tls_record_layer.cpp:86,94,130`** + **`tls_core.cpp:320`** — convert `new uint8_t[length]` + manual `delete[]` to `std::vector<uint8_t>` (RAII through PPL continuations).
2. **`vianium-tls\src\winrt\TlsStream.cpp:172,176`** + **`vianium-tls\src\core\socket_transport.cpp:72`** — replace `new wchar_t[len + 1]` + `delete[]` with `std::vector<wchar_t>`.
3. **`vianium-tls\src\tls\handshake_parser.cpp:93`** — `new uint8_t[sessionIdLength]` has NO matching `delete[]` in the file. Likely leaks per handshake. Fix by owning the buffer via `std::vector<uint8_t>` inside `ServerHelloParseResult`.

### P1.2 — Fix HIGH-severity races in MTProxy transport (my own code)

From the memory audit (finding 5):

**`vianium-mtproto\src\mtproto\infrastructure\mtproxy_tcp_transport.cpp:211–228, 246–260`** — `ReadAsync` and `WriteAsync` capture `recv_key_ptr`, `send_key_`, `send_iv_` as raw pointers (pointers into `this`). If `Close()` zeroes those buffers while a `.then()` continuation is still in flight, the in-flight `AesCtrXorInPlace` decrypts against all-zero keys and silently corrupts the next read.

**Fix:** Capture by value (copy the 32+16 bytes into a stack array for the lambda), or wrap the key state in a `std::shared_ptr<KeyState>` and capture the shared_ptr.

### P1.3 — Fix the `EncryptedSession` raw `task<void>*` lifetime

**`vianium-mtproto\src\mtproto\application\encrypted_session.cpp:380,415,426`** — `receive_task_` is a raw `concurrency::task<void>*`. The self-deadlock branch at line 410 spawns a detached `create_task` to delete it. On shutdown the detached task may not run, leaking the task object + the captured `keepalive` `shared_ptr<EncryptedSession>` (which keeps the session alive indefinitely).

**Fix:** Hold by value (`concurrency::task<void> receive_task_`) or `std::unique_ptr<concurrency::task<void>>`.

### P1.4 — Orphan duplicate WinRT projection

The `Vianigram.Core.Voip` project (under `vianigram\Core\Vianigram.Core.Voip`) emits a separate `Vianigram::Voip::VoipRuntime` ref class that has **zero managed callers** — all managed VoIP code goes through `Vianium::VoIP`.

**Action:** Decide which one survives. The audit recommendation is to keep `vianium-voip` (Apache 2.0 sibling repo) and remove `Vianigram.Core.Voip` from the vianigram solution + composition root. Dual `VoipRuntime` registrations in the same appx is a footgun.

### P1.5 — Fix `vianium-store/README.md` tier classification

Line 212 lists `vianium-managed-kernel` as "Tier 2 - Domain". Canonical
is **Tier 1 - Foundation** (per `MIGRATION_PLAN.md`, `licensing-policy.md`,
the `vianium-org-structure` memory, and `donations.md`).

Also line 163: `Open vianium-store/src/Vianium.Store/Vianium.Store.csproj`
references a path that does not contain a .csproj.

### P1.6 — Remove "Pivora" from public READMEs

The migration policy is Vianium-only. `vianium-grpc/README.md` line 34
still references `PivoraTLS` (in the ADR-0004 migration-note context).
Reword or move to an ADR appendix.

Same in `vianium-store/README.md` lines 61–65 — the architecture
diagram leaks `TODO[ADR-0004]` markers into user-facing documentation.

### P1.7 — Localization gap: `QrLoginRetryButton.Content`

`vianigram\Clients\Vianigram.App\Pages\Auth\QrLoginPage.xaml:150` has
`x:Uid="QrLoginRetryButton"` but Resources.resw only ships a bare
`QrLoginRetryButton` key — no `.Content` child. The button shows a
hard-coded `Content="retry"` instead. Add the localized strings to
both `en-US/Resources.resw` and `es-ES/Resources.resw`.

---

## P2 — Architectural follow-ups (post-publish)

### P2.1 — Complete the ADR-0004 chain (vianium-tls WinRT projection)

`vianium-tls\winrt\` is intentional scaffolding (~35 TODOs across
`AlpnProtocols.{cpp,h}`, `TlsStream.{cpp,h}`, `TlsClient.{cpp,h}`).
Each throws `Platform::FailureException` referencing ADR-0004.

Once the projection is real:
- 9 `TODO[ADR-0004]` markers clear in `vianium-grpc\Http2\Http2Connection.cs` (lines 52, 56, 61, 67, 81, 102, 127, 146, 170, 185).
- 12 markers clear in `vianium-store\src\Vianium.Store\` (DocumentReference.cs, CrashLogger.cs, FirestoreSettings.cs, HttpTransportLayer.cs, QueryBuilder.cs, Grpc\FirestoreGrpcTransport*.cs, Grpc\Http2ListenConnection*.cs).

### P2.2 — Native ECDSA-P256/P384 verifier

`vianium-crypto\src\signature_verifier.cpp:9,163,230` defers pure-software
ECDSA verification (BCrypt.h not available on WP 8.1 Silverlight). Add
a portable ECDSA-P256 + P384 verifier in pure C++ (no platform crypto).

### P2.3 — Native HTTP/2 path

`vianium-http\src\client\http_connection_pool.cpp:77,94,95,151` —
HTTP/2 is documented as deferred. Currently falls back to HTTP/1.1.
Implement to unlock the perf benefits for vianium-store + vianigram CDN.

### P2.4 — VoIP V3.1 + V6 P4.4 follow-ups

- `vianium-voip\src\voip\src\application\voip_engine.cpp:1563` — media snapshot bridge
- `dtls_client_session.cpp:308` — V3.1 ECDSA re-verify (depends on P2.2)
- `tgcalls_connection.cpp:1199` — V6 P4.4-D payload refactor

### P2.5 — Music product follow-ups

`vianium-music\Core\Vianium.Music.Composition\Roots\MusicCompositionRoot.cs:753–756`
lists 4 phase markers: ArtistDetail (Fase 1.3), playlists polish (Fase 2.2),
streaming (Fase 3), lyrics/casting (Fase 4).

Casting stubs at `vianium-music\Core\Vianium.Music.Casting\Infrastructure\StubDlnaRenderer.cs`
+ `StubChromecastDevice.cs` return typed Error results ("not implemented
in this build") — promote to real impls.

EchoFind at `vianium-music\Core\Vianium.Music.EchoFind\Domain\Errors\EchoFindErrors.cs:22`
— same.

### P2.6 — Browser product follow-ups

`vianium-browser\Core\Vianium.Browser.Composition\Roots\BrowserCompositionRoot.cs:51–55`
phases 2–6: cross-context ACL, native bridges, extensions, presentation.

`vianium-browser\Core\Vianium.Browser.Credentials\` — encrypted creds
export/import deferred (post roadmap milestone A.5).

`vianium-browser\Core\Vianium.Core.WebEngine\src\api\v1\c_api.cpp:59,78,84`
+ `Vianium.Core.Wasm\…\winrt_shim.cpp:30` + `Vianium.Core.ExtSandbox\…\winrt_shim.cpp:30,31`
— browser engine Phase D orchestration, WASM Compile/Validate,
ExtSandbox lifecycle.

---

## P3 — Test coverage gaps (post-publish)

From the test-coverage agent's P0/P1 prioritisation:

### P3.1 — UNIT-TEST the new MTProxy code paths

- `MtProxyRuntime.SetActiveProxy` / `ClearActiveProxy` / `IsActive` — round-trip + rejection cases (wrong secret length, mode=2 without SNI, port out of range).
- `MtProxyRuntimeSink.SetReconnectHook` — verify the sink invokes the hook exactly once per apply/clear.
- `MtProtoChannel.OpenWithDcAsync` / `OpenWithDcAndPoolSizeAsync` — extend `MtProtoSessionPoolSmokeTest` with a DC-aware variant.
- `LocalSettingsPreferencesStore` — write/read/clear round-trip against `ApplicationData.LocalSettings`.
- `ProxyBootstrap.LoadAndApply` — feed a serialised `ProxyConfig` and verify the sink got applied.
- `AuthKeyGeneratorAdapter.GenerateForDcAsync` — covered indirectly through `LiveAuthKeyCache` only for DC #2.
- `MtProtoChannelAdapter.ReopenAfterProxyChangeAsync` — stub the channel port and assert reopen invoked.

### P3.2 — Live-proxy E2E test

`MtProxyProbe.ProbeAsync` and `MtProxyTcpTransport` only have indirect
coverage. Park behind a live-test gate analogous to `LiveAuthKeyCache`.
Provision a known-good MTProxy endpoint for the test pipeline.

### P3.3 — Mirror `mt_proxy_contract.c` invariants in C#

Add a managed smoke unit that exercises `Vianium.MtProxy.Api.V1`
through the C# WinRT projection — same invariants the C test pins,
but executed through the .winmd.

### P3.4 — Promote `MtProtoSelfTest.RunExpensive` + `CryptoSelfTest.RunExpensive`

Both are explicitly deferred ("after bignum performance work"). Promote
once vianium-crypto bignum perf lands.

---

## P4 — Documentation polish (post-publish)

### P4.1 — Update `MIGRATION_PLAN.md` version history

The document caps at v0.4.0 (19 repos). Reality is 20 repos with
vianium-mtproxy added. Bump to v0.5.0 with a "Phase 3.7 — vianium-mtproxy"
entry in the doc-history table and decisions log.

(The Tier 2 list + tree diagram already updated 2026-05-19.)

### P4.2 — Add CI-build instructions for self-hosted WP 8.1 runner

The current `.github/workflows/ci.yml` only does lint. A real native
build would need a self-hosted Windows runner with VS 2015 + WP 8.1
SDK. Document the runner provisioning in
`vianium-docs/getting-started.md` so a future contributor can set it up.

### P4.3 — Add a unified ARCHITECTURE.md to vianium-docs

Each repo has its own. The org would benefit from one overview that
spans all four tiers.

### P4.4 — Add `vianium-docs/contribution-guide.md` external-contributor walkthrough

Currently only `CONTRIBUTING.md` per repo + DCO sign-off note. A
walkthrough showing "your first PR" would lower the bar for outside
contributors when the org goes public.

---

## P5 — GitHub-side manual steps (only you can do)

### P5.1 — Pre-flight gate

```powershell
pwsh D:\Projects\2026\VianiumProject\tools\Validate-Org.ps1
```
Must exit with `Org is publish-ready`. Re-run after every fix above.

### P5.2 — Follow `vianium-docs/PUBLISH-CHECKLIST.md`

The checklist walks step-by-step: create org → push meta repos →
foundation → domain → products → tag v0.1.0 → archive legacy →
publish announcement → enable GitHub Sponsors.

### P5.3 — Verify pre-push that `TelegramAppConfig.Local.cs` is gitignored

```bash
cd D:\Projects\2026\VianiumProject\vianigram
git check-ignore -v Core/Vianigram.Composition/Configuration/TelegramAppConfig.Local.cs
git ls-files Core/Vianigram.Composition/Configuration/TelegramAppConfig.Local.cs
```
The first must print the matching .gitignore rule; the second must
produce no output.

### P5.4 — Enable GitHub Sponsors at https://github.com/sponsors/vianium/dashboard

Existing `FUNDING.yml` across all 20 repos already points at this
profile. Once Sponsors is live, every Sponsor button across the org
becomes clickable simultaneously.

### P5.5 — Confirm Buy Me a Coffee handle

Verify https://www.buymeacoffee.com/soyangelcareaga resolves. If the
handle is different, update `templates/FUNDING.yml.template` and re-run
`tools\Apply-CommunityFiles.ps1 -Force` to propagate.

### P5.6 — Archive the 4 legacy repos on GitHub

After pushing the new org repos, archive these 4 from
**Settings → Danger Zone → Archive this repository**:
- D:\Projects\2026\WP\VianiumBrowser (DEPRECATED.md already written)
- D:\Projects\2026\WP\Vianigram (DEPRECATED.md already written)
- D:\Projects\2026\WP\VianiumMusic (DEPRECATED.md already written)
- D:\Projects\2026\WP\PivoraLocalSend (DEPRECATED.md already written)

The DEPRECATED.md files were committed locally (no push from AI); push
them before archiving so visitors land on the redirect notice.

### P5.7 — Publish the announcement

Choose ONE of:
1. As a **Discussion** on `vianium/.github` (most visible, low friction).
2. As a **Release note** attached to `v0.1.0` of `vianium-docs`.
3. **Posted to angelcareaga.com** + cross-linked from the org profile.

Source draft: `vianium-docs/announcement.md`.

---

## Inventory of what's complete and verified

For reference (so you know what NOT to redo):

- 20 repos with full community-file stack (LICENSE, NOTICE, README,
  CONTRIBUTING, CODE_OF_CONDUCT, SECURITY, SUPPORT, FUNDING.yml,
  CHANGELOG, CODEOWNERS, ISSUE_TEMPLATE, PR template, ROADMAP,
  TRADEMARKS where applicable, workflows/ci.yml, .gitignore,
  .gitattributes, .editorconfig).
- 20 READMEs with shields.io badge row + "Support this project"
  section bound to GitHub Sponsors / Buy Me a Coffee / angelcareaga.com.
- Sponsors / donations central transparency page at
  `vianium-docs/donations.md`.
- Announcement draft + archive-notice template at
  `vianium-docs/announcement.md` + `archive-notice.template.md`.
- DEPRECATED.md placed in all 4 legacy `D:\Projects\2026\WP\*` repos.
- PUBLISH-CHECKLIST.md walks the manual git/gh sequence.
- MTProxy integration end-to-end (vianium-mtproxy library + native
  transport in vianium-mtproto + managed `ISettingsApi` + UI + smoke
  tests + tg:// URI activation + live handshake probe).
- 5 native-build ref-graph fixes in vianium-mtproto (preexisting bug
  surfaced by the audit, now resolved).
- SPDX header coverage 100% across all first-party source (every
  file carries the canonical `2026 Angel Careaga <hello@angelcareaga.com>`).
- `tools/`: `Apply-CommunityFiles.ps1`, `Add-SponsorsSection.ps1`,
  `Add-LicenseBadge.ps1`, `Validate-Org.ps1`, `Bootstrap-VianiumRepo.ps1`,
  `Add-SpdxHeaders.ps1` — all parse-clean.
- Solution-file integrity: all 4 .sln files (Vianigram, VianiumBrowser,
  VianiumMusic, Vianium.LocalSend) internally consistent.
- ProjectGuid uniqueness across all .vcxproj (no native-side duplicates).
- 4 critical refs newly repaired today: Vianium.Core.Tls.WinRT.vcxproj
  XML tail; Vianium.Core.Tls.vcxproj crypto-GUID mismatch;
  Vianium.Core.Innertube.vcxproj missing-GUID set; vianium-tls README
  XML tail.

---

**Angel Careaga** · [angelcareaga.com](https://angelcareaga.com) · [@AngelCareaga](https://github.com/AngelCareaga) · `hello@angelcareaga.com`
