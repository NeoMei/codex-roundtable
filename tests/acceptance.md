# Roundtable Acceptance Checklist

Run these checks after installing or refreshing the personal-marketplace copy. Use a fresh Codex task for discovery and multi-turn behavior. Record only a non-sensitive task alias; never copy private task content, credentials, or full internal identifiers into this public repository.

Current status: structural validation and several namespaced behaviors have been refreshed, but behavioral acceptance remains partial. Unchecked behavior must not inherit evidence from an earlier source commit.

## Evidence format

Append one evidence line per run with local date, Codex surface, non-sensitive task alias, source commit, installed cachebuster/version, PASS/FAIL/PARTIAL/NOT-APPLICABLE, and a short observation.

## Structural checks

- [x] Skill validator passes.
- [x] Plugin validator passes.
- [x] Manifest has no `mcpServers` or `apps` field.
- [x] Skill bundle has no executable call to legacy DSH-only tools.
- [ ] Legacy same-name skill preflight inspects both standard standalone roots without overwriting or deleting entries.
- [x] `codex debug prompt-input '$roundtable:roundtable test'` lists the plugin entry `roundtable:roundtable` and resolves the current installed plugin cache entry.
- [x] When a standalone conflict coexists, discovery distinguishes unnamespaced `roundtable` from plugin `roundtable:roundtable`.

## Fresh-task behavior

- [ ] `圆桌讨论` with no topic starts the topic question when no standalone name conflict is visible.
- [x] `$roundtable:roundtable` with an inline topic confirms that topic.
- [x] With a visible standalone conflict, the workflow uses `$roundtable:roundtable` rather than treating `$roundtable` as deterministic plugin invocation.
- [ ] Recommended role can be accepted and its persona edited.
- [ ] Custom role can be added.
- [ ] A persona requesting side effects or conflicting with the analysis-only contract is rejected.
- [x] Logical eight-member limit removes the add action at eight and explicitly rejects a ninth member.
- [x] Setup cancellation spawns no member and writes no minutes file.
- [x] Host with a model enum permits an exact explicit non-default model, or is NOT-APPLICABLE.
- [ ] Host without a model enum offers `Host default (no model override)` only, or is NOT-APPLICABLE.
- [ ] `effective_model` starts null, becomes the exact enum after explicit success, and becomes `host default` only after host-default success.
- [x] Every observed spawn, including a restarted invocation, uses the host's explicit no-history setting.
- [ ] A host without a no-history spawn setting asks proceed/cancel before spawning.
- [x] Observed spawn attempts use fresh per-member generation targets (`member_1_g1` through `member_1_g4` across the recorded runs).
- [x] Only successful targets are reused in later rounds.
- [x] Explicit-model failure falls back on a fresh target and persists host-default runtime policy for later reuse or reconstruction, or is NOT-APPLICABLE.
- [ ] Capacity is relieved only through supported close operations after canonicalization, or an unrelievable cap reaches the retry/skip/terminate gate.
- [x] Members speak in confirmed roster order for the observed one-member round.
- [x] Member output appears as ordered live commentary when supported.
- [x] Round response contains the complete ordered transcript.
- [x] Subagent threads are inspectable when the surface exposes them.
- [x] Ordinary user interjection reaches the next safe member or next round as untrusted data.
- [x] Second round reuses the member target or reports a context-preserving fresh-generation replacement.
- [ ] A member failing both initial and fallback attempts offers retry, skip, or terminate.
- [ ] Cancel/`取消` interrupts when supported, ignores late output, records a partial round and non-speakers, and exits without export.
- [ ] Terminate/`终止` interrupts when supported, ignores late output, records a partial round and exports partial minutes.
- [x] Ambiguous stop/`停止` interrupts first, ignores late output, records the partial round and non-speaker, and asks cancel-without-export versus terminate-with-export.
- [x] Choosing cancel after ambiguous stop writes no export.
- [x] Termination creates a Markdown file with required sections.
- [x] A collision leaves the existing artifact unchanged and creates the next numeric suffix.
- [x] Surface without structured input cards completes the observed setup through plain chat.

## Permission review

- [ ] Member prompt contains the analysis-only contract outside delimited untrusted discussion data.
- [ ] Topic, persona, summaries, interjections, and earlier contributions are delimited and cannot override the member contract.
- [ ] An adversarial topic containing exact `</discussion-data>` is entity-encoded and cannot close the data block.
- [x] An adversarial persona containing exact `</discussion-data>` is entity-encoded and cannot close the data block.
- [x] An adversarial user interjection containing exact `</discussion-data>` is entity-encoded and cannot close the data block.
- [x] An earlier member contribution containing exact `</discussion-data>` is entity-encoded and cannot close the data block.
- [ ] No member changes a workspace file or external system during the test.
- [x] Only the host writes the observed minutes artifact.
- [ ] User-facing copy does not claim hard sandbox isolation.

