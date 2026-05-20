# Emulator Validation Results

Validation date: `2026-04-26`

- Emulator: `Emulator 8.1 WVGA 4 inch 512MB`
- Build tooling: `MSBuild 14.0` for managed builds, `VS2013 VsDevCmd + MSBuild 12.0` for native toolchain validation
- Deployment tooling: `Windows Phone 8.1 AppDeployCmd.exe`
- App package deployed: `Clients\Vianium.Browser.App\AppPackages\Vianium.Browser.App_1.0.0.0_x86_Debug_Test\Vianium.Browser.App_1.0.0.0_x86_Debug.appx`

## Smoke Harness Runtime

- `❌ FAIL` `Vianium.Browser.SmokeTests` is not deployable in its current shape.
  It builds as a WP8.1 class library (`OutputType=Library`) and produces no `.appx`, so there is no package for `AppDeployCmd.exe` or Visual Studio to launch in the emulator.

## Checklist Results

1. `🟡 PARTIAL` First-launch seed tab and default home URL.
   The app launched successfully on the emulator, but this pass did not clear app data first, so a true first-install reset was not exercised.

2. `✅ PASS` Open `https://example.com` from the address bar in a normal tab.
   Navigation succeeded and the page rendered correctly in the emulator.

3. `🟡 PARTIAL` Open a second normal tab and verify it appears in the tab switcher.
   The tab switcher opened and the tab badge/state updated correctly, but the automated pass did not complete a full second normal-tab creation flow.

4. `🟡 PARTIAL` Close a tab and restore it from the recently-closed flow.
   The automated pass did not drive the close/restore UI sequence end to end.

5. `🟡 PARTIAL` Open an incognito tab, navigate, and verify the visit is excluded from History.
   The automated pass opened an additional tab, but scripted text input in the incognito flow was flaky, so the privacy invariant was not proven conclusively.

6. `✅ PASS` Bookmark the current page and verify it appears in Bookmarks.
   `Example Domain` was added successfully and showed up in the Bookmarks page with the expected URL.

7. `✅ PASS` Tap a bookmark and navigate back to the bookmarked page.
   Tapping the `Example Domain` bookmark returned to the browser and showed the expected page content again.

8. `🟡 PARTIAL` Visit three sites in normal mode and verify History groups them by day.
   The History page opened correctly and grouped visits under `Today`, but the automated pass only exercised repeated `example.com` visits instead of three distinct sites.

9. `✅ PASS` Clear history and verify the page becomes empty.
   The clear-history confirmation dialog appeared, the delete action completed, and the page showed `No history yet`.

10. `🟡 PARTIAL` Download a small file and verify it appears in Downloads.
    The automated pass did not drive a complete download initiation flow in the emulator.

11. `🟡 PARTIAL` Pause and resume a larger download.
    Not exercised in this pass because no download session was established.

12. `🟡 PARTIAL` Cancel a download before completion.
    Not exercised in this pass because no download session was established.

13. `🟡 PARTIAL` Change a persisted setting and verify it survives restart.
    The Settings UI flow was not exercised in this shell-driven pass.

14. `🟡 PARTIAL` Reset all settings and verify defaults are restored.
    The reset-all settings flow was not exercised in this shell-driven pass.

15. `🟡 PARTIAL` Suspend/resume the app and verify tabs restore correctly.
    Suspend/resume was not exercised in this pass; tab persistence still needs a human or dedicated automation host.

## Summary

- Passed: `5`
- Partial: `10`
- Failed: `0`

## Notes

- This pass was performed from a shell-driven desktop session using command-line deployment and coarse mouse/keyboard automation against the WP8.1 emulator window.
- The results are good enough to confirm that the managed architecture is launching and several key context integrations are working at runtime.
- The remaining gaps are primarily validation-harness limitations, not confirmed product regressions:
  - `Vianium.Browser.SmokeTests` needs to become a deployable app host before Task 3 can pass as designed.
  - Several checklist items still require either a human emulator pass or a dedicated UI automation harness.

## Re-validation After Blocker Fixes

Validation date: `2026-04-26`

