> For the AI building the owner's Agentic OS. The advanced capstone: a generator agent that, given ONE approved niche, scaffolds a whole draft shop in a single pass (starter listings, matching POD drafts, a week of promo, a content calendar) and parks all of it on the board for one human review. Build this only AFTER the core system is proven.

Level: capstone, build last | Time: ~1 evening once the modules it composes are proven | Cost: the child modules' metered POD/asset credits per batch (capped), plus your engine

# Module: shop in a box

This is the advanced build (build it after the core works). Given an approved niche, one agent scaffolds an entire draft shop in a single pass: a batch of starter listings, a matching set of print-on-demand drafts, a week of promotion, and a simple content calendar to run it all. Then it stops. Everything it made lands on the board as drafts, and the owner does one review pass over the whole shop instead of approving twenty pieces one at a time.

Read this clearly before you build it: **this is not new magic. It is composition.** Every capability the generator uses already exists in modules the owner built and proved. The listing drafting is `modules/etsy-autopilot.md`. The POD drafts are `modules/pod-printify.md`. The pins and posts are `modules/content-engine.md`. The board, the queue, the dispatcher, the spend caps, the no-auto-publish line: all already built and already working. The generator is a conductor that calls those pieces in order and gathers the output into one review. It invents nothing new and it is allowed to skip nothing safe.

## When to build this (and when not to)

Do not lead a beginner here. If the owner does not yet have a working shop that reports its numbers, drafts a real listing, creates a real POD draft, and generates real pins, this module is premature. Building a whole draft shop before you can build one proven listing just multiplies whatever is broken by twenty.

Build this only when all of these are true:

- The spine (queue, dispatcher, board, guardrails) is built and proven (`02-architecture.md`, `03-build-order.md`, `04-first-agent.md`).
- The Etsy autopilot works end to end: a real digest, a real draft listing on the board (`modules/etsy-autopilot.md`).
- If the owner does POD, the Printify pipeline creates a real draft and returns mockups (`modules/pod-printify.md`).
- The content engine produces at least real pins with the correct destination URL (`modules/content-engine.md`).
- The owner has approved and published at least one real thing by hand, so they trust the board.

If any of those is missing, stop and say so plainly: "This one stacks on top of the working shop. Let's get the first sale from the core system first, then I'll build the generator." Then go build the missing piece. This is the reward for having a working system, not a shortcut around building one.

## What the generator produces (one niche in, one draft shop out)

The input is a single approved niche from the owner. Not a guess the agent made: a niche the owner said yes to. The generator does not pick the business. It fills one in.

From that one niche, in a single pass, it scaffolds:

- **A batch of starter listings.** A small set of draft listings (title, 13 tags, description, SEO), each for a distinct product idea inside the niche. Default the batch to 5. Not 50. Enough to open a shop, few enough to review in one sitting.
- **Matching POD drafts.** If the owner sells print-on-demand, a draft Printify product for each listing that needs a physical good, with mockups pulled back. Digital or handmade sellers skip this and keep just the listings.
- **A week of promotion.** Seven days of promo pieces: pins for the listings and short post copy, enough to seed the first week of traffic. This reuses the content engine's draft-and-generate half, never its post half.
- **A simple content calendar.** A plain file that lays out which promo piece goes out on which day for the first week, so the owner has a runnable plan instead of a pile of loose assets.

All of it lands as **drafts on the board.** Nothing publishes. Nothing spends. Nothing posts. The generator's entire job ends at "here is a draft shop, review it."

## How it works: a conductor, not a new engine

The generator is one agent task type, `scaffold_shop`, but it does not do the drafting, the POD calls, or the pin rendering itself. It **enqueues the existing task types** and lets the dispatcher run each one in its own fresh session (rule 2). This matters: one giant agent trying to write listings and build products and render pins in a single context would drift, forget, and blow its budget. The pattern that already works is one task, one fresh agent. The generator respects that by fanning the work out across the same queue everything else uses.

The flow, in plain ASCII:

```
approved niche
     |
     v
  scaffold_shop agent  (the conductor: plans the batch, enqueues children, then STOPS)
     |
     +--> enqueue N x draft_listing        (etsy-autopilot: title, tags, description)
     |
     +--> enqueue N x make_pod_product     (pod-printify: draft product + mockups)   [if POD]
     |
     +--> enqueue 7 x draft_post/make_pins (content-engine: a week of promo)
     |
     +--> write data/content-calendar.md   (which promo runs which day)
     |
     v
  every child task runs in its OWN fresh agent via the dispatcher
     |
     v
  all results land in needs_approval, grouped by a shared batch_id
     |
     v
  BOARD: one review pass over the whole draft shop
     |
   approve per item (or approve-all-listings, etc.)
     |
     v
  owner publishes / posts the approved pieces (still gated, still one click each for live actions)
```

