<!-- generated from shared/realization-probe.md — edit the source, run bun skills:materialize -->

# The Realization Probe

The read-from-code test that answers **"does the product actually DO this, end to end?"** for a candidate `feature`. It is **orthogonal to intent** — the *why* stays elicited from the human, never scanned — and it does **not** produce a grade you write into the node. Onboarding runs it on every proposed feature; `aporia-sync` re-runs it on every feature a diff touched. Its output is *structure*; the map derives whether the feature is **Built** from that structure server-side.

## The five signals

Probe the slice, **citing the deciding refs** for each:

- **surface** — a real route / page / entry point a user actually reaches.
- **logic** — a backend handler / query / mutation the surface calls.
- **persistence / IO** — real storage or a real external system, not an in-memory stand-in.
- **data realness** — real values, not `MOCK_*` / fixtures / hard-coded returns / a pervasive `TODO`.
- **gating** — is the path open, or reached only behind a flag / env / role / beta guard / `if (false)` / killswitch?

## The three actions (conclude in ONE — never a stored grade)

- **fully wired on the default path** — surface ↔ logic ↔ persistence, real data, not gated ⇒ report **all** the structure `as_built`, wire its `realized_by` / `touches` edges, and give the feature its own `externalRefs` (kind `code`) — the paths that carry it. *(Intent may still be `""`: you see THAT it works, not WHY.)*
- **half-baked / mocked / gated** — only one side exists (UI with no backend, or backend with no surface), or the path is satisfied by mock / stub / fixture / hard-coded data / pervasive `TODO`, or it's reached only behind a flag / env / role / beta / killswitch ⇒ report the structure that **does** exist `as_built`, and **file a `task` (or a `bug`, if it contradicts a decision) on the feature naming the missing side**. Give it a real title and a body — that item is now the record of what's missing, and it closes on evidence like any other work.
- **nothing on either side** — a coming-soon shell, an empty route, a stated-but-unbuilt surface ⇒ push it `planned: true` (intent, not structure); it lands `intended`, sweep-exempt. Never report an unbuilt surface as `as_built`.

## What the structure derives

**Built is BINARY and scan-owned: a feature is Built when it is `as_built` AND carries at least one `externalRef` of kind `code`.** Both halves matter, and both are yours to report — nothing a human types can raise it, and there is no middle rung.

| The code you probed | Map shows | How you produce it |
|---|---|---|
| no code carries the feature — only intent | **Not built** | push it `planned: true`, or report the structure without claiming the feature itself |
| the feature's own code exists | **Built** | report it `as_built` with `externalRefs` (kind `code`) and wire its edges |

Note what the second row does **not** say: a feature is never Built because the components it points at happen to exist. Evidence about a feature must be the feature's own code. And a document you are citing — a blueprint, a PDF, a Figma frame — goes in `externalRefs` with `kind: "doc"`; it is shown to the team but it never makes anything Built.

**There is no Partial.** A feature that is built but mocked, gated, or half-wired is Built *plus an open task* naming what's missing. That is strictly more useful than a flag: the task carries a title, an owner and closure evidence, and the later sync that proves the missing side landed closes it via `aporia:resolve_items` with that evidence.

The probe reads from code; the *why* (intent, success criteria, persona served) is elicited from the human, never fabricated from the probe.
