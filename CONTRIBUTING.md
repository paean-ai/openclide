# Contributing to OpenClide

OpenClide is a spec, not a codebase. Contributions look different
here.

## What we accept

- **Clarifications and corrections** to any document in `docs/`.
  If you built something from a prompt and hit ambiguity, a doc PR
  is the right fix.
- **New feature specs** that slot cleanly into the architecture.
  Before writing one, open an issue describing the surface and why
  the current design doesn't already cover it.
- **Better prompts** in `prompts/`. If you've found wording that
  reliably produces better Claude Code output, show us with a
  before/after transcript.
- **New Claude Code skills** under `skills/`. Follow the
  `SKILL.md` format in `skills/design-review/`.
- **Typo fixes.** Always welcome; no issue needed.

## What we do not accept

- **Source code that implements any of the specs.** That's what the
  repo is designed to *produce* when you follow the prompts, not
  what we publish. Source-level PRs will be closed with a link to
  the manifesto.
- **Adding a dependency** on a specific vendor's product unless the
  spec is clearly about that integration.
- **Secrets, internal URLs, company-specific identifiers.** We will
  squash-merge these out even if the rest of the PR is fine.

## Style

Plain American English, second person ("you"), present tense where
possible. Code fences for shell commands and JSON examples. No
emojis unless illustrating a UI element.

Each doc page should be readable in under 10 minutes. If it's
longer, split it.

## The "why before what" rule

Every design recommendation needs a one-line rationale the reader can
disagree with. Specs that assert "do X" without "because Y" aren't
useful to someone deviating — they just look like cargo-culted
dogma.

## Legal

By opening a PR you agree the content of your contribution is
licensed under the same CC BY 4.0 terms as the rest of the repo.
