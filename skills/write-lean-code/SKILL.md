---
name: write-lean-code
description: Keep generated and edited code small, readable, and low-complexity by favoring existing abstractions, cross-module reuse, and direct implementations over verbose designs. Use when generating, refactoring, or reviewing code where there is risk of code bloat, duplicated logic, single-file tunnel vision, unnecessary abstraction, or rising cyclomatic complexity.
---

# Write Lean Code

## Overview

Keep changes narrow, readable, and easy to maintain. Reuse existing code before adding new structure, and treat DRY and KISS as design constraints during implementation rather than cleanup goals for later.

## Survey Before Editing

- Inspect the target file and the nearby modules that already own related behavior.
- Search for existing helpers, validators, formatters, constants, adapters, and test fixtures before writing new ones.
- Check callers, sibling modules, and shared layers so the change does not solve a cross-cutting problem in only one place.
- Follow the lightest existing pattern that fits the codebase instead of introducing a new style locally.

## Prefer the Smallest Correct Change

- Modify an existing function or type before adding wrappers, indirection layers, or parallel abstractions.
- Add a new module only when the behavior is clearly shared or the current module boundary is already wrong.
- Keep APIs narrow. Avoid adding option bags, feature flags, or generic extension points unless multiple real callers need them now.
- Prefer simple data flow over orchestration objects such as managers, factories, registries, or strategy layers unless the codebase already uses them for the same problem.

## Reuse Across Module Boundaries

- Extend a shared abstraction when the new behavior matches its responsibility.
- Move logic to the narrowest existing shared layer when multiple modules need it.
- Avoid cloning logic into the current file just because the current task started there.
- Update tests where the behavior truly lives, not only where the current change was made.

## Treat Complexity as a Refactor Trigger

- Prefer straight-line control flow, early returns, and named intermediate values.
- Split logic when a function mixes setup, business rules, formatting, and error handling.
- Treat deep nesting as a smell and flatten it before adding more branches.
- Treat large conditionals over unrelated cases as a sign that responsibilities are blended together.
- Treat multiple boolean flags or temporary state variables steering behavior as a sign that the design is becoming implicit and brittle.

## Apply DRY With Judgment

- Remove duplication in behavior and decision rules, not only in text.
- Extract a helper when it improves naming, isolates unstable logic, or serves multiple call sites.
- Do not extract one-off helpers that only hide three obvious lines and force the reader to jump around.
- Keep similar code paths separate when they are likely to diverge soon and forcing them together would make both harder to read.

## Keep the Code Obvious

- Prefer domain names over generic names such as `manager`, `processor`, `handler`, or `util` when the role can be stated precisely.
- Prefer one clear path through the function over nested condition trees.
- Prefer explicit local code over clever reuse that obscures intent.
- Add comments only when the code cannot reasonably explain itself through structure and naming.

## Run a Final Lean Pass

- Remove dead branches, unused parameters, leftover debug code, and speculative extension points.
- Collapse single-use wrappers and pass-through helpers that add no semantic value.
- Check whether the change can delete or simplify existing code instead of only adding more code.
- Verify that each new abstraction has more than one real reason to exist.

## Explain the Tradeoff Briefly

- State what existing code you reused.
- State what abstraction you intentionally did not introduce and why.
- State any remaining complexity that could not be removed without changing scope.

## Exceptions

- Prefer clarity over forced compactness.
- Preserve an established architectural pattern when local simplification would create inconsistency.
- Accept extra structure when it materially improves safety, testability, or separation of responsibilities.
