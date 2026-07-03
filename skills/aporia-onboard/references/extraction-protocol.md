# The Extraction Protocol

**The contract this skill must meet to produce a world-view, not a slop dump.** Inlined so the skill is portable to any repo. The data model is trivial; the map is only as good as what you *choose to put in it*. One promise: when the team first opens their bootstrapped map, they recognize their product.

---

## Contents

1. The Bar — the Recognition Test
2. Observation vs. Interpretation
3. The Five Disciplines (D1 plumbing · D2 grain · D3 language · D4 evidence · D5 intent)
4. The Staged Pipeline
5. Disciplined Elicitation
6. Division of Labor
7. Acceptance Checklist
8. Anti-Patterns

## 1. The Bar — the Recognition Test

A bootstrap passes if and only if:

1. **A founder, shown the map, says "yes, that's my product" within 30 seconds.** (Recognition.)
2. **A new engineer can use it to find where to make a change.** (Usefulness.)

300 plumbing nodes fail recognition. Five vague boxes fail usefulness. The grain that passes **both** is the target. Every rule below serves this test. The two failure modes:

- **Scan slop** — every DTO/config/table a node; the raw import graph as a hairball; `tbl_inv_ln` instead of "Invoice Line." A UML dump nobody recognizes.
- **Skill slop** — generic intent true of every SaaS ("User Management", "it helps users be productive"). Meaning useful to no one.

---

## 2. Observation vs. Interpretation

Every extraction is one of two things; conflating them is the root of slop:

- **Observation** — *this table exists; this module imports that one; this route is registered.* Fact → an `as_built` node/edge. Carries no confidence.
- **Interpretation** — *this is the "Invoice" aggregate; these files are the "Billing" service; this is "Checkout".* Judgment → carries **confidence** and cites **evidence**.

Your entire value is the interpretation — clustering, naming, dropping, inferring. Serializing the AST produces slop by construction. **Judge.**

> **The hard line: report structure with evidence; never fabricate intent — elicit it.** The *why*, the success criterion, the persona served are not in the code. Where they aren't observable, you **ask** (Phase 5); you do not invent.

---

## 3. The Five Disciplines

**D1 — The Plumbing Filter** *(the single most important anti-slop rule).* A node earns its place only if **a non-engineer would point at it and say "yes, that's part of my product."** Exclude framework scaffolding, DTOs/serializers, config, utils/helpers, generic CRUD wrappers, adapters, test fixtures, build tooling. Include domain concepts. Litmus: `Invoice`, `Subscription`, `Lead` are in; `RetryConfig`, `PaginationParams`, `Logger` are out — though the parser sees them identically. This is what separates a *product map* from an *architecture diagram*.

**D2 — Grain Rules** *(recluster before emitting).* Entity = an aggregate root (fold value objects / child rows: an `Invoice` has `LineItem`s — one node). Component = a meaningful module/service (fold files: a `billing/` package is one component, not nineteen). Feature = a slice of user value (fold operations: "Checkout", not `createOrder` + `chargeCard` + `sendReceipt`). **If a scope yields > ~30 nodes of one type, you are at the wrong grain — coarsen before emitting.** Cap and group; never dump.

**D3 — Domain Language.** Names are what the team *says*, not what the code declares. Sourced in priority: **UI strings → route/page names → public API names → comments & docstrings → README/docs → identifiers** (last resort). `tbl_inv_ln` → "Invoice Line." DDD without knowing DDD, made operational.

**D4 — Evidence + Confidence** *(the anti-hallucination clamp).* Every interpreted node/edge cites `externalRefs` (the files/symbols it derives from — no evidence, no node) and a `confidence`. `confidence` is a **realization grade read from the code** — how completely the repo *builds* the thing — **not** certainty about its *intent*; the two are orthogonal (a feature can be `confirmed`-built with its `intent` still empty). Grade by build-state evidence (the Feature Realization Ladder, D5 below / skill Phase 4): end-to-end on the default path ⇒ `confirmed`; gated or partial ⇒ `provisional`; one-sided, or satisfied by mock/stub/hard-coded data ⇒ `hypothesis`. When a side can't be established, grade **down**. A pure scanned entity/component fact may carry `provisional`/none and still render as confirmed reality. The clamp: never assert a grade you can't cite, and never let the realization grade smuggle in an unstated *why*. The `hypothesis` and `provisional` features are exactly what Phase 5 surfaces to the human — half-baked and conditional, never silently passed off as shipped.

**D5 — Intent is Elicited, never Fabricated.** Propose `feature` nodes and their bindings from routes/UI/folders, each carrying a **realization grade read from code** — *The Feature Realization Ladder*: probe the slice for **surface** (a real route/page reached), **logic** (a backend handler/query/mutation it calls), **persistence/IO** (real storage or external system), **data realness** (real values, not `MOCK_*`/fixtures/hard-coded/`TODO`), and **gating** (flag / env / role-or-beta guard / killswitch), then grade — built end-to-end on the default path ⇒ `confirmed`; conditional or one partial seam ⇒ `provisional`; one-sided or mock/stub/hard-coded ⇒ `hypothesis`. What you must **not** fabricate is the **intent** — the *why*, the success criteria, the persona served. Leave `intent: ""` and the arrays empty; that is Phase 5's job. A `confirmed` *build* with an honest empty intent is correct; a feature with a fabricated rationale is worse than one with none.

