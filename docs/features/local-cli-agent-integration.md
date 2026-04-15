# Feature: Local CLI Agent Integration

A generic template for wrapping any stdio-NDJSON AI coding CLI as an
in-app child process. We use this for the **Code** tab in the agent
sidebar.

Compatible with any CLI that speaks a "stream-JSON" protocol over
stdin/stdout — Anthropic's Claude Agent SDK uses this shape; so do
several other agent CLIs. Adapt the message types to your target.

## Components

```
                AgentChatView (SwiftUI)
                        │
                        ▼
             LocalAgentService  (@MainActor singleton)
                        │
                        ▼
            LocalAgentTransport  (@MainActor)
                        │
                        ▼
               CLIProcess        (Foundation.Process wrapper)
                        │  stdin / stdout NDJSON
                        ▼
                    your-cli   (installed on the user's system)
```

| Type | Role |
| --- | --- |
| `LocalAgentService` | Owns `AgentUIState`, persists transcripts, observes workspace changes. |
| `LocalAgentTransport` | Bridges SDK messages ⇄ `AgentStreamEvent`. One per service. |
| `CLIProcess` | Spawns the CLI, owns stdio pipes, emits decoded NDJSON lines on the main queue. |

All three run on the main actor.

## Launching the CLI

Typical invocation:

```
<cli-name> -p \
     --input-format  stream-json \
     --output-format stream-json \
     --verbose \
     --session-id <uuid>
```

Flags will vary per CLI. The principle: you want **JSON in both
directions** over stdio, and a **session id** you generate client-side
so you can respawn the process later and continue the same thread.

### Executable resolution

GUI-launched macOS apps inherit only launchd's minimal `PATH`, which
almost never contains the user's Node/Bun install. Resolve in order:

1. `${SHELL} -l -c 'command -v <cli>'` — a real login shell so fnm,
   volta, mise, asdf, and Homebrew shims resolve.
2. `/usr/bin/env which <cli>` — fallback when `SHELL` is unusual.
3. Hard-coded candidates: `/opt/homebrew/bin`, `/usr/local/bin`,
   `~/.local/bin`, `~/.npm-global/bin`, `~/.volta/bin`, `~/.bun/bin`.

If all three miss, show an **install onboarding sheet** with the two
or three canonical install commands (homebrew tap / npm global / curl
installer) and a "Docs" link. Cache the detection result for a short
window so repeatedly opening the sidebar doesn't hammer the shell.

### Environment

```
var env = ProcessInfo.processInfo.environment
env["NO_COLOR"]   = "1"
env["CI"]         = "1"   // suppress tty-only prompts
env["PATH"]       = loginShellPATH() ?? env["PATH"]
```

`loginShellPATH()` runs `$SHELL -l -c 'echo $PATH'` once and caches
the result. Without this, a `#!/usr/bin/env bun` shebang script
resolves itself but then can't find `bun` at exec time and dies with
a cryptic error.

### cwd

Whatever the file-tree sidebar currently points at. This is how the
agent's tools (read/write/bash/etc.) act on the user's current
project by default.

## Wire protocol

One JSON object per line. Unknown `type` values must decode as
`.unknown(type:, raw:)` — the protocol evolves; silently ignoring
new envelopes keeps you forward-compatible.

### Outbound (CLI → app)

Typical envelopes:

| `type` | Meaning |
| --- | --- |
| `system` | Session init / metadata. |
| `assistant` | Model output. Content blocks: `text`, `thinking`, `tool_use`. |
| `user` | Echoed user turns + `tool_result` content blocks. |
| `result` | End of turn. Carries `is_error` / `result` text. |
| `control_request` | Subtype `can_use_tool` — permission prompt. |
| `control_response` | Ack of our interrupt / permission reply. |
| `keep_alive` | Heartbeat. |

Content blocks:

| `type` | Meaning |
| --- | --- |
| `text` | Streamed model text. |
| `thinking` | Extended-thinking segment (may be streaming). |
| `tool_use` | Model wants to invoke a tool — has `id`, `name`, `input`. |
| `tool_result` | Result of a tool call — matches `id`, has `content`, `is_error`. |
| `image` | Model returned an image (rare). |

