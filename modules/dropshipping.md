> For the buyer's AI. Build a dropshipping store run by agents: a scout finds products, a listing agent writes pages, a content agent drives traffic, a fulfillment-watch agent keeps orders honest. Function-first. Nothing spends, orders, or publishes without a human click.

Level: advanced | Time: ~3 to 4 evenings | Cost: real ad-test and supplier spend at risk (set hard caps), plus your engine

# Dropshipping on agents

This module builds a dropshipping store where agents do the grunt work: finding products, writing listings, making the videos that bring traffic, and watching orders after the sale. The owner still approves every move that costs money or goes public.

Read `PROFILE.md` first. Build from the owner's real answers (their store platform, their niche, their budget, their caps). Do not build a generic store.

## Read this to the owner before you build (the honest part)

Dropshipping is not a get-rich button. It lives or dies on two things: traffic and margins. Say this plainly:

- **Margins are thin.** You buy from a supplier and resell higher. After the supplier cost, the payment fees, and the cost of getting a visitor to the page, the profit per order is small. One product that does not sell still cost you the time and the ad test to find that out.
- **Traffic is the whole game.** A perfect store with no visitors makes zero dollars. Most of the work here is the content engine bringing people to the page, not the store itself. If the owner is not willing to feed the content machine, this money-maker is the wrong pick, and Etsy or print-on-demand is safer.
- **This is testing, not magic.** You will test many products and most will flop. The agents make testing cheap and fast. They do not make a bad product sell. The win is running more tests, faster, than a person could alone.

If the owner understands that and still wants it, build. If they wanted a passive money printer, point them back to `modules/etsy-autopilot.md` and `modules/pod-printify.md` first. I run my own Etsy shop for the safe, proven income; a dropship store is the higher-risk, higher-effort play on top of it, not a replacement.

Results are not typical. Nobody is owed a sale.

## What you are building

A store on **Shopify** (the concrete path this module builds against) fed by four agents wired to the same queue, dispatcher, and board you already built. The agents produce product picks, listing copy, and content; the store is where an approved listing gets pushed as a **draft product** through Shopify's Admin API. Everything the agents output stays clean and portable (a title, a description, bullets, a price, an image), so if the owner is on a different host you swap only the one publish step. But do not leave "the store" as a vague idea. Below is exactly how a listing reaches the store, end to end, on Shopify. If the owner is not on Shopify, use the CSV path in the same section.

New task types this module adds to the queue (from `02-architecture.md`):

- `research_product`: the scout finds a candidate.
- `draft_listing`: the listing agent writes the product page.
- `make_content`: the content agent produces a video/post to drive traffic (this is the content engine, see below).
- `watch_orders`: the fulfillment-watch agent checks open orders and flags problems.
- `publish_product`: the board-triggered follow-up that pushes an approved listing to the store as a draft.

## The four agents

Each one is the same shape from `04-first-agent.md`: a tight prompt, a whitelisted tool set, a hard timeout, output written where the board can see it, fresh session per run. None of them spend, order, or publish on their own.

### 1. The SCOUT (product research)

Job: find one candidate product worth testing, and write up why. One product per run. Not a list of fifty.

It reads the owner's niche and constraints from `PROFILE.md`, looks at what the owner told it to watch (supplier catalogs, trend sources, the owner's own idea list), and returns a short structured pick: what the product is, the supplier cost, a realistic sell price, the rough margin after fees, why it might sell, and the obvious risk (saturated, fragile, slow shipping, seasonal).

- **Tools:** read, and web/fetch if the owner wants it pulling from live sources. No spending. No ordering.
- **Output:** a `needs_approval` card on the board. The scout proposes. The owner decides what gets tested. Finding a product is not the same as committing money to it.
- **Stop condition:** one candidate, or "nothing worth testing today" if the sources are thin. Do not pad the pile with junk to look busy.

The scout never places a sample order or buys anything. Sourcing a physical sample, if the owner wants one, is a spend and goes through the board like every other spend.

Example scout output card:

```json
{
  "type": "research_product",
  "product": "collapsible silicone travel bowl for dogs",
  "supplier_cost": 3.10,
  "suggested_price": 16.99,
  "est_margin_after_fees": 9.20,
  "why": "portable pet gear trends up in summer, light to ship, easy demo video",
  "risk": "some saturation on TikTok, check the angle before spending",
  "status": "needs_approval"
}
```

### 2. The LISTING agent (writes the page)

Job: turn one approved product into a finished product page. Title, description, bullet features, an honest shipping-time line, and the copy for the store. One product per run.

This is the same pattern as the Etsy listing work. It writes warm, plain, honest copy. No fake scarcity, no invented reviews, no "doctors hate this." Honest shipping expectations especially: dropshipping ships slow, and hiding that buys chargebacks and angry buyers. Say the real delivery window.

