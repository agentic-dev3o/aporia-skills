# Evaluations — aporia-sync

Per Anthropic's skill best practices: evaluations are the source of truth for whether the
skill works. These ten scenarios lock in the disciplines most likely to regress — the
scope-boundary rules that decide when `completeScope` is safe, the same-session pagination
spare, the canonical-worldline gate that keeps a branch scan a preview, the attestation
half of the close (PR-linked when one exists), the binary scan-owned Built axis, the
wholesale replace that makes a re-reported entity's field list all-or-nothing, the
post-merge decision capture that must settle in its own run, and the partial-carry path
that comments instead of closing. There is no built-in
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
    "name": "S-3 — mock replaced by real logic: close the stale missing-side task",
    "setup": "feature:reports.export carries an OPEN task citing a MOCK_ROWS fixture that stood in for real export logic. This PR replaces that fixture with real persistence on the default path — the mock is gone.",
    "query": "Sync the map — the reports export now hits the real store, no more fixture.",
    "expected_behavior": [
      "Re-walks the Realization Probe on feature:reports.export and confirms the mock is gone: real persistence, reached on the default path, not gated",
      "Reports the now-real structure as_built AND resolves the stale task in Phase 6 — aporia:resolve_items on its shortId, evidence citing the landed code (the task is sync-watched)",
      "NEGATIVE: a run that reports the real structure but LEAVES the task open FAILS — the map would still claim the export is a fixture after the mock is already gone",
      "Does NOT try to raise the feature by writing a grade or a level into a field — Built is derived from the feature's own state + code refs, and the task is closed with evidence like any other work",
      "Only resolves the task against real code evidence — were the fixture still present, it would leave it open; and were the task a teammate's to close, it would still cite the evidence — earning an attestation rather than silently dropping the proof"
    ]
  },
  {
    "skills": ["aporia-sync"],
    "name": "S-4 — evidence-gated closure: no scanned evidence, the item stays open",
    "setup": "An open inbox item rides on a node this sync touches, but the PR's diff does NOT contain the fix it asks for — the scanned code shows no change that proves the item.",
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
    "name": "S-5b — Cursor cloud `cursor/*` branches behave like canonical on clean direct checkouts",
    "setup": "The product declares a canonicalRef (e.g. main). The agent runs on a Cursor-created branch named cursor/<something>, but that branch points to the exact same commit as canonicalRef and the working tree is clean. The branch name alone must not force preview.",
    "query": "Sync my Cursor cloud branch's changes to the shared map.",
    "expected_behavior": [
      "Phase 0 applies the Cursor special-case: since the current ref starts with cursor/ AND points at the same commit as canonicalRef, Phase 0 stamps observed.ref as canonicalRef",
      "Therefore the scan is not forced to preview: apply_scan returns mode:'applied' (when dryRun is false) and writes as-built structure",
      "Phase 6 treats resolve as canonical (no branch attestation): sync-watched items close instead of only accumulating an attested count",
      "NEGATIVE: a run that keeps observed.ref as the cursor/* branch (off-canonical) must force mode:'preview' and only attest, never close"
    ]
  },
  {
    "skills": ["aporia-sync"],
    "name": "S-6 — rejected PR: no false as-built, no falsely-closed ticket",
    "setup": "A pre-PR sync ran on a feature branch (canonicalRef declared, so the scan was forced to preview and wrote nothing). The branch also carries a sync-watched ticket <code>-<n> whose fix IS proven in the branch's code, and an open PR exists for the branch (e.g. #57). The PR is then closed / rejected — the branch never merges.",
    "query": "Walk me through what the shared map should hold after that PR was rejected.",
    "expected_behavior": [
      "Because the pre-PR scan was a preview (branch worldline), it wrote NO as-built structure — the shared map retains none of the branch's nodes / edges, so a rejected PR leaves no orphan as-built to clean up",
      "The branch's resolve of <code>-<n> ran off the canonical ref, so it ATTESTED the item ('fix ready — awaiting merge') rather than closing it — the attested count went up, resolved did not",
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
  },
  {
    "skills": ["aporia-sync"],
    "name": "S-8 — entity shape drift: re-report the WHOLE field list, never a partial one",
    "setup": "entity:billing.invoice is mapped with eight fields and was renamed / summarized by a human (origin authored, so its meaning is pinned). This PR's migration adds one column (e.g. couponCode) and retypes another (id: uuid -> string); the other six are unchanged. A second touched entity in the same scope has NO shape change at all.",
    "query": "Sync this PR — the migration adds a coupon code to invoices and switches the id column to a string.",
    "expected_behavior": [
      "Re-reads the entity's CURRENT shape from the schema/model in the repo, and pushes data.fields with ALL of them — the new column, the retyped one, and the six unchanged — because data is replaced wholesale, never merged",
      "For the untouched second entity, carries its existing fields forward verbatim (read from pull_context) rather than re-pushing it with an empty or guessed list",
      "NEGATIVE: a run that pushes only the changed fields (or fields: []) FAILS — the omitted fields are DELETED from the map, so a 'shape refresh' silently empties the entity",
      "Does NOT skip the refresh because the node is authored — a human's name / summary / district survive the re-scan, but an entity's field list is SHAPE the code owns and the scan is expected to relearn it",
      "Does NOT reach for update_node or any prose door to fix a field list — apply_scan is the only door to a node's shape, and update_node renames only"
    ]
  },
  {
    "skills": ["aporia-sync"],
    "name": "S-9 — post-merge capture: a stated decision lands settled, never as inbox debt",
    "setup": "An unattended post-merge sync runs ON the canonical ref (clean). The merged PR's description and commit message STATE the choices behind a new voting feature (e.g. 'the public API exposes hasVoted plus display order, never a raw count — counts stay staff-only'). The target feature node is still state=intended from an earlier planning pass, and no existing inbox item carries this verdict.",
    "query": "Post-merge sync on main — the voting PR just landed.",
    "expected_behavior": [
      "Phase 3/4 realizes the feature: reports it as_built with its own externalRefs of kind code, so the node stops reading intended the moment its code is on the trunk",
      "Phase 5 captures the stated decision — split test applied, so a clause that still binds in two years may come out a rule — with a timeless title/body per content-style and a rationale citing WHERE the why was stated (the PR body / commit message), not a paraphrase of the commit",
      "Phase 6 settles the capture in the SAME run: the shortId that record_notes returned rides the resolve_items call with the scanned code evidence, so the verdict lands on the node as a settled choice, not an open triage row",
      "NEGATIVE: the observed field regression FAILS — a run that mints a decision whose body restates the commit message, leaves it open, leaves the node intended, and never calls resolve_items has manufactured stale triage, not memory",
      "Had the why NOT been stated anywhere citable, it files a question (or nothing) — it never authors a decision from the diff"
    ]
  },
  {
    "skills": ["aporia-sync"],
    "name": "S-10 — partly-carried decision: comment + missing-side task, never a majority close",
    "setup": "An open directive decision states four clauses. The canonical code the sync re-scans carries three (each citable as file/symbol); the fourth (e.g. 'voting subscribes the org to the availability email') has no code behind it. The decision rides a feature node this diff touched.",
    "query": "Post-merge sync — close out what this merge finished.",
    "expected_behavior": [
      "Phase 6 judges the decision clause-by-clause against the scanned code, finds it PARTLY carried, and passes NO resolve verdict for it",
      "Comments the item (aporia:comment_item) with the clause-by-clause reading — each carried clause cited as a code fact, the missing one named as absent",
      "Files the missing-side task with the feature node as primary target and the decision note as secondary, so the residue is claimable work",
      "The hand-off names it explicitly — the ticket cannot fully close, and what is missing — rather than burying it under the resolved list",
      "NEGATIVE: a run that resolves on three-of-four evidence closes work that is not done, and a run that leaves the item untouched and unmentioned strands it open forever — both FAIL"
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
- **A mock left misdescribed.** Reporting a mock `as_built` without filing the task that
  names the missing side leaves nothing on the map saying the data is fiction; the
  inverse — leaving a now-stale task open after the mock is replaced (S-3) — leaves the
  map claiming work that is already done. Both are the map lying. **Built** is derived
  from the feature's own `state` + its `externalRefs` of kind `code`, never a grade
  written into a field, and there is no *Partial* to hide the difference in.
- **Closing an item without scanned evidence.** Optimistically resolving a sync-watched
  item the diff doesn't prove (S-4) is the "resolve with 'done'" anti-pattern; evidence
  must be a citable code fact from the scan you actually ran.
- **Writing shared as-built truth from a branch.** A non-canonical scan must stay a
  preview (S-5) and a branch resolve must attest, not close (S-6). Flipping the trunk's
  worldline from a branch — or closing a ticket whose fix never merged — corrupts state
  every teammate reads. The preview gate and the attestation gate are not errors to retry
  around; they are the design.
- **A partial shape push.** `data` is replaced, not merged, so re-reporting an entity with
  only the fields this diff changed (S-8) deletes the ones it didn't — the same class of
  silent reap as a mis-scoped `completeScope`, one node down. The refresh is expected (an
  entity's field list is code-owned even on an authored node), but it is all-or-nothing:
  send the complete current list, or carry the mapped one forward verbatim.
- **Overwriting authored intent or inventing rationale.** Sync touches as-built structure
  and the derived Built state only — it preserves the authored *why* verbatim and
  raises a genuinely new unknown as a question, never a fabricated decision.
- **A capture left as debt.** Post-merge decision capture exists to keep the *why* of what
  merged; a captured decision the code fully carries that is not settled by the same run's
  Phase 6 — or one whose body paraphrases the commit message — is manufactured triage, not
  memory (S-9). The why must be one a human stated, and the settle must be the same run.
- **A partial carry mis-closed.** Majority evidence never closes a decision (S-10); the
  honest landing is the clause-by-clause comment, the missing-side task, and the hand-off
  flag. Leaving the item silently open is the same failure from the other side.
