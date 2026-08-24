---
name: aporia-sync
description: >-
  Keeps Aporia's living map honest against the code AFTER onboarding — the
  diff-scoped re-scan to run before opening a PR or after one merges.
  Re-inventories only the subsystems the change touched (aporia:apply_scan),
  re-derives each touched feature's Built state from the code evidence it can cite,
  flags open questions/tensions whose premise the code moved (comment, never
  close), closes sync-watched inbox items with code evidence
  (aporia:resolve_items — resolve, reopen on contradiction, or attest pre-merge),
  captures the decisions and rules a merged change embodied (stated why only,
  post-merge only, settled in the same run), and hands a partly-carried decision
  its clause-by-clause comment + missing-side task instead of a close. Use when
  syncing a PR to Aporia, updating the map after a merge, refreshing as-built
  before merging, re-scanning changed code into the map, recording what a merge
  decided, or closing the Aporia ticket a PR fixes.
---

# Aporia PR sync

Onboarding is a one-time bootstrap; **this is the heartbeat.** A codebase changes every PR, and the map is only worth trusting if `as_built` tracks the code continuously. This skill is the map's **repo-facing half**: a *diff-scoped* re-scan that keeps the structure objective and, crucially, **moves each touched feature along its Implementation axis** as the code that backs it appears, matures, or regresses.

Run it **before opening a PR** (preview the delta the change creates) or **after a merge** (record the new as-built truth). It is idempotent and key-reconciled — re-running converges, never duplicates.

## The bar

The same Recognition Test as onboarding, plus one sync-specific line: **after a sync, every feature the PR touched reads its true Built state** — `Built` only if the code really carries it, `Not built` if it's still just intent. A sync that leaves a feature reading `Built` when no code carries it has failed.

The hard line is unchanged: **report structure with evidence; never fabricate intent.** Sync touches as-built reality — it never *authors* the why. It may **carry** a why a human actually stated (Phase 5's post-merge capture, which cites its source); supplying one nobody stated is fabrication. Preserve authored intent and notes verbatim; a re-scan that rediscovers an authored node realizes it, it does not clobber it.

## The Built axis (what your scan controls)

**Built is BINARY and scan-owned: a feature is Built when it is `as_built` AND carries at least one `externalRef` of kind `code`.** It is never a grade you write into a field, and no human click can raise it. Read it against the **[Realization Probe](references/shared/realization-probe.md)** (its five signals + three actions). Two things your sync controls:

1. **the feature's own evidence** — reporting it `as_built` with the `externalRefs` (kind `code`) that carry it. This, and only this, is what makes it Built;
2. **its bindings** — the `realized_by` / `touches` edges to the components/entities it uses. These describe the *shape* of the work; they never decide Built on their own.

That second line is the whole point: a feature is not Built because the components it points at happen to exist. A cited document belongs in `externalRefs` with `kind: "doc"` — shown to the team, never counted as evidence.

**There is no Partial.** A feature that is built but mocked, gated or one-sided is Built **plus an open `task`** (or a `bug`, if it contradicts a decision) naming what's missing. That item carries a title, an owner and closure evidence — strictly more than a flag ever did — and a later sync closes it on that evidence. A node carries **no certainty field at all** — "how sure are we?" is read off its open discourse, so an unsettled node carries an open question rather than a grade you type.

## How this skill reaches Aporia

Through the **Aporia MCP server only**, pinned to one product (org + product derived from the API key — you never pass them). Call tools fully-qualified as `aporia:*` so they resolve alongside other MCP servers. If they're unavailable, stop and tell the user to configure the Aporia MCP server (`APORIA_API_KEY` + `APORIA_PRODUCT_ID`).

Tools: `aporia:pull_constitution` (ground), `aporia:search_graph` / `aporia:pull_context` (what's already mapped + a feature's current bindings), `aporia:apply_scan` (push the refreshed structure + edges), `aporia:record_notes` (new questions/tensions, the tasks/bugs that name what a built-but-mocked feature is still missing, and the post-merge capture of stated decisions/rules), `aporia:comment_item` (premise flags on open questions/tensions; the clause-by-clause reading on a partly-carried decision), `aporia:resolve_items` (close sync-watched items with the evidence this scan produced — this run's own captures included; reopen closed items it contradicts). (To go the other way — compile a feature's gap into a build plan before writing code — use `aporia:feature_gaps_spec`; this skill closes the loop that opens.)

## Workflow

Copy this checklist into your response and check off each phase:

```
Sync progress:
- [ ] Phase 0 — Ground: aporia:pull_constitution; the diff's touched scopes; the branch's ticket (`<code>-<n>`); mint the run's sessionId + capture observed{ref,sha}
- [ ] Phase 1 — Diff scope: which subsystems the change actually touched
- [ ] Phase 2 — Re-inventory + distill the touched slices (D1–D4 + D6)
- [ ] Phase 3 — Re-derive Built: the feature's own code evidence + its bindings, read from code (the Realization Probe)
- [ ] Phase 4 — aporia:apply_scan per touched scope (sessionId + observed on every page; completeScope on its final page)
- [ ] Phase 5 — aporia:record_notes: new questions/tensions; each built-but-mocked feature's missing side; post-merge, the stated decisions/rules the merge embodied
- [ ] Phase 5b — Premise check: comment on open questions/tensions whose premise the code moved — flag only, never close
- [ ] Phase 6 — aporia:resolve_items: close sync-watched items the code now proves (this run's captures included); reopen what it contradicts; comment + task each partly-carried decision
```

**The run is one loop — finish it.** The phases are ordered because each depends on the last: the scan (4) is what makes the notes (5) honest and the verdicts (6) possible. A run that records notes and stops — no scan pushed, no Phase 6 verdicts — leaves the map WORSE than no run at all: fresh open items aimed at nodes whose state the same run left stale, which is the exact shape of inbox rot. If you cannot finish the loop, stop before Phase 5, not after it.

### Phase 0 — Ground

`aporia:pull_constitution` for the invariants — its `canonicalRef` also tells you up front which run this is: declared while your checkout is off it (or dirty) ⇒ a **gated preview run** (scans force-preview, resolves attest — Phases 4/6); `null` ⇒ no gate (scans apply, resolves close). For each scope the diff touches, `aporia:search_graph { keyPrefix }` (or `{ group }`) to load what's already mapped, and `aporia:pull_context { key }` on the touched features to see their **current bindings (edges) and authored intent/notes** — so you realize and refresh in place instead of duplicating, and never overwrite an authored *why*. (The derived Implementation level itself isn't in `pull_context` — read it from `aporia:feature_gaps_spec { key }` when you need it.)

