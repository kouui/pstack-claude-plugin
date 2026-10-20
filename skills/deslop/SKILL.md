---
name: deslop
description: "Strip AI slop from a code diff: narrating comments, unsupported guards, dead compatibility paths, and unrelated edits. Use before committing, or when a diff feels padded."
disable-model-invocation: false
user-invocable: true
---

# Deslop

Remove the extra code an agent adds on top of the change that was asked for. Keep behavior the same. Keep every line the task needs.

For prose, use **unslop**. For a fresh reviewer's pass over comments, use **no-comments**.

## Scope

Use the caller's files or diff. Otherwise use the current diff against the base branch, default `main`, including the working tree. Touch only lines the diff adds or changes. Do not clean up code the diff leaves alone.

## What to remove

- **Narrating comments.** Comments that restate the next line, describe the edit ("added", "now we", "fixed"), or mark a section the code already shows. Keep a comment only when it records a reason the code cannot show: a constraint, a trade-off, a workaround for a named bug.
- **Unsupported guards.** Null checks, `try`/`catch`, fallbacks, and default values for cases the types, the callers, or the data model rule out. Keep a guard at a real boundary: user input, network, disk, a third-party return value.
- **Dead compatibility paths.** Shims, flags, aliases, and fallbacks that keep an old behavior alive when no caller needs it. If the change replaces an API, migrate the callers and delete the old path.
- **Unrelated edits.** Reformatting, renames, import reordering, drive-by refactors, and new helpers the task did not need. Revert them. If one is worth doing, report it as separate work.
- **Speculative code.** Parameters, options, abstractions, and branches that have no caller yet.

## Steps

1. Read the diff in full. For each hunk, ask whether the task needs it.
2. Delete or revert what the list above names. Do not rewrite working code in a new style.
3. If removing a guard could change behavior, check the callers or run the code. When it is still unclear, keep the guard and report it.
4. Run the narrowest checks that cover the touched files (tests, type check, lint). Fix what your cleanup broke.
5. Report what you removed in a few lines, grouped by kind, and what you kept on purpose with the reason.

## Rules

- Do not change behavior. A cleanup that changes behavior is a new change and needs its own check.
- Do not widen scope. Out-of-diff slop goes in the report, not in the diff.
- Do not delete a comment that carries a constraint such as `do not remove` or `talk to X before changing`. Hand those to **no-comments**.
- Prefer deleting over rewording.

## Who runs it

Deslop is a review of the author's own diff, so the author does not run it. Spawn a fresh subagent (`general-purpose`, `sonnet high`, or the `swarm workers` line) and give it the diff range, the one-sentence task, and the rules in this skill. Do not give it the author's reasoning or chat history. It edits the working tree, runs the checks in step 4, and returns the report from step 5. Review its diff before you commit.
