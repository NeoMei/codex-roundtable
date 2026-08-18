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
