---
name: aporia-session-notes
description: >-
  Captures what a coding session produced back into Aporia's living map — the
  decisions made (with rationale), the questions left open, the tensions where
  code diverged from intent — routed by the rule: owes a verdict → new item
  (aporia:record_notes); context on an existing item → comment
  (aporia:comment_item); owes nobody → PR body. Also records a feature's
  OBSERVED behavior — the swimlane its code actually follows — with
  aporia:record_process (to design an INTENDED process WITH the human, use
  aporia-design-process instead). Use when recording a session's decisions in
  Aporia, capturing what was decided / what's open / where the code diverged
  from the plan, logging a question or tension to the map, or syncing session
  notes.
---

# Aporia session notes

A coding session is one of the two ways the product changes (the other is an Aporia Session with the team). The code iterated; along the way you **decided** things, hit **open questions**, and sometimes **deviated** from the intent you were given. That reasoning is exactly what normally evaporates into the diff. This skill writes it back onto the map, attached to the parts it's about, for the team to curate.

You reach Aporia through the **MCP server only**. It's pinned to one product; you never pass a product id. Call the tools fully-qualified as tools of the `aporia` MCP server (e.g. `aporia:search_graph`) so they resolve even when other MCP servers are connected. If the tools aren't available, tell the user to configure the Aporia MCP server first.

## The hard line

**Never fabricate rationale.** If the *why* behind a choice wasn't actually established in the session, it is an **open question**, not a decision. Every note must trace to a real moment in this session — something decided, asked, or observed-as-diverging. No invented confidence.

## The routing rule — item vs comment vs PR body

Not everything a session produces deserves a **new item**. Route by one question — *does this owe a human a verdict, or a unit of work?*

- **A new item** (`aporia:record_notes`) — only when it does: an open question someone must answer, a tension to adjudicate, a decision made, a bug/task to build. An item is a triage row a human must process; over-filing buries the verdicts the team actually owes.
- **A comment** (`aporia:comment_item { ticket, body }`) — context on an **existing** item: progress while working it, evidence you gathered, a premise the code now contradicts, "while working ticket 8 I noticed…". A comment is non-triageable by construction — no ticket number, no status — it lands in the item's thread and exerts zero inbox pressure. Never mint a new item to mirror or annotate one that already exists.
- **The PR body** — ephemeral narration of a diff (what you changed, how you tested). It belongs with the code review, not in Aporia at all.

## The kinds of note

