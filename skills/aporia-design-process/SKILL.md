---
name: aporia-design-process
description: >-
  Designs a feature's INTENDED functional process — the swimlane of lanes,
  steps, and flows the team means to build — into Aporia's living map. Elicits
  the human's mental model, drafts the shape for them to react to (guess and
  mark, never interrogate), validates it, then pushes with aporia:record_process
  for polishing in the canvas editor. The DESIGN door — distinct from
  aporia-session-notes, which records behavior OBSERVED from code. Use when
  asked to design, draw, or map the process for a feature, map out how a feature
  should work, build the flow / swimlane / sequence for X, or model a feature's
  behavior.
---

# Aporia design process

You are turning a human's idea of **how a feature should work** into a process the team can iterate on — a swimlane of lanes (who acts), steps (what happens, in order), and flows (the messages between them), bound to a feature in the map. The data model is simple; the value is fidelity to *what the human actually means*, captured fast enough that they stay in flow.

This is the **DESIGN door**. Its twin, `aporia-session-notes`, records a process **observed/traced from code** ("don't draw a flow the code doesn't take"). Here the discipline is inverted: **the human's mental model is the source of truth**, and the process is allowed to be unfinished.

## The hard line (flipped from session-notes)

**Never assert a step, lane, or branch the human didn't confirm.** Where their model is silent, you draft a *hypothesis* and **mark it** — you do not fill the gap with invented certainty, and you do not put words in their mouth. The acceptance bar is **not completeness**: it's a faithful skeleton the human recognizes, honestly flagged where it's a guess, safe to hand to the editor. The map has a `state: intended` for exactly this — a process the team *intends*, not one the code *proves*.

## The method: you propose, the human disposes

You do the drafting; you spend the human's attention only at genuine forks. At each fork, present a **structured multiple-choice question** (`AskUserQuestion`) with **your best guess as the first option, marked `(Recommended)`** — so validating is one keystroke and correcting is one click. Never make the human type a swimlane into a form; that's what the canvas editor is for.

**Budget the forks — this is the crux.** Ask only the *structural* questions (depth, lanes, the final shape gate). For the unambiguous majority of steps, **guess silently and mark the guess** — do not interrogate the human step by step. Per-step questions ("is this a step or a decision?", "where does the 'no' branch go?") are the **exception**, asked only when a step is load-bearing *and* genuinely ambiguous. A skill that asks 12 questions to pin every step has become the form `AskUserQuestion` exists to avoid, and has stolen the editor's job. Seed; don't finish.

## How this skill reaches Aporia

Through the **Aporia MCP server only** — pinned to one product, org derived from its API key, so you never pass a product id. Call tools **fully-qualified as tools of the `aporia` server** (e.g. `aporia:search_graph`) so they resolve alongside other MCP servers. If they aren't available, stop and tell the user to configure the Aporia MCP server (`APORIA_API_KEY` + `APORIA_PRODUCT_ID`) first.

Tools: `aporia:pull_constitution` (personas to populate lanes), `aporia:search_graph` (resolve the feature + candidate participants), `aporia:pull_context` (zoom the feature; with `includeProcess: true`, read any process it already has so you can refine it), `aporia:apply_scan` (create the feature if it doesn't exist yet), `aporia:record_process` (push the process), `aporia:record_notes` (optional — log what's still unconfirmed), `aporia:attach_file` (optional — drop a rendered artifact on the hand-off note).

## Workflow

Copy this checklist into your response and check off each phase:

```
Design process progress:
- [ ] Phase 0  — Ground: aporia:pull_constitution; aporia:search_graph to resolve the feature + lane participants
- [ ] Phase 0b — Existence: feature must exist (else create it planned); read any existing process (pull_context includeProcess) → refine, don't redraft
- [ ] Phase 1  — Draft: assemble lanes/steps/flows (from the existing process if any); mark guesses
- [ ] Phase 2  — Validate: depth gate · lane set · shape gate (guess-as-default, human disposes)
- [ ] Phase 3  — Import: aporia:record_process (no replace); on CONFLICT, ask before overwriting
- [ ] Phase 4  — Hand off: name the guesses; point the human to the Process editor tab
```

### Phase 0 — Ground

`aporia:pull_constitution` → the product's thesis, principles, and **personas** (each has an `id`). Personas are your candidate `persona` lanes — a lane binds to one via `refKey` (the persona `id`, never its name). Hold the principles: a process that violates a core invariant is a **tension** worth surfacing, not a silent draft. Then `aporia:search_graph { query | keyPrefix | type: "feature" }` to resolve the feature this process is about, and `{ type: "component" }` to find the systems that could be lanes (a `system` lane binds `refKey` to a component `key`).

