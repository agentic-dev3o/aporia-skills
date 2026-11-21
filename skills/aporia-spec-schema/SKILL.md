---
name: aporia-spec-schema
description: >-
  Draws the schema of an Aporia specification with the human: the lanes (who or
  what acts), the ordered steps of the flow, optional phases, and what leads to
  what. Reads the specification with aporia:get_spec, drafts the flow for the
  human to react to (guess and mark, never interrogate), validates it, then
  writes it with aporia:write_schema. Use for /aporia-spec-schema, or when
  asked to design, draw or map how a feature works, build the flow or the
  swimlane of a specification, or write the schema of a specification. To
  record a rule the flow revealed, use aporia-report.
---

# Aporia specification schema

A specification says **why** a feature exists (its intention) and **what must hold** (its criteria and rules). Its schema says **how it goes**: who does what, in what order. The Specification page draws it: one column per lane, the steps from top to bottom, every flow an arrow with its label. This skill writes that drawing, with the human.

You reach Aporia through the **MCP server only**, pinned to one product. Call its tools fully qualified, as `aporia:*`. If the tools are not available, stop and ask the user to connect the Aporia MCP server.

**A question to the human goes through the tool your agent has for asking one** (`AskUserQuestion` in Claude Code): one decision per question, with the answers you expect as its options. A question written in the middle of a message gets lost. Without such a tool, or when nobody is there to answer (a scheduled run), the questions close your message, numbered, one decision each.

**`aporia:write_schema` is not among your tools?** The Aporia server was added with `?tools=work`, the profile of a coding session, which leaves it out. Ask the human to add the server again without `?tools=work`, then start over.

## The hard lines

- **The human's flow, not yours.** You draft so they have something to correct. What they do not confirm stays marked as a guess, or stays out.
- **A yes covers what it answers.** "Yes, write it" accepts the drawing. It does not answer the questions you asked beside it: a gap with the code, a rule the flow revealed. Each stays open until it gets its own answer, and you keep saying so.
- **No yes, no write.** `aporia:write_schema` waits for a human who read the draft in this session and said yes. A request to draw the flow and write it is not that yes, because nobody had read the draft when it was made. When nobody is there to answer, the run ends on the draft and writes nothing to Aporia, no schema and no clause.
- **A schema is not a contract.** It shows the flow. What must always hold is a rule, and it goes through `aporia:add_clause` with the human's yes. Never hide a rule in a step's detail.
- **One schema per specification, replaced whole.** Writing over an existing one erases it: you read it first and you ask.

## The method: you propose, the human disposes

People correct a drawing faster than they describe one. So do not interview for ten questions. Read what exists, draw a first flow, mark every guess, show it as a drawing, and ask the human what is wrong.

## Workflow

```
Schema progress:
- [ ] Phase 0: load the context, read the specification
- [ ] Phase 1: draft the flow, mark every guess
- [ ] Phase 2: show the drawing, validate with the human
- [ ] Phase 3: write, only on the human's yes (aporia:write_schema)
- [ ] Phase 4: hand off
```

### Phase 0: ground

`aporia:get_context` once: the personas are your candidate lanes. The first write returns a `sessionId`: pass it on every later write.

`aporia:get_spec { spec }` with the key (`commerce.checkout`) or the number the human gave. They gave a name: find the specification with `aporia:list_specs`, and when several match, ask which one. Read:

- the **intention**: the flow serves it. Unsigned means a draft: say so;
- the **criteria** and the **rules**: the flow must not contradict one;
- the **schema**: `null`, or the drawing that already exists;
- the **personas** it names.

No specification yet? This skill never creates one. Ask the human to run `/aporia-onboard`, or to create the specification in Aporia, then resume here.

### Phase 1: draft

Read `references/shared/content-style.md` for the shape of every string.

- **`lanes`**: who or what acts. A persona, a part of the system, a third party. Each has an `id` and a `label` of one to three words, and is one column of the drawing. Two to five lanes read well.
- **`steps`**: what happens, in reading order: the page reads from top to bottom, and a step sits under the steps that lead to it. Each has an `id`, the `laneId` of who does it, and a `title` of two to five words, verb first. A `phaseId` puts it in a phase.
- **`phases`**: optional titled stages ("Before payment", "After payment") when the flow has real ones, each with an `id` and a `label`. A phase is a band across the lanes. Otherwise send `[]`.
- **`flows`**: what leads to what. `from` and `to` are step ids. Every flow is an arrow, and its `label` sits on the arrow: one or two words that name a branch ("valid", "refused"). When a step has more than one way out, label each. An end is a step with no flow out.

Three things the drawing depends on:

- **The lane is the subject of the title.** A step sits in the lane of who acts, in the active voice: "Reads the calendar" in the lane of the system, not "Is read" in the lane of the calendar. Someone who only receives still acts: "Receives the contact details".
- **`failure: true` marks the unhappy path**: a refusal, an error, a timeout, a conflict. The page draws these steps in another colour, to the right of the way that works. Leave it out on every other step.
- **A `detail` hides behind its step.** It is one sentence, only when the title is not enough, and it opens when the reader points at the step. The drawing must read without it.

