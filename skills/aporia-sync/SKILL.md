---
name: aporia-sync
description: >-
  Keeps Aporia's living map honest against the code AFTER onboarding — the
  continuous, diff-scoped re-scan to run before opening a PR or after one merges.
  Re-inventories only the subsystems a change touched, realizes intended bindings
  the code now implements, and re-grades each touched feature's Implementation
  (absent/partial/complete) read from code — pushing it with aporia:apply_scan,
  recording new questions/tensions with aporia:record_notes, and CLOSING THE LOOP
  on sync-watched inbox items with aporia:resolve_items (resolve with code
  evidence; reopen closes the code contradicts). Drives the MCP tools — never a
  database CLI. Triggers: sync this PR to Aporia, update the map after merge,
  refresh as-built before merging, aporia sync, re-scan the changed code into the
  map, close the aporia ticket this PR fixes.
---

# Aporia PR sync

Onboarding is a one-time bootstrap; **this is the heartbeat.** A codebase changes every PR, and the map is only worth trusting if `as_built` tracks the code continuously. This skill is the **Repo/Git-app role** of the model (master-data.md §10): a *diff-scoped* re-scan that keeps the structure objective and, crucially, **moves each touched feature along its Implementation axis** as the code that backs it appears, matures, or regresses.

Run it **before opening a PR** (preview the delta the change creates) or **after a merge** (record the new as-built truth). It is idempotent and key-reconciled — re-running converges, never duplicates.

## The bar

The same Recognition Test as onboarding, plus one sync-specific line: **after a sync, every feature the PR touched reads its true Implementation level** — `Implemented` only if the code really ships it end-to-end, `Partial` if it's gated / one-sided / mocked, `Not implemented` if it's still just intent. A sync that leaves a mocked feature reading `Implemented` has failed.

The hard line is unchanged: **report structure with evidence; never fabricate intent.** Sync touches as-built reality and the realization grade — it never authors or rewrites the *why*. Preserve authored intent and notes verbatim; a re-scan that rediscovers an authored node realizes it, it does not clobber it.

## The Implementation axis (what your scan controls)

A feature's Implementation level is **derived from coverage** — it is never a grade you write into a field. The map rolls it up from two things your sync controls, read against the **Feature Realization Ladder** (see [../aporia-onboard/references/extraction-protocol.md](../aporia-onboard/references/extraction-protocol.md), D5):

1. **its bindings** — the `realized_by` / `touches` edges from the feature to the components/entities that implement it, and whether each target is `as_built` (real in the repo) or still `intended` (planned);
2. **deficiency flags** — an open note you set with `blocksImplementation: true` when the structure exists but is mocked / stubbed / one-sided.

| The code you scan | Map shows | How you produce it |
|---|---|---|
| no `as_built` binding — only intent | **Not implemented** | the feature is `planned`, or no built structure realizes it yet |
| some bindings `as_built`, some still `intended`, OR a deficiency flag is open | **Partial** | report only the built structure as_built (leave the rest `planned`), and/or flag the mock with `blocksImplementation` |
| every binding `as_built` and no open deficiency | **Implemented** | report all the structure that realizes it as_built and wire its edges |

`Implemented` is reachable **only through this scan** — it needs `as_built` code evidence a manual edit can't fabricate. So you never *assert* a level; you report structure + edges honestly and let coverage compute it. The trap: a feature whose realizing component exists in the repo but is a **mock** will read `Implemented` from structure alone — flag it `blocksImplementation` (citing the missing side) to hold it at Partial until the real logic lands. Confidence is **not** the grade; the team's certainty about a feature's *why* lives on its notes, not on the pill.

## How this skill reaches Aporia

Through the **Aporia MCP server only**, pinned to one product (org + product derived from the API key — you never pass them). Call tools fully-qualified as `aporia:*` so they resolve alongside other MCP servers. If they're unavailable, stop and tell the user to configure the Aporia MCP server (`APORIA_API_KEY` + `APORIA_PRODUCT_ID`).

