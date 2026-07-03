# Content style — proportionality for the living map

Aporia's bet is that **humans interpret complex state visually**. The map reads as a drill-down — the canvas of node chips, then a row in a list, then a dedicated page — and every string an agent writes becomes typography at one of those depths: a label, a headline, a line, or prose. Text written for the wrong depth breaks the map: a sentence crammed into a title truncates mid-word in every row; a wall of prose buries its own claim; headings wrapped around three sentences read as bureaucracy. **Write for the surface the words will land on, and when in doubt go shorter** — map text is read a hundred times for every time it is written.

This is the contract for *how* text is shaped; the skill that sent you here says *which* fields and *when*. Only the note title is machine-enforced (≤60, rejected); every other shape is a target you hold yourself to.

## The proportionality table

| You are writing | It renders as | Shape |
|---|---|---|
| node `name` | a label among hundreds on the canvas | 1–4-word domain noun the team says — "Invoice", "Product Memory" |
| node `summary` | one line on a card / hover | one sentence: what it is |
| feature `intent` | the Intent panel | 1–3 markdown sentences: the why, and for whom |
| `successCriteria[]` | checklist rows | one testable line each |
| note `title` | a bold row in the inbox / feed | **≤60-char plain-text headline** naming the claim |
| note `body` | the note detail | markdown prose — a lead sentence, then structure only if there are parts |
| note `rationale` | the RATIONALE aside | one tight paragraph: the why |
| persona `name` | a chip | 2–5-word handle anchored in role + situation |
| persona `situation` | the dossier portrait | 2–5-sentence JTBD narrative, light emphasis, no headings |
| principle `statement` | a law in a list | one testable sentence, plain |
| principle `rationale` | under the law | 1–2 markdown sentences tying it to the bet |
| thesis `product` | the product header | 1–2 sentences |
| thesis `problem.statement` / `insight` | constitution panels | one short markdown paragraph each |
| process `name` | the swimlane title | 2–4 words naming the flow |
| process lane label | a swimlane header | 1–3 words |
| process step label | a box on the canvas | 2–5 words, verb-first |
| flow `label` / `branch` | an edge tag | 1–3 words / a one-word guard |

## The title — a headline, not a sentence

The `title` names the claim the way a commit subject names a change; the body IS the claim.

- **≤60 characters, enforced by rejection** — an over-length title fails the whole write (never truncated). If it doesn't fit, you're writing a sentence: cut to the noun phrase and move the substance into the body.
- **Plain text** — no markdown, no trailing period, no "Decision:" prefix.
- **Match the kind** — a question titles as a question ("Partial refunds: reopen or credit note?"); a decision as the choice made ("Invoices are immutable once issued"); a tension names the conflict ("Money math: UI vs Billing service").
- **Timeless** — no status narration ("SHIPPED", "WIP"), no PR numbers or branch names. Git holds history; the map holds meaning.

## The body — markdown at reading distance

- **Lead with one sentence that states the whole claim** — never an echo of the title. The UI stacks title over body; an echo makes the human read everything twice.
- **Structure carries parts, not prestige.** Use `##` sections and bullets when the content has real parts (zones, steps, options); a two-sentence body stays two sentences. The wall and the padded outline are both disproportion.
- **Bold the load-bearing terms**; *italicize a quoted stance*. Name code facts precisely (paths, symbols, the mock, the `TODO`) — and drop the narration around them.
- **Timeless voice** — state the decision / question / tension itself, not the session news ("we just", "in this PR", "now"). The note outlives the session that wrote it.

A decision's `rationale` is the why in one tight paragraph. If the why doesn't fit a paragraph, it isn't understood yet — record a `question` instead.

## Anti-patterns (each observed on a real map)

- **The sentence-title** — the whole decision crammed into `title`, truncating mid-word in every row.
- **The echo** — the body's first line repeats the title.
- **The status report** — "SHIPPED (PR #27, branch feat/…): …" — provenance narrated as meaning.
- **The wall** — eight flat lines with three claims buried inside.
- **The padded dossier** — headings wrapped around three sentences to look thorough.

## Before → after

✗ What the map showed:

```
title: "SHIPPED (PR #27, branch feat/prose-editor): the reusable inline
        DescriptionEditor (Tiptap, markdown-string API) in @aporia/web-ui, wired acr…"
body:   the same sentence again, then five more flat lines
```

✓ The same decision, proportioned:

```jsonc
{ "kind": "decision",
  "title": "One inline DescriptionEditor for all rich text",
  "body": "Every rich-text surface shares the inline **DescriptionEditor** (markdown-string API) — the rendered content IS the editing surface, no edit mode.\n\n## Wired across\n- Inbox note detail\n- Node page — feature intent\n- Personas dossier — situation",
  "rationale": "One editor keeps humans and agents on a single markdown body and drops into the existing string-based autosave." }
```

## Checklist

- [ ] Title ≤60, plain, names the claim — no status prefix, PR number, or trailing period.
- [ ] Body leads with the claim (no title echo); sections only where there are real parts.
- [ ] Rationale is one paragraph.
- [ ] Every other field matches its row in the table; when in doubt, shorter.
