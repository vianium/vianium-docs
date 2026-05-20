# Real-Time Events & Notifications — Architecture Analysis

> **Audience**: engineers extending the push pipeline (Sync, Messages, Calls, Notifications) or wiring new bounded contexts to react to push updates.
>
> **Last reviewed**: 2026-05-04 (snapshot before the Tasks-1-through-8 refactor).
>
> This document maps the end-to-end path a Telegram push update takes from the wire to a XAML bubble, and catalogs the gaps that produced visible UX bugs (no toast when backgrounded, full ChatPage reloads on every message, missing typing indicators, etc.).

---

## 1. The 7-layer pipeline

```
[1] Telegram server (MTProto wire)
       ↓ TCP encrypted frame
[2] MtProtoChannel (native C++)             ← receive loop on a native thread
       ↓ WinRT delegate MtProtoUpdateHandler
[3] MtProtoUpdatesAdapter (managed bridge)  ← async fan-out → thread-pool
       ↓ Subscribe(byte[] handler)
[4] IUpdatesPort                            ← Sync.Ports.Outbound
       ↓ subscribers
[5] Per-context update processors           ← decode TL → emit IDomainEvent
       ↓ IEventBus.Publish<T>
[6] Bounded-context applications            ← raise CLR events
       ↓ EventHandler<T>
[7] ViewModels (Dispatch.OnUiAsync) → XAML bindings
```

Threading invariants:

- **Layer 2** runs on the native receive thread; never block here.
- **Layer 3** awaits handler tasks, which crosses into the **thread pool**.
- **Layers 4–6** all run on whatever thread the publisher lands on — currently the thread pool.
- **Layer 7** is the only place that touches XAML; marshaling to `CoreDispatcher` is the subscriber's responsibility (see Task 7 — centralized via `IUiDispatcher`).

---

## 2. Layer-by-layer map

### Layer 1 — Wire

Telegram MTProto delivers updates as TL-serialized envelopes inside encrypted frames. The high-level containers are:

| Constructor | Hex | Notes |
|---|---|---|
| `updates` | `0x74ae4240` | Vector of Update + users + chats + date + seq |
| `updatesCombined` | `0x725b04c3` | Same with seq_start range |
| `updateShort` | `0x78d4dec1` | Single Update + date |
| `updateShortMessage` | `0x313bc7f8` | Compressed 1-on-1 message |
| `updateShortChatMessage` | `0x4d6deea5` | Compressed chat message |
| `updateShortSentMessage` | `0x9015e101` | RPC echo (skip in push path) |
| `updatesTooLong` | `0xe317af7e` | Server signal: call `updates.getDifference` |

### Layer 2 — Native channel

**File**: sibling `vianium-mtproto/src/mtproto/api/v1/mtproto_api.h`

- `int64 MtProtoChannel::Subscribe(MtProtoUpdateHandler^ handler)` — registers a callback fired on the native receive thread.
- `void Unsubscribe(int64 token)` — removes a registration.
- The channel itself does not parse update TL beyond the envelope; raw bytes are surfaced as `MtProtoUpdate.Bytes`.

### Layer 3 — Managed bridge

**File**: `Core/Vianigram.Composition/Infrastructure/MtProtoUpdatesAdapter.cs`

Responsibilities:

1. Register a single native handler (line 62).
2. On each push:
   - Log the first 20 ctors for diagnostics (line 95).
   - Hydrate the shared `IPeerCache` from the `users:Vector<User>` and `chats:Vector<Chat>` slices (Phase 4.1, lines 100–120). Best-effort — failures don't block dispatch.
   - Snapshot subscribers under a lock (line 124).
   - Fire-and-forget `await handler(bytes)` for each subscriber (line 138). This crosses from the native thread to the thread pool.
3. Per-handler exceptions are swallowed (line 147) so one failing subscriber doesn't poison the chain.

### Layer 4 — `IUpdatesPort`

**File**: `Core/Vianigram.Sync/Ports/Outbound/IUpdatesPort.cs`

```csharp
public interface IUpdatesPort
{
    IDisposable Subscribe(Func<byte[], Task> handler);
}
```

Implementations:

