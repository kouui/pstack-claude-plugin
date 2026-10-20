---
name: what-should-i-ask
description: Reads the pstack guide for the user's request and recommends which skills to use and what to ask, with the keywords to put in the prompt. Type /what-should-i-ask followed by what you want to get done.
disable-model-invocation: false
user-invocable: true
---

# What should I ask

In Claude Code, read [the Claude Code adapter](../../docs/harness/claude-code.md) once per session before you follow this skill. It maps the Cursor names in pstack skills to Claude Code.

Turn the user's request into a recommendation: which pstack skills to use, and how to word the question. The guide in [`docs/guide/`](../../docs/guide/README.md) is the source. Do not answer from memory.

1. Take the request as the user wrote it. If the user gave none, ask for one sentence about what they want to get done, then stop.
2. Spawn one agent with `subagent_type: "Explore"` and `model: "haiku"`. Give it the request verbatim and these instructions:
   - Do not edit files.
   - Read every `.md` file directly in the guide directory (`README.md` and `01` to `10`). Skip `zh/` and `images/`. Resolve the absolute path of `../../docs/guide/` from this skill's directory and put that path in the brief.
   - Pick the 1 to 3 skills that fit the request best. A task can need more than one, such as a design skill and then a build skill. Order them as the user should run them.
   - For each skill, return: the skill name, one sentence on why it fits, the guide file and section that says so, and 3 to 5 keywords or phrases to put in the prompt. Take keywords from the guide's own wording first: principle names (`08-principles.md`), recipe prompts (`10-recipes-and-pitfalls.md`), and the phrases each skill page uses.
   - Add one line on the pitfall the guide warns about for this kind of task, when it names one.
   - Quote the guide. Do not invent skill names or keywords the guide lacks.
3. Check each skill name the agent returned against the directories in `skills/`. Drop any that does not exist, and say so.
4. Reply with the recommendation: the skills in run order, a ready-to-paste prompt for the first one that uses the keywords, and the pitfall line. Keep it short. Do not run the skills yourself unless the user asks.
