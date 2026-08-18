# Codex Roundtable Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build and locally validate an independent, MIT-licensed, skills-only `roundtable` plugin that runs guided, ordered, multi-round discussions with real Codex subagents and exports Markdown minutes.

**Architecture:** The repository root is the distributable plugin root. A compact `SKILL.md` orchestrates the workflow, while `setup-wizard.md` owns configuration rules and `minutes-format.md` owns export rules. Codex-native subagent tools provide member execution; no MCP server, custom UI, hosted service, or state script is introduced.

**Tech Stack:** Codex plugin manifest JSON, Agent Skills Markdown, `agents/openai.yaml`, Git, bundled Codex plugin/skill validators, and manual fresh-task acceptance checks.

## Global Constraints

- Repository name: `codex-roundtable`.
- Plugin name and skill name: `roundtable`.
- Initial version: `0.1.0`.
- License: MIT, copyright 2026 NeoMei.
- Plugin shape: skills only; omit `mcpServers` and `apps`.
- Runtime dependencies: Codex-native subagent and filesystem tools only.
- Member count: one to eight; run one member at a time in roster order.
- Model selection: offer only model identifiers enumerated by the active spawn tool; otherwise offer inherited default only.
- Never accept or invent a free-form model identifier.
- Member agents are analysis-only and may perform only read-only investigation when needed.
- The host is the only agent allowed to write meeting artifacts.
- Do not promise a permanent main-task message per member or hard sandbox isolation.
- Restart recovery is best-effort and is not a v1 acceptance requirement.
- Do not silently overwrite, delete, or modify an existing same-name standalone skill.
- Do not create or edit marketplace metadata by hand; use the bundled plugin-creator workflow.
- No implementation scripts are shipped in v1.

---

## File Map

- `.codex-plugin/plugin.json`: plugin identity, version, discovery path, and presentation metadata.
- `skills/roundtable/SKILL.md`: trigger boundary, phase routing, subagent orchestration, failure policy, and termination behavior.
- `skills/roundtable/agents/openai.yaml`: skill display metadata and implicit-invocation policy.
- `skills/roundtable/references/setup-wizard.md`: complete topic/member/persona/model setup flow and card/plain-chat fallback.
- `skills/roundtable/references/minutes-format.md`: canonical summaries, filename sanitation, collision behavior, and Markdown template.
- `README.md`: scope, installation, legacy-skill migration, usage, limitations, and development validation.
- `LICENSE`: MIT license.
- `tests/acceptance.md`: manual, evidence-bearing fresh-task acceptance matrix.

---

### Task 1: Scaffold the skills-only plugin

**Files:**
- Create: `.codex-plugin/plugin.json`
- Create: `skills/roundtable/`
- Create: `LICENSE`
- Modify: `docs/superpowers/specs/2026-08-19-codex-roundtable-design.md`

**Interfaces:**
- Consumes: approved plugin name `roundtable`, repository owner `NeoMei`, version `0.1.0`.
- Produces: a validator-readable plugin root whose `skills` path is exactly `./skills/`.

- [ ] **Step 1: Verify the unimplemented plugin fails structural validation**

Run:

```bash
python3 /Users/neomei/.codex/skills/.system/plugin-creator/scripts/validate_plugin.py .
```

Expected: non-zero exit reporting that `.codex-plugin/plugin.json` is missing.

- [ ] **Step 2: Generate the canonical baseline with the bundled scaffold**

Run:

```bash
plugin_scaffold_dir=$(mktemp -d /tmp/codex-roundtable.XXXXXX)
python3 /Users/neomei/.codex/skills/.system/plugin-creator/scripts/create_basic_plugin.py \
  roundtable \
  --path "$plugin_scaffold_dir" \
  --with-skills
cp -R "$plugin_scaffold_dir/roundtable/.codex-plugin" .
mkdir -p skills/roundtable/references skills/roundtable/agents
```

Expected: `.codex-plugin/plugin.json` and `skills/` exist in the repository root. Do not retain the temporary directory in Git.

- [ ] **Step 3: Replace the generated manifest with release metadata**

Use `apply_patch` to make `.codex-plugin/plugin.json` exactly:

