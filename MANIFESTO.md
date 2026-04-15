# The Spec-Source Manifesto

## What we are open-sourcing, and what we are not

Traditional open-source projects publish **source code**. The
implicit contract: "here is the artifact; fork it, modify it, redistribute
it, credit us."

OpenClide does something different. We publish:

- The **product vision** (what problem, for whom, what it refuses to do).
- The **architecture** (components, process model, data flow).
- The **design decisions** (why NSSplitView over NSStackView here, why
  stdio NDJSON over a WebSocket there, why per-workspace persistence).
- The **Claude Code prompts** needed to regenerate the product from the
  above.

We do **not** publish the source code of [Clide](https://clide.paean.ai).
Clide stays closed. Everything that lets you build your own Clide is open.

We call this model **Spec-Source**.

## Why this is enough

AI coding agents are now capable enough that a well-written spec plus
staged initialization prompts will, in a day or two of iteration, give
you a working app of comparable shape. You don't need our code — you
need our *thinking*.

The shift happening under all of us:

| Era | High-leverage artifact | What open-source meant |
| --- | --- | --- |
| 1990s–2010s | Source code | Publish the source |
| 2020s+ | Design spec + agent prompts | Publish the spec |

Spec-source is what happens when you take that shift seriously.

## Why this is better than publishing the source

### For builders who use this repo

- You end up with **your own codebase**, under **your own license**. No
  viral clauses, no attribution in your About dialog, no legal
  entanglement with us.
- You can deviate from our design at any point. Forking a codebase is
  expensive; forking an idea is free.
- You learn the reasoning, not just the result. Re-reading someone
  else's `UIViewController` teaches you what they did. Re-reading a
  design spec teaches you *why* — which is what you need when your
  product grows past the spec.

### For us, the maintainers

- We can keep shipping a polished commercial product without losing the
  community benefit of open sourcing.
- We don't have to sanitize the repo — there are no secrets, API URLs,
  or customer-specific configuration to leak, because there is no
  source.
- PRs are about ideas, not syntax. Code reviews become design reviews.
- We don't compete with a thousand half-maintained forks of our own
  binary.

### For the ecosystem

- Design patterns propagate faster than source. A good spec can power
  implementations in Swift, Rust, Electron, Tauri — each tuned to its
  own audience.
- The spec is a teaching artifact. Students can read why modern desktop
  apps make the choices they do; professionals can read the non-obvious
  trade-offs; competitors can read what we're *not* doing and why.

## What this is NOT

- **Not a gimmick to hide code**. We didn't write a toy spec and call
  it a day. These docs are the same docs we'd hand to a new engineer on
  our own team.
- **Not anti-open-source**. It is open-source, in a form suited to the
  era of coding agents. Classic source-code OSS remains valuable. We
  just think it's not the only shape open-source can take.
- **Not "prompt engineering theatre"**. The prompts in `prompts/` are
  staged so a real, modern agent can execute them and produce real code
  — not a decorative performance.
- **Not a replacement for your taste**. Our spec reflects what *we*
  think is a good terminal + AI companion. Your version will differ.
  Good.

## The rules of this repo

1. **No source code.** Pseudocode fragments for clarity are fine when
   they illustrate a protocol; full implementations are not.
2. **No secrets.** No API endpoints, no keys, no internal URLs, no team
   IDs, no customer names.
3. **Why before what.** Every design decision gets a one-line rationale.
   Readers who disagree with the rationale will cheerfully deviate —
   that's the point.
4. **The spec is the product.** Treat `docs/` the way you'd treat code:
   land changes through PRs, keep it current, delete what's obsolete.

## Where this fits with Paean and a8e

- [Paean](https://paean.ai) builds AI products for end users — Clide is
  one of them.
- [a8e](https://a8e.ai) is the agent platform those products run on.
- OpenClide is how we share what we've learned building desktop
  agent-native software, without giving up the product we sell.

If you build something from these specs, we'd love to hear about it —
tag us, send us a link, file a PR that describes your deviation and
what worked better. The model only earns its keep if other people can
actually ship with it.

— The Paean team
