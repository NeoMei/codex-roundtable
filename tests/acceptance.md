# Roundtable Acceptance Checklist

Run these checks after installing or refreshing the personal-marketplace copy. Use a fresh Codex task for discovery and multi-turn behavior. Record only a non-sensitive task alias; never copy private task content, credentials, or full internal identifiers into this public repository.

## Evidence format

Append one evidence line per run with local date, Codex surface, non-sensitive task alias, PASS/FAIL/NOT-APPLICABLE, and a short observation.

## Structural checks

- [ ] Skill validator passes.
- [ ] Plugin validator passes.
- [ ] Manifest has no `mcpServers` or `apps` field.
- [ ] Skill bundle has no executable call to legacy DSH-only tools.
- [ ] Legacy same-name skill preflight ran without overwriting or deleting it.
- [ ] A fresh task exposes only the intended Codex-native `roundtable` skill.

## Fresh-task behavior

- [ ] `圆桌讨论` with no topic starts the topic question.
- [ ] `$roundtable` with an inline topic confirms that topic.
- [ ] Recommended role can be accepted and its persona edited.
- [ ] Custom role can be added.
- [ ] Logical eight-member limit is enforced while one member runs at a time.
- [ ] Host with a model enum permits an explicit non-default model, or is NOT-APPLICABLE.
- [ ] Host without a model enum offers inherited default only, or is NOT-APPLICABLE.
- [ ] Members speak in confirmed roster order.
- [ ] Member output appears as ordered live commentary when supported.
- [ ] Round response contains the complete ordered transcript.
- [ ] Subagent threads are inspectable when the surface exposes them.
- [ ] User interjection reaches the next safe member or next round.
- [ ] Second round reuses the member target or reports a context-preserving replacement.
- [ ] Schema-valid unavailable-model fallback works, or is NOT-APPLICABLE.
- [ ] Interrupted member offers retry, skip, or terminate.
- [ ] Partial round lists members who did not speak.
- [ ] Termination creates a non-overwriting Markdown file with required sections.
- [ ] Surface without structured input cards completes through plain chat.

## Permission review

- [ ] Member prompt contains the analysis-only contract.
- [ ] No member changes a workspace file or external system during the test.
- [ ] Only the host writes the minutes artifact.
- [ ] User-facing copy does not claim hard sandbox isolation.

## Evidence log

The evidence log is intentionally empty before the first installed-plugin test.