```json
{
  "name": "roundtable",
  "version": "0.1.0",
  "description": "Run guided multi-agent roundtable discussions with ordered speakers and Markdown minutes.",
  "author": {
    "name": "NeoMei",
    "url": "https://github.com/NeoMei"
  },
  "homepage": "https://github.com/NeoMei/codex-roundtable",
  "repository": "https://github.com/NeoMei/codex-roundtable",
  "license": "MIT",
  "keywords": ["roundtable", "multi-agent", "discussion", "meeting-minutes"],
  "skills": "./skills/",
  "interface": {
    "displayName": "Roundtable",
    "shortDescription": "Run guided multi-agent roundtable discussions.",
    "longDescription": "Configure expert roles and models, run them in a fixed speaking order across multiple rounds, and export a structured Markdown record.",
    "developerName": "NeoMei",
    "category": "Productivity",
    "capabilities": ["Multi-agent", "Write"],
    "websiteURL": "https://github.com/NeoMei/codex-roundtable",
    "defaultPrompt": ["Start a roundtable discussion and help me configure each member."],
    "brandColor": "#4F46E5"
  }
}
```

- [ ] **Step 4: Add the MIT license**

Use `apply_patch` to create `LICENSE` exactly:

```text
MIT License

Copyright (c) 2026 NeoMei

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

- [ ] **Step 5: Validate and commit the scaffold**

Run:

```bash
python3 -m json.tool .codex-plugin/plugin.json >/dev/null
python3 /Users/neomei/.codex/skills/.system/plugin-creator/scripts/validate_plugin.py .
git add .codex-plugin/plugin.json skills LICENSE docs/superpowers/specs/2026-08-19-codex-roundtable-design.md
git commit -m "chore: scaffold roundtable plugin"
```

Expected: JSON parsing succeeds. Plugin validation has no manifest-schema error; commit contains only scaffold files and the approved spec status.

---

### Task 2: Define the complete setup wizard

**Files:**
- Create: `skills/roundtable/references/setup-wizard.md`

**Interfaces:**
- Consumes: active structured-input tool availability and active `spawn_agent` model enum, when exposed.
- Produces: canonical in-memory roster records with `id`, `label`, `persona`, `model_mode`, and optional `model`.

- [ ] **Step 1: Verify the reference does not exist**

Run `test -f skills/roundtable/references/setup-wizard.md`.

Expected: exit 1.

- [ ] **Step 2: Create the setup wizard reference**

Use `apply_patch` to create `skills/roundtable/references/setup-wizard.md` exactly:

```markdown
# Roundtable Setup Wizard

Use this reference only while creating or changing a roundtable roster.

## Interaction mode

- Ask one question at a time.
- Use a structured input tool only when it is available and can represent the complete current choice set.
- If a card would omit valid choices, use plain chat with a numbered list instead.
- After every answer, echo the accepted value before asking the next question.
- Never start a member agent until the user confirms the complete roster.

## Topic

If the opening request already contains a topic, confirm it. Otherwise ask for it. When a task-title tool is available, rename the current task to the confirmed topic.

Echo: `Topic confirmed: <topic>`

## Add members

Add members one at a time. The roster must contain at least one and at most eight members.

For each member:

1. Recommend two or three topic-relevant roles and allow a custom role.
2. Propose a concise default persona and allow the user to edit it.
3. Offer a model choice using the rules below.
4. Echo the completed member record.
5. Ask whether to add another member. At eight, move to roster confirmation.

Use stable IDs `member-1` through `member-8`; labels need not be unique, but IDs do.

Canonical member record:

```yaml
id: member-1
label: Architecture expert
persona: Focus on boundaries, operability, and long-term maintenance.
model_mode: inherit
model: null
effective_model: inherited default
agent_target: null
```

Before execution, `effective_model` and `agent_target` are provisional. Update them only from observed runtime behavior.

Echo accepted fields as `Role confirmed`, `Persona confirmed`, `Model confirmed`, and finally `Member added` with the full accepted values.

## Model choices

`Inherited default` is always the first and recommended choice.

Offer explicit models only when the active member-spawn tool exposes a finite list of accepted model override values. Copy those identifiers exactly. Do not infer aliases, add providers from memory, or accept a free-form model identifier.