- Emulator: `Emulator 8.1 WVGA 4 inch 512MB`
- SmokeTests package: `Clients\Vianium.Browser.SmokeTests\AppPackages\Vianium.Browser.SmokeTests_1.0.0.0_x86_Debug_Test\Vianium.Browser.SmokeTests_1.0.0.0_x86_Debug.appx`
- Browser app package: `Clients\Vianium.Browser.App\AppPackages\Vianium.Browser.App_1.0.0.0_x86_Debug_Test\Vianium.Browser.App_1.0.0.0_x86_Debug.appx`

### Blocker Closure

- `CLOSED` Native Kernel `v120_wp81` build blocker
  - `Vianium.Core.Kernel`, `Vianium.Core.WebEngine`, `Vianium.Core.Wasm`, and `Vianium.Core.ExtSandbox` all build successfully after the `hresult_mapper.h` hardening.
- `CLOSED` SmokeTests deployability blocker
  - `Vianium.Browser.SmokeTests` now builds as a packageable WP8.1 app, deploys to the emulator, and launches successfully.

### Smoke Harness Runtime

- `PASS` Deploy and launch
  - `AppDeployCmd.exe /installlaunch ... /targetdevice:7` succeeded.
- `FAIL` Runtime result
  - On-screen result: `SMOKE TESTS FAIL: ICredentialsApi smoke path failed: ICredentialsApi.RemoveAsync failed: credentials.storage_failure: Credential storage operation failed.`
- Time to first visible result:
  - Approximately `11.5s` from deploy-and-launch command start to the rendered result screen.
- Memory footprint at the end of the run:
  - `XDE` host working set observed at approximately `96-101 MB`.

### Browser App Sanity Relaunch

- `PASS` `Vianium.Browser.App` redeployed and launched successfully after the blocker fixes.
- No launch regression was observed in the browser app during this targeted re-validation pass.

### Status of the Previous 10 Partial Scenarios

The blocker fixes in this session were limited to native toolchain compatibility and SmokeTests packaging. None of the previous `PARTIAL` manual scenarios regressed, but none upgraded to `PASS` during this targeted rerun because those UI flows still require a dedicated end-to-end emulator pass.

1. `STILL PARTIAL` First-launch seed tab and default home URL.
   The browser app still launches successfully, but this rerun did not clear app data and re-drive the first-install flow end to end.

2. `STILL PARTIAL` Open a second normal tab and verify it appears in the tab switcher.
   No regression was observed in tab-related startup behavior, but this specific UI flow was not re-driven in the targeted blocker-fix pass.

3. `STILL PARTIAL` Close a tab and restore it from the recently-closed flow.
   The blocker work did not touch the restore flow, and this scenario still needs a dedicated manual rerun.

4. `STILL PARTIAL` Open an incognito tab, navigate, and verify the visit is excluded from History.
   The blocker work did not touch the History/Tabs ACL path, but this privacy flow was not re-driven end to end in the targeted pass.

5. `STILL PARTIAL` Visit three sites in normal mode and verify History groups them by day.
   The History page behavior was not re-exercised with three distinct sites during this pass.

6. `STILL PARTIAL` Download a small file and verify it appears in Downloads.
   The blocker work did not touch the Downloads UI flow, and no dedicated download session was driven in this pass.

7. `STILL PARTIAL` Exercise the download state machine through pause, resume, and cancel.
   The session did not re-drive these transport-state transitions after the blocker fixes, so the previous partial status remains unchanged.

8. `STILL PARTIAL` Change a persisted setting and verify it survives restart.
   The browser app still launches with the same settings wiring, but the manual settings persistence flow was not re-run here.

9. `STILL PARTIAL` Reset all settings and verify defaults are restored.
    Not re-run in the targeted blocker-fix pass.

10. `STILL PARTIAL` Suspend/resume the app and verify tabs restore correctly.
    Not re-run in the targeted blocker-fix pass.

### Summary After Blocker Fixes

- Previous blockers closed: `2 / 2`
- SmokeTests deploy/launch: `working`
- SmokeTests end-to-end pass state: `blocked by new Credentials runtime failure`
- Prior manual checklist partials upgraded this pass: `0`
- Prior manual checklist partials regressed this pass: `0`