---

## 4. The Staged Pipeline

Each stage gates the next:

1. **Inventory** *(observation, high recall, no judgment)* — schema entities, modules, exports, routes, dependency edges.
2. **Distill** *(interpretation — D1 filter, D2 grain, D3 language, D4 evidence)* — emit `entity`/`component` nodes + structural edges as `as_built`.
3. **Propose** *(features + realization grade — D5)* — candidate `feature` nodes + `realized_by`/`touches` bindings, each graded by how completely the code realizes it (the Ladder); `intent` left empty for Phase 5.
4. **Elicit** *(§5)* — hand the draft to the human.
5. **Commit** — push per scope via `apply_scan` (scopeKey + completeScope), reconciled by stable `key`. Re-scans run the same pipeline; identity is the `key`, so refactors and file moves never orphan bound intent.

**Planned features** — a `feature` the product *means* to have but the code does not implement yet (UI scaffolding with no backing logic, an empty route, a "coming soon" surface, a feature the human says to register "to be defined") is **intent, not structure**. This is the floor of the Ladder: no implementation on *either* side. Push it through `apply_scan` with `planned: true` at `hypothesis`: it lands `intended` (not `as_built`) and is exempt from the completeScope sweep, so it survives until a real scan finds it in code and realizes it. (A feature wired on one side only is still `as_built` `hypothesis` — the code *is* there, just half-baked — not `planned`.) Never report an unbuilt surface as `as_built` — that asserts the product ships something it doesn't.

---

## 5. Disciplined Elicitation

Intent quality is won or lost here:

- **Draft-first, never blank-page.** Show the proposed map; ask the human to *react*. "Is this right?" beats "what is your product?" — correcting a wrong draft is faster and more accurate than authoring from zero.
- **Highest-leverage questions only.** Budget attention. Ask about: features the scan couldn't infer, the *why* behind the most central nodes, the success criteria, the persona each feature serves. Do **not** interrogate every node.
- **Socratic, grounded in the invariants.** Pressure-test proposals against the thesis and personas ("which persona does this serve? what problem?"). Elicitation *is* the productive friction — applied at bootstrap.
- **Capture the disagreement, not just the answer.** When the human says "no, it's actually X," record the correction as a **decision** note with its rationale — and a **tension** when it contradicts the as-built. That correction is the meaning that normally evaporates into the code.

You may read the repo's own README/PRDs/`docs/` to enrich proposals before asking — but the human still confirms. Proposed, never asserted.

---

## 6. Division of Labor (the line that must not blur)

| | The scan (you, from code) | The elicitation (you, with the human) |
|---|---|---|
| Produces | `as_built` **structure** | `intended` **intent** |
| Method | observe, then interpret with evidence | draft, then elicit and confirm |
| Confidence | `confirmed` for **realization** the code proves end-to-end | the human owns **intent** (why / criteria / persona) |
| Never does | fabricate intent | invent structure the code doesn't have |

Structure flows from the code; intent flows from the team. The map's honesty is that the two are kept distinct and bound by `key`, not smeared together.

---

## 7. Acceptance Checklist (before declaring done)

- [ ] Passes the Recognition Test (§1) — founder recognizes it; an engineer can navigate it.
- [ ] No node fails the Plumbing Filter (D1).
- [ ] No node type exceeds ~30 per scope without justification (D2).
- [ ] Every node uses domain language, not raw identifiers (D3).
- [ ] Every node cites `externalRefs`; nothing ungrounded (D4).
- [ ] Every feature carries a realization grade earned from code (built ⇒ confirmed · gated ⇒ provisional · half-baked ⇒ hypothesis); none blanket-stamped (D4).
- [ ] No intent was fabricated; every *why* traces to a human answer or is an open question (D5).
- [ ] Each realization grade cites its deciding evidence (the gate, the missing side, the mock); `hypothesis`/`provisional` features surfaced in Phase 5, none asserted blind.
- [ ] Corrections were captured as decisions/tensions, not silently applied.

## 8. Anti-Patterns (reject)

The raw import graph as edges · one node per file/function/table · entities named after DB tables or code symbols · a `feature` invented with a confident rationale nobody stated · **every `feature` stamped `hypothesis` no matter how completely the code builds it** (the grade is read from code, not defaulted) · "User Management", "Settings", "Core", "Utils" as features · a 200-node first map "for completeness" · 40 questions instead of a draft to react to.

> If a rule and the Recognition Test ever disagree, the Recognition Test wins — the map exists to be recognized, or it is worthless.
