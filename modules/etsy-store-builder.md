> For the AI building the owner's Agentic OS. THE FLAGSHIP FIRST BUILD: orchestrate the pieces the owner already has into an actual branded, launch-ready Etsy store. Brand kit, then shop identity, then a starter product line, then listings, then a launch calendar, all as drafts under ONE batch_id for a single "here is your store" review. On approval, it is launch-ready. This is the concrete "the AI built me a branded Etsy store" outcome.

Level: intermediate, the recommended first build | Time: ~2 to 3 evenings once the modules it composes exist | Cost: the image-generator and POD draft credits for one starter line (capped), plus your engine

# Module: the Etsy store builder (flagship)

This is the headline of the whole system. Everything else in the Blueprint is the machinery. This is the payoff the owner actually wanted: they say "build me a store," and the AI comes back with a branded Etsy shop, a matching starter product line, written listings, and a plan to launch it, all sitting on the board for one review. Approve it, and the store is launch-ready.

Build this FIRST when the owner's answer to the money-maker question was "a branded Etsy store from scratch." It is the most concrete win the system produces, and it is the one that makes the owner believe the rest.

Read this before you build it, because it is the same truth as `modules/shop-in-a-box.md`: **this is not new magic, it is composition.** The brand comes from `modules/brand-kit.md`. The products come from `modules/pod-printify.md`. The listings come from the listing agent in `modules/etsy-autopilot.md`. The launch content comes from `modules/content-engine.md`. The store builder is a conductor that calls those pieces in the right order, on-brand, and gathers everything into one review. It invents nothing new and it skips nothing safe.

**Timing note.** This is Step 5 of `03-build-order.md`, BEFORE the automated queue and board are built (Step 6). So the "one review" here is you showing the owner the whole batch of drafts (the shop identity, the product line, the listings) and getting their yes by hand, not a `needs_approval` board that does not exist yet. The pieces still route through the owner's approval; that approval is just informal until Step 6 formalizes it. Everything the owner approves stays as clean, portable drafts (titles, descriptions, prices, images) ready to publish on their yes.

**If this is a fresh build and the child agents do not exist yet, that is expected.** A brand-new owner who chose "branded store from scratch" will not have the POD agent or the listing agent built. Build each one AS YOU REACH ITS STEP, from its own module (the POD agent from `modules/pod-printify.md` at Step 3, the listing agent from `modules/etsy-autopilot.md` at Step 4), prove it works on one item, then let the conductor use it for the rest. You do not need to build all of `etsy-autopilot` (the digest and promotion agents come later, when the store is live); you only need its listing agent for now.

The difference between this and shop-in-a-box: shop-in-a-box fills in a niche you already have a shop for. The store builder creates the whole IDENTITY first (name, logo, voice, shop pages) and builds the product line to match it. It is the "start a real business from zero" version.

## What "done" looks like

One batch on the board, grouped under a single `batch_id`, that the owner reviews in one sitting:

+ A **brand kit** the owner approved (name pick, logo, palette, voice) from `modules/brand-kit.md`.
+ A **shop identity**: the shop icon, banner, title, announcement, About section, and policies, all written in the brand voice, ready to paste into Etsy.
+ A **starter product line**: a handful of on-brand Printify DRAFTS with mockups (or digital/handmade listings if that is the owner's model).
+ **Listings** for each product, written in the brand voice.
+ A **launch content calendar**: the first week or two of promo mapped to the products.

Nothing is live. Nothing published. Nothing spent past the cap. The owner does one review pass, approves, and then follows the launch-ready checklist to open the doors. After that, `modules/etsy-autopilot.md` runs and grows the store day to day, and `modules/manager.md` runs it on a schedule if the owner wants that.

## The conductor and the flow

The store builder is one task type, `build_store`. Like every conductor in this system, it does the small safe work itself (plan, enqueue, write the calendar, stop) and hands every real drafting job to the existing agents through the queue, each in its own fresh session (rule 2). One giant agent trying to brand, design products, write listings, and plan content in a single context would drift and blow its budget. Fan it out.

The flow, in plain ASCII:

```
owner says "build my store"  (money-maker = branded Etsy store)
     |
     v
  build_store conductor  (reads/ runs brand, plans the line, enqueues children under ONE batch_id, then STOPS)
     |
     +--> STEP 1  brand kit           -> run modules/brand-kit.md (or read existing brand.md)
     |
     +--> STEP 2  shop identity        -> enqueue draft_shop_identity  (icon, banner, title, announcement, About, policies)
     |
     +--> STEP 3  starter product line -> enqueue N x make_pod_product  (pod-printify: on-brand drafts + mockups)   [if POD]
     |
     +--> STEP 4  listings             -> enqueue N x draft_listing     (etsy-autopilot: title, 13 tags, description, in brand voice)
     |
     +--> STEP 5  launch calendar       -> write data/launch-calendar.md  (cross-ref content-engine)
     |
     v
  every child runs in its OWN fresh agent via the dispatcher
     |
     v
  all results land in needs_approval, grouped by the shared batch_id
     |
     v
  BOARD: one review pass  "here is your store"
     |
   approve (or reject-with-feedback per item)
     |
     v
  LAUNCH-READY: owner follows the paste-in checklist, opens the shop, then etsy-autopilot runs and grows it
```

## Step 0: the plan gate (cheap early exit)

Before it spends a single image credit or POD credit, have the `build_store` conductor read the owner's `PROFILE.md` and return a PLAN, and park that plan on the board first. The plan is one text draft. It lays out:

+ The niche and the shop concept in one or two lines.
+ How many products the starter line has (default 5, see Step 3) and what each one is.
+ Which product types (digital download, tee, mug, tote) each maps to.
+ The shape of the launch calendar (how many days, which products lead).

This is the confidence ramp from `07-approval-loops.md` applied to the biggest build in the system. The owner sees the shape of their whole store for the price of one text draft, and can redirect before any credits burn. Approve the plan, then the conductor runs the steps below. If the owner has run this before and trusts it, a profile flag can auto-proceed past this gate, but default to showing the plan.

## Step 1: the brand (identity first, because this is a real business)

This is the one place the store builder deliberately leads with looks, and the vanity-trap rule in `CLAUDE.md` still holds: you are not polishing a system that does not work yet, you are creating the brand identity a real storefront legally and practically needs before it can open. A shop with no name, no icon, and no About section is not a store. So the brand is step one, not a distraction from function.

Run `modules/brand-kit.md`. If the owner already has a `brand.md` (or `brand/` folder), read it instead and skip the interview. The brand kit produces, all as board drafts:

+ 3 to 5 shop-name options (with the honest "check availability on Etsy yourself" note, Etsy name availability is not something the API guarantees for you).
+ A logo, generated via an AI image generator (the owner picks a current tool, verify its current docs, same honest posture as every third-party tool in this Blueprint).
+ A color palette.
+ A short brand-voice guide (tone, words to use, words to avoid).
+ A product-design style.

The output is the `brand.md` that every later step reads, so the shop identity, the product designs, and the listing copy all come out of the same brand instead of five different vibes. Do not let the owner skip this and then wonder why the store feels generic. The brand is the thread that ties the batch together.

Everything the brand kit produces lands under the same `batch_id` as the rest of the store, so it is part of the one review.

## Step 2: the shop identity (the storefront pages, in brand voice)

This is the piece that makes it a STORE and not just a pile of listings. One task type, `draft_shop_identity`, takes `brand.md` and drafts every part of the Etsy shopfront in the brand voice:

+ **Shop icon.** Derived from the approved logo (a square crop or simplified mark). Written to a file under the task's board folder.
+ **Shop banner.** A wide header image in the brand palette and style, generated by the same image tool the brand kit used. File on disk.
+ **Shop title.** The short line under the shop name (Etsy calls it the shop title), front-loaded with what the shop sells so it reads as a search phrase, not a slogan.
+ **Shop announcement.** The greeting at the top of the shop, in the brand voice, one short paragraph.
+ **About section.** The shop story: who makes this, why, what the buyer gets. Written in the brand voice from `brand.md`, honest and specific, not filler.
+ **Shop policies.** Draft shipping, returns/exchanges, and (for POD/digital) a clear "made to order" or "instant download" note. These are legal-adjacent, so draft them plainly and tell the owner to read and adjust before publishing. Not legal advice.

### Honest note: what the Etsy API can set vs what the owner sets by hand

This matters for the flagship, so be straight with the owner. **Verify all of this against Etsy's current v3 API docs before you wire anything.** Etsy changes endpoints, scopes, and what is editable by API, and a newly registered Etsy app may sit in a provisional or pending-approval state before any write works (same caveat as `modules/etsy-autopilot.md`). As a rule of thumb, at the time this was written:

+ Several shop text fields (like the shop title and announcement) are editable through the authenticated `updateShop` write endpoint if the owner has wired the OAuth write scope.
+ Visual and structured pages (the shop icon, the banner, the full About section with its photos and video, and the individual shop policies) are commonly set by hand in Etsy Shop Manager, not reliably through the public API.

So the store builder's job for the shop identity is to **produce every piece as a finished draft (the text ready to paste, the icon and banner as image files)** and hand the owner a short paste-in checklist for whatever Etsy makes them set manually. Where the write API genuinely supports a field and the owner has the write scope wired, an approved identity item can enqueue a follow-up write action, gated on the board like any other publish. Do not claim the API sets something it does not. A wrong claim here is a refund risk.

The `draft_shop_identity` result carries the text in the DB `result` and the icon and banner as files under `board/pending/<task_id>/`, with their paths in `result.preview` and `result.files`, per the storage model. The board shows the rendered banner and icon, not raw JSON, so the owner approves what they can actually see (`07-approval-loops.md`).

## Step 3: the starter product line (on-brand, drafts only)

**Ordering note for a from-scratch build:** Printify can only create draft products inside a shop that is CONNECTED to an Etsy shop, and the API cannot create the Etsy shop. So once the brand (and its shop name) is approved, have the owner do the manual shop-open step from the launch-ready checklist NOW (register the shop at etsy.com/sell with the approved name, then connect Printify to it in Printify's settings), before this step runs. The design files and mockup thinking can be drafted while they do it, but the actual `make_pod_product` calls need the connected shop to exist. If the owner sells digital or handmade goods instead (no Printify), this ordering note does not apply.

The conductor enqueues the product line through `modules/pod-printify.md`. It does not reimplement any of it. For each product idea in the approved plan it enqueues a `make_pod_product` task with the design reference (in the brand's product-design style from `brand.md`) and the product type. The POD agent does exactly what it already does: uploads the transparent PNG, picks the blueprint and provider by title, creates a Printify DRAFT, and pulls mockups back. Every per-product-type rule still applies because it is the same agent: the scale-per-product (mug ~0.52, tote ~0.72 to 0.85), the 100-variant cap, no auto-publish.

Default the line to 5 products. Enough to open a real shop, few enough to review in one sitting. Not 50.

If the owner sells **digital downloads or handmade goods** instead of POD, skip the Printify calls. The product line is then just the listings in Step 4 plus the owner's own files, exactly as `modules/etsy-autopilot.md` describes. The store builder still produces the same branded shop identity and calendar around them.

Every product draft lands in `needs_approval` under the shared `batch_id`. Nothing pushes to Etsy.

## Step 4: the listings (brand voice, one per product)

For each product the conductor enqueues a `draft_listing` task, the listing agent from `modules/etsy-autopilot.md`, and passes it the product plus the brand voice from `brand.md`. The listing agent does what it already does, now on-brand:

+ A title front-loaded with the terms buyers search (Etsy weights the first ~40 characters).
+ 13 tags, each under 20 characters, each a real multi-word search phrase.
+ A description that leads with what the buyer gets, written in the shop's voice, not generic template copy.
+ Any category attributes the product supports.

Pull recent lines from `lessons.md` into the prompt so it stops repeating known mistakes (tags over 20 chars get rejected, and so on). Each draft parks in `needs_approval` under the batch. It does not publish. A published listing needs the OAuth write scope AND an owner who has read the draft.

## Step 5: the launch calendar (a runnable first-week plan)

The conductor writes a plain-text `data/launch-calendar.md` (a readable file, matching how the rest of the system stores memory). It maps the launch: which product goes up which day, and which promo piece ships with it. The promo pieces themselves come from `modules/content-engine.md` when the owner is ready to run them, this step lays out the plan, it does not generate or post anything.

```
# Launch calendar   (store batch 2026-07-14-store-a)

Day 1  Publish shop + "Cozy Season Mug" listing   Promo: pin set A -> mug listing URL
Day 2  Publish "Autumn Tote" listing               Promo: pin set B -> tote listing URL
Day 3  (no new listing)                            Promo: brand-story hook "why this shop exists"
Day 4  Publish "Fall Leaves Tee" listing           Promo: pin set C -> tee listing URL
Day 5  (no new listing)                            Promo: reshare best pin, second board
Day 6  Publish "Warm Drinks Print" listing         Promo: pin set D -> print listing URL
Day 7  (no new listing)                            Promo: seasonal hook + best performer
```

Keep it simple. It is a map for the launch, not a content strategy engine. The owner can edit it in a text editor because it is a text file, and later `modules/manager.md` can read it to enqueue the day's promo automatically.

## Step 6: one review pass, then launch-ready

This is the payoff and the guardrail in the same place. Everything sits on the board under one `batch_id`. The owner does a single review pass over the whole store:

```
$ os board --batch 2026-07-14-store-a
STORE BATCH 2026-07-14-store-a   (brand kit, shop identity, 5 products, 5 listings, calendar)

  Brand
  [70] brand_kit          "3 name options, logo, palette, voice"          approve? (y/n)

  Shop identity
  [71] draft_shop_identity "icon + banner + title + announcement + About + policies"   approve? (y/n)

  Products
  [72] make_pod_product   "Printify draft: Cozy Season Mug, 2 mockups"    approve? (y/n)
  [73] make_pod_product   "Printify draft: Autumn Tote, 2 mockups"        approve? (y/n)
  ...

  Listings
  [77] draft_listing      "Cozy Season Mug | 13 tags | $18 draft"         approve? (y/n)
  ...

  approve-all-listings  approve-all-products  reject <id> <note>
```

Always render the real artifact, never approve from raw JSON (`07-approval-loops.md`). Show the logo and banner images, the Printify mockups, and the listing text. A batch review is not a blind bulk yes, it is the owner moving fast through good drafts and stopping on the ones that need work, all in one sitting, with the real rendered output in front of them.

Reject-with-feedback still works per item: reject a listing or the About section with a note, and that note is both stored in `result.reject_note` and a candidate line for `lessons.md`, and it can enqueue a revision task carrying `owner_feedback` in its payload so the next draft fixes it. Approving a POD product or a listing moves it to its normal follow-up publish action, still one gated click each. Approving the shop identity marks it ready to paste in (or, for the API-writable fields with the write scope wired, enqueues the gated write).

When the batch is approved, the store is **launch-ready.** Hand the owner the launch-ready checklist below.

## Launch-ready checklist (what the owner does after approving the batch)

Give the owner these steps plainly. This is where the drafts become a live shop:

+ **Open the Etsy shop itself, and set up everything that comes with it.** This is a manual owner step and it comes FIRST: the API cannot create a shop. Walk the owner through the whole thing plainly, do not just say "go make an account":
  1. Go to etsy.com/sell and start the shop-open flow. If they do not have an Etsy login yet, they create a free Etsy account first (email and password), then click to open a shop.
  2. Set the shop **language, country, and currency** from `PROFILE.md`. These are hard to change later, so get them right now.
  3. Name the shop with the approved name from the brand kit. Names are first-come: if it is taken, fall back to the next approved option. (This is why the shop opens only after the brand step picks the name.)
  4. Add the **first listing**. Etsy makes you create at least one during setup, so use the first approved listing draft. Each listing costs about $0.20 US to publish.
  5. Set up **how they get paid** (Etsy Payments: their bank account and identity details, so sales money can reach them) and **how they pay Etsy** (a card on file for listing and transaction fees). A shop cannot go live without this, so have bank and card details ready.
  6. Some regions charge a small **one-time shop setup fee** at this point. Tell the owner the number is shown on screen before they confirm, so there are no surprises.
  Until this whole flow is done there is no shop to paste anything into, and no way to get paid.
+ Set the shop **icon** and **banner** in Etsy Shop Manager from the approved image files.
+ Paste the approved **shop title**, **announcement**, **About**, and **policies** into Etsy (or, for the API-writable fields with the write scope wired and approved on the board, let the gated write action set them).
+ Publish the approved **listings** (paste-in, or the approved OAuth-write follow-up task, one gated action each).
+ Confirm the **Printify to Etsy** connection so approved POD products push correctly (`modules/pod-printify.md`).
+ Follow the **launch calendar** for the first week, and turn on `modules/etsy-autopilot.md` to run and grow the store from here.

## Where it goes next

The store builder gets the owner to a launch-ready branded store. Then:

+ **Run and grow:** `modules/etsy-autopilot.md` is the day-to-day layer. Its digest reports the shop's real numbers, its listing agent improves listings over time, its promotion agent drives Pinterest traffic. That is the RUN layer for the store this module BUILT.
+ **Run it on a schedule:** `modules/manager.md` is the once-daily conductor that reads the digest, the board backlog, `lessons.md`, and the launch calendar, then enqueues the day's work and sends a short morning brief. Add it once the store is live and the owner wants the queue fed for them. Advanced, build after the core works.
+ **More products, same brand:** `modules/shop-in-a-box.md` scaffolds another batch of on-brand products into the existing shop in one pass.

## Guardrails (all inherited, none relaxed)

Going bigger does not mean going looser. Every guardrail from the child modules stays exactly as strict:

+ **Nothing publishes automatically.** Every `draft_shop_identity`, `make_pod_product`, and `draft_listing` lands in `needs_approval`. There is no code path in the store builder that publishes a listing, pushes a Printify product live, or writes to the live shop without a click (rule 3). Building a store produces drafts. Live actions are one human click each.
+ **Nothing spends past the cap.** Logo and banner generation and POD draft creation cost credits. The conductor counts every child against the owner's daily spend and task caps from `PROFILE.md` and refuses to enqueue past them. A "whole store in one pass" is exactly the kind of build that could run up a bill, so the batch gets its own hard ceiling: max products, max image generations, max total spend per store build. If the plan exceeds a cap, the conductor shrinks the line and tells the owner why (rule 4).
+ **The batch has its own stop conditions.** Max children per build, max total spend, max runtime. A conductor that enqueues in a loop is a runaway risk, cap it like every loop.
+ **The plan gate protects the biggest spend.** On the first run the conductor parks the PLAN before generating anything, so the owner can redirect before credits burn.
+ **Fresh session per child.** The conductor enqueues, the dispatcher runs each child in its own fresh agent (rule 2). A whole-store build stays as cheap and predictable as its individual pieces, because that is what it is.
+ **No secrets in payloads.** Child tasks read keys from `.env` like always. The conductor passes specs, not credentials, and never logs a token (rule 11).
+ **Honest about the Etsy API.** Do not claim the API sets an identity field it does not. Verify against Etsy's current v3 docs, account for a provisional/pending app state, and hand the owner a paste-in path for anything the API cannot reliably set.

## Prove it works

Do not tell the owner the store is built until you have run it once, small, and shown the output. Run the first real build with the line capped low (say 2 products) to keep the spend tiny:

- [ ] The conductor parked a readable PLAN on the board before spending anything, and the owner could approve or redirect it.
- [ ] The **brand was approved**: a name pick, a logo, a palette, and a voice exist in `brand.md`, and the later steps read from it.
- [ ] The **shop identity is drafted**: icon file, banner file, shop title, announcement, About, and policies, all in the brand voice, all rendered on the board (not raw JSON).
- [ ] **N products are drafted** on-brand (Printify drafts exist as unpublished with mockups returned, or digital/handmade listings are drafted), and none published.
- [ ] **Listings are drafted**, one per product, each valid: title within limits, 13 tags each under 20 characters, a real description in the brand voice, all in `needs_approval`.
- [ ] `data/launch-calendar.md` exists, is readable, and maps the launch to the products actually made.
- [ ] **Everything is grouped under one `batch_id`** and the owner reviewed it as a single "here is your store" pass.
- [ ] Nothing published, wrote to the live shop, posted, or spent past the cap without a click. Try to exceed the batch spend cap on purpose and confirm the conductor refuses and shrinks the line.
- [ ] The owner understands, in plain sentences, that they hold a launch-ready store (drafts), what the launch-ready checklist does next, and that `modules/etsy-autopilot.md` runs it from here.

When those are all true, scale the line up to the real default (5) and let the owner run it for the store they actually want to open. Function first, even here: a launch-ready branded store that is correct and fully gated is the win. The polish that already went into the brand is enough, resist adding more shine before the first listing is live and the owner has seen the shop open.

---

_This module composes other modules, it introduces no new selling channel and no new risk surface, only a branded storefront wrapped around the same gates. All third-party costs from the child modules still apply: the AI image generator meters its own credits, Printify charges per item on a real order, Etsy charges listing and transaction fees, and the AI engine has its usage cost. A built store is drafts and a launch plan, not sales. Selling on Etsy carries no guaranteed outcome and results are not typical, a shop can open and get zero orders. Draft the shop policies plainly and have the owner read and adjust them, this is not legal advice. Keep the owner's spend caps enforced and their expectations honest._