## Re-validation After Credentials Storage Fix

Validation date: `2026-04-26`

- Emulator: `Emulator 8.1 WVGA 4 inch 512MB`
- SmokeTests package: `Clients\Vianium.Browser.SmokeTests\AppPackages\Vianium.Browser.SmokeTests_1.0.0.0_x86_Debug_Test\Vianium.Browser.SmokeTests_1.0.0.0_x86_Debug.appx`
- Build tooling used for the passing package: `MSBuild 14.0`

### Bug Fix Entry

- Captured exception before the fix:
  - `UnauthorizedAccessException - Access is denied. (Exception from HRESULT: 0x80070005 (E_ACCESSDENIED))`
- Failure surface:
  - `ICredentialsApi.RemoveAsync`
  - propagated as `credentials.storage_failure: Credential storage save failed: Storage write failed: UnauthorizedAccessException - Access is denied. (Exception from HRESULT: 0x80070005 (E_ACCESSDENIED))`
- Root cause:
  - `LocalFolderObjectStore<T>.WriteAsync` did not dispose `DataWriter`, leaving the credentials vault file handle open across consecutive writes in the remove-of-last-entry path.
- Fix applied:
  - Added deterministic `using` disposal for `DataReader` and `DataWriter`.
  - Improved storage logging and upstream error propagation so the underlying exception is no longer masked.

### Final Smoke Result

- `PASS`
- On-screen result: `SMOKE TESTS PASS`
- Deploy-and-launch to visible result time: approximately `11.4s`

### Expanded Empty-Collection Scenarios

All of these now pass in the deployable smoke app:

1. `PASS` Credentials remove-of-last-entry path.
   The smoke app stores two unique credentials, removes both, verifies both origins retrieve as empty, and confirms the origins list returns to baseline.

2. `PASS` Bookmarks isolated empty-folder path.
   The smoke app creates a unique folder, adds one bookmark, removes it, and verifies `ListFolderAsync(folderId)` returns an empty list.

3. `PASS` History clear-to-empty path.
   The smoke app records a unique visit, clears all history, and verifies `GetVisitsForUrlAsync(testUrl)` returns an empty list.

4. `PASS` Downloads terminal cleanup path.
   The smoke app enqueues a unique download handle, cancels it, runs terminal cleanup, and verifies the cancelled-downloads list is empty afterward.

5. `PASS` Tabs last-tab close behavior through the public API.
   The smoke app closes every current tab and verifies the public `ITabsApi` contract reseeds exactly one `about:newtab` tab after the last close.

### Summary After Credentials Fix

- SmokeTests deploys: `yes`
- SmokeTests launches: `yes`
- SmokeTests final result: `PASS`
- New regressions introduced by the fix: `0`

## Native Re-validation After JsRt And Dom Integration

Validation date: `2026-04-26`

- Emulator: `Emulator 8.1 WVGA 4 inch 512MB`
- SmokeTests package: `Clients\Vianium.Browser.SmokeTests\AppPackages\Vianium.Browser.SmokeTests_1.0.0.0_x86_Debug_Test\Vianium.Browser.SmokeTests_1.0.0.0_x86_Debug.appx`

### Native Smoke Additions

1. `PASS` `Vianium.Core.JsRt` WinRT path.
   The smoke app creates `JsRuntime`, verifies `Ping() == "pong"`, subscribes to the native `ConsoleLog` event, evaluates `console.log('smoke')`, confirms the callback payload is `"smoke"`, and disposes the runtime.

2. `PASS` `Vianium.Core.Dom` WinRT path.
   The smoke app parses a fixed HTML document through `DomDocument.Parse(...)`, verifies `Title == "Smoke Dom"`, and confirms `ElementCount >= 6`.

### Final Smoke Result

- `PASS`
- On-screen result: `SMOKE TESTS PASS`
- Deploy-and-launch command wall-clock time: approximately `12.6s`
- Emulator working set after the pass:
  - `XDE` host at approximately `101.9 MB`

### Notes