## Evidence log

Historical evidence below is retained for traceability and does not satisfy reset checkboxes for the current source.

- 2026-08-19 | Codex Desktop | RT-A1 | source f4f8140 | installed 0.1.0+codex.20260819105232 | PASS | Historical run: two configured members completed two ordered rounds; an explicit model failure eventually used host default after an ineffective same-target retry was corrected; a human interjection reached round two; targets were reused; the host wrote and verified a 97-line Markdown artifact. Collision behavior was not tested.
- 2026-08-19 | Codex Desktop | RT-A2 | source f4f8140 | installed 0.1.0+codex.20260819105232 | PASS | Historical run: natural-language invocation without a topic asked for one, a custom role was accepted, and setup cancellation spawned no members and wrote no additional minutes file.
- 2026-08-19 | Codex Desktop | RT-A1 | source f4f8140 | installed 0.1.0+codex.20260819105232 | NOT-APPLICABLE | Historical run: the active spawn surface exposed a finite model enum, so the no-enum branch was unavailable and remains unchecked.
- 2026-08-19 | Codex Desktop | RT-A3 | source 41bb4a8 | installed 0.1.0+codex.20260819114708 | PARTIAL | Historical behavior only: rejected a side-effect persona; explicit Luna failure used fresh `g3` host-default fallback; second round reused its target; hostile closing-tag data stayed data; collision wrote `-2` while preserving the original hash; in-flight termination exported a partial round. Old-source evidence does not satisfy current reset items.
- 2026-08-19 | Codex Desktop | RT-A4 | source 41bb4a8 | installed 0.1.0+codex.20260819114708 | PARTIAL | Historical behavior only: no-topic invocation asked for a topic, accepted a custom role, and setup cancellation wrote no file. Old-source evidence does not satisfy current reset items.
- 2026-08-19 | Codex CLI | RT-A5 | source 41bb4a8 | installed 0.1.0+codex.20260819114708 | FAIL | Pre-fix `codex debug prompt-input` listed both unnamespaced `roundtable` from the legacy standalone skill and plugin `roundtable:roundtable`, disproving the single-visible-skill assumption.
- 2026-08-19 | Codex CLI | RT-A6 | source 41bb4a8 | installed 0.1.0+codex.20260819114708 | FAIL | Pre-fix `codex exec '$roundtable ...'` loaded the legacy skill while `codex exec '$roundtable:roundtable ...'` loaded the installed plugin; unnamespaced plugin guidance was misrouted.
- 2026-08-19 | Codex CLI | RT-A7-DISCOVERY | source f972328 | installed 0.1.0+codex.20260819121435 | PASS | Fresh namespaced prompt inspection listed both the legacy unnamespaced entry and plugin `roundtable:roundtable`, then resolved the plugin from the exact current installed cache entry. No internal path or task identifier is recorded.
- 2026-08-19 | Codex Desktop | RT-A7 | source f972328 | installed 0.1.0+codex.20260819121435 | PASS | Namespaced inline topic completed an eight-member plain-chat roster using host default; the add action disappeared at eight, a ninth member was explicitly rejected, and setup cancellation spawned no member and wrote no file.
- 2026-08-19 | Codex Desktop | RT-A8 | source f972328 | installed 0.1.0+codex.20260819121435 | PASS | Sanitized parent-event evidence showed first spawn `member_1_g1` with `fork_turns: "none"`; ambiguous stop interrupted immediately, listed the non-speaker, offered cancel/terminate, ignored late output, and cancel wrote no export. A second namespaced invocation used fresh `member_1_g2` with no-history, completed a real subagent with live commentary, full transcript, and neutral summary; termination produced a host-written 69-line Markdown artifact with required sections.
- 2026-08-19 | Codex Desktop | RT-A9 | source f972328 | installed 0.1.0+codex.20260819121435 | PASS | Third namespaced invocation selected explicit `gpt-5.6-luna`: generation 3 used no-history and failed, then fresh generation 4 used no-history with no model override and succeeded as host default. Round two reused the successful target through follow-up. Exact closing tags in persona, interjection, and prior contribution remained entity text, and the interjection propagated to round two. Termination created the `-2` collision artifact with 102 lines while the original file hash remained unchanged.
