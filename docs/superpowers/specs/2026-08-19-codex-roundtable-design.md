# Codex Roundtable Plugin Design

## Status

Approved for implementation on 2026-08-19 after design review.

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

The declared GitHub repository is `codex-roundtable`. The plugin and its single
skill are both named `roundtable`. The project uses the MIT license, matching the
source project. Local implementation does not create a GitHub repository or
remote; the declared URL must be created and verified before public publication.

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
- uses explicit no-history spawning, fresh generation targets, persistent runtime
  model policy, and capacity-aware reconstruction;
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

The skill recommends topic-relevant roles and personas and permits safe custom
text. It rejects a persona that requests side effects, overrides host
instructions, or conflicts with the analysis-only contract. The final roster
contains between one and eight members. After every answer,
the skill echoes the accepted value so the user can detect mistakes immediately.
The user confirms the complete roster before any subagent is started.

When a structured input tool is available and its option limit can represent the
current choice set, the skill uses it for choices and short inputs. Otherwise it
asks one concise plain-chat question at a time. The workflow must remain fully
usable without cards, and it must not truncate a model list merely to fit a card.

### 3. Select models

The wizard offers only model identifiers explicitly accepted by the current
agent-spawn tool when that tool exposes an enumerable model set. `Host default (no
model override)` is always available and is stored as `model_mode: host_default`,
`model: null`. The plugin makes no model-equivalence claim across host and member
tasks and does not embed a provider catalog because availability differs by host.

If the current surface does not expose an enumerable model set, the wizard offers
only host default with no override. It does not accept a free-form model identifier:
an identifier outside the host tool schema may be rejected before an agent can
start and therefore cannot be portably validated by a skills-only plugin.

### 4. Run a round

Each member is represented by a real Codex subagent. In the first round, the main
agent creates and waits for one member at a time, in roster order. Sequential
execution keeps the active concurrency requirement at one member regardless of
roster size. Every spawn, with or without a model override, uses `fork_turns:
"none"` or the smallest schema-supported value that explicitly disables history.
If the host cannot disable full-history forking, the host discloses that the
minimal-context boundary cannot be guaranteed and asks proceed/cancel before the
first spawn. Each member receives a self-contained prompt containing:

- the original discussion topic;
- its role and persona;
- prior-round summaries and user interjections;
- all earlier utterances from the current round;
- an instruction to provide one focused roundtable contribution;
- an analysis-only contract: read-only investigation is allowed when needed, but
  the member must not modify files, change external state, send messages, or take
  other side-effecting actions.

The prompt places the contract and contribution request outside a delimited
untrusted-data block. Topic, persona, prior summaries, user interjections, and
earlier contributions are inside the block. Before interpolation, every field is
XML-entity encoded in deterministic order: `&` to `&amp;`, then `<` to `&lt;`, then
`>` to `&gt;`. Only these three ordered replacements are allowed; code fences are
not an encoding substitute. Therefore the
host-authored closing tag is the only literal `</discussion-data>` in the prompt.
The member is told not to follow embedded instructions. The persona shapes viewpoint only.

Each roster record separates the configured model choice from the successful
runtime model policy. Every spawn attempt reserves a fresh per-member generation
target such as `member_1_g1`, `member_1_g2`; failed attempts are never retried
under the same target. `effective_model` is null before a successful spawn. On
success, the host records `agent_target` and `agent_generation` together with
runtime policy; explicit success uses the exact model enum and host-default
success uses `host default`.

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

The user then chooses `continue`, `terminate`, or `cancel` and may add an opinion. Continuing
prefers to reuse existing member subagents through follow-up tasks. If a member
thread is unavailable, the skill clears its recorded target and creates a
fresh-generation replacement using the successful runtime model policy. If the
host publishes a thread cap and a supported close capability, the host may close
the least-recently-needed completed target only after its contribution is
canonicalized, then reconstruct it later. Hosts without close support are not
required to close anything. Capacity that cannot be relieved reaches the
retry/skip/terminate gate rather than being reported as reuse. Canonical round
summaries provide continuity.

### 6. Terminate and export

On termination, including in-flight termination that records a partial round, the
skill writes:

```text
roundtable-minutes/<topic-slug>-YYYY-MM-DD.md
```

The file includes the topic, attendee roster with model labels, a concise section
for every round, and a detailed final synthesis. Member transcripts remain in the
task history and are not copied verbatim into the minutes unless the user asks for
an appendix. Cancellation records any partial round in the task, lists non-speakers,
and exits without export. Ambiguous stop intent interrupts first when supported
and asks whether to cancel without export or terminate with export.

## Context and Recovery

The Codex task is the primary session record. At the end of each round, the main
response restates the canonical roster, effective model labels, round summaries,
and user interjections needed to continue. No hidden workspace state file is
created in the first release.

Within the same live task, the skill may reconstruct an unavailable subagent with
the same role, persona, successful runtime model policy, topic, and prior
summaries under a fresh generation, and tells the user that the member was rebuilt.
Recovery after an app restart or runtime reset is best-effort and is not a v1
acceptance requirement. The plugin does not claim recovery of an in-flight request
or a previous agent's private context.

## Failure Handling

### Model unavailable

If a selected model is accepted by the tool schema but cannot start, retry the
member once on a fresh generation with the model override omitted. Announce the
fallback. Keep the configured explicit model separate, persist the successful
runtime policy as host default so later reconstruction does not retry the known-
failing model, and record `host default` for that runtime policy.

