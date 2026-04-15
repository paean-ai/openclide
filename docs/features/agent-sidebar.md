# Feature: Agent Sidebar

The right-side panel. A chat UI with two providers the user toggles
between:

- **Agent** — a cloud AI backend over HTTP+SSE. Your product's
  proprietary assistant.
- **Code** — a local AI CLI spawned as a child process, streamed over
  stdio NDJSON. See
  [`local-cli-agent-integration.md`](local-cli-agent-integration.md).

Both drive the *same* renderer. Only the transport differs.

## Surface layout

```
┌─────────────────────────────────────────┐
│ [Agent | Code]   [history] [new] [×]   │   header
├─────────────────────────────────────────┤
│                                         │
│  user bubble                            │
│                                         │
│      assistant bubble                   │
│      ├─ thinking (collapsible)          │   scrollback
│      ├─ [tool-call chip] [chip] [chip]  │
│      ├─ markdown body w/ code blocks    │
│      └─ [permission card]?              │
│                                         │
│  user bubble                            │
│  ...                                    │
│                                         │
├─────────────────────────────────────────┤
│ ┌─────────────────────────────────────┐ │
│ │ [attach][mic] text area      [send] │ │  composer
│ └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

## Core state

```
AgentUIState {
  messages:    [AgentMessage]
  sessionId:   UUID
  isStreaming: Bool
  errorMessage: String?
}

AgentMessage {
  id:        UUID
  role:      .user | .assistant
  content:   String            // markdown
  attachments: [AgentAttachment]
  thinking:  String?           // extended thinking (optional)
  toolCalls: [ToolCallInfo]    // ordered
  pendingPermission: PendingPermission?
}

ToolCallInfo {
  id, name, displayName?, status, arguments?, resultPreview?
}
```

A transport produces a stream of `AgentStreamEvent`s; the service
applies them to `uiState`:

```
.content(text)            → append to current assistant bubble
.thinking(text, partial)  → append to thinking; toggle spinner
.toolCallStatus(id, …)    → upsert chip in toolCalls
.permissionRequest(…)     → set pendingPermission → card renders
.done                     → isStreaming = false
.error(msg)               → failed + errorMessage
```

## Header controls

- **Provider picker**: two-segment `Picker` bound to an
  `AgentProviderStore`. Persist the selection.
- **History**: opens a floating overlay (ZStack over the chat) listing
  prior transcripts for this workspace. Click to resume, × to close,
  click-outside to dismiss.
- **New conversation** (`+`): clears `uiState` under the current
  workspace's persistence key. For the local provider, this also
  tears down the CLI child (its session id is fixed at spawn) and
  pre-warms a replacement so the next send has no launch latency.
- **Close the whole panel** (`×`): collapses the right split-view
  item; the toolbar's "agent" button flips to its outlined variant.

## Composer

- Multi-line text area; Cmd-Enter sends, Enter inserts a newline.
- Attach button → native file picker, adds to the attachment strip.
- Mic button → opens a voice-recorder popover that transcribes locally
  or via the cloud API, then inserts the text into the composer.
- Drag-and-drop & clipboard paste accept:
  - `public.image` — images (PNG, JPEG, HEIC, WebP, TIFF).
  - `public.file-url` — any file path.
  - `public.text` — plain text gets inserted at cursor.
- Image data is written to `$TMPDIR/clide-images/drop-<uuid>.png`; the
  attachment carries the URL, not the bytes (keeps the `AgentUIState`
  light and avoids re-encoding on save).

Placeholder text swaps per provider ("Message Agent…" vs "Ask the
local coding agent…") so the active mode is obvious at a glance.

## Bubble rendering

- **User bubble**: right-aligned, accent-colored, markdown-rendered.
  Attachment thumbnails stack above the text.
- **Assistant bubble**: left-aligned, plain background. Sub-sections:
  - `ThinkingSection` — collapsible, monospace, rendered when
    `message.thinking != nil`. Pulsing brain icon while streaming.
  - `ToolCallsSection` — horizontally scrollable chip row.
    Each chip shows tool name + status glyph (spinner / ✓ / ×).
    Click a chip to expand arguments (pretty JSON) and result preview.
  - Body — markdown → rich text; code blocks get syntax highlighting
    and a "copy" button.
  - `PermissionCard` — inline at the point of the request. Shows tool
    name, rendered input, Allow / Deny buttons. On click, calls
    `transport.respondToPermission(...)`.

## Persistence

One JSON file per workspace, keyed by `SHA256(workspacePath)`:

```
~/Library/Application Support/<YourApp>/agent-sessions/<hash>.json
```

- Saves are debounced 2 s; writes are atomic.
- Transient fields (`isStreaming`, `pendingPermission`, `errorMessage`,
  attachment `Data`) are dropped.
- Partial streaming rows are skipped on save to avoid persisting
  half-rendered turns.

Switching workspaces:
1. Cancel in-flight stream, shutdown any child.
2. Flush current `uiState` synchronously under the *outgoing*
   workspace's key.
3. Load the *incoming* workspace's file (or start empty).

Switching providers:
1. Stop the current transport's stream.
2. Swap `uiState` to the other provider's state (each provider owns
   its own).
3. Done — render.

## Narrow widths

At ≤ 340 pt:
- Hide the "Agent / Code" segmented picker label ("Agent/Code" alone
  is enough).
- Thinking section defaults collapsed.
- Tool-call chips horizontally scroll rather than wrapping (wrapping
  pushes the composer off-screen).
- The composer never drops below 48 pt height.

## Explicit non-features

- **No conversation branching / regenerate-from-here** in v1. It
  complicates persistence and rarely earns its weight.
- **No shared transcripts across workspaces** — that's what the cloud
  provider's server-side history is for.
- **No @-mentions of files** in v1. Users can drag files into the
  composer, which is equivalent and simpler.
