# 01 · Scaffold

**Prerequisite:** the agent has read the docs and produced a summary
you agree with (see `00-bootstrap.md`).

---

```
Create the Xcode project scaffold for our app. Requirements:

  - Xcode project (not Swift Package Manager at the top level, though
    SPM dependencies are fine). Name it whatever I tell you, or if I
    didn't specify, ask now and wait.
  - macOS 14+ deployment target.
  - Single app target. No framework/extension sub-targets in v1.
  - AppKit app lifecycle (NSApplicationMain via @main
    AppDelegate). We'll use SwiftUI inside NSHostingView panels — do
    not use the `App` lifecycle with `WindowGroup`, we need fine
    control over windows.

Directory layout inside the project folder:

  AppName/
    AppDelegate.swift
    MainWindowController.swift       # NSWindowController + NSToolbar
    Sidebar/
      SidebarViewController.swift
      FileTreeView.swift              # SwiftUI
      WorkspaceManager.swift          # singleton
    Terminal/
      TerminalContainerViewController.swift
      TerminalPane.swift
      PTYProcess.swift                # wraps a shell via forkpty/Process
    Agent/
      AgentModels.swift
      AgentProviderStore.swift
      AgentChatView.swift
      AgentMessageBubble.swift
      Services/
        (left empty for now — we'll add CloudAgentService and
         LocalAgentService in later prompts)
    Support/
      JSONValue.swift                 # untyped JSON helper
      Logger+Extensions.swift

Create:
  - Info.plist with the bare minimum (no ATS exceptions yet; no
    sandbox yet — I'll tell you when we need those).
  - Assets.xcassets with a default app icon placeholder.
  - A Release.xcconfig (git-ignored) template at
    `Release.xcconfig.example` listing TEAM_ID, DEVELOPMENT_TEAM, and
    CODE_SIGN_IDENTITY as build-setting placeholders. Do NOT populate
    real values — leave them as XXX.

Do NOT add any third-party dependencies yet. We'll discuss the
terminal emulator library (SwiftTerm or alternative) in prompt 02.

At the end, open the project in Xcode (via `open AppName.xcodeproj`),
build in Debug, and confirm you get an empty window titled with the
app name.

If the build fails, fix it before handing back to me.
```

---

## Pitfalls to watch for

- Agents sometimes re-bootstrap the project with the App lifecycle
  and `WindowGroup` "because it's simpler." Push back — the split
  view controller we need is much cleaner with AppKit lifecycle.
- Don't let it scaffold a tests target with `XCTest` until there's
  something worth testing. Empty test targets collect rot.
