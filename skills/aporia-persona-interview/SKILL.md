---
name: aporia-persona-interview
description: >-
  Proposes one Persona in the Vision of an Aporia product through a short
  interview with the human: the part the person plays in the decision to adopt,
  how badly the situation hurts, and the five lines of the situation (when it
  arises, what they do today, what they want, what it would change, what stands
  in the way), written with aporia:propose_persona. The persona lands as a
  hypothesis for a human to validate in Aporia, then is linked to the
  specifications it serves with aporia:link_personas. Use for
  /aporia-persona-interview, adding a persona, capturing who feels a pain and
  how badly, turning a fuzzy "our user" into a typed record, or separating the
  buyer from the user, or linking a persona to a specification. Never invents a
  persona from code.
---

# Aporia persona interview

A Persona in Aporia is not a character sheet. It is **a person in a situation**: the moment something stops working for them, what they do about it today, what they want instead. Every specification names the personas it serves, so a vague persona makes every feature vague.

You reach Aporia through the **MCP server only**, pinned to one product. Call its tools fully qualified, as `aporia:*`. If the tools are not available, stop and ask the user to connect the Aporia MCP server.

**A question to the human goes through the tool your agent has for asking one** (`AskUserQuestion` in Claude Code): one decision per question, with the answers you expect as its options. A question written in the middle of a message gets lost. Without such a tool, or when nobody is there to answer (a scheduled run), the questions close your message, numbered, one decision each.

**`aporia:propose_persona` is not among your tools?** The Aporia server was added with `?tools=work`, the profile of a coding session, which leaves it out. Ask the human to add the server again without `?tools=work`, then start over.

## The hard lines

- **Authored with the human, never from code.** The code shows what the product does. Only a person knows who it is for. You interview, you draft, they correct.
- **You propose, a human validates.** `aporia:propose_persona` writes the persona as a **hypothesis**. A human validates it on the Vision page. You cannot validate it.
- **You create, you do not rewrite.** There is no tool to edit or retire a persona that exists: a human does it in Aporia. When the interview shows an existing persona is wrong, say what you would change and let the human change it.

## The record

| Field | What it holds |
|---|---|
| `name` | two to five words anchored in a role and a situation: "Site manager on Friday" |
| `role` | their part in the DECISION to adopt, not a job title: `buyer`, `user`, `influencer` or `blocker` |
| `painSeverity` | how badly it hurts: `low`, `moderate`, `severe` or `extreme` |
| `when` | the moment the situation arises |
| `today` | what they do about it today |
| `want` | what they want instead |
| `soThat` | what it would change for them |
| `obstacle` | what stands in the way |

The five lines are one sentence each, and all five are required: a persona with a missing line is not ready to propose.

## Before you start

`aporia:get_context` once. Read the `personas` that exist: each comes with its `situation` and `validated` (false means still a hypothesis). **Do not propose a second persona for someone the product already describes.** A name already taken is refused.

Read the vision too: the persona you are about to write is someone the product's bet is about.

## How to run it

Ask one question at a time. Draft from the answer, show the draft, let the human correct it. Guess and mark the guess, rather than interrogate: through the question tool, your guesses are the options, and the human picks one or writes their own. Read `references/shared/content-style.md` for the shape of each line, and `references/frameworks.md` when you need the reasoning behind a classification.

### Step 1: the part they play → `role`

"When this product gets adopted, what is this person's part in that decision?"

- **`buyer`**: holds the budget, gives the final yes.
- **`user`**: lives with the product every day.
- **`influencer`**: shapes the decision without owning it. The internal champion is here.
- **`blocker`**: can stall or veto (security, legal, the owner of the tool being replaced).

A founder who is all four at once is a `user` first: write the person who feels the pain. When the one who pays is someone else, that is a second persona: offer it, do not merge the two.

### Step 2: the situation → the five lines

Start from a real moment, not a description: "Tell me about the last time this went wrong."

- **`when`**: the trigger. "A site report is due every Friday at five."
- **`today`**: the workaround, with what it costs when they said it. "She copies the numbers from three tools into a document, two hours every Friday."
- **`want`**: the job. "The report to write itself from what is already entered."
- **`soThat`**: the progress. "Friday afternoon goes back to the site, not to the report."
- **`obstacle`**: what holds them back, in the situation or in the switch. "The numbers live in three tools nobody may change."

Keep their words. A line that could describe anyone is not done: ask for the specific.

### Step 3: how badly → `painSeverity`

Judge by what they do today, not by what they say:

- **`low`**: they notice it and live with it.
- **`moderate`**: they complain, and have a workaround they accept.
- **`severe`**: they pay for the workaround in time or money, every week. A private list kept beside the tool is a workaround.
- **`extreme`**: they are leaving: actively looking for another tool, or already built one to replace it.

Between two levels, urgency promotes and tolerance demotes.

### Step 4: the name

Last, once the situation is clear: two to five words that carry the role and the moment. Not a first name, not a job title alone.

### Step 5: confirm, then propose

Show the whole record. Ask: "Is this the person? What is wrong in it?" Fix it. Then, on their yes:

```jsonc
// aporia:propose_persona input
{
  "name": "Site manager on Friday",
  "role": "user",
  "painSeverity": "severe",
  "when": "A site report is due every Friday at five.",
  "today": "She copies the numbers from three tools into a document.",
  "want": "The report to write itself from what is already entered.",
  "soThat": "Friday afternoon goes back to the site, not to the report.",
  "obstacle": "The numbers live in three tools nobody may change."
}
```

The answer says `validated: false`. Tell the human: the persona is in Vision as a hypothesis, and they validate it there once they have checked it against a real person.

### Step 6: link it to the specifications it serves

Ask which specifications this person is for: propose the ones whose intention speaks of their situation, read with `aporia:list_specs`. On the human's yes, link each one by the persona's name:

```jsonc
// aporia:link_personas input
{ "spec": "reports.weekly", "add": ["Site manager on Friday"] }
```

`remove` unlinks a persona the human says a specification does not serve. A persona is named exactly as `aporia:get_context` lists it: an unknown name refuses the whole call and names the known ones. The same call also links an existing persona, without an interview, when that is all the human asks.

## Acceptance checklist

- [ ] The persona came from an interview, not from the code.
- [ ] `role` is a part in the decision, not a job title.
- [ ] The five lines are specific: a real trigger, a real workaround.
- [ ] `painSeverity` matches what they do today.
- [ ] No existing persona already describes this person.
- [ ] The human confirmed the record before it was proposed, and knows it waits for their validation.
- [ ] Every specification it was linked to, the human named or accepted.

## Anti-patterns (reject)

A persona invented from the README. Demographics in place of a situation. "Wants a better experience" as a `want`. Every persona `severe`. A buyer and a user merged into one. Proposing a clone of an existing persona under another name.

## Reference

`references/frameworks.md`: why behavior beats demographics, the struggling moment behind the five lines, the buying committee behind `role`, and the evidence behind `painSeverity`.
