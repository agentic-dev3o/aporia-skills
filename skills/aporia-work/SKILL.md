---
name: aporia-work
description: >-
  Works ONE Aporia inbox ticket end-to-end: pulls and claims it as a work order
  with aporia:pull_item (bug / task / directive decision — the kinds that carry
  a verdict), zooms its map targets, implements on a ticket-named branch that
  uses the product's own prefix from pull_constitution (a product coded NEW
  branches new-8-coupon-fix), then closes the loop with the aporia-sync skill so
  scan evidence resolves the item. An epistemic ticket (idea / question /
  tension) refuses the claim and becomes an adjudication conversation with the
  human instead of a guess. Use for /aporia-work NEW-8, any product-prefixed
  ticket, working on ticket 8, picking up a ticket from the inbox, or
  implementing an Aporia inbox item.
---

# Aporia work

The ticket id a human reads in the Inbox (`NEW-8` — your product's prefix + the number; `aporia:pull_constitution` returns it as `product.shortCode`, and it is NEVER carried over from another product) is the **whole association** between triage and a coding session — no export, no sync tool, no copy-pasted description. You pull the item by that handle, the map tells you where its work lives in the code, and when you're done a **scan closes it with evidence** — not your say-so.

You reach Aporia through the **MCP server only** (pinned to one product; call tools fully-qualified as `aporia:*`). If the tools aren't available, stop and tell the user to configure the Aporia MCP server.

## The hard line

**Judgment stays human.** You build only what carries a verdict. If the pull is refused because the item is an idea, question, or tension, you do NOT pick a side and code it — you surface the judgment to the human, help them register the Decision (they can do it from the item's action zone in the Inbox, or you propose one with `aporia:record_notes` for them to confirm), and only then pull the follow-up work.

## Workflow

Copy this checklist into your response:

```
Work progress:
- [ ] Phase 0 — Ground: aporia:pull_constitution
- [ ] Phase 1 — Pull + claim the ticket: aporia:pull_item { ticket, claim: true }
- [ ] Phase 2 — Zoom: pull_context on node targets; feature_gaps_spec if feature-anchored
- [ ] Phase 3 — Branch `new-8-slug` style (the exact name Phase 1's guidance gave you); plan against the decisions/Rules
- [ ] Phase 4 — Implement + test
- [ ] Phase 5 — Close the loop: run the aporia-sync skill (scan + resolve_items)
```

### Phase 0 — Ground

`aporia:pull_constitution` once — the thesis, principles, and personas your change must honor. It also returns `product.shortCode`: **this product's ticket prefix**. Every ticket you write, and the branch you cut, use that prefix — never one you saw on another product.

### Phase 1 — Pull + claim the ticket

`aporia:pull_item { ticket: "8", claim: true }` — the **bare number always works**, whatever this product's prefix is, so prefer it over guessing a prefixed form. `claim: true` stamps an attributed, advisory claim — Aporia records and shows the acting human behind your key (and your client as the instrument), never a lock — so the Inbox shows the ticket **In progress** and a parallel session sees who's on it. Claiming also **assigns** the ticket to that human, unless the product turned auto-assign off in its settings; that assignment is durable, so it outlives the claim. You get:

- **item** — kind, title, markdown body, rationale, priority, `claimedAt` (how it closes derives from its kind);
- **targets** — the map spots it's about: node targets carry the stable `key` and `externalRefs` (the files to read first); a bug's note target carries the **decision it traces to** (the intent the code contradicts — your fix closes THAT gap, nothing more);
- **guidance** — branch convention and the next tool to call;
- **attachments** — the artifacts hanging off this ticket, as METADATA (`attachmentId`, `filename`, `fileType`, `size`, `createdAt`). A ticket whose real spec is an attached file says so here — read it with `aporia:read_attachment { attachmentId }` before you plan, or you will build from a body that only references it;
- **enforcedBy** / **enforcedByTruncated** — on a plain READ of a **rule** (claim omitted): the open items TARGETING that Rule — the work implementing the law — and whether that list is incomplete. The claim route carries neither field: a Rule refuses the claim, and the same docket arrives NAMED IN THE REFUSAL MESSAGE, which is what turns the refusal from a dead end into a hop (see the table below);
- **previousClaimedAt** — the prior claim's timestamp (`null` if none; the claim route only — a plain read always reports `null`);
- **item.owner** / **previousOwner** — who the ticket is assigned to after your claim, and who held it before (`null` if nobody). `previousOwner` non-null and not you means **your claim took a teammate's ticket** — say so to the human rather than working it silently. A product with auto-assign off answers with `owner` unchanged.

**Reading an attachment.** `aporia:read_attachment` takes the `attachmentId` from a listing — never a URL, never an id you assembled. Markdown/HTML come back as UTF-8 text, paged: when `truncated` is true, call again with `offsetBytes` set to the returned `nextOffset` (and an optional `length` to take smaller bites) until it is false. A PDF comes back whole as base64, or is refused if it is over the read ceiling — that one the human opens in the app.

**Claim collision:** a FRESH `previousClaimedAt` (within roughly two hours) means someone is likely mid-flight — stop and surface it to the human before continuing; double-working produces rival diffs. Stale or `null` → proceed. The claim is etiquette, never a lock: it locks nobody out, and you never hand-clear it. The **assignment** it makes is a separate, durable fact — only a human unassigns, from the item's rail.

**Reading is not working.** A plain read (`claim` omitted) resolves **any** ticket — a non-workable item comes back with `workOrder: false`, a not-a-work-order notice, its comment thread, and (when closed) `resolvedBy`, the successor to read first. Use that to understand context; never build from it. Only `claim: true` — the working route — refuses:

| Refusal | Your move |
|---|---|
| idea | Tell the human it needs adoption; if they adopt, a Decision is registered and that becomes the work |
| question / tension | Adjudicate WITH the human — present the sides, they decide; register the Decision, then pull the follow-up |
| rule | Standing law, never work — but the refusal MESSAGE names its **enforcement docket**: pull one of the tickets it names and work THAT. Says *no open item implements it*? Then the law implies work nobody filed — put it to the human and file it as a task on the node the work touches (`primary`) carrying the Rule as a **secondary** note target. Says the docket is *UNKNOWN* (the scan hit its cap)? File nothing yet — search the inbox for items targeting this ticket first, or you duplicate work already on the board. Either way, honor the Rule in whatever you build |
| already closed | Nothing to do; if the user disagrees with the closure, that's a reopen conversation, not a build |

### Phase 2 — Zoom

`aporia:pull_context { key }` on each node target — its data, edges, neighbors, the open notes (decisions and Rules you must honor carry their `shortId`), and `attachments`: the node's own files, plus each note's, again as metadata to read on purpose. If a target is a **feature**, compile the real spec first: `aporia:feature_gaps_spec { key }` — it returns the unbuilt gap, the decisions to honor, `openWork` (open bugs AND tasks) to close along the way, and a readiness verdict (a `blocked` spec means open questions gate the work — back to Phase 1's refusal table).

**A ticket that names a milestone, not a node.** The team tags its own map — `mvp`, `v1`, `Q2` — and those tags are readable: `aporia:search_graph { tag: "v1" }` returns every node in that bucket, whatever its type, case-insensitively. Use it when the work is scoped to a milestone rather than to one target, so the bucket the team drew on the map is the bucket you actually work. A tag is a human's plan and never a verdict: it tells you WHAT is in scope, never that something is decided.

**Read each decision's `comments` before you build from its body.** Decisions and `openWork` arrive newest-first, each carrying the tail of its thread (`moreComments` means older ones exist — `aporia:pull_item { ticket }` for the whole thread). An item stays open and in force while part of its text goes stale, so a comment saying *"we no longer deploy this to that service"* is the current truth even though the body still says otherwise. Where the thread and the body disagree, the newer comment wins. If you confirm from the code that the body itself is wrong, correct it with `aporia:update_item { ticket, body }` — the next agent reads the body, not your session.

### Phase 3 — Branch + plan

Branch as **`<code>-<n>-<short-slug>`** — `<code>` is this product's `shortCode` lowercased, so a product coded NEW branches `new-8-coupon-persistence`. Phase 1's `guidance` spells the exact branch name; use it verbatim rather than composing one — the PR-time sync parses the ticket number back out of the branch to know **which items this diff intends to close**, and checks their evidence first. Plan against what Phase 2 returned: the decisions are constraints, not suggestions; an unbuilt binding is the shape of the work.

### Phase 4 — Implement + test

**Tier the work first.** A one-file fix, config change, or pattern replication → implement solo, right here. Work that is cross-layer, has UI/UX taste at stake, or is high blast-radius → if a graded-build harness skill is installed (e.g. `agent-team`), hand the build to it instead of implementing solo: Phase 0–2's pull IS its contract grounding (don't re-pull — the constitution, gap spec, decisions-to-honor, and `openWork` materialize into its contract, with this ticket as the item ref), and its passing build closes the loop with the same aporia-sync as Phase 5 — don't run the sync twice. No such skill installed → implement solo with care proportional to the blast radius.

**Building a UI surface? Read the palette first.** `aporia:fetch_design_tokens` returns the product's tokens as one generic `design-tokens.css` — plain custom properties carrying both appearances in a single read — so what you build is set in the product's own colors, type and shape instead of ones you picked. It emits no framework dialect on purpose: you are in the repo, so read its real config and map the properties into its idiom. If the response says nothing has been authored yet, say so rather than treating a neutral starter set as the team's palette. Only write back with `aporia:upsert_design_tokens` when the human has actually settled a value — that write lands live.

Normal engineering discipline applies (project rules, tests, lint). Stay inside the ticket: a bug fix closes the traced decision's gap — scope creep belongs in new items (`aporia:record_notes`), not in this diff. Mid-flight findings ABOUT this ticket — progress, evidence, a surprise in its scope — go to its thread with `aporia:comment_item { ticket, body }`, never a new item; diff narration stays in the PR body. If the work proves that an **open** item's own text is wrong (this ticket's or one it depends on), correct the text with `aporia:update_item { ticket, body }` and comment the reasoning — never supersede a live item to fix a line inside it, which would close work this PR may still be about to close.

### Phase 5 — Close the loop

Run the **aporia-sync** skill (install it alongside this one — this phase depends on it): it re-scans the touched scopes (`apply_scan`), then calls `aporia:resolve_items` citing what the code now proves. A **bug or directive decision** closes on that evidence, provenance reading *scan-verified*; a **task** is attested and a human confirms it in one click — either way the evidence lands.

**Pre-merge, the close is a two-step.** If the product declares a canonical ref and you're still on your `<code>-<n>` branch, that sync **attests** the ticket rather than closing it: the item stays **open** with a *"fix ready — awaiting merge"* badge (the fix is proven, just not on the trunk yet). The ticket **closes** when the same sync runs **after the merge, on the canonical ref** — which drops the attestation and marks it resolved. So opening the PR earns the badge; merging and re-syncing on the trunk earns the close. Either transition (attest or close) also clears the Phase 1 claim — the evidence-backed `resolve_items` is what drops the "In progress" stamp, never a hand-edit. It does **not** clear the assignment the claim made: who is mid-flight and who owns the work are two different facts, and only a human changes the second. An item that is a teammate's to close is attested the same way — the sync cites your evidence, the item stays open, and they confirm in one click. Record any decisions/questions/deviations the work surfaced (the aporia-session-notes discipline).

## Acceptance checklist

- [ ] The item was pulled by ticket — never guessed from a title or re-transcribed by hand.
- [ ] A refused pull became a human adjudication, not a coded guess.
- [ ] The branch is named `<code>-<n>-…` (this product's code, lowercased) so the PR sync can match it.
- [ ] Every decision/Rule from the targets' context is honored in the diff.
- [ ] The loop was closed by scan evidence (`resolve_items`), not by editing the item's status.

## Anti-patterns (reject)

Working a question/tension by picking the answer yourself · building an open idea ("it was in the inbox") · hand-declaring an item done instead of letting the sync cite evidence · a branch with no ticket number, orphaning the diff from the item it closes · fixing three unrelated things under one bug's ticket.