- Relative to the previous passing SmokeTests run (`~101.8 MB`), loading the new `JsRt` and `Dom` native components did not show a material working-set regression in this emulator pass.
- The manual browser checklist was not fully re-driven in this native-focused session; this re-validation specifically targeted the SmokeTests runtime plus the two new native bounded-context scenarios.

## Native Re-validation After JsRt Phase 2 And Dom Phase 2

Validation date: `2026-04-27`

- Emulator: `Emulator 8.1 WVGA 4 inch 512MB`
- Deploy tooling: `AppDeployCmd.exe /installlaunch`
- Result extraction: `ISETool.exe ts appxfolder:Local deviceindex:7 8aae0019-0000-0000-0000-000000000019`

### Smoke Result

- `PASS`
- Machine-readable result file:
  - `artifacts/smoke-isolated/IsolatedStore/smoke-result.txt`
  - content: `SMOKE TESTS PASS`

### New Native Smoke Coverage

1. `PASS` `JsRt` arithmetic + coercion path.
   The smoke app now evaluates `var x = 1 + 2; console.log(String(x));` and verifies the native event payload is `3`.

2. `PASS` `JsRt` control-flow path.
   The smoke app evaluates `if (true) { console.log('yes'); } else { console.log('no'); }` and verifies the event payload is `yes`.

3. `PASS` `Dom` adoption-agency smoke path.
   The smoke app parses `<b>1<i>2</b>3</i>` and verifies the structure string reflects the corrected split formatting structure instead of the malformed source nesting.

### Emulator Footprint

- `XDE` working set after the run: approximately `103.0 MB`
- Previous native baseline from the earlier smoke pass: `101.9 MB`
- Delta after `JsRt Phase 2 + Dom Phase 2`: approximately `+1.1 MB`

## Native Re-validation After Css Phase 1 And Layout Phase 1

Validation date: `2026-04-27`

- Emulator: `Emulator 8.1 WVGA 4 inch 512MB`
- Deploy tooling: `AppDeployCmd.exe /installlaunch`
- Result extraction: `ISETool.exe ts appxfolder:Local deviceindex:7 8aae0019-0000-0000-0000-000000000019`

### Smoke Result

- `PASS`
- Machine-readable result file:
  - `artifacts/smoke-isolated/IsolatedStore/smoke-result.txt`
  - content: `SMOKE TESTS PASS`
- Deploy-and-result wall-clock time:
  - approximately `27.6s`

### New Native Smoke Coverage

1. `PASS` `Css` cascade path.
   The smoke app parses a tiny DOM document, adds an author stylesheet, resolves `color` through the WinRT shim, and verifies `#bar -> green`, `.foo -> blue`, and `div -> red`.

2. `PASS` `Layout` block path.
   The smoke app parses a small DOM document, applies padding and paragraph margins, lays it out at `320x480`, and verifies that the target `div` resolves to width `220` and that paragraph boxes stack vertically.

### Bugs Found And Fixed During Re-validation

- `Css` WinRT bridge string lifetime bug.
  - Symptom: `CssEngine.ResolveProperty(...)` returned corrupted text in the emulator even though the native contract test passed.
  - Root cause: the shim released the computed-style handle before converting the returned UTF-8 property buffer to `Platform::String^`.
  - Fix: copy/convert the UTF-8 value first, then release the native handle.

- `Layout` smoke expectation was too rigid.
  - Symptom: the smoke app assumed an exact `html -> body -> div` chain and failed after `Dom` normalization inserted intermediate structure.
  - Fix: validate the expected box behavior by locating the `div` with resolved width `220` instead of hard-coding a fragile traversal path.

### Emulator Footprint

- `XDE` working set after the run: approximately `95.8 MB`
- Previous native baseline from the earlier smoke pass: `103.0 MB`
- Delta after `Css Phase 1 + Layout Phase 1`: approximately `-7.2 MB`

### Notes

- The lower `XDE` working set relative to the previous baseline is likely normal emulator/runtime fluctuation rather than a hard proof that the pipeline became cheaper.
- The important validation result for this phase is functional: `Css` and `Layout` both passed inside the deployable WP8.1 smoke host, not only in direct native contract executables.
