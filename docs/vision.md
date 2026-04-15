# Product Vision

## One-sentence version

**A real macOS terminal whose AI copilot already knows which project
you're working on.**

## Who it's for

Developers who live in a terminal and don't want to paste paths, `cd`
into the same folder twice, or tab between Slack, VS Code, and a web
chat to get help with a shell command.

Also: non-technical people who'd benefit from an agent that can edit
files and run commands — but for whom "open a terminal, install a CLI,
log in, type `zero`" is too many steps. Clide wraps the CLI in a chat
UI; they never see the terminal if they don't want to.

## Non-negotiables

1. **It's a real terminal.** PTY, full ANSI, respects the user's shell,
   behaves indistinguishably from iTerm2 / Terminal.app when the AI
   panel is closed.
2. **The workspace is the single source of truth for cwd.** A file
   tree in the sidebar lets the user pick a folder. Every shell pane,
   every agent tab, every tool call respects it.
3. **Both agents — cloud and local — are first-class.** Cloud is the
   polished path; local (a spawned CLI) is the power path. Same chat
   UI; transport is the only thing that differs.
4. **Zero setup for the happy path.** If the user has a CLI installed,
   we find it. If they don't, we tell them exactly how to install it
   and detect when they have.
5. **Every action on shared state is reversible or confirmable.** No
   surprise file writes, no silent `git push`, no destructive tools
   without an in-UI permission prompt.

## Anti-goals

- **Not an IDE.** No LSP, no inline completions, no debugger UI. The
  terminal is the center of gravity.
- **Not a Slack replacement for the AI.** The agent sidebar is for
  one-shot help, not a log of lifetime conversations. (We do persist
  transcripts per workspace — but not per conversation-forever.)
- **Not a sandbox or a container.** The terminal runs against the
  user's real filesystem and shell. If they `rm -rf`, it's gone. This
  is a feature, not a bug — power users will not tolerate wrappers.
- **Not cross-platform on day one.** macOS-native, AppKit + SwiftUI,
  no Electron. If you want Windows or Linux, fork the design.

## What "done" looks like

A user opens the app, picks a folder, types `ls` in the shell pane and
it behaves like `ls`. They open the agent sidebar, ask "what's in this
repo", and the agent answers having actually read the files — because
the cwd is the folder they just picked. They drag an image onto the
composer, ask "what's in this screenshot", and the agent sees it.

Anything past that is polish, not vision.

## What this spec will not specify

- Your bundle ID, team ID, signing identity.
- Your backend URLs, auth scheme, rate limits.
- Your pricing, your update channel, your analytics.
- Your icon, your fonts, your accent color.

These are product decisions. We won't make them for you.
