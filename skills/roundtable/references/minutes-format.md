# Roundtable Minutes Format

Use this reference when summarizing a round, terminating a discussion, or recovering enough visible state to continue.

## Canonical round record

Keep these fields in the main task context after every round:

- round number and topic;
- ordered roster with effective model labels;
- successful runtime model policy and current target generation for each member;
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

Before a successful spawn, the effective model is unknown and remains null in canonical state. After success, use the exact enum for an explicit runtime policy and `host default` for a host-default runtime policy. Never infer an identifier from the parent session.

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

Cancellation exits after recording this partial round in the task and does not create an export. Termination includes the partial round in the exported minutes.

## Write and verify

The main host is the only writer. Create the directory and file with the available filesystem editing tool, then verify that the exact reported path exists and contains the topic, participant list, every completed round, and final synthesis.

If the workspace is unavailable or the write fails, return the complete Markdown in chat, state that no file was written, and do not report a nonexistent path.
