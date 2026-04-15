# Feature: Magic Summary

A toolbar button that summarizes the visible terminal pane's recent
output into a friendly paragraph. Useful when a long build errors out
and the user wants the gist without re-reading 400 lines of logs.

## UX

- Toolbar icon (a small sparkles / wand glyph) next to the mic.
- Click → the icon shows an activity spinner.
- A popover appears below the icon with the summary text, streaming
  as it arrives.
- Buttons in the popover: "Copy", "Open in chat" (pastes into the
  agent composer), "Regenerate".
- Background terminal output keeps scrolling normally — the summary
  is a snapshot of the last N lines at the moment of the click.

## Data flow

```
 user clicks magic                MagicSummaryService.start()
                                  ├─ snapshot: grab last 4k chars from
                                  │  the focused pane's emulator
                                  ├─ POST to your summarizer endpoint
                                  │  (SSE for streaming) OR invoke your
                                  │  local AI provider with a dedicated
                                  │  "summarize this" prompt template
                                  ├─ stream tokens into the popover
                                  └─ emit a state notification so the
                                     toolbar icon reflects in-flight
                                     state
```

## Key decisions

- **Snapshot at click time, not at send time.** If the build is still
  producing output, the user wants "tell me about what I saw", not
  "tell me about what's on screen two seconds from now".
- **The popover is stateless.** Dismissing it cancels the summary
  stream. Re-opening starts a fresh one. We do not persist summaries.
- **Toolbar progress indicator.** Even when the popover is closed,
  the toolbar icon shows an in-flight state so the user knows
  something is still happening. If they accidentally dismissed the
  popover, re-opening shows the in-progress stream.

## Content selection

- Prefer the **focused** pane. If nothing is focused, the "most
  recently active" pane (last to receive output). If still nothing,
  disable the toolbar button.
- Take the last ~4k characters (configurable). If the output is
  shorter, take all of it.
- Strip ANSI escape codes before sending to the model — they're
  noise for a summarizer.

## Tone

The summarizer's system prompt should produce 2–4 sentences,
conversational, in the user's inferred language. No markdown
headers; this is a one-shot paragraph, not a document.

Example output:

> Your `cargo build` failed because `serde_json` 1.0.120 isn't
> compatible with the Rust version your toolchain is pinned to. The
> compiler's suggestion is to upgrade to 1.0.128, which you can do
> with `cargo update -p serde_json`.

## Narrow-width behavior

Keep the toolbar icon; drop the label. The popover's width is
independent of the toolbar (it's a `NSPopover`), so a narrow window
doesn't cramp the summary.
