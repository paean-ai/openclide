# 04 · Local CLI Agent

**Prerequisite:** prompt 03 landed. Cloud provider's chat UI works.

---

```
Add a second provider: a local AI coding CLI spawned as a hidden
child process, streamed over stdio NDJSON. See the whole of
openclide/docs/features/local-cli-agent-integration.md before
writing any code.

Tell me which CLI you intend to target. Reasonable defaults:
  - Anthropic Claude Agent SDK CLI (the stream-json protocol in the
    spec is modeled on this).
  - Any other CLI that speaks stream-JSON on stdio.

If I haven't told you which one, ask. Don't guess my brand.

Implement:

  1. CLIProcess in Agent/Services/
     - Wraps Foundation.Process + three Pipes.
     - Resolves the executable in this order:
         a) $SHELL -l -c 'command -v <cli>'
         b) /usr/bin/env which <cli>
         c) hard-coded candidate paths
     - Cache the login-shell PATH (run $SHELL -l -c 'echo $PATH' once)
       and inject into the child's env. NO_COLOR=1, CI=1.
     - Line-buffered stdout reader (split on 0x0A, decode each line
       as SDKOutbound).
     - stderr ring buffer (200 lines, exposed as recentStderr).
     - onOutbound / onTerminate / onStderrLine callbacks, all
       dispatched to the main queue.
     - terminate(): cancel readers, close stdin, process.terminate().

  2. SDK protocol DTOs (in the same file is fine)
     - enum SDKOutbound: .system, .assistant, .user, .result,
       .controlRequest, .controlResponse, .controlCancelRequest,
       .streamEvent, .keepAlive, .unknown.
     - enum SDKContentBlock: .text, .thinking, .toolUse,
       .toolResult, .image, .unknown.
     - enum SDKInbound: .userText, .userBlocks, .permissionAllow,
       .permissionDeny, .interrupt — each serializes to the JSON
       objects shown in the spec.
     - Decoding must be lenient: unknown types → .unknown(type:, raw:).

  3. LocalAgentTransport (conforms to AgentTransport from prompt 03)
     - ensureStarted(options) → spawns if not running.
     - sendUserTurn(text:, attachmentPaths:) → returns an
       AsyncThrowingStream of AgentStreamEvent.
     - respondToPermission / cancel / shutdown.
     - route(_:) function that translates SDKOutbound → events.

  4. LocalAgentService (@MainActor singleton)
     - Same shape as CloudAgentService but uses LocalAgentTransport.
     - Per-workspace transcript persistence keyed by
       sha256(workspacePath).hex.
     - On workspaceDidChange:
         - cancel in-flight stream, shutdown child
         - flush current uiState synchronously under the OLD key
         - load new key (or empty AgentUIState)
     - newConversation(): shutdown child, set uiState to fresh,
       then pre-warm a replacement child asynchronously.
     - sendMessage: if activeCliCwd != current workspace, shutdown
       child before send (so the new child starts in the right cwd).

  5. UI touch-ups
     - AgentProvider enum now has .cloud and .local.
     - AgentProviderStore persists the selection.
     - AgentChatView's header picker becomes a two-segment control.
     - Switching providers: cancel current stream, swap the
       displayed uiState (each provider owns its own).
     - PermissionCard renders inline at the point of the request;
       Allow / Deny buttons call
       activeService.respondToPermission(...).

  6. Install onboarding
     - On first switch to the local provider, run the CLI resolver.
     - If missing, present an NSHostingView sheet with two install
       recipes (e.g. `brew install ...`, `npm install -g ...`),
       copy-to-clipboard buttons, and a "Docs" link.
     - Dismiss → re-run resolver; if still missing, user is
       gracefully stuck on cloud.
     - Cache the detection result for 5 minutes so the sheet doesn't
       flicker on every panel open.

Acceptance:
  - Build + run.
  - Switch provider to local → (if CLI installed) first message
    launches it, see assistant stream.
  - Tool-call chips appear for every tool the CLI uses; permission
    cards appear for destructive tools; allow/deny works.
  - Drag an image → it arrives as a file path in the turn; the
    agent can "see" it.
  - Workspace switch → child dies; send → child respawns with the
    new cwd. Switching back restores the prior workspace's chat.
  - Hit Stop mid-stream → interrupt goes out, UI returns to idle.
  - Kill the CLI externally → "Exited, click retry" with the last
    stderr lines. Retry respawns.
```

---

## Watch for

- `process.environment = [:]` — agents do this "for isolation" and
  then the shebang can't find `bun`. Copy `ProcessInfo.environment`
  first, then overlay.
- Decoding breaking on a new CLI message type the agent hasn't seen.
  Your lenient `.unknown` path should absorb it silently; double-check
  this with a fixture that contains an unknown type.
- Forgetting to hop `readabilityHandler` callbacks to the main queue
  before touching `@Published` state. Swift concurrency won't save
  you here — these are Foundation callbacks on a private queue.
- Persisting two transcripts in one file. Keep them in separate
  per-workspace files — simpler restore semantics.
