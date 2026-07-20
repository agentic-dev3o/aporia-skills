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

`aporia:pull_item { ticket: "8", claim: true }` — the **bare number always works**, whatever this product's prefix is, so prefer it over guessing a prefixed form. `claim: true` stamps an anonymous, advisory claim so the Inbox shows the ticket **In progress** and a parallel session sees it's being worked. You get:

- **item** — kind, title, markdown body, rationale, priority, `closesBy` (how it will close), `claimedAt`;
- **targets** — the map spots it's about: node targets carry the stable `key` and `externalRefs` (the files to read first); a bug's note target carries the **decision it traces to** (the intent the code contradicts — your fix closes THAT gap, nothing more);
- **guidance** — branch convention and the next tool to call;
- **previousClaimedAt** — the prior claim's timestamp (`null` if none; the claim route only — a plain read always reports `null`).

**Claim collision:** a FRESH `previousClaimedAt` (within roughly two hours) means someone is likely mid-flight — stop and surface it to the human before continuing; double-working produces rival diffs. Stale or `null` → proceed. The claim is etiquette, never a lock: it locks nobody out, and you never hand-clear it.

**Reading is not working.** A plain read (`claim` omitted) resolves **any** ticket — a non-workable item comes back with `workOrder: false`, a not-a-work-order notice, its comment thread, and (when closed) `resolvedBy`, the successor to read first. Use that to understand context; never build from it. Only `claim: true` — the working route — refuses:

| Refusal | Your move |
|---|---|
| idea | Tell the human it needs adoption; if they adopt, a Decision is registered and that becomes the work |
| question / tension | Adjudicate WITH the human — present the sides, they decide; register the Decision, then pull the follow-up |
| Rule | Not work — honor it in whatever you build |
| already closed | Nothing to do; if the user disagrees with the closure, that's a reopen conversation, not a build |

### Phase 2 — Zoom

`aporia:pull_context { key }` on each node target — its data, edges, neighbors, and the open notes (decisions and Rules you must honor carry their `shortId` and `closesBy`). If a target is a **feature**, compile the real spec first: `aporia:feature_gaps_spec { key }` — it returns the unbuilt gap, the decisions to honor, `openBugs` to fix along the way, and a readiness verdict (a `blocked` spec means open questions gate the work — back to Phase 1's refusal table).

### Phase 3 — Branch + plan

Branch as **`<code>-<n>-<short-slug>`** — `<code>` is this product's `shortCode` lowercased, so a product coded NEW branches `new-8-coupon-persistence`. Phase 1's `guidance` spells the exact branch name; use it verbatim rather than composing one — the PR-time sync parses the ticket number back out of the branch to know **which items this diff intends to close**, and checks their evidence first. Plan against what Phase 2 returned: the decisions are constraints, not suggestions; an unbuilt binding is the shape of the work.

### Phase 4 — Implement + test

**Tier the work first.** A one-file fix, config change, or pattern replication → implement solo, right here. Work that is cross-layer, has UI/UX taste at stake, or is high blast-radius → if a graded-build harness skill is installed (e.g. `agent-team`), hand the build to it instead of implementing solo: Phase 0–2's pull IS its contract grounding (don't re-pull — the constitution, gap spec, decisions-to-honor, and `openBugs` materialize into its contract, with this ticket as the item ref), and its passing build closes the loop with the same aporia-sync as Phase 5 — don't run the sync twice. No such skill installed → implement solo with care proportional to the blast radius.

Normal engineering discipline applies (project rules, tests, lint). Stay inside the ticket: a bug fix closes the traced decision's gap — scope creep belongs in new items (`aporia:record_notes`), not in this diff. Mid-flight findings ABOUT this ticket — progress, evidence, a surprise in its scope — go to its thread with `aporia:comment_item { ticket, body }`, never a new item; diff narration stays in the PR body.

### Phase 5 — Close the loop

Run the **aporia-sync** skill (install it alongside this one — this phase depends on it): it re-scans the touched scopes (`apply_scan`), then calls `aporia:resolve_items` citing what the code now proves — which closes this ticket if it's sync-watched (`closesBy: "sync"`), with provenance reading *scan-verified*.

**Pre-merge, the close is a two-step.** If the product declares a canonical ref and you're still on your `<code>-<n>` branch, that sync **attests** the ticket rather than closing it: the item stays **open** with a *"fix ready — awaiting merge"* badge (the fix is proven, just not on the trunk yet). The ticket **closes** when the same sync runs **after the merge, on the canonical ref** — which drops the attestation and marks it resolved. So opening the PR earns the badge; merging and re-syncing on the trunk earns the close. Either transition (attest or close) also clears the Phase 1 claim — the evidence-backed `resolve_items` is what drops the "In progress" stamp, never a hand-edit. A manual item instead waits for a teammate to attest it — say so in your handoff. Record any decisions/questions/deviations the work surfaced (the aporia-session-notes discipline).

## Acceptance checklist

- [ ] The item was pulled by ticket — never guessed from a title or re-transcribed by hand.
- [ ] A refused pull became a human adjudication, not a coded guess.
- [ ] The branch is named `<code>-<n>-…` (this product's code, lowercased) so the PR sync can match it.
- [ ] Every decision/Rule from the targets' context is honored in the diff.
- [ ] The loop was closed by scan evidence (`resolve_items`), not by editing the item's status.

## Anti-patterns (reject)

Working a question/tension by picking the answer yourself · building an open idea ("it was in the inbox") · hand-declaring an item done instead of letting the sync cite evidence · a branch with no ticket number, orphaning the diff from the item it closes · fixing three unrelated things under one bug's ticket.
