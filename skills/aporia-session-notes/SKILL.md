---
name: aporia-session-notes
description: >-
  Captures what a coding session produced back into Aporia's living map — the
  decisions made (with rationale), the questions left open, the tensions where
  code diverged from intent — routed by the rule: owes a verdict → new item
  (aporia:record_notes); context on an existing item → comment
  (aporia:comment_item); an existing item's own text is now wrong → correct it
  (aporia:update_item); owes nobody → PR body. Also records a feature's
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

## The routing rule — item vs comment vs correction vs PR body

Not everything a session produces deserves a **new item**. Route by two questions, in order — *does this owe a human a verdict, or a unit of work?* and, if so, *is that human in this session right now?*

- **Ask, first** — when a human is working with you and the verdict is theirs to give, **put the question to them instead of filing it**. An item addressed to someone who is already reading your output is the slowest possible way to ask them; they answer in a sentence, and what you record is a **decision** — worth more to the map than an open question ever was. This is the default in an interactive session, not an optional shortcut. File instead when they defer it, decline to settle it now, say it needs someone else, or when the answer must outlive the session as a standing constraint. Running unattended — a scheduled sync, a CI run, a background agent with nobody to ask — there is no one to put it to, so file.
- **A new item** (`aporia:record_notes`) — when it owes a verdict and asking is not available or not enough: an open question someone must answer, a tension to adjudicate, a decision made, a bug/task to build. An item is a triage row a human must process; over-filing buries the verdicts the team actually owes. The tell that you have over-filed: you could have gotten the answer in the time it took to write the body.
- **A comment** (`aporia:comment_item { ticket, body }`) — context on an **existing** item: progress while working it, evidence you gathered, a premise the code now contradicts, "while working ticket 8 I noticed…". A comment is non-triageable by construction — no ticket number, no status — it lands in the item's thread and exerts zero inbox pressure. Never mint a new item to mirror or annotate one that already exists.
- **A correction** (`aporia:update_item { ticket, body }`) — when an **open** item's own text has gone **wrong**, not merely incomplete. The classic case: a decision whose core call still stands, but whose implementation line names a module, path or service the code no longer uses. Fix the text so the next reader gets the current truth, and leave a comment saying *why* it moved. `body` and `rationale` **overwrite** — restate everything still true, and write the correction in the item's own voice rather than bolting a changelog onto the end.
- **The PR body** — ephemeral narration of a diff (what you changed, how you tested). It belongs with the code review, not in Aporia at all.

**Comment or correct?** A comment adds; a correction replaces. If a reader following the item's body would now do the wrong thing, commenting is not enough — the wrong sentence stays there as the thing agents build from. Use both: correct the body, comment the reasoning.

**Never supersede an item just to fix a line inside it.** A superseding decision closes the original, which is false when the rest of it still stands — and it silently removes the ticket an open PR was about to close on scan evidence. Supersede when the *verdict* changed; correct when only the *wording* did.

Two refusals to expect: a **Rule** (`kind: "rule"`) can't be corrected in place — law changes by supersession — and a **closed** item can't either, because its body is the historical record its successor points back at.

### Correcting the map itself — a node's name

The doors above correct an **item**. When the thing that is wrong is a **node's name on the map** — a label that no longer matches what the code is, or one the human just asked you to change — rename it with `aporia:update_node { key, name }`. It is the only rename door: `aporia:apply_scan` reports as-built **structure**, never names, so a scan that reports a different name for a renamed node changes nothing.

- The stable `key` is **identity** and is never touched by a rename, so nothing re-links and no binding breaks. Only the display name moves.
- **Prose only**, the same authority split `aporia:update_item` holds for an item: a rename can't change a node's type, group, state or origin. A node's summary, description and a feature's intent are **not** editable here — that is authored meaning a human owns; propose a change to it with `aporia:record_notes` instead.
- The edit is **stamped**, so the team sees an agent touched the label.

**The discipline.** Rename what is *wrong*, or what the human in the session asked you to fix — never to impose your own preferred vocabulary on a map the team curates. The map's names are how a founder recognizes their own product; a rename nobody asked for is drift wearing a helpful face. When you believe a name is wrong and there is a human present, propose it and let them say yes.

## The kinds of note

- **decision** — a resolved choice **+ its rationale**. The choice AND why.
- **rule** — **standing law**: a line the code must keep obeying, which governs until it is superseded or retired. This is the kind that changes what the item *can be*, so pick it deliberately (see the split test below).
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
- [ ] Phase 2 — Discipline gate: unstated why ⇒ question; tension links two targets; each traces to a moment; ask-first pass over the questions (a human present answers instead of being filed at)
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

**The split test — law vs work.** Run it over every decision before you file it: *does this sentence still bind in two years, or does it finish when the code lands?*

- Still binds ⇒ **rule**. It governs; nobody can claim it, edit it, or scan-close it.
- Finishes ⇒ **decision** (a directive someone builds) or a **task**.
- **Both** ⇒ it is **two items**, not one. This is the case that goes wrong: a single note that says *"every write is notify-only and carries a watermark"* (law) *and* *"build the bus, delete the three old mechanisms"* (a one-off chantier) gets filed as a Rule, and the chantier becomes unclaimable — a later session pulls the ticket, is refused, and has to diagnose it and hand-mint the work itself.

