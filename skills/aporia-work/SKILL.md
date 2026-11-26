---
name: aporia-work
description: >-
  Works ONE Aporia task or specification end to end: takes the task with
  aporia:take_task, which returns its work order (the task, the intention of
  its specification, its criteria and its rules in force, the principles and
  the language of the product), implements on a branch named after its number, settles any gap between the code and the specification with
  the human in the session, then reports back with aporia:report_task so the
  team reads it in Activity. Understands both forms the app copies: a task
  number, like /aporia-work NEW-8, and a specification key, like /aporia-work
  feature:billing.invoices. Use for working a task, picking up a task from the
  board, implementing a specification, or reporting what a session did on a
  task.
---

# Aporia work

A task in Aporia is a **work order**. The number a human reads on the board (`NEW-8`: the product's prefix and the number) is the whole link between the board and a coding session. You read the task by that number, its specification tells you the contract, and when you are done you **report back** where the team reads it.

You reach Aporia through the **MCP server only**, pinned to one product. Call its tools fully qualified, as `aporia:*`. If the tools are not available, stop and ask the user to connect the Aporia MCP server.

**A question to the human goes through the tool your agent has for asking one** (`AskUserQuestion` in Claude Code): one decision per question, with the answers you expect as its options. A question written in the middle of a message gets lost. Without such a tool, or when nobody is there to answer (a scheduled run), the questions close your message, numbered, one decision each.

## The hard lines

- **The specification is the contract.** Its rules always hold. Its criteria say what done means. You code against them, you do not reinterpret them.
- **A gap is settled here, with the human.** When the code and the specification disagree, or the specification is silent, you ask the human in this session. Only what they accept reaches Aporia, as a criterion or a rule.
- **You never sign, never abrogate and never delete.** An intention is signed by a human in Aporia. A clause is abrogated by a human in Aporia. A clause written by mistake is deleted, once abrogated, by an owner or an admin in Aporia. There is no tool for any of them, on purpose.
- **You never say a task is done.** Humans own Ready, Blocked and In progress. Git owns On staging and In production. A task is in production when the deployment says so, never because a pull request merged. You report, a human or production closes.

## What you were given

| The command | What it is | Your first call |
|---|---|---|
| `/aporia-work NEW-8` or `/aporia-work 8` | a task to work | `aporia:take_task { task: "NEW-8" }` |
| `/aporia-work feature:billing.invoices` | a specification | `aporia:get_spec { spec: "feature:billing.invoices" }` |
| "look at NEW-8", "what does NEW-8 ask?" | a task to read, not to work | `aporia:get_task { task: "NEW-8" }` |

The bare number always works. A prefix that is not this product's is refused: never carry one over from another product.

**`sessionId`**: the first write of the session returns one. Pass it unchanged on every later write that takes one. Activity then shows what the session wrote as a single message.

## Workflow

```
Work progress:
- [ ] Phase 1: take the task and read its work order (aporia:take_task)
- [ ] Phase 2: branch and plan against the contract
- [ ] Phase 3: implement, and settle gaps with the human
- [ ] Phase 4: report back (aporia:report_task)
```

### Phase 1: take the task and read its work order

`aporia:take_task { task }` starts the work and returns the work order in the same call (`workOrder: false` when you read it already with `aporia:get_task`, to get only the task, `taken` and `branch`), with the `sessionId` of the session when you had none yet. The task moves to **In progress** and the human behind your key becomes its owner. Do not read it with `aporia:get_task` first: that would fetch the same work order twice.

The work order holds everything the work needs:

- **`attention`**, first: what to act on before you build, said in plain words (an unsigned intention, a dependency still open, a teammate who owns the task, bodies to read in `aporia:get_spec`). Act on each line;

- **`task`**: title, `url` (link it when you name the task to the human), `stage` (`closed` when a human closed it), summary, body, `owner` (a name) and `mine`, `blockedReason`, labels, `fields` (the custom fields a human set, as `{ name, value }` pairs), `project`, the idea it comes from, `createdAt` and `lastActivityAt`;
- **`spec`**: the contract it serves, with its `url`, or null. `intention.text` says why the feature exists, and `intention.signed` says whether a human signed that wording. `body` holds its details in markdown, or null. A long one comes as `bodyExcerpt`, its start, with `bodyTruncated: true` and `body: null`: an excerpt is never sent back as a body, which would erase the rest. It is not signed. `criteria` and `rules` are the clauses in force, each with its number. A criterion carries `report`: what the latest report of a task said of it (`state`, `by`), or null. A criterion another task already covered stays covered when you report it `notVerified`. A clause carries its text and, when short, its `body`: an exception or a precision that governs with the text. `bodyTruncated: true` says its body was left out here (long, or past what one work order carries): `aporia:get_spec { spec, clause }` reads it whole, and it may be a short rule that governs;
- **`waitsFor`**: the tasks it still waits for, neither in production nor closed, with their stage: a real dependency, say so before you start;
- **`pullRequests`** and **`release`**: what is already linked, and the release that shipped it;
- **`messages`**: the last five of its room, newest first, the newest whole. A teammate who sent the task back says why here: read them before you plan. `messagesTruncated: true` says older ones are on the task's page (`url`), for the human to open when they matter;
- **`files`**: the files of its room, as handles. Read one with `aporia:read_attachment { attachmentId }` when the real brief is a file. A text comes paged: call again with `offsetBytes` set to the returned `nextOffset`, and an optional `length`, until `truncated` is false. An image comes back as an image you can see;
- **`branch`**: the name to give your branch;
- **`contentLanguage`** and **`principles`**: the language of everything you write into Aporia, and the principles in force. Every change answers to them.

The work order replaces `aporia:get_context` for this session. Call `aporia:get_context` only when you need what it alone holds: the vision and the personas. A new task's `project` needs no read: an unknown one is created, and `projectCreated` names it.

**A teammate owns the task.** `aporia:take_task` refuses it with `CONFLICT` and names them. Tell the human whose task it is. Call again with `takeOver: true` only when the human says to take it. `taken.tookOver` then says you did, and `taken.previousOwner` whose it was: say so in your report.

**The task waits for a task still open.** `aporia:take_task` refuses it with `CONFLICT` and names what it waits for, with its stage. Tell the human. Call again with `anyway: true` only when the human says to start it anyway. When the human says the dependency is wrong, remove it with `aporia:update_task { task, removeWaitsFor }` once they agreed, then take the task. One refusal names both reasons when a teammate also owns it: send `takeOver: true` and `anyway: true` together, once the human agreed to both.

Other refusals, and your move:

| Refusal | Your move |
|---|---|
| the task is blocked | A human set it waiting, with a reason. Show the reason. A human unblocks it |
| the task seems done | It waits for a human to close it or send it back. Nothing to take |
| git owns the task | It is on staging or in production. There is nothing to take |
| the task is closed | Nothing to do. Reopening is the human's call |

**An unsigned intention is a draft.** Tell the human, and ask whether to build against it or to wait for the signature. Never treat it as approved.

**The body explains, it does not govern.** It is not signed and holds no rule. When it disagrees with a criterion or a rule, the clause wins, and the gap is settled with the human.

**A task without a specification** is plain work: its title and body are the brief.

**You were given a specification, not a task.** Read it with `aporia:get_spec`. It lists its open `tasks`. Pick the task with the human, or create one for the work you are about to do with `aporia:create_task { title, spec, take: true }`, which takes it in the same call. `aporia:list_tasks { spec }` shows every open task of the specification, each row with the open tasks it `waitsFor`.

**You were given nothing, or asked what to take next.** `aporia:list_tasks { takeable: true }` lists the Ready tasks `aporia:take_task` starts now, the next one first (the human's or unowned, waiting for nothing open). Propose its first row to the human, then take it. To look wider, its filters combine: `mine: true` for the human's own, `stage` for one stage or several (`production` for what the latest releases shipped), `project`, `label`, `type`, and `query` for words of a title or a summary; `takeable` reads Ready alone and takes no other `stage`. When `truncated` is true, narrow, or send `next` back unchanged as the whole arguments of `aporia:list_tasks`: it carries the `after` cursor and the filters. Do not build it yourself.

### Phase 2: branch and plan

Name the branch exactly as `branch` says (`new-8-persist-the-coupon`). A pull request joins its task by that number, in its branch or its title, and that link is what moves the task to staging and to production. A branch with no number orphans the diff.

Plan against the contract: every rule holds in the diff, every criterion is covered, and the `principles` of the work order are respected.

Read the code the task touches before you write any. Then restate the plan to the human in three lines at most: what changes, which rule or criterion it serves, what stays untouched. Write it in your own words, from what you read, never the task's wording copied back. A misreading caught here costs a sentence. When the human corrects it, the correction is the plan.

### Phase 3: implement, and settle gaps

Before the first write, read `references/shared/content-style.md` for the shape of every string you write: clauses, tasks, intentions, block reasons and the report. It also says which language to write in.

Normal engineering discipline applies: the project's own rules, its tests, its lint. Stay inside the task.

**When the code and the specification disagree**, stop and ask the human, with the two readings side by side. Then write only the outcome they accept, and prefer the smallest change. A clause in force that nearly says it is reworded, never doubled:

- a new clause: `aporia:add_clause { spec, kind, text }`. `kind` is `criterion` (what must be true to call it done) or `rule` (what always holds). It is in force at once, under its own number;
- a clause to reword: `aporia:revise_clause { clause, text }`. A new version under the same number, the old wording kept;
- an intention to rewrite: `aporia:write_intention { spec, text }`. It arrives **unsigned**, even over a signed one. Tell the human it waits for their signature.
- the details under the intention: `aporia:write_intention { spec, body }`, in markdown, with the human's yes. `body` replaces the whole body: start from the one `aporia:get_spec` returned and send it back complete. `null` clears it. The body is not signed, so writing it alone keeps the signature.
- a rename, a summary or a status of a specification is not an intention: `aporia:propose_spec_update` proposes it as a plan a human applies in Aporia. When that tool is not listed, offer the human `/aporia-spec-update`.

What you find and nobody decided is not a clause: it is an idea or a note. Work outside this task is a new task, once `aporia:list_tasks` shows the board does not hold it yet: `aporia:create_task { title, summary, spec, sessionId }`, with `project` (the app calls it a list) when it belongs to one: an unknown one is created, and `projectCreated` names it to tell the human. The aporia-report skill routes all of it at the end of the session. Its custom fields are a human's to set, in Aporia.

**Some tasks ship no code**: the work is the outcome itself, a clause to reword or a choice to make. Settle it with the human, write what they accept, and report.

**You are stuck on a human**: `aporia:update_task { task, stage: "blocked", blockedReason }`, naming who or what the task waits for: a block without a reason is refused. From then on the task is a human's: a human unblocks it. You may still correct its `blockedReason` (never empty it) and its other fields, never its column. `waitsFor` adds a dependency, `removeWaitsFor` removes one.

### Phase 4: report back

Keep to the shape of `references/shared/content-style.md`. Then `aporia:report_task { task, summary, body, criteria, sessionId }`:

- `summary`: what was done, in one or two sentences a teammate understands without the diff;
- `body`: what changed, what was verified and how, what was not verified, what is left. It closes with **what a human should read**: the one to three places where a mistake would cost the most (access rights, a migration, money, data that leaves the product). Leave that line out when nothing in the diff carries that risk.
- `criteria`: the state of each criterion of the specification of the task, one line per criterion, each `{ criterion, state }`. `criterion` is its number, as the work order listed it. `state` is `covered` (you verified it holds: a test, a run by hand), `notCovered` (you verified it does not hold yet) or `notVerified` (you did not verify it). Give every criterion its line: `covered` or `notCovered` only for what you verified, `notVerified` for the others, never `covered`. Name only the criteria of this task's specification, each once: a rule, an abrogated criterion or a criterion of another specification refuses the whole report. Leave `criteria` out when the task serves no specification.

The page of the specification shows, for each criterion, its latest state and the task that said it.

It lands in the room of the task and in Activity. It does not close the task and does not move it. A longer document (a rendered report, a screenshot of the result) goes with `aporia:attach_file { task, filename, fileType, content }`: `fileType` is `markdown`, `html`, `pdf`, `png` or `jpeg`, and the content is sent inline (base64 for the binary types).

Tell the human what happens next: the pull request carries the task to staging and to production, and production closes it. A task that ships no code is closed by a human with "Done". "To verify" is a human's call for a task whose pull request was never attached.

## Acceptance checklist

- [ ] The task was read by its number, never guessed from a title.
- [ ] An unsigned intention was named as a draft to the human.
- [ ] The plan was restated to the human, in your own words, before the first line of code.
- [ ] Every rule of the specification holds in the diff, and every criterion is covered or reported as not covered.
- [ ] The report gives each criterion of the specification its state, and calls `covered` only what was verified.
- [ ] Every gap was settled with the human, and only the accepted outcome was written.
- [ ] The branch carries the number the work order gave.
- [ ] The report says what was verified, not only what was written, and names what was not.
- [ ] The report points a human at what deserves their eyes, when the diff carries that risk.

## Anti-patterns (reject)

Building against an unsigned intention without saying so. Coding before the plan was restated. Picking a side in a gap and coding it. Writing a clause the human never saw. A second clause beside one that nearly says the same. Saying the task is done. A branch without the number. Fixing three unrelated things under one task. Inventing a link to Aporia. Writing a body the human never read. Hiding a rule in the body. Reporting a criterion as `covered` without having checked it.
