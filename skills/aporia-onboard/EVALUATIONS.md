# Evaluations — aporia-onboard

Per Anthropic's skill best practices: evaluations are the source of truth for whether the
skill works. These two scenarios lock in the disciplines most likely to regress — the
migration from a confidence-graded realization to the shipped **evidence-derived,
binary Built** model, and the Constitution-first ground rule. There is no built-in
runner: execute each `query` against a fresh Claude instance with the skill loaded and the
**Aporia MCP server connected to a seeded test product**, then score the transcript against
`expected_behavior`.

Like the other live-graph skills, onboarding operates on a product graph, so each scenario
describes the required graph + repo `setup` instead of bundling input files.

```json
[
  {
    "skills": ["aporia-onboard"],
    "name": "O-1 — a mock must not pass as shipped",
    "setup": "Seeded product with a grounded Constitution (thesis, personas, and principles all present). The repo slice under scan has TWO features. (a) Checkout: a real page src/billing/checkout-page.tsx calls submitCheckout in src/billing/checkout.ts, which persists through the Invoice entity — real data, reached on the default path, not gated (end-to-end). (b) Reports Export: a page src/reports/export-page.tsx whose only handler src/reports/export.ts returns a hard-coded MOCK_REPORT_ROWS fixture — the files exist, but the logic is a mock with no real persistence.",
    "query": "Onboard this billing/reports slice into Aporia.",
    "expected_behavior": [
      "Grounds first: aporia:pull_constitution, then aporia:search_graph per scope before proposing anything",
      "Runs the Realization Probe per feature (surface / logic / persistence-IO / data-realness / gating) and concludes in an ACTION, not a stored grade",
      "Checkout is fully wired on the default path: reports the page, submitCheckout, and the Invoice binding as_built, wires realized_by / touches, and gives the feature its OWN externalRefs of kind code — which is what makes it read Built",
      "Reports Export is half-baked: reports the page/handler structure that exists as_built AND files an open task on the feature naming the missing side — the MOCK_REPORT_ROWS fixture standing in for real report logic",
      "NEGATIVE: reporting the mocked feature's structure with NO task naming the missing side FAILS — the map would show a shipped feature and nothing would say the rows are fiction",
      "Does NOT stamp a certainty grade onto either scanned feature (a node has no such field), and does NOT blanket-push the built Checkout as planned",
      "Leaves feature intent empty for Phase 5 rather than fabricating a why"
    ]
  },
  {
    "skills": ["aporia-onboard"],
    "name": "O-2 — empty Constitution stops the scan",
    "setup": "Seeded product whose Constitution is EMPTY — thesis is null, no personas, no principles — with a real repo available to scan.",
    "query": "Onboard this repo into Aporia — scan the code into the map.",
    "expected_behavior": [
      "Calls aporia:pull_constitution, detects the empty Constitution, and STOPS before Phase 1 — it does not scan onto an ungrounded Ring 0",
      "Elicits the thesis WITH the human (a short interview reflected back, then aporia:upsert_thesis) rather than authoring it alone",
      "Runs the persona interview via the aporia-persona-mapper skill and authors principles WITH the human (aporia:upsert_principles) — never invents personas or principles from the code",
      "Never fabricates the thesis, personas, or principles from the codebase — an unarticulable thesis is surfaced as the finding, not filled in",
      "Only proceeds to inventory/distill once a real Constitution exists for features to bind to"
    ]
  }
]
```

## Scoring rubric

A run **passes** a scenario when every `expected_behavior` is observed. The cross-cutting
failure modes these scenarios exist to catch (the exact drift this skill was migrated to
kill):

- A built-but-hollow feature pushed `as_built` with **no open task** naming the missing
  side, so nothing on the map says the logic is a fixture. This is the O-1 regression.
- Writing a realization **grade** into a field to raise a feature's level. **Built** is
  derived server-side and binary — `as_built` AND at least one `externalRef` of kind
  `code`. A node carries no certainty field to grade it with.
- Citing a **document** (a blueprint, a PDF) as `kind: "code"`, which would fabricate
  Built out of a file nobody can run.
- Blanket-pushing built features as `planned` (burying real structure under vapor), or the
  inverse — asserting an unbuilt surface as `as_built`.
- Scanning onto an **empty Constitution**, or fabricating the thesis / personas /
  principles from the code instead of eliciting them WITH the human (the O-2 regression).
- Fabricating feature intent the human never stated instead of leaving it empty for
  elicitation.