File the pair in **one** `aporia:record_notes` call: the Rule first, then the task, and give the task **two** targets — its own node target as `primary`, plus `{ "refType": "note", "ref": "#0", "role": "secondary" }`, where `#0` is the index of the Rule in this same `notes` array. That note link is the whole association: `aporia:pull_item` on the Rule lists the open items targeting it, so the next agent's refusal names the ticket to pull instead of dead-ending on *"honor it"*.

**The node target stays primary.** A task whose *only* target is the Rule takes the Rule as its primary target, and a note is not a place: the ticket then shows on no node page, in no docket, and compiles no feature — the work the split test exists to make findable ends up findable only by scrolling the Inbox.

**Backward refs only.** A `#n` must name an **earlier** note of the same call that actually recorded (a note whose targets all failed to resolve mints nothing, so nothing points at it). A forward ref, a self-ref, or a ref to a note that didn't land is refused with `BAD_REQUEST` and **the whole batch rolls back** — nothing at all is recorded, not just the bad target. Order the array law first, work second.

Then the ask-first pass, over the questions and tensions only: for each, *could the human in this session settle it right now?* Put those to them in one batch before you record anything — the answers turn into decisions, and what's left is the real filing list. A question filed at someone who is in the room is a routing error, not diligence.

### Phase 3 — Record

Push with `aporia:record_notes`. A node target is referenced by its stable `key`; an edge/note target by the `id` from `aporia:pull_context`. ≤50 notes per call (≤8 targets each); page if more.

Every string is sized for the surface it renders on — read **[references/shared/content-style.md](references/shared/content-style.md)** (the proportionality table) before composing. The one hard bound: `title` is a ≤60-char plain-text headline, **rejected** (never truncated) past that — the substance goes in the markdown `body`.

When a `decision` — or a `rule`, when the verdict is standing law — answers an open question, tension, idea, or supersedes a decision or rule, pass `resolves` with that source note's `id` (from `aporia:pull_context` or the inbox) — the source closes and links to your verdict, same as the UI action zone. Targets are inherited from the source on this path.

**Superseding a Rule takes `kind: "rule"`.** Law is replaced by law, and a `decision` naming a Rule in `resolves` is refused: filing the successor as a directive would turn standing law into build debt — claimable, scan-closable, listed as work. If the law genuinely no longer applies, that demotion is the human's call from the item's action zone, not yours.

**One verdict usually settles several items.** `resolves` also takes an **array** of ids (≤10) — use it when the same decision genuinely answers all of them, and they close together as one act. A decision minted per item is a closure receipt, not a verdict, and it buries the one statement a reader needed. If the items do not share a single answer, they are not one subject: write a decision each.

