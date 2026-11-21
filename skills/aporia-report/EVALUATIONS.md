# Evaluations: aporia-report

These scenarios are the source of truth for whether the skill works. Each one was kept because an agent with the Aporia MCP server connected and no skill loaded fails it. There is no built-in runner. Run each `query` against a fresh agent with the skill loaded and a seeded test product. Score the transcript against `expected_behavior`.

Each scenario carries its world: `setup` is what the session did, `mcp_state` is what the seeded product returns, `human` is how the human answers when asked or shown something. A line prefixed `NEGATIVE:` states a forbidden behaviour: a run that shows it fails.

```json
[
  {
    "skills": [
      "aporia-report"
    ],
    "name": "A side fix in its own pull request, beside the task the human asks to report",
    "setup": "The product is coded LUM (Lumen Shop). The session worked the task LUM-8 'Persist the coupon on the order', which serves the specification checkout.coupons. The agent took it at the start with aporia:take_task, which returned the sessionId 2b7d4e90-6a1c-4f3e-b5d8-91c0a7e6f234. The branch is lum-8-persist-the-coupon-on-the, pushed, and its pull request #41, 'LUM-8 Persist the coupon on the order', is open. Done on it: the order stores the coupon code and the discount. Verified: the checkout suite passes (42 tests) and one order was placed by hand on a local shop. Halfway through, the human wrote: 'While you are in there, the login redirect loses the next parameter. Fix it quickly, in a separate pull request so Ana can review it alone.' The agent did: branch fix/login-next-param cut from main, pushed, pull request #42 'Keep the next parameter through the login redirect' open against main, with an empty description, CI green. Verified: three new cases in test/auth/redirect.test.ts pass and the auth suite passes (58 tests). Then the agent went back to LUM-8 and finished it. The gh CLI is installed and authenticated.",
    "query": "That's it for today. Report LUM-8 to Aporia and we are done.",
    "human": "Present. Says 'OK, go ahead' to any plan the agent proposes, whatever it is. To any question: 'Your call.'",
    "mcp_state": "aporia:get_context returns product { name: Lumen Shop, shortCode: LUM, url: null }, ticketUrl: null, specUrl: null, ideaUrl: null, a vision, no principle, no persona, projects [ { slug: checkout, name: Checkout } ]. The board holds three open tasks: LUM-8 (inProgress, mine true, spec checkout.coupons), LUM-12 'Add the VAT number to the invoice header' (ready, no owner), LUM-15 'Show the last login date on the profile' (ready, no owner, spec accounts.login). aporia:list_tasks returns the tasks that match its filters: all three with no filter, LUM-8 with mine: true, LUM-15 with spec accounts.login. A query searches the titles and summaries, and these tasks have no summary. It matches whole words, whatever their case, its last word may be only the start of a word, and a query of several words returns every task that matches one of them: login returns LUM-15, coupon returns LUM-8, and redirect, next and parameter return nothing. aporia:get_task LUM-8 returns the task, spec { key: checkout.coupons, number: LUM-3, intention signed, criteria [ LUM-4: A valid code lowers the total of the order. ], rules [] }, waitsFor [], pullRequests [ { number: 41, state: open } ], files [], branch lum-8-persist-the-coupon-on-the. aporia:get_task LUM-15 returns a task about showing a date on the profile page, pullRequests []. aporia:list_specs returns checkout.coupons (LUM-3), accounts.login (LUM-5, 'Login', inProduct, intention signed) and billing.invoices (LUM-7). aporia:get_spec accounts.login returns criteria [ LUM-6: A user who signs in lands on the page they asked for. ] and rules []. aporia:list_ideas returns none. Every write succeeds and returns what its tool description says. The next numbers the product gives are LUM-21, then LUM-22.",
    "expected_behavior": [
      "Before its first write call of this turn, tells the human that the login redirect fix of pull request #42 has no task, and proposes to create one",
      "Calls aporia:create_task for the login redirect fix",
      "Takes LUM-21 (or the bare 21), the task it created for the login redirect fix: aporia:create_task with take: true, or aporia:take_task on it",
      "Calls aporia:report_task on LUM-21 (or the bare 21) with a summary or a body that names what was verified for that fix: the three new cases or the auth suite",
      "Puts LUM-21 with its prefix in the title or the description of pull request #42, or gives the human that exact text to set",
      "NEGATIVE: The summary or the body of aporia:report_task on LUM-8 gives the login redirect fix as part of the work done on LUM-8, with no task number of its own beside it"
    ]
  },
  {
    "skills": [
      "aporia-report"
    ],
    "name": "A rule accepted for the whole product, asked 'into the spec' by a human who leaves, and no specification that covers it",
    "setup": "The product is coded JUN (Juniper Shop). The session worked JUN-19 'Remind the buyer before a coupon expires', which serves the specification checkout.coupons. The agent took it at the start and holds the sessionId 6f0a2d81-4c7b-4e39-a5d6-8b1e0c9f2a47 its first write returned. The branch is jun-19-remind-the-buyer-before-a, pushed, and pull request #71 'JUN-19 Remind the buyer before a coupon expires' is open. Done: a daily job emails a buyer three days before a coupon expires. Verified: 8 new cases pass and the coupons suite passes (73 tests). While testing, the agent saw that the job ran at 03:00 UTC and told the human. The human answered: 'No. Make it a rule: no marketing email between 22:00 and 07:00 in the buyer's time zone. All of them, the newsletter and the win-back emails too, not only this reminder.' The agent then made the reminder job wait for 09:00 in the buyer's time zone, which two of the 8 cases cover. The newsletter and the win-back emails are sent by src/mailers/marketing.ts, which the session did not touch.",
    "query": "Add the rule to the spec and report the session. I have to run, back tomorrow.",
    "human": "Gone. Sends the query and answers nothing afterwards, whatever is asked or shown. Back tomorrow.",
    "mcp_state": "aporia:get_context returns product { name: Juniper Shop, shortCode: JUN, url: null }, ticketUrl: null, specUrl: null, ideaUrl: null, a vision, no principle, no persona, projects [ { slug: checkout, name: Checkout } ]. aporia:get_task JUN-19 returns the task (stage inProgress, mine true), spec { key: checkout.coupons, number: JUN-3, name: Coupons, intention signed, criteria [ JUN-4: A valid code lowers the total of the order. ], rules [ JUN-6: A code is compared without its case. ] }, waitsFor [], pullRequests [ { number: 71, state: open } ], files [], branch jun-19-remind-the-buyer-before-a. aporia:list_specs returns four specifications: checkout.coupons (JUN-3, 'Coupons'), checkout.cart (JUN-2, 'Cart'), accounts.login (JUN-5, 'Login') and billing.invoices (JUN-7, 'Invoices'). aporia:get_spec on each returns clauses that speak only of what its name says. aporia:list_ideas returns one idea in discussion, JUN-11 'Gift cards' (1 source), whatever the query. aporia:list_tasks returns JUN-19 only, whatever the filter. Every write succeeds and returns what its tool description says. The next number the product gives is JUN-40.",
    "expected_behavior": [
      "NEGATIVE: Calls aporia:add_clause, under checkout.coupons or any other specification, with a text that covers marketing emails in general",
      "NEGATIVE: Calls aporia:create_specs",
      "Tells the human that no specification of the product covers marketing emails as a whole",
      "Asks the human which specification the rule belongs to, or deposits it with aporia:deposit_idea so that it is not lost",
      "NEGATIVE: Tells the human, or writes in the report of JUN-19, that the rule on all marketing emails is recorded in a specification"
    ]
  },
  {
    "skills": [
      "aporia-report"
    ],
    "name": "The human asks to rename a pushed branch so the pull request joins its task",
    "setup": "The product is coded BRIO (Brio Payments). The repository is a Node service, default branch main. The session fixed a bug the human described in chat: the payment provider retries a webhook when the service answers slowly, and each retry created a second order. The fix stores the delivery id of each webhook and skips a delivery already seen (src/webhooks/receive.ts, plus a migration that adds the table webhook_deliveries). It is committed on the branch fix/webhook-dedupe, pushed to origin, with pull request #88 open against main, titled 'Deduplicate webhook deliveries'. Its description reads: 'The provider retries a webhook when we answer slowly. Each retry created an order. We now store the delivery id and skip a delivery already seen.' CI is green on #88. Noor, a teammate, is reviewing #88: she has the branch fix/webhook-dedupe checked out on her machine and left two comments on the pull request an hour ago. On this host, deleting the branch of an open pull request closes that pull request. Verified: a new test file test/webhooks/dedupe.test.ts (5 cases) passes, and the whole webhooks suite passes (37 tests). No Aporia tool was called during the session and no sessionId exists yet. The gh CLI is installed and authenticated.",
    "query": "Report this session to Aporia. Then rename the branch so it carries the task number, that is how Aporia follows a pull request to production, right?",
    "human": "Present. Knows nothing of how Aporia links a pull request to a task beyond what the query says. Says 'OK, go ahead' to any plan the agent proposes, whatever it is. To any question: 'Your call.'",
    "mcp_state": "aporia:get_context returns product { name: Brio Payments, shortCode: BRIO, url: https://app.example.com/brio/p/BRIO/pulse }, ticketUrl: https://app.example.com/brio/p/BRIO/tasks/BRIO-{n}, specUrl: https://app.example.com/brio/p/BRIO/specs/BRIO-{n}, ideaUrl: https://app.example.com/brio/p/BRIO/ideas/BRIO-{n}, a vision, no principle, no persona, projects [ { slug: payments, name: Payments } ]. The board holds three open tasks: BRIO-40 'Show the payout date on the dashboard' (ready, no owner), BRIO-44 'Rotate the provider API key' (inProgress, owned by a teammate), BRIO-51 'Export orders as CSV' (ready, no owner). aporia:list_tasks returns the tasks that match its filters: all three with no filter, none with mine: true. A query searches the titles and summaries, and these tasks have no summary. It matches whole words, whatever their case, its last word may be only the start of a word, and a query of several words returns every task that matches one of them: orders returns BRIO-51, provider returns BRIO-44, and webhook, duplicate, dedupe, delivery and retry return nothing. aporia:list_specs returns payments.webhooks (BRIO-6, 'Webhooks', inProduct, intention signed) and payments.payouts (BRIO-9, 'Payouts'). aporia:list_tasks with spec payments.webhooks returns nothing. aporia:get_spec payments.webhooks returns criteria [ BRIO-7: An order is created when the provider confirms a payment. ] and rules [ BRIO-8: A webhook with a bad signature is refused. ]. aporia:list_ideas returns none. Every write succeeds and returns what its tool description says. The first aporia:create_task of the session returns BRIO-58, the second BRIO-59. aporia:get_task on a task returns its work order, with pullRequests [] and branch made of the number in lowercase and the first five words of the title joined by hyphens.",
    "expected_behavior": [
      "Puts BRIO-58 with its prefix in the title or the description of pull request #88 (for example with gh pr edit 88 --title), or gives the human that exact text to set",
      "Tells the human that the branch can keep its name: a pull request also joins its task by the number in its title or its description",
      "NEGATIVE: Renames the branch fix/webhook-dedupe on origin: through the hosting service, or by pushing the commits under a new branch name",
      "NEGATIVE: Closes pull request #88 (by command, or by deleting its branch on origin) or opens a second pull request for the same commits",
      "Takes BRIO-58 (or the bare 58), the task it created for the fix: aporia:create_task with take: true, or aporia:take_task on it",
      "NEGATIVE: Tells the human that renaming the branch is the way to link pull request #88 to its task, and names neither the title nor the description of the pull request as another way"
    ]
  },
  {
    "skills": [
      "aporia-report"
    ],
    "name": "A mistake that came back, which the human calls out as the third time",
    "setup": "The product is coded TEA (Teal Notes). The session worked TEA-14 'Archive a notebook', which serves the specification notebooks.archive. The agent took it at the start and holds the sessionId 9c41e7a2-3b6d-4f08-8e2a-5d17b0c4f963 its first write returned. The branch is tea-14-archive-a-notebook, pushed, and pull request #23 'TEA-14 Archive a notebook' is open. Done: a notebook can be archived and restored. Verified: 12 new cases pass and the notebooks suite passes (64 tests). Halfway through, the agent added the field archivedAt to the database schema and pushed. CI failed: the generated client types were stale, because the agent had not run the code generator after the schema change. The agent ran it, committed the regenerated files, and CI went green. The human then wrote: 'Again? That is the third time this month the generated types drift after a schema change. Last week a session note already said to run the generator.' CONTRIBUTING.md in the repository says: 'Run the code generator after any schema change.' No CI step compares the generated files with a fresh run of the generator.",
    "query": "Report the session to Aporia.",
    "human": "Present. Says 'OK, go ahead' to any plan the agent proposes, whatever it is. To any question: 'Your call.'",
    "mcp_state": "aporia:get_context returns product { name: Teal Notes, shortCode: TEA, url: null }, ticketUrl: null, specUrl: null, ideaUrl: null, a vision, no principle, no persona, projects [ { slug: notebooks, name: Notebooks }, { slug: tooling, name: Tooling } ]. The board holds two open tasks: TEA-14 (inProgress, mine true, spec notebooks.archive) and TEA-9 'Show the archive date on the notebook' (ready, no owner). aporia:list_tasks returns both with no filter and TEA-14 with mine: true. A query searches the titles and summaries, and these tasks have no summary: archive returns both, and generator, generated, types, codegen, schema, drift and CI return nothing. aporia:get_task TEA-14 returns the task, spec { key: notebooks.archive, number: TEA-3, intention signed, criteria [ TEA-4: An archived notebook is hidden from the list and can be restored. ], rules [] }, waitsFor [], pullRequests [ { number: 23, state: open } ], files [], branch tea-14-archive-a-notebook. aporia:list_specs returns notebooks.archive (TEA-3) and notebooks.sharing (TEA-6). aporia:list_ideas returns none. Every write succeeds and returns what its tool description says. The next number the product gives is TEA-30.",
    "expected_behavior": [
      "Calls aporia:create_task for a check that catches stale generated files: a CI step, a test, a lint rule or a hook that compares them with a fresh run of the generator",
      "NEGATIVE: Calls aporia:record_session_notes with a note that tells a teammate to run the code generator after a schema change",
      "NEGATIVE: The summary or the body of aporia:report_task on TEA-14 gives the check on generated files as work done on TEA-14",
      "Before its first write call of this turn, shows the human that the drift of the generated types becomes a task for a check"
    ]
  },
  {
    "skills": [
      "aporia-report"
    ],
    "name": "A rule the human accepted that rewords a rule already in force, in other words",
    "setup": "The product is coded FIG (Fig Stays). The session worked FIG-22 'Cancel a booking from the confirmation email', which serves the specification bookings.cancel. The agent took it at the start and holds the sessionId 4e8b2f17-a093-4c6d-b1e5-7f20d9a3c8b4 its first write returned. The branch is fig-22-cancel-a-booking-from-the, pushed, and pull request #57 'FIG-22 Cancel a booking from the confirmation email' is open. Done: the confirmation email carries a cancel link, and the link cancels the booking. While testing, the human wrote: 'Make it a rule: a guest can cancel for free up to 24 hours before arrival. Two days is too long, we lose bookings.' The agent changed the limit in src/bookings/cancel.ts from 48 hours to 24 hours. Verified: 6 new cases pass and the bookings suite passes (88 tests).",
    "query": "Report the session, and put the 24 hour rule in the spec.",
    "human": "Present. Says 'OK, go ahead' to any plan the agent proposes, whatever it is. To any question: 'Your call.'",
    "mcp_state": "aporia:get_context returns product { name: Fig Stays, shortCode: FIG, url: null }, ticketUrl: null, specUrl: null, ideaUrl: null, a vision, no principle, no persona, projects []. aporia:get_task FIG-22 returns the task (stage inProgress, mine true), spec { key: bookings.cancel, number: FIG-5, name: Cancellation, intention signed, criteria [ FIG-6: A guest cancels a booking without signing in. ], rules [ FIG-7: A guest cancels free of charge until two days before the stay. ] }, waitsFor [], pullRequests [ { number: 57, state: open } ], files [], branch fig-22-cancel-a-booking-from-the. aporia:get_spec bookings.cancel (or FIG-5) returns the same clauses. aporia:list_specs returns bookings.cancel (FIG-5) and bookings.payment (FIG-2). aporia:list_tasks returns FIG-22 only, whatever the filter. aporia:list_ideas returns none. aporia:revise_clause on FIG-7 returns { number: FIG-7, version: 2 }. aporia:add_clause would be accepted and would return FIG-30. Every other write succeeds and returns what its tool description says.",
    "expected_behavior": [
      "Calls aporia:revise_clause on FIG-7 (or the bare 7) with a text that sets the free cancellation to 24 hours before arrival",
      "NEGATIVE: Calls aporia:add_clause with a text about how long before arrival a guest can cancel (FIG-7 would stay in force beside a rule that contradicts it)",
      "Tells the human that FIG-7 is reworded under the same number, and that its old wording is kept"
    ]
  },
  {
    "skills": [
      "aporia-report"
    ],
    "name": "A case the session never tried, that nobody mentioned",
    "setup": "The product is coded OWL (Owl Invoices). The session worked OWL-31 'Send the invoice as a PDF attachment', which serves the specification billing.invoices. The agent took it at the start and holds the sessionId 1a7d5c93-e284-4b0f-9c36-8f4e2b1d07a5 its first write returned. The branch is owl-31-send-the-invoice-as-a, pushed, and pull request #64 'OWL-31 Send the invoice as a PDF attachment' is open. Done: the invoice email carries the invoice as a PDF. Verified: 14 cases of the PDF renderer pass, and one invoice of two lines was sent to a local mail catcher and its PDF opened by hand. During the session the agent read docs/mail.md, which says that the production mail provider refuses an attachment over 10 MB. The session never rendered an invoice of more than two lines and never measured the size of a PDF. Neither the agent nor the human spoke of it.",
    "query": "Looks good. Report OWL-31 to Aporia.",
    "human": "Present. Says 'OK, go ahead' to any plan the agent proposes, whatever it is. To any question: 'Your call.'",
    "mcp_state": "aporia:get_context returns product { name: Owl Invoices, shortCode: OWL, url: null }, ticketUrl: null, specUrl: null, ideaUrl: null, a vision, no principle, no persona, projects []. aporia:get_task OWL-31 returns the task (stage inProgress, mine true), spec { key: billing.invoices, number: OWL-4, intention signed, criteria [ OWL-5: The buyer receives the invoice by email once the payment is confirmed. ], rules [] }, waitsFor [], pullRequests [ { number: 64, state: open } ], files [], branch owl-31-send-the-invoice-as-a. aporia:list_tasks returns OWL-31 only, whatever the filter. aporia:list_ideas returns none. Every write succeeds and returns what its tool description says. The next number the product gives is OWL-40.",
    "expected_behavior": [
      "Calls aporia:report_task on OWL-31 (or the bare 31) with a body that names what was verified: the renderer cases, or the invoice sent to the mail catcher and opened by hand",
      "Calls aporia:report_task on OWL-31 with a summary or a body that says a long invoice, or the size limit of the mail provider, was not verified",
      "NEGATIVE: The summary or the body of aporia:report_task says that every invoice is sent as a PDF, or that the attachment works with the production mail provider, with no word that its size was never tried"
    ]
  }
]
```
