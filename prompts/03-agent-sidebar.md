# 03 · Agent Sidebar (cloud provider)

**Prerequisite:** prompt 02 landed. Terminal grid is working.

---

```
Build the agent sidebar with a single provider to start: a cloud AI
backend over HTTP + SSE. The spec is
openclide/docs/features/agent-sidebar.md — read the "Surface layout",
"Core state", and "Bubble rendering" sections closely.

Implement in this order:

  1. AgentModels.swift
     - enum AgentRole: .user | .assistant
     - enum AgentProvider: .cloud (add .local in prompt 04)
     - struct AgentAttachment (id, filename, mimeType, kind, localURL?,
       data?). A from(URL:) static helper that reads the file and
       infers mime.
     - struct AgentMessage (id, role, content, attachments, thinking?,
       toolCalls, pendingPermission?, isStreaming, failed, createdAt)
     - struct ToolCallInfo (id, name, displayName?, status, arguments?,
       resultPreview?). status is an enum: pending / running / success
       / error.
     - struct PendingPermission (requestId, toolName, input).
     - struct AgentUIState (messages, sessionId, isStreaming,
       errorMessage).
     - enum AgentStreamEvent — all the event cases the UI needs
       (content, thinking, toolCallStatus, permissionRequest, done,
       error, heartbeat, ignored).

  2. JSONValue helper in Support/
     - Untyped JSON. Cases: .string / .number / .bool / .array /
       .object / .null. Codable, plus `anyValue` for re-serialization.

  3. CloudAgentService (@MainActor final class)
     - @Published var uiState: AgentUIState
     - send(message:attachments:) → kicks off SSE request, streams
       deltas into uiState.
     - newConversation() → fresh sessionId, clears transcript,
       persists empty state.
     - respondToPermission(requestId:allow:) → for now a no-op; will
       be meaningful for the local provider in prompt 04.
     - Persist transcripts per-workspace: file at
       ~/Library/Application Support/<AppName>/agent-sessions/<sha>.json
       where <sha> = sha256 of the workspace path.
     - On workspaceDidChange: flush current state synchronously under
       the old key, load the new key's file (or create empty).

     TRANSPORT LAYER: define an AgentTransport protocol with
       func stream(text, attachments) -> AsyncThrowingStream<
         AgentStreamEvent, Error>
       func cancel()
       func respondToPermission(requestId:allow:)
     Provide a CloudAgentTransport that makes a POST request to an
     endpoint the user will configure in environment/config, with an
     SSE-formatted response body. You do NOT need to know my exact
     backend protocol — ask me for the SSE event shape when you get
     there, or leave a TODO that reads "configure stream parser for
     <PRODUCT>'s SSE schema" and make the parser pluggable.

  4. AgentChatView (SwiftUI)
     - Header: provider picker (one option for now), new-conversation
       button, history button, close button.
     - Scrollback: LazyVStack of bubbles, auto-scroll to bottom.
     - Composer: multi-line text, attachments chip strip, send
       button. Cmd-Enter sends; Enter inserts newline.
     - Drag-and-drop + paste: see the spec's list of accepted UTTypes.
       Save image data to $TMPDIR/<AppName>-images/drop-<uuid>.png;
       attachments carry the URL, not the bytes.

  5. AgentMessageBubble (SwiftUI)
     - UserBubble: right-aligned, accent background, markdown.
     - AssistantBubble: subviews ThinkingSection (collapsible,
       monospace), ToolCallsSection (horizontal chip row),
       markdown body, PermissionCard (hidden unless
       pendingPermission != nil).

  6. Wire the sidebar into MainWindowController
     - Replace the agent-panel placeholder from prompt 02 with
       NSHostingView(rootView: AgentChatView(...)).
     - Expose a toolbar button that toggles the right split-view
       item's collapsed state. Filled glyph when visible, outlined
       when collapsed.

Acceptance:
  - Build + run.
  - Open the agent panel; pick a workspace; type "hi" → (will fail
    until you give me a real endpoint — leave it in a state where it
    fails gracefully with a visible error banner).
  - Drag an image onto the composer → thumbnail chip appears.
  - Cmd-Enter sends; Esc cancels in-flight stream.
  - Switch workspaces → transcript clears, switch back → transcript
    restores from disk.

You are deliberately building this BEFORE integrating a live backend
so the UI is fully exercised in isolation. Stub the transport with
a test harness that replays a recorded SSE stream from a fixture file
and keep it around for future regression tests.
```

---

## Watch for

- The agent wanting to fold attachments into the content string as
  markdown (`![img](file://...)`). Don't. Keep attachments as a
  separate array; they render as chips, not inline images.
- Over-eager auto-scroll that fights the user when they scroll up to
  read earlier output. Standard fix: only auto-scroll if the view was
  within ~40 pt of the bottom before the new message arrived.
- Persisting `isStreaming == true` rows. Skip them on save.