If the active tool exposes no model enum, offer only `Inherited default` and explain that the surface does not expose portable per-member discovery. If a card cannot display the complete enum, show the complete numbered list in plain chat.

Store inherited choice as `model_mode: inherit`, `model: null`; store explicit choice as `model_mode: explicit`, `model: <exact enum value>`.

## Roster confirmation

Show the complete ordered roster with role, persona, and configured model. Ask for one action: start, edit, remove, add if below eight, or cancel. Apply changes and show the full roster again. Start only after explicit confirmation.

## Setup cancellation

If the user cancels before execution, do not spawn agents and do not create a minutes file. Return the confirmed topic and roster draft in chat.
```

- [ ] **Step 3: Verify and commit the wizard**

Run:

```bash
rg -n '^## (Interaction mode|Topic|Add members|Model choices|Roster confirmation|Setup cancellation)$' skills/roundtable/references/setup-wizard.md
git add skills/roundtable/references/setup-wizard.md
git commit -m "feat: define roundtable setup wizard"
```

Expected: all six headings are reported before the commit succeeds.

---

### Task 3: Define the minutes and export contract

**Files:**
- Create: `skills/roundtable/references/minutes-format.md`

**Interfaces:**
- Consumes: confirmed topic, ordered roster, effective model labels, completed or partial rounds, user interjections, and final host synthesis.
- Produces: verified path `roundtable-minutes/<topic-slug>-YYYY-MM-DD.md` or complete fallback Markdown in chat.

- [ ] **Step 1: Verify the reference does not exist**

Run `test -f skills/roundtable/references/minutes-format.md`.

Expected: exit 1.

- [ ] **Step 2: Create the minutes-format reference**

Use `apply_patch` to create `skills/roundtable/references/minutes-format.md` exactly:

````markdown
# Roundtable Minutes Format

Use this reference when summarizing a round, terminating a discussion, or recovering enough visible state to continue.

## Canonical round record

Keep these fields in the main task context after every round:

- round number and topic;
- ordered roster with effective model labels;
- completed member contributions in speaking order;
- skipped or failed members;
- user interjections;
- principal positions;
- agreements;
- disagreements and trade-offs;
- risks and assumptions;
- unresolved questions;
- recommended next-round focus.

At the end of each round, include the ordered member transcript and high-level host summary in the main response. Live commentary is helpful but is not the durable record.

## Output path

Write completed minutes beneath the current workspace:

`roundtable-minutes/<topic-slug>-YYYY-MM-DD.md`

Build `<topic-slug>` by trimming whitespace, replacing whitespace runs with `-`, removing path separators, control characters and reserved punctuation, preserving readable Chinese and Unicode letters, limiting to 64 Unicode code points, and using `roundtable` if empty.

Use the current local date. Never overwrite an existing file. If the base path exists, append `-2`, `-3`, and so on until an unused path is found.

## Markdown template

```markdown
# <topic> Roundtable Minutes

## Participants

- **<role>** — <effective model label>
- **Meeting host** — neutral facilitator

## Round 1

**Focus:** <round topic>

### Summary

<high-level positions, agreements, disagreements, risks, and unresolved questions>

### Human input

<user interjections, or "None">

### Participation

- Completed: <roles>
- Skipped or failed: <roles, or "None">

## Round 2

<repeat the same structure>

## Final Synthesis

### Recommendation

<concrete recommended approach>

### Rationale

<how the rounds support the recommendation>

### Agreements and Disagreements

<resolved and unresolved trade-offs>

### Risks and Assumptions

<risks, assumptions, and mitigations>

### Open Questions

<remaining decisions, or "None">

### Next Actions

