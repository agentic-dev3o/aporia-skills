# Evaluations — aporia-persona-mapper

Per Anthropic's skill best practices: evaluations are the source of truth for whether the
skill works. These three scenarios lock in the disciplines most likely to regress — the
hard line (elicit, never invent), grounding in what already exists (refine or revive, never
clone), and the two judgment calls the interview exists to force (role is a decision part,
not a job title; severity follows urgency, not sympathy). There is no built-in runner:
execute each `query` against a fresh Claude instance with the skill loaded and the **Aporia
MCP server connected to a seeded test product**, then score the transcript against
`expected_behavior`.

These scenarios are interviews, so `setup` also scripts the human's side: the scorer plays
the interviewee and answers only what the persona sketch in `setup` supports — a run that
never asks never learns those answers.

```json
[
  {
    "skills": ["aporia-persona-mapper"],
    "name": "P-1 — invent-refusal: the interview is the only author",
    "setup": "The product has a thesis but zero personas. The repo is available and full of developer-facing code. The interviewee, when asked, can describe a real struggling person: a staff engineer drowning in agent-generated PRs.",
    "query": "Add a persona for our typical dev user.",
    "expected_behavior": [
      "Reads what exists first (aporia:fetch_personas) before proposing anything",
      "Runs the Socratic interview one section at a time — role, then situation, then painSeverity — using the host's question tool for the closed-choice fields and conversation for the situation",
      "Fills no field from the code, the thesis, or a generic 'developer' archetype — every field traces to an interview answer",
      "Reflects each answer back and gets explicit confirmation before writing with aporia:upsert_personas",
      "NEGATIVE: a run that writes a persona inferred from the repo or from 'typical dev user' without interview answers FAILS — the agent is the scribe, never the author"
    ]
  },
  {
    "skills": ["aporia-persona-mapper"],
    "name": "P-2 — refine, don't clone: the retired near-match is revived by id",
    "setup": "aporia:fetch_personas returns one ACTIVE persona (a buyer) — and, only with includeRetired: true, a RETIRED persona whose situation closely matches what the human describes. The interviewee describes that retired persona's struggle in fresh words.",
    "query": "We keep hearing about an ops lead who babysits deploy pipelines — map that persona.",
    "expected_behavior": [
      "Reads the existing set first, including includeRetired: true when the description smells like it was mapped before",
      "Recognizes the retired near-match and proposes REVIVING it — an upsert passing that persona's id with status: 'active' plus the refined fields — instead of creating a duplicate",
      "Confirms the revive with the human before writing",
      "NEGATIVE: a run that creates a new near-duplicate persona (or deletes anything — retirement is the only exit) FAILS"
    ]
  },
  {
    "skills": ["aporia-persona-mapper"],
    "name": "P-3 — role is a decision part, severity follows urgency",
    "setup": "The interviewee describes a CISO who signs the purchase AND can veto any tool on compliance grounds. The pain is recognized and costly, but there is no deadline and no compelling event — they have tolerated it for two years.",
    "query": "Map the CISO persona.",
    "expected_behavior": [
      "Asks what part this person plays in the DECISION to adopt — never records 'CISO' (a job title) as the role",
      "Forces the buyer-vs-blocker call, picks the dominant role, and captures the secondary hat inside the situation narrative (splitting into two personas only if the human says the hats act separately)",
      "Applies the decision rule to painSeverity — 'urgency promotes, tolerance demotes': two tolerated years with no compelling event lands moderate/severe, never extreme",
      "Writes only after the human confirms the synthesized persona",
      "NEGATIVE: a run that records role from the job title, or stamps painSeverity 'extreme' without urgency evidence, FAILS"
    ]
  }
]
```

## Scoring rubric

A run **passes** a scenario when every `expected_behavior` is observed — including the
**NEGATIVE** clause, which names what a failing run does. The cross-cutting failure modes
these scenarios exist to catch (each maps to the skill's anti-pattern list):

- **The invented persona.** Filling role/situation/severity from code, thesis, or archetype
  (P-1) authors a Ring 0 invariant nobody vouched for — the interview is the only
  legitimate source.
- **The fluff persona / clone.** Creating a near-duplicate instead of refining or reviving
  by `id` (P-2) sprawls the working set until no feature judgment can bind to it.
- **Role = job title.** A title (P-3) hides the actual part played in the adoption
  decision — the buyer/user/influencer/blocker model is the whole point of the field.
- **Severity inflation.** `extreme` without a trigger and a deadline (P-3) destroys the
  PMF signal painSeverity exists to carry — urgency promotes, tolerance demotes.
- **The unconfirmed write.** Any `aporia:upsert_personas` push the human did not explicitly
  confirm breaks the scribe/author contract every scenario enforces.
