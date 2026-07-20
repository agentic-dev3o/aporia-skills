---
name: aporia-onboard
description: >-
  Onboards a codebase into Aporia's living map through the Aporia MCP server —
  the one-time deep scan that becomes a graph a founder recognizes in 30 seconds.
  Grounds on the product Constitution first (an empty thesis / personas /
  principles is authored WITH the human before scanning), distills the repo into
  domain entities, components, and features with cited evidence, elicits the
  intent the code can't show, and pushes with aporia:apply_scan. Use when
  onboarding a repo into Aporia, scanning code into the map, bootstrapping the
  product graph, defining the thesis, personas, and principles, or running the
  first as-built scan of an existing codebase. For the per-PR re-scan of an
  already-mapped product, use aporia-sync instead.
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
- [ ] Phase 3 — Distill: apply D1–D4 + D6 (plumbing filter, grain, language, evidence, legibility)
- [ ] Phase 4 — Propose features; run the Realization Probe → structure as_built + deficiency flags / planned (D5)
- [ ] Phase 5 — Elicit the why from the human (draft-first)
- [ ] Phase 6 — aporia:apply_scan per scope (one sessionId + observed on every page; completeScope on the final page)
- [ ] Phase 7 — aporia:record_notes: the elicited decisions, questions, tensions
```

### Phase 0 — Ground the Constitution (Ring 0 first)

Call `aporia:pull_constitution` → the product's `thesis`, `principles`, and `personas` (each persona carries an `id` — a feature's `relatedPersonaIds` uses those ids, never names). The Constitution is the **ground the scan stands on**: every feature is graded and bound against it (D4/D5), so scanning onto an empty or wrong Constitution produces a map with nothing to anchor to. **Establish it before Phase 1** — and like the rest of this skill, author it **WITH the human, never from the code**:

- **Thesis — elicit the BET, don't grade the problem.** If `thesis` is null/empty, run a short, conversational interview (reflect each answer back before the write), then write it with `aporia:upsert_thesis`:
  1. `product` — what is it, who's it for, and the **wedge** (what it does that's different)? One or two sentences.
  2. `problem.statement` — the problem it attacks, stated once and concretely (the moment it bites) — not graded.
  3. `insight` — **the bet: the non-obvious thing the founder believes that makes this win, and why _now_ is the moment.** This is the centre of a thesis — a claim they could be wrong about, not a feature list or a restatement of the problem. If they can't name it yet, that gap *is* the finding — surface it; never invent one.

  The `problem.statement` and `insight` render as **markdown** — write them as tight prose (a short paragraph, emphasis on the load-bearing claim), never a wall; keep `product` to one or two sentences ([content-style](references/shared/content-style.md)). It lands `draft` / `hypothesis`. **Do not ask _painful / urgent / recognized_ here** — DUR is a **challenge** lens (does the problem hold up?), applied when the in-app Assistant pressure-tests the thesis in a session, not a scan's lead question. Grading the problem before the bet is even stated is the rubric that makes a thesis feel hollow; it also just re-asks what persona `painSeverity` already captures, better, from a concrete situation. If a thesis already EXISTS but has drifted, **refine** it: `aporia:upsert_thesis` partial-merges, so one field moves without restating the rest.
- **Personas.** If `personas` is empty or under-mapped, author them WITH the human via the **aporia-persona-mapper** skill — the Socratic role → situation → painSeverity interview, written with `aporia:upsert_personas`. Don't duplicate that interview here; run that skill. If personas exist but are misaligned, refine them (pass the `id` from `aporia:fetch_personas`). Once they exist, you may link the personas who *have* the problem onto the thesis via `aporia:upsert_thesis { personaIds }`.
- **Principles.** If `principles` are empty (or don't trace to the bet), author **3–5** WITH the human and write them via `aporia:upsert_principles` (read what exists first with `aporia:fetch_principles`; refine/retire by `id` rather than cloning). A principle is a **testable guardrail that operationalizes the thesis-bet** — a product-wide law you can hold a feature against and say *"this violates it."* It must be product-specific, falsifiable, and trace to the `insight`; each carries a `statement` + a `rationale` (why it matters, tied to the bet). Keep the `statement` one crisp, testable sentence (the law itself — not markdown-structured); write the `rationale` as tight **markdown** prose ([content-style](references/shared/content-style.md)). **Reject platitudes** ("be user-centric", "keep it simple") — those aren't principles. And don't confuse a principle with a node-scoped constraint-note: a principle is one of the few *foundational* laws, not emergent discourse.
- **The floor.** At minimum get the one-sentence `product` + the core `problem` so features have something real to bind to. If the human genuinely can't articulate the thesis yet, that is itself the finding — surface it (it's the work the from-scratch Assistant session exists to do) rather than inventing one. **An empty Constitution is filled, not skipped; and never fabricated from the code.**

The `aporia:pull_constitution` response also carries `canonicalRef` — if it's declared and your checkout is off it (or dirty), **stop before scanning**: every push would be force-previewed and write nothing (Phase 6). Onboard from the canonical ref on a clean tree.

Then, for each scope you're about to scan, call `aporia:search_graph { keyPrefix }` (or `{ group }`) to see what's already mapped, so a re-scan converges instead of duplicating.

### Phase 1 — Scope

Split the repo into the **scopes the team thinks in** — a package, a bounded context, a service (e.g. `pim`, `indexing`, `billing`, `admin`). Each becomes a `scopeKey`, scanned and pushed independently with `completeScope: true` on its final page so its tombstone sweep is correct. Never scan the whole repo as one blob.

### Phase 2 — Inventory (observation, high recall, no judgement)

Per scope, gather raw facts — do not filter yet: schema/model/table definitions (entities); modules, services, packages, deploy units, UI surfaces (components); which module owns which entity; module→module dependencies; entity→entity relations (FKs/refs). Also name the scope's **entry surfaces** (screens, API routes, webhooks, crons — what the outside world invokes) and its **externals** (vendors/APIs it depends on but doesn't own) — D6's rails; a scope with zero entries is a smell.

### Phase 3 — Distill (interpretation — this is the whole job)

Apply the five structural disciplines — D5 (intent) is Phase 4 (full text in the reference):
- **D1 Plumbing filter** — keep only what a non-engineer would call "part of my product." Drop DTOs, config, utils, adapters, generic CRUD, fixtures, scaffolding.
- **D2 Grain** — Entity = aggregate root; Component = meaningful module; Feature = slice of user value. **> ~30 nodes of one type in a scope ⇒ you're too fine; recluster.**
- **D3 Domain language** — name things what the team *says* (UI strings → routes → API → comments → README → identifiers last). `tbl_inv_ln` → "Invoice Line".
- **D6 Architecture legibility** — every component's `componentKind` follows the reference's decision table (`ui` = human screen · `trigger` = machine-invocable route/webhook/cron · `agent` = LLM loop with tools · `tool` = capability bundle an agent calls · `service`/`store`/`external`/`module`); every `depends_on` edge carries a ≤4-word verb `label` stating what crosses it ("mints Convex JWT", "gated: ENABLE_V4"); a component's `data.sub` is its evidence-backed signature line ("POST /api/chat", "ToolLoopAgent · 30 steps") — concrete numbers over adjectives, models never as nodes; an `external`'s `data.domain` names the vendor ("clerk.com"); lifecycle rides the group name ("NOA v3 (legacy)"); a risky external (unofficial API, deprecated SDK) gets a `tension` note.
- **D4 Evidence, honest structure** — every node cites `externalRefs`; nothing ungrounded, never a claim you can't cite. A scanned entity/component/feature carries **no stored realization grade** — how completely a feature is built is not written into a field. You report the structure you can cite `as_built`, and hold a built-but-hollow feature at Partial with an **open `blocksImplementation` note** naming its missing side (Phase 4 carries the coverage rule). `confidence` lives only on **authored intent** — a `planned: true` feature you author-as-`hypothesis` (Phase 4); on scanned structure it is absent.

### Phase 4 — Propose features + run the Realization Probe (D5)

Infer candidate `feature` nodes and their `realized_by` / `touches` bindings from routes/UI/folders. **Do not blanket-stamp them `planned`** — that buries the built features under the vapor, the exact failure this phase exists to prevent. The code shows how completely each feature is realized; read the whole slice.

**The Realization Probe** answers *"does the product actually DO this, end to end?"* — read from code, **orthogonal to intent** (which stays empty until Phase 5), and it does **not** produce a grade you write into the node. Run it on each candidate feature exactly as **[references/shared/realization-probe.md](references/shared/realization-probe.md)** defines it — probe the five signals (surface · logic · persistence/IO · data realness · gating) and conclude in one of its **three actions**, never a stored grade. The half-baked / mocked / gated action is the one to watch: report the structure that exists `as_built` **and** record an open `blocksImplementation` note naming the missing side (Phase 7), which holds the feature at Partial.

**Implementation is derived server-side from binding coverage + open deficiency flags; nothing you write into a field can raise or hold it — only structure and flags can.** The level rolls up from what you can cite — the coverage table in **[realization-probe](references/shared/realization-probe.md)**, the same rule **aporia-sync** re-runs per PR.

The probe reads from code; the **rationale/why** is Phase 5's — never fabricate it.

### Phase 5 — Elicit (draft-first, highest-leverage only)

Show the draft map; ask the human to *react*. Budget their attention — ask only about: features the scan couldn't infer, the *why* behind the most central nodes, the success criteria, the persona each feature serves. When they correct you ("no, it's actually X"), that correction is the meaning that normally evaporates — capture it (Phase 7) as a **decision** (+ a **tension** if it contradicts the as-built). Never silently fold an unstated *why* into the scan.

### Phase 6 — Push structure with `aporia:apply_scan`

Push **per scope**, ≤200 nodes+edges per call (page large scopes; `completeScope: true` only on the final page of each scope). Mint ONE `sessionId` (a UUID) for the whole onboarding run and pass it — plus `observed` from git: `{ ref, sha }` (`git rev-parse --abbrev-ref HEAD` / `git rev-parse HEAD`), adding `dirty: true` if `git status --porcelain` is non-empty — on **every** page of **every** scope, so a scope's final `completeScope` page can't tombstone its earlier pages and each node/edge records the worldline it was seen at. `data.type` MUST equal the node `type`. Keys are stable identity — survive file moves; NOT the file path.

```jsonc
// aporia:apply_scan input — sessionId + observed ride EVERY page of EVERY scope
{ "scopeKey": "billing", "completeScope": true,
  "sessionId": "3f2a…", "observed": { "ref": "main", "sha": "a1b2c3d" },
  "nodes": [
    { "key": "entity:billing.invoice", "type": "entity", "name": "Invoice",
      "summary": "A bill issued to a customer", "group": "Billing",
      "externalRefs": [{ "path": "src/billing/invoice.ts", "symbol": "Invoice" }],
      "data": { "type": "entity", "fields": [{ "name": "total", "type": "number" }] } },
    { "key": "component:billing-service", "type": "component", "name": "Billing Service",
      "group": "Billing", "externalRefs": [{ "path": "src/billing/service.ts" }],
      // sub = evidence-backed signature line (optional); domain = vendor domain on external kinds
      "data": { "type": "component", "componentKind": "service", "sub": "invoices · 12 operations" } },
    { "key": "feature:billing.checkout", "type": "feature", "name": "Checkout",
      // core wired on the default path (page ↔ submitCheckout ↔ Invoice): report that structure as_built + edges, no confidence field.
      // Phase 7 flags its one hollow seam — a hard-coded tax — with blocksImplementation, holding it at Partial until real logic lands.
      // intent stays "" — you observe THAT it works; the WHY is elicited in Phase 5.
      "externalRefs": [{ "path": "src/billing/checkout-page.tsx" },
                       { "path": "src/billing/checkout.ts", "symbol": "submitCheckout" }],
      "data": { "type": "feature", "intent": "",
                "successCriteria": [], "relatedPersonaIds": [] } }
  ],
  "edges": [
    // depends_on always carries a ≤4-word verb label — what crosses the edge (D6)
    { "type": "owns",        "fromKey": "component:billing-service", "toKey": "entity:billing.invoice" },
    { "type": "realized_by", "fromKey": "feature:billing.checkout",  "toKey": "component:billing-service" },
    { "type": "touches",     "fromKey": "feature:billing.checkout",  "toKey": "entity:billing.invoice" }
  ] }
