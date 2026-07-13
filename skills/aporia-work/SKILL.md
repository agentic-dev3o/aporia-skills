---
name: aporia-work
description: >-
  Works ONE Aporia inbox ticket end-to-end: pulls — and claims — the item as a
  work order with aporia:pull_item (bug / task / directive decision — the kinds
  that carry a verdict), zooms its map targets, implements against them on an apo-<n> branch,
  then closes the loop with the aporia-sync skill so scan evidence resolves the
  item. When the ticket is epistemic (idea / question / tension) the pull is
  refused and this skill turns into an adjudication conversation with the human
  instead of guessing. Triggers: /aporia-work APO-8, work on APO-8, pick up
  ticket 8 from the inbox, implement this aporia item, work the inbox backlog
  item.
---

# Aporia work

The ticket id a human reads in the Inbox (`APO-8`) is the **whole association** between triage and a coding session — no export, no sync tool, no copy-pasted description. You pull the item by that handle, the map tells you where its work lives in the code, and when you're done a **scan closes it with evidence** — not your say-so.

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
- [ ] Phase 3 — Branch apo-<n>-<slug>; plan against the decisions/Rules
- [ ] Phase 4 — Implement + test
- [ ] Phase 5 — Close the loop: run the aporia-sync skill (scan + resolve_items)
```

### Phase 0 — Ground

`aporia:pull_constitution` once — the thesis, principles, and personas your change must honor.

### Phase 1 — Pull + claim the ticket

`aporia:pull_item { ticket: "APO-8", claim: true }` (the bare number works too). `claim: true` stamps an anonymous, advisory claim so the Inbox shows the ticket **In progress** and a parallel session sees it's being worked. You get:

- **item** — kind, title, markdown body, rationale, priority, `closesBy` (how it will close), `claimedAt`;
- **targets** — the map spots it's about: node targets carry the stable `key` and `externalRefs` (the files to read first); a bug's note target carries the **decision it traces to** (the intent the code contradicts — your fix closes THAT gap, nothing more);
- **guidance** — branch convention and the next tool to call;
- **previousClaimedAt** — the prior claim's timestamp (`null` if none; the claim route only — a plain read always reports `null`).

**Claim collision:** a FRESH `previousClaimedAt` (within roughly two hours) means someone is likely mid-flight — stop and surface it to the human before continuing; double-working produces rival diffs. Stale or `null` → proceed. The claim is etiquette, never a lock: it locks nobody out, and you never hand-clear it.

**If the pull is refused**, the error tells you why — act on it, don't work around it:

| Refusal | Your move |
|---|---|
| idea | Tell the human it needs adoption; if they adopt, a Decision is registered and that becomes the work |
| question / tension | Adjudicate WITH the human — present the sides, they decide; register the Decision, then pull the follow-up |
| Rule | Not work — honor it in whatever you build |
| already closed | Nothing to do; if the user disagrees with the closure, that's a reopen conversation, not a build |

### Phase 2 — Zoom

`aporia:pull_context { key }` on each node target — its data, edges, neighbors, and the open notes (decisions and Rules you must honor carry their `shortId` and `closesBy`). If a target is a **feature**, compile the real spec first: `aporia:feature_gaps_spec { key }` — it returns the unbuilt gap, the decisions to honor, `openBugs` to fix along the way, and a readiness verdict (a `blocked` spec means open questions gate the work — back to Phase 1's refusal table).

### Phase 3 — Branch + plan

Branch as **`apo-<n>-<short-slug>`** (e.g. `apo-8-coupon-persistence`) — the PR-time sync parses the ticket number back out of the branch to know **which items this diff intends to close**, and checks their evidence first. Plan against what Phase 2 returned: the decisions are constraints, not suggestions; an unbuilt binding is the shape of the work.

### Phase 4 — Implement + test

Normal engineering discipline applies (project rules, tests, lint). Stay inside the ticket: a bug fix closes the traced decision's gap — scope creep belongs in new items (`aporia:record_notes`), not in this diff.

### Phase 5 — Close the loop

Run the **aporia-sync** skill (install it alongside this one — this phase depends on it): it re-scans the touched scopes (`apply_scan`), then calls `aporia:resolve_items` citing what the code now proves — which closes this ticket if it's sync-watched (`closesBy: "sync"`), with provenance reading *scan-verified*.

**Pre-merge, the close is a two-step.** If the product declares a canonical ref and you're still on your `apo-<n>` branch, that sync **attests** the ticket rather than closing it: the item stays **open** with a *"fix ready — awaiting merge"* badge (the fix is proven, just not on the trunk yet). The ticket **closes** when the same sync runs **after the merge, on the canonical ref** — which drops the attestation and marks it resolved. So opening the PR earns the badge; merging and re-syncing on the trunk earns the close. Either transition (attest or close) also clears the Phase 1 claim — the evidence-backed `resolve_items` is what drops the "In progress" stamp, never a hand-edit. A manual item instead waits for a teammate to attest it — say so in your handoff. Record any decisions/questions/deviations the work surfaced (the aporia-session-notes discipline).

## Acceptance checklist

- [ ] The item was pulled by ticket — never guessed from a title or re-transcribed by hand.
- [ ] A refused pull became a human adjudication, not a coded guess.
- [ ] The branch is named `apo-<n>-…` so the PR sync can match it.
- [ ] Every decision/Rule from the targets' context is honored in the diff.
- [ ] The loop was closed by scan evidence (`resolve_items`), not by editing the item's status.

## Anti-patterns (reject)

Working a question/tension by picking the answer yourself · building an open idea ("it was in the inbox") · hand-declaring an item done instead of letting the sync cite evidence · a branch with no ticket number, orphaning the diff from the item it closes · fixing three unrelated things under one bug's ticket.