### Phase 0b — Existence (the feature must exist)

A process binds to a feature node, and `aporia:record_process` **hard-rejects** if no feature with that `key` exists. So resolve it first:

- **Feature exists** → use its `key`. Good.
- **Feature doesn't exist yet** → it's the thing being designed. Confirm with the human, then create it with `aporia:apply_scan` as `planned: true` (lands `intended`, sweep-exempt, `intent: ""` until elicited) **before** recording the process. Don't fabricate its intent — leave it empty or capture the why as a note.

**Does a process already exist?** Read it up front: `aporia:pull_context { key: <featureKey>, includeProcess: true }` returns the feature's `process` (the full swimlane) or `null`. If one exists — **especially if `process.author.kind` is `user`** (a human drew it in the editor) — do **not** redraft from scratch. **Refine it**: seed your draft from its lanes/steps/flows (Phase 1), make your changes, and overwrite with `replace: true` in Phase 3 as a deliberate merge, not a blind clobber. `null` means none — draft fresh. (The Phase 3 `CONFLICT` guard remains as a backstop for the race where someone saves between your read and your write.)

### Phase 1 — Draft (assemble; mark every guess)

If Phase 0b found an existing process, **start from it** — load its lanes/steps/flows and apply your changes, preserving the human's structure. Otherwise, from the conversation (and any code you traced this session), assemble a first-pass swimlane:

- **Lanes** — the participants. `persona` (a human role, `refKey` = persona id) · `system` (a component, `refKey` = component key) · `external` (a third party). Give each a stable string `id`.
- **Steps** — ordered activities, each in a lane. `step` (a plain activity) · `decision` (a branch point) · `trigger` (started by a `timer` / `webhook` / `event`). `order` sequences them; `refKey` optionally binds a step to an Entity it touches.
- **Flows** — the messages between steps (`from` → `to` step ids); a `branch` label names a decision's exit ("yes" / "no").

Labels are map typography — a lane 1–3 words, a step 2–5 words verb-first, a branch a one-word guard ([content-style](references/shared/content-style.md) has the full proportionality table).

**A `decision` is binary — give it both exits.** Record a flow for *each* branch, each with a `branch` label (the guard: `yes`/`no`, `valid`/`invalid`, `found`/`missing`) — never only the taken path, or the diagram reads as a straight line running *through* the failure step. If you can't yet name where a branch leads, that exit is a **guess**: draft a placeholder target in the lane that would own it and mark it (Phase 4). Don't drop the branch, and don't aim a flow at a step that doesn't exist — it silently won't render.

**Place each branch's target in the lane of whoever handles it — lane choice *is* the branch direction.** A user-facing rejection/error → the **persona** lane (the human sees the error); a system rollback/retry → the **system** lane; a third-party failure → the **external** lane. The editor draws a branch into a side lane as a *sideways fork* off the decision, and a terminal error sits on the decision's own line — so putting the error in the Merchant lane *is* sending the "no" branch left. You pick the lane, not a direction.

Where the human's model is silent, draft the most plausible step **and mark it as a guess** (see Phase 4 — the wire schema has no confidence field, so the mark lives in the hand-off, not the data). Partial shapes are fine: dangling flows, a step whose branch target is unknown — all persist. Do not invent a branch or a participant just to look complete.

### Phase 2 — Validate (you propose, the human disposes)

Spend attention at the structural forks only. Best guess first, `(Recommended)`.

1. **Depth gate** — *"How far should this first pass go?"* → **Happy path only** (Recommended — it's a seed; still give each decision *both* exits, the error side flagged as a guess) · **+ main error branches** (wire the negative targets into their handler's lane) · **Full, incl. edge cases**. Sets how much you draft; don't over-build past what they pick.
2. **Lane set** (`multiSelect`) — *"Who acts in this flow?"* Options are **real keys** from Phase 0 — the relevant personas + components — plus **"External system."** Ticked lanes get real `refKey` bindings, not guesses. (multiSelect questions can't carry a preview — that's fine here.)
3. **Shape gate** (the crux, single-select) — assemble the draft, **render it back as an ASCII swimlane in the option preview**, and ask: *"This is the shape I have — import it to Aporia to polish in the editor?"* → **Import as-is** (Recommended) · **Fix lanes** · **Fix a step** · **Keep refining**. The human literally sees and validates the shape — finished or not — before it lands. (Single-select supports previews; put the diagram on the recommended option.)

