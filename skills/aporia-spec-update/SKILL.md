---
name: aporia-spec-update
disable-model-invocation: true
description: >-
  Updates the specifications of a product that already has some, after its
  repository or the way Aporia models it changed: reads the code again, maps
  every specification, clause and open task (or those of the one specification
  the human names) onto the systems the product has today (keep, fold into a
  system, retire), has the human read the table, then
  proposes it with aporia:propose_spec_update as one plan a human reviews and
  applies in Aporia. Nothing changes before a human applies it. Use for
  /aporia-spec-update, rescanning a product, cleaning up old specifications,
  merging duplicates, or bringing a product onboarded long ago up to date. For
  a product without specifications, use aporia-onboard.
---

# Aporia specification update

A product onboarded months ago drifts. Its code changed, features merged or left, and the specifications kept their old shape: duplicates, specifications of screens that no longer exist, clauses in another language, tasks attached to nothing that lives. This skill reads the repository again and proposes **one plan** that brings the specifications back to what the product is today.

You never change a specification yourself. You propose, a human applies. The plan lands in Aporia on its own page, where the human unticks what should stay as it is and applies the rest in one click, or dismisses it with a reason.

You reach Aporia through the **MCP server only**, pinned to one product. Call its tools fully qualified, as `aporia:*`. If the tools are not available, stop and ask the user to connect the Aporia MCP server.

**A question to the human goes through the tool your agent has for asking one** (`AskUserQuestion` in Claude Code): one decision per question, with the answers you expect as its options. Without such a tool, or when nobody is there to answer, the questions close your message, numbered, one decision each.

**`aporia:propose_spec_update` is not among your tools?** The Aporia server was added with `?tools=work`, the profile of a coding session, which leaves it out. Ask the human to add the server again without `?tools=work`, then start over.

## The bar

- **Numbers survive.** A clause that still holds keeps its number: it moves to its system, and a new wording becomes a new version of it. A new clause is only for what nothing says yet.
- **You delete nothing that governs.** A specification that leaves is retired, and names its successor when it has one. A clause that no longer holds is abrogated, and names the clause that says the same thing when there is one. Only a clause written by mistake is ever deleted, after its abrogation, by an owner or an admin in Aporia. An open task that no longer makes sense is the one thing a plan deletes, once a human applies it.
- **The code shows what, never why.** An intention you draft is a guess the human confirms. It arrives unsigned.
- **The human reads the whole table before you propose.**

## Workflow

```
Update progress:
- [ ] Phase 0: load the context and what exists
- [ ] Phase 1: read the repository: the systems of today
- [ ] Phase 2: map what exists onto them
- [ ] Phase 3: validate the table with the human
- [ ] Phase 4: propose the plan (aporia:propose_spec_update)
- [ ] Phase 5: hand off
```

**One specification only.** When the human names a specification, map that one alone: read the code it covers, map its clauses and its open tasks, and leave the rest of the product out of the plan. The `note` names it ("Rescan of TBK-12 Reservations"), so the human tells it apart from another plan waiting.

### Phase 0: what exists

`aporia:get_context` once: the vision, `contentLanguage` (the language of every string you write) and `specUrl`. Then `aporia:list_specs`, and `aporia:get_spec` on each specification to read its clauses in force. `aporia:list_tasks` gives the open tasks, the ones in To verify included.

### Phase 1: the systems of today

Read the repository the way aporia-onboard does: the navigation, the README, the data model and its modules, the names of the tests. The result is the list of systems a teammate would name today, each with a key (`domain.key`), a name, a domain, a one-line summary and a drafted intention. A new system may also carry a `body`: its details in markdown, not signed.

When a specification of the product already is one of these systems, it stays: it becomes the system (`spec`), with its number and its key.

### Phase 2: map what exists

Every specification, clause and open task lands in exactly one place.

**Specifications.** Each one is either:
- the system itself (`spec` on that system);
- folded into a system: it goes in that system's `replaces`. It is retired, "replaced by" the system;
- gone from the product: it goes in `retire`, with a short `reason`.

**Clauses in force** of the specifications that are replaced or retired. Each one is either:
- still true: it goes in the system's `clauses` with its number (`clause`). Give `text` only when the wording changes, for example a translation into the content language;
- a repeat of a clause that is kept: it goes in `abrogate`, with `replacedBy` naming the kept one;
- no longer true: it goes in `abrogate` with a `reason`.

