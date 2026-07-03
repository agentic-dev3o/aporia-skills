# Evaluations — aporia-design-process

Per Anthropic's skill best practices: evaluations are the source of truth for whether the
skill works. These three scenarios test the disciplines most likely to fail without the
skill. There is no built-in runner — execute each `query` against a fresh Claude instance
with the skill loaded and the **Aporia MCP server connected to a seeded test product**,
then score the transcript against `expected_behavior`.

Unlike file-based skills, this one operates on a live product graph, so each scenario
describes the required graph `setup` instead of bundling input files.

```json
[
  {
    "skills": ["aporia-design-process"],
    "name": "fresh-design-happy-path",
    "setup": "Product has a feature node feature:billing.checkout (no process yet), personas in the constitution, and a component:billing.checkout-ui node.",
    "query": "Let's design the process for checkout — shopper submits payment, we create a PaymentIntent, Stripe confirms the charge.",
    "expected_behavior": [
      "Grounds first: calls aporia:pull_constitution and aporia:search_graph to resolve the feature and candidate lane participants",
      "Reads existing process via aporia:pull_context { includeProcess: true } and finds none, so drafts fresh",
      "Asks ~3-5 structural AskUserQuestion forks (depth, lane set, shape gate), each with a (Recommended) default — NOT a per-step interrogation",
      "Binds persona/system lanes to real refKeys (persona ids, component keys), not invented strings",
      "Renders the draft as an ASCII swimlane in the shape-gate preview before importing",
      "Pushes with aporia:record_process WITHOUT replace and reports created: true",
      "Hands off to the Process editor tab and names any guessed steps"
    ]
  },
  {
    "skills": ["aporia-design-process"],
    "name": "refine-existing-human-authored",
    "setup": "feature:billing.checkout already has a process authored by { kind: 'user' } (drawn in the editor).",
    "query": "Add a refund branch to the checkout process when the charge fails.",
    "expected_behavior": [
      "Reads the existing process via aporia:pull_context { includeProcess: true } before drafting",
      "Recognizes author.kind === 'user' and REFINES the existing lanes/steps/flows rather than redrafting from scratch",
      "Confirms the overwrite with the human at the shape gate before writing",
      "Pushes with replace: true as a deliberate merge — never auto-overwrites silently",
      "Does NOT discard the human's existing structure"
    ]
  },
  {
    "skills": ["aporia-design-process"],
    "name": "feature-does-not-exist-yet",
    "setup": "No feature node for the requested feature exists in the graph.",
    "query": "Map out how the new referral-rewards feature should work end to end.",
    "expected_behavior": [
      "Detects the feature node is missing (search_graph / pull_context returns nothing)",
      "Confirms with the human, then creates it via aporia:apply_scan with planned: true BEFORE recording the process",
      "Does NOT fabricate the feature's intent — leaves it empty or captures the why as a note",
      "Only then records the process, which would otherwise be hard-rejected for a non-existent feature",
      "Marks unconfirmed steps as guesses in the hand-off rather than asserting them as fact"
    ]
  }
]
```

## Scoring rubric

A run **passes** a scenario when every `expected_behavior` is observed. The cross-cutting
failure modes to watch for (the skill exists to prevent these):

- Fabricating steps, branches, or rationale the human never confirmed.
- Interrogating the human step-by-step instead of guess-and-mark + a few structural forks.
- Silently overwriting an existing process (especially a human-authored one).
- Recording a process for a feature that doesn't exist.
- Treating the draft as finished rather than a seed for the editor.
