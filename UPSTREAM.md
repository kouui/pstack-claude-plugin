# Upstream

Ported from [cursor/plugins `pstack`](https://github.com/cursor/plugins/tree/main/pstack) by Lauren Tan (MIT, see `LICENSE`).

- Upstream commit: `d457567332aa6989e753ea289f30ea2bfbc80bc0` (2026-10-19), pstack 0.15.28.
- Base: the generated Claude Code tree `harness/pstack/claude-code/` of that commit (skills, agents, `docs/guide`, README, LICENSE; the upstream `docs/harness/claude-code.md` adapter was folded into `poteto-mode` and removed).
- Not ported: `automations/benny/` (Cursor Automations and Slack only), `skills/make-bot-ui` (Cursor Automations and Grok Bot only; removed), `assets/logo.png`, `plugin.json` / `.cursor-plugin/`, and the generator `scripts/generate-pstack-harness-trees.mjs`. This repo edits the Claude Code version directly.
- Changed in the port: model defaults (only `opus`, `sonnet`, `haiku`; see below), `skills/setup-pstack` rewritten for Claude Code, install instructions, and `skills/interrogate` model fallback paragraph.
- Model mapping used for defaults: upstream `claude-opus-5-5-xhigh` -> `opus high`; upstream `grok-4.7-xhigh-fast` -> `sonnet high`. Only `high` and `medium` effort are used (no `xhigh`, no `max`). Rule file: `~/.claude/rules/pstack-models.md`.

Everything else is unchanged. Open problems are in `docs/known-issues.md`.
