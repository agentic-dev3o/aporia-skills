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

Tools you'll use: `aporia:pull_constitution` (ground), `aporia:fetch_personas` / `aporia:fetch_principles` / `aporia:upsert_thesis` / `aporia:upsert_personas` / `aporia:upsert_principles` / `aporia:fetch_design_tokens` / `aporia:upsert_design_tokens` (author the Constitution when it's empty), `aporia:search_graph` (see what's mapped), `aporia:apply_scan` (push structure + proposed features), `aporia:record_notes` (capture the elicited why).

## Workflow

Copy this checklist into your response and check off each phase:

```
Scan progress:
- [ ] Phase 0 — Ground the Constitution: aporia:pull_constitution; if the thesis/personas/principles are empty (or misaligned), DEFINE them WITH the human (aporia:upsert_thesis + the aporia-persona-mapper interview + aporia:upsert_principles) BEFORE scanning; then aporia:search_graph per scope
- [ ] Phase 1 — Scope: split the repo into the subsystems the team thinks in
- [ ] Phase 2 — Inventory: raw facts per scope (high recall, no judgement)
- [ ] Phase 3 — Distill: apply D1–D4 + D6 (plumbing filter, grain, language, evidence, legibility)
- [ ] Phase 4 — Propose features; run the Realization Probe → structure as_built + missing-side tasks / planned (D5)
- [ ] Phase 5 — Elicit the why from the human (draft-first)
- [ ] Phase 6 — aporia:apply_scan per scope (one sessionId + observed on every page; completeScope on the final page)
- [ ] Phase 7 — aporia:record_notes: the elicited decisions, questions, tensions
```

### Phase 0 — Ground the Constitution (Ring 0 first)

Call `aporia:pull_constitution` → the product's `thesis`, `principles`, and `personas` (each persona carries an `id` — a feature's `relatedPersonaIds` uses those ids, never names). The Constitution is the **ground the scan stands on**: every feature is graded and bound against it (D4/D5), so scanning onto an empty or wrong Constitution produces a map with nothing to anchor to. **Establish it before Phase 1** — and like the rest of this skill, author it **WITH the human, never from the code**:

- **Thesis — elicit the BET, don't grade the problem.** If `thesis` is null/empty, run a short, conversational interview (reflect each answer back before the write), then write it with `aporia:upsert_thesis`:
  1. `product` — what is it, who's it for, and the **wedge** (what it does that's different)? One or two sentences.
  2. `problem.statement` — the problem it attacks, stated once and concretely (the moment it bites) — not graded.
  3. The **argument**, as four short parts, one claim each — `shift` (what changed in the world) → `bet` (**the non-obvious thing the founder believes that makes this win** — the centre, a claim they could be wrong about, never a feature list or a restatement of the problem) → `whyNow` (why this moment, not two years ago) → `endState` (what winning unlocks). If they can't name the bet yet, that gap *is* the finding — surface it; never invent one.
  4. `falsifiedIf` — **what would prove the bet wrong.** A thesis that cannot name its own falsifier isn't a bet, it's a slogan. Ask for it in the same breath as the bet.

  Every part is **length-capped and rejected over-length, never truncated** — `bet` at 240 chars (~35 words), `shift` / `whyNow` / `endState` / `product` / `problem.statement` at 200 (~30 words), `falsifiedIf` at 140 (~20 words). The caps are the point: the Thesis view renders the argument as four bounded cards, so a part that doesn't fit isn't a formatting problem, it's two claims wearing one field — split it or cut it ([content-style](references/shared/content-style.md)). A thesis carries no confidence grade and no lifecycle status: how sure the team is, is answered by whether `falsifiedIf` has fired, not by a field. **Do not ask _painful / urgent / recognized_ here** — DUR is a **challenge** lens (does the problem hold up?), applied when the in-app Assistant pressure-tests the thesis in a session, not a scan's lead question. Grading the problem before the bet is even stated is the rubric that makes a thesis feel hollow; it also just re-asks what persona `painSeverity` already captures, better, from a concrete situation. If a thesis already EXISTS but has drifted, **refine** it: `aporia:upsert_thesis` partial-merges, so one field moves without restating the rest.
