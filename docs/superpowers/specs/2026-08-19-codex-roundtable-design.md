# Codex Roundtable Plugin Design

## Status

Approved for implementation planning on 2026-08-19 after design review.

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
├── tests/
│   └── acceptance.md
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
- runs members in roster order and prefers to reuse the same subagent in later
  rounds when the runtime still exposes that agent thread;
- loads the minutes reference only when summarizing or exporting;
- constrains member agents to analysis and read-only investigation, leaving the
  host as the only writer of meeting artifacts;
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

When a structured input tool is available and its option limit can represent the
current choice set, the skill uses it for choices and short inputs. Otherwise it
asks one concise plain-chat question at a time. The workflow must remain fully
usable without cards, and it must not truncate a model list merely to fit a card.

### 3. Select models

The wizard offers only model identifiers explicitly accepted by the current
agent-spawn tool when that tool exposes an enumerable model set. `Default (inherit
the current task model)` is always available. The plugin does not embed a provider
catalog because model availability differs by host and changes over time.

If the current surface does not expose an enumerable model set, the wizard offers
only the default inherited model. It does not accept a free-form model identifier:
an identifier outside the host tool schema may be rejected before an agent can
start and therefore cannot be portably validated by a skills-only plugin.

### 4. Run a round

Each member is represented by a real Codex subagent. In the first round, the main
agent creates and waits for one member at a time, in roster order. Sequential
execution keeps the active concurrency requirement at one member regardless of
roster size. A model override uses a self-contained member prompt and the smallest
supported history fork so the member does not depend on inheriting the main task's
entire transcript. Each member receives:

- the original discussion topic;
- its role and persona;
- prior-round summaries and user interjections;
- all earlier utterances from the current round;
- an instruction to provide one focused roundtable contribution;
- an analysis-only contract: read-only investigation is allowed when needed, but
  the member must not modify files, change external state, send messages, or take
  other side-effecting actions.

After a member completes, the main task emits an ordered progress/commentary
update as `[角色名]` followed by the text when the current surface supports live
updates. The member's subagent thread remains the authoritative inspectable source
when the surface exposes subagent activity. At the end of the round, the main
response includes an ordered transcript of all completed member contributions.
The plugin does not promise a separate permanent main-task message for every
member, because Codex may consolidate subagent results into one response. The main
agent does not paraphrase a member before the host summary.

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
prefers to reuse existing member subagents through follow-up tasks. If a member
thread is unavailable or the host's agent-thread capacity prevents reuse, the
skill creates a replacement member with the same self-contained role context. In
both cases, the prompt includes the canonical round summaries so continuity does
not depend on private subagent history.

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

The Codex task is the primary session record. At the end of each round, the main
response restates the canonical roster, effective model labels, round summaries,
and user interjections needed to continue. No hidden workspace state file is
created in the first release.

Within the same live task, the skill may reconstruct an unavailable subagent with
the same role, persona, effective model, topic, and prior summaries, and tells the
user that the member was rebuilt. Recovery after an app restart or runtime reset
is best-effort and is not a v1 acceptance requirement. The plugin does not claim
recovery of an in-flight request or a previous agent's private context.

## Failure Handling

### Model unavailable

If a selected model is accepted by the tool schema but cannot start, retry the
member once with the model override omitted so the host resolves its inherited
default. Announce the fallback. Record the effective model when the runtime
reports it; otherwise record `inherited default` rather than guessing a model ID.

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
- Every member prompt states that the member is analysis-only: it may use
  read-only investigation when needed, but it must not write files, change
  external state, send messages, or perform destructive actions.
- This is an instruction-level boundary, not a separate hard sandbox. The plugin
  must not claim stronger isolation than the host provides.
- The main host is the only agent allowed to write the meeting-minutes artifact.
- A member receives only the current customer/workspace context needed for the
  discussion.
- The skill does not send data to an external service beyond the models and tools
  already selected in Codex.
- The skill does not create new top-level Codex tasks; member work uses native
  subagent threads under the current task.
- File output stays inside the current workspace unless the user explicitly
  provides another destination.

## Installation and Legacy Migration

