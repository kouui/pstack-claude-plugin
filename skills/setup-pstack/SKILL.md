---
name: setup-pstack
description: Configure which models pstack uses per role and at what reasoning budget. Detects your available models and writes an always-applied rule that overrides the skill defaults. Use for /setup-pstack, "configure pstack models", "pstack budget", or changing pstack's model choices.
disable-model-invocation: false
user-invocable: true
---

# Setup pstack

In Claude Code, read [the Claude Code adapter](../../docs/harness/claude-code.md) once per session before you follow this skill. It maps the Cursor names in pstack skills to Claude Code.

Write `~/.claude/rules/pstack-models.md`, a rule that sets pstack's model per role. Claude Code loads every `.md` file in `~/.claude/rules/` into each new session.

This plugin supports three models for now: `opus`, `sonnet`, `haiku` (the 5.5 family). Other models can be added later.

## Steps

### 1. Detect available models

Read the `model` values that the `Agent` tool schema lists, and keep only `opus`, `sonnet`, and `haiku`. If you cannot read them, ask the user which of the three they can use. Never write a model you have not confirmed is available. The aliases `inherit-parent` and `auto` are always valid even though they are not models.

### 2. Load current state

The default role-to-model mapping is the rule shape shown in step 5 below. If `~/.claude/rules/pstack-models.md` already exists, read it and treat its `# budget` line and its role values as the current choices. Otherwise start from those defaults. A line whose role is not in step 5, such as `how critics`, is from a retired role. Drop it.

### 3. Budget, map, and confirm

**(a) Ask for a budget.** Prefer AskUserQuestion over free text; in `-p` mode ask in plain text. Offer these two options with these exact labels, and name the current budget when the rule records one. With no rule, say that `high` matches the skill defaults.

- `high — high reasoning`
- `medium — medium reasoning`

**(b) Apply it.** Build the working table from the skill defaults, and on a re-run keep any role whose model you changed. A role value is `<model> <effort>`, and the effort is only `high` or `medium`. The budget sets the effort of every real entry, panel entries included. `inherit-parent` and `auto` do not change. So `medium` turns `sonnet high` into `sonnet medium`. If the session does not accept the effort for that model, ask the user to choose.

**(c) Show the roles and confirm.** Show every role with its value, marking any model not in the detected set as needing a choice. Also list each line step 2 dropped. Ask whether to accept as-is or change specific roles, offering the detected models plus `inherit-parent` and `auto` (both mean: this role runs on the parent chat model) as the options. For panel roles (arena runners, architect runners, interrogate reviewers) the value is a list, and one subagent runs per entry, alias entries included, so the list length sets the count. `arena cross-judge pool` is also a list, but Arena selects one value from it whose model differs from the parent's when possible. `swarm workers` is the default model for every worker unless a race or comparison assigns another model per arm.

### 4. Validate

Every model written must be in the detected set. `inherit-parent` and `auto` always pass. If a chosen model is not available, stop and ask again.

### 5. Write the rule

Write `~/.claude/rules/pstack-models.md` with no frontmatter, then the comment lines, a `# budget` line with the chosen label and its target effort, and one line per role, using the same labels poteto-mode uses. Overwrite the whole file so re-runs stay idempotent. Never put the role lines in `CLAUDE.md` or `AGENTS.md`. Shape:

```
# pstack model configuration. One line per role. Delete a line to fall back to the skill default.
# `inherit-parent` or `auto` as a value: the role runs on the parent chat model (omit the Agent `model`). Alias entries in a panel list still count toward its fan-out.
# budget: high
feature, refactoring: sonnet high
bug-fix: sonnet high
perf-issue: sonnet high
hillclimb: sonnet high
judgment and prose: opus high
hardest tasks: opus high
how explorer: sonnet high
how explainer: opus high
why investigators: sonnet high
why synthesizer: opus high
reflect tooling: sonnet high
reflect judgment, divergent, synthesizer: opus high
arena runners: opus high, sonnet high
arena cross-judge pool: opus high, sonnet high
swarm workers: sonnet high
architect runners: opus high, sonnet high
interrogate reviewers: opus high, sonnet high
```

### 6. Confirm

Tell the user the rule was written and that it applies to new sessions. Re-running this skill updates it.

### 7. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a `verify-*` skill, or an existing harness). If not, offer once: "want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with /create-verification-skill." On yes, invoke `/create-verification-skill` (resolves wherever pstack is installed: workspace, user, or plugin). On no, move on without pushing.

### 8. Offer a formal verification skill (optional)

Check whether the project already has a formal verification skill (a project skill that runs a model checker, a proof, or another verifier against the code). If not, offer once: "want a formal verification skill, so agents can prove a key property of your code holds in every case in scope, not only the cases tests reach? I can generate one with /create-formal-verification-skill." On yes, invoke `/create-formal-verification-skill` (resolves wherever pstack is installed: workspace, user, or plugin). On no, move on without pushing.