```

Key formats: `entity:<domain>.<name>` · `component:<name>` · `feature:<group>.<slug>`.
Feature `data` keys are all **required** — the union won't validate without them (a missing key aborts the whole batch). `intent` is a required string: leave it `""` here and elicit it in Phase 5 (record the *why* as a decision note) — never drop the key. `successCriteria` is an array. `relatedPersonaIds` is an array of **persona ids from `aporia:pull_constitution`'s `personas[].id`** (never names or guesses) — `[]` if unknown; the link is confirmed during elicitation. Every node (features included) cites `externalRefs` — for a feature, the route/page/folder it was inferred from (D4). A scanned feature carries **no `confidence`** — omit the field; its Implementation is derived from coverage (its `as_built` bindings and any open `blocksImplementation` flag), and a built-but-hollow feature is held at Partial by that flag, not by a grade. `confidence` appears only when you author a `planned: true` feature (`'hypothesis'`).
A node's `name` and `summary` are map typography — a 1–4-word domain noun and a one-sentence summary; every authored field's shape is in [content-style](references/shared/content-style.md).
Edge vocabulary: `relates_to` (entity↔entity, label carries cardinality) · `depends_on` (component↔component / feature↔feature — **label mandatory**: ≤4 words for what crosses it, conditions included, e.g. "gated: ENABLE_V4") · `owns` (component→entity) · `realized_by` (feature→component) · `touches` (feature→entity). The response reports `skippedEdges` (an endpoint key didn't resolve — push the scope that *defines* a node before one that only references it, or include both in the same batch) and, for `completeScope`, the `removed`/`removedEdges` **counts** (tombstoned because the scan no longer reports them). **A response carrying `mode: "preview"` wrote NOTHING**: the product declares a canonical ref and your `observed` is off it (a branch) or dirty — as-built truth is only written from the canonical ref on a clean tree. Never proceed as if that scan landed; onboard from the canonical ref, or hand the human the preview delta. A large run can also hit `RATE_LIMITED` — wait the returned `retryAfter`, then retry the same call.

### Phase 7 — Record the why with `aporia:record_notes`

Capture the elicited reasoning **and** every deficiency the probe found, each note targeting the node(s) it's about by `key` — each a ≤60-char headline `title` over a markdown `body`, sized per [content-style](references/shared/content-style.md):

- a **`blocksImplementation` deficiency flag** (a `tension` note with `blocksImplementation: true`, targeting the feature) on every feature the Phase 4 probe found **half-baked / mocked** — naming the missing side / the mock / the `TODO`. This IS the Partial signal and the visible "what's left to build" beside the pill; without it a feature whose structure exists but is a mock reads *Implemented*. The flag lands **sync-watched**: the later sync scan that proves the missing side landed resolves it with that evidence, and coverage flips the feature up. (A feature that's Partial only because a binding is still `planned` needs no flag — the unbuilt binding already says so.)
- a **decision** for a stated *why* (with its rationale); a **question** for an unstated one; a **tension** when a correction contradicts the as-built.

```jsonc
// aporia:record_notes input
{ "notes": [
  { "kind": "decision",
    "title": "Invoices are immutable once issued",
    "body": "Once issued, an invoice never changes. Corrections are made by issuing a separate **credit note**, never by editing the original.",
    "rationale": "Audit and tax records require an immutable issued document.", "isConstraint": true,
    "targets": [{ "refType": "node", "ref": "entity:billing.invoice", "role": "primary" }] },
  { "kind": "tension", "blocksImplementation": true,
    "title": "Checkout tax is hard-coded to 0",
    "body": "`checkout.ts` returns a hard-coded `0` tax — no real engine is wired. Holds Checkout at Partial until the missing tax logic lands.",
    "targets": [{ "refType": "node", "ref": "feature:billing.checkout", "role": "primary" }] },
  { "kind": "question",
    "title": "Which tax engine for Checkout?",
    "body": "Checkout needs tax computation, but the engine isn't chosen — *build* vs. **Stripe Tax** vs. **Avalara**. Gates the EU rollout.",
    "targets": [{ "refType": "node", "ref": "feature:billing.checkout", "role": "primary" }] }
] }
```

Notes land `provisional`/`open` for the team to curate. Don't invent rationale — if the *why* wasn't stated, it's a **question**, not a decision. A `blocksImplementation` flag can only hold a feature BELOW Implemented, never raise it — it's the agent counterpart to the canvas's human *Flag as partial*.

## Acceptance checklist (before declaring done)

- [ ] The Constitution grounded the scan FIRST — the thesis, the personas features bind to, and 3–5 testable principles exist and were authored WITH the human; an empty Constitution was filled via `aporia:upsert_thesis` / aporia-persona-mapper / `aporia:upsert_principles`, never skipped or invented from the code.
- [ ] Passes the Recognition Test — recognizable to a founder, navigable by an engineer.
- [ ] No node fails the plumbing filter (D1); no node type exceeds ~30 per scope without reason (D2).
- [ ] Every node uses domain language (D3) and cites `externalRefs` (D4).
- [ ] Every component's kind follows the D6 table; entries and externals identified per scope; every `depends_on` edge verb-labeled; `sub`/`domain` filled where evidence supports them (D6).
- [ ] No `feature.intent` was invented — it's a human answer or left empty/open (D5).
- [ ] The Realization Probe drove every feature to structure, not a grade: fully-wired features report all bindings `as_built`; every half-baked / mocked feature carries an **open `blocksImplementation` flag** citing its missing side (no mock left reading Implemented); nothing-on-either-side pushed `planned` at `hypothesis` (D4/D5).
- [ ] Every `aporia:apply_scan` returned `skippedEdges: 0` (or you understand each skip) — and every response **applied** (no `mode: "preview"`: a canonical-ref/dirty preview writes nothing).
- [ ] Corrections were recorded as decisions/tensions, not silently applied.
- [ ] Every authored string sized for its surface — node names 1–4 words, summaries one sentence, note titles ≤60-char headlines over markdown bodies ([content-style](references/shared/content-style.md)).

## Anti-patterns (reject)

The raw import graph as edges · one node per file/function/table · entities named after DB tables · invented features with confident rationale · **a mocked feature pushed `as_built` with no `blocksImplementation` flag** (it will read Implemented) · **every feature pushed `planned` no matter how completely the code builds it** (burying built features under vapor) · "User Management", "Core", "Utils", "Settings" as features · a 200-node map "for completeness" · 40 questions instead of a draft to react to · **scanning onto an empty Constitution** (every feature left ungrounded) · **inventing the thesis or a persona from the code** instead of eliciting it from the human · an unlabeled `depends_on` between services · a model/LLM as a graph node (it rides `sub`) · an API route classified `service` instead of `trigger` (D6).