```
        Shopper             │  Checkout UI       │  Stripe
        ────────────────────┼────────────────────┼────────────────────
        ● Submit payment    │  ● Create intent   │  ● Confirm charge
        ▢ See decline ◀─no──┤  ◆ Succeeded?      │  ⏱ Charge webhook
                            │     └── yes ──▶ Charge webhook
  ●=step ◆=decision ⏱=trigger   no◀ = a branch peels to its handler's lane   (?)=guess
```

**Per-step / per-branch questions are the exception** — fire one only when a step is load-bearing *and* you genuinely can't guess its kind or its branch target. Otherwise guess, mark it, move on.

### Phase 3 — Import with `aporia:record_process`

Push the process. If Phase 0b found **no** existing process, **omit `replace`** (it defaults to refusing) — `created: true` confirms a clean create. If you read an existing process and chose to **refine** it (with the human's go-ahead from the shape gate), push with `replace: true` — a deliberate merge of your changes onto theirs, not a blind clobber:

The wire shape — lanes · steps · flows · caps — is in **[references/shared/record-process-schema.md](references/shared/record-process-schema.md)**, the same schema both process doors push. **The design discipline it carries here: a binary `decision` needs BOTH exits** — record a flow for each branch, each `branch`-labelled, the negative target placed in the lane of whoever handles it (a user-facing rejection → the persona lane); never draw only the taken path.

Caps (in that schema): ≤12 lanes · ≤80 steps · ≤160 flows. One process per feature; it lands `origin: authored` / `state: intended` for the team to curate.

**Backstop — if a *fresh* push errors with `CONFLICT`** ("a process already exists … last edited by `user`/`assistant`"), someone created one between your Phase 0b read and this write. **Do not auto-retry.** Re-read it (`pull_context { includeProcess: true }`) and handle it exactly as Phase 0b: refine-and-merge, or surface the overwrite as a fork — *"A process now exists (last edited by a person). Overwrite it?"* → **Keep theirs, record mine as notes** (Recommended if it was a person) · **Overwrite — I'll lose their edits** · **Cancel**. Only on explicit confirmation re-push with `"replace": true`.

### Phase 4 — Hand off (name what's still a guess)

The process is a seed; the editor is where it's finished. Tell the human plainly:

1. **Where to polish** — open the feature in Aporia → the **Process** tab (the swimlane editor: add / edit / move lanes·steps·flows, drag to connect, save).
2. **What's still rough** — name the steps/branches you guessed (e.g. *"S4's 'no' branch and the refund path are guesses — confirm them in the editor"*). The wire schema has **no `provisional` field for a step**, so this honesty lives in the hand-off message and, optionally, in a note.
3. **Optional** — if unconfirmed parts are significant, drop a `question` note via `aporia:record_notes` targeting the feature (*"steps S4, S7 unconfirmed — finish in the editor"*), so the gap is curatable on the map, not just in chat. If the session also produced a rendered artifact — the swimlane preview, a decision doc — attach it to that note with `aporia:attach_file` (`noteId` from the `notes` refs `record_notes` returns, or from `aporia:pull_context`; `fileType` markdown | html | pdf, `content` inline, ≤10 MB) so the team reads the rendered shape next to the question.

## Acceptance checklist (before declaring done)

- [ ] Every lane, step, and branch traces to something the human said — or is **explicitly marked a guess**, never asserted as fact.
- [ ] Every `decision` has **both exits** recorded (or the missing one flagged a guess), and each branch's target sits in the lane of whoever handles it (user-facing error → persona lane).
- [ ] You asked ~3–5 structural forks, not a per-step interrogation; best guess was offered first as `(Recommended)`.
- [ ] The feature existed (or you created it `planned`) before recording — no orphaned process.
- [ ] You read any existing process first (`includeProcess`) and **refined** it rather than redrafting; `replace: true` was used only as a human-confirmed merge, never a silent clobber.
- [ ] The hand-off points to the Process tab and names exactly what's still rough.

## Anti-patterns (reject)

A swimlane invented past what the human confirmed · 12 questions to pin every step instead of guess-and-mark · asserting a guessed branch as fact · a binary `decision` recorded with only one exit (the diagram then runs straight *through* the failure step) · an error branch dumped in the system lane when the persona is the one who sees it · **redrafting from scratch when a process already exists** (read it with `includeProcess` and refine) · auto-retrying a `CONFLICT` with `replace:true` (destroys a human's editor edits) · recording a process for a feature that doesn't exist · holding the import hostage until the flow is "complete" — the editor is the finishing surface, not this skill.