**Mint the run's identity once, up front:**

- a **`sessionId`** — one fresh UUID for this whole sync run (e.g. `uuidgen`, or any random UUID). Hold it and pass it **unchanged** on **every** `aporia:apply_scan` page of **every** scope you push below. It groups a scope's pages into one run so the final `completeScope` page can't tombstone the earlier pages of the same run (Phase 4), and it lets Aporia detect a *concurrent* sync racing the same scope.
- an **`observed`** worldline — `{ ref, sha }` from git: `git rev-parse --abbrev-ref HEAD` for `ref` and `git rev-parse HEAD` for `sha` (add `dirty: true` if `git status --porcelain` is non-empty). Pass it on every `apply_scan` too — it stamps each node/edge with which code state observed it.

Also read the branch name: **`<code>-<n>-…` names the ticket this diff intends to close** (`<code>` is the product's `shortCode` lowercased, from `pull_constitution` — the /aporia-work convention). Note the number — Phase 6 checks that item's evidence first.

On a **post-merge canonical run**, also gather the merged change's **stated discourse** — `gh pr view <n> --json title,body,comments` (or the merge commit's body from `git log`). That is the only admissible source for Phase 5's decision capture: what a human actually stated, never what the diff implies.

### Phase 1 — Diff scope (the whole point — stay narrow)

Read the change: `git diff` (working tree, or the PR's merge base..head). Map the changed files to the **scopes** they belong to (the same `scopeKey`s onboarding used — a package / bounded context / service). **Only those scopes are in play.** Never re-scan the whole repo; an untouched scope must not be re-pushed (a partial re-push with `completeScope` would wrongly tombstone what you didn't re-report).

### Phase 2 — Re-inventory + distill the touched slices

For each touched scope, gather the raw facts *now* (entities/components/edges) and apply D1–D4 + D6 from the extraction protocol — exactly as onboarding, but only over the changed slice. D6 rides every re-push: kinds from the decision table (`ui`/`trigger`/`agent`/`tool`/`service`/`store`/`external`/`module`), a ≤4-word verb `label` on every `depends_on` edge (conditions included — "gated: ENABLE_V4"), `data.sub` refreshed when the signature changed (route, step caps, tool counts — concrete numbers), `data.domain` on externals, lifecycle in the group name. A re-scan that strips an existing verb label or `sub` is a regression, not a simplification. New code → new nodes/edges. Deleted code → omit it from the scope's complete batch so the tombstone sweep removes it. Renamed/moved code → same stable `key`, refreshed `externalRefs` (reconciliation handles the move).

