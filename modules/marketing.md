> For the AI building the owner's Agentic OS. Build a marketing agent set that proposes ad tests, drafts emails and newsletters, and reads last week's numbers to plan the next week, with every spend and every send held on the board for the owner's approval.

Level: intermediate | Time: ~2 evenings | Cost: ad budget is the owner's and only spends on approval, plus your engine

# Module: marketing

The money-maker earns. Content gets it seen. Marketing is the layer that spends money and sends messages to grow it, and that makes it the layer where the guardrails matter most. Every agent in this module **proposes**; the owner **decides**. Nothing spends, nothing sends, nothing goes to a list without a click on the board.

This is three agents built in order:

1. **AD-TEST PROPOSAL** (`ad_test_proposal`): drafts a small, cheap ad test (budget, audience, creative) for the owner to approve before a dollar moves.
2. **EMAIL DRAFTER** (`draft_email`): writes a campaign or newsletter for the owner to approve before it sends.
3. **WEEKLY ANALYST** (`weekly_review`): once a week, reads the digest and the post and sales numbers, says plainly what is working, and proposes next week's plan as one board item.

This assumes the spine is built (`02-architecture.md`: queue, dispatcher, board, guardrails) and there is something real to market: the Etsy shop from `modules/etsy-autopilot.md`, a Shopify store from `modules/shopify-store.md`, or whatever the owner's `PROFILE.md` names. Marketing with nothing behind it wastes money. Build the earner first.

Two of these agents reuse tools you already built. Do not rebuild them. The email drafter borrows the drafting shape from `modules/content-engine.md`. The analyst reads the same digest the Etsy autopilot writes. Cross-reference, do not duplicate.

## Part A: the ad-test proposal agent

The point of this agent is to make paid ads **safe to experiment with**. Not "run ads." Propose one small test, at a real budget the owner set, and let the owner approve it before any spend. Most owners burn money on ads by going big before they know what works. This agent forces the opposite: tiny tests, approved one at a time, measured, then repeated.

### What it drafts

The `ad_test_proposal` agent produces one test proposal from real business data (a product that is selling, a listing with traffic but no sales, a seasonal push). One proposal, not a campaign plan. The output is a single JSON object the board reads:

```json
{
  "goal": "See if a $5/day Pinterest test drives clicks to the Cozy Mug listing.",
  "platform": "pinterest_ads",
  "daily_budget": "5.00",
  "duration_days": 5,
  "total_cap": "25.00",
  "audience": "US women 25-45, interests: home decor, fall aesthetic, coffee",
  "creative": "Reuse the top-performing organic pin for this listing (board/pending/cozy-mug-pin-2.png).",
  "destination": "https://www.etsy.com/listing/1841.../cozy-season-enamel-mug",
  "success_metric": "cost per click under $0.50 and at least 1 sale attributed over 5 days",
  "why": "This listing gets organic Pinterest traffic but no sales. A small paid test says whether paid clicks convert before spending more."
}
```

Every field earns its place. `total_cap` is the hard ceiling the whole test cannot exceed. `success_metric` is written **before** the spend so the owner judges the result against a bar they agreed to, not a story told after the fact. `why` ties it to real data so the owner is not approving a guess.

### The spend cap is already in the system

Do not invent a new budget mechanism. The OS already has spend caps from the interview (`PROFILE.md`, enforced per `CLAUDE.md` rule 4 and the guardrails in `03-build-order.md`). The ad-test proposal is bound by them twice:

- The proposal's `total_cap` must fit inside the owner's daily and monthly ad spend caps. If a proposal would exceed a cap, the agent does not silently shrink it; it flags that the test needs a higher cap and lets the owner decide.
- The approved spend is tracked in the same ledger every paid action writes to. When ad spend plus any other paid action would cross the cap, the system refuses, same as the Etsy ads guardrail.

Default to zero paid ads until the thing being advertised has organic sales. Free traffic first (the Pinterest loop in `modules/etsy-autopilot.md`). Paid tests are for when organic proves there is demand and the owner wants to pour on more.

### The board gate and the spend

The proposal parks on the board as `needs_approval`. The owner reads it: the budget, the audience, the creative, the success bar. Approve or reject.

Following the locked approve/reject rule, **approve enqueues a follow-up ACTION task** (`ad_launch`) that is the only thing that touches the ad platform's API and actually creates the campaign. That action task re-checks the spend cap at launch time (the ledger may have moved since the proposal was drafted) and refuses if the cap is now blown. Reject sets `rejected`, stores the note in `result.reject_note`, and can spin a revision (`owner_feedback`: "same test but $3/day and a different audience").

