# Feature: Terminal Panes

## What the user sees

A rectangular area filled with one or more live terminal emulators,
arranged in a grid. Each pane has its own PTY, its own shell, its own
scrollback. A small title bar above each pane shows the pane name
(editable) and three controls: zoom (maximize in-place), edit (rename),
close (kill pane + repack the grid).

The user can:
- Pick a grid shape from the toolbar (1×1, 1×2, 2×1, 2×2, 2×3, 3×2).
- Drag a pane's title bar onto another to reorder.
- Double-click a title bar to zoom the pane full-container; double-click
  again to restore.
- Close any pane; survivors shift forward and a fresh shell fills the
  vacated slot (grid size stays constant).

## Data model

```
TerminalPane {
  id: UUID
  title: String       // editable; defaults to "Shell 1" / "Shell 2" etc.
  pty: PTYProcess     // owns the shell subprocess + emulator state
  cwd: URL            // starting cwd; shell may `cd` from there
}

TerminalContainer {
  panes:      [TerminalPane]    // row-major order
  gridRows:   Int (1...3)
  gridCols:   Int (1...3)
  zoomedId:   UUID?             // which pane is in zoom mode, if any
}
```

Pane count = `gridRows * gridCols`. When the user changes grid shape,
panes are retained if shrinking (extras are killed from the tail) and
new shells are spawned if growing.

## Layout mechanics

Use an `NSGridView` or a stack of `NSStackView` rows. Each pane is an
`NSView` containing the terminal emulator and the title bar.

Zoom mode:
- Hide all other panes.
- The zoomed pane fills the container.
- Restore by swapping visibility back.

Resizing:
- Each pane gets `1.0 / columns` of the container width and
  `1.0 / rows` of its height.
- Row heights can diverge if a user has drag-resized — store ratios if
  you want this; skip it for v1.

## Drag-to-reorder

The title bar advertises a `public.data` paste type carrying the
pane's UUID. Every pane's title bar is a drop target. On drop:

1. Remove the source pane from `panes` at its old index.
2. Insert it at the target index.
3. Rebuild the grid view.
4. Refocus the dropped pane.

Important: **do not** recreate PTYs during a reorder. The shell's
process + scrollback must survive. Reparent the emulator `NSView` into
the new grid position.

## Close + repack

```
before: [A, B, C, D]   grid 2x2
user closes B →
after:  [A, C, D, E*]  where E* is a fresh shell
```

Rationale: users who picked "2×2" want 2×2. An empty slot looks like
a bug. Auto-filling with a new shell keeps the invariant while not
losing any running work (A, C, D keep running).

## PTY sizing

Each pane computes its cell dimensions from its own bounds and the
terminal font. On resize, send `TIOCSWINSZ` to the PTY so programs
like `vim`, `htop`, `tmux` see the right dimensions. Debounce resize
events ~50 ms — continuous drag produces lots of events.

## cwd policy

- When the workspace changes via the file tree, **existing panes stay
  where they are** (the user's running commands shouldn't `cd` out
  from under them). Optionally, we show a small toast inviting the
  user to open a fresh pane in the new cwd.
- A newly spawned pane starts in the current workspace cwd.

## Title bar details

- Editable title: click the title text or the "edit" icon; a text
  field replaces it; Enter commits, Esc reverts. The name is
  user-facing only — no program consumes it.
- Close: confirms with a dim overlay for 500 ms (prevents accidental
  middle-click close in a dense grid). Optional; can be omitted for
  v1.
- Hover reveals the controls; they're dim-to-invisible when the pane
  is unfocused, full-tint when hovered or focused.

## Rough mental model

Think of the pane grid as "tmux without the user needing to learn
tmux." Grid shape > arbitrary tiling — fewer decisions for the user,
easier visual parsing.
