---
name: lean-code
description: How to write and review code. Do the least that actually works, change only what the request touches, and say what you skipped. Distilled from the karpathy-guidelines and ponytail skills.
triggers: ["code", "diff", "pr", "review", "refactor", "bug", "implement"]
version: "1.0"
---

# Lean Code

Two modes. Read the request and pick one before writing anything.

- **Review mode** — you were given a diff, a PR, or existing code to judge. Output is findings.
- **Write mode** — you were asked to add, fix, or refactor. Output is code.

The principles below apply to both. Only the output format differs.

## Never lazy about understanding

The shortcuts here shorten the solution, never the reading. Trace what the change
actually touches, end to end, before you pick an approach. A small diff shipped
without understanding is the dangerous kind of laziness: it looks efficient and
it is a confident wrong answer.

## Say what you assumed

If two readings of the request are possible, name them instead of silently
picking one. If something is unclear, say what is unclear. If a simpler approach
exists than the one you were asked for, say so in one line and then do what was
asked. Do not hide confusion behind plausible code.

## The ladder (write mode)

Stop at the first rung that holds:

1. Does this need to exist at all? Speculative need, skip it and say so in one line.
2. Already in this codebase? A helper, util, type, or pattern that already lives here, reuse it. Re-implementing what sits a few files over is the most common waste.
3. Stdlib does it? Use it.
4. Native platform feature covers it? A built-in input type over a picker library, CSS over JS, a DB constraint over app code.
5. Already-installed dependency solves it? Use it. Never add a new dependency for what a few lines can do.
6. Can it be one line? One line.
7. Only then: the minimum code that works.

Two rungs both work, take the higher one and move on.

## Surgical changes

Every changed line must trace directly to the request.

- Do not improve adjacent code, comments, or formatting.
- Do not refactor what is not broken. Match existing style even where you would do it differently.
- Remove imports, variables, and functions that *your* change orphaned. Leave pre-existing dead code alone; mention it instead of deleting it.
- No abstraction with one implementation, no config for a value that never changes, no scaffolding for later.
- No error handling for scenarios that cannot occur.

If you wrote 200 lines where 50 would do, rewrite it.

## Fix the cause, not the symptom

A bug report names a symptom. Before editing, find every caller of the function
you are about to touch. One guard in the shared function is a smaller diff than a
guard in every caller, and patching only the path the report names leaves every
sibling caller broken.

## Never simplify away

Input validation at trust boundaries, error handling that prevents data loss,
security, accessibility basics, and anything explicitly requested. Between two
options of the same size, take the one that is correct on edge cases. Doing less
means writing less code, not picking the flimsier algorithm.

## Mark the corners you cut

When a simplification has a known ceiling (a global lock, an O(n²) scan, a naive
heuristic), leave a one-line comment naming the ceiling and the upgrade path:
`# lean: global lock, per-account locks if throughput matters`.

## Leave one check

Non-trivial logic (a branch, a loop, a parser, a money or security path) leaves
behind the smallest runnable thing that fails if the logic breaks: an assert-based
self-check or one small test file. No frameworks, no fixtures, no per-function
suites unless asked. Trivial one-liners need no test.

## Building from a shaped issue

1. **The doc is the spec.** Read it and the code it cites before planning.
   Anything marked not locked gets asked, never quietly picked.
2. **Cut by what ships, not by mechanism.** The fewest sub-issues that each
   leave something verifiable.
3. **One feature branch from the named base.** One sub-issue is one PR into
   it, in roadmap order.
4. **The loop per sub-issue.** Plan at the top of the PR, build, run that
   item's own check from the doc, fix until green, merge into the feature
   branch, then start the next.
5. **Every PR says what it did not verify.** That debt carries into the final
   check; it never disappears.
6. **Final check against the Outcome.** Real instances, and the browser
   wherever there is UI. The PR into the base waits for a human.

## Output

**Write mode.** Code first, then at most three short lines: what you skipped and
when to add it. Pattern: `[code] → skipped: [X], add when [Y].` No essays, no
feature tours, no design notes. If the explanation is longer than the code,
delete the explanation. Every paragraph defending a simplification is complexity
smuggled back in as prose.

**Review mode.** Findings only, ordered most severe first. Separate confirmed
defects from possible risks and label which is which. For each: the exact symbol
or smallest relevant snippet, the condition that triggers it, and the observable
failure. Never assume the contents of code you did not read, and never invent
line numbers. Over-engineering is itself a finding: name the rung of the ladder
the code skipped past.

Explanation the requester explicitly asked for (a report, a walkthrough) is not
debt. Give it in full. The rule is only against unrequested prose.