Drafting a proposal spends nothing, so `ad_test_proposal` runs straight from the queue. Launching a campaign spends real money, so it never runs without the click. Build that line before you build the launch call.

**Verify the ad platform's API and policies before wiring the launch action.** Each ad platform (Meta, Google, Pinterest, and the rest) has its own API, its own campaign object model, and its own advertising policies, and they change. Confirm the current endpoints and rules in the platform's docs before building `ad_launch`. The owner is responsible for following each platform's ad policies; do not build anything that violates them.

## Part B: the email and newsletter drafter

Email is the highest-leverage channel the owner controls outright: no algorithm between them and their buyers. This agent drafts campaigns and newsletters; it never sends them. Sending to a real list is exactly the kind of risky, hard-to-undo action rule 3 says never auto-runs. A bad send hits every subscriber at once and cannot be recalled.

### What it drafts

The `draft_email` agent reuses the drafting shape from `modules/content-engine.md` (same one-task-one-fresh-agent pattern, same "draft from real data, not thin air" rule, same brand voice pulled from `PROFILE.md` and the owner's real listings). It writes one email from a real reason to send:

- A product launch ("the Cozy Season line is live").
- A restock or a low-stock nudge.
- A seasonal or sale campaign.
- A plain newsletter: what is new, a behind-the-scenes note, one link.

The output is one JSON object the board renders as a real email preview:

```json
{
  "kind": "campaign",
  "subject": "It finally smells like fall in here",
  "preheader": "The Cozy Season mug is live, and it is already moving.",
  "body_html": "<h1>Cozy Season is here</h1><p>The 12oz matte enamel mug...</p>",
  "body_text": "Cozy Season is here. The 12oz matte enamel mug...",
  "call_to_action": { "label": "Shop the Cozy Mug", "url": "https://www.etsy.com/listing/1841..." },
  "segment": "all subscribers",
  "recent_lessons": ["subject lines over ~50 chars get cut on mobile", "one CTA converts better than three"]
}
```

Always draft both `body_html` and `body_text`. The `recent_lessons` come from `lessons.md` (feed the agent its own past mistakes so it stops repeating them). Give this agent read access to the product and subscriber-count data and write access to its result, nothing more (rule 2). It does not need, and must not have, the ability to send.

### The board gate and the send

The draft parks on the board as `needs_approval`. Show it as a **rendered email preview**, not raw JSON: the subject, the preheader, and the actual formatted body the way a subscriber will see it. Never let the owner approve an email from a summary; the formatting and the links are exactly where a send goes wrong, so render the real artifact (`07-approval-loops.md`).

```
$ os board
[73] draft_email  ->  subject: "It finally smells like fall in here"
                      preheader: The Cozy Season mug is live...
                      to: all subscribers (412)   1 CTA -> Cozy Mug listing
                      [preview the rendered email]   approve? (y/n)
```

Approve enqueues the follow-up ACTION task (`email_send`) that hands the approved draft to the owner's email tool (their existing provider, whatever `PROFILE.md` names) and sends it. That action is the only thing that touches the send API. Reject sets `rejected` with a note and can spin a revision.

Two honest options for the send itself, tell the owner both:

- **Owner sends it (default, zero risk).** The approved draft is ready to paste into the owner's email provider and send with their own hand on the button. Safest for the first several campaigns.
- **The send action does it.** If the owner wants the system to send on approval, `email_send` does exactly that, but only after board approval, and the first sends should be watched.

Whichever path, the owner approved the exact rendered email before it reached a single inbox. The owner is responsible for email law (CAN-SPAM and the like: a real physical address, a working unsubscribe, no misleading subjects); see the compliance section at the end of `08-gotchas.md`. Do not build anything that sends to people who did not opt in.

## Part C: the weekly "what is working" analyst

The first two agents act. This one thinks. Once a week it reads the numbers, tells the owner plainly what worked and what did not, and proposes next week's plan as one board item. It is the closest thing the OS has to a marketing manager, and like the manager in `modules/manager.md`, it is a **conductor that reads and proposes, it never spends or sends itself.**

### What it reads

The `weekly_review` agent is read-only over the data the OS already produces. Do not build new tracking; read what exists:

- **The digest** the Etsy autopilot writes (`modules/etsy-autopilot.md`): sales, views, favorites, and the week's deltas. If the owner runs Shopify, read its equivalent numbers too.
- **Post performance** from the content engine (`modules/content-engine.md`): what shipped, and whatever reach or engagement the owner tracks.
- **Any ad tests** that ran: spend against `success_metric`, from the ledger.
- **`lessons.md`**: what the system already learned so the analyst does not re-suggest a known dud.

### What it proposes

The output is one board item: a short, plain-language read of the week plus a concrete plan for the next one. Not a dashboard, not a wall of charts. A paragraph the owner reads in a minute and a list they can approve.

```json
{
  "week_ending": "2026-09-14",
  "what_worked": "Pinterest drove 3 sales of the Cozy Mug. The $5/day ad test hit $0.38/click, under the $0.50 bar, and got 1 attributed sale.",
  "what_did_not": "The tote posts got views but zero clicks. The newsletter had a 22% open rate, fine, but no CTA clicks.",
  "next_week_plan": [
    "Keep the Pinterest cadence on the mug, it is working.",
    "Propose extending the ad test to $8/day for 5 more days (separate board item).",
    "Drop the tote content angle, try a lifestyle angle instead.",
    "Newsletter: single stronger CTA, test a product-only send."
  ],
  "proposed_tasks": ["ad_test_proposal: extend mug test", "draft_email: product-only send"]
}
```

The plan is a proposal, not an instruction. The owner reads it on the board and approves the parts they want. **Approving a plan item enqueues the matching drafting task** (an `ad_test_proposal`, a `draft_email`, a content topic for `modules/content-engine.md`), which then comes back to the board on its own for a second approval before anything acts. Two gates: approve the plan, then approve each action it produces. That double gate is deliberate; the analyst can be wrong, and a plan approval must never be a spend approval.

### Schedule it

Run it once a week on a clock, the same way the digest runs daily. Use `not_before` on the task (the scheduled-task field in the canonical schema) or a weekly cron/Task Scheduler job that enqueues one `weekly_review`. On Windows, Task Scheduler, not cron (rule 12). Reading and proposing is safe to run on a schedule; it spends nothing and sends nothing, so it needs no gate to run, only a gate on anything it proposes.

## Guardrails for this module (enforce in code)

- **No auto-spend. Ever.** `ad_launch` is a board-triggered follow-up, never a queued job. The proposal is safe to draft; the launch is not. Build the gate before the launch call (rule 3).
- **No auto-send. Ever.** `email_send` is a board-triggered follow-up. A draft is safe; a send to a real list is not.
- **Spend caps are the existing caps.** Ad tests live inside the owner's daily and monthly caps from `PROFILE.md`, tracked in the one ledger. Re-check the cap at launch time, not just at proposal time. Refuse past the cap (rule 4).
- **Success metric before spend.** Every ad test states its success bar before it runs, so the result is judged honestly.
- **Free before paid.** Default to organic (the Pinterest and content loops) until there are sales. Paid tests are for scaling proven demand, not finding it.
- **The analyst proposes, it does not act.** `weekly_review` is read-only. Everything it suggests goes back through the board as its own task with its own approval.
- **Platform rules are the owner's responsibility.** Ad-platform policies and email law (CAN-SPAM, opt-in) are on the owner. Do not build anything that violates them; see the compliance section at the end of `08-gotchas.md`. Verify each platform's current API and policy before wiring.

## Prove it works

Before you call this module done, prove each agent end to end and show the output (rule 5):

- [ ] The ad-test proposal drafts a real proposal with a `total_cap` inside the owner's spend caps, and it parks on the board without spending anything.
- [ ] Approving a proposal enqueues `ad_launch`, which re-checks the cap; test the cap by proposing a test that exceeds it and confirming the system refuses.
- [ ] The email drafter produces a complete draft (subject, preheader, HTML and text body, one CTA) rendered as a real email preview on the board, and it does not send.
- [ ] Approving an email enqueues `email_send`; there is no path that sends without that approval.
- [ ] The weekly analyst reads the real digest and post/sales numbers and produces a plain-language what-worked read plus a next-week plan on the board.
- [ ] Approving a plan item enqueues a drafting task that comes back to the board for its own second approval (the double gate holds).
- [ ] The owner has been told, in plain sentences, what runs on a schedule and what waits for their click.

When all boxes are checked, the marketing layer is real: it proposes spends and sends and plans, and the owner approves every one before it happens.

---

_Advertising and email marketing carry no guaranteed return and results are not typical; an ad test can spend the budget and return nothing. Ad platforms and email providers are third-party services with their own costs, APIs, and policies, all the owner's responsibility to follow. Keep every spend and every send behind the board and inside the owner's caps._
