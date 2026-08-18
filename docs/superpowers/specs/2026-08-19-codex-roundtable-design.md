# Codex Roundtable Plugin Design

## Status

Approved for planning on 2026-08-19.

## Goal

Create an independent `codex-roundtable` repository containing an installable,
skills-only Codex plugin. The plugin recreates the useful behavior of
`dsh-roundtable` with Codex-native subagents and task tools, without depending on
DeepSeek Harness, an MCP server, a hosted service, or custom UI.

The plugin must support a guided multi-round discussion in which the user chooses
each member's role, persona, and model; members speak in a fixed order; a neutral
host summarizes each round; and the completed discussion is exported as Markdown.

## Non-goals

- Port the DeepSeek Harness TypeScript host engine or Cordis services.
- Add a native Codex sidebar button or modify the Codex desktop shell.
- Build an MCP server, external persistence service, or custom MCP App UI.
- Maintain a hard-coded global catalog of providers and models.
- Guarantee recovery of an in-flight model request after the Codex process exits.

## Repository and Plugin Shape

The GitHub repository is named `codex-roundtable`. The plugin and its single skill
are both named `roundtable`. The project uses the MIT license, matching the source
project.

```text
codex-roundtable/
├── .codex-plugin/
│   └── plugin.json
├── skills/
│   └── roundtable/
│       ├── SKILL.md
│       ├── agents/
│       │   └── openai.yaml
│       └── references/
│           ├── setup-wizard.md
│           └── minutes-format.md
├── docs/
│   └── superpowers/
│       └── specs/
├── README.md
└── LICENSE
```

The first release contains no scripts. Instructions and existing Codex tools are
sufficient, and avoiding a state-management script keeps the workflow portable
across Codex installations.

## Component Responsibilities

### Plugin manifest

`.codex-plugin/plugin.json` declares a skills-only plugin named `roundtable` and
points to `./skills/`. It contains valid semver and presentation metadata but no
`mcpServers` or `apps` fields.

### Main skill

`skills/roundtable/SKILL.md` is the compact router and orchestration contract. It:

- activates for explicit `$roundtable` use and clear phrases such as
  `圆桌讨论` or `圆桌会议`;
- establishes the fixed, neutral meeting host;
- loads the setup reference while configuring the meeting;
- authorizes and requires Codex-native subagent delegation for each configured
  member;
- runs members in roster order and reuses the same subagent in later rounds;
- loads the minutes reference only when summarizing or exporting;
- keeps ordinary, unrelated requests outside its trigger boundary.

### Setup wizard reference

`setup-wizard.md` defines the complete member-by-member setup flow, input
validation, runtime model selection, confirmation echoes, and fallbacks for Codex
surfaces that do not expose structured input cards.

### Minutes format reference

`minutes-format.md` defines the round summary, final synthesis, filename rules,
and the distinction between visible member utterances and the high-level record
written to disk.

### UI metadata

`agents/openai.yaml` provides the desktop-facing display name, description, and
starter prompt. Implicit invocation remains enabled because the skill has narrow,
unambiguous trigger language.

## User Flow

### 1. Start and topic

The skill starts when the user explicitly invokes `$roundtable` or clearly asks
for a roundtable discussion. If the request already contains a topic, the skill
uses it; otherwise it asks for one. Once known, the skill renames the current
Codex task to the topic when the task-title tool is available.

### 2. Configure members

Members are added one at a time. For each member, the skill collects:

1. role label;
2. editable persona;
3. model choice;
4. whether another member should be added.

The skill recommends topic-relevant roles and personas but always permits custom
text. The final roster contains between one and eight members. After every answer,
the skill echoes the accepted value so the user can detect mistakes immediately.
The user confirms the complete roster before any subagent is started.

When a structured input tool is available, the skill uses it for choices and
short inputs. Otherwise it asks one concise plain-chat question at a time. The
workflow must remain fully usable without cards.

### 3. Select models

The wizard offers only model choices exposed by the current Codex runtime or
agent-spawn tool. `Default (current task model)` is always available. The plugin
does not embed a provider catalog because model availability differs by host and
changes over time.

If model enumeration is not available, the wizard offers the current model and
allows a custom model identifier. A model identifier is not treated as validated
until the member starts successfully.

### 4. Run a round

Each member is represented by a real Codex subagent. In the first round, the main
agent creates and waits for one member at a time, in roster order. Each member
receives:

- the original discussion topic;
- its role and persona;
- prior-round summaries and user interjections;
- all earlier utterances from the current round;
- an instruction to provide one focused roundtable contribution.