The existing DSH-oriented standalone skill uses the same `roundtable` name and is
not compatible with the Codex-native plugin. Before local installation, the README
and acceptance checklist require a preflight check for an existing
`~/.agents/skills/roundtable/SKILL.md` or another visible skill with the same name.

If a legacy skill is found:

- identify it by its references to DSH-only tools;
- ask the user to disable it through Codex skill configuration or move it out of
  the discovered skill roots;
- never overwrite, delete, or edit the legacy file silently;
- verify in a fresh task that only the intended Codex-native `roundtable` skill is
  selected by `$roundtable`.

The Git repository root is the distributable plugin root, but the personal
marketplace uses its conventional managed source location. Local testing therefore
creates or refreshes a generated personal-marketplace copy at
`~/plugins/roundtable` from the repository root; the marketplace entry points to
that managed copy rather than directly to the development checkout. Updates use
the plugin creator's cachebuster and reinstall flow instead of hand-editing
marketplace metadata.

The README documents the source-to-managed-copy installation command,
refresh/reinstall flow, legacy-skill migration, and a fresh-task smoke test.
Public marketplace submission remains separate from local development
installation.

## Validation and Testing

### Structural validation

- Validate `skills/roundtable` with the Codex skill validator.
- Validate the plugin root with the Codex plugin validator.
- Confirm the manifest contains neither `mcpServers` nor `apps`.
- Scan for unfinished scaffold placeholders.
- Confirm the main skill and both references contain no executable calls to the
  DSH-only tools.

`tests/acceptance.md` is a manual, evidence-bearing checklist. Each run records the
Codex surface, task identifier, date, effective model behavior, and observed
result. The first release does not claim an automated harness for host-level
subagent failures.

### Behavioral scenarios

Test the installed plugin in fresh Codex tasks with these scenarios:

1. Trigger with `圆桌讨论` and no topic.
2. Trigger with `$roundtable` and an inline topic.
3. Accept a recommended role and edit its persona.
4. Add a custom role and reach the logical eight-member limit while running only
   one member at a time.
5. On a host that exposes an explicit model enum, select an available non-default
   model.
6. On a host without an explicit model enum, verify that the wizard offers only
   the inherited default and does not solicit a free-form identifier.
7. Run a full round and verify fixed speaking order.
8. Interject between members and verify propagation.
9. Continue to a second round and verify either member reuse or an explicitly
   reported replacement with canonical context.
10. Where the test host exposes a schema-valid but unavailable model, verify the
    inherited-model retry. Otherwise record this branch as not applicable rather
    than fabricating a failure.
11. Interrupt or cancel a member run and exercise retry, skip, and terminate.
12. Verify ordered live commentary when supported, inspectable member threads,
    and the consolidated ordered transcript in the round response.
13. Terminate and verify the Markdown file contents and reported path.
14. Run on a surface without structured input cards and complete the plain-chat
    fallback.
15. Run the legacy-skill preflight and verify that a duplicate skill is reported
    without overwriting or deleting it.

### Acceptance criteria

The implementation is accepted when:

- it installs as a skills-only plugin in Codex;
- the installation preflight detects a legacy same-name skill, and a fresh task
  exposes only the intended Codex-native skill after the user resolves the
  conflict;
- both implicit Chinese trigger phrases and explicit `$roundtable` invocation
  activate the intended workflow;
- its executable skill instructions contain no calls to the DSH-only
  `ask_user_question`, `roundtable`, `roundtable_models`, or `roundtable_title`
  tools;
- each member is a real Codex subagent with the chosen persona and either a
  schema-exposed selected model or the inherited default;
- member agents follow the analysis-only contract and the plugin does not claim a
  hard isolation boundary;
- speaking order, multi-round continuation, model fallback, and user
  interjections behave as specified;
- every completed round includes an ordered transcript in the main response; no
  acceptance criterion requires one permanent main-task message per member;
- the exported minutes pass a format review and the output path exists;
- validation and applicable fresh-task installation tests pass, with unsupported
  host-specific failure branches recorded as not applicable.

## Release Strategy

The first release is an independent MIT-licensed GitHub repository and a local
skills-only plugin suitable for personal-marketplace testing. Public marketplace
submission is a later release step after representative behavioral tests pass.
