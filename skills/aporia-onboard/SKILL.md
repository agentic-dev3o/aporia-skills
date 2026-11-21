---
name: aporia-onboard
disable-model-invocation: true
description: >-
  Onboards a product into Aporia from its repository: reads the code for the
  major systems the product is made of (a task system, billing, a connector, an
  assistant), proposes one specification per system with its key, its domain,
  whether it is in the product or planned, and its first criteria and rules,
  has the human read and correct the list, then writes it with
  aporia:create_specs. Screens, views and sub-features are folded into the
  system they belong to, never specified on their own. Every intention arrives
  unsigned for a human to sign. Use for /aporia-onboard, onboarding a
  repository into Aporia, creating the first specifications of a product, or
  starting a product that has none. To draw the architecture, use
  aporia-architecture-scan.
---

# Aporia onboarding

A new product in Aporia has no specification: nothing says what each part of the product is for, and no task has a contract to serve. This skill writes the first specifications from the repository, **with the human**: the code shows what exists, only a person says why it exists.

Run it once per product. It can be run again safely: a specification whose key already exists is skipped.

You reach Aporia through the **MCP server only**, pinned to one product. Call its tools fully qualified, as `aporia:*`. If the tools are not available, stop and ask the user to connect the Aporia MCP server.

**A question to the human goes through the tool your agent has for asking one** (`AskUserQuestion` in Claude Code): one decision per question, with the answers you expect as its options. A question written in the middle of a message gets lost. Without such a tool, or when nobody is there to answer (a scheduled run), the questions close your message, numbered, one decision each.

**`aporia:create_specs` is not among your tools?** The Aporia server was added with `?tools=work`, the profile of a coding session, which leaves it out. Ask the human to add the server again without `?tools=work`, then start over.

## The bar

- **A specification is a major system of the product.** It is the technical and functional contract of one component a new teammate must understand before touching it: the records it holds, the states they move through, what it refuses, what it exchanges with the rest. "Checkout", "Task system", "GitHub connector", "AI assistant". Most products hold five to twelve.
- **A screen, a view or a sub-feature is a part of a system, never a specification.** A board and a list over the same tasks are two views of one task system. Labels, dependencies and the order of a column are parts of it. A part lands in its system as a clause or as a word of its summary.
- **The code shows what, never why.** You draft an intention as a guess the human confirms or rewords. You never write one they did not confirm. Every intention arrives **unsigned**, and a human signs it in Aporia.
- **A clause is a law.** A criterion says what must be true to call the system done, a rule says what always holds. Write the few the code makes certain. A guess is not a clause.
- **Nothing is written before the human read it.** You propose the whole list, they correct it, then you write.

## Workflow

```
Onboarding progress:
- [ ] Phase 0: load the context, stop on an unframed vision
- [ ] Phase 1: find the systems in the repository
- [ ] Phase 2: draft one specification per system
- [ ] Phase 3: validate the list with the human
- [ ] Phase 4: write the specifications (aporia:create_specs)
- [ ] Phase 5: hand off
```

### Phase 0: context

`aporia:get_context` once. Keep `specUrl`, the link template of a specification, for the hand-off. Read the vision: every specification you propose serves it. Read the `personas` too: they are who the systems are for.

**An unframed vision stops the onboarding.** `visionFramed` says it: it is false until the vision has both its purpose and its bet. The code shows what the product does, never who it is for or what it bets on, so without a framed vision you have nothing to sort the systems against. Before you read the repository, tell the human the vision is not framed and ask them to open `product.visionUrl` and use **Frame the vision** there, where the assistant helps them word it. Never give an invented link (when it is null, name the Vision page of the product). You do not write it for them, and you do not draft it from the code. Then wait. Start again from `aporia:get_context` once they say it is done. Only the human's own "go on without it" lets you continue, and when nobody answers you stop here.

`aporia:list_specs` tells you what already exists. On a product that already has specifications, propose only the systems they do not cover. To bring existing specifications up to date (merge, retire, move their clauses), use aporia-spec-update instead.

The first write returns a `sessionId`: pass it on every later write.

### Phase 1: find the systems in the repository

Read for the systems the product is made of, not for its screens. In this order:

1. **the data model and the server modules.** Records that live and change together, with their own states (a status, a lifecycle), make one system. A module folder is a strong hint, not a rule: two folders serving the same records are one system, and a folder of shared helpers is none. Code kept only for a migration or a retired version is not a system: you name it once, as a group, among what you left out;
2. **what talks to the outside**: connectors, webhooks, sync jobs, a public API or an MCP server, a model provider. A sync with an outside service is usually a system of its own, with everything that keeps it running (its keys, its jobs, its webhook);
3. **the README and the pages that present the product**: they give the team's own names for these systems;
4. **the navigation**: the menu, the route table, the commands. Attach each screen to the system whose records it shows. A screen is never a candidate on its own. A screen whose records belong to no system you found points to a system you missed;
5. **the names of the tests**: they say what each system guarantees, and they become its clauses.

Stop when every module and every screen has a home. You name a system from its records, its screens and its tests: you do not need to read its code line by line.

### Phase 2: draft one specification per system

**Group first, then name.** Start from the systems of Phase 1 and give everything else a home in one of them:

- views of the same records (a board, a list, a calendar, a detail page) belong to one system;
- a capability that only acts on the records of a system (labels, filters, dependencies, an export) is a part of that system;
- a tool that only feeds or serves a system (its access keys, its agent skills, its importer) is a part of the system it serves;
- a part that two systems touch belongs to the one whose rules decide it. The delivery status shown on a task is decided by the deployments, so it is a part of the git connector. A task drafted in a conversation must pass the rules of the task system, so it is a part of the task system;
- an engine that several systems call, with rules of its own (a model that finds duplicates, a scoring or sorting service), stays whole: one system, or a part of the one the team names it with. Never scatter it across its callers;
- a step that only gets a new account started (first steps, a setup checklist, a tour) is not a specification;
- what every product has (sign-in, accounts, organisations, teams and their access, the workspace or project that holds everything with its settings, its identifiers and its archive, the billing of the product's own plan, a back office) is a specification only when the team talks about it as a system of its own. A rule of that base that a system relies on (a number is never reused) lands as a rule of that system.

**The test of one specification**: could you write one page on how it works (its records and their states, its rules, what it exchanges with the other systems) that a new teammate needs before touching it? Then it is one specification. When that page would mostly say "it shows the records of another system in another way", it is a part of that system. When it needs two unrelated stories, it is two. Two systems that show on the same screen stay two: the task system and the git connector both appear on a task.

Past fifteen, you are listing screens or parts: merge.

Every string you draft is in the content language of the product (`contentLanguage`), the key excepted. You talk to the human in their own language. For each system, draft:

- **`name`**: one to four words the team would say for the system, not for one of its screens: "Task system", not "Task board";
- **`domain`**: the group it is listed under ("Commerce", "Connections"). Two to four for the whole product, each holding two systems or more. A domain with one system folds into a neighbour: never split a system to fill a domain;
- **`key`**: `domain.key`, stable (`commerce.checkout`). Lowercase letters, digits and hyphens on each side of the dot, no accent. It is what `/aporia-work feature:commerce.checkout` names, so choose it to last;
- **`status`**: `inProduct` when the code ships it, `planned` when it is only announced (a TODO, an empty screen, a README promise);
- **`summary`**: one or two sentences: what the system does, then the parts a reader might look for elsewhere ("A buyer turns a cart into a paid order. Covers coupons, saved cards and the receipt.");
- **`intention`**: why it exists, as a one-line guess the human reacts to. It is sent only when they confirm it or reword it;
- **`body`** (optional): how the system works, in markdown: short sections, a table of states or cases. Only what the code makes certain. Never a criterion or a rule: those are clauses. Never the why: that is the intention. It is not signed. Leave it out when the summary and the clauses say enough;
- **`criteria`** and **`rules`**: only the ones the code makes certain, across the whole system. This is where its parts land: a limit on labels is a rule of the task system, not a specification of labels. A rule is what the system refuses or guarantees at all times: a validation, a permission, a limit ("A code is used once per buyer"). A criterion is an outcome a user sees when the system works ("A valid code lowers the total before payment"). Keep the few that define the system, three to six of each is a good start. What every product does (a rate limit, a sign-in error) is not a clause of a system.

Read `references/shared/content-style.md` before writing any of these strings.

### Phase 3: validate with the human

Show the whole list as a table: name, domain, key, status, **what it covers** (the screens, views and parts folded into it), and the drafted intention. Under the table, a short list names what you left out and why (a setup checklist, the settings, a retired version). The human sees at a glance that nothing was lost, only grouped. Then give the summary, the body and the clauses of each system, under its name, or in a file you name when they are long: the human reads them before anything is written.

Then ask, through the question tool:

- **the groupings you hesitated on**, at most four, the ones you are least sure of, each as its own question: the grouping you drew, and the split or the merge you weighed ("Vision and personas: one specification or two?"). For an engine several systems call, the options are its own specification and each system that could hold it. A system you are sure of needs no question;
- **the drafted intentions, in one question**: keep the drafts, leave them all empty, or reword some. Ask system by system only for the ones they want to reword; their words become the `intention`;
- whether anything is missing that the code does not show yet (those are `planned`).

Take their corrections. When they split or merge, show the table again before you write. Drop what they drop. An intention they did not confirm is left out: an empty intention is honest, an invented one is not.

### Phase 4: write the specifications

One call takes at most 20 specifications, each with at most 10 criteria and 10 rules. A larger product takes several calls, split by domain, with the same `sessionId`. A clause that did not fit goes through `aporia:add_clause` afterwards. `intention`, `body`, `criteria` and `rules` are optional: leave the key out when you have none.

```jsonc
// aporia:create_specs input
{
  "specs": [
    {
      "name": "Checkout",
      "domain": "Commerce",
      "key": "commerce.checkout",
      "status": "inProduct",
      "summary": "A buyer turns a cart into a paid order. Covers coupons, saved cards and the receipt.",
      "intention": "Let a buyer pay in one go, and be rewarded without a manual discount.",
      "body": "## States\n\n| Order | Means |\n| --- | --- |\n| Pending | The payment is not confirmed yet. |\n| Paid | The receipt is sent. |",
      "criteria": [
        "A valid code lowers the total before payment.",
        "A paid order sends its receipt to the buyer."
      ],
      "rules": [
        "A code is used once per buyer.",
        "An order is charged once."
      ]
    }
  ]
}
```

The answer lists what was `created` (each with its number) and what was `skipped`: a key that already exists is left untouched, so a second run converges, and a call that failed is simply sent again. A new clause on an existing specification goes through `aporia:add_clause`, with the human's yes (the aporia-work skill).

### Phase 5: hand off

Tell the human, with links built from the templates of Phase 0:

- how many specifications were created, and which were skipped;
- that **every intention waits for a signature**: they sign each one on its page;
- what you left out because the code did not say (the `planned` systems to confirm, the intentions to write), and what you folded away on purpose (a setup checklist, the settings);
- when the human chose to go on without a framed vision: that it is still to frame, at `product.visionUrl`;
- the next step: create a first task from a specification, and run `/aporia-work` on it;
- what is optional from here: `/aporia-architecture-scan` draws the Architecture page from the repository, `/aporia-spec-schema` draws how one system works, `/aporia-persona-interview` adds who a system is for.

## Acceptance checklist

- [ ] `visionFramed` is true, or the human said to go on without it.
- [ ] Each specification is a major system, with its own records and states.
- [ ] No screen, view or sub-feature has a specification of its own: each is covered by its system.
- [ ] Every module and every screen of the repository has a home, or is named as left out.
- [ ] Each key is `domain.key`, lowercase, and chosen to last.
- [ ] No intention was invented: each is the human's words, or absent.
- [ ] Every clause is something the code makes certain.
- [ ] Every body was read by the human, and holds no clause and no why.
- [ ] The human read the list before anything was written.
- [ ] The human knows every intention is unsigned.

## Anti-patterns (reject)

One specification per screen, per view of the same records, per file or per endpoint. A sub-feature specified on its own (labels, dependencies, a delivery status, a setup checklist). Two systems merged because they show on the same screen. A system split to fill a domain. An intention written from the code alone. Twenty rules guessed from function names. Reading the repository on an unframed vision. Drafting the vision from the code. Writing before the human read the list. Calling `inProduct` what is only a TODO. Renaming a key on a second run.