**An entity's `fields` are code-owned — re-report them WHOLE.** A node's *shape* is the one thing a re-scan may relearn even on a node the team authored: the human's name, summary and district survive, but `data` is **replaced by what you send, never merged**. So an entity whose columns this diff added, renamed, retyped or dropped comes back with its **complete current field list**, re-read from the schema/model — and a re-report carrying three of eight fields DELETES the other five (`fields: []` empties it outright). When you re-push an entity whose shape the diff didn't touch, take its current `fields` from `aporia:pull_context` and carry them forward verbatim. Same rule, same reason as a component's `componentKind` / `sub`: shape is the code's to own, meaning is the team's.

### Phase 3 — Re-derive Built (report the evidence, not a grade)

You don't write a grade — you report the **evidence** and the **bindings** honestly, and the map reads Built from them. For every feature the diff touched, re-walk the **Realization Probe** against the current code — surface, logic, persistence/IO, data realness, gating — then:

- a feature the code now **carries** → report it `as_built` with its own `externalRefs` (kind `code`), and wire its `realized_by` / `touches` edges → **Built**;
- a feature still backed by a **mock / stub / `TODO` / one side only**, even though the file exists → report the structure as_built (it IS in the repo) AND file a **task** on the feature naming the missing side (Phase 5). It reads Built, with an open item saying what's left — which is the honest reading, not a downgrade;
- a feature whose realizing structure is **genuinely not built yet** → leave that binding `planned` (intended), and don't claim the feature's own evidence: it reads **Not built**;
- an `intended` (planned) feature the code now implements → report it as_built (omit `planned`) with its own `externalRefs` of kind `code`, plus its structure and edges — the scan realizes it (intended → as_built) and it reads Built;
- a feature whose real logic **replaced a mock** this PR → the task that named that mock is now done. If it is sync-watched, resolve it in Phase 6 (`aporia:resolve_items`, citing the landed code); if it is a teammate's to close, resolve it anyway — the scan **attests** it with your evidence rather than closing it, and the human confirms in one click.

Never report an unbuilt surface as `as_built`, and never try to raise a feature by writing a grade — the level is read from the structure you can cite.

### Phase 4 — Push with `aporia:apply_scan`

