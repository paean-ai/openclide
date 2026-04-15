# OpenClide

**Build a Clide-like terminal + AI coding companion for macOS — from a
design spec, not a source drop.**

OpenClide is the first repository in a new kind of open-source project:
we publish the **idea**, the **architecture notes**, the **design
decisions**, and the **initialization prompts for Claude Code** — but
not the product's source.

Given modern coding agents, these artifacts are enough. You hand them
to Claude Code, iterate, and you get a working app. Your code belongs
to you; our design belongs to everyone.

This is the companion design spec for [Clide](https://clide.paean.ai)
by [Paean](https://paean.ai) / [a8e](https://a8e.ai).

---

## What's in here

```
openclide/
├── MANIFESTO.md           # why we open-source the spec, not the source
├── docs/
│   ├── vision.md          # product stance, non-negotiables, anti-goals
│   ├── architecture.md    # component map, process model, data flow
│   ├── design-principles.md
│   └── features/          # one doc per major surface
│       ├── terminal-panes.md
│       ├── workspace-file-tree.md
│       ├── agent-sidebar.md
│       ├── local-cli-agent-integration.md
│       ├── loop-mode.md
│       └── magic-summary.md
├── prompts/               # paste-and-go prompts for Claude Code
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
3. Open your empty project in Claude Code and paste
   [`prompts/00-bootstrap.md`](prompts/00-bootstrap.md).
4. Follow up with the phase prompts in order.
5. When in doubt, point Claude Code at the relevant `docs/features/*.md`
   and ask it to "reconcile my implementation with this spec."

You'll end up with *your* codebase — not ours. Rename it, license it,
ship it, sell it. Your decisions; your code.

## Why this model

See [`MANIFESTO.md`](MANIFESTO.md).

TL;DR: When AI can reliably turn a good spec into working software, the
*spec* becomes the high-leverage artifact. Open-sourcing source code is
still valuable, but it doesn't scale to letting thousands of teams ship
**their own** variant of your product without forking. Open-sourcing the
*design* does.

## License

- **Docs and prompts**: Creative Commons Attribution 4.0
  ([CC BY 4.0](LICENSE)). Use them, remix them, ship products built from
  them. Credit is the only ask.
- **Any code you write from these prompts**: yours entirely. No viral
  clauses. No attribution requirement on your product.

## Related projects

- Clide (the product these specs describe): <https://clide.paean.ai>
- Paean platform: <https://paean.ai>
- a8e (agent platform): <https://a8e.ai>

## Contributing

Open a PR with:
- Clarifications or corrections to a `docs/**` file.
- A new feature spec that slots into our design.
- A new Claude Code prompt that demonstrably produces better output than
  the existing one (include a before/after transcript).

We don't accept PRs that add implementation code — by design.

---

*Built by [Paean](https://paean.ai). Follow the philosophy at
[a8e.ai](https://a8e.ai).*