- **decision** — a resolved choice **+ its rationale**. The choice AND why. (`isConstraint: true` when it's a rule the code must keep obeying.)
- **question** — an unknown the work surfaced and left open **that gates further work** — someone owes an answer. Use this whenever the *why* is missing — do not upgrade it to a decision.
- **tension** — a conflict between two things. The signature case for a coding session: the code took a **different path than the intended plan**, or the plan was underspecified and you had to choose. A tension **links two targets** (e.g. the feature/decision you were given and the node where the code diverged).
- **idea** — a proposal with **no verdict owed**: "I noticed we could…". Non-binding and non-gating — it never blocks a spec and is never projected back into agent context. Target the feature it concerns. The line vs question: file a question when work can't proceed without an answer; an idea sitting open for a year is not debt.
- **bug** — code contradicting an **already-decided** intent (drift to fix). Only file it when the governing decision exists — no verdict yet means it's a tension, not a bug. A bug closes on scan evidence only, never by hand.
- **task** — plain work with no open judgment (a chore, an upgrade). Rare from a session; prefer it over misfiling work items as decisions.

## Workflow

Copy and track:

```
Session notes progress:
- [ ] Phase 0 — Resolve targets: aporia:search_graph / aporia:pull_context for the nodes this session touched
- [ ] Phase 1 — Extract: decisions, open questions, deviations from this session's history
- [ ] Phase 2 — Discipline gate: unstated why ⇒ question; tension links two targets; each traces to a moment
- [ ] Phase 3 — aporia:record_notes
```

### Phase 0 — Resolve targets

Find the map nodes this session actually touched. `aporia:search_graph { keyPrefix | group | query | type }` to locate them by path/area/text; `aporia:pull_context { key }` to confirm a node and to read what's **already** decided/asked there (don't duplicate an existing note; if your finding contradicts one, that's a tension). `aporia:pull_context` also returns edge and note **ids** — use those when a note is about an edge (e.g. a dependency that drifted) or supersedes another note.

### Phase 1 — Extract from the session

Walk this session's history and pull out, each tied to a concrete moment:
- **Decisions** — "we chose X because Y." Capture X as `body`, Y as `rationale`.
- **Questions** — "we weren't sure whether…", "TODO: decide…", anything left open.
- **Deviations / tensions** — the implementation diverged from the feature's intent or a recorded decision, or the plan didn't cover the case and you made a call. Name both sides.
- **Ideas** — "we could also…" moments you deliberately did NOT act on. Record them on the feature they concern instead of letting them evaporate — but never as a question (nobody owes an answer) and never as a decision (nothing was decided).

### Phase 2 — Discipline gate

Before writing: is each note real and correctly typed? Unstated *why* ⇒ **question**, not a fabricated decision. A **tension** must link two targets (primary + secondary). Drop anything you can't tie to a moment in the session. Prefer attaching to the most central node a human would inspect.

### Phase 3 — Record

Push with `aporia:record_notes`. A node target is referenced by its stable `key`; an edge/note target by the `id` from `aporia:pull_context`. ≤50 notes per call (≤8 targets each); page if more.

Every string is sized for the surface it renders on — read **[references/shared/content-style.md](references/shared/content-style.md)** (the proportionality table) before composing. The one hard bound: `title` is a ≤60-char plain-text headline, **rejected** (never truncated) past that — the substance goes in the markdown `body`.

When a `decision` answers an open question, tension, idea, or supersedes a decision, pass `resolves` with that source note's `id` (from `aporia:pull_context` or the inbox) — the source closes and links to your decision, same as the UI action zone. Targets are inherited from the source on this path.

```jsonc
// aporia:record_notes input
{ "notes": [
  { "kind": "decision",
    "title": "Charge via Stripe PaymentIntents",
    "body": "Checkout charges through the Stripe **PaymentIntents** API, not the legacy Charges API.\n\nPaymentIntents is the only path that carries SCA/3DS, which EU cards require.",
    "rationale": "3DS/SCA is mandatory for EU customers; the Charges API can't satisfy it.",
    "targets": [{ "refType": "node", "ref": "feature:billing.checkout", "role": "primary" }] },
  { "kind": "decision",
    "title": "Issued invoices stay immutable — credit notes",
    "body": "Partial refunds spawn a separate **credit note**; issued invoices are never reopened.",
    "rationale": "Reopening breaks the audit trail a ledger exists to keep.",
    "resolves": "<question-note-id>",
    "targets": [] },
  { "kind": "tension",
    "title": "Money math: UI vs Billing service",
    "body": "Invoice totals are computed in the **checkout UI**, but the feature's intent says the *Billing service* owns money math.\n\nThe two will drift the moment a second surface needs a total.",
    "targets": [
      { "refType": "node", "ref": "feature:billing.checkout",  "role": "primary" },
      { "refType": "node", "ref": "entity:billing.invoice",    "role": "secondary" }
    ] },
  { "kind": "question",
    "title": "Partial refunds: reopen or credit note?",
    "body": "Should a partial refund **reopen the Invoice**, or spawn a separate *credit note*?\n\nDecides whether an issued invoice stays immutable.",
    "targets": [{ "refType": "node", "ref": "entity:billing.invoice", "role": "primary" }] }
] }
```

Notes land `provisional` / `open`, authored by your agent — flagged for the team to confirm or resolve in an Aporia Session. The response reports `recorded`, `skippedTargets` (a `ref` that didn't resolve — a node key with no match, or an edge/note id outside this product), and `notes` — one `{ shortId, id }` ref per recorded note, in input order (cite the returned `ticket` verbatim in your hand-off — it already carries this product's prefix; pass the `id` to `aporia:attach_file`). If `skippedTargets > 0`, check the keys/ids against `aporia:pull_context` and re-push the affected notes. The server does **not** enforce two targets per tension: if a tension's secondary target lands in `skippedTargets`, the recorded note is one-sided and wrong — re-resolve the missing key/id and re-push before trusting it.

### Attach a rendered artifact (optional)

When the session produced a rendered document — a Markdown spec, a Claude Artifact's HTML, a PDF (≤10 MB) — drop it onto its note or map node with `aporia:attach_file`, so the team opens it **rendered** in the Inbox or Node Workspace instead of reading markup in chat history:

```jsonc
// aporia:attach_file input — pass exactly one target
{ "noteId": "<id>", "filename": "decision-mockup.html", "fileType": "html", "content": "<!doctype html>…" }
// or attach straight to the node you were working on:
{ "nodeKey": "feature:billing.checkout", "filename": "spec.md", "fileType": "markdown", "content": "# Checkout spec\n…" }
```

`noteId` comes from the `notes` refs `aporia:record_notes` just returned (or an existing note's id from `aporia:pull_context`); `nodeKey` is a map node's stable key from `aporia:search_graph` or `aporia:pull_context`. Pass **exactly one** of `noteId` or `nodeKey`. `fileType` is `markdown` | `html` | `pdf`; `content` rides inline — UTF-8 text, base64 for a pdf.

## Recording a feature's process (observed behavior)

When a session **implements or traces how a feature works** — the steps, who does what, the branches and triggers — capture it as a **Functional Process** (a sequence / swimlane bound to the feature) with `aporia:record_process`. This is the BEHAVIOR half of what a session produces, alongside the notes above. It is *observed behavior* — the same discipline as a note: don't draw a flow the code doesn't take.

> **Observed only — not designed.** If you're **designing** how a feature *should* work *with* the human (eliciting their intended flow rather than tracing the code), use the **`aporia-design-process`** skill instead. Same tool, opposite discipline: there the human's mental model is the source of truth and the process may be unfinished. This section is strictly for the flow the code actually takes.

Resolve the feature first (`aporia:search_graph { type: "feature" }` → its `key`; the feature must exist). Then push the process. **Lanes** are the participants (`persona` | `system` | `external`); **steps** are ordered activities in a lane (`step` | `decision` | `trigger`, a trigger carrying `timer` / `webhook` / `event`); **flows** are the messages between steps, a `branch` labelling a decision's exit. Give every lane/step/flow a stable string `id` so steps and flows reference each other. One process per feature — re-recording **refuses** if a process already exists (pass `replace: true` to overwrite, which discards any edits a human made in the canvas editor — confirm first). A lane may bind to a Persona or Component via `refKey`, a step to an Entity it touches via `refKey`.

The wire shape — lanes · steps · flows · caps — is in **[references/shared/record-process-schema.md](references/shared/record-process-schema.md)**, the same schema both process doors push. **The discipline it carries here: record the flow the code ACTUALLY takes** — the observed path, its real branches and triggers; don't draw a step or branch the code doesn't have.

The response reports the process `key` and whether it was `created`. The process lands `authored` / `intended` for the team to curate on the canvas.

## Acceptance checklist

- [ ] Routing honored: context on an existing item went to `aporia:comment_item`, diff narration stayed in the PR body — only verdict-owing findings became new items.

- [ ] Every note traces to a real moment in this session.
- [ ] No invented rationale — missing *why* is a `question`.
- [ ] Each tension links two targets and names both sides of the conflict.
- [ ] Targets resolved (`skippedTargets: 0`), and you didn't duplicate an existing note.
- [ ] Every note has a ≤60-char headline `title` and a markdown `body` — not a sentence crammed into the title, not a wall of text ([content-style](references/shared/content-style.md)).
