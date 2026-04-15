# Feature: Workspace & File Tree

## What it does

Left sidebar shows:

1. A **workspace picker** at top — the selected folder's name, clickable
   to open an `NSOpenPanel` for picking a different folder.
2. A **file tree** below it, rooted at the workspace folder, showing
   files and directories. Click to open; double-click a file to open
   it in an external editor (or a reveal-in-Finder action).

The selected workspace is **the** cwd for the entire app: new shell
panes spawn there, the local AI agent's cwd is there, the agent's
tool calls resolve relative paths there.

## Data model

```
WorkspaceManager (singleton) {
  currentWorkspace: URL?     // published; nil until user picks
  recents:          [URL]    // most recent at index 0
}
```

State:
- `currentWorkspace` is persisted in `UserDefaults` as a bookmark
  (so the permission survives relaunches on sandboxed builds).
- `recents` drops to 10 entries, unique, most-recent-first.

Emits a `workspaceDidChange` notification whenever `currentWorkspace`
is assigned — this is the signal other components observe.

## File tree details

- Load the immediate children of the workspace folder on selection.
- Lazily load subdirectories on expand (directories with thousands
  of files can otherwise stall the UI).
- Respect `.gitignore`? Tricky — we recommend: don't, for v1.
  Filtering hides files users *want* to see (build artifacts they're
  debugging, `.env` they're editing). Users who want ignore-style
  filtering can toggle a "Show hidden files" button later.
- **Do** hide `.DS_Store` unconditionally. That file brings joy to no
  one.

Visual:
- Single-column list with 16×16 icons (folders and file types).
- Indent sub-levels by 12 pt.
- Expand/collapse chevrons on the left.
- Selected row gets a subtle highlight; active (focused) row gets the
  system accent color.

## Picking a workspace

Two paths:

1. **Empty state**: app launches with no prior workspace. Show a large
   call-to-action in the sidebar: "Pick a folder to get started."
2. **Existing workspace**: user clicks the workspace name at the top;
   an `NSOpenPanel` opens. New selection updates
   `currentWorkspace` → notification fires.

## Interaction with other surfaces

| Other surface | What happens on workspace change |
| --- | --- |
| Shell panes | Existing panes left alone; new panes spawn in new cwd. |
| Cloud agent | Unaffected; state is server-side, not cwd-dependent. |
| Local AI child | Torn down; next send respawns with new cwd. |
| Local AI transcript | Current transcript flushed under outgoing workspace's key; incoming workspace's transcript loaded (or empty). |
| Magic-summary popover | Closes if open. |

## Why a single workspace instead of many

Multi-root workspaces (VS Code style) are tempting. They compound
cognitive load:
- "Which root did the agent just write to?"
- "Which root does `cd` drop me in?"
- "Why did the agent grep across all of them?"

One folder at a time. If the user needs two, they open a second window
(native macOS tabbed windows make this cheap).

## Edge cases

- **Folder deleted externally while the app runs**: the tree should
  show a greyed-out root with an error banner, not crash. Offer a
  "pick a new folder" CTA.
- **Symlinks**: follow, but show a small arrow badge.
- **Very large folders** (~100k files): lazily load; debounce search
  queries; never sort the whole tree in memory.
- **Network drives / backup folders**: expected to be slow; don't
  freeze the UI on `attributesOfItem(atPath:)`. Move anything that
  touches the filesystem off the main queue.