- **Tools:** read, write. No network, no spend.
- **Output:** the product object written as JSON into the task's `result` column, in the exact shape the publish step needs. Any large artifact (the hero image) is written as a file under `board/pending/<task_id>/` with its path stored in `result.preview`. **Publishing to the store is a separate approved step.** The agent produces the draft; the owner approves; only then does the `publish_product` follow-up push it live. Nothing publishes on its own. This is rule 3 in `CLAUDE.md`.

The listing agent writes this shape, portable enough to push to Shopify or drop into a CSV:

```json
{
  "type": "draft_listing",
  "title": "Collapsible Travel Bowl for Dogs",
  "body_html": "<p>Fold it flat, clip it to the leash, unfold it at the trail...</p><ul><li>Food-grade silicone</li><li>Holds 12oz</li><li>Dishwasher safe</li></ul><p><strong>Ships in 10 to 18 days.</strong> Ordered directly from our maker, so it takes a little longer and costs a little less.</p>",
  "price": "16.99",
  "sku": "DOGBOWL-TEAL-01",
  "image_url": "board/pending/88/dogbowl-hero.png",
  "tags": ["dog gear", "travel", "hiking with dogs"],
  "status": "needs_approval"
}
```

### How a listing actually reaches the store (the concrete path)

This is the part most guides leave vague. Here is exactly how an approved listing becomes a real product, on Shopify, as a DRAFT the owner can eyeball in their admin before it ever sells.

**Setup (once).** The owner creates a custom app in their Shopify admin (**Settings > Apps and sales channels > Develop apps**), grants it the `write_products` scope, and installs it. Shopify hands back an Admin API access token. It goes in `.env`, never in the chat, never committed:

```
SHOPIFY_STORE=the-owners-store           # the *.myshopify.com subdomain
SHOPIFY_ADMIN_TOKEN=shpat_xxxxxxxxxxxxxxxx
```

**The publish step (`publish_product`).** This task runs ONLY after the owner approves the listing card on the board. It reads the approved product object and makes one call to the Shopify Admin REST API. The single detail that keeps it safe: `"status": "draft"`. A draft product exists in the store but is not visible to buyers and cannot be purchased until the owner flips it to active in the admin. The agent never sets it active.

```bash
# Runs only from an approved board item. status:draft is the guardrail.
curl -X POST "https://$SHOPIFY_STORE.myshopify.com/admin/api/2024-10/products.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "product": {
      "title": "Collapsible Travel Bowl for Dogs",
      "body_html": "<p>Fold it flat, clip it to the leash...</p><p><strong>Ships in 10 to 18 days.</strong></p>",
      "vendor": "the-owners-store",
      "status": "draft",
      "tags": "dog gear, travel, hiking with dogs",
      "variants": [ { "price": "16.99", "sku": "DOGBOWL-TEAL-01" } ],
      "images": [ { "src": "https://.../dogbowl-hero.png" } ]
    }
  }'
```

Shopify returns the created product with an `id` and an `admin_graphql_api_id`. Write that back to the board item so the owner has a direct link to the draft in their admin. The owner opens it, checks the photos and price, and clicks "make active" in Shopify themselves. Going live is a human action in the store, on top of the board approval. Two gates, on purpose.

Note the image: `images.src` must be a public URL Shopify can fetch, or you base64 the file into an `attachment` field instead (Shopify supports both, check the current Admin API version in the URL, `2024-10` above is an example). Do not build against a remembered API version; confirm the current one in Shopify's docs and match it.