At most 12 lanes, 12 phases, 60 steps and 120 flows. A schema a person cannot read in a minute is two schemas, or two specifications.

Mark every guess as you go. When the code exists, read it for the flow it takes, and say where the code and the specification's clauses disagree: that is a gap to settle, not a detail to draw around.

### Phase 2: validate

**Show the draft as the page will draw it.** A list of arrows ("3 to 4 valid, 3 to 5 refused") does not show a branch: nobody reads a flow from it.

- Your agent can render a diagram in the conversation (a widget, an artifact, a rendered diagram block): draw it there. One column per lane, the steps from top to bottom, one band per phase, every flow an arrow with its label, the `failure` steps in another colour.
- It cannot: write the lane table, in the language of the human. One column per lane, one row per step in reading order, and a last column that says where the step leads, with the label of each branch. Mark a `failure` step. When the flow has phases, a first column names the phase of each step.

```
| # | Buyer         | Checkout               | Leads to           |
|---|---------------|------------------------|--------------------|
| 1 | Enters a code |                        | 2                  |
| 2 |               | Checks the code        | 3 valid, 4 refused |
| 3 |               | Lowers the total       | end                |
| 4 |               | Says why not (failure) | end                |
```

Under the drawing, name your guesses in a few short lines. Nothing else: the questions do not go there.

**Then ask, through the question tool.** Ask what is wrong, not whether it is right. One question on the flow, with the usual faults as its options:

- nothing is wrong, write it;
- a lane is missing (someone who acts and is not drawn);
- the order is wrong;
- a failure is missing (the unhappy path is where flows are usually wrong).

Then one question per thing the draft could not settle, each with its own options:

- a gap between the code and a clause: keep what the code does, change it, or leave it out of the drawing;
- a rule the flow revealed: make it a rule, not a rule, or decide later.

Correct, draw again and ask again until they say yes to the flow.

**Nobody answers** (a scheduled run, a human who left): stop here, without Phase 3, and do not call the question tool: nobody is there to answer it. Your last message holds the whole draft, as the lane table. It says what the draft was read from and that no human confirmed it. It says that nothing was written to Aporia and that the schema takes their yes to be written. It closes on the open questions, numbered: the question on the flow first, then each gap and each rule the flow revealed.

### Phase 3: write, on the human's yes

```jsonc
// aporia:write_schema input
{
  "spec": "commerce.checkout",
  "caption": "Paying with a code",
  "lanes": [
    { "id": "buyer", "label": "Buyer" },
    { "id": "checkout", "label": "Checkout" }
  ],
  "phases": [],
  "steps": [
    { "id": "enter", "laneId": "buyer", "title": "Enters a code" },
    { "id": "check", "laneId": "checkout", "title": "Checks the code", "detail": "Valid, not expired, not used by this buyer." },
    { "id": "apply", "laneId": "checkout", "title": "Lowers the total" },
    { "id": "refuse", "laneId": "checkout", "title": "Says why not", "failure": true }
  ],
  "flows": [
    { "from": "enter", "to": "check" },
    { "from": "check", "to": "apply", "label": "valid" },
    { "from": "check", "to": "refuse", "label": "refused" }
  ]
}
```

Every `id` is a short stable string of your own. A step names a lane that exists and a flow names steps that exist, or the call is refused.

**The specification already has a schema**: the call is refused unless `replace` is `true`. Show the human what the existing drawing holds and what yours changes, and pass `replace: true` only once they agree. `caption` titles the drawing, `note` says in one line what it is based on, and `sourceUrl` links to its source when there is one.

### Phase 4: hand off

Tell the human, with the link to the specification (`specUrl` from the context, with its number):

- that the schema is on the page of the specification;
- what is still a guess in it. A step drawn as the code does it, while the human never said it is the intended flow, is still a guess;
- every question that got no answer of its own, asked again through the question tool: a gap to settle, a rule the flow revealed and nobody wrote yet.

## Acceptance checklist

- [ ] The specification was read before a single step was drawn.
- [ ] The human saw the draft as a drawing, or as the lane table, and said yes to the final flow. Or nothing was written and the last message says so.
- [ ] Every question went through the question tool, or closed the message. None sits in the middle of one.
- [ ] No open question was closed by a yes that answered another.
- [ ] No step contradicts a rule or a criterion in force.
- [ ] The unhappy path is drawn, each of its steps with `failure: true`, or named as left out.
- [ ] An existing schema was shown before being replaced.
- [ ] Every title reads in two to five words, with its lane as the subject.

## Anti-patterns (reject)

Ten questions before a first draft. A list of arrows in place of a drawing. A question lost in the middle of a message. A yes to the drawing read as a yes to every open question. A passive title in the lane of what is acted on. A flow copied from the code and presented as the intended one. A rule hidden in a step's `detail`. Twenty lanes. A schema written before a human read it. Replacing a schema the human drew without showing it. A step per function call.
