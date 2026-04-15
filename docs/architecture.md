# Architecture

## Shape

```
┌─────────────────────────────────────────────────────────────┐
│  NSWindow (titled, resizable, tabbable)                      │
│ ┌──────────────────────────────────────────────────────────┐ │
│ │  NSToolbar  [agent] [shell] [magic] [mic] [theme] ...    │ │
│ └──────────────────────────────────────────────────────────┘ │
│ ┌─────────┬─────────────────────────────────┬──────────────┐ │
│ │ Sidebar │  Terminal container             │  Agent panel │ │
│ │         │                                 │              │ │
│ │  File   │  ┌─────────┬─────────┐          │  Chat tabs:  │ │
│ │  tree   │  │ pane    │ pane    │          │  [Agent]     │ │
│ │         │  ├─────────┼─────────┤          │  [Code]      │ │
│ │         │  │ pane    │ pane    │          │              │ │
│ │         │  └─────────┴─────────┘          │  Composer    │ │
│ │         │  (grid of PTYs)                  │              │ │
│ └─────────┴─────────────────────────────────┴──────────────┘ │
│                                                              │
│  NSSplitViewController coordinates the three panes.         │
└──────────────────────────────────────────────────────────────┘
```

Each of the three split-view items can be collapsed. The user picks
which surfaces they want at any moment. Toolbar icons reflect state
(filled when visible, outlined when hidden).

## Technology choices

| Layer | Choice | Why |
| --- | --- | --- |
| Shell | **AppKit** (`NSWindowController`, `NSSplitViewController`, `NSToolbar`) | Precise control over split-view resize behavior and toolbar customization. SwiftUI can't yet express all of this without hacks. |
| Panes | **SwiftUI** hosted in `NSHostingView` | Each pane's internals (chat bubbles, file list, composer) are declarative. |
| Terminal emulator | A third-party Swift PTY renderer (we use [SwiftTerm](https://github.com/migueldeicaza/SwiftTerm)). | Battle-tested; handles Unicode, scrollback, selection. Writing your own is a 2-year project. |
| Subprocess bridge | `Foundation.Process` + `Pipe`. | No 3rd-party dependency, straightforward to reason about. |
| Persistence | `UserDefaults` for small preferences; JSON files under `~/Library/Application Support/<YourApp>/` for larger state. | Standard macOS convention. |
| Updates | Self-hosted manifest file on a CDN + in-app polling. Sparkle works too. | Whatever fits your distribution model. |

## Process model

One process owns the UI. PTYs and the local AI CLI (if used) are
children.

```
Clide (GUI)
├── PTY child #1   (user's shell)
├── PTY child #2
├── PTY child #3
└── zero-cli child (hidden; NDJSON on stdio)
```

We explicitly **do not**:
- Embed a Node runtime (users already have one; if not, guide them).
- Run an HTTP server inside the app.
- Use XPC for the AI bridge (stdio NDJSON is simpler and sufficient).

## State containers

- **`WorkspaceManager`** (singleton): the currently selected folder.
  Emits a notification on change. Every other component observes it.
- **`TerminalContainerViewController`**: owns the pane grid, pane
  ordering, pane focus, and the drag-to-reorder pipeline.
- **`AgentService`** (cloud) and **`LocalZeroAgentService`** (local):
  each owns its own `AgentUIState` — independent transcripts.
- **`AgentProviderStore`**: which of the two is selected; persisted.

## Data flow for "send a message to the Code agent"

```
 User types in composer                  SwiftUI: TextEditor binding
 Hits Cmd-Enter                          AgentChatView.send(…)
 ──────────────────────────────►         LocalZeroAgentService.sendMessage(…)
    append user bubble to uiState
    append empty assistant bubble (streaming)
    ensureStarted(cwd, sessionId)  ───►  LocalZeroAgentTransport
                                         ZeroCLIProcess.start() if not running
    send(userText / userBlocks)    ───►  child stdin (NDJSON)

 ◄────────────────────────────── child stdout (NDJSON, line-buffered)
 decode SDKOutbound
 route to AgentStreamEvent
 apply to uiState (append text, add
 tool-call chip, surface permission
 card, etc.)
```

The **entire** UI update path goes through `uiState` mutations on
`@Published` properties. SwiftUI re-renders; no ad-hoc view surgery.

## Per-workspace state

When the file tree changes `WorkspaceManager.currentWorkspace`:

1. Terminal container optionally opens a fresh pane in the new cwd
   (existing panes keep their own cwd — user moved, not kicked).
2. Local AI child is torn down; next send respawns with the new cwd.
3. Agent transcripts are keyed by `SHA256(workspacePath)`. The outgoing
   transcript is flushed to disk under its old key; the incoming
   workspace's transcript is loaded or created empty.

Cloud AI state is server-side, so workspace switches don't affect it.

## Deliberate constraints

- **Main-actor everywhere for UI-adjacent code.** No detached tasks
  mutating `uiState`. Any Foundation reader callbacks hop to the main
  queue before invoking handlers.
- **No global singletons for business state.** `WorkspaceManager`,
  `*AgentService`, `AgentProviderStore` are singletons because there
  is, by design, exactly one window's worth of state. If you want
  multi-window independent state, each window owns its own instances.
- **No MVVM ceremony.** SwiftUI views read published state directly.
  No protocol-based view-model layer. The apparent simplicity is
  intentional and has been cheap to maintain.