- **Personas.** If `personas` is empty or under-mapped, author them WITH the human via the **aporia-persona-mapper** skill — the Socratic role → situation → painSeverity interview, written with `aporia:upsert_personas`. Don't duplicate that interview here; run that skill. If personas exist but are misaligned, refine them (pass the `id` from `aporia:fetch_personas`). Once they exist, you may link the personas who *have* the problem onto the thesis via `aporia:upsert_thesis { personaIds }`.
- **Principles.** If `principles` are empty (or don't trace to the bet), author **3–5** WITH the human and write them via `aporia:upsert_principles` (read what exists first with `aporia:fetch_principles`; refine/retire by `id` rather than cloning). A principle is a **testable guardrail that operationalizes the thesis-bet** — a product-wide law you can hold a feature against and say *"this violates it."* It must be product-specific, falsifiable, and trace to the `bet`; each carries a `statement` + a `rationale` (why it matters, tied to the bet). Keep the `statement` one crisp, testable sentence (the law itself — not markdown-structured); write the `rationale` as tight **markdown** prose ([content-style](references/shared/content-style.md)). **Reject platitudes** ("be user-centric", "keep it simple") — those aren't principles. And don't confuse a principle with a node-scoped Rule (`record_notes` kind `rule`): a principle is one of the few *foundational* laws, not emergent discourse.
- **Design tokens.** If the product has a real UI, read what's there with `aporia:fetch_design_tokens` and write it with `aporia:upsert_design_tokens`. These are the tokens of **the product being onboarded**, never Aporia's own theme, and they are **authored, never derived**: a role holds one value per appearance (`light` and `dark`), entered directly, because most palettes are not derivable from two or three inputs. Take them from what the repo already ships — its stylesheet, its theme config — rather than inventing a palette; that is one of the few Constitution facts the code CAN tell you. `roles` and `states` are sent as the **whole ordered list** and replace that group wholesale, so fetch first, edit the array you got back, and send it whole: sending one role alone drops the rest. Order is meaningful (surfaces, then the hairline, then the ink, then the hued roles), so the palette reads as a ramp. The read side emits ONE generic stylesheet of plain custom properties, both appearances together — Aporia ships no per-framework adapters, because which role fills which slot and which file it lands in are per-repo facts it would be guessing at. A coding agent maps it into the repo's own idiom.
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
- **D4 Evidence, honest structure** — every node cites `externalRefs`; nothing ungrounded, never a claim you can't cite. A scanned entity/component/feature carries **no stored realization grade**: **Built is binary and derived** — the feature is `as_built` AND carries at least one `externalRef` of kind `code`. You report the structure you can cite `as_built`, and name what a built-but-hollow feature is still missing with an **open `task`** on it (Phase 4 carries the rule). A cited document rides `kind: "doc"` and never counts as evidence. A node carries **no certainty field of any kind** — never stamp one; "how sure are we?" is read off the node's open discourse, so genuine doubt is an open **question** you file, not a grade you assert.

### Phase 4 — Propose features + run the Realization Probe (D5)

Infer candidate `feature` nodes and their `realized_by` / `touches` bindings from routes/UI/folders. **Do not blanket-stamp them `planned`** — that buries the built features under the vapor, the exact failure this phase exists to prevent. The code shows how completely each feature is realized; read the whole slice.

**The Realization Probe** answers *"does the product actually DO this, end to end?"* — read from code, **orthogonal to intent** (which stays empty until Phase 5), and it does **not** produce a grade you write into the node. Run it on each candidate feature exactly as **[references/shared/realization-probe.md](references/shared/realization-probe.md)** defines it — probe the five signals (surface · logic · persistence/IO · data realness · gating) and conclude in one of its **three actions**, never a stored grade. The half-baked / mocked / gated action is the one to watch: report the structure that exists `as_built` **and** file a `task` naming the missing side (Phase 7).

**Built is derived server-side and binary — `as_built` AND at least one `externalRef` of kind `code`, both of them yours to report; nothing you write into a field can raise it, and there is no middle rung.** The table is in **[realization-probe](references/shared/realization-probe.md)**, the same rule **aporia-sync** re-runs per PR.

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
      // core wired on the default path (page ↔ submitCheckout ↔ Invoice): report that structure as_built + edges, no certainty field.
      // Phase 7 files a task for its one hollow seam — a hard-coded tax — naming what's still missing.
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
Feature `data` keys are all **required** — the union won't validate without them (a missing key aborts the whole batch). `intent` is a required string: leave it `""` here and elicit it in Phase 5 (record the *why* as a decision note) — never drop the key. `successCriteria` is an array. `relatedPersonaIds` is an array of **persona ids from `aporia:pull_constitution`'s `personas[].id`** (never names or guesses) — `[]` if unknown; the link is confirmed during elicitation. Every node (features included) cites `externalRefs` — for a feature, the route/page/folder it was inferred from (D4). A feature carries **no certainty field** — whether it is **Built** is derived from its own `state` + its own `externalRefs` of kind `code`, and a built-but-hollow feature is described by an open task naming the gap, not by a grade.
An `entity`'s `data.fields` is its **shape**, read from the schema/model — the aggregate root's own fields (D2 already folded the child rows into it). Send the **complete** list every time you push that key: `data` is written **wholesale, never merged**, so what you report *is* the entity's shape and everything you leave out is absent from the map (`fields: []` says it has none). That same wholesale write is what lets a later re-scan relearn a shape the code changed — name, summary and district stay the team's, but the field list is the code's to own. Re-pushing an entity whose shape this run didn't re-read? Carry its current `fields` forward verbatim (`aporia:search_graph` → `aporia:pull_context`). **Never reach for the other escape hatch — omitting the key — inside a `completeScope` batch: the sweep tombstones every scanned node the batch doesn't name, so "leave it out" there means "delete it".**
A node's `name` and `summary` are map typography — a 1–4-word domain noun and a one-sentence summary; every authored field's shape is in [content-style](references/shared/content-style.md).
Edge vocabulary: `relates_to` (entity↔entity, label carries cardinality) · `depends_on` (component↔component / feature↔feature — **label mandatory**: ≤4 words for what crosses it, conditions included, e.g. "gated: ENABLE_V4") · `owns` (component→entity) · `realized_by` (feature→component) · `touches` (feature→entity). The response reports `skippedEdges` (an endpoint key didn't resolve — push the scope that *defines* a node before one that only references it, or include both in the same batch) and, for `completeScope`, the `removed`/`removedEdges` **counts** (tombstoned because the scan no longer reports them). **A response carrying `mode: "preview"` wrote NOTHING**: the product declares a canonical ref and your `observed` is off it (a branch) or dirty — as-built truth is only written from the canonical ref on a clean tree. Never proceed as if that scan landed; onboard from the canonical ref, or hand the human the preview delta. A large run can also hit `RATE_LIMITED` — wait the returned `retryAfter`, then retry the same call.

### Phase 7 — Record the why with `aporia:record_notes`

Capture the elicited reasoning **and** every gap the probe found, each note targeting the node(s) it's about by `key` — each a ≤60-char headline `title` over a markdown `body`, sized per [content-style](references/shared/content-style.md):

- a **`task`** (or a **`bug`**, when the gap contradicts a decision the team already made) on every feature the Phase 4 probe found **half-baked / mocked** — naming the missing side / the mock / the `TODO`. This is the visible "what's left to build"; unlike a flag it carries a title, an owner and closure evidence. A later scan that finds the missing side landed attests it, and a human confirms. (A feature whose binding is still `planned` needs no task — the unbuilt binding already says so.)
- a **decision** for a stated *why* (with its rationale), or a **rule** when that *why* is standing law the code must keep obeying; a **question** for an unstated one; a **tension** when a correction contradicts the as-built. A Rule is never claimable as work — if the law also implies a build, that build is a separate `task` on the node the work touches, carrying the Rule as a **secondary** note target (the node target stays `primary`, or the task lands on no page of the map).

```jsonc
// aporia:record_notes input
{ "notes": [
  { "kind": "rule",
    "title": "Invoices are immutable once issued",
    "body": "Once issued, an invoice never changes. Corrections are made by issuing a separate **credit note**, never by editing the original.",
    "rationale": "Audit and tax records require an immutable issued document.",
    "targets": [{ "refType": "node", "ref": "entity:billing.invoice", "role": "primary" }] },
  { "kind": "task",
    "title": "Checkout tax is hard-coded to 0",
    "body": "`checkout.ts` returns a hard-coded `0` tax. No real engine is wired. Checkout ships, but the tax it charges is fiction until real logic lands.",
    "targets": [{ "refType": "node", "ref": "feature:billing.checkout", "role": "primary" }] },
  { "kind": "question",
    "title": "Which tax engine for Checkout?",
    "body": "Checkout needs tax computation, but the engine isn't chosen: *build* vs. **Stripe Tax** vs. **Avalara**. Gates the EU rollout.",
    "targets": [{ "refType": "node", "ref": "feature:billing.checkout", "role": "primary" }] }
] }
```

Notes land `provisional`/`open` for the team to curate. Don't invent rationale — if the *why* wasn't stated, it's a **question**, not a decision. (`blocksImplementation` is deprecated — the server accepts and ignores it. File the task instead.)

## Acceptance checklist (before declaring done)

- [ ] The Constitution grounded the scan FIRST — the thesis, the personas features bind to, and 3–5 testable principles exist and were authored WITH the human; an empty Constitution was filled via `aporia:upsert_thesis` / aporia-persona-mapper / `aporia:upsert_principles`, never skipped or invented from the code.
- [ ] Passes the Recognition Test — recognizable to a founder, navigable by an engineer.
- [ ] No node fails the plumbing filter (D1); no node type exceeds ~30 per scope without reason (D2).
- [ ] Every node uses domain language (D3) and cites `externalRefs` (D4).
- [ ] Every entity carries its COMPLETE current field list from the schema/model — `data` is written wholesale, so a partial list is all the shape the map will have (and a re-push of the key replaces it).
- [ ] Every component's kind follows the D6 table; entries and externals identified per scope; every `depends_on` edge verb-labeled; `sub`/`domain` filled where evidence supports them (D6).
- [ ] No `feature.intent` was invented — it's a human answer or left empty/open (D5).
- [ ] The Realization Probe drove every feature to structure, not a grade: fully-wired features report their own `externalRefs` (kind `code`) plus all bindings `as_built`; every half-baked / mocked feature carries an **open task** citing its missing side; nothing-on-either-side pushed `planned` at `hypothesis` (D4/D5).
- [ ] Every `aporia:apply_scan` returned `skippedEdges: 0` (or you understand each skip) — and every response **applied** (no `mode: "preview"`: a canonical-ref/dirty preview writes nothing).
- [ ] Corrections were recorded as decisions/tensions, not silently applied.
- [ ] Every authored string sized for its surface — node names 1–4 words, summaries one sentence, note titles ≤60-char headlines over markdown bodies ([content-style](references/shared/content-style.md)).

## Anti-patterns (reject)

The raw import graph as edges · one node per file/function/table · entities named after DB tables · **an entity pushed with a partial or empty `fields` list** (`data` is written wholesale, so the omitted fields simply don't exist on the map) · invented features with confident rationale · **a mocked feature pushed `as_built` with no task naming its missing side** · **a document cited as `kind: "code"`** (it would fabricate Built) · **every feature pushed `planned` no matter how completely the code builds it** (burying built features under vapor) · "User Management", "Core", "Utils", "Settings" as features · a 200-node map "for completeness" · 40 questions instead of a draft to react to · **scanning onto an empty Constitution** (every feature left ungrounded) · **inventing the thesis or a persona from the code** instead of eliciting it from the human · an unlabeled `depends_on` between services · a model/LLM as a graph node (it rides `sub`) · an API route classified `service` instead of `trigger` (D6).
