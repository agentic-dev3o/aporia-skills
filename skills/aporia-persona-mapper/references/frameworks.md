# Frameworks behind the Aporia persona mapper

The evidence layer for the mappings in `SKILL.md`. Read it when you need to justify a classification, push back on a user's instinct with reasoning, or explain why a field is shaped the way it is. Each section ends with where the thinking comes from.

The throughline across all four frameworks is one rejection: stop describing *who people are* (demographics, titles) and start capturing *what they're trying to get done and how badly* (jobs, forces, severity, decision role). In Aporia that record is Ring 0 — part of the Constitution, the lens every feature is judged against — so the bar is "would this change a build decision?", not "is this a believable character?"

## 1. Why behavior beats demographics

The dominant shift in persona practice over the last decade: lead with behavior, motivation, and context; treat demographics as supporting colour. Age, gender, and location rarely explain *why* someone acts — two 35-year-olds with identical profiles can hire the same product for completely different reasons. Personas built as demographic checklists are too broad to guide a decision, and the most common failure mode is the persona made once that then gathers dust while the team works on gut.

Consequences baked into the skill:
- A persona must be **specific enough to drive a decision**. Test: could a designer, an engineer, or an agent act differently because of it? If not, it's fluff.
- **3–5 personas per product** is the working sweet spot — enough to cover distinct segments without diluting focus. (Aporia's `aporia:upsert_personas` caps a batch at 20 as a backstop, not a target.)
- Personas need a **refresh cadence**; markets move, so `retire` stale ones rather than trust them.

Sources: Miro (*What is a User Persona?*), Mural (*Creating user personas*), Kaizenko (*Personas That Drive Product Decisions* — the Persona Canvas), hellopm, ProductLeadership.

## 2. Jobs-to-be-Done, the struggling moment, and the four forces → `situation`

JTBD (Clayton Christensen, developed in practice by Bob Moesta and Chris Spiek at the Re-Wired Group) reframes buying as **hiring a product to make progress** in a specific situation. When the job changes or a better option appears, the product gets "fired." This is the spine of the `situation` field.

**The struggling moment.** Demand isn't created by a product; it's created by a moment when the status quo becomes unacceptable. People are creatures of habit and won't move until something gives. The single most important thing to capture in `situation` is *that moment* — the trigger — not a general description of the person.

**Three paths out of a struggling moment** (useful for judging severity): they find a fix and hire it; they hire-and-fire several before one sticks; or they want progress but never act — the largest and most invisible "market" of all. A persona stuck in the third path is `low`/`moderate`, not `severe`.

**The four forces of progress** — every switch is a tug-of-war:
- **Push** — frustration with the current situation (drives them to look).
- **Pull** — attraction of the new solution and the progress it promises.
- **Anxiety** — fear and uncertainty about the new, unproven choice.
- **Habit** — the comfortable inertia of what they already do.

A switch happens only when **Push + Pull > Anxiety + Habit**. This is why a recognized, painful problem can still go unsolved for years. Capturing push and pull is mandatory in `situation`; anxiety and habit are worth noting because they explain stalls.

**The statement format**: *"When [situation/trigger], I want to [motivation/job], so I can [outcome]"* — often extended with *"without [pain point]."* The situation captures the trigger, the motivation names the job, the outcome names the progress.

**The timeline** of a real switch, for probing: first thought → passive looking → active looking → decision → first use. Asking *what happened* at each step surfaces the real causes instead of the tidy after-the-fact story.

**B2B nuance.** Only people have jobs-to-be-done; organizations have mission statements and strategies. Even in enterprise sales you must find the job of the *person* hiring you — the one spending their personal credibility to make the purchase happen. That's why an Aporia Persona is always an individual, never "the company."

Sources: jobstobedone.org, therewiredgroup.com (*Complete Guide to JTBD*), Intercom blog (Moesta interviews), Business of Software talks (Moesta/Spiek), uxcrush, gonogo.team, koji.so. Christensen's *Competing Against Luck* and Moesta's *Demand-Side Sales* are the primary books.

## 3. The buying committee / Decision-Making Unit → `role`

In modern B2B, purchases are made by a group, not a person — averages land around 6–7 stakeholders, rising to 10+ (Forrester cites 13 at the enterprise high end). The literature names many roles; Aporia compresses them into four. The full taxonomy and how it folds in:

- **Economic buyer / decision-maker / financial approver / signer / executive sponsor** → **`buyer`**. Controls budget, holds final approval, judges ROI. CFO sign-off is now near-universal on meaningful purchases.
- **End user / business user / practitioner** → **`user`**. Lives with the product day-to-day; cares about usability and workflow fit.
- **Champion/advocate, technical evaluator, consultant, respected internal expert, initiator/sponsor** → **`influencer`**. Shapes the decision without final authority; credibility is the currency. The **champion** (internal advocate, often also an end user) lives here — there is deliberately no separate value for it.
- **Security/risk reviewer, legal, procurement, IT gatekeeper, incumbent-tool defender** → **`blocker`**. Can stall or veto. The most underestimated members of any committee. A lone blocker rarely kills a deal when the economic buyer is firmly aligned, but *unaddressed* blocker concerns are a leading cause of deals dying without explanation.

Two rules the skill enforces:
- **Title ≠ role.** The same job title can be a buyer in one deal and a blocker in another. Classify by behaviour in the decision.
- **One human, many hats.** In smaller orgs roles compress into fewer people (owner-manager = initiator + buyer + user). Split into separate Personas only when the perspectives are genuinely distinct and worth tracking apart; otherwise record the dominant role and note the rest in `situation`.

Sources: Traction Complete, Influ2 Academy (buying-committee survey), IntentAmplify (DMU), Accord, Demandbase, Flowla (Miller Heiman framing). Webster & Wind originated the "buying center" concept (1972); Miller Heiman's *Strategic Selling* is the classic role model.

## 4. Severity, must-have vs. nice-to-have, and DUR → `painSeverity`

The cleanest behavioral measure of how much a problem matters is the **Sean Ellis test**: "How would you feel if you could no longer use this?" — and the share answering **"very disappointed."** Forty percent or more is the rough product-market-fit threshold. The three answers map to relationships, which is what `painSeverity` captures:
- *Very disappointed* = emotional dependency, a **must-have**.
- *Somewhat disappointed* = interested but uncommitted; could switch with little friction.
- *Not disappointed* = no real attachment; their feedback often misleads.

Rahul Vohra used this at Superhuman, moving from 22% to 58% by segmenting on the "very disappointed" group and doubling down. The deliberate focus on a *negative* emotion is the point: asking whether people "like" a product invites polite inflation; asking about loss reveals necessity.

**DUR** (Douloureux / Urgent / Reconnu) is the francophone framing of the same instinct, split into three testable axes — Painful, Urgent, Recognized. The English cousins: the "hair on fire" problem (acute + urgent), the must-have/nice-to-have line, and MEDDIC's *Pain* + compelling event.

**The mapping** the skill applies, combining DUR with the must-have test and the struggling-moment intensity:

| `painSeverity` | DUR | Must-have test | Struggling moment |
|---|---|---|---|
| **extreme** | D + U + R, all strong; compelling event with a deadline | beyond "very disappointed" | acute; will reprioritize now |
| **severe** | D + R strong, U building; actively looking | "very disappointed" | real, costly, no hard deadline |
| **moderate** | D moderate, R yes, U absent; workaround tolerated | "somewhat disappointed" | recognized but livable |
| **low** | weak on all three; not looking | "not disappointed" | latent or unrecognized |

**Decision rule:** urgency promotes, tolerance demotes. And the PMF flag: if the *primary* personas for a product sit at `low`/`moderate`, the problem may not be DUR enough — worth raising before building further.

Sources: learningloop.io & pmfsurvey.com (Sean Ellis test), FitSignal, Sleekplan, Zonkafeedback (Superhuman 22%→58%). Sean Ellis introduced the 40% benchmark in 2009; Rahul Vohra's First Round Review essay on Superhuman's PMF engine is the canonical case study.

## 5. How the pieces connect — and where they land in Aporia

- `situation` is built from the **JTBD switch interview** (trigger → workaround → outcome → forces).
- The **intensity of that struggling moment** plus **DUR** and the **must-have test** set `painSeverity`.
- The person's place in the **DMU** sets `role`.
- Modern persona doctrine keeps the whole thing **behavioral, specific, and decision-driving** — and capped at a handful of live personas per product.

In Aporia these four fields (`name`, `role`, `situation`, `painSeverity`, plus `status`) are not a marketing artifact — they are a **Constitution invariant (Ring 0)**. Features bind to a persona by id (`relatedPersonaIds`), and the product's grounding rule says every feature should trace to an invariant — a persona, a principle, or the thesis. A feature grounded in *nothing* gets flagged. So a persona that can't change a build decision fails the test twice over: as a persona, and as a piece of the Constitution. That is why the interview is strict and why the write only happens with the human in the loop.
