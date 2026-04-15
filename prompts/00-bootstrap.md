# 00 · Bootstrap prompt

**Paste this into Claude Code at the root of an empty macOS project
directory.** It tells the agent what you're building, where the spec
lives, and how to proceed.

---

```
You are going to help me build a macOS terminal + AI coding companion
app. This is a ground-up implementation; the working directory is
currently empty (or nearly so).

The design spec lives alongside this project in a folder called
`openclide/` (a sibling or sub-folder). Before you write any code,
read these, in this order:

  1. openclide/README.md
  2. openclide/MANIFESTO.md           (philosophy — informs why we deviate below)
  3. openclide/docs/vision.md
  4. openclide/docs/architecture.md
  5. openclide/docs/design-principles.md
  6. openclide/docs/features/terminal-panes.md
  7. openclide/docs/features/workspace-file-tree.md
  8. openclide/docs/features/agent-sidebar.md
  9. openclide/docs/features/local-cli-agent-integration.md
  10. openclide/docs/features/loop-mode.md         (skippable for v1)
  11. openclide/docs/features/magic-summary.md    (skippable for v1)

After you've read those, summarize back to me in 4-6 bullets:
  - Target surface (what the app looks like)
  - Technology stack (what frameworks, what libraries)
  - The three must-haves for v1
  - Two anti-goals I should remind you of if you drift
  - One thing in the spec you'd change for my specific use case
  - What you want to build FIRST (smallest testable slice)

Do NOT start writing Swift yet. I'll confirm the summary before we
proceed.

Constraints for the whole engagement:

  - macOS native. AppKit + SwiftUI, no Electron.
  - Swift 5.9+, targeting macOS 14+.
  - No backend assumptions — I'll tell you what my AI provider is
    later.
  - You will build this in phases. Each phase is a separate prompt I
    will hand you. Do NOT skip ahead to later phases.
  - The spec is a guide, not a contract. If an idea in the spec is
    wrong for my product, tell me and propose an alternative. Your
    judgment matters more than my fidelity to the doc.

When you're ready, just produce the summary.
```

---

## After the summary

If the agent's summary is accurate, reply with:

```
Good. Before you start, create:

  - A README.md with the app name, one-sentence description, and the
    fact that this repo was bootstrapped from the openclide spec.
  - A .gitignore appropriate for an Xcode macOS project.
  - A plain-text NOTES.md where you'll log decisions as we go
    (architectural choices, library picks, trade-offs). This is our
    shared memory for the engagement.

Then proceed to prompt 01-scaffold.md.
```

If the summary is wrong somewhere, fix it inline — don't re-paste
the bootstrap. The agent will update its mental model.