Push **per touched scope**, ≤200 nodes+edges per call, `completeScope: true` only on the final page **of each scope** (so the sweep is scoped to what you actually re-scanned). Pass the **same `sessionId`** (from Phase 0) and `observed` on **every** page of **every** scope. Hold that one `sessionId`: **omitting** it lets the final `completeScope` page tombstone the earlier pages of the very same run (the legacy pagination trap), while **switching** sessionIds mid-run trips the race guard into a `CONFLICT` — either way the run fails its own pagination. `data.type` MUST equal `type`. Same shapes and key formats as onboarding Phase 6. Check the response: `skippedEdges: 0` (or understand each), and the `removed`/`removedEdges` **counts** match exactly the code the PR deleted (they are counts — `dryRun`'s `wouldRemoveKeys` is what names the keys). A heavy sync can hit `RATE_LIMITED` — wait the returned `retryAfter`, then retry the same call.

**If `apply_scan` returns a `CONFLICT`:** another sync is sweeping this same scope right now (a `completeScope` under a *different* sessionId, seen within the last ~10 minutes). Do **not** work around it — either **re-run the whole scope under one fresh `sessionId`** (so all its pages agree), or **wait for the other sweep to finish**, then retry. A CONFLICT means two runs are racing; it is never resolved by changing anything but the timing or the sessionId.

**The server guards a disproportionate sweep.** A `completeScope` that would tombstone a large share of a scope is almost always a *partial* re-scan you flagged complete by mistake — the server **refuses** it with a `BAD_REQUEST` rather than silently reaping live nodes. The skill's discipline (only `completeScope` a scope you re-scanned *whole*) is the first line of defence; this refusal is the backstop. When it fires, the fix is never to force it through — it is one of: **drop `completeScope`** if you only re-scanned part of the scope, or **re-scan the WHOLE scope under one `sessionId`** so every page agrees, or **preview first** (below) to see exactly what would be reaped.

**Preview before you sweep with `dryRun`.** Running a real PR-preview? Call `aporia:apply_scan` with `dryRun: true` to compute the would-be delta **without writing anything**: it returns `mode: "preview"`, `wouldRemoveKeys` (the nodes a `completeScope` would tombstone — capped at 50), and `notices` (a guard refusal or a race surfaces here as a note instead of throwing, so a preview always returns its delta). Use it as the pre-PR self-check — confirm `wouldRemoveKeys` names exactly the code the PR deleted (past 50 it truncates; page the scope rather than eyeball a truncated list) and no `notices` warn of a disproportionate sweep, *then* run the real (non-dryRun) scan.

**The canonical worldline gate.** When a product declares a canonical ref (its trunk, e.g. `main`), a scan whose `observed.ref` is **not** that ref — a feature branch — or whose working tree is **dirty** is **server-forced to `mode: "preview"` and writes nothing**, returning a `notice` that names the canonical ref. This is by design: as-built truth is shared state that belongs to the trunk, so **you cannot write it from a branch**. A pre-merge sync is therefore always a preview (use it to check the delta); the scan that actually writes as-built runs **after the merge, observed on the canonical ref (clean)**. If your scan comes back `preview` with a canonical-ref notice when you expected it to write, that is the gate — merge first, then re-scan on the trunk. And on a run Phase 0 already told you is gated, the per-scope `dryRun` preview **is** the push: run it for **every** touched scope and read its full delta (`wouldRemoveKeys` + `notices`) as the pre-PR self-check — a single-node probe verifies the gate but not the delta, and no as-built write will land before the merge either way.

### Phase 5 — Record what the change surfaced

`aporia:record_notes` for the discourse a change creates — each targeting the node(s) it's about by `key`, each a ≤60-char headline `title` over a markdown `body` sized per [content-style](references/shared/content-style.md); a missing-side task's body names the mock / the `TODO` / the missing side in a line or two.

**Two gates before any note.** It rides a node **this diff touched** — a sync never files an item about pre-existing state on a node the change didn't move; that is the backlog's business, not this run's narration. And it **owes the team a verdict or a unit of work** — an observation that owes nobody stays in the PR body. A sync that files what it merely noticed turns the inbox into a log.

- a **`task`** (or a **`bug`**, when the gap contradicts a decision the team already made) on any feature whose structure is built-but-mocked — naming the missing side / the mock / the `TODO`. This is the visible "what's left to build", and unlike the flag it replaces it carries a title, an owner and closure evidence. A later scan that proves the missing side landed attests it, and a human confirms. (A feature whose binding is still `planned` needs no task — the unbuilt binding already says so.)
- a **tension** when the new code **contradicts an authored decision** (drift) — target both the decision note and the thing that breaks it;
- a **question** for a new unknown the change raises **that gates further work** — someone owes an answer before something can proceed. A finding that is merely interesting, or that you could act on tomorrow without anyone deciding anything, is not a question; a sync that files one per observation turns the inbox into a log. **And if a human is in this session, ask them before you file** — a question they answer in a sentence should become a decision (or nothing at all), not a triage row addressed to someone who is already reading. File only what they defer, decline to settle now, or aren't the right person for. Running unattended (a post-merge or CI sync, nobody to ask) — file it.
- a **decision** — or a **rule**, when the sentence still binds in two years (the split test; when it is BOTH law and work, mint the pair in one call: the rule first, then the task carrying its own node target as primary plus `{ "refType": "note", "ref": "#0", "role": "secondary" }`) — **only on the post-merge canonical run, and only for a why a human actually stated** in the PR description or discussion, a commit message's rationale, or the session's own discourse. This is how the map keeps the *why* of what just merged. It is not a change-log: the title and body state the choice in timeless voice per [content-style](references/shared/content-style.md) — a body that paraphrases the commit message is the *status report* anti-pattern, and a why nobody stated is a **question** or nothing. Before minting, `pull_context` the target: a verdict an open item already carries takes a `comment_item`, not a twin; one the merge contradicts is a tension; an open question the capture answers closes through `resolves` on the capture itself. And never capture decisions from an unmerged branch — a rejected PR must leave no phantom law behind.

**A capture is not inbox debt — the run that mints it settles it.** The scan you just pushed holds the capture's evidence: hand the `shortId` that `record_notes` returned to Phase 6, where the code that carries its verdict resolves it in the same call as everything else. It lands on the node as a settled choice with its rationale — which is the whole point of a post-merge capture — instead of another open triage row. A run that mints a decision about built code and leaves it open, worse still aimed at a node the same run left `intended`, has manufactured stale triage, not memory. And when the code carries the verdict only partly, neither resolve nor walk away: Phase 6's partial-carry path is the honest landing.

Never invent rationale, and never re-author intent — if the *why* of a change wasn't stated, it's a question for the team, not a decision. (`blocksImplementation` is deprecated — the server accepts and ignores it. File the task instead.)

### Phase 5b — Premise check (flag, never close)

The scan just read the nodes that open **questions and tensions** hang on — so it is the moment to notice a **contradicted premise**: the question assumes a module that this PR deleted, the tension's second side no longer exists, the code moved under the debate. When you find one, add a comment to that item's thread — `aporia:comment_item { ticket, body }` — citing the code fact that moved (file/symbol), so the human adjudicating it sees the new ground.

**The hard line (the collaboration model's evidence table): an epistemic item closes only through a human Decision. You NEVER close or resolve a question or a tension yourself — not via `resolve_items`, not by proposing it as "obviously moot".** A contradicted premise is CONTEXT for the human's verdict, not the verdict: you flag it with a comment, the human decides whether the item is answered, reframed, or superseded. Comment — never close.

### Phase 6 — Resolve the watched items (the close half of the loop)

Sync doesn't just report structure — it settles the **inbox items whose closure is code-evident**. Gather the candidates:

1. the **branch ticket** from Phase 0 (`<code>-<n>` — the item this diff explicitly set out to close);
2. the **open notes on the nodes you re-scanned** — `pull_context` notes carry a `shortId`. Every open **bug, task or directive decision** is a candidate; how it closes derives from its kind, so you never read a per-item flag. The missing-side task whose mock this PR replaced belongs here (Phase 3);
3. the **captures Phase 5 of this run minted** — `record_notes` returned each one's `shortId`; a post-merge capture the scanned code fully carries is settled here, in the run that minted it, never left as fresh debt.

For each candidate, judge against the code you just scanned — then one `aporia:resolve_items` call (pass the same `observed` `{ ref, sha }` from Phase 0; ≤50 items per call — page past that) with per-item verdicts. On a **gated (off-canonical) run**, also check for an open PR — `gh pr view --json number,url` — and attach `pullRequest: { number, url }` to each branch `resolve`, so the attestation carries the link (the Inbox reads *"Fix ready · PR #n"*). No PR yet? Note in your hand-off that re-running `aporia:resolve_items` after opening it upgrades the attestation (attestation is latest-wins). The server guards it hard: a `pullRequest` on a canonical resolve or on a `reopen` SKIPS THAT WHOLE ITEM rather than dropping the field, so the item does not close at all. Attach it only on the off-canonical resolve path, and never on the post-merge run.

- **`resolve`** when the code now proves it — the bug's drift is gone, the directive's verdict is built, the chore landed. `evidence` cites the code fact (file/symbol/test), not "done": *"checkout.ts persists the coupon; regression test added"*.
- **`reopen`** when the code CONTRADICTS a resolved sync-watched item — an optimistic hand-close the diff disproves, or a regression that resurrects a fixed bug. The map stays honest by re-checking, not by forbidding.
- **leave alone** anything you can't cite evidence for — an item you merely believe is done stays open for the next scan or a human.

**Evidence is the scanned code's current state, not this diff's authorship.** An open sync-watched item on a node you re-scanned whose demand the code already satisfies — even by a change that landed PRs ago — closes now, citing that code. Leaving it open because "this PR didn't build it" is how a docket rots into a backlog nobody trusts.

**The partial-carry path — a decision the code only partly proves.** A directive decision some of whose clauses the scan can cite and some it cannot gets neither a resolve nor silence:

- `aporia:comment_item` on it with the clause-by-clause reading — each carried clause cited as a code fact (file/symbol), each missing one named as absent;
- a missing-side **task** (Phase 5's shapes: the feature as primary target, the decision note as secondary) so the residue is claimable work instead of a haunting;
- a named line in your hand-off — *"<ticket> cannot fully close — missing: …"* — so a human reads it today, and the map's own signals (the comment, the task) carry it after you.

Resolving on majority evidence closes work that isn't done; saying nothing strands the decision open forever. Both are the map lying.

The response reports per-item outcomes `{ resolved, reopened, attested, skipped }`.

**The `attested` outcome — the pre-merge half of the close.** Just like the scan gate above, a `resolve` you run **off the canonical ref** (a branch, pre-merge) does **not** close the item — it **attests** it: the fix is proven in the branch's code but not yet on the trunk, so the item stays **open** carrying a *"fix ready — awaiting merge"* attestation (its `attested` count goes up, `resolved` does not) — with the PR link when you attached `pullRequest`. That badge is honest — the work is done but unmerged — and it clears automatically when the **post-merge canonical sync** runs the same `resolve` **on the canonical ref**, which finally closes the item (`resolved`) and drops the attestation. So the full close is two syncs: attest on the branch, close on the trunk. **A scan attests what it cannot close.** The same `attested` outcome covers the other way authority can be missing: an item whose KIND is a human's to close (a task). Do **not** withhold a `resolve` because you lack the authority — cite the evidence and let the server decide. It stamps an `awaiting: "confirm"` attestation, the item stays open, and the human closes it in one click with your reasoning already attached. Withholding is how evidence dies: a task proven by a scan and never attested sat open six days with a comment saying it had shipped. Never close an item whose evidence you didn't actually scan this run — but never drop evidence you did.

## Acceptance checklist

- [ ] Only the scopes the diff touched were re-scanned; untouched scopes left alone.
- [ ] Every touched feature reads its TRUE Built state — nothing reads `Built` without its own code evidence, nothing shipped reads `Not built`.
- [ ] Intended features the code now implements were realized (intended → as_built); deleted code tombstoned via `completeScope`.
- [ ] Authored intent and notes preserved verbatim — reconciled by `key`, never clobbered.
- [ ] Every re-scanned entity carries its COMPLETE current field list (shape is code-owned, and `data` is replaced wholesale — a partial list deletes the rest).
- [ ] Each built-but-mocked feature carries an open task/bug citing its missing side; each new contradiction is a tension, not a silent overwrite.
- [ ] Every `aporia:apply_scan` returned `skippedEdges: 0` (or each skip is understood).
- [ ] Every note recorded has a ≤60-char headline `title` and a markdown `body` ([content-style](references/shared/content-style.md)).
- [ ] The branch's `<code>-<n>` ticket was checked, and every sync-watched item on the touched nodes got a verdict: resolved with cited evidence, reopened on contradiction, or deliberately left for lack of evidence.
- [ ] Every item minted this run rides a node the diff touched and owes a verdict or a unit of work — no observations, no notes about pre-existing state the change didn't move.
- [ ] Any captured decision/rule cites a human-stated why with its source, on the post-merge canonical run only — and one the code fully carries was settled by this same run's Phase 6.
- [ ] Every partly-carried decision got its clause-by-clause comment, its missing-side task, and a hand-off line — never a majority-evidence resolve, never silence.

## Anti-patterns (reject)

Re-scanning the whole repo for a one-package change · `completeScope` on a scope you only partially re-scanned (the server **refuses** a disproportionate sweep with a `BAD_REQUEST` — don't try to force it; drop `completeScope`, re-scan the whole scope under one sessionId, or `dryRun` to preview first) · re-reporting an entity with a partial (or empty) `fields` array because only some columns changed — `data` is replaced wholesale, so every field you omit is deleted · reporting a mock's structure as_built without filing the task that names the missing side · claiming a feature's own `externalRefs` when no code carries it · citing a document (`kind: "doc"`) as if it were code evidence · trying to raise a feature by writing a grade instead of reporting the evidence you can cite · overwriting an authored `intent` or decision with scan output · inventing a rationale for why the code changed · filing a question at a human who is in the session with you instead of asking them · filing a question that gates nothing, turning the inbox into a log of things the scan noticed · resolving an item with "done" instead of a citable code fact · reopening a superseded item (that's a human verdict — the tool refuses it anyway) · minting a decision whose body paraphrases the commit message or the diff — a status report wearing a verdict's face; the why must be one a human stated, or it's a question · capturing decisions from an unmerged branch (a rejected PR must leave no phantom law) · minting a post-merge decision about built code and leaving it open — or aimed at a node the same run left `intended` — instead of settling it in Phase 6 · filing an item on a node the diff didn't touch — narrating the backlog instead of syncing the change · resolving a partly-carried decision on majority evidence, or leaving it silently open instead of comment + missing-side task + hand-off flag · a notes-only run — `record_notes` without the same run's scan and verdicts.
