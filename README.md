# disinfect

An agent skill that cleans up a finished change with the benefit of hindsight.

Code written while a design is still moving keeps the shape of that uncertainty: feature flags for migrations that already finished, wrappers with one caller, `try/catch` around states that can no longer happen, tests that assert a design you abandoned halfway through. Once the implementation has settled, `/disinfect` re-reads the diff, works out which design actually survived, and deletes everything that only existed because you passed through the alternatives.

It is deletion-focused. It does not hunt for bugs, and it only reshapes structure when doing so removes complexity from the code the diff touched. If the diff is already clean, it says so and changes nothing.

## Install

With the [`skills`](https://github.com/vercel-labs/skills) CLI:

```bash
bunx skills add sasoder/disinfect
```

or

```bash
npx skills add sasoder/disinfect
```

Useful flags:

```bash
# install for the current user instead of the current project
bunx skills add sasoder/disinfect -g

# target a specific agent and skip prompts
bunx skills add sasoder/disinfect -a claude-code -y
```

Manual install for Claude Code (safe to rerun; replaces any existing copy):

```bash
mkdir -p ~/.claude/skills && rm -rf ~/.claude/skills/disinfect && tmp=$(mktemp -d) && git clone --depth 1 https://github.com/sasoder/disinfect.git "$tmp" && cp -r "$tmp/skills/disinfect" ~/.claude/skills/disinfect && rm -rf "$tmp"
```

## Use

Run it after a feature, fix, or refactor is done and the tests pass:

```
/disinfect
```

By default it reviews your uncommitted changes plus every commit since the merge-base with the default branch. A ref or range argument changes the base, and a path argument restricts the review to those paths. Uncommitted changes are always included.

```
/disinfect main
/disinfect HEAD~3
/disinfect src/auth
```

The full residue catalog, the rules for what it keeps, and the report format are in [`skills/disinfect/SKILL.md`](skills/disinfect/SKILL.md). Every run ends with a short report of what was removed, what was reshaped, what was deliberately kept and why, and what was verified.

## Layout

```
skills/disinfect/
  SKILL.md            # the skill
  agents/openai.yaml  # display metadata for Codex
```