A system's `clauses` also carries its new clauses (`kind` and `text`, no number): only what the code makes certain and nothing already says.

**Bodies.** A specification that is replaced or retired takes its body with it: carry what still holds into the body of the new system. A system that stays keeps its key, its domain, its intention and its body, and the plan refuses them on its line: name it in the hand-off when its intention or its body needs rewriting.

**Name, summary, status.** A system that stays takes on its line the `name` and the `summary` a teammate would give it today, in the content language: a V1 name translated is a rename, proposed here, never written directly. Give `status` (`inProduct` or `planned`) when the code says otherwise than the specification. The human sees each one before and after, and applies or unticks it on its own. Leave out a value the specification already holds.

**Open tasks.** Each task of a replaced specification goes in its system's `tasks`. A task in To verify whose work shipped goes in `closeTasks`. An open task that no longer makes sense (its screen left the product) goes in `deleteTasks`. A task with code that is not in To verify cannot be closed by hand: delete it or leave it.

Read `references/shared/content-style.md` before writing any string.

### Phase 3: validate the table with the human

Show the table by system: what it replaces, its body when it is new, the clauses it keeps (with the new wording when there is one), the new ones, the tasks it takes. Then what is retired, what is abrogated, which tasks are closed or deleted. Ask only about the lines you are unsure of:

- Is this specification part of that system, or a system of its own?
- Does this clause still hold?
- Is this task finished, or still to do?

Take their corrections and show the table again when it moved. An intention they did not confirm is left out.

### Phase 4: propose the plan

One call carries the whole plan, at most 40 systems and 1,500 lines. Several plans wait side by side as long as they touch nothing in common: a plan that touches a specification, a key, a clause or a task of a plan still waiting is refused with that plan's `reviewUrl`. Tell the human to apply or dismiss that one on its page, then propose again. A plan larger than the cap is split by domain, into plans that share nothing.

```jsonc
// aporia:propose_spec_update input
{
  "note": "Rescan of the repository",
  "systems": [
    {
      "key": "booking.reservations",
      "name": "Reservations",
      "domain": "Booking",
      "intention": "Let a restaurant fill its tables without a phone call.",
      "replaces": ["TBK-12", "TBK-19"],
      "clauses": [
        { "clause": "TBK-31", "text": "A table never holds two reservations at the same time." },
        { "clause": "TBK-32" },
        { "kind": "criterion", "text": "A reservation is released 20 minutes after its time when the guests did not come." }
      ],
      "tasks": ["TBK-57"]
    }
  ],
  "retire": [{ "spec": "TBK-8", "reason": "The floor plan editor left the product." }],
  "abrogate": [{ "clause": "TBK-40", "replacedBy": "TBK-31" }],
  "closeTasks": ["TBK-44"],
  "deleteTasks": ["TBK-61"]
}
```

A number that is not of this product, a specification already retired, a clause already abrogated or a task already closed refuses the whole plan, with the line named: fix it and send it again. The answer gives `lines` and `reviewUrl`.

### Phase 5: hand off

Tell the human:

- the link to `reviewUrl`, where they untick what should stay and apply the rest;
- that nothing changed yet;
- that every new intention waits for a signature once applied;
- what you left out: the vision, the principles and the personas are theirs to edit on the Vision page, and the ideas stay as they are.

## Acceptance checklist

- [ ] Every mapped specification is a system, replaced, or retired: all of the product's, or the one the human named.
- [ ] Every clause in force of a replaced or retired specification moves or is abrogated.
- [ ] A clause that still holds keeps its number.
- [ ] A system that stays carries the name and the summary it goes by today, and its status when the code contradicts it.
- [ ] Every string is in the content language.
- [ ] The human read the table before the plan was proposed.

## Anti-patterns (reject)

A new clause that repeats one you could move. Abrogating a clause that still holds. Retiring a specification whose system exists without naming it in `replaces`. Two plans that touch the same specification. Proposing before the human read the table. Writing any change directly (aporia:create_specs, aporia:add_clause, aporia:write_intention) instead of proposing it.