Tools: `aporia:pull_constitution` (ground), `aporia:search_graph` / `aporia:pull_context` (what's already mapped + a feature's current bindings), `aporia:apply_scan` (push the refreshed structure + edges), `aporia:record_notes` (new questions/tensions, and `blocksImplementation` deficiency flags on built-but-mocked features), `aporia:resolve_items` (close sync-watched items with the evidence this scan produced; reopen closes it contradicts). (To go the other way — compile a feature's gap into a build plan before writing code — use `aporia:feature_gaps_spec`; this skill closes the loop that opens.)

## Workflow

Copy this checklist into your response and check off each phase:

```
Sync progress:
- [ ] Phase 0 — Ground: aporia:pull_constitution; the diff's touched scopes; the branch's ticket (apo-<n>)
- [ ] Phase 1 — Diff scope: which subsystems the change actually touched
- [ ] Phase 2 — Re-inventory + distill the touched slices (D1–D4)
- [ ] Phase 3 — Re-grade coverage: bindings as_built + deficiency flags read from code (the Ladder)
- [ ] Phase 4 — aporia:apply_scan per touched scope (completeScope on its final page)
- [ ] Phase 5 — aporia:record_notes: new questions/tensions; flag each built-but-mocked feature's missing side
- [ ] Phase 6 — aporia:resolve_items: close sync-watched items the code now proves; reopen what it contradicts
```

### Phase 0 — Ground

`aporia:pull_constitution` for the invariants. For each scope the diff touches, `aporia:search_graph { keyPrefix }` (or `{ group }`) to load what's already mapped, and `aporia:pull_context { key, includeNotes: true }` on the touched features to see their **current bindings, grade, and authored intent/notes** — so you realize and re-grade in place instead of duplicating, and never overwrite an authored *why*.

Also read the branch name: **`apo-<n>-…` names the ticket this diff intends to close** (the /aporia-work convention). Note the number — Phase 6 checks that item's evidence first.

### Phase 1 — Diff scope (the whole point — stay narrow)

Read the change: `git diff` (working tree, or the PR's merge base..head). Map the changed files to the **scopes** they belong to (the same `scopeKey`s onboarding used — a package / bounded context / service). **Only those scopes are in play.** Never re-scan the whole repo; an untouched scope must not be re-pushed (a partial re-push with `completeScope` would wrongly tombstone what you didn't re-report).

### Phase 2 — Re-inventory + distill the touched slices

For each touched scope, gather the raw facts *now* (entities/components/edges) and apply D1–D4 from the extraction protocol — exactly as onboarding, but only over the changed slice. New code → new nodes/edges. Deleted code → omit it from the scope's complete batch so the tombstone sweep removes it. Renamed/moved code → same stable `key`, refreshed `externalRefs` (reconciliation handles the move).

### Phase 3 — Re-grade coverage (move the Implementation axis)

You don't write a grade — you make the **bindings** and **deficiency flags** tell the truth, and coverage computes the level. For every feature the diff touched, re-walk the **Feature Realization Ladder** against the current code — surface, logic, persistence/IO, data realness, gating — then:

- a feature whose realizing component/entity **landed** this PR → report that structure `as_built` and wire its `realized_by` / `touches` edge; with every binding as_built and no open deficiency it reads **Implemented**;
- a feature still backed by a **mock / stub / `TODO` / one side only**, even though the file exists → report the structure as_built (it IS in the repo) AND flag the feature `blocksImplementation` naming the missing side (Phase 5) → **Partial**;
- a feature whose realizing structure is **genuinely not built yet** → leave that binding `planned` (intended): no as_built binding reads **Not implemented**, some reads **Partial**;
- an `intended` (planned) feature the code now implements → report its structure as_built (omit `planned`) and its edges — the scan realizes the binding (intended → as_built) and coverage flips the feature up;
- a feature whose real logic **replaced a mock** this PR → its old `blocksImplementation` flag is now stale; clear it (the canvas *Flag-clear*, or have the team resolve the note) so coverage can read it Implemented.

Never report an unbuilt surface as `as_built`, and never try to raise a feature by writing a grade — the level is read from the structure you can cite.

### Phase 4 — Push with `aporia:apply_scan`

Push **per touched scope**, ≤200 nodes+edges per call, `completeScope: true` only on the final page **of each scope** (so the sweep is scoped to what you actually re-scanned). `data.type` MUST equal `type`. Same shapes and key formats as onboarding Phase 6. Check the response: `skippedEdges: 0` (or understand each), and `removed`/`removedEdges` are exactly the code the PR deleted.

### Phase 5 — Record what the change surfaced

`aporia:record_notes` for the discourse a change creates — each targeting the node(s) it's about by `key`, each a ≤60-char headline `title` over a markdown `body` sized per [content-style](../aporia-session-notes/references/content-style.md); a `blocksImplementation` flag's body names the mock / the `TODO` / the missing side in a line or two:

- a **`blocksImplementation` deficiency flag** (a note with `blocksImplementation: true`, targeting the feature) on any feature held at **Partial** because its structure is built-but-mocked — naming the missing side / the mock / the `TODO`. This IS the Partial signal and the visible "what's left to build" beside the pill. (A feature that's Partial only because a binding is still `planned` needs no flag — the unbuilt binding already says so.)
- a **tension** when the new code **contradicts an authored decision** (drift) — target both the decision note and the thing that breaks it;
- a **question** for a genuinely new unknown the change raises.

Never invent rationale, and never re-author intent — if the *why* of a change wasn't stated, it's a question for the team, not a decision. The `blocksImplementation` flag is the agent's counterpart to the canvas's human *Flag as partial*: both can only hold a feature below complete, never raise it.

### Phase 6 — Resolve the watched items (the close half of the loop)

Sync doesn't just report structure — it settles the **inbox items whose closure is code-evident**. Gather the candidates:

1. the **branch ticket** from Phase 0 (`apo-<n>` — the item this diff explicitly set out to close);
2. the **open notes on the nodes you re-scanned** — `pull_context` notes carry `shortId` and `closesBy`; every open one with `closesBy: "sync"` (bugs, directive decisions, watched code chores) is a candidate.

For each candidate, judge against the code you just scanned — then one `aporia:resolve_items` call with per-item verdicts:

- **`resolve`** when the code now proves it — the bug's drift is gone, the directive's verdict is built, the chore landed. `evidence` cites the code fact (file/symbol/test), not "done": *"checkout.ts persists the coupon; regression test added"*.
- **`reopen`** when the code CONTRADICTS a resolved sync-watched item — an optimistic hand-close the diff disproves, or a regression that resurrects a fixed bug. The map stays honest by re-checking, not by forbidding.
- **leave alone** anything you can't cite evidence for — an item you merely believe is done stays open for the next scan or a human.

The response reports per-item outcomes; read the `skipped` reasons (a manual item is a teammate's to attest — say so in your handoff rather than forcing it). Never close an item whose evidence you didn't actually scan this run.

## Acceptance checklist

- [ ] Only the scopes the diff touched were re-scanned; untouched scopes left alone.
- [ ] Every touched feature reads its TRUE Implementation level — no mock left reading `Implemented`, no shipped feature stuck at `Partial`.
- [ ] Intended features the code now implements were realized (intended → as_built); deleted code tombstoned via `completeScope`.
- [ ] Authored intent and notes preserved verbatim — reconciled by `key`, never clobbered.
- [ ] Each built-but-mocked Partial feature carries a `blocksImplementation` flag citing its missing side; each new contradiction is a tension, not a silent overwrite.
- [ ] Every `aporia:apply_scan` returned `skippedEdges: 0` (or each skip is understood).
- [ ] Every note recorded has a ≤60-char headline `title` and a markdown `body` ([content-style](../aporia-session-notes/references/content-style.md)).
- [ ] The branch's `apo-<n>` ticket was checked, and every sync-watched item on the touched nodes got a verdict: resolved with cited evidence, reopened on contradiction, or deliberately left for lack of evidence.

## Anti-patterns (reject)

Re-scanning the whole repo for a one-package change · `completeScope` on a scope you only partially re-scanned (tombstones live nodes) · reporting a mock's structure as_built without a `blocksImplementation` flag, so it reads Implemented · trying to raise a feature by writing a grade instead of reporting the bindings you can cite · overwriting an authored `intent` or decision with scan output · inventing a rationale for why the code changed · resolving an item with "done" instead of a citable code fact · reopening a superseded item (that's a human verdict — the tool refuses it anyway).