After a member completes, the main task presents the utterance as
`[角色名]` followed by the text. Subagent activity may also remain inspectable in
the Codex interface. The main agent does not paraphrase a member before the host
summary.

### 5. Summarize and continue

After every configured member has spoken, the neutral host produces a round
summary containing:

- the principal positions;
- agreements;
- disagreements and trade-offs;
- risks and assumptions;
- unresolved questions;
- a recommended focus for the next round.

The user then chooses `continue` or `terminate` and may add an opinion. Continuing
reuses the existing member subagents through follow-up tasks so their role context
is preserved. The follow-up also includes the canonical round summaries so the
workflow remains understandable if a member's local context is compacted.

### 6. Terminate and export

On termination, the skill writes:

```text
roundtable-minutes/<topic-slug>-YYYY-MM-DD.md
```

The file includes the topic, attendee roster with model labels, a concise section
for every round, and a detailed final synthesis. Member transcripts remain in the
task history and are not copied verbatim into the minutes unless the user asks for
an appendix.

## Context and Recovery

The Codex task is the primary session record. The main agent retains the roster,
round summaries, user interjections, and subagent identifiers in task context.
No hidden workspace state file is created in the first release.

If an existing subagent can no longer be continued after an app restart or runtime
reset, the skill reconstructs a new subagent with the same role, persona, selected
model, topic, and prior summaries. It tells the user that the member was rebuilt.
This restores semantic continuity but does not claim byte-for-byte recovery of
the previous agent's private context.

## Failure Handling

### Model unavailable

If a selected model cannot start, retry the member once using the current task
model. Announce the fallback and record the actual model label in subsequent
summaries and final minutes.

### Member failure

If the fallback attempt also fails, pause the round and ask the user to retry,
skip that member, or terminate. Never convert an empty, cancelled, or failed
result into a normal contribution.

### User interjection

If the user sends a message while a round is running, preserve it as an explicit
human interjection. Incorporate it into the next member prompt when safe; if the
current member has already completed, incorporate it into the next member or next
round. Do not silently discard it.

### Partial round

If a round terminates early, the host labels it as partial and summarizes only
completed contributions. The minutes state which members did not speak.

### Export failure

If the workspace is unavailable or the file write fails, return the complete
Markdown in the final response and explain that no file was written. Do not claim
an output path that was not verified.

## Security and Permission Boundaries

- Subagents inherit the parent task's permission mode.
- A member receives only the current customer/workspace context needed for the
  discussion.
- The skill does not send data to an external service beyond the models and tools
  already selected in Codex.
- The skill does not create new top-level Codex tasks; member work uses native
  subagent threads under the current task.
- File output stays inside the current workspace unless the user explicitly
  provides another destination.

## Validation and Testing

### Structural validation

- Validate `skills/roundtable` with the Codex skill validator.
- Validate the plugin root with the Codex plugin validator.
- Confirm the manifest contains neither `mcpServers` nor `apps`.
- Scan for unfinished scaffold placeholders.

### Behavioral scenarios

Test the installed plugin in fresh Codex tasks with these scenarios:

1. Trigger with `圆桌讨论` and no topic.
2. Trigger with `$roundtable` and an inline topic.
3. Accept a recommended role and edit its persona.
4. Add a custom role and reach the eight-member limit.
5. Select an available non-default model.
6. Select an invalid model and verify fallback to the current model.
7. Run a full round and verify fixed speaking order.
8. Interject between members and verify propagation.
9. Continue to a second round and verify member reuse.
10. Simulate a member failure and exercise retry, skip, and terminate.
11. Terminate and verify the Markdown file contents and reported path.
12. Run on a surface without structured input cards and complete the plain-chat
    fallback.

### Acceptance criteria

The implementation is accepted when:

- it installs as a skills-only plugin in Codex;
- both implicit Chinese trigger phrases and explicit `$roundtable` invocation
  activate the intended workflow;
- its executable skill instructions contain no calls to the DSH-only
  `ask_user_question`, `roundtable`, `roundtable_models`, or `roundtable_title`
  tools;
- each member is a real Codex subagent with the chosen persona and effective
  model;
- speaking order, multi-round continuation, model fallback, and user
  interjections behave as specified;
- the exported minutes pass a format review and the output path exists;
- validation and fresh-task installation tests pass.

## Release Strategy

The first release is an independent MIT-licensed GitHub repository and a local
skills-only plugin suitable for personal-marketplace testing. Public marketplace
submission is a later release step after representative behavioral tests pass.
