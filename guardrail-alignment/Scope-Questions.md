# Scope Questions: How to Surface Drift to the User

When a deviation from the plan is detected, use these question patterns to force a conscious decision without being aggressive or unhelpful.

## Category 1: New Feature / Unplanned Work

Use when: Code or a request introduces something not in any plan file.

> "This feature (`<feature name>`) is not in our current `plan/tasks/phase-N-*.md`. Before I implement it, I need to know:
> 1. Are we officially expanding the scope of Phase N?
> 2. Or did you want to note this for a future phase and stay on track?"

## Category 2: Wrong Tech Stack Choice

Use when: The user asks to use a library, framework, or tool that conflicts with `plan/TRD.md`.

> "`plan/TRD.md` explicitly mandates `<correct technology>` for `<purpose>`. You've asked me to use `<proposed technology>` instead.
> Before I proceed:
> 1. Should I update `plan/TRD.md` to reflect this change? (This is a permanent architectural decision.)
> 2. Or should I use `<correct technology>` as originally planned?"

## Category 3: Implementation Ahead of Phase

Use when: A user asks to implement something that belongs to a later phase.

> "This task (`<task name>`) is planned for Phase N, but we are currently in Phase M.
> Implementing it now could create untested dependencies that block our Phase M Definition of Done.
> Do you want to:
> 1. Proceed anyway and accept the risk?
> 2. Park it in a `plan/tasks/phase-N-*.md` task and stay focused on Phase M?"

## Category 4: Missing Plan Entry

Use when: The user provides a detailed implementation that has no corresponding plan file entry.

> "This implementation doesn't have a matching entry in our plan. Before I merge this pattern into the codebase, should I:
> 1. Add it to `plan/technical/entities.md` (if it's a new table/field)?
> 2. Add it to the relevant task file as a completed item?
> 3. Or is this a temporary workaround that should NOT become part of the official plan?"

## Category 5: Ambiguous Scope

Use when: The plan is unclear and the user hasn't explicitly addressed a specific scenario.

> "The plan in `plan/technical/entities.md` doesn't specify the behavior for `<scenario>`.
> There are two valid approaches:
> 1. `<Option A>` — simpler, less flexible.
> 2. `<Option B>` — more complex, more future-proof.
> Which do you want, and should I update the plan to reflect this decision?"
