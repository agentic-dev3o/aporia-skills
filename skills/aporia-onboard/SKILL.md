---
name: aporia-onboard
description: >-
  Onboards a codebase into Aporia's living map through the Aporia MCP server —
  a deep scan that becomes a graph a founder recognizes. Grounds on the product
  Constitution first: when the thesis, personas, or principles are empty it authors
  them WITH the human (aporia:upsert_thesis + aporia:upsert_principles + the
  aporia-persona-mapper interview) before scanning, so every feature binds to a
  real Ring 0. Inventories the repo,
  distills it into domain entities/components (filtering plumbing, reclustering
  to the right grain, naming in the team's language with evidence), proposes
  features, then interviews the human for the intent the code can't show, and
  pushes the result with aporia:apply_scan / aporia:record_notes. Drives the MCP tools — never
  a database CLI. Triggers: onboard this repo into Aporia, scan the code into the
  map, bootstrap the product graph, define the thesis, personas, and principles, sync as-built structure.
---

# Aporia onboarding scan

You are turning a codebase into a product **map a founder would recognize** — not a UML dump. The data model is trivial; the value is entirely in *what you choose to put in the map*. This skill is the **process**; [references/extraction-protocol.md](references/extraction-protocol.md) is the **contract** — re-read it every run.

## The bar (hold throughout)

A scan passes only if **(1)** a founder shown the result says *"yes, that's my product"* within 30s, and **(2)** an engineer could use it to find where to make a change. 300 plumbing nodes fail (1); five vague boxes fail (2). The grain that passes both is the target.

The one hard line: **report structure with evidence; never fabricate intent — elicit it.** The *why*, the success criteria, the persona served are not in the code — where they aren't observable you **ASK** (Phase 5), you do not invent.

## How this skill reaches Aporia

Through the **Aporia MCP server only**. The server is pinned to one product and derives the org from its API key, so you never have to pass a product id or org; you just call the tools. Call them **fully-qualified as tools of the `aporia` MCP server** (e.g. `aporia:pull_constitution`) so they resolve even when other MCP servers are connected. If the MCP tools aren't available, stop and tell the user to configure the Aporia MCP server (`APORIA_API_KEY` + `APORIA_PRODUCT_ID`) first.

Tools you'll use: `aporia:pull_constitution` (ground), `aporia:fetch_personas` / `aporia:fetch_principles` / `aporia:upsert_thesis` / `aporia:upsert_personas` / `aporia:upsert_principles` (author the Constitution when it's empty), `aporia:search_graph` (see what's mapped), `aporia:apply_scan` (push structure + proposed features), `aporia:record_notes` (capture the elicited why).

## Workflow

Copy this checklist into your response and check off each phase:

```
Scan progress:
- [ ] Phase 0 — Ground the Constitution: aporia:pull_constitution; if the thesis/personas/principles are empty (or misaligned), DEFINE them WITH the human (aporia:upsert_thesis + the aporia-persona-mapper interview + aporia:upsert_principles) BEFORE scanning; then aporia:search_graph per scope
- [ ] Phase 1 — Scope: split the repo into the subsystems the team thinks in
- [ ] Phase 2 — Inventory: raw facts per scope (high recall, no judgement)
- [ ] Phase 3 — Distill: apply D1–D4 (plumbing filter, grain, language, evidence)
- [ ] Phase 4 — Propose features; grade each by realization read from code (D5)
- [ ] Phase 5 — Elicit the why from the human (draft-first)
- [ ] Phase 6 — aporia:apply_scan per scope (completeScope on the final page)
- [ ] Phase 7 — aporia:record_notes: the elicited decisions, questions, tensions
```

### Phase 0 — Ground the Constitution (Ring 0 first)

Call `aporia:pull_constitution` → the product's `thesis`, `principles`, and `personas` (each persona carries an `id` — a feature's `relatedPersonaIds` uses those ids, never names). The Constitution is the **ground the scan stands on**: every feature is graded and bound against it (D4/D5), so scanning onto an empty or wrong Constitution produces a map with nothing to anchor to. **Establish it before Phase 1** — and like the rest of this skill, author it **WITH the human, never from the code**:

- **Thesis — elicit the BET, don't grade the problem.** If `thesis` is null/empty, run a short, conversational interview (reflect each answer back before the write), then write it with `aporia:upsert_thesis`:
  1. `product` — what is it, who's it for, and the **wedge** (what it does that's different)? One or two sentences.
  2. `problem.statement` — the problem it attacks, stated once and concretely (the moment it bites) — not graded.
  3. `insight` — **the bet: the non-obvious thing the founder believes that makes this win, and why _now_ is the moment.** This is the centre of a thesis — a claim they could be wrong about, not a feature list or a restatement of the problem. If they can't name it yet, that gap *is* the finding — surface it; never invent one.

  The `problem.statement` and `insight` render as **markdown** — write them as tight prose (a short paragraph, emphasis on the load-bearing claim), never a wall; keep `product` to one or two sentences ([content-style](../aporia-session-notes/references/content-style.md)). It lands `draft` / `hypothesis`. **Do not ask _painful / urgent / recognized_ here** — DUR is a **challenge** lens (does the problem hold up?), applied when the in-app Assistant pressure-tests the thesis in a session, not a scan's lead question. Grading the problem before the bet is even stated is the rubric that makes a thesis feel hollow; it also just re-asks what persona `painSeverity` already captures, better, from a concrete situation. If a thesis already EXISTS but has drifted, **refine** it: `aporia:upsert_thesis` partial-merges, so one field moves without restating the rest.
- **Personas.** If `personas` is empty or under-mapped, author them WITH the human via the **aporia-persona-mapper** skill — the Socratic role → situation → painSeverity interview, written with `aporia:upsert_personas`. Don't duplicate that interview here; run that skill. If personas exist but are misaligned, refine them (pass the `id` from `aporia:fetch_personas`). Once they exist, you may link the personas who *have* the problem onto the thesis via `aporia:upsert_thesis { personaIds }`.
- **Principles.** If `principles` are empty (or don't trace to the bet), author **3–5** WITH the human and write them via `aporia:upsert_principles` (read what exists first with `aporia:fetch_principles`; refine/retire by `id` rather than cloning). A principle is a **testable guardrail that operationalizes the thesis-bet** — a product-wide law you can hold a feature against and say *"this violates it."* It must be product-specific, falsifiable, and trace to the `insight`; each carries a `statement` + a `rationale` (why it matters, tied to the bet). Keep the `statement` one crisp, testable sentence (the law itself — not markdown-structured); write the `rationale` as tight **markdown** prose ([content-style](../aporia-session-notes/references/content-style.md)). **Reject platitudes** ("be user-centric", "keep it simple") — those aren't principles. And don't confuse a principle with a node-scoped constraint-note: a principle is one of the few *foundational* laws, not emergent discourse.
- **The floor.** At minimum get the one-sentence `product` + the core `problem` so features have something real to bind to. If the human genuinely can't articulate the thesis yet, that is itself the finding — surface it (it's the work the from-scratch Assistant session exists to do) rather than inventing one. **An empty Constitution is filled, not skipped; and never fabricated from the code.**

Then, for each scope you're about to scan, call `aporia:search_graph { keyPrefix }` (or `{ group }`) to see what's already mapped, so a re-scan converges instead of duplicating.

### Phase 1 — Scope

Split the repo into the **scopes the team thinks in** — a package, a bounded context, a service (e.g. `pim`, `indexing`, `billing`, `admin`). Each becomes a `scopeKey`, scanned and pushed independently with `completeScope: true` on its final page so its tombstone sweep is correct. Never scan the whole repo as one blob.

### Phase 2 — Inventory (observation, high recall, no judgement)

Per scope, gather raw facts — do not filter yet: schema/model/table definitions (entities); modules, services, packages, deploy units, UI surfaces (components); which module owns which entity; module→module dependencies; entity→entity relations (FKs/refs).

### Phase 3 — Distill (interpretation — this is the whole job)

Apply the four structural disciplines — D5 (intent) is Phase 4 (full text in the reference):
- **D1 Plumbing filter** — keep only what a non-engineer would call "part of my product." Drop DTOs, config, utils, adapters, generic CRUD, fixtures, scaffolding.
- **D2 Grain** — Entity = aggregate root; Component = meaningful module; Feature = slice of user value. **> ~30 nodes of one type in a scope ⇒ you're too fine; recluster.**
- **D3 Domain language** — name things what the team *says* (UI strings → routes → API → comments → README → identifiers last). `tbl_inv_ln` → "Invoice Line".
- **D4 Evidence + confidence** — every node cites `externalRefs`; nothing ungrounded. `confidence` grades **realization read from code** — how completely the repo *builds* the thing — **not** certainty of its *intent*; the two are orthogonal (a feature can be `confirmed`-built with its `intent` still empty). Grade by build-state evidence (the ladder is in Phase 4); when a side can't be established, grade **down**. A pure scanned entity/component fact may carry `provisional`/none and still render as confirmed reality — never assert a grade you can't cite.

