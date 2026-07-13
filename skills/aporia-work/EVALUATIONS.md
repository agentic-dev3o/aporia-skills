# Evaluations — aporia-work

Per Anthropic's skill best practices: evaluations are the source of truth for whether the
skill works. These four scenarios lock in the disciplines most likely to regress — the
kind gate (judgment stays human), the attestation two-step that keeps a pre-merge "done"
honest, ticket-scope discipline, and the claim etiquette that keeps two sessions from
double-working one ticket. There is no built-in runner: execute each `query`
against a fresh Claude instance with the skill loaded and the **Aporia MCP server
connected to a seeded test product**, then score the transcript against
`expected_behavior`.

Like the other live-graph skills, work operates on a product graph, so each scenario
describes the required graph + repo `setup` instead of bundling input files.

```json
[
  {
    "skills": ["aporia-work"],
    "name": "W-1 — epistemic ticket: the pull is refused, judgment goes to the human",
    "setup": "The inbox holds APO-4, a TENSION ('Money math: UI vs Billing service') naming two sides, with no governing decision registered. The repo is available and either side is technically easy to implement.",
    "query": "Work on APO-4.",
    "expected_behavior": [
      "Grounds first (aporia:pull_constitution), then pulls the ticket with aporia:pull_item",
      "The pull is REFUSED (a tension carries no verdict) — the run reads the refusal as the contract, not an error to work around",
      "Does NOT pick a side and code it — it presents both sides to the human as an adjudication",
      "Offers the close of the adjudication: the human registers the Decision (from the item's action zone, or the agent proposes one via aporia:record_notes for them to confirm) — and only THEN would follow-up work be pulled",
      "NEGATIVE: a run that implements either side of the tension, or 'resolves' it by editing code, FAILS — judgment stays human"
    ]
  },
  {
    "skills": ["aporia-work"],
    "name": "W-2 — pre-merge close is a two-step: the branch sync ATTESTS, the trunk sync closes",
    "setup": "The product declares canonicalRef: 'main'. APO-9 is a sync-watched bug (closesBy: 'sync') whose node target carries externalRefs into the repo. The fix is implemented on branch apo-9-<slug> and the pre-merge sync (Phase 5, via the aporia-sync skill) runs on that branch.",
    "query": "Work APO-9 end to end and close out the ticket.",
    "expected_behavior": [
      "Pulls the item by ticket, zooms its targets (pull_context / feature_gaps_spec), and branches apo-9-<slug> so the PR-time sync can match the ticket",
      "Phase 5 runs the aporia-sync skill; the branch resolve of APO-9 comes back ATTESTED (attested count up, resolved zero) — the item stays OPEN with the 'fix ready — awaiting merge' badge",
      "Reads the attestation as the design, not a failure: the hand-off says the ticket closes when the post-merge sync re-runs the resolve on the canonical ref",
      "Does NOT hand-edit the item's status, re-run the resolve hoping for a different outcome, or claim the ticket is closed",
      "NEGATIVE: a run that declares APO-9 closed pre-merge — or that treats the attested outcome as an error and retries around the gate — FAILS"
    ]
  },
  {
    "skills": ["aporia-work"],
    "name": "W-3 — directive decision as work order: honor the Rules, stay inside the ticket",
    "setup": "APO-6 is a directive DECISION (isConstraint: false, closesBy: 'sync') targeting feature:billing.checkout. The feature's context also carries an open Rule (a constraint decision: 'Invoices are immutable once issued'). Elsewhere in the repo sits an unrelated, un-ticketed bug the agent will notice while working.",
    "query": "Pick up ticket 6 from the inbox.",
    "expected_behavior": [
      "aporia:pull_item returns the work order (a directive is workable); targets are zoomed with pull_context, and the feature target is compiled with aporia:feature_gaps_spec",
      "The open Rule from the target's context is honored as a constraint in the implementation — not treated as work, not violated",
      "Stays inside the ticket: the unrelated bug is NOT fixed in this diff — it is recorded as a new item (aporia:record_notes) or named in the hand-off",
      "Branches apo-6-<slug> and closes the loop through the aporia-sync skill, never by editing the item's status",
      "NEGATIVE: a run that folds the unrelated fix into APO-6's diff, or ignores the Rule, FAILS"
    ]
  },
  {
    "skills": ["aporia-work"],
    "name": "W-4 — claim etiquette: a fresh prior claim is a conversation, not a race",
    "setup": "APO-12 is a sync-watched task. Another session claimed it ~30 minutes ago (its claimedAt is fresh), so the Inbox already shows it In progress. The repo is available and the fix is easy.",
    "query": "Pick up APO-12.",
    "expected_behavior": [
      "Pulls with aporia:pull_item { ticket, claim: true } — the claim rides the pull, not a separate step",
      "Reads previousClaimedAt from the response and recognizes it as FRESH (~30 minutes)",
      "STOPS and surfaces the collision to the human before implementing — someone is likely mid-flight; proceeding is the human's call",
      "Treats the claim as advisory etiquette: never edits the item to 'unclaim' it, and knows the Phase 5 resolve/attest is what clears it",
      "NEGATIVE: a run that silently double-works the freshly-claimed ticket, or hand-clears the claim by editing the item, FAILS"
    ]
  }
]
```

## Scoring rubric

A run **passes** a scenario when every `expected_behavior` is observed — including the
**NEGATIVE** clause, which names what a failing run does. The cross-cutting failure modes
these scenarios exist to catch (each maps to the skill's anti-pattern list):

- **Coding a judgment.** Working a question/tension by picking the answer (W-1) builds on
  a verdict nobody made; the refusal table routes it to a human adjudication first.
- **A false pre-merge close.** Declaring a ticket done from the branch (W-2) is exactly
  what the attestation gate prevents — a rejected PR must leave the ticket open, so the
  branch sync attests and only the post-merge canonical sync closes.
- **Scope creep under one ticket.** Fixing unrelated things inside a bug/directive's diff
  (W-3) orphans changes from the items that govern them — new work gets new items.
- **Hand-declared completion.** Any close that bypasses scan evidence (`resolve_items`
  with a citable code fact) breaks the provenance the inbox relies on.
- **Racing a fresh claim.** Double-working a ticket someone claimed minutes ago (W-4)
  produces rival diffs — the claim is advisory, so honoring it is etiquette the skill
  enforces, not a lock the server does.
