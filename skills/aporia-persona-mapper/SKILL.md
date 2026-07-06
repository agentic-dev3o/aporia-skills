---
name: aporia-persona-mapper
description: >-
  Define or refine a Persona in Aporia's living map through a short Socratic
  interview, then write it through the MCP server. A Persona is a Ring 0
  Constitution invariant — a situation portrait (role + pain severity) that every
  feature is judged against. Use whenever someone wants to add or sharpen a persona
  for the pinned product, capture who feels a problem and how badly, turn a fuzzy
  "our user" into a typed record, separate the buyer from the user, or retire a
  persona a product no longer pursues. Reads existing personas with
  aporia:fetch_personas and writes with aporia:upsert_personas. Triggers: add a
  persona, map this persona, define the persona for this product, who feels this
  pain, is this person a buyer or a user, retire that persona. Distinct from
  aporia-session-notes (records a coding session's decisions/questions) and
  aporia-onboard (scans code into the map) — this skill authors the Constitution's
  personas, and only ever WITH the human.
---

# Aporia persona mapper

This skill turns a vague idea of a customer into one **typed Persona** in Aporia's living map. A persona here is **a real person caught in a real struggle**, not a demographic sketch — modern practice moved away from "35-year-old, likes apps" toward behavior, motivation, and the moment a problem bites, because age and income don't explain why someone adopts a product; a struggling moment does. Every field must help a team or an agent make a decision, or it doesn't belong.

In Aporia a persona is **Ring 0 — part of the Constitution** (the *why*: thesis, principles, personas). It is the lens every feature is judged against, and features bind to it by id (`relatedPersonaIds`). That makes it the most human-owned layer in the system, so it is authored **only through a human interview** — you map their answers to the typed record; you never invent one.

You reach Aporia through the **MCP server only**. It is pinned to one product, so you **never pass a product id** — the server injects it. Call the tools fully-qualified as tools of the `aporia` server (`aporia:fetch_personas`, `aporia:upsert_personas`) so they resolve when other MCP servers are connected. If they aren't available, tell the user to configure the Aporia MCP server first.

## The hard line

**Never invent a persona — elicit it.** If the human didn't establish a field, it isn't filled from assumption or from the code. The agent is the scribe in a human interview; the human is the author. No struggling moment, no persona — keep probing. (This mirrors `aporia-onboard`'s rule: report structure with evidence, never fabricate intent.)

## The data model (authoritative)

Map to exactly these fields — the Persona record in Aporia's data model. Enums are **closed**; reject any value outside the set. Field names are camelCase (the wire contract).

```yaml
Persona:                              # productId, organizationId, id → server-owned, never sent
  name
  role: buyer | user | influencer | blocker
  situation                          # the JTBD narrative
  painSeverity: low | moderate | severe | extreme
  status: active | retired           # defaults to active on create
```

There is **no client-supplied id** — Convex assigns it. To refine or retire a persona you pass back the `id` you got from `aporia:fetch_personas`.

## Before you start

1. **Read what exists.** Call `aporia:fetch_personas` first — to avoid duplicates, to spot gaps (three `user` personas and no `blocker` is usually under-mapped), and to find a near-identical one to *refine* instead of cloning. Pass `includeRetired: true` if you suspect this persona was mapped before and retired — **revive it** (a refine that sets `status: active`) rather than recreating a duplicate.
2. **Anchor to the product's why (optional but recommended).** `aporia:pull_constitution` returns the thesis and principles. A strong persona traces to the thesis — the pain you map should be the pain the product exists to solve. If they diverge, that tension is worth surfacing to the human.
3. **Person, not organization.** Even for B2B, only people have jobs-to-be-done; organizations have strategies. Capture the *individual* whose personal stakes drive the decision (their credibility, their risk, looking good to their boss), never "the company."

## How to run it

The interview is stateful — copy this checklist and check off each section as you go, so you neither skip a field nor push a half-mapped persona:

```
Persona mapping:
- [ ] Read what exists first (aporia:fetch_personas)
- [ ] Step 1 — Decision role
- [ ] Step 2 — Situation (the JTBD narrative)
- [ ] Step 3 — Pain severity
- [ ] Step 4 — Name & status
- [ ] Step 5 — Confirm with the human, then aporia:upsert_personas
```

Go **one section at a time**. Use your host's interactive question tool (e.g. `AskUserQuestion`) for the closed-choice fields (`role`, `painSeverity`, `status`) so the user taps instead of typing; use plain conversation for `situation`, which needs probing. After each section, reflect the answer back in one line and confirm before moving on — this is the Socratic part: you help them *discover* the persona, not transcribe a dictation.

Role comes first because *whose* struggle you're capturing sharpens every later question. But stay non-linear — if the situation reveals a different dominant role, go back and fix it.

### Step 1 — Decision role → `role`

Ask what part this person plays **in the decision to adopt this product** — not their job title. A CISO is a title; their *role* might be `buyer` (signs) or `blocker` (vetoes on risk). Separate the two.

Aporia compresses the whole buying-committee literature into four:

| Role | They are the person who… | Absorbs the classic roles |
|---|---|---|
| **buyer** | controls budget and gives final approval; judges ROI | economic buyer, decision-maker, signer, executive sponsor |
| **user** | uses the product day-to-day; judges usability and fit | end user, business user, practitioner |
| **influencer** | shapes the decision without final say; lends credibility | champion/advocate, technical evaluator, consultant, initiator |
| **blocker** | can stall or veto; raises risk, compliance, "we have X" | security/legal/procurement, IT gatekeeper, incumbent defender |

Three judgment calls:
- **One human can wear several hats.** A solo founder is buyer + user + influencer. Split into two personas only when the perspectives are *genuinely distinct and worth tracking apart*; otherwise pick the **dominant** role and note the secondary hat in `situation`.
- **The champion is an `influencer`** here, even though sales frameworks treat it separately. Don't invent a fifth value.
- **Don't skip the blocker.** It's the most under-mapped role and the quietest reason adoption dies. If the product touches security, money, data, or an incumbent tool, there is almost always a blocker — find them.

### Step 2 — The situation → `situation`

The heart of the persona, and the only free-text field. It renders as **markdown** in the persona dossier, so light emphasis on the struggling moment is welcome — but keep it a tight **2–5 sentence narrative in jobs-to-be-done form**, anchored in a concrete triggering moment: not a list of attributes, and not padded into headed sections ([content-style](references/shared/content-style.md)). Probe in order:

1. **Trigger / struggling moment** — when does the problem actually bite? ("The auditor's email landed and the register was three months stale.")
2. **Current behavior** — what are they doing *right now*? The workaround is the real competitor (a spreadsheet, a junior hire, doing nothing). No workaround may mean no real demand — flag it.
3. **Desired progress** — what does "better" look like? Name the outcome, not your feature.
4. **Forces** — at minimum the **push** (what frustrates them today) and the **pull** (what attracts them to change). Note **anxiety** (fear of switching) and **habit** (inertia) when relevant — they explain why a recognized problem still goes unsolved.

Then compose:

> **When** [trigger], **I'm currently** [workaround], **but I want to** [job], **so I can** [outcome] **— what holds me back is** [anxiety/habit].

Keep demographics out unless they are *causally* part of the job ("regulated EU entity" matters; "lives in the suburbs" doesn't).

### Step 3 — Pain severity → `painSeverity`

Severity is how much this struggle *hurts and presses* — the synthesis of **DUR** (Douloureux, Urgent, Reconnu) and the must-have test ("how disappointed would they be without a fix?"). Reflect the situation back and offer these four as tappable options:

| Level | Feels like | Signals | DUR / must-have |
|---|---|---|---|
| **extreme** | Hair on fire | compelling event + deadline; already paying real money/time for a painful workaround; drops other priorities now | D + U + R fully — "couldn't live without a fix" |
| **severe** | Actively hunting | recognized, costly, evaluating options, no hard deadline | D + R, U building — "very disappointed" |
| **moderate** | Annoying but livable | recognized, workaround tolerated, no urgency | D-ish + R, no U — "somewhat disappointed" |
| **low** | Nice to have | latent or unrecognized; not looking | weak on all three — "not disappointed" |

**Decision rule: urgency promotes, tolerance demotes.** A painful problem with no trigger is `moderate`, not `severe`; a trigger with a deadline pushes toward `extreme`.

**PMF flag:** if a product's *primary* personas cluster at `low`/`moderate`, the problem may not be DUR enough to build on. Surface that to the human rather than quietly recording it.

### Step 4 — Name & status → `name`, `status`

**`name`** — a short, memorable **handle anchored in role + situation**, not a fake demographic identity. Something a team would say out loud: *"Deadline-Driven Compliance Lead"*, *"Skeptical Platform Architect"*, *"Solo Founder Under Investor Pressure."* A first name only if it earns its place as a mnemonic.

**`status`** — default `active`. Use `retired` for a persona no longer pursued (segment invalidated, deprioritized). Retire rather than delete so the product's history of who-it-was-for stays intact. Only ask when it's ambiguous or you're explicitly retiring/reviving.

### Step 5 — Synthesize & confirm → `aporia:upsert_personas`

Show the complete persona back to the human, **validate every enum**, and get an explicit confirmation before writing. The write is direct and lands on the shared map — the confirmation is what makes it legitimate.

Then push. **Omit `id` to create; pass an `id` from `aporia:fetch_personas` to refine or retire.** A create needs all four portrait fields; a refine/retire patches only the fields you send.

```jsonc
// aporia:upsert_personas input — create, refine, and retire in one call
{ "personas": [
  // CREATE (no id) — the four portrait fields are required; status defaults active
  { "name": "Deadline-Driven Compliance Lead",
    "role": "buyer",
    "situation": "When a regulator requests the ICT third-party register, she assembles it by hand from scattered contracts and emails, but wants a register that stays current and exports on demand, so she can answer an audit in an afternoon — held back by doubt that a new tool will map to her existing contract data.",
    "painSeverity": "severe" },

  // REFINE (id from fetch_personas) — patch only what changed
  { "id": "k17e0…", "painSeverity": "extreme" },

  // RETIRE (id + status) — never a destructive delete
  { "id": "k9920…", "status": "retired" }
] }
```

The response reports `created`, `updated`, and the resulting `personas` (each with its `id`). Read those ids back to the human, and use them when a feature should cite this persona in its `relatedPersonaIds`.

## Anti-patterns to refuse

- **The fluff persona** — demographics and hobbies, no struggling moment. No trigger and no workaround ⇒ not a persona yet; keep probing.
- **Role = job title** — "CISO" is not a role. Force the buyer/user/influencer/blocker call by what they do *in the decision*.
- **The missing blocker** — money/data/security/incumbent in play and zero `blocker` personas is almost always incomplete.
- **Severity inflation** — everything `extreme`. Apply urgency-promotes/tolerance-demotes; reserve `extreme` for hair-on-fire-with-a-deadline.
- **Persona sprawl** — more than ~5 active personas per product usually means the segments aren't distinct. Merge or retire (the `aporia:upsert_personas` batch is capped at 20 as a backstop, not a target).
- **Cloning a retired one** — if `includeRetired` shows a near-match, revive it (`status: active`) instead of creating a duplicate.
- **Set and forget** — personas rot as markets move. Retire the stale ones rather than leave misleading records active.

## When to create vs. refine

Create a **new** persona for a genuinely distinct decision perspective (a different role, or the same role with a fundamentally different struggle). **Refine** (pass the `id`) when the input is just more detail on someone already mapped, or you're adjusting severity/status. Two personas that differ only cosmetically should be merged: refine one, retire the other.

## Acceptance checklist

- [ ] Grounded in `aporia:fetch_personas` first — not a duplicate (or a needless clone of a retired one).
- [ ] `role` reflects their part in the *decision*, not a job title; the blocker was considered.
- [ ] `situation` names a real trigger and the current workaround — JTBD form, no demographic filler.
- [ ] `painSeverity` follows urgency-promotes/tolerance-demotes; not inflated.
- [ ] Every field was authored by the human and confirmed before the push — nothing invented.
- [ ] Enums valid; `id` sent only for a refine/retire, omitted for a create.

## Reference

For the underlying frameworks — the JTBD switch interview and four forces, the full buying-committee taxonomy, the Sean Ellis severity logic, the DUR mapping, and modern persona doctrine with sources — read `references/frameworks.md` when you need the *why* behind a mapping or want to push back on a classification with evidence.