<ordered, actionable next steps>
```

Do not copy full member transcripts into the file unless the user asks for a transcript appendix. The main task history remains the transcript source.

## Partial rounds

Label an interrupted round as partial. Summarize only completed contributions and list every member who did not speak. Do not invent missing positions.

## Write and verify

The main host is the only writer. Create the directory and file with the available filesystem editing tool, then verify that the exact reported path exists and contains the topic, participant list, every completed round, and final synthesis.

If the workspace is unavailable or the write fails, return the complete Markdown in chat, state that no file was written, and do not report a nonexistent path.
````

- [ ] **Step 3: Verify and commit the export contract**

Run:

```bash
rg -n '^## (Canonical round record|Output path|Markdown template|Partial rounds|Write and verify)$' skills/roundtable/references/minutes-format.md
rg -n 'Never overwrite|do not report a nonexistent path|main host is the only writer' skills/roundtable/references/minutes-format.md
git add skills/roundtable/references/minutes-format.md
git commit -m "feat: define roundtable minutes format"
```

Expected: all five headings and three safety statements are reported before commit.

---

### Task 4: Implement the Codex-native orchestration skill

**Files:**
- Create: `skills/roundtable/SKILL.md`
- Create: `skills/roundtable/agents/openai.yaml`

**Interfaces:**
- Consumes: setup roster from `setup-wizard.md`; active `spawn_agent`, `wait_agent`, `followup_task`, task-title, and filesystem capabilities when available.
- Produces: ordered member contributions, canonical round summaries, reusable member targets where available, and final minutes through `minutes-format.md`.

- [ ] **Step 1: Verify the skill entrypoint is absent or scaffold-only**

Run:

```bash
test -f skills/roundtable/SKILL.md && python3 /Users/neomei/.codex/skills/.system/skill-creator/scripts/quick_validate.py skills/roundtable
```

Expected: non-zero exit because the final entrypoint is absent or still scaffold content.

- [ ] **Step 2: Create the main skill**

Use `apply_patch` to make `skills/roundtable/SKILL.md` exactly:

````markdown
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
````

- [ ] **Step 3: Create skill UI metadata**

Use `apply_patch` to create `skills/roundtable/agents/openai.yaml` exactly:

```yaml
interface:
  display_name: "Roundtable"
  short_description: "Configure expert agents, run an ordered discussion, and export minutes."
  default_prompt: "Start a roundtable discussion and guide me through configuring each member."

policy:
  allow_implicit_invocation: true
```

- [ ] **Step 4: Validate and commit the complete skill**

Run:

```bash
python3 /Users/neomei/.codex/skills/.system/skill-creator/scripts/quick_validate.py skills/roundtable
if rg -n 'ask_user_question|roundtable_models|roundtable_title|use the roundtable tool' skills/roundtable; then exit 1; fi
python3 /Users/neomei/.codex/skills/.system/plugin-creator/scripts/validate_plugin.py .
git add skills/roundtable
git commit -m "feat: add Codex-native roundtable orchestration"
```

Expected: both validators succeed, the forbidden legacy-tool scan produces no matches, and the commit contains only the skill bundle.

---

### Task 5: Document installation, migration, and acceptance

**Files:**
- Create: `README.md`
- Create: `tests/acceptance.md`

**Interfaces:**
- Consumes: repository-root plugin, personal marketplace convention, and possible same-name legacy skill.
- Produces: non-destructive migration instructions and a repeatable manual evidence format.

- [ ] **Step 1: Verify documentation is absent**

Run `test -f README.md || test -f tests/acceptance.md`.

Expected: exit 1.

- [ ] **Step 2: Create README.md**

Use `apply_patch` to create `README.md` exactly:

````markdown
# codex-roundtable

A skills-only Codex plugin for guided multi-agent roundtable discussions. Configure each member's role, persona, and available model; run speakers in a fixed order over multiple rounds; then export structured Markdown minutes.

Inspired by [NeoMei/dsh-roundtable](https://github.com/NeoMei/dsh-roundtable), rewritten for Codex-native subagents without DeepSeek Harness dependencies.

## Features

- Complete topic and member setup wizard.
- One real Codex subagent per member execution.
- Runtime-safe model selection with inherited-model fallback.
- Ordered multi-round discussion and user interjections.
- Neutral host summaries and verified Markdown export.
- Plain-chat fallback when structured input cards are unavailable.

## Requirements

- A current Codex release with subagents enabled.
- A writable workspace to save minutes. Without one, the plugin returns Markdown in chat.
- Installed models and permissions are determined by the active Codex host.

## Install for local development

This repository root is the distributable plugin root. Create a personal marketplace entry and managed copy, then synchronize this checkout into it:

```bash
plugin_creator_root="${CODEX_HOME:-$HOME/.codex}/skills/.system/plugin-creator"
python3 "$plugin_creator_root/scripts/create_basic_plugin.py" roundtable --with-skills --with-marketplace
rsync -a --exclude '.git/' --exclude 'docs/superpowers/' ./ "$HOME/plugins/roundtable/"
python3 "$plugin_creator_root/scripts/validate_plugin.py" "$HOME/plugins/roundtable"
```

Treat `~/plugins/roundtable` as generated installation state; source changes belong in this checkout. Do not hand-edit marketplace JSON.

Before installing, run the legacy-skill check below.

### Legacy skill migration

An older standalone `roundtable` skill may still depend on DeepSeek Harness tools. Check:

```bash
legacy_roundtable_skill="$HOME/.agents/skills/roundtable/SKILL.md"
if test -f "$legacy_roundtable_skill"; then
  rg -n 'roundtable_models|roundtable_title|ask_user_question' "$legacy_roundtable_skill"