**If the owner is not on Shopify: the CSV path.** Every major store host (Shopify, WooCommerce, BigCommerce, Etsy's own bulk tools) imports products from a CSV. The publish step writes the approved product as one row to `board/approved/products-import.csv` with the host's expected column headers, and the owner uploads it in their admin. Shopify's product CSV headers, for example: `Handle, Title, Body (HTML), Vendor, Tags, Published, Variant SKU, Variant Price, Image Src, Status`. Set `Status` to `draft` and `Published` to `FALSE` so the same "never live without a human" rule holds. The agent builds the file; the owner does the upload and the go-live. Pick ONE of these two paths per owner, do not build both.

### 3. The CONTENT agent (drives the traffic)

Job: make the short video or post that brings people to the product. This is the single most important agent in the store, because traffic is the whole game.

**Do not rebuild this here.** It is the same content engine from `modules/content-engine.md`. Reuse it. The only difference is the input: the product and its selling angle instead of a general topic. Feed the approved product's title and hook into the `draft_post` payload as the `topic`, and the content engine's CONTENT agent writes the hook, caption, hashtags, and visual brief exactly as that module specifies. It produces the video the same way (the video tools in the Higgsfield section of `tools/tools.md`, and the owner's 3D world from the Meshy section of `tools/tools.md` if they built one for their brand look).

Posting is the same too. The content ships through the Zernio section of `tools/tools.md`: one approved video goes to all the platforms at once. And posting is a publish, so it routes through the board: the agent produces the video and the caption, parks it on the board, and nothing goes public until the owner approves. Reuse the posting tool, reuse the approval gate. Do not build a second one.

- **Tools:** whatever the content engine already uses (read, write, the video and posting tooling). No autonomous posting.
- **Output:** a finished video plus caption, `needs_approval`, then out through Zernio on the owner's click.

Point the owner at `modules/content-engine.md` and the Zernio section of `tools/tools.md` and tell them plainly: the store is the easy part, the content is the work, and this agent is where the sales actually come from.

### 4. The FULFILLMENT-WATCH agent (after the sale)

Job: watch open orders and flag anything wrong. It does not fulfill orders on its own and it does not move money. It watches and reports.

Every run it reads the current open orders (from the Shopify Orders API with a read-only token, the store's CSV export, or a file the owner drops in) and checks for the things that turn into refunds and chargebacks:

- orders paid but not yet placed with the supplier past a set window
- orders with no tracking number after N days
- tracking that has not moved in N days (stuck in transit)
- supplier price jumped above the owner's margin floor (the product is now selling at a loss)
- a spike in the same complaint across orders

It writes a short daily digest of what needs attention and parks anything requiring an action (issue a refund, message a buyer, cancel an order) on the board as `needs_approval`.

- **Tools:** read, and fetch if it pulls from the store API. No spend, no refund power, no messaging on its own.
- **Output:** a plain digest plus board cards for anything actionable. Placing the actual supplier order, refunding, or messaging a buyer is an approved action, never automatic.

Actually placing the supplier order when a sale comes in is the highest-risk step in the whole store, because it spends real money on every order. Default it to the board: a sale creates a `needs_approval` "place supplier order" card, the owner approves, then a follow-up task (or the owner, by hand) places it. Only once the owner has watched this run clean for a while and explicitly asks should you discuss auto-placing orders, and even then it stays under a hard per-day spend cap. Build the guardrail before the action.

## What auto-runs vs what waits for the board

Auto-runs (safe, produces text or a draft, reversible):

- the scout finding and writing up product candidates
- the listing agent drafting page copy
- the content agent generating videos and captions
- the fulfillment-watch agent reading orders and writing the digest

Waits for a human click on the board (spends money, goes public, or is hard to undo):

- **spending on ads** to push a product
- **placing a supplier order** for a sale
- **publishing a product** to the store (even as a draft, it is the board that triggers the `publish_product` call, and going active is a second human step in the admin)
- **posting content** to the platforms
- issuing a refund or messaging a buyer

That split is the safety of the whole thing. The agents do all the producing and watching. The owner keeps the finger on anything that costs money or is public. Do not blur that line to make the store feel more "automatic." It is not supposed to be.

## Guardrails specific to this module

On top of the standard caps from `02-architecture.md`:

- **A margin floor.** Set a minimum acceptable profit per order in `PROFILE.md`. The scout rejects anything under it. The fulfillment-watch agent flags any product whose supplier price rose above it. Never sell at a loss silently.
- **A hard ad-spend cap per day.** Ads are the fastest way to lose money here. Cap daily spend in code, route every spend through the board, and make the fulfillment-watch digest report spend against the cap so the owner sees the burn.
- **A product-test budget.** Decide up front how much the owner will risk testing one product before killing it (for example, an ad-spend ceiling and a days-live limit). When a test hits the ceiling with no sales, the agent flags it "kill this one" instead of quietly spending more.
- **Draft, never auto-active.** The `publish_product` step always creates the product as `draft` (or `Status: draft` in the CSV). No code path sets a product active. The owner flips it live in the store by hand. (Deliberately stricter than `modules/shopify-store.md`, whose `shopify_publish` sets approved items active: that is a one-time launch event the owner watches happen, while dropshipping is an ongoing catalog where products arrive continuously, so the manual flip is a second gate that keeps a busy pipeline honest.)
- **Honest shipping copy, enforced.** The listing agent must state the real delivery window. Add a lesson to `lessons.md` the first time a slow-shipping complaint shows up, and feed it back into the listing prompt.

## Prove it works

Do not tell the owner the store runs. Show them. Run the scout for real and show the candidate card. Run the listing agent on an approved product, then run `publish_product` and show the actual DRAFT product sitting in their Shopify admin (or the CSV row ready to import), not live, not buyable. Run the content agent and show the actual video plus where it would post (held on the board, not sent). Run fulfillment-watch against a sample order file and show the digest. Then say, in plain words: "This finds products, writes the pages, pushes them as drafts you flip live yourself, makes the traffic videos, and watches your orders. It never spends, orders, posts, or goes live without you. Your job is to approve the good ones and feed the content machine, because traffic is what makes the sales."

Then get back to whichever money-maker earns first. If the content is not going out and the traffic is not coming, the store is a parked car. Say so.