```jsonc
// aporia:record_notes input
{ "notes": [
  { "kind": "decision",
    "title": "Charge via Stripe PaymentIntents",
    "body": "Checkout charges through the Stripe **PaymentIntents** API, not the legacy Charges API.\n\nPaymentIntents is the only path that carries SCA/3DS, which EU cards require.",
    "rationale": "3DS/SCA is mandatory for EU customers; the Charges API can't satisfy it.",
    "targets": [{ "refType": "node", "ref": "feature:billing.checkout", "role": "primary" }] },
  // the split test says "still binds in two years" — so this one is a rule, not a
  // decision, and a rule resolves the question it settles exactly as a decision does
  { "kind": "rule",
    "title": "Issued invoices stay immutable, credit notes reverse",
    "body": "Partial refunds spawn a separate **credit note**; issued invoices are never reopened.",
    "rationale": "Reopening breaks the audit trail a ledger exists to keep.",
    // one id — or an array when the same verdict settles several:
    //   "resolves": ["<question-note-id>", "<tension-note-id>"]
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

**Law and the work it implies, minted as one act.** The Rule is index `0` of this array, so the task points at it with `"#0"`:

```jsonc
// aporia:record_notes input — the split test came back "both"
{ "notes": [
  { "kind": "rule",
    "title": "Every ledger write is notify-only and watermarked",
    "body": "No surface writes the ledger directly. Writes go through the bus, notify-only, and every row carries a watermark — server side and client side both.",
    "rationale": "Two surfaces already disagreed about a total; the watermark is what makes a replay detectable.",
    "targets": [{ "refType": "node", "ref": "component:billing.ledger", "role": "primary" }] },
  { "kind": "task",
    "title": "Build the write bus, retire the three direct paths",
    "body": "Stand up the bus, move the three existing direct-write paths onto it, then delete them.",
    "targets": [
      { "refType": "node", "ref": "component:billing.ledger", "role": "primary" },
      { "refType": "note", "ref": "#0",                        "role": "secondary" }
    ] }
] }
```

Notes land `provisional` / `open`, authored by your agent — flagged for the team to confirm or resolve in an Aporia Session. The response reports `recorded`, `skippedTargets` (a `ref` that didn't resolve — a node key with no match, or an edge/note id outside this product), and `notes` — one `{ shortId, id }` ref per recorded note, in input order (cite the returned `ticket` verbatim in your hand-off — it already carries this product's prefix; pass the `id` to `aporia:attach_file`). If `skippedTargets > 0`, check the keys/ids against `aporia:pull_context` and re-push the affected notes. The server does **not** enforce two targets per tension: if a tension's secondary target lands in `skippedTargets`, the recorded note is one-sided and wrong — re-resolve the missing key/id and re-push before trusting it.

### Attach a rendered artifact (optional)

When the session produced a rendered artifact — a Markdown spec, a Claude Artifact's HTML, a PDF, or a PNG/JPEG screenshot (≤10 MB, and smaller still for an image: it must fit what one `read_attachment` can hand back, so downscale a big screenshot before sending) — drop it onto its note or map node with `aporia:attach_file`, so the team opens it **rendered** in the Inbox or Node Workspace instead of reading markup in chat history:

```jsonc
// aporia:attach_file input — pass exactly one target
{ "noteId": "<id>", "filename": "decision-mockup.html", "fileType": "html", "content": "<!doctype html>…" }
// or attach straight to the node you were working on:
{ "nodeKey": "feature:billing.checkout", "filename": "spec.md", "fileType": "markdown", "content": "# Checkout spec\n…" }
```

`noteId` comes from the `notes` refs `aporia:record_notes` just returned (or an existing note's id from `aporia:pull_context`); `nodeKey` is a map node's stable key from `aporia:search_graph` or `aporia:pull_context`. Pass **exactly one** of `noteId` or `nodeKey`. `fileType` is `markdown` | `html` | `pdf` | `png` | `jpeg`; `content` rides inline — UTF-8 text for markdown/html, base64 for a pdf or an image.

**The door reads both ways.** What you attach is not write-only: `aporia:pull_item` and `aporia:pull_context` list every artifact as metadata (`attachmentId`, `filename`, `fileType`, `size`, `createdAt`), and `aporia:read_attachment { attachmentId }` hands back the content — UTF-8 text for markdown/html, paged with `offsetBytes` / `length` while `truncated` is true; a PNG or JPEG as a real image block you can see, downscaled and flagged `downscaled` when the original is too big to return whole; a whole PDF as base64. So before you attach a second spec to a note, read the one already there; and when you pick up a ticket whose body references a document, read the document rather than guessing at it.

## Recording a feature's process (observed behavior)

When a session **implements or traces how a feature works** — the steps, who does what, the branches and triggers — capture it as a **Functional Process** (a sequence / swimlane bound to the feature) with `aporia:record_process`. This is the BEHAVIOR half of what a session produces, alongside the notes above. It is *observed behavior* — the same discipline as a note: don't draw a flow the code doesn't take.

> **Observed only — not designed.** If you're **designing** how a feature *should* work *with* the human (eliciting their intended flow rather than tracing the code), use the **`aporia-design-process`** skill instead. Same tool, opposite discipline: there the human's mental model is the source of truth and the process may be unfinished. This section is strictly for the flow the code actually takes.

Resolve the feature first (`aporia:search_graph { type: "feature" }` → its `key`; the feature must exist). Then push the process. **Lanes** are the participants (`persona` | `system` | `external`); **steps** are ordered activities in a lane (`step` | `decision` | `trigger`, a trigger carrying `timer` / `webhook` / `event`); **flows** are the messages between steps, a `branch` labelling a decision's exit. Give every lane/step/flow a stable string `id` so steps and flows reference each other. One process per feature — re-recording **refuses** if a process already exists (pass `replace: true` to overwrite, which discards any edits a human made in the canvas editor — confirm first). A lane may bind to a Persona or Component via `refKey`, a step to an Entity it touches via `refKey`.

The wire shape — lanes · steps · flows · caps — is in **[references/shared/record-process-schema.md](references/shared/record-process-schema.md)**, the same schema both process doors push. **The discipline it carries here: record the flow the code ACTUALLY takes** — the observed path, its real branches and triggers; don't draw a step or branch the code doesn't have.

The response reports the process `key` and whether it was `created`. The process lands `authored` / `intended` for the team to curate on the canvas.

## Acceptance checklist

- [ ] Routing honored: context on an existing item went to `aporia:comment_item`, an item whose own text had gone wrong was corrected with `aporia:update_item` (never superseded for it), diff narration stayed in the PR body — only verdict-owing findings became new items.
- [ ] Any node rename went through `aporia:update_node` (never a scan), was wrong-or-asked-for rather than a preference, and left the stable `key` alone.
- [ ] Ask-first honored: every question a human present could have settled was put to them, not filed at them — what they answered became a decision, and only what they deferred (or nobody was there to answer) was filed.

- [ ] Every note traces to a real moment in this session.
- [ ] No invented rationale — missing *why* is a `question`.
- [ ] Each tension links two targets and names both sides of the conflict.
- [ ] Targets resolved (`skippedTargets: 0`), and you didn't duplicate an existing note.
- [ ] Every note has a ≤60-char headline `title` and a markdown `body` — not a sentence crammed into the title, not a wall of text ([content-style](references/shared/content-style.md)).
