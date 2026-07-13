# Evaluations — aporia-sync

Per Anthropic's skill best practices: evaluations are the source of truth for whether the
skill works. These seven scenarios lock in the disciplines most likely to regress — the
scope-boundary rules that decide when `completeScope` is safe, the same-session pagination
spare, the canonical-worldline gate that keeps a branch scan a preview, the attestation
half of the close (PR-linked when one exists), and the coverage-derived Implementation axis. There is no built-in
runner: execute each `query` against a fresh Claude instance with the skill loaded and the
**Aporia MCP server connected to a seeded test product**, then score the transcript against
`expected_behavior`.

Like the other live-graph skills, sync operates on a product graph, so each scenario
describes the required graph + diff `setup` instead of bundling input files. The example
paths (`src/billing`, `src/reports`, …) are illustrative — substitute the seed's real
scopes.

```json
[
  {
    "skills": ["aporia-sync"],
    "name": "S-1 — cross-scope refactor: re-scan both, or withhold completeScope from the source",
    "setup": "A PR moves a module from scope A (e.g. src/billing) to scope B (e.g. src/payments). The moved component nodes still describe live code — only their home scope changed, and their stable keys are unchanged. Scope A retains OTHER, untouched nodes the agent did not re-inventory.",
    "query": "Sync this PR to Aporia — I moved the coupon engine out of billing into the new payments package.",
    "expected_behavior": [
      "Phase 1 diff-scoping recognizes BOTH scope A (source) and scope B (destination) are touched — a move is a two-scope change, not a B-only add",
      "Re-scans the moved nodes under their new scope B keeping their stable key with refreshed externalRefs, so reconciliation reads it as a move, not a delete-and-create",
      "Either re-scans BOTH scopes whole and sets completeScope on each scope's final page, OR — if it only re-inventoried B — WITHHOLDS completeScope from scope A entirely",
      "NEGATIVE: a run that sets completeScope on scope A while its complete batch omits the moved-but-still-live nodes FAILS — that A sweep tombstones the moved nodes as 'deleted' even though their code is alive in B",
      "If the disproportionate-sweep guard fires on scope A, it does NOT force it through — it drops completeScope on A or re-scans A whole under the one sessionId, or dryRuns to preview the reap first"
    ]
  },
  {
    "skills": ["aporia-sync"],
    "name": "S-2 — asymmetric deletion: tombstone through the deleted file's own scope",
    "setup": "A PR deletes a file in scope A AND edits code in scope B. Reading the diff, the agent's attention lands on scope B's substantive edits; scope A appears only as one 'file deleted' line. Scope A's node for the deleted file is still live in the map.",
    "query": "Sync this before I open the PR — I deleted the legacy exporter and wired the new one in reports.",
    "expected_behavior": [
      "Phase 1 diff-scoping surfaces BOTH the scope-B edits AND scope A's deletion — a deleted file is a touched scope, not noise",
      "Re-scans scope A whole and OMITS the deleted node from A's complete batch, then sets completeScope on A so the tombstone sweep removes exactly that node",
      "Confirms the apply_scan response's removed / removedEdges COUNTS match exactly the one deleted node (they are counts, not key lists — a dryRun's wouldRemoveKeys is what names the keys) — no more, no less",
      "NEGATIVE: a run that only re-scans scope B and never pushes a completeScope batch for scope A leaves the deleted file's node LIVE — the map keeps asserting code that no longer exists",
      "Does NOT reach for any delete-by-hand path — deletion is expressed by omission from a complete batch, never by a direct database edit"
    ]
  },
  {
    "skills": ["aporia-sync"],
    "name": "S-3 — mock replaced by real logic: clear the stale blocksImplementation flag",
    "setup": "feature:reports.export sits at Partial, held there by an OPEN agent-recorded blocksImplementation note (a sync-watched tension, closesBy: 'sync') citing a MOCK_ROWS fixture that stood in for real export logic. This PR replaces that fixture with real persistence on the default path — the mock is gone.",
    "query": "Sync the map — the reports export now hits the real store, no more fixture.",
    "expected_behavior": [
      "Re-walks the Realization Probe on feature:reports.export and confirms the mock is gone: real persistence, reached on the default path, not gated",
      "Reports the now-real structure as_built AND resolves the stale blocksImplementation flag in Phase 6 — aporia:resolve_items on the flag's shortId, evidence citing the landed code (the flag is sync-watched) — so coverage can read the feature Implemented",
      "NEGATIVE: a run that reports the real structure but LEAVES the blocksImplementation flag open FAILS — coverage still sees an open deficiency and the feature stays stuck at Partial after the mock is already gone",
      "Does NOT try to raise the feature by writing a grade or a level into a field — it resolves the flag with evidence and lets coverage compute Implemented from the bindings + the now-cleared deficiency",
      "Only resolves the flag against real code evidence — were the fixture still present, it would leave the flag open; and were the flag a HUMAN's canvas demote (closesBy manual), it would name it stale in the handoff instead of forcing it"
    ]
  },
  {
    "skills": ["aporia-sync"],
    "name": "S-4 — evidence-gated closure: no scanned evidence, the item stays open",
    "setup": "A sync-watched inbox item (closesBy: 'sync') rides on a node this sync touches, but the PR's diff does NOT contain the fix it asks for — the scanned code shows no change that proves the item.",
    "query": "Sync this PR and close out whatever tickets it fixes.",
    "expected_behavior": [
      "Phase 6 gathers the candidate (the sync-watched item on a re-scanned node) and judges it against the code actually scanned this run",
      "Finds NO code fact in the scanned diff that proves the fix, so LEAVES the item open — passes no resolve verdict for it through resolve_items",
      "NEGATIVE: a run that optimistically resolves the item because it 'looks done' or because the PR is in that area FAILS — resolve_items must cite a scanned code fact (file / symbol / test), never a belief",
      "Any resolve it DOES pass elsewhere cites a concrete code fact, never the bare word 'done'",
      "Reports the left-open item in the handoff as awaiting evidence, rather than force-closing it to look complete"
    ]
  },
  {
    "skills": ["aporia-sync"],
    "name": "S-5 — divergent branches: a non-canonical scan lands as a preview",
    "setup": "The product declares a canonicalRef (its trunk). Two agents sync the same scope from different checkouts; this run's observed.ref is a feature branch, NOT the canonical ref (or the working tree is dirty).",
    "query": "Sync my branch's changes to the shared map.",
    "expected_behavior": [
      "Phase 0 captures observed { ref, sha, dirty } from git and passes it on every apply_scan page",
      "Phase 0 reads canonicalRef from pull_constitution, so the gated preview is anticipated up front — not discovered mid-push or probed for with a throwaway call",
      "The scan comes back mode:'preview' with a notice naming the canonical ref — the server forced preview because observed.ref is off-trunk — and the agent reads that as the worldline gate, not an error to retry around",
      "Uses the preview delta (wouldRemoveKeys) as the pre-PR self-check, then tells the human: merge to the canonical ref first, then re-scan on trunk (clean) to write as-built",
      "NEGATIVE: a run that treats the preview as a failed write and retries, or that tries to force an as-built write from the branch, is wrong — a branch scan must NOT flip the shared map's worldline, and two divergent checkouts must never ping-pong the trunk's as-built",
      "Does NOT record the branch's structure as the trunk's truth — the shared map is only written from the canonical ref"
    ]
  },
  {
    "skills": ["aporia-sync"],
    "name": "S-6 — rejected PR: no false as-built, no falsely-closed ticket",
    "setup": "A pre-PR sync ran on a feature branch (canonicalRef declared, so the scan was forced to preview and wrote nothing). The branch also carries a sync-watched ticket apo-<n> whose fix IS proven in the branch's code, and an open PR exists for the branch (e.g. #57). The PR is then closed / rejected — the branch never merges.",
    "query": "Walk me through what the shared map should hold after that PR was rejected.",
    "expected_behavior": [
      "Because the pre-PR scan was a preview (branch worldline), it wrote NO as-built structure — the shared map retains none of the branch's nodes / edges, so a rejected PR leaves no orphan as-built to clean up",
      "The branch's resolve of apo-<n> ran off the canonical ref, so it ATTESTED the item ('fix ready — awaiting merge') rather than closing it — the attested count went up, resolved did not",
      "The branch resolve attached pullRequest:{number,url} (read from the open PR), so the attestation carried the 'Fix ready · PR #n' link in the Inbox — honest even after the rejection: the fix is proven but unmerged, and the link points at where",
      "NEGATIVE: a run that had CLOSED the ticket on the branch would leave a falsely-closed item pointing at code that never merged — the attestation gate exists precisely so a rejected PR leaves the ticket OPEN, not closed",
      "The attestation would clear on its own only when a post-merge canonical sync re-runs the resolve on trunk — since this PR was rejected, no such sync runs and the ticket correctly stays open",
      "Nets out: a rejected PR leaves NEITHER a false as-built structure NOR a falsely-closed ticket — the two gates (preview + attestation) each cover one half"
    ]
  },
  {
    "skills": ["aporia-sync"],
    "name": "S-7 — paginated scope: same-session pages must not self-tombstone",
    "setup": "One touched scope re-inventories to more than 200 nodes+edges, so it must page across multiple apply_scan calls. completeScope is set only on the FINAL page.",
    "query": "Sync this big refactor — the whole engine package changed.",
    "expected_behavior": [
      "Mints ONE sessionId in Phase 0 and passes it UNCHANGED on every apply_scan page of this scope",
      "Pages the scope at ≤200 nodes+edges per call, with completeScope:true only on the final page",
      "Because every page shares the one sessionId, the final page's completeScope sweep spares the nodes pushed on the earlier pages of the SAME run — pages 1..N-1 SURVIVE, they are not self-tombstoned",
      "NEGATIVE: a run that OMITS sessionId lets the final completeScope page see the earlier pages as strangers and TOMBSTONE them — the scope keeps only its last page (the legacy self-tombstone trap); a run that mints a FRESH sessionId per page instead trips the race guard into a CONFLICT (the server refuses rather than reaps) — either way the run failed its own pagination",
      "On a CONFLICT (a different sessionId sweeping this scope) it does NOT work around it — it re-runs the whole scope under one fresh sessionId, or waits for the other sweep to finish, then retries"
    ]
  }
]
```