The conductor's own work is small and safe: it reads the niche, decides the batch (how many listings, which product types, what the week of promo covers), enqueues the child tasks with the right payloads, writes the calendar file, and marks itself done. It does not call Etsy, Printify, or any posting API directly. It hands specs to the agents that already know how.

## Step 1: the plan (the only thing the conductor decides)

Give the `scaffold_shop` agent the approved niche plus the owner's `PROFILE.md` (brand voice, product types, spend caps) and have it return a plan first, before enqueuing anything:

- **The product ideas.** N distinct product concepts inside the niche (default 5). Each is a short spec: a product name, a one-line angle, and which product type it maps to (digital download, tee, mug, tote).
- **The promo week.** 7 promo slots, each tied to one of the product ideas or a general niche hook, so the week does not repeat the same pin seven times.
- **The calendar.** A day-by-day layout: which listing goes up when, which promo piece ships which day.

Have the conductor write this plan to a result and, on a first run, park the PLAN itself on the board before it builds anything. That gives the owner a cheap early exit: they see the shape of the shop (five product ideas, a week of promo) for the cost of one text draft, and they can redirect before any POD credits or asset generation get spent. This is the confidence ramp from `07-approval-loops.md` applied to a big build: approve the plan, then let it scaffold.

Once the owner approves the plan (or if the owner has run this before and trusts it, set a profile flag to auto-proceed past the plan gate), the conductor enqueues the children.

## Step 2: fan out to the existing agents

For each product idea, the conductor enqueues the real task types the owner already built. It does not reimplement them. It passes each one a tight payload:

- `draft_listing` with the product name, angle, niche, and the owner's voice notes. The listing agent from `modules/etsy-autopilot.md` does exactly what it already does: title front-loaded with search terms, 13 tags each under 20 characters, a real description. It parks the draft in `needs_approval`.
- `make_pod_product` (only if the product idea is a physical good and the owner does POD) with the design reference and product type. The POD agent from `modules/pod-printify.md` creates a Printify DRAFT and returns mockups. It never publishes. The per-product-type scale rules (mug ~0.52, tote ~0.72 to 0.85) and the 100-variant cap still apply, because it is the same agent.
- `draft_post` / `make_pins` for each of the 7 promo slots. The content engine from `modules/content-engine.md` drafts the hook and caption and renders the pins from the listing's own images, with the correct Etsy destination URL. It stops at generated files. It does not post.

Tag every child task with a shared `batch_id` (the nullable `batch_id` column in the tasks schema, made for exactly this) so the board can group them. That grouping is what turns twenty scattered approvals into one shop review.

Because each child runs as its own fresh `claude -p` call through the dispatcher, they are cheap, isolated, and independently retryable. If one listing draft fails, the dispatcher recovers that one task; it does not take down the whole shop. Orphan recovery and per-task timeouts you already built cover this for free.

## Step 3: the content calendar

The conductor writes a plain-text `data/content-calendar.md` (a readable file, not a database, matching how the rest of the system stores memory). It maps the week:

```
# First-week content calendar  (shop batch 2026-07-14-a)

Day 1  Publish: "Cozy Season Mug" listing      Post: pin set A -> mug listing URL
Day 2  Publish: "Autumn Tote" listing          Post: pin set B -> tote listing URL
Day 3  (no new listing)                         Post: niche hook "how these get made"
Day 4  Publish: "Fall Leaves Tee" listing       Post: pin set C -> tee listing URL
Day 5  (no new listing)                         Post: pin set A reshare, second board
Day 6  Publish: "Warm Drinks Print" listing     Post: pin set D -> print listing URL
Day 7  (no new listing)                         Post: seasonal hook + best performer
```

This is a runnable plan the owner can follow by hand or feed to a daily scheduler later. Keep it simple. It is a map for the first week, not a content strategy engine. The owner can edit it in a text editor because it is a text file.

## Step 4: one review pass

This is the payoff and the guardrail in the same place. All the drafts sit on the board under one `batch_id`. The owner does a single review pass over the whole draft shop:

```
$ os board --batch 2026-07-14-a
SHOP BATCH 2026-07-14-a  (5 listings, 5 POD drafts, 7 promo pieces)

  Listings
  [51] draft_listing   "Cozy Season Mug | 13 tags | $18 draft"        approve? (y/n)
  [52] draft_listing   "Autumn Tote | 13 tags | $24 draft"           approve? (y/n)
  ...
  POD drafts
  [56] make_pod_product "Printify draft: Cozy Season Mug, 2 mockups"  approve? (y/n)
  ...
  Promo
  [61] make_pins       "3 pins -> mug listing URL"                    approve? (y/n)
  ...

  approve-all-listings  approve-all-pods  approve-all-promo  reject <id> <note>
```

Always render the real artifact, never approve from raw JSON (this is the rule from `07-approval-loops.md`). Show the listing text, the Printify mockup image, and a pin preview. A batch review does not mean a blind bulk yes. It means the owner can move fast through good drafts and stop on the ones that need work, all in one sitting, with the real rendered output in front of them.

Reject with feedback still works per item: reject one listing with a note, and that note is a candidate line for the plain-text lessons memory (`lessons.md`) so the next scaffold does not repeat it. Approving a listing or a POD draft moves it to its normal follow-up publish task, which is still one gated action each. Approving promo marks it ready to post. The generator got the owner to a full draft shop in one pass; it did not remove a single live-action gate.

## Guardrails (all inherited, none relaxed)

The whole point of this module is that going bigger does not mean going looser. Every guardrail from the child modules stays exactly as strict:

- **Nothing publishes automatically.** Every `draft_listing` and `make_pod_product` lands in `needs_approval`, same as always. There is no code path in the generator that publishes a listing or pushes a Printify product live. Scaffolding produces drafts. Live actions are still one human click each (rule 3).
- **Nothing posts automatically.** The promo pieces are generated files and copy. The content engine's post step is never called by the generator. Posting stays a separate, board-approved action (`modules/content-engine.md`).
- **Nothing spends past the cap.** POD draft creation and asset generation cost credits. The generator counts every child against the owner's daily spend and task caps from `PROFILE.md`, and it refuses to enqueue past them. A "whole shop in one pass" is exactly the kind of build that could run up a bill, so the batch itself gets a hard ceiling: max listings, max POD drafts, max assets per scaffold. If the plan would exceed a cap, the generator shrinks the batch and tells the owner why, it does not blow through the limit (rule 4).
- **The batch has its own stop conditions.** Max children per scaffold, max total spend per scaffold, max runtime. A generator that enqueues in a loop is a runaway risk; cap it like every other loop.
- **One niche, owner-approved.** The generator never invents or auto-approves a niche. It fills in a niche the owner said yes to. On the first run it also parks the plan for approval before spending anything.
- **Fresh session per child.** The conductor enqueues; the dispatcher runs each child in its own fresh agent. No giant do-everything agent (rule 2). This keeps a twenty-piece build as cheap and predictable as twenty separate one-piece builds, because that is exactly what it is.
- **No secrets in the payloads.** Child tasks read keys from `.env` like they always do. The conductor passes specs, not credentials, and never logs a token (rule 11).

## Prove it works

Do not tell the owner the generator is done until you have run it once, small, and watched it behave:

- [ ] Run `scaffold_shop` on one approved niche with the batch capped low (say 2 listings, 2 promo pieces) for the first real test.
- [ ] The conductor parks a readable PLAN on the board before spending anything, and the owner can approve or redirect it.
- [ ] On approval, the child tasks fan out and each runs in its own fresh agent; the board shows them grouped under one `batch_id`.
- [ ] Every listing draft is valid (title within limits, 13 tags each under 20 characters, real description) and sits in `needs_approval`.
- [ ] If POD applies, each POD draft exists in Printify as unpublished with mockups returned, and none published.
- [ ] The promo pieces are real generated files carrying the correct destination URL, and none posted.
- [ ] `data/content-calendar.md` exists, is readable, and maps the week to the drafts that were actually made.
- [ ] Nothing published, posted, or spent past the cap without a click. Try to exceed the batch spend cap on purpose and confirm the generator refuses and shrinks the batch.
- [ ] The owner did one review pass and understands, in plain sentences, that they now hold a draft shop, not a live one, and what each approval will do next.

When those are all true, scale the batch up to the real default (5) and let the owner run it on a niche they actually want to open. Function first, even here: a draft shop that is correct and fully gated is the win. Make it look nicer only after the owner has published from it and seen it work.

---

_This module composes other modules; it introduces no new selling channel and no new risk surface, only more volume behind the same gates. All third-party costs from the child modules still apply: Printify charges per item on a sale, the content tools meter their own credits, and the AI engine has its usage cost. A scaffolded shop is drafts, not sales. Results are not typical; a full draft shop can still get zero orders. Keep the owner's spend caps enforced and their expectations honest._
