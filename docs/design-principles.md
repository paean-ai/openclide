# Design principles

Short, non-negotiable. When in doubt, re-read these.

## 1. The terminal is the floor, not the ceiling

We're adding AI to a terminal, not a terminal to an AI. Every AI
feature must degrade gracefully to "user pretends the AI panel doesn't
exist and keeps working." If the shell pane stops being first-class,
we've lost the product.

## 2. The workspace folder is a singleton

Exactly one "current folder" at any moment. File tree, shell panes,
AI tools, and persistence keys all derive from it. Do not invent
per-surface overrides — that way lies the hell of "which cwd applies
to this tool call."

## 3. Reversible over clever

Every destructive action is either:
- Explicitly confirmed in-UI (permission card, modal, etc.), or
- Logged enough that the user can recover (session JSON on disk).

No hidden state. No silent side effects.

## 4. Defaults > settings

A setting is an admission that we couldn't pick. Each setting costs:
- Code to implement.
- UI surface in a preferences pane.
- Support burden when users configure themselves into a broken state.
- A slightly worse product for everyone who doesn't use it.

Ship the good default. Add the setting only when a specific, named
user cohort can't live without it.

## 5. AppKit where it matters, SwiftUI where it doesn't

Window chrome, split behavior, toolbar, menu bar: AppKit. Bubble
content, composer, file tree row, settings forms: SwiftUI. Mixing
them via `NSHostingView` and `NSViewRepresentable` is routine on
modern macOS — don't be a purist in either direction.

## 6. No servers inside the app

The app is a client. If a feature needs a server, it talks to a
remote one. Inside-the-app HTTP/WebSocket servers introduce port
conflicts, localhost security surface, and a deployment model Apple
doesn't love. Prefer stdio + subprocess for local AI.

## 7. Narrow widths matter

Users dock the app to half the screen. They snap panels into
overlapping zones. Every view must behave at 280 pt width — not
pretty-at-any-width, but at minimum *usable*. Overflow → horizontal
scroll; lose the label, keep the control.

## 8. Cost-aware UI

Every toolbar icon, every always-visible badge, every watching
observer has a lifetime cost: one more thing the user has to
mentally suppress. Add UI when it earns its pixels.

## 9. Don't invent a framework

When a built-in pattern exists (`NSSplitViewItem.isCollapsed`,
`NSNotification`, `@Published`), use it. Swift is *not* a language
where abstracting over abstractions pays off. The codebase stays
readable when it looks like plain Cocoa + plain SwiftUI.

## 10. One pane of glass

The user should never have to ask "where is feature X?" There's
exactly one place for each: toolbar, menu bar, right-click, or the
relevant sidebar. Cmd-K palettes are fine as a second route; they're
not a substitute for discoverable UI.

---

If your implementation starts violating one of these, stop and pick a
different approach — even if the violating approach is "simpler
right now." Debt compounds.
