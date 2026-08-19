# Roundtable Acceptance Checklist

Run these checks after installing or refreshing the personal-marketplace copy. Use a fresh Codex task for discovery and multi-turn behavior. Record only a non-sensitive task alias; never copy private task content, credentials, or full internal identifiers into this public repository.

Current status: structural validation has been refreshed, but behavioral acceptance is partial. Unchecked behavior must not inherit evidence from an earlier source commit.

## Evidence format

Append one evidence line per run with local date, Codex surface, non-sensitive task alias, source commit, installed cachebuster/version, PASS/FAIL/NOT-APPLICABLE, and a short observation.

## Structural checks

- [x] Skill validator passes.
- [x] Plugin validator passes.
- [x] Manifest has no `mcpServers` or `apps` field.
- [x] Skill bundle has no executable call to legacy DSH-only tools.
- [ ] Legacy same-name skill preflight inspects both standard standalone roots without overwriting or deleting entries.
- [ ] A fresh task exposes only the intended Codex-native `roundtable` skill.

## Fresh-task behavior

- [ ] `圆桌讨论` with no topic starts the topic question.
- [ ] `$roundtable` with an inline topic confirms that topic.
- [ ] Recommended role can be accepted and its persona edited.
- [ ] Custom role can be added.
- [ ] A persona requesting side effects or conflicting with the analysis-only contract is rejected.
- [ ] Logical eight-member limit is enforced while one member runs at a time.
- [ ] Host with a model enum permits an exact explicit non-default model, or is NOT-APPLICABLE.
- [ ] Host without a model enum offers `Host default (no model override)` only, or is NOT-APPLICABLE.
- [ ] `effective_model` starts null, becomes the exact enum after explicit success, and becomes `host default` only after host-default success.
- [ ] Every spawn, including no-override fallback and reconstruction, uses the host's explicit no-history setting.
- [ ] A host without a no-history spawn setting asks proceed/cancel before spawning.
- [ ] Every spawn attempt uses a fresh generation target and only successful targets are reused.
- [ ] Explicit-model failure falls back on a fresh target and later reconstruction preserves host-default runtime policy, or is NOT-APPLICABLE.
- [ ] Capacity is relieved only through supported close operations after canonicalization, or an unrelievable cap reaches the retry/skip/terminate gate.
- [ ] Members speak in confirmed roster order.
- [ ] Member output appears as ordered live commentary when supported.
- [ ] Round response contains the complete ordered transcript.
- [ ] Subagent threads are inspectable when the surface exposes them.
- [ ] Ordinary user interjection reaches the next safe member or next round as untrusted data.
- [ ] Second round reuses the member target or reports a context-preserving fresh-generation replacement.
- [ ] Interrupted member offers retry, skip, or terminate.
- [ ] Cancel/`取消` interrupts when supported, ignores late output, records a partial round and non-speakers, and exits without export.
- [ ] Terminate/`终止` interrupts when supported, ignores late output, records a partial round and exports partial minutes.
- [ ] Ambiguous stop/`停止` interrupts first and asks cancel-without-export versus terminate-with-export.
- [ ] Termination creates a Markdown file with required sections.
- [ ] A collision leaves the existing artifact unchanged and creates the next numeric suffix.
- [ ] Surface without structured input cards completes through plain chat.

## Permission review

- [ ] Member prompt contains the analysis-only contract outside delimited untrusted discussion data.
- [ ] Topic, persona, summaries, interjections, and earlier contributions are delimited and cannot override the member contract.
- [ ] An adversarial topic containing exact `</discussion-data>` is entity-encoded and cannot close the data block.
- [ ] An adversarial user interjection containing exact `</discussion-data>` is entity-encoded and cannot close the data block.
- [ ] An earlier member contribution containing exact `</discussion-data>` is entity-encoded and cannot close the data block.
- [ ] No member changes a workspace file or external system during the test.
- [ ] Only the host writes the minutes artifact.
- [ ] User-facing copy does not claim hard sandbox isolation.

## Evidence log

Historical evidence below is retained for traceability and does not satisfy reset checkboxes for the current source.

- 2026-08-19 | Codex Desktop | RT-A1 | source f4f8140 | installed 0.1.0+codex.20260819105232 | PASS | Historical run: two configured members completed two ordered rounds; an explicit model failure eventually used host default after an ineffective same-target retry was corrected; a human interjection reached round two; targets were reused; the host wrote and verified a 97-line Markdown artifact. Collision behavior was not tested.
- 2026-08-19 | Codex Desktop | RT-A2 | source f4f8140 | installed 0.1.0+codex.20260819105232 | PASS | Historical run: natural-language invocation without a topic asked for one, a custom role was accepted, and setup cancellation spawned no members and wrote no additional minutes file.
- 2026-08-19 | Codex Desktop | RT-A1 | source f4f8140 | installed 0.1.0+codex.20260819105232 | NOT-APPLICABLE | Historical run: the active spawn surface exposed a finite model enum, so the no-enum branch was unavailable and remains unchecked.