### Phase 4 — Propose features + grade realization (D5)

Infer candidate `feature` nodes and their `realized_by` / `touches` bindings from routes/UI/folders. **Do not blanket-stamp them `hypothesis`** — that buries the built features under the vapor, the exact failure this phase exists to prevent. The code shows how completely each feature is realized; read the whole slice and grade it.

**The Feature Realization Ladder** — `confidence` answers *"does the product actually DO this, end to end?"*, read from code and **orthogonal to intent** (which stays empty until Phase 5). Probe the slice — **surface** (a real route/page a user reaches), **logic** (a backend handler/query/mutation it calls), **persistence/IO** (real storage or a real external system), **data realness** (real values, not `MOCK_*`/fixtures/hard-coded returns/`TODO`), **gating** (flag / env / role-or-beta guard / `if (false)` / killswitch) — then grade, **citing the deciding refs**:

- **`confirmed`** — surface ↔ logic ↔ persistence wired, real data, reached on the **default path** (not gated). The code demonstrably ships it. *(Intent may still be `""`: you see that it works, not why.)*
- **`provisional`** — realized but **conditional**: behind a flag / env / role / beta / killswitch, or one clearly-partial seam in an otherwise-wired slice. Real, not unconditionally on — cite the gate.
- **`hypothesis`** — **half-baked**: only one side exists (UI with no backend, or backend with no surface), or the path is satisfied by mock / stub / fixture / hard-coded data or pervasive `TODO`. Cite the missing side or the stub.

**These grades surface as the map's Implementation axis**, not as a certainty: `confirmed` → *Implemented* · `provisional`/`hypothesis` → *Partial* · a `planned` feature → *Not implemented*. Confidence here is the realization grade read from code — the team's certainty about a feature's *why* lives on its notes, never on the node. Keeping the map honest as the code evolves after this bootstrap is the **aporia-sync** skill's job (run per PR).

When you can't establish a side, grade **down**, never up. A surface with **no implementation on either side** (coming-soon shell, empty route, stated-but-unbuilt) is intent, not structure — push it `planned: true` (lands `intended`, sweep-exempt) at `hypothesis`. The realization grade is from code; the **rationale/why** is Phase 5's — never fabricate it.

### Phase 5 — Elicit (draft-first, highest-leverage only)

Show the draft map; ask the human to *react*. Budget their attention — ask only about: features the scan couldn't infer, the *why* behind the most central nodes, the success criteria, the persona each feature serves. When they correct you ("no, it's actually X"), that correction is the meaning that normally evaporates — capture it (Phase 7) as a **decision** (+ a **tension** if it contradicts the as-built). Never silently fold an unstated *why* into the scan.

### Phase 6 — Push structure with `aporia:apply_scan`

Push **per scope**, ≤200 nodes+edges per call (page large scopes; `completeScope: true` only on the final page of each scope). `data.type` MUST equal the node `type`. Keys are stable identity — survive file moves; NOT the file path.

```jsonc
// aporia:apply_scan input
{ "scopeKey": "billing", "completeScope": true,
  "nodes": [
    { "key": "entity:billing.invoice", "type": "entity", "name": "Invoice",
      "summary": "A bill issued to a customer", "group": "Billing",
      "externalRefs": [{ "path": "src/billing/invoice.ts", "symbol": "Invoice" }],
      "confidence": "provisional",
      "data": { "type": "entity", "fields": [{ "name": "total", "type": "number" }] } },
    { "key": "component:billing-service", "type": "component", "name": "Billing Service",
      "group": "Billing", "externalRefs": [{ "path": "src/billing/service.ts" }],
      "data": { "type": "component", "componentKind": "service" } },
    { "key": "feature:billing.checkout", "type": "feature", "name": "Checkout",
      // realization read from code: page ↔ submitCheckout ↔ Invoice, not gated ⇒ confirmed.
      // intent stays "" — you observe THAT it works; the WHY is elicited in Phase 5.
      "confidence": "confirmed",
      "externalRefs": [{ "path": "src/billing/checkout-page.tsx" },
                       { "path": "src/billing/checkout.ts", "symbol": "submitCheckout" }],
      "data": { "type": "feature", "intent": "",
                "successCriteria": [], "relatedPersonaIds": [] } }
  ],
  "edges": [
    { "type": "owns",        "fromKey": "component:billing-service", "toKey": "entity:billing.invoice" },
    { "type": "realized_by", "fromKey": "feature:billing.checkout",  "toKey": "component:billing-service" },
    { "type": "touches",     "fromKey": "feature:billing.checkout",  "toKey": "entity:billing.invoice" }
  ] }
```

