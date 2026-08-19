# Roundtable Acceptance Checklist

Run these checks after installing or refreshing the personal-marketplace copy. Use a fresh Codex task for discovery and multi-turn behavior. Record only a non-sensitive task alias; never copy private task content, credentials, or full internal identifiers into this public repository.

## Evidence format

Append one evidence line per run with local date, Codex surface, non-sensitive task alias, PASS/FAIL/NOT-APPLICABLE, and a short observation.

## Structural checks

- [x] Skill validator passes.
- [x] Plugin validator passes.
- [x] Manifest has no `mcpServers` or `apps` field.
- [x] Skill bundle has no executable call to legacy DSH-only tools.
- [x] Legacy same-name skill preflight ran without overwriting or deleting it.
- [x] A fresh task exposes only the intended Codex-native `roundtable` skill.

## Fresh-task behavior

- [x] `圆桌讨论` with no topic starts the topic question.
- [x] `$roundtable` with an inline topic confirms that topic.
- [x] Recommended role can be accepted and its persona edited.
- [x] Custom role can be added.
- [ ] Logical eight-member limit is enforced while one member runs at a time.
- [x] Host with a model enum permits an explicit non-default model, or is NOT-APPLICABLE.
- [x] Host without a model enum offers inherited default only, or is NOT-APPLICABLE.
- [x] Members speak in confirmed roster order.
- [x] Member output appears as ordered live commentary when supported.
- [x] Round response contains the complete ordered transcript.
- [x] Subagent threads are inspectable when the surface exposes them.
- [x] User interjection reaches the next safe member or next round.
- [x] Second round reuses the member target or reports a context-preserving replacement.
- [x] Schema-valid unavailable-model fallback works, or is NOT-APPLICABLE.
- [ ] Interrupted member offers retry, skip, or terminate.
- [ ] Partial round lists members who did not speak.
- [x] Termination creates a non-overwriting Markdown file with required sections.
- [x] Surface without structured input cards completes through plain chat.

## Permission review

- [x] Member prompt contains the analysis-only contract.
- [x] No member changes a workspace file or external system during the test.
- [x] Only the host writes the minutes artifact.
- [x] User-facing copy does not claim hard sandbox isolation.

## Evidence log

- 2026-08-19 | Codex Desktop | RT-A1 | PASS | Installed personal plugin resolved in a fresh task; two configured members completed two ordered rounds, an explicit model failure fell back to inherited default, a human interjection reached round two, existing member targets were reused, and the neutral host wrote and verified a 97-line Markdown artifact.
- 2026-08-19 | Codex Desktop | RT-A2 | PASS | Natural-language invocation without a topic asked for one, a custom role was accepted, and cancellation before execution spawned no members and wrote no additional minutes file.
- 2026-08-19 | Codex Desktop | RT-A1 | NOT-APPLICABLE | The active spawn surface exposed a finite model enum, so the no-enum branch was not available in this run.