### Inbound (app → CLI)

```jsonc
// Plain text turn
{"type":"user","session_id":"<uuid>","message":{"role":"user","content":"hi"}}

// Rich content (attachments)
{"type":"user","session_id":"<uuid>","message":{"role":"user","content":[
  {"type":"text","text":"(attached) /tmp/clide-images/drop-<uuid>.png"},
  {"type":"text","text":"what's in this image"}
]}}

// Allow a tool
{"type":"control_response","response":{
  "subtype":"success","request_id":"<rid>",
  "response":{"behavior":"allow"}}}

// Deny (also interrupts the current turn)
{"type":"control_response","response":{
  "subtype":"success","request_id":"<rid>",
  "response":{"behavior":"deny","interrupt":true}}}

// Cancel the current turn
{"type":"control_request","request_id":"<new uuid>","request":{"subtype":"interrupt"}}
```

Attachments: pass filesystem paths only — no base64. Save
pasted/dragged images to `$TMPDIR/your-app-images/drop-<uuid>.png`;
the CLI ingests via its own file resolver.

## Stdio plumbing

The process wrapper wires three `Pipe`s and a line-buffered reader:

- A `Data` ring buffer accumulates `readabilityHandler` chunks and
  splits on `0x0A`. Parse after split — JSON lines are self-delimiting
  via newline even when embedded unicode separators are present.
- Keep the last N stderr lines in a small ring (200 is plenty). On
  non-zero termination, append the last few stderr lines to the
  error surfaced in the UI ("click retry" vs. forever-identical
  "status 1").
- Writes go through a serial `DispatchQueue` so concurrent `send(_:)`
  calls from the UI can't interleave payloads.
- Install readers **before** `process.run()` — otherwise you miss the
  CLI's first init event.

## Lifecycle rules

### Spawn

Lazy. First `sendMessage(...)` after service boot or workspace change
starts the child.

### Workspace switch

On `workspaceDidChange`:

1. Cancel in-flight stream; shutdown the child.
2. **Flush the current transcript synchronously** under the outgoing
   workspace's persistence key. The debounced save would otherwise
   lose the last edit when `uiState` is swapped.
3. Load the incoming workspace's transcript (or empty).

### User clears the conversation

Tear down the existing CLI — the session id is fixed at spawn — and
**pre-warm** a replacement asynchronously with a new session id. The
user's next send then pays no launch latency.

### User hits Stop

Send `control_request{subtype:"interrupt"}` with a fresh request id,
then finish the stream continuation so the UI leaves the streaming
state. Gate the expected non-zero `result` that follows so it doesn't
surface as an error.

### CLI exits abnormally

`process.terminationHandler` dispatches to the main queue. If
`cancelRequested == false` and `status != 0`:

```
Your-CLI exited (status 1). Click retry.

<last 5 stderr lines>
```

The next send respawns a fresh child on the same session id.

## Persistence

Per-workspace files, keyed by `SHA256(workspacePath).hex`:

```
~/Library/Application Support/<YourApp>/agent-sessions/<hash>.json
```

Workspaces with nothing selected use a literal `__no_workspace__`
bucket so empty-state chats don't bleed into the first real
workspace the user opens.

## Gotchas

- **Do not** set `process.environment` to a minimal dictionary. Inherit
  the full parent env first.
- **`proc.isRunning` is not thread-safe** relative to the termination
  handler. Maintain an `isTerminated` bool set on the main queue from
  the handler, and read that instead.
- **`readabilityHandler` is invoked on a background queue.** Hop to
  the main queue before touching `@Published` state or calling
  back into `@MainActor` code.
- **Some CLIs require both `--verbose` and stream-json I/O together.**
  Read the CLI's docs; drop `--verbose` only if it's harmless.

## What this design deliberately is not

- Not an XPC service. Stdio NDJSON is simpler, debuggable with
  `tee`, and has no mach-port permission surface.
- Not a WebSocket bridge on localhost. Users don't want a random
  port open, and we don't want the localhost security surface.
- Not a bundled Node runtime. Users either have one or they don't;
  tell them how to get one.