Key formats: `entity:<domain>.<name>` · `component:<name>` · `feature:<group>.<slug>`.
Feature `data` keys are all **required** — the union won't validate without them (a missing key aborts the whole batch). `intent` is a required string: leave it `""` here and elicit it in Phase 5 (record the *why* as a decision note) — never drop the key. `successCriteria` is an array. `relatedPersonaIds` is an array of **persona ids from `aporia:pull_constitution`'s `personas[].id`** (never names or guesses) — `[]` if unknown; the link is confirmed during elicitation. Every node (features included) cites `externalRefs` — for a feature, the route/page/folder it was inferred from (D4). A feature **always carries an explicit `confidence`** (its realization grade): omit it and a scanned feature renders *confirmed* by default in the map, hiding the half-baked ones.
A node's `name` and `summary` are map typography — a 1–4-word domain noun and a one-sentence summary; every authored field's shape is in [content-style](../aporia-session-notes/references/content-style.md).
Edge vocabulary: `relates_to` (entity↔entity, label carries cardinality) · `depends_on` (component↔component / feature↔feature) · `owns` (component→entity) · `realized_by` (feature→component) · `touches` (feature→entity). The response reports `skippedEdges` (an endpoint key didn't resolve — push the scope that *defines* a node before one that only references it, or include both in the same batch) and, for `completeScope`, `removed`/`removedEdges` (tombstoned because the scan no longer reports them).

### Phase 7 — Record the why with `aporia:record_notes`

Capture the elicited reasoning as notes, each targeting the node(s) it's about by `key` — each a ≤60-char headline `title` over a markdown `body`, sized per [content-style](../aporia-session-notes/references/content-style.md):

```jsonc
// aporia:record_notes input
{ "notes": [
  { "kind": "decision",
    "title": "Invoices are immutable once issued",
    "body": "Once issued, an invoice never changes. Corrections are made by issuing a separate **credit note**, never by editing the original.",
    "rationale": "Audit and tax records require an immutable issued document.", "isConstraint": true,
    "targets": [{ "refType": "node", "ref": "entity:billing.invoice", "role": "primary" }] },
  { "kind": "question",
    "title": "Which tax engine for Checkout?",
    "body": "Checkout needs tax computation, but the engine isn't chosen — *build* vs. **Stripe Tax** vs. **Avalara**. Gates the EU rollout.",
    "targets": [{ "refType": "node", "ref": "feature:billing.checkout", "role": "primary" }] }
] }
```

Notes land `provisional`/`open` for the team to curate. Don't invent rationale — if the *why* wasn't stated, it's a **question**, not a decision.

## Acceptance checklist (before declaring done)

- [ ] The Constitution grounded the scan FIRST — the thesis, the personas features bind to, and 3–5 testable principles exist and were authored WITH the human; an empty Constitution was filled via `aporia:upsert_thesis` / aporia-persona-mapper / `aporia:upsert_principles`, never skipped or invented from the code.
- [ ] Passes the Recognition Test — recognizable to a founder, navigable by an engineer.
- [ ] No node fails the plumbing filter (D1); no node type exceeds ~30 per scope without reason (D2).
- [ ] Every node uses domain language (D3) and cites `externalRefs` (D4).
- [ ] No `feature.intent` was invented — it's a human answer or left empty/open (D5).
- [ ] Every feature carries a realization grade earned from code (built ⇒ confirmed · gated ⇒ provisional · half-baked ⇒ hypothesis) — none left at a blanket default, each cites the deciding ref (D4).
- [ ] Every `aporia:apply_scan` returned `skippedEdges: 0` (or you understand each skip).
- [ ] Corrections were recorded as decisions/tensions, not silently applied.
- [ ] Every authored string sized for its surface — node names 1–4 words, summaries one sentence, note titles ≤60-char headlines over markdown bodies ([content-style](../aporia-session-notes/references/content-style.md)).

## Anti-patterns (reject)

The raw import graph as edges · one node per file/function/table · entities named after DB tables · invented features with confident rationale · **every feature stamped `hypothesis` no matter how completely the code builds it** · "User Management", "Core", "Utils", "Settings" as features · a 200-node map "for completeness" · 40 questions instead of a draft to react to · **scanning onto an empty Constitution** (every feature left ungrounded) · **inventing the thesis or a persona from the code** instead of eliciting it from the human.
