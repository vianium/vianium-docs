# Vianium Browser MVP — Release Notes

## What Works (Validated)

- Loading and rendering of plain HTML/CSS sites from Tier 0-1 in `docs/TEST_PAGES.md`.
- Native HTTP through `Vianium.Core.Net` and managed `INetApi`, including TLS metadata propagation and POST support.
- Native render pipeline: Dom parse, Css cascade, Layout tree, Render to XAML-backed surface.
- PageOrchestration owns navigation and coordinates network, rendering, script execution, and fallback UX.
- Click-to-navigate on links through the native render bridge.
- Vertical scroll through the XAML host.
- Image loading for `<img>` via the native/render image path and bitmap caching.
- Basic JS execution behind `javascript.use_native_js`, including host objects for Window, Location, History, Navigator, and Console.
- DOM bindings for common MVP interactions: `document.getElementById`, `querySelector`, text access, and `addEventListener`.
- Managed bounded contexts: Tabs, Bookmarks, History, Downloads, Settings, and Credentials.
- Native bounded contexts in the MVP path: Net, JsRt, Dom, Css, Layout, and Render.
- Explicit "Coming soon" fallback for pages that require unsupported MVP features.

## Known Limitations (Phase Beta / Completo Scope)

- Forms (`input`, `select`, `textarea`, submit navigation) are deferred to Beta.
- `Promise`, `setTimeout`, `setInterval`, and broader async JS behavior are deferred to Beta Session 28.
- `async` / `await` syntax support is deferred until after the timer/microtask foundation.
- Heavy SPAs are expected to hit the "Coming soon" fallback until JS compatibility expands.
- Service Workers, Cache API, IndexedDB, and offline/PWA behavior are Phase Completo scope.
- WebAssembly is Phase Completo scope.
- Manifest V3 extensions and extension sandboxing are Phase Completo scope.
- Advanced anti-fingerprinting is Phase Completo scope.
- DevTools are Phase Completo scope.

## Architecture Milestones

- Native engine path is now the product path, not a demo path.
- `Vianium.Browser.Web` was eliminated from the repo/build, removing about 64k LOC of legacy Web implementation.
- `VianiumHttpClient.cs` and `HttpUrl.cs` were removed; the browser uses `INetApi`.
- BrowserPage was reduced to a thin XAML host that delegates navigation to `IPageNavigator`.
- `IPageRenderingHost` late-binds the transient XAML page to singleton PageOrchestration adapters safely.
- The emergency rollback setting remains available as `emergency.force_legacy_pipeline`.
- C ABI discipline and bounded-context seams are preserved for the future Rust/multi-platform port.
- The fallback model is productized: unsupported features produce clear UX instead of blank canvas or crash.

## Stats

- Total sessions completed: 31.
- Approximate legacy LOC eliminated: 64,000+.
- Approximate current repo size: 186,000 LOC.
- MVP bounded contexts shipped: 12 total, split across managed and native layers.
- Strategic/tactical audits accumulated: 21+ docs covering MVP, Beta, Completo, and post-Completo planning.
- Smoke baseline: roughly 75 smokes in the emulator/build baseline; manual visual QA is tracked in `docs/MVP_MANUAL_QA_CHECKLIST.md`.

## What Is Next

- Session 28 starts Phase Beta with JsRt 5b: `Promise`, `setTimeout`, `setInterval`, and microtask/macrotask plumbing.
- Phase Beta target: top-100 sites load decently, forms work, and common JS patterns stop falling back.
- Phase Completo target: Service Workers, IndexedDB, Wasm, extensions, DevTools, anti-fingerprinting, and broader feature parity.
- Phase 4 target: Rust port plus native shells for macOS, Windows desktop, Linux, iOS, and Android.
- Phase 5 target: Vianium Runtime and `.vapp` packaging on top of the shared engine.

See `docs/VIANIUM_DIFFERENTIATORS.md` for the product strategy and `docs/SMOKE_CATALOG.md` for the validation surface.