fi
```

If matches appear, disable that exact skill path in `~/.codex/config.toml` or move it outside Codex skill roots before installing this plugin. Do not overwrite or delete it silently.

Example disable entry:

```toml
[[skills.config]]
path = "/absolute/path/to/the/legacy/roundtable/SKILL.md"
enabled = false
```

Restart or refresh Codex after changing skill configuration. In a fresh task, verify that `$roundtable` resolves to this plugin's skill.

## Usage

Explicit:

```text
$roundtable Discuss whether we should split this service into independent deployments.
```

Natural language:

```text
圆桌讨论：这个产品是否应该转向企业市场？
```

Member model choices are limited to identifiers explicitly exposed by the active Codex spawn tool; otherwise members inherit the current task model.

## Output

Completed minutes are written to `roundtable-minutes/<topic-slug>-YYYY-MM-DD.md`. Existing files are never overwritten; numeric suffixes are added on collisions.

## Limitations

- Member analysis-only behavior is an instruction boundary, not a separate hard sandbox.
- Codex may consolidate member output in the main response; inspectable subagent threads remain the original source.
- App-restart and in-flight request recovery are best-effort.
- Host-specific model failure branches may be unavailable to test on every installation.

## Development validation

```bash
python3 /Users/neomei/.codex/skills/.system/skill-creator/scripts/quick_validate.py skills/roundtable
python3 /Users/neomei/.codex/skills/.system/plugin-creator/scripts/validate_plugin.py .
```

Run [tests/acceptance.md](tests/acceptance.md) in a fresh task before publishing.

## License

MIT
````

- [ ] **Step 3: Create the acceptance checklist**

Use `apply_patch` to create `tests/acceptance.md` exactly:

```markdown
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
```

- [ ] **Step 4: Verify and commit documentation**

Run:

```bash
rg -n '^## (Features|Requirements|Install for local development|Usage|Output|Limitations|Development validation|License)$' README.md
rg -n '^## (Structural checks|Fresh-task behavior|Permission review|Evidence log)$' tests/acceptance.md
rg -n 'Do not overwrite or delete it silently|only the intended Codex-native' README.md tests/acceptance.md
git add README.md tests/acceptance.md
git commit -m "docs: add installation and acceptance guidance"
```

Expected: all required headings and migration safeguards are reported before commit.

---

### Task 6: Validate, install a managed personal copy, and run acceptance

**Files:**
- Modify after testing: `tests/acceptance.md`
- External managed copy: `/Users/neomei/plugins/roundtable/`
- External marketplace metadata: managed only by the bundled plugin-creator workflow.

**Interfaces:**
- Consumes: complete repository-root plugin and user choice for handling the detected legacy standalone skill.
- Produces: validated managed personal plugin, fresh-task acceptance evidence, and a clean source worktree.

- [ ] **Step 1: Run the complete source validation suite**

Run:

```bash
python3 -m json.tool .codex-plugin/plugin.json >/dev/null
python3 /Users/neomei/.codex/skills/.system/skill-creator/scripts/quick_validate.py skills/roundtable
python3 /Users/neomei/.codex/skills/.system/plugin-creator/scripts/validate_plugin.py .
if rg -n 'ask_user_question|roundtable_models|roundtable_title|use the roundtable tool' skills/roundtable; then exit 1; fi
git diff --check
```

Expected: validators exit 0, forbidden-tool scan has no matches, and `git diff --check` is clean.

- [ ] **Step 2: Run the non-destructive legacy preflight**

Run:

```bash
legacy_roundtable_skill=/Users/neomei/.agents/skills/roundtable/SKILL.md
if test -f "$legacy_roundtable_skill"; then
  printf 'legacy skill found: %s\n' "$legacy_roundtable_skill"
  rg -n 'roundtable_models|roundtable_title|ask_user_question' "$legacy_roundtable_skill" || true
