<!-- generated from shared/content-style.md: edit the source, run bun skills:materialize -->

# Content style: write for where the words land

Everything an agent writes into Aporia is read by a team, many times, in a list, on a card or in a feed. Text written for the wrong place breaks it: a sentence crammed into a title wraps in every row, a wall of prose buries its own claim. The next agent reads it too, as the truth about the product, and builds on it. **Write for the surface the words land on, and when in doubt go shorter.**

This is the contract for how text is shaped. The skill that sent you here says which fields to write, and when.

## The proportionality table

| You are writing | It is read as | Shape |
|---|---|---|
| task `title` | a card on the board | a short headline naming the work, verb first: "Persist the coupon on the order" |
| task `summary` | "In short", on the open task | one or two sentences: what, and why |
| task `body` | the open task | markdown: a lead sentence, then structure only if there are parts |
| report `summary` | one line of Activity | what was done, in one or two sentences a teammate understands without the diff |
| report `body` | the room of the task | what changed, what was verified, what is left, what a human should read |
| specification `name` | a row of the list | one to four words the team says for a system of the product, not a screen: "Checkout", "Task system" |
| specification `key` | what an agent names | `domain.key`, lowercase, stable: `commerce.checkout` |
| `intention` | the top of the specification | one to three sentences: why the feature exists, and for whom |
| specification `body` | "Details", under the intention | optional markdown: how the system works (its records, its states, a table of cases). Never a criterion or a rule, never the why, never the news of a session |
| clause `text` | a numbered line | ONE sentence a feature can pass or break |
| clause `body` | under the line | the detail, only when the sentence needs it |
| schema step `title` | a box of the drawing | two to five words, verb first, its lane as the subject |
| schema step `detail` | behind the box, when the reader points at it | one sentence, only when the title is not enough |
| schema lane `label` | the head of a column | one to three words |
| schema flow `label` | on an arrow | one or two words that name a branch |
| persona `name` | a card of Vision | two to five words anchored in a role and a situation |
| persona line (`when`, `today`, `want`, `soThat`, `obstacle`) | one line of the "Situation" block | one plain sentence each |
| idea `text` | a source, quoted | the words as they were given, untouched |
| session note `title` | one line of a session card | one plain sentence: what was decided or found |
| session note `body` | under the line | the why, in a sentence or two |
| architecture node `title` | a card of the diagram | the name the team uses, sixteen characters at most: "API", "Job queue" |
| architecture node `summary` | the second line of the card | two or three words: "7 pools", "One writer" |
| architecture node `description` | the panel under the diagram | what it does, then the choice or the limit to know, in two or three sentences |
| architecture zone `note` | one line under a frame | one sentence: "Called by the API and the workers" |
| architecture edge `label` | a word on an arrow | one to three words: "HTTPS", "webhook" |

## Punctuation and syntax

These bind every string you write, and the MCP server states them too.

1. No em-dashes. Use a period, a comma, or parentheses.
2. Do not chain independent clauses with semicolons or colons. A colon before a list, an example or a quoted value is fine.
3. Keep sentences simple and direct. Vary their length naturally.
4. One claim per sentence. A sentence holding two parenthetical thoughts is two sentences.

Bad: `"The coupon (a percentage, never an amount) is applied once (per buyer, not per order)."`
Good: `"A coupon is a percentage. It is applied once per buyer."`

A template you copy is a style you inherit: the rule applies to the examples in every skill too.

## A title is a headline

- **Plain text.** No markdown, no trailing period, no "Task:" prefix.
- **Timeless.** No status ("WIP", "SHIPPED"), no pull request number, no branch name. Git holds history, Aporia holds meaning.
- **It names the thing.** If it does not fit, you are writing a sentence: cut to the claim and move the rest to the body.

## A clause is a law, not a comment

A criterion says what must be true to call the feature done. A rule says what always holds. Both are read as a test: someone points at the product and says "this breaks it" or "this passes".

- One sentence, in the present tense: "A code is used once per buyer."
- No "should", no "ideally", no implementation detail that will change next month.
- If it finishes when the code lands, it is a task, not a clause.

## A body is markdown at reading distance

- Lead with one sentence that states the whole claim. Never repeat the title.
- Structure carries parts, not prestige: sections and bullets when there are real parts, two sentences when there are two sentences.
- Name code facts precisely (a path, a symbol, the test that proves it) and drop the narration around them.
- Write the state of things, not the news of the session ("we just", "in this PR", "now").

## Write only what you checked

Writing costs you little. Reading costs the team, every time. A claim earns that cost only when you checked it.

- A verified claim names its proof: the test that passes, the page you opened, the query you ran.
- A claim you did not check is marked as such ("Not verified: refunds."), or it is cut.
- A guess stays a guess. It never becomes a fact because the sentence reads well.

## The language

Write into Aporia in the content language of the product: `contentLanguage`, which a work order (`aporia:get_task`, `aporia:take_task`) and `aporia:get_context` both return (`fr` is French, `en` is English). It is the language the team reads its product in, whoever speaks to you and whatever language the code is in. Talk to the human in their own language.
