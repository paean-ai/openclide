---
name: design-review
description: Reviews your in-progress implementation against the OpenClide spec and flags deviations.
---

# Design review

Use this skill mid-build to keep your implementation honest with the
OpenClide spec.

## When to use

- After you've landed a feature prompt (e.g. `prompts/02-terminal-core.md`)
  and want a sanity-check before moving on.
- Before merging a PR that touches a feature covered by `docs/features/`.
- When you feel you've "drifted" from the design and can't tell if
  the drift is good or bad.

## Procedure

1. Ask the user which feature to review, or infer from their most
   recent changes (git diff).
2. Read the matching spec file(s) under `openclide/docs/features/`
   and `openclide/docs/design-principles.md`.
3. Read the actual implementation — the relevant Swift files.
4. Produce a report:

```
## Design review: <feature name>

### Aligned
- bullet
- bullet

### Deviations (with assessment)
- <what differs>
  - **Spec says:** <short quote or paraphrase>
  - **Code does:** <what the code actually does>
  - **Assessment:** <good / neutral / bad, with one-line reason>

### Missing (spec calls for, code lacks)
- bullet (+ suggested minimal change)

### Over-built (code has, spec doesn't require)
- bullet (+ recommended removal or justification)

### Risks
- bullet

### Recommended next actions
1. ...
2. ...
```

5. Don't write code in the report. That's a separate decision the
   user makes after reading it.

## Principles for this skill

- **Deviations are often correct.** The spec is a starting point;
  the user's context may justify changes. Flag, don't scold.
- **Cite the spec precisely** (file path + short quote) so the user
  can push back or edit the spec if they disagree.
- **Prefer small, concrete suggestions** over sweeping rewrites.
- **Never recommend over-engineering**. If the feature works with
  less code, say so.
