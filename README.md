<div align="center">

<img src="https://clide.paean.ai/icon.svg" alt="Clide" width="120" />

<h1>OpenClide</h1>

<p>
  <strong>Build a Clide‑like AI‑native terminal for macOS —<br/>
  from a design spec, not a source drop.</strong>
</p>

<p>
  <a href="https://clide.paean.ai">Website</a> ·
  <a href="MANIFESTO.md">Manifesto</a> ·
  <a href="docs/architecture.md">Architecture</a> ·
  <a href="docs/features/">Feature specs</a> ·
  <a href="prompts/">Agent prompts</a>
</p>

<p>
  <a href="LICENSE"><img alt="Docs license" src="https://img.shields.io/badge/docs-CC%20BY%204.0-4AA3E0.svg" /></a>
  <img alt="Your code" src="https://img.shields.io/badge/your%20code-yours-3DD8EB.svg" />
  <img alt="Model" src="https://img.shields.io/badge/model-spec--not--source-A78BFA.svg" />
  <a href="https://clide.paean.ai"><img alt="Reference app" src="https://img.shields.io/badge/reference-Clide%20for%20macOS-000000.svg?logo=apple" /></a>
</p>

<p>
  <a href="https://www.producthunt.com/products/clide-the-ai-native-mac-terminal?embed=true&amp;utm_source=badge-featured&amp;utm_medium=badge&amp;utm_campaign=badge-clide" target="_blank" rel="noopener noreferrer">
    <img alt="Clide — Grid-layout terminal with an AI that drives your shells. | Product Hunt" width="250" height="54" src="https://api.producthunt.com/widgets/embed-image/v1/featured.svg?post_id=1123333&amp;theme=dark&amp;t=1776353001959" />
  </a>
</p>

<br/>

<img src="https://clide.paean.ai/docs/screenshots/1-hero.png" alt="Clide — AI‑native terminal for macOS" width="860" />

</div>

---

## The idea

OpenClide is the first repository in a new kind of open‑source project: we publish the **idea**, the **architecture notes**, the **design decisions**, and the **initialization prompts for Claude Code** — but not the product's source.

Given modern coding agents, these artifacts are enough. You hand them to Claude Code, iterate, and you end up with a working app. Your code belongs to you; our design belongs to everyone.