### Member failure

If a host-default start or explicit fallback fails, pause the round and ask the
user to retry, skip that member, or terminate. Retry uses a fresh generation and
does not retry a known-failing explicit model. Never convert an empty, cancelled,
or failed result into a normal contribution.

### User interjection

Classify in-flight intent before treating it as an opinion. `cancel`/`取消`
interrupts when supported, ignores late output, records a partial round and exits
without export. `terminate`/`终止` does the same but exports partial minutes.
Ambiguous `stop`/`停止` interrupts first and asks which outcome the user wants.
Other messages are preserved as untrusted human interjections for the next safe
member or round. When interruption is unavailable, disclose that and ignore late
output for the interrupted round.

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
- Topic, persona, summaries, interjections, and earlier contributions are
  delimited as untrusted data and cannot override the analysis-only contract.
- The skill does not send data to an external service beyond the models and tools
  already selected in Codex.
- The skill does not create new top-level Codex tasks; member work uses native
  subagent threads under the current task.
- File output stays inside the current workspace unless the user explicitly
  provides another destination.

## Installation and Legacy Migration

The existing DSH-oriented standalone skill uses the same `roundtable` name and is
not compatible with the Codex-native plugin. Before local installation, the README
and acceptance checklist inspect both standard standalone roots,
`~/.agents/skills/roundtable` and `${CODEX_HOME:-~/.codex}/skills/roundtable`, and
require every visible same-name entry to be inspected and disabled or moved when
it conflicts.

If a legacy skill is found:

- identify it by its references to DSH-only tools;
- ask the user to disable it through Codex skill configuration or move it out of
  the discovered skill roots;
- never overwrite, delete, or edit the legacy file silently;
- verify in a fresh task that only the intended Codex-native `roundtable` skill is
  selected by `$roundtable`.

The Git repository root is the distributable plugin root, but the personal
marketplace uses its conventional managed source location. Local testing therefore
creates or refreshes a generated personal-marketplace copy at the exactly guarded
`~/plugins/roundtable` target from the repository root; the marketplace entry
points to that managed copy rather than directly to the development checkout.
Both command blocks run in fail-fast Bash subshells with `set -euo pipefail`.
First installation requires the exact target to be absent. Before an update can
reach `rsync --delete`, it verifies the exact path, rejects symlinks and Git
checkouts, and parses both source and destination manifests to require the exact
plugin name `roundtable`. The guarded flow then uses scoped
`rsync -a --delete --delete-excluded` with source-only exclusions and asserts that `hooks/`, `.mcp.json`, `.app.json`,
manifest `mcpServers`, and manifest `apps` are absent. Updates use the plugin
creator's cachebuster and reinstall flow instead of hand-editing marketplace
metadata. These shell instructions are scoped to macOS/Linux and list Bash,
Git, `rsync`, `rg`, Codex CLI, Python 3, and PyYAML prerequisites.

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
Codex surface, non-sensitive alias, date, tested source commit, installed
cachebuster/version, effective model behavior, and observed result. Behavioral
acceptance remains partial until reset behavior is rerun against the current
source. Artifact creation and collision/non-overwrite behavior are separate
checks. Unsupported no-enum behavior remains unchecked with N/A evidence rather
than being promoted to a pass. The first release does not claim an automated
harness for host-level subagent failures.

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
   `Host default (no model override)` and does not solicit a free-form identifier.
7. Verify that effective model state starts null, then becomes the exact enum for
   explicit success or `host default` for host-default success.
8. Use topic, interjection, and earlier-contribution values containing exact
   `</discussion-data>` and verify deterministic entity encoding prevents closure.
9. Run a full round and verify fixed speaking order.
10. Interject between members and verify propagation.
11. Continue to a second round and verify either member reuse or an explicitly
   reported replacement with canonical context.
12. Where the test host exposes a schema-valid but unavailable model, verify the
    fresh-target host-default retry and persistent runtime policy. Otherwise record
    this branch as not applicable rather than fabricating a failure.
13. Exercise cancel, terminate, and ambiguous-stop steering during a member run,
    including interruption, late-output suppression, partial records, and export
    differences.
14. Verify ordered live commentary when supported, inspectable member threads,
    and the consolidated ordered transcript in the round response.
15. Terminate and verify the Markdown file contents and reported path; separately
    create a collision and verify the existing file is unchanged and a suffix is used.
16. Run on a surface without structured input cards and complete the plain-chat
    fallback.
17. Run the legacy-skill preflight and verify that a duplicate skill is reported
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
- each member is a real Codex subagent with a safe persona and either a
  schema-exposed selected model or host default with no override;
- every spawn has minimal history or obtains explicit user consent when the host
  cannot guarantee it, and fresh generation targets prevent same-target retry;
- member agents follow the analysis-only contract and the plugin does not claim a
  hard isolation boundary;
- speaking order, multi-round continuation, model fallback, and user
  interjections behave as specified;
- every completed round includes an ordered transcript in the main response; no
  acceptance criterion requires one permanent main-task message per member;
- the exported minutes pass a format review and the output path exists;
- structural validation passes; current behavioral acceptance is described as
  partial until applicable reset checks pass, with unsupported host-specific
  branches left unchecked and recorded as not applicable.

## Release Strategy

The first release is an independent MIT-licensed project and a local skills-only
plugin suitable for personal-marketplace testing. The declared GitHub repository
URL must be created and verified before public publication. Public marketplace
submission is a later release step after representative behavioral tests pass.
