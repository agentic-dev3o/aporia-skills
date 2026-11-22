---
name: aporia-report
description: >-
  Reports a coding session back to Aporia, task first: what was done lands on
  its task (aporia:report_task), work that had no task gets one
  (aporia:create_task) so the pull request can carry it to production, work
  left to do becomes new tasks, a task stuck on a human is blocked with its
  reason (aporia:update_task), what is not clear yet becomes an idea or a
  source of one (aporia:deposit_idea), a rule the human accepted becomes a
  clause (aporia:add_clause), and the few things a teammate should know
  tomorrow become session notes (aporia:record_session_notes). Use for
  /aporia-report, reporting a session to Aporia, wrapping up a session, filing
  follow-up tasks, logging work done without a task, or depositing an idea
  from a session. To work one task from its number, use aporia-work.
---

# Aporia report

The team reads the product in Aporia, not in your session. A session that leaves nothing on the board did not happen for them: its diff ships under no task, and what it found is lost. This skill reports a session back, **task first**.

You reach Aporia through the **MCP server only**, pinned to one product. Call its tools fully qualified, as `aporia:*`. If the tools are not available, stop and ask the user to connect the Aporia MCP server.

**A question to the human goes through the tool your agent has for asking one** (`AskUserQuestion` in Claude Code): one decision per question, with the answers you expect as its options. A question written in the middle of a message gets lost. Without such a tool, or when nobody is there to answer (a scheduled run), the questions close your message, numbered, one decision each.

## The hard lines

- **Work lives on a task.** What was done is reported on its task. Work that had none gets one, because a pull request reaches production under the number of a task and nothing else.
- **You report, you never close.** Humans own Ready, Blocked and In progress. Git owns On staging and In production. A report moves nothing.
- **You record, you do not rule.** What governs the product (a criterion, a rule, an intention) is written only when the human accepted it in this session. Never dress a guess as a rule.

## The routing rule

Ask of each thing the session produced: **what does it owe, and to whom?**

| What it is | Where it goes | Tool |
|---|---|---|
| Work done on a task | the room of that task, and Activity | `aporia:report_task` |
| Work done with no task | a task, then its report | `aporia:create_task`, then `aporia:report_task` |
| Clear work left to do | the board, in Ready | `aporia:create_task` |
| A mistake that came back | a guard: a task for the check that catches it, or a rule the human accepted | `aporia:create_task` (or `aporia:add_clause`) |
| A task stuck on a human | Blocked, with who or what it waits for | `aporia:update_task` |
| A need, a request, a question nobody decided | Ideas, in discussion | `aporia:deposit_idea` |
| A rule or a criterion the human accepted here | the specification, in force | `aporia:add_clause` (or `aporia:revise_clause` to reword one) |
| Something a teammate should know tomorrow | Activity, under this session | `aporia:record_session_notes` |
| The narration of the diff | the pull request body | nothing in Aporia |

Four tests keep the routing honest:

- **Does it finish when the code lands?** Then it is a task. If it still binds in two years, it is a clause, and it needs the human's yes first.
- **Did anybody decide it?** If not, it is an idea, however sure you are.
- **Would a teammate miss it tomorrow?** If not, it belongs in the pull request, or nowhere.
- **Did it happen before?** A mistake seen twice is not a note. It is a guard: a check that catches it (a lint rule, a type, a test, a CI step), filed as a task, or a rule the human accepts. A mistake came back when the human says so, when the repository's own notes record it, or when the session hit it twice.

Over-filing buries what matters. A session that writes twelve notes wrote none.

## Workflow

```
Report progress:
- [ ] Phase 0: keep the sessionId; no context to load
- [ ] Phase 1: find the tasks the session worked
- [ ] Phase 2: extract what the session did, left and found
- [ ] Phase 3: route each item, and confirm with the human
- [ ] Phase 4: write
```

### Phase 0: context

