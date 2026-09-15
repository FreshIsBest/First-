> The flagship build: an Etsy shop that runs on agents. For the AI to build for the owner, function-first, in this exact order. This is the first thing you should build.

Level: beginner-safe | Time: ~2 to 3 evenings | Cost: $0 to read and draft, Printify only charges on a real order, plus your engine

# Etsy autopilot

**Note: this is the RUN-and-GROW layer for a store that already exists.** It gives an existing shop a sales digest, a listing drafter, a POD pipeline, and a traffic engine. If the owner does NOT have a store yet, start with `modules/etsy-store-builder.md` first: that is the flagship build that creates the branded store (identity, starter product line, launch-ready listings), and then you come back here to run and grow it.

This is the money-maker to build first. It is the most proven one, it is the cheapest to run, and it is exactly what I run my own Etsy shop on: a daily sales digest, a print-on-demand pipeline, and a pin generator that drives free traffic. You are building the owner their own version of that.

Do not build a dashboard. Do not pick colors. The goal of this module is a real Etsy shop that reports its own numbers, drafts its own listings, and generates its own traffic, with the owner approving every publish and every dollar. First sale is the win, not a pretty screen. Hold the vanity-trap rule from `CLAUDE.md` the whole way through.

**Timing note.** This module has two halves that come at different points in `03-build-order.md`. The **listing agent** (drafting listings, pricing, tags) is part of the crew you build at Step 3, and in the early steps you run it BY HAND, with the owner approving each draft directly (there is no queue or board yet; that is Step 6). The **run-and-grow autopilot** (the daily digest, the scheduled drafting, the ongoing loop) needs the automated system, so that half is switched on at or after Step 6, once the queue, dispatcher, and board exist. So: build and use the listing agent early by hand; turn on the scheduled autopilot once the system is built. Where this module below says "the board" or "enqueue," that is the Step 6-and-later behavior; before then, the same drafts are approved by hand.

## The agents you are building

Five agents plus the board. Each is one task type, one fresh `claude -p` call, one tight prompt in `prompts/`. Build them in this order. Do not build the next one until the previous one is proven.

| Agent | Task type | Auto-runs? | What it does |
|---|---|---|---|
| DIGEST | `etsy_digest` | Yes (read-only) | Pulls shop stats: sales, views, favorites. Reports deltas. |
| LISTING | `draft_listing` | Yes (draft only) | Drafts title, tags, description, SEO for one product. Parks for approval. |
| PRODUCT / POD | `make_pod_product` | Yes (draft only) | Creates a draft product on Printify. Never publishes. See `modules/pod-printify.md`. |
| PROMOTION | `make_pins` | Yes (files only) | Generates Pinterest pins and post copy to drive traffic. |
| REVIEW / board | (the board) | Human click | Where publish, price change, and ad spend wait for a yes. |

The split that matters: **drafting and generating auto-run. Publishing, spending, and price changes never do.** A draft on disk costs nothing and risks nothing. A published listing or a spent dollar is hard to undo, so it routes through the board every time. Build that line in and never cross it.

## Build order

### Step 1: DIGEST agent (get real numbers first)

Nothing else matters until the owner can see their shop's real numbers coming out of the system. This is the fastest win and it needs no OAuth.

**The key fact:** the public Etsy v3 API returns shop and listing stats (sales count, listing views, favorites, reviews) with just an API key in the `x-api-key` header. No OAuth, no user token, no app review. The owner registers an app at etsy.com/developers, copies the keystring, and that is it for reads.

**Before you wire anything, verify against Etsy's current v3 API docs.** Etsy is a third-party platform and its API endpoints, fields, auth rules, and rate limits change. This is the flagship money-maker, so a wrong first call is a refund risk: confirm the real endpoints and header rules in Etsy's current developer docs before you build against them. Also know that a newly registered Etsy app may sit in a provisional or pending-approval state for a while; the owner may need to wait or request access before the key works. If the first read 401s or 403s, check the app's approval status before you assume the code is wrong.

Get the owner's key into their `.env` (never printed to chat, never committed):

```
ETSY_API_KEY=xxxxxxxxxxxxxxxxxxxx        # the app keystring, this alone is the x-api-key header
ETSY_SHARED_SECRET=yyyyyyyyyyyyyyyy       # only used later in the OAuth write flow, NOT in the read header
ETSY_SHOP_NAME=TheirShopName
```

**The `x-api-key` header is the app keystring ALONE.** Do not combine it with anything. The shared secret belongs to the OAuth flow (used later if the owner wires write access), never to this header. For the read endpoints the digest needs, the keystring by itself is the whole auth:

```
GET /v3/application/shops?shop_name=<name>                  -> resolve shop_id (cache it)
GET /v3/application/shops/<shop_id>                          -> sales count, favorites, reviews
GET /v3/application/shops/<shop_id>/listings/active?limit=100 -> per-listing views + favorites + price
```

A minimal fetch (the whole read layer is this simple):

```js
async function etsy(pathname, key) {
  const res = await fetch(`https://openapi.etsy.com/v3/application${pathname}`, {
    headers: { 'x-api-key': key },
  });
  if (!res.ok) throw new Error(`Etsy ${res.status} on ${pathname}`);
  return res.json();
}
// The x-api-key value is the keystring alone. Never append the shared secret to it.
const key = process.env.ETSY_API_KEY;
```

The digest agent does this, once a day:
1. Fetch the shop and its active listings.
2. Snapshot the numbers that matter: total sales, total views, favorites, reviews, and views per listing.
3. Write the snapshot to a dated history file so tomorrow's run can diff against today.
4. Compute deltas versus the previous snapshot (sales +2, views +140 since last run).
5. Write `data/digest-latest.md` and print the headline (Sales, Views, and what needs the owner).

That last part is the whole point. The digest is not a data dump, it is a report the owner reads in ten seconds: what sold, what got traffic, and what is waiting on them. On Windows, a best-effort toast with the headline is a nice touch, but never let a failed toast break the run.

Schedule it. On Windows, a Scheduled Task at 8 AM. On Mac or Linux, a cron line. The digest is the one thing that runs on a clock instead of through the queue, because it is pure read-only reporting and it is safe to auto-run forever.

**Prove it:** run the digest by hand. It must print the owner's actual sales count and view numbers pulled live from Etsy. If it prints real numbers, step 1 is done. Tell the owner in one plain sentence what they are looking at, then move on.

### Step 2: LISTING agent (draft one better listing)

Now improve one listing. Pick the product with the most views but no sales, or the owner's newest product. That is where a better title and tags move the needle fastest.

The listing agent takes one product and drafts:
- A **title** front-loaded with the terms buyers actually search (Etsy weights the first 40 characters most).
- **13 tags**, each under 20 characters (Etsy rejects longer), each a real multi-word search phrase, no single throwaway words.
- A **description** that leads with what the buyer gets and why, not a wall of policy text.
- Any **attributes** the category supports (occasion, color, material).

Feed the agent the owner's existing listings (so it matches their voice and does not duplicate tags) and the current digest (so it knows what is already getting traffic). **If a `brand.md` exists (from `modules/brand-kit.md`), read it and write in that brand voice**, so the store's identity survives into the run-and-grow phase instead of drifting back to generic copy. Pull recent lines from `lessons.md` into the prompt so it stops repeating known mistakes (for example: "tags over 20 chars get rejected").

The agent writes the draft as JSON into the task's `result` column and sets the task to `needs_approval`; any large artifact (a rendered preview, an image) goes as a file under `board/pending/<task_id>/` with its path in `result.preview`, per the storage model in `02-architecture.md`. **It does not publish.** Publishing to a live shop needs the Etsy OAuth write scope and, more importantly, an owner who has read the draft. The draft costs nothing sitting on the board. A bad live listing costs the shop's search ranking.

**Prove it:** the agent produces a complete, valid draft (title under the limit, 13 tags all under 20 chars, a real description) parked on the board. The owner reads it, and either approves it to paste into Etsy themselves, or approves an OAuth-write follow-up task if they have wired write access. Either way, a human said yes before anything went live.

### Step 3: PRODUCT / POD agent (create sellable product)

If the owner sells print-on-demand (mugs, tees, posters, totes), this agent creates draft products on Printify from a design file. This is its own build with its own API. Do not inline it here. Follow `modules/pod-printify.md` for the full pipeline: design in, transparent PNG, blueprint and print-provider selection, draft product created, mockups pulled back for review.

The same guardrail applies: the POD agent creates the product as a **draft**. It never publishes to the connected Etsy shop on its own. Publish is a board decision.

If the owner sells digital downloads or handmade goods instead, skip this step. The listing agent plus their own upload is enough to get to a first sale.

### Step 4: PROMOTION agent (drive the traffic)

Here is the reality nobody tells you: **a new Etsy listing gets close to zero traffic on its own.** Etsy shows established, proven listings first. A brand-new shop with no sales sits on page 40. If you build the perfect listing and stop, nothing happens. The system has to bring its own traffic.

The proven free channel, and the one I actually run, is **Pinterest.** Pinterest is a search engine that sends buyers straight to an Etsy URL, it rewards fresh pins, and it costs nothing. That is why the promotion agent generates pins.

The promotion agent, per listing, produces:
- **3 distinct vertical pins** (2:3 ratio, 1000x1500 or 2000x3000) built from the listing's own images plus a hook headline, each a different layout so they do not read as spam.
- A **pins.md** with a Pinterest-ready title and description, suggested board, keyword tags, and the **Etsy destination URL** to paste into every pin.

Rendering is free and dependency-light: build the pin as an HTML template in the shop's look, then screenshot it with headless Chrome. No paid image credits needed for this. (My real generator does exactly this, three templates per listing, and it never lays a hook over the listing's own text-heavy graphic.)

Posting is where the guardrail sits. Two honest options, tell the owner both:
- **Manual (default, zero risk):** the agent drops the pin files and the copy. The owner posts them to Pinterest themselves, a couple minutes a day. Three pins to one Etsy URL, spaced a day or two apart, is Pinterest best practice, not spam.
- **Automated posting:** if the owner wants pins (and other content) pushed to Pinterest and more platforms automatically, wire the Zernio section of `tools/tools.md`, which ships one asset to many platforms through one API and has an MCP server the agents can call. Even then, treat the first sends as board-approved until the owner trusts the output.

For turning pins into short video, or generating fresh promo content beyond static pins, see `modules/content-engine.md`. Do not build that until the static pin loop is proven and posting.

**Prove it:** the agent generates three real pin images and valid post copy with the correct Etsy URL, and the owner can post one. Traffic is the deliverable, not the pin file.

### Step 5: The board and the loop

Wire the Etsy task types into the board you already built. The board lists everything in `needs_approval` and lets the owner approve or reject:

```
$ os board
[41] draft_listing  -> "Cozy Fall Mug | 13 tags | $18 draft"     approve? (y/n)
[42] make_pod_product   -> "Printify draft: Autumn Tote, 2 mockups"   approve? (y/n)
[43] make_pins      -> "3 pins ready for listing 1841..."         approve? (y/n)
```

Approve on a listing or product moves it to a follow-up publish task (or the owner does the publish by hand). Approve on pins marks them ready to post. Reject marks it rejected with a note, and that note is a candidate line for `lessons.md`.

Once all five run, the daily rhythm is: the digest reports at 8 AM, the owner glances at the board, approves what looks good, posts the pins. The system does the drafting and generating overnight. That is the autopilot.

## Guardrails specific to Etsy (enforce in code)

- **No auto-publish, ever.** `draft_listing` and `make_pod_product` always land in `needs_approval`. There is no code path that publishes a live listing without a human click. This is the single most important rule in this module.
- **No auto price changes.** Changing a live listing's price routes through the board like a publish. An agent must never reprice the shop on its own.
- **Spend cap on any paid action.** Etsy Ads, promoted pins, or any paid boost has a hard daily ceiling from the interview (`PROFILE.md`). Track spend in a ledger and refuse the action past the cap. Default to zero paid ads until the shop has organic sales. Free traffic first.
- **Read-only stays read-only.** The digest uses only the public read key. It has no ability to write, publish, or spend. Give each agent the least power that does its job (dispatcher tool whitelist), and the digest gets almost none.
- **Rate-limit the reads.** Etsy caps API calls. Cache the `shop_id`, batch listing reads, and do not hammer the endpoints in a loop.

## Prove-it-works checklist (run before calling the Etsy build done)

Do not tell the owner "the Etsy autopilot is done" until every one of these is true and you have shown the output:

- [ ] The digest, run live, prints the owner's real sales count and view numbers from Etsy.
- [ ] A second digest run shows correct deltas against the first (numbers changed, or explicitly show plus or minus zero).
- [ ] The listing agent produced one complete, valid draft: title within limits, 13 tags each under 20 characters, a real description.
- [ ] That draft parked in `needs_approval` and did NOT publish anything on its own.
- [ ] If POD applies: a draft product exists in Printify and its mockups came back for review (per `modules/pod-printify.md`), and it did not publish.
- [ ] The promotion agent generated 3 real pin images plus post copy carrying the correct Etsy destination URL.
- [ ] The board shows the `needs_approval` items and the owner can approve or reject each.
- [ ] Attempting a publish, a price change, or any paid spend routes through the board, not around it.
- [ ] Spend caps from `PROFILE.md` are enforced in code (test one by trying to exceed it and confirming it refuses).
- [ ] The owner has been told, in plain sentences, what runs automatically and what waits for them.

When all boxes are checked, the Etsy autopilot is real. Only after the shop is actually reporting, drafting, and generating traffic should you talk to the owner about making any of it look nicer. Function first. First sale is the goal.

---

_Selling on Etsy carries no guaranteed outcome and results are not typical; a shop can get zero sales. Third-party services cost money: Printify charges per product ordered, and the AI engine (Claude Code, OpenRouter, or similar) has its own usage cost. Keep the owner's spend caps enforced and their expectations honest._
