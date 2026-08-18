---
name: roundtable
description: Run a guided multi-agent roundtable when the user says 圆桌讨论 or 圆桌会议, asks expert roles to debate, or explicitly invokes $roundtable. Do not use for ordinary brainstorming or single-perspective advice.
---

# Roundtable

Run a multi-round discussion with a fixed neutral host and user-configured Codex subagents.

## Preconditions

- Use real Codex subagents for members. Do not simulate several members inside the host response.
- This skill instruction is an explicit request to delegate the configured member work.
- Never create separate top-level Codex tasks for members.
- If subagent tools are unavailable or disabled, explain that a true roundtable cannot run and stop before presenting simulated member output.

## Configure the discussion

Read [references/setup-wizard.md](references/setup-wizard.md) completely and follow it.

Keep the confirmed topic and ordered canonical roster in the main task context. Rename the current task to the topic when a task-title tool is available.

## Run a round

Run members one at a time in roster order. The active concurrency requirement is one member.

For every member, build a self-contained prompt containing the confirmed topic; member ID, role, and persona; the analysis-only contract below; canonical prior-round summaries and human interjections; earlier completed contributions in the current round; and a request for one focused contribution.

Analysis-only contract for every member:

> Participate only as an analyst in this roundtable. You may inspect provided or workspace context with read-only tools when necessary. Do not modify files, change external state, send messages, create tasks, or perform destructive actions. Return only your focused roundtable contribution to the host.

### First round

Use the active subagent-spawn tool with a unique target name derived from the member ID.

Normalize canonical member IDs to the active spawn tool grammar: replace hyphens with underscores (`member-1` -> `member_1`) and keep only lowercase letters, digits, and underscores.

- For `model_mode: explicit`, pass only the exact schema-exposed model identifier and use no full-history fork; the self-contained prompt is the complete context.
- For `model_mode: inherit`, omit the model override.
- Wait for that member to finish before starting the next member.
- Record the returned member target for preferred reuse.
- Treat empty, cancelled, or error results as failures, not contributions.

When an explicit model is schema-valid but fails to start, announce the fallback and retry once with the model override omitted. Record the reported effective model; if unavailable, record `inherited default`.

If the retry also fails, pause and ask the user to choose `retry`, `skip member`, or `terminate`. Never invent a missing contribution.

### Later rounds

Prefer sending a follow-up task to the recorded member target, then wait for its result. The follow-up prompt must still be self-contained and include canonical prior-round summaries.

If the target is unavailable or host capacity prevents reuse, create a replacement subagent with the same member record, announce the replacement, and continue. Semantic continuity comes from the canonical prompt, not private agent history.

### Present contributions

After each successful member, emit an ordered progress/commentary update when supported:

`[角色名]\n<verbatim contribution>`

Do not paraphrase before the host summary. At the end of the round, include all completed contributions in roster order in the main response. Do not promise one permanent main-task message per member; the inspectable subagent thread is authoritative when the surface exposes it.

If the user interjects while work is running, preserve the message as human input. Add it to the next safe member prompt or, if the current round is already complete, the next round. Never discard it silently.

## Host summary and gate

The host remains neutral and has no configurable persona. After every round, read [references/minutes-format.md](references/minutes-format.md) and produce the canonical round record: positions, agreements, disagreements, risks, assumptions, unresolved questions, user input, participation status, and recommended next focus.

Ask the user to choose `continue` or `terminate`; allow an additional opinion. Continue only after a user response. On continue, increment the round number and use the same roster unless the user explicitly asks to change it.

## Terminate and export

On terminate, read [references/minutes-format.md](references/minutes-format.md) completely. Produce the final synthesis and write the verified Markdown artifact exactly as specified.

If the write cannot be completed, return the complete Markdown in chat and state that no file was written.

## Recovery boundary

Within the same live task, reconstruct an unavailable member from the canonical roster and round records when possible, and announce the reconstruction. App-restart and in-flight request recovery are best-effort and are not guaranteed.