## Scoring rubric

A run **passes** a scenario when every `expected_behavior` is observed — including the
**NEGATIVE** clause, which names what a failing run does. A run that produces the right
tool calls but would also do the failing thing has NOT passed. The cross-cutting failure
modes these scenarios exist to catch (each maps to the skill's anti-pattern list):

- **`completeScope` on a scope you only partially re-scanned.** A move (S-1), an
  asymmetric deletion (S-2), and a paginated push (S-7) all fail the same way: a sweep
  scoped wider than what was actually re-inventoried tombstones live nodes. The fix is
  always to re-scan the whole scope, drop `completeScope`, or `dryRun` to preview — never
  to force a guard refusal through.
- **A mock left reading its true level wrong.** Reporting a mock `as_built` without a
  `blocksImplementation` flag reads it *Implemented*; the inverse — leaving a now-stale
  flag open after the mock is replaced (S-3) — sticks a real feature at *Partial*. Both
  are the coverage axis lying. Implementation is derived from bindings + open deficiency
  flags, never a grade written into a field.
- **Closing an item without scanned evidence.** Optimistically resolving a sync-watched
  item the diff doesn't prove (S-4) is the "resolve with 'done'" anti-pattern; evidence
  must be a citable code fact from the scan you actually ran.
- **Writing shared as-built truth from a branch.** A non-canonical scan must stay a
  preview (S-5) and a branch resolve must attest, not close (S-6). Flipping the trunk's
  worldline from a branch — or closing a ticket whose fix never merged — corrupts state
  every teammate reads. The preview gate and the attestation gate are not errors to retry
  around; they are the design.
- **Overwriting authored intent or inventing rationale.** Sync touches as-built structure
  and the derived Implementation level only — it preserves the authored *why* verbatim and
  raises a genuinely new unknown as a question, never a fabricated decision.