**The session took its task with `aporia:take_task`?** Its work order already holds what this report needs: the number and `url` of the task, `contentLanguage`, the criteria of its specification with what the latest report said of each, and `branch`. Do not read it again. Otherwise, Phase 1 holds the rest: `aporia:list_tasks`, `aporia:list_specs`, `aporia:list_ideas`, `aporia:get_task` (even with `workOrder: false`) and `aporia:get_spec` give `contentLanguage` (and the listings their link template, `ticketUrl`), and every number carries the prefix of this product. Call `aporia:get_context` only for what it alone holds: the vision and the personas. Use **the `sessionId` of this session**: the one already passed to earlier writes, or none on the first write: it returns one. Every write that takes one carries it, so Activity shows the session as a single message.

### Phase 1: find the tasks

Every piece of work the session did belongs to a task. Find it before you write:

- the session took it with `aporia:take_task`: you hold it already;
- the number is in the branch name: read it with `aporia:get_task { task }`, or `aporia:get_task { task, workOrder: false }` when you only need its stage and pull requests;
- no number: look for it with `aporia:list_tasks`. `mine: true` lists the tasks of the human behind your key, `query` searches the words of a title or a summary, `spec` the tasks of one specification, `stage` one stage or several (`production` for what already shipped: work done before need not be done again).

Still nothing: the work had no task. Say so to the human and propose one, linked to its specification with `spec` when it serves one, and to its `project` (the app calls it a list; an unknown one is created, and `projectCreated` names it: tell the human) when it belongs to one. Set `noCode: true` when no code will ship for it: a human then closes it by hand.

### Phase 2: extract

Read back over the session and list, in the human's language:

- what was done, per task, and how it was verified. What was not verified is named too;
- work that was seen and not done;
- mistakes the session made or hit, and whether one came back;
- what the session waits for, and from whom;
- choices made, with the why;
- what was found in the code that contradicts what the team believes, or that nobody decided;
- needs and requests voiced along the way, and whose words they were.

### Phase 3: route and confirm

Route each item with the table above. Then **show the human the list before writing**: what goes where, in one line each. A clause is written only on an explicit yes. Drop what they drop.

### Phase 4: write

Read `references/shared/content-style.md` for the shape of every string. Write in this order, with the same `sessionId` on every call that takes one.

**1. The task of work that had none.** `aporia:create_task { title, summary, spec, sessionId }` creates it in Ready and returns its number. Set `type: "bug"` for a fix. `aporia:create_task { ..., take: true }` creates it already taken, in one call, or `aporia:take_task { task, sessionId }` takes it after; either moves it to In progress under the human behind your key, so the board shows whose work it is. Put the number in the title of the pull request (rename it when it is already open), or in the branch name when the branch is not pushed yet: a pull request joins its task by that number, and that link is what carries the task to staging and to production. Its custom fields are a human's to set, in Aporia.

**2. Work left to do.** Search the board first with `aporia:list_tasks` (`query`, `spec`). Work it already holds is cited by its number, never filed twice. Then one `aporia:create_task` per clear unit of work, before the reports: a report cites what is left by the number the call returns. Name the task it waits for with `aporia:update_task { task, waitsFor }` when it cannot start before another lands.

**3. The report of each task worked.**

```jsonc
// aporia:report_task input
{
  "task": "NEW-8",
  "summary": "The coupon is stored on the order and survives checkout.",
  "body": "Changed: the order keeps the code and the discount. Verified: the checkout tests, and one order placed by hand. Not verified: an order paid in two currencies. Left: refunds ignore the coupon, filed as NEW-31. To read: the rounding of the discount in the order total.",
  "criteria": [
    { "criterion": "NEW-5", "state": "covered" },
    { "criterion": "NEW-6", "state": "notVerified" }
  ]
  // and sessionId: the one your first write returned, none on the first write itself
}
```

`criteria` gives the state of each criterion of the task's specification, one line per criterion: `criterion` is its number, `state` is `covered` (verified to hold), `notCovered` (verified not to hold yet) or `notVerified` (not verified). Give every criterion its line: `covered` or `notCovered` only for what the session verified, `notVerified` for the others. Name only criteria of the specification of that task, each once: a rule, an abrogated criterion or a criterion of another specification refuses the whole report. A task with no specification takes no `criteria`. The page of the specification shows the latest state of each criterion and the task that said it. The answer's `missing` names the criteria no word covers yet: complete them with `aporia:report_task { task, criteria, sessionId }`, without a summary, and nothing is posted again.

