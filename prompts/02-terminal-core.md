# 02 · Terminal Core

**Prerequisite:** scaffold from prompt 01 is in place and builds.

---

```
Build the terminal pane grid. This is the single most load-bearing
feature — the app must feel like a real terminal. See
openclide/docs/features/terminal-panes.md for the behavioral spec.

Technology decision:

  - For the PTY emulator, add SwiftTerm via SPM
    (https://github.com/migueldeicaza/SwiftTerm). It is MIT-licensed,
    actively maintained, and the only mature AppKit PTY renderer.
    Use its `LocalProcessTerminalView` as the pane's inner view.
  - If you believe another library is a better fit for my target
    (Alacritty via FFI, a Metal-backed fork, etc.) argue for it first
    and wait for my green light. Default is SwiftTerm.

Implement:

  1. WorkspaceManager singleton
     - currentWorkspace: URL? (published)
     - recents: [URL] (persisted)
     - Emits a Notification.Name.workspaceDidChange on change.
     - Persists `currentWorkspace` as a security-scoped bookmark so
       relaunches work cleanly on sandboxed builds.

  2. SidebarViewController + FileTreeView
     - A workspace picker at the top (click to show NSOpenPanel).
     - A SwiftUI file tree below, lazily loading subfolders on expand.
     - Hide .DS_Store unconditionally. Show everything else.
     - Min width 220, max 420.

  3. TerminalContainerViewController
     - Grid of terminal panes, sized 1×1 / 1×2 / 2×1 / 2×2 / 2×3 / 3×2.
     - Toolbar picker (NSToolbar segmented control) selects grid shape.
     - Each pane wraps a SwiftTerm view + a custom title bar NSView
       above it with: title text (editable), zoom button (maximize
       in-place), close button.
     - Double-click the title bar to zoom / restore.
     - Drag a title bar onto another to reorder. Use NSPasteboard
       type "public.data" carrying the pane's UUID.
     - Close pane → remove from array, spawn a fresh shell, append,
       repack into gridRows × gridCols so there's no empty slot.

  4. PTYProcess
     - Owns the PTY file descriptor + the shell child.
     - Lets you resize the PTY (TIOCSWINSZ) so vim/htop behave.
     - Exposes read/write async streams. (If using
       LocalProcessTerminalView from SwiftTerm, this is mostly
       handled for you; you're writing the glue that starts it in
       the right cwd with the user's SHELL.)

  5. Integrate with WorkspaceManager:
     - Existing panes are left alone on workspace change.
     - New panes spawn with cwd = WorkspaceManager.currentWorkspace.

  6. Main window:
     - NSSplitViewController with three items, left-to-right:
       [Sidebar, TerminalContainer, (AgentPanel placeholder)]
     - The agent placeholder is a blank NSView with a label "Agent
       panel — coming in phase 03". We'll fill it in the next prompt.

Acceptance:
  - Build + run.
  - Click the empty-state CTA, pick a folder → file tree populates.
  - Terminal pane runs the user's default shell, in that folder,
    with a real TTY (test: `tty`, `echo $SHELL`, `vim`, resize).
  - Change grid shape → panes reflow; surviving panes keep their
    scrollback.
  - Drag pane A onto pane B's title bar → they swap positions; both
    keep running.
  - Close a pane → a fresh shell appears in the last slot.
```

---

## What the agent will get wrong if you're not watching

- Spawning the shell with `/bin/zsh` hard-coded. Use
  `ProcessInfo.processInfo.environment["SHELL"]` with `/bin/zsh` as
  the fallback.
- Forgetting to send `TIOCSWINSZ` on resize → full-screen apps look
  broken.
- Recreating the PTY process during drag-reorder → scrollback is
  lost. The emulator view must be reparented, not rebuilt.
- Using `NSStackView` with equal distribution — it works until the
  user drag-resizes, then panes fight. Use `NSGridView` or manual
  frame math with `autoresizingMask`.