This is the companion design spec for [Clide](https://clide.paean.ai) — an AI‑native Mac terminal built by [Paean](https://paean.ai) / [a8e](https://a8e.ai).

## What you'll build

A **native macOS terminal** that turns your shell into an AI‑native workspace — shipped from a 2.5 MB signed and notarized app.

- **✦ AI Summary** — Watches terminal activity and drafts concise, reviewable session summaries. Catch errors you missed; resume context after a break.
- **⊞ Grid panes** — Flexible tiled shells from 1×1 up to 6×6. Toolbar grid picker; one click to rearrange.
- **📁 Workspace sidebar** — Source‑list file tree with drag‑to‑insert paths. Switch workspaces from the dropdown. Toggle with `⌘B`.
- **🎤 Voice input** — Hold‑to‑talk or toggle mode with Apple's on‑device speech recognition. Configurable hotkeys.
- **⚡ Loop mode** — Automate multi‑shell flows (build → test → deploy) with declarative step definitions.
- **🤖 Local CLI agent integration** — Wire Claude Code / codex‑cli as a first‑class pane, with shell‑level context.
- **🎨 Theme presets** — Midnight, Solarized, Nord, Dracula, Light, Catppuccin. Adaptive light/dark.
- **⌘ Native AppKit** — Not Electron. Standard macOS shortcuts, Reveal in Finder, toolbar customization.

Every surface has a matching design doc in [`docs/features/`](docs/features/) and a bootstrap prompt in [`prompts/`](prompts/).

## Architecture at a glance

```
┌────────────────────────────────────────────────────┐
│                   AppKit Window                    │
├─────────────────────┬──────────────────────────────┤
│                     │     Terminal Grid            │
│  Workspace Sidebar  │  ┌─────────┬─────────┐       │
│   (file tree,       │  │ Pane 1  │ Pane 2  │       │
│   drag‑to‑path)     │  ├─────────┼─────────┤       │
│                     │  │ Pane 3  │ ✦  AI   │       │
│                     │  └─────────┴─────────┘       │
├─────────────────────┴──────────────────────────────┤
│  Session Model           │   Input Pipeline        │
│  ├─ PaneTree             │   ├─ Keystroke router   │
│  ├─ HistoryStore         │   ├─ Voice → STT        │
│  └─ SummaryCache         │   └─ Drag‑and‑drop      │
├──────────────────────────┴─────────────────────────┤
│  SwiftTerm               │   Agent Bridge          │
│  (PTY + ANSI renderer)   │   ├─ Claude Code spawn  │
│                          │   └─ Loop orchestrator  │
└──────────────────────────┴─────────────────────────┘
```

Full component breakdown, process model, and data flow live in [`docs/architecture.md`](docs/architecture.md).

## What's in here

```
openclide/
├── MANIFESTO.md           # why we open‑source the spec, not the source
├── docs/
│   ├── vision.md          # product stance, non‑negotiables, anti‑goals
│   ├── architecture.md    # component map, process model, data flow
│   ├── design-principles.md
│   └── features/          # one doc per major surface
│       ├── terminal-panes.md
│       ├── workspace-file-tree.md
│       ├── agent-sidebar.md
│       ├── local-cli-agent-integration.md
│       ├── loop-mode.md
│       └── magic-summary.md
├── prompts/               # paste‑and‑go prompts for Claude Code
│   ├── 00-bootstrap.md
│   ├── 01-scaffold.md
│   ├── 02-terminal-core.md
│   ├── 03-agent-sidebar.md
│   └── 04-local-agent.md
└── skills/
    └── design-review/     # a Claude Code skill that critiques your
                           # implementation against this spec
```

## Quickstart

1. Install [Claude Code](https://claude.com/claude-code).
2. Clone this repo next to (or inside) your target project folder.
3. Open your empty project in Claude Code and paste [`prompts/00-bootstrap.md`](prompts/00-bootstrap.md).
4. Follow up with the phase prompts in order.
5. When in doubt, point Claude Code at the relevant [`docs/features/*.md`](docs/features/) and ask it to *"reconcile my implementation with this spec."*

You'll end up with **your** codebase — not ours. Rename it, license it, ship it, sell it.

## Why this model

See [`MANIFESTO.md`](MANIFESTO.md).

**TL;DR** — When AI can reliably turn a good spec into working software, the *spec* becomes the high‑leverage artifact. Open‑sourcing source code is still valuable, but it doesn't scale to letting thousands of teams ship **their own** variant of your product without forking. Open‑sourcing the *design* does.

## License

| What | License |
| --- | --- |
| Docs and prompts | [CC BY 4.0](LICENSE) — use them, remix them, ship products built from them. Credit is the only ask. |
| Any code you write from these prompts | **Yours entirely.** No viral clauses. No attribution requirement on your product. |

## Related projects

- **[Clide](https://clide.paean.ai)** — the AI‑native Mac terminal these specs describe (signed DMG, ready to download)
- **[Paean](https://paean.ai)** — the platform that powers Clide's hosted agent
- **[a8e.ai](https://a8e.ai)** — the agent infrastructure behind it all

## Contributing

PRs welcome for:

- Clarifications or corrections to a `docs/**` file.
- A new feature spec that slots into the design.
- A new Claude Code prompt that demonstrably produces better output than the existing one *(include a before/after transcript)*.

We don't accept PRs that add implementation code — **by design**.

---

<p align="center">
  <sub>
    Built by <a href="https://paean.ai">Paean</a>
    · Powered by <a href="https://a8e.ai">a8e</a>
    · Terminal rendering by <a href="https://github.com/migueldeicaza/SwiftTerm">SwiftTerm</a>
  </sub>
</p>