`summary` is what a teammate reads in Activity. `body` says what changed, what was verified and how, what was not verified, what is left. It closes with **what a human should read**: the one to three places where a mistake would cost the most (access rights, a migration, money, data that leaves the product). A human cannot read everything an agent writes, so you point at what deserves their eyes. Leave the line out when nothing in the work carries that risk. A report is a page, not a transcript: a longer document goes with `aporia:attach_file`. It closes nothing: tell the human the pull request carries the task from here.

**4. A task stuck on a human.** `aporia:update_task { task, stage: "blocked", blockedReason }` (a label the task should carry: `addLabels`, or `removeLabels`, the others kept), naming who or what it waits for. A block without a reason is refused, and a human unblocks it.

**5. An idea, or a source of one.** Read `aporia:list_ideas` first (those in discussion, the most recently active first; `query` searches the titles). Open a candidate with `aporia:get_idea { idea }` to read its sources.

- An idea **already asks for the same thing**: `aporia:deposit_idea { text, idea: "NEW-41" }`. Your words join it as a new source. Two texts on the same subject that ask for different things are two ideas.
- Nothing matches: `aporia:deposit_idea { text }` creates the idea. Its `title` defaults to the first line of the text: set it when that line is a spoken sentence and not a headline.

Keep the `text` as it was given. When the words are a customer's, set `customer: true` and `who` to the name they were said under. `ref` is what the source points at: a number of the product, written bare, or a word. Pass the `sessionId` of this session.

**6. A clause the human accepted.** Read the clauses in force first, with `aporia:get_spec`. When one already says nearly the same thing, reword it with `aporia:revise_clause` rather than adding a second. Otherwise `aporia:add_clause { spec, kind, text }`, where `kind` is `criterion` (what must be true to call the feature done) or `rule` (what always holds). It is in force at once. `aporia:revise_clause { clause, text }` rewords one under the same number. A clause lives in a specification: find it with `aporia:list_specs`. When none fits, ask the human where it belongs, or deposit it as an idea. Never file it under the nearest one.

**7. Session notes**, in one call:

```jsonc
// aporia:record_session_notes input
{
  // sessionId: the one your first write returned; this call mints one when it is the first
  "notes": [
    {
      "title": "The coupon is stored on the order, not on the cart.",
      "body": "A cart is thrown away at checkout. The order is what the invoice reads.",
      "about": "NEW-8"
    },
    { "title": "The code is compared without its case.", "about": "commerce.checkout" }
  ]
}
```

A `title` is one plain sentence (120 characters at most). A `body` is the why, kept short. Each note names what it is `about`: the number of a task, an idea, a specification or a clause (`NEW-8`, or the bare `8`), or a specification key (`commerce.checkout`). A note about nothing in particular stays with the work of the session. At most 20 notes per call: if you have more, you are narrating the diff. One unknown number refuses the whole call and nothing is written: fix it and send again.

Close by telling the human what was written, each number linked with the url its write returned (or the templates from Phase 0), and what happens next: the pull request carries each task to staging and to production, and production closes it.

## Acceptance checklist

- [ ] Every piece of work done is reported on a task, found or created.
- [ ] The pull request or the branch carries the number of its task.
- [ ] Each report says what was verified, not only what was written, and names what was not.
- [ ] Each report points a human at what deserves their eyes, when the work carries that risk.
- [ ] A mistake that came back was filed as a guard, not as a note.
- [ ] The board and the specification were read before a task or a clause was added.
- [ ] Every item was routed by what it owes, not by habit.
- [ ] The human saw the list before anything was written.
- [ ] No clause was written without an explicit yes.
- [ ] The ideas were searched before a new one was created.
- [ ] Every write of the session carries the same `sessionId`.
- [ ] Nothing only narrates the diff, and nothing was said to be done.

## Anti-patterns (reject)

A diff that ships under no task. A second task for work the board already holds. A second clause beside one that nearly says the same. A note per commit. A second note for a mistake that came back. A report that hides what it did not verify. A rule the human never accepted. A second idea for a need the product already discusses. A customer's words rewritten into your own. A new `sessionId` for each call. A task filed for work that is only a thought. Saying a task is done.