- `MtProtoUpdatesAdapter` (real)
- `StubUpdatesPort` (no-op, anonymous boot)
- `MtProtoChannelUpdatesPort` (Phase 2.2 wrapper)

**Principle M6** (`docs/managed-architecture/principles.md`): _"Only the Sync context subscribes to `IUpdatesPort`; everyone else listens to typed domain events."_ — currently violated by `MessagesUpdatesProcessor` and `CallsUpdatesProcessor`, which subscribe directly. Task 6 below restores the invariant.

### Layer 5 — Per-context processors

#### `MessagesUpdatesProcessor`

**File**: `Core/Vianigram.Composition/Infrastructure/MessagesUpdatesProcessor.cs`

Subscribed via `_subscription = updates.Subscribe(OnUpdateAsync)` (line 91).

Decodes:

- **Containers**: `updates`, `updatesCombined`, `updateShort`, `updateShortMessage`, `updateShortChatMessage`, `updateShortSentMessage` (skipped), `updatesTooLong` (logged, dropped — Sync's `getDifference` recovers).
- **Update variants**: `updateNewMessage` / `updateNewChannelMessage` → `MessageReceived`; `updateEditMessage` / `updateEditChannelMessage` → `MessageEdited`; `updateDeleteMessages` / `updateDeleteChannelMessages` → `MessageDeleted`.
- **Inner peers**: `peerUser`, `peerChat`, `peerChannel`.
- **Message types**: `message`, `messageService` (3 layer variants), `messageEmpty`.

Important: the processor extracts only **`(peerKey, messageId, timestamp)`** — never decodes message body, media, entities, etc. ChatPage compensates by reloading the entire conversation page.

Other update types (typing, status, reactions, polls, read state, sticker packs, themes, langpack, secret chats) are **silently dropped** at line ~235.

#### `CallsUpdatesProcessor`

**File**: `Core/Vianigram.Composition/Infrastructure/CallsUpdatesProcessor.cs`

Decodes `updatePhoneCall#ab0f6b1e` and 7 `phoneCall*` variants. Passes the inner `phoneCall` bytes to `CallsApplication.UpdateHandler.HandleAsync(UpdateCallStateCommand)`.

Brittleness: scans every 4-byte boundary looking for the `updatePhoneCall` ctor (line 68). Misaligned containers fall through silently.

#### Processors that don't exist (current gaps)

No processor decodes any of these — every event listed below is silently dropped today:

| Update | Effect on UX |
|---|---|
| `updateUserStatus` | "online now / last seen X" frozen |
| `updateUserTyping` / `updateChatUserTyping` | Typing indicator never appears |
| `updateMessageReactions` | Reactions don't appear in real time |
| `updateReadHistoryInbox` / `Outbox` / `Channel` | Unread badge stale, "✓✓" delayed |
| `updateNotifySettings` | Mute changes from another device invisible |
| `updateChatParticipants` / `Add` / `Delete` / `Admin` | Group membership stale |
| `updateNewEncryptedMessage` | Secret chats receive nothing in real time |
| `updateStickerSets` / `updateRecentStickers` | Sticker library doesn't refresh |
| `updatePeerBlocked` | Blocked-list out of sync |

### Layer 6 — `IEventBus`

**Files**:

- `Core/Vianigram.Kernel/Events/IEventBus.cs`
- `Core/Vianigram.Kernel/Events/InMemoryEventBus.cs`

Contract:

```csharp
void Publish<TEvent>(TEvent e) where TEvent : IDomainEvent;
IDisposable Subscribe<TEvent>(Action<TEvent> handler) where TEvent : IDomainEvent;
```

Behavior:

- **Synchronous delivery on the publisher's thread** — no marshaling, no queue.
- Handlers snapshotted under a lock; exceptions per-subscriber are swallowed.
- Singleton instance shared by every bounded context.

### Layer 7 — ViewModels

`ChatPageViewModel.OnMessagesChanged` (lines 167–178) shows the canonical subscriber pattern:

```csharp
private void OnMessagesChanged(object sender, MessagesChangedEventArgs args)
{
    if (!string.Equals(args.PeerKey, _peerKey, StringComparison.Ordinal)) return;
    var ignore = Dispatch.OnUiAsync(
        () => { var loadIgnore = ReloadAsync(CancellationToken.None); });
}
```

Three observations:

1. **Filter by peer** — bus is global, only the active peer's VM reacts.
2. **Reload, not append** — `ReloadAsync` re-fetches the entire history. For long conversations this produces a visible flash and unnecessary work.
3. **`Dispatch.OnUiAsync`** — every VM has to remember to marshal. Forgetting it crashes the page. Task 7 centralizes this.

---

## 3. The Sync engine in parallel

**File**: `Core/Vianigram.Sync/Application/SyncApplication.cs`

Sync maintains the canonical `(pts, qts, seq, date)` cursor used to detect missed updates and recover via `updates.getDifference` / `updates.getChannelDifference`.

Lifecycle:

1. **Boot bootstrap** — `App.OnUserLoggedIn` → `MigrateMainChannelToHomeDcThenBootstrapSync` → `Sync.BootstrapAsync`:
   - `updates.getState` reads the server's current cursor.
   - Diff against persisted cursor; gap → `updates.getDifference(pts, date, qts)`.
   - Persist the new cursor.
2. **Push processing** — Sync subscribes to `IUpdatesPort`. Every push goes through a `SemaphoreSlim _applyGate` (single-dispatcher invariant), then `ProcessUpdatesHandler.HandleAsync`:
   - Decode envelope.
   - `SyncState.Apply(envelope)` returns `Applied` / `NeedsGetDifference` / `NeedsChannelDifference`.
   - Persist cursor.
   - Publish derived domain events.
3. **Recovery** — gap detection triggers a separate `getDifference` RPC before further pushes are applied.

The architectural problem: because `MessagesUpdatesProcessor` and `CallsUpdatesProcessor` _also_ subscribe to `IUpdatesPort` directly, their events can be published _before_ Sync applies the cursor. In the gap scenario this produces visible reordering (mid-conversation messages appearing out of order, although `_messages` keyed by message_id usually masks it).

---

## 4. Notifications & lifecycle

### Foreground

`Core/Vianigram.Notifications/Ports/Inbound/INotificationsApi.cs` exposes mute/unmute and per-chat settings. It does **not** subscribe to `MessageReceived` — the contract is purely a settings store. There is no foreground toast system; the user just sees the new message in ChatList / ChatPage.

### Backgrounded / suspended

`App.OnSuspending` (App.xaml.cs:420-432) only logs and completes the deferral. The native channel is not closed but the OS suspends process threads. There is no `RawNotificationTask` registered, no MPNS channel URI obtained, and no call to `account.registerDevice`. Result: while the app is closed or backgrounded, **the user receives no notifications, no vibration, and no incoming-call ring**. They see the missed messages only on next launch when Sync's `getDifference` runs.

This is the single most-visible UX gap and is addressed by Task 1.

### Resume

`App.OnLaunched` re-fires:

- `ScheduleMainChannelWarmup()` reopens the main channel against the home DC.
- `ScheduleDeferredSyncBootstrap()` runs `Sync.BootstrapAsync` again to reconcile the cursor.

So nothing is lost permanently — but real-time push during the suspended window is silent.

---

## 5. Catalog of gaps (in priority order)

The following list is the seed for Tasks 1–8. They're ordered by user-visible impact.

1. **Background notifications**: zero. Without MPNS + `RawNotificationTask`, the app is invisible when closed. (Task 1)
2. **ChatPage full reload on every message**: visible flash, wasteful in long conversations. Should append the new bubble in place. Requires full Message decoding in the processor. (Task 2)
3. **`updateUserStatus` / `updateUserTyping` not decoded**: the app feels less alive than the official client. (Task 3)
4. **`updateMessageReactions` not decoded**: reactions to your messages don't appear in real time. (Task 4)
5. **`updateReadHistoryInbox` / `Outbox` / `Channel` not decoded**: unread badge stale; "✓✓" delayed when peer reads on another device. (Task 5)
6. **Double subscription to `IUpdatesPort`** (Sync + processors): violates M6, can cause reordering on gaps. Sync should be the only direct subscriber; processors should listen to a `TelegramUpdatesApplied` event Sync raises after cursor application. (Task 6)
7. **No central UI dispatcher**: every VM marshals manually with `Dispatch.OnUiAsync`. Easy to forget; intermittent crashes. (Task 7)
8. **Hardcoded TL ctors with no schema awareness**: a Telegram layer bump breaks decoders silently. Need warn-on-unknown-ctor logging so layer drift is visible early. (Task 8)

Also listed but not in the v1 batch:

- `updateNotifySettings` propagation (mute changes from another device).
- Group membership updates (`updateChatParticipants` family).
- Secret chat real-time delivery (`updateNewEncryptedMessage`).
- Sticker / theme / langpack updates.
- Per-channel typing aggregation.

---

## 6. Implementation roadmap (Tasks 1-8 batch)

The order below respects dependencies — fundamentals first, decoder additions after, background push last.

| # | Task | Dependencies | Files touched (approx.) |
|---|---|---|---|
| 8 | Schema-aware decoders (warn-on-unknown) | none | `MessagesUpdatesProcessor.cs`, `CallsUpdatesProcessor.cs`, new `UnknownCtorTelemetry.cs` |
| 7 | `IUiDispatcher` port + WP impl | none | new `Vianigram.Kernel/Threading/IUiDispatcher.cs`, new `WindowsPhoneUiDispatcher.cs` (App), refactor `Dispatch` callers |
| 6 | Sync owns subscription, raises `TelegramUpdatesApplied` | 8 | `Sync.Application.SyncApplication`, `MessagesUpdatesProcessor`, `CallsUpdatesProcessor`, new `Sync.Domain.Events.TelegramUpdatesApplied`, composition root |
| 2 | Full Message decode + granular append | 6 | `MessagesUpdatesProcessor`, `MessagesApplication`, `ChatPageViewModel` |
| 3 | Presence (status + typing) | 6 | `MessagesUpdatesProcessor` (or new `PresenceUpdatesProcessor`), `Contacts.Domain.Events`, `ChatListPageViewModel`, `ChatPageViewModel` |
| 4 | Reactions | 6 | `MessagesUpdatesProcessor`, `Messages.Domain.Events.MessageReactionsChanged`, ChatPage bubble |
| 5 | Read state | 6 | `MessagesUpdatesProcessor`, `Messages.Domain.Events.ReadStateChanged`, ChatList unread badge, ChatPage seal |
| 1 | Background notifications (MPNS + RawNotificationTask) | none (independent) | new `Vianigram.App.BackgroundTasks/RawNotificationTask.cs`, `Package.appxmanifest`, `account.registerDevice` wire-up |

---

## 7. Threading model after the batch

Post-refactor invariants:

- **Single subscriber to `IUpdatesPort`**: only `SyncApplication`.
- **Push order**: Sync applies cursor → publishes `TelegramUpdatesApplied(rawBytes, appliedSeq)` → processors decode → publish typed events.
- **UI marshaling**: only via `IUiDispatcher.RunOnUiAsync(Action)`. Both XAML host and tests register their own implementation.
- **Background**: a `RawNotificationTask` registered in the manifest decrypts incoming pushes and emits OS-native toasts even when the app is closed.

---

## 8. Telemetry and diagnostics

Existing logs that should keep working after the refactor:

- `MtProtoUpdatesAdapter` first-20 push log.
- `MessagesUpdates` / `CallsUpdates` per-event log.
- `Account.QrLogin` end-to-end trace.

New diagnostics added by the batch:

- `Realtime.UnknownCtor`: emitted by Task 8 with `(context=Messages|Calls|...,ctor=0xXXXXXXXX,size=N)`.
- `Realtime.UiDispatch`: dropped vs delivered counter.
- `Realtime.PushReceived`: each `TelegramUpdatesApplied` with applied pts/qts/seq.
- `Notifications.Push.Registered`: MPNS channel URI obtained, `account.registerDevice` outcome.
- `Notifications.Push.Received`: each raw push decrypted by the background task, with peer + chat type.

These give us the visibility to detect schema drift, unmarshaled events, push registration failures, and message-loss conditions in production.
