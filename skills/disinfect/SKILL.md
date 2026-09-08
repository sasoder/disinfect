---
name: disinfect
description: Simplify a completed implementation by removing residue from assumptions, uncertainty, and intermediate designs that no longer apply. Use for a deletion-focused cleanup pass after a feature, fix, refactor, or architectural change has settled, or when the user says "disinfect", "clean up the diff", "remove leftover scaffolding", or "simplify with hindsight". Not for bug hunting, and not for work that is still in progress.
---

# Disinfect

Code written under uncertainty keeps the shape of that uncertainty: hedges, fallbacks, wrappers, and branches that were reasonable while the design was moving and are wrong now that it has settled. Disinfect removes that residue with the benefit of hindsight.

## Scope

1. Determine the diff under review. Uncommitted changes are always included. If the invocation includes an argument, a ref or commit range (`main`, `HEAD~3`, `abc123..def456`) sets the base, and anything that exists as a path in the working tree restricts the review to those paths against the default base. Otherwise the base is the merge-base with the default branch: resolve it with `git symbolic-ref refs/remotes/origin/HEAD` or `@{upstream}`, falling back to `main` then `master`. If the merge-base is HEAD (you are on the default branch, or the branch is already merged) and the working tree is clean, do not declare the work clean; use the commits the conversation identifies as the change, or ask the user for a range.
2. The neighborhood of the diff is the files it touches plus their direct callers and callees. Only change code inside the neighborhood: what the diff introduced, what it made obsolete, and adjacent structure when a contained adjustment (see Decide) requires it. Pre-existing mess elsewhere is out of scope; if it matters, mention it in one line at the end.
3. If the design looks unsettled (the user mentions open questions, the diff carries work-in-progress markers, or tests that pass on the base fail here), ask before proceeding instead of guessing. Failures that already exist on the base branch do not count.

## Method

Re-read the final diff against the surrounding code and the current requirements. State to yourself, in one or two sentences, the design that actually survived. Everything in the diff that does not serve that design is a candidate.

Keep the residue catalog in mind while reading. It is illustrative, not exhaustive: anything that exists only because development passed through an alternative is residue, whether or not it is listed.

**Scaffolding from the journey**
- Feature flags, environment switches, or config keys added to stage a migration that is now complete.
- Compatibility shims, re-exports, and adapter layers between an old and a new shape when only one shape remains, and `legacy` / `v2` / `new_` names that now distinguish from nothing.
- Parameters, options, or generics that every caller passes the same value for.
- Wrappers, helpers, and indirection with a single caller that add no boundary.

**Defense against states that can no longer occur**
- Null checks, try/catch, default fallbacks, and "should never happen" branches for inputs the final design makes impossible.
- Validation duplicated across layers that already trust each other.
- Retry, timeout, or normalization logic added to paper over a bug that was later fixed properly.

**Encoded abandoned designs**
- Comments that explain an approach that no longer exists, or apologize for a constraint that is gone.
- Tests that assert the behavior of a discarded design, test doubles for removed collaborators, and setup that exercises paths no longer reachable.
- TODOs and "temporary" markers whose condition has already been met.
- Types, interfaces, enums, and constants with no remaining references.

**Speculation**
- Extension points, abstract bases, or configuration for variants that were never built.
- Exports and public surface that nothing uses. Consumers outside this repo count as callers: in a published package, an exported symbol is part of the contract unless it is clearly internal.

## Decide

Delete each candidate unless one of these holds:
- It is part of an external contract: public API, serialized format, wire protocol, persisted data, or a compatibility promise, documented or not. If you cannot tell whether an external consumer still relies on a path, ask rather than delete.
- The user, the conversation, or a comment in the diff states it is intentional, for example an extension point for work that is explicitly planned next.
- It is a boundary that still earns its keep: it separates real concerns or isolates a dependency.
- Removing it would require changes outside the neighborhood.

Simplify at the right level. Do not preserve an awkward abstraction merely to keep the diff small; make a contained architectural adjustment when it clearly reduces total complexity, better matches the final requirements, and stays inside the neighborhood. Judge simplicity by the resulting system, not by line count. Prefer deletion and directness over replacing old complexity with new structure. No unrelated refactors, no renames for taste, no speculative redesign.

## Implement and verify

Make the changes. Update tests so they describe the surviving design rather than the abandoned one, and delete tests that only existed for removed paths. Run verification proportionate to the change and consistent with the repository's own instructions: the affected test files at minimum, and the full suite plus typecheck and lint when the cleanup crosses module boundaries. If the repo has no test command, run whatever typecheck or lint exists and say so in the report. If verification fails, fix or revert that item; never leave the tree worse than you found it.

If nothing qualifies, say the implementation is clean and change nothing. Manufactured cleanup is itself residue.

## Report

Keep it short:
- **Removed**: what, and the assumption or intermediate design it belonged to.
- **Reshaped**: any structural adjustment and why it reduced complexity.
- **Kept deliberately**: candidates left in place, and the contract or boundary that justified each.
- **Verified**: what you ran and the result.
