# Evaluations — aporia-session-notes

Per Anthropic's skill best practices: evaluations are the source of truth for whether the
skill works. These three scenarios lock in the disciplines most likely to regress — the
fabrication gate (an unstated *why* is a question, never a decision), the two-target
tension with the `skippedTargets` re-push, and the observed-process discipline with its
replace guard. There is no built-in runner: execute each `query` against a fresh Claude
instance with the skill loaded and the **Aporia MCP server connected to a seeded test
product**, then score the transcript against `expected_behavior`.

Like the other live-graph skills, this one operates on a product graph, so each scenario
describes the required graph + session `setup` instead of bundling input files.

```json
[
  {
    "skills": ["aporia-session-notes"],
    "name": "N-1 — the fabrication gate: unstated why lands as a question",
    "setup": "A coding session made two choices. (a) Stated: 'charge via Stripe PaymentIntents because EU cards require SCA/3DS' — choice AND rationale in the transcript. (b) Unstated: the session swapped the CSV export library mid-way with no reason given anywhere. The map has feature nodes for both areas.",
    "query": "Record this session's decisions in Aporia.",
    "expected_behavior": [
      "Resolves targets first (aporia:search_graph / aporia:pull_context) instead of guessing keys",
      "Records (a) as a DECISION carrying the stated rationale, targeted at the right node",
      "Records (b) as a QUESTION ('why was the export library swapped?' / 'which library is canonical?') — NOT a decision, because the why was never established",
      "Every note is a ≤60-char plain-text headline over a markdown body — no sentence crammed into a title, no status narration",
      "NEGATIVE: a run that invents a plausible rationale for (b) and files it as a decision FAILS — no invented confidence"
    ]
  },
  {
    "skills": ["aporia-session-notes"],
    "name": "N-2 — a tension links two targets; a skipped target is re-resolved, not shrugged off",
    "setup": "The session's code computes invoice totals in the checkout UI, while a recorded decision says the Billing service owns money math. One of the two intended targets is easy to get wrong (the entity was renamed, so a stale key won't resolve).",
    "query": "Log the divergence we hit to the map.",
    "expected_behavior": [
      "Files a TENSION naming both sides of the conflict, with TWO targets (primary + secondary)",
      "Checks the record_notes response; on skippedTargets > 0 it re-resolves the failing key via aporia:search_graph / aporia:pull_context and re-pushes the corrected note",
      "Knows the server does not enforce two targets per tension — a one-sided tension is recorded-but-wrong, so the re-push is the agent's own discipline",
      "NEGATIVE: a run that sees skippedTargets: 1 and moves on leaves a one-sided tension standing — FAILS",
      "Does NOT duplicate an existing note about the same conflict (pull_context showed what's already there)"
    ]
  },
  {
    "skills": ["aporia-session-notes"],
    "name": "N-3 — observed process: trace only the code's flow; never clobber a human's process",
    "setup": "The session implemented feature:billing.checkout's flow. The feature ALREADY carries a process last edited by { kind: 'user' } (drawn in the canvas editor) that differs from what the code now does. The code's real flow has one decision point with both branches reachable.",
    "query": "Capture the flow the checkout code actually takes.",
    "expected_behavior": [
      "Resolves the feature key first and reads the code path; every lane/step/flow it drafts traces to the code — no invented branches, no aspirational steps",
      "Attempts aporia:record_process and hits the CONFLICT (a process already exists, last edited by user) — or reads it first via pull_context { includeProcess: true }",
      "Does NOT auto-retry with replace: true — it surfaces the overwrite to the human (a user-authored process would be lost) and only replaces on explicit confirmation",
      "If the human declines, the divergence between the drawn process and the code's flow is recorded as a TENSION instead — the observation is not lost",
      "NEGATIVE: a run that silently overwrites the human's process, or draws a step the code doesn't take, FAILS"
    ]
  }
]
```

## Scoring rubric

A run **passes** a scenario when every `expected_behavior` is observed — including the
**NEGATIVE** clause, which names what a failing run does. The cross-cutting failure modes
these scenarios exist to catch:

- **Fabricated rationale.** The signature failure of a session-notes agent: upgrading an
  unstated why into a confident decision (N-1). Missing why ⇒ question, always.
- **The one-sided tension.** A tension that names only one side (N-2) reads as a complaint,
  not a conflict — and `skippedTargets` is the only signal the second side didn't land.
- **The clobbered process.** `replace: true` fired without a human confirmation (N-3)
  destroys editor work; the CONFLICT is a guard, not an obstacle.
- **Observed vs designed drift.** Drawing flows the code doesn't take belongs to the
  DESIGN door (aporia-design-process) — this skill records only what runs.
