<!-- generated from shared/record-process-schema.md — edit the source, run bun skills:materialize -->

# The `record_process` schema

The wire shape for `aporia:record_process` — a **Functional Process** (a swimlane / sequence) bound to one feature. Both process doors push this same shape: the DESIGN door (`aporia-design-process`, the flow the team *intends*) and the OBSERVE door (`aporia-session-notes`, the flow the code *takes*). This reference is the schema only; each skill's own SKILL.md carries the *discipline* for the door you're in.

## Shape

- **`featureKey`** — the feature this process binds to; `aporia:record_process` **hard-rejects** if no feature with that key exists.
- **`name`** — 2–4 words naming the flow.
- **`lanes`** — the participants. `kind`: `persona` (a human role, `refKey` = persona id) · `system` (a component, `refKey` = component key) · `external` (a third party). Each needs a stable string `id`.
- **`steps`** — ordered activities, each in a lane (`laneId`). `kind`: `step` (a plain activity) · `decision` (a branch point) · `trigger` (started by a `timer` / `webhook` / `event`, carried in `trigger.mode`). `order` sequences them; `refKey` optionally binds a step to an Entity it touches.
- **`flows`** — the messages between steps (`from` → `to` step ids); a `branch` label names a decision's exit (`yes` / `no`, `valid` / `invalid`).

Give every lane / step / flow a stable string `id` so steps and flows can reference each other. Labels are map typography — a lane 1–3 words, a step 2–5 words verb-first, a branch a one-word guard (the same proportionality the content-style reference sets out).

Caps: ≤12 lanes · ≤80 steps · ≤160 flows. One process per feature — it lands `origin: authored` / `state: intended` for the team to curate on the canvas. Re-recording **refuses** if a process already exists; pass `replace: true` to overwrite (this discards any human edits in the canvas editor — confirm first).

## Example

```jsonc
// aporia:record_process input
{
  "featureKey": "feature:billing.checkout",
  "name": "Checkout & charge",
  "lanes": [
    { "id": "l1", "kind": "persona",  "label": "Shopper", "refKey": "<persona-id>" },
    { "id": "l2", "kind": "system",   "label": "Checkout UI", "refKey": "component:billing.checkout-ui" },
    { "id": "l3", "kind": "external", "label": "Stripe" }
  ],
  "steps": [
    { "id": "s1", "laneId": "l1", "order": 0, "kind": "step",     "label": "Submit payment" },
    { "id": "s2", "laneId": "l2", "order": 1, "kind": "step",     "label": "Create PaymentIntent" },
    { "id": "s3", "laneId": "l3", "order": 2, "kind": "step",     "label": "Confirm charge" },
    { "id": "s4", "laneId": "l2", "order": 3, "kind": "decision", "label": "Succeeded?" },
    { "id": "s5", "laneId": "l3", "order": 4, "kind": "trigger",  "label": "Charge webhook", "trigger": { "mode": "webhook" } },
    { "id": "s6", "laneId": "l1", "order": 4, "kind": "step",     "label": "See decline message" }
  ],
  "flows": [
    { "id": "f1", "from": "s1", "to": "s2", "label": "submit" },
    { "id": "f2", "from": "s2", "to": "s3", "label": "create intent" },
    { "id": "f3", "from": "s4", "to": "s5", "branch": "yes" },
    { "id": "f4", "from": "s4", "to": "s6", "branch": "no" }
  ]
}
```
