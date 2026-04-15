# Feature: Loop Mode

"Run this prompt every N seconds / every cron tick" — without turning
the chat into a cron manager.

## Why it exists

Common ask patterns:

- "Check this log every 30 seconds and tell me if anything looks off."
- "Every morning at 9 AM, summarize yesterday's commits."
- "Every 15 minutes, run my test suite and report failures."

None of these justify building a real job scheduler. They all fit
inside the chat itself if we give the user a small "repeat" switch
per turn.

## UX

Below the composer there's a collapsible "Loop" row. When expanded:

```
┌───────────────────────────────────────────────────────────┐
│ Loop                                               [× off] │
├───────────────────────────────────────────────────────────┤
│  Preset: [60s] [5m] [1h] [9am daily] [custom...]            │   ← horizontal scroll
│  Interval: [ input ] [ unit: s/m/h ]  [New session each run]│
│  Next run in: 0:42                                          │
└───────────────────────────────────────────────────────────┘
```

- **Preset chips** in a horizontal scroll view so they never wrap and
  never push the composer off-screen.
- **Custom interval**: number + unit, or a cron-like expression.
- **New session each run** toggle: if on, each tick spins up a clean
  conversation; if off, each tick is appended to the same thread.
- **Next run countdown** ticks every second so users can see it's
  live.

Turning loop *on* requires a prompt in the composer. The prompt is
what gets replayed each tick.

## Data model

```
LoopConfig {
  isEnabled:           Bool
  intervalSeconds:     Int            // or a cron string
  cronExpression:      String?        // null when using plain interval
  newSessionEachRun:   Bool
  prompt:              String
  startedAt:           Date?
  lastFiredAt:         Date?
  nextFireAt:          Date?
}
```

Store one `LoopConfig` per workspace; persist alongside the agent
transcript.

## Implementation sketch

One `Timer` per active loop, rescheduled after each fire using the
user's config (not a fixed interval computed at start — if the user
edits the interval mid-loop, the *next* fire uses the new value).

On fire:
1. If `newSessionEachRun`, call the same "new conversation" path the
   user-facing `+` button uses.
2. Invoke `service.sendMessage(text: config.prompt, attachments: [])`.
3. Update `lastFiredAt` / `nextFireAt`.

If the app quits, the loop does not survive. This is deliberate for
v1 — persistent background scheduling is a whole separate product
surface (launch agents, notifications on schedule, etc.).

## Provider-specific behavior

| Provider | On "new session each run" | On same-session loop |
| --- | --- | --- |
| Cloud | New server-side conversation per tick. | Same conversation — context grows turn by turn. |
| Local | Fresh session id → CLI respawned (or tear-down + pre-warm flow if you want). | Same CLI child, same session id. Context grows turn by turn. |

For the **Code** (local) provider, the `/clear` pre-warm pattern
(tear down + spawn replacement async) means "new session each run"
doesn't impose a launch-latency penalty on the fire itself.

## Layout at narrow widths

Constraints:
- Preset row: `ScrollView(.horizontal, showsIndicators: false)`, no
  wrapping.
- Interval row: same. The "New session each run" toggle lives in its
  own options row below, not inline with the unit picker.
- Footer hint strings are `.lineLimit(1)`.

At very narrow widths (< 280 pt) the whole panel can hide its preset
row behind a disclosure; the interval field stays visible because
that's the primary control.

## Risks

- **Runaway API cost**: a user sets interval=1 minute and leaves the
  machine overnight. Mitigations:
  - Never fire more frequently than 30 s.
  - Show the accumulated fire count in the header; a high count is
    visible at a glance.
  - The cloud provider should report its own usage — we don't try to
    re-implement that client-side.
- **Forgotten loops**: if the sidebar is collapsed, the loop is still
  running. Show a small "LOOP ON" indicator in the toolbar's agent
  icon (e.g. a pulsing dot) so it's never invisible.

## What this is not

- Not a cron replacement. If a user wants "at 3 AM every Tuesday",
  they can set it — but the app has to be open. It's a foreground
  feature.
- Not a workflow engine. One prompt, one interval. Chains of steps
  belong in the agent's prompt, not in the loop config.