else
  printf 'no legacy roundtable skill found\n'
fi
```

Expected on this machine: report the legacy skill and DSH-only references. Stop and ask the user to choose whether to disable or move that exact file. Do not modify it before the user chooses.

- [ ] **Step 3: After conflict resolution, create the personal marketplace entry**

Run:

```bash
python3 /Users/neomei/.codex/skills/.system/plugin-creator/scripts/create_basic_plugin.py \
  roundtable \
  --with-skills \
  --with-marketplace
```

Expected: `/Users/neomei/plugins/roundtable/` and the standard personal marketplace entry exist. If an entry exists, do not use `--force` blindly; inspect it and use the documented update flow.

- [ ] **Step 4: Refresh and validate the managed copy**

Run:

```bash
rsync -a \
  --exclude '.git/' \
  --exclude 'docs/superpowers/' \
  /Users/neomei/项目/codexprojects/roundtable/ \
  /Users/neomei/plugins/roundtable/
python3 /Users/neomei/.codex/skills/.system/plugin-creator/scripts/validate_plugin.py /Users/neomei/plugins/roundtable
python3 /Users/neomei/.codex/skills/.system/plugin-creator/scripts/update_plugin_cachebuster.py /Users/neomei/plugins/roundtable
```

Expected: installed-copy validation and cachebuster update succeed. Do not hand-edit marketplace JSON.

- [ ] **Step 5: Run the fresh-task acceptance checkpoint**

Ask the user to open a fresh Codex task after refresh and begin with:

```text
$roundtable 讨论是否应该将当前单体服务拆分为多个独立部署单元
```

Follow `tests/acceptance.md`. Record only non-sensitive PASS/FAIL/NOT-APPLICABLE evidence. This checkpoint requires user interaction; do not infer behavioral acceptance from structural validation.

- [ ] **Step 6: Commit sanitized acceptance evidence**

```bash
git add tests/acceptance.md
git commit -m "test: record roundtable acceptance evidence"
```

Expected: commit contains no task content, credentials, private identifiers, or marketplace files.

- [ ] **Step 7: Run final verification**

Run:

```bash
python3 -m json.tool .codex-plugin/plugin.json >/dev/null
python3 /Users/neomei/.codex/skills/.system/skill-creator/scripts/quick_validate.py skills/roundtable
python3 /Users/neomei/.codex/skills/.system/plugin-creator/scripts/validate_plugin.py .
if rg -n 'ask_user_question|roundtable_models|roundtable_title|use the roundtable tool' skills/roundtable; then exit 1; fi
git diff --check
git status --short --branch
```

Expected: validators exit 0, forbidden scan has no matches, `git diff --check` is clean, and `git status` reports a clean `main` branch.

---

## Plan Self-Review Checklist

- Spec coverage: plugin structure, full wizard, dynamic model restrictions, ordered native subagents, multi-round continuation, commentary/transcript distinction, analysis-only boundary, restart limitation, export, legacy migration, managed marketplace copy, and fresh-task acceptance are each assigned to a task.
- Placeholder scan: the plan contains no unfinished implementation markers or unspecified test steps.
- Interface consistency: canonical roster fields are defined in Task 2 and consumed by Task 4; minutes inputs are defined in Task 3 and produced by Task 4; installation paths match Task 5 and Task 6.
- Scope: no MCP server, UI, state script, top-level member task, or public marketplace submission is introduced.
