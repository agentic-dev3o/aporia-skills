<!-- generated from shared/realization-probe.md — edit the source, run bun skills:materialize -->

# The Realization Probe

The read-from-code test that answers **"does the product actually DO this, end to end?"** for a candidate `feature`. It is **orthogonal to intent** — the *why* stays elicited from the human, never scanned — and it does **not** produce a grade you write into the node. Onboarding runs it on every proposed feature; `aporia-sync` re-runs it on every feature a diff touched. Its output is *structure + deficiency flags*; the map derives the Implementation level from that coverage server-side.

## The five signals

Probe the slice, **citing the deciding refs** for each:

- **surface** — a real route / page / entry point a user actually reaches.
- **logic** — a backend handler / query / mutation the surface calls.
- **persistence / IO** — real storage or a real external system, not an in-memory stand-in.
- **data realness** — real values, not `MOCK_*` / fixtures / hard-coded returns / a pervasive `TODO`.
- **gating** — is the path open, or reached only behind a flag / env / role / beta guard / `if (false)` / killswitch?

## The three actions (conclude in ONE — never a stored grade)

- **fully wired on the default path** — surface ↔ logic ↔ persistence, real data, not gated ⇒ report **all** the structure `as_built` and wire its `realized_by` / `touches` edges. *(Intent may still be `""`: you see THAT it works, not WHY.)*
- **half-baked / mocked / gated** — only one side exists (UI with no backend, or backend with no surface), or the path is satisfied by mock / stub / fixture / hard-coded data / pervasive `TODO`, or it's reached only behind a flag / env / role / beta / killswitch ⇒ report the structure that **does** exist `as_built` **and** record an **open `blocksImplementation` note naming the missing side**. That flag — not a field — holds the feature at Partial.
- **nothing on either side** — a coming-soon shell, an empty route, a stated-but-unbuilt surface ⇒ push it `planned: true` at `confidence: 'hypothesis'` (intent, not structure); it lands `intended`, sweep-exempt. Never report an unbuilt surface as `as_built`.

## What coverage derives

**Implementation is derived server-side from binding coverage + open deficiency flags — nothing you write into a field can raise or hold it, only structure and flags can.** The level rolls up from what you can cite:

| The code you probed | Map shows | How you produce it |
|---|---|---|
| no `as_built` binding — only intent | **Not implemented** | the feature is `planned`, or no built structure realizes it yet |
| some bindings `as_built`, some still `intended`, OR an open deficiency flag | **Partial** | report the built structure `as_built` (leave the rest `planned`), and/or flag the mock with `blocksImplementation` |
| every binding `as_built` and no open deficiency | **Implemented** | report all the structure that realizes it `as_built` and wire its edges |

An agent-recorded `blocksImplementation` flag lands **sync-watched** (`closesBy: "sync"`): the later sync scan that proves the missing side landed resolves it via `aporia:resolve_items` with that evidence, and coverage flips the feature back up. (A human's canvas flag stays the human's to clear.)

The probe reads from code; the *why* (intent, success criteria, persona served) is elicited from the human, never fabricated from the probe.
