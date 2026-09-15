> For the buyer's AI. Build the owner a resale sourcing agent: it reads marketplace listings a fetcher already collected, prices each against what similar items ACTUALLY SOLD for, and drops a short ranked list of real buy opportunities for the owner to approve. It never buys, never messages a seller, never scrapes on its own. Analyst, not buyer. Lead with sanctioned data sources; treat local-marketplace scraping as advanced and at the owner's risk.

Level: intermediate | Time: ~1 to 2 evenings | Cost: $0 (public data and a free eBay developer key; no paid tools required)

# Module: flipping (the scout who prices deals)

This is the money-maker that has no store and no products of its own. The owner makes money by buying something underpriced and reselling it at what the market actually pays. The agent's job is the hard part of that: finding the underpriced thing and proving the spread is real before a dollar moves.

I run this myself. An agent I call the scout pulls local resale listings, prices each one against real sold data, and hands me a short ranked list of what is actually worth buying. It never buys anything. I look at its list, check the photos, and decide. That division of labor is the whole design: the agent does the tireless pricing math, the human makes every purchase.

Understand up front that this is a different shape from the Etsy flagship. There is no branded storefront to build. It reuses the same system (the queue, the agents, the board, the memory), just pointed at a different job: sourcing and arbitrage instead of listing creation. If the owner picked this as their money-maker, build it the same function-first way, and the two rules that never bend still apply: nothing gets bought without the owner's yes, and a spend cap is in code before any tool that could spend.

## The one idea the whole thing rests on

**A resale opportunity needs two numbers, and most people only look at one.** The listing price (what someone is asking) means nothing on its own. What matters is the gap between that asking price and what the item ACTUALLY SOLD for, after the costs of reselling it. So the agent always works with two sides:

- **Buy candidates:** live listings the owner could actually buy right now, ideally from a cheaper local source.
- **Sold comps:** what that same item recently SOLD for, not what it is listed for. This is the pricing truth.

Without both, there is no deal to find. This is the single most important thing to get right, and it is where naive versions fail: they compare an asking price to other asking prices and call a normal item a bargain.

## The hard rules that make the pricing honest

Bake these into the agent's prompt. They are the difference between a useful list and a confident-but-wrong one.

- **Comp against SOLD listings only, never active ones.** An asking price is a hope, not a value. Comparing one ask to another ask is worthless. On eBay, that means completed-and-sold listings, not current ones.
- **Use the median sold price, not the highest.** The top sold result is usually a bundle or an outlier, and it will talk the owner into a bad buy. The middle of the recent sold prices is the honest number.
- **No comp means no recommendation.** If the agent cannot find a real sold comp for an item, it says "no comp" and ranks it last. It never invents a price to fill the gap. A guess here costs the owner real money.
- **The spread must survive the real costs.** Asking price against the median comp, minus the selling platform's fees (roughly 13 percent on eBay), minus shipping (which is brutal on heavy or fragile items), minus the owner's time and travel to pick it up. A spread that only works if everything goes perfectly is not a spread.
- **The agent is an analyst, not a buyer.** It reads, it prices, it ranks, it drops the list. It never buys, bids, messages a seller, or places an order. Give it the tightest tool whitelist in the system: read and write only. It has no reason to hold a spend or send tool, ever.

## Where the data comes from (sanctioned first)

The comp side is easy and clean. The buy side is where the judgment is, and the sources run from safe to risky. Lead the owner toward the safe ones.

### Comps: eBay's official API (free, sanctioned, reliable)

eBay publishes sold-and-completed prices, and its official developer API returns them with a free key. This is the backbone. Have the owner register a free eBay developer app, put the key in `.env` (never in chat), and the agent's fetcher pulls real sold comps for any search. It is stable, it is allowed, and it does not risk anything. Verify the current eBay API docs before wiring, because their endpoints and rules change (same posture as every third-party tool in this Blueprint).

### Buy side, safest first

1. **eBay auctions ending soon with no bids.** A real edge that needs no new source and no gray-area anything: auction-format listings closing within the hour, sitting under their own sold comp. Same official API.
2. **Auction sites that want bidders.** Online-thrift and government-surplus sites (the kind whose whole business is attracting buyers) are far friendlier to automated reading than a marketplace that does not want it. Genuinely below-retail inventory, and no hostile terms to fight.
3. **Local marketplaces (Facebook Marketplace, Craigslist, OfferUp), advanced and at the owner's own risk.** This is where the best local deals are, and it is also where the rules get real. Read the next section before you build any of it.

## Local marketplaces: the honest warning (read before building)

The best flipping inventory is local, and local marketplaces mostly do not want to be read by software. Be straight with the owner about all of this; do not quietly wire a scraper.

- **It usually breaks the platform's terms.** Automated reading of Facebook Marketplace, Craigslist, and similar is against their terms of service. The owner is responsible for that choice, not the software. Tell them plainly and point them to the platform's current terms (`08-gotchas.md` compliance posture).
- **Never scrape from a logged-in account you cannot afford to lose.** Platforms that ban by identity (Meta especially) can take down the account you use, and if that account also administers your ads or your business page, you can lose those too. If the owner runs ads on the same identity, this risk is severe. The only version worth considering reads PUBLIC listings with NO login and NO saved session, so there is no account to ban. Do not add a login, a cookie jar, or a saved session to a marketplace fetcher.
- **Craigslist specifically is a dead end right now.** Its old machine-readable feed returns a hard block, and its search page loads results with JavaScript so a simple read gets nothing. Do not build on it, and do not spend the owner's evening trying.
- **These are the fragile part of the system.** A marketplace fetcher breaks whenever the site changes, and it always eventually changes. Treat it as a rental, not a foundation. The eBay API half keeps working; the local-scrape half needs re-checking.

The honest recommendation for a beginner: start on the sanctioned sources (eBay sold-vs-auction, the auction sites). Add a no-login local fetcher only if the owner understands and accepts the tradeoffs above.

## Build it: a fetcher and an agent

Same two-part shape as the research scout (`modules/research.md`): a cheap non-AI fetcher collects listings, and the agent does the thinking.

1. **The fetcher (no AI).** A small script that pulls buy candidates and sold comps, dedupes against what it has already seen, and drops the batch into a file the agent reads. Crucially, **the comp searches must cover the same categories as the buy searches.** If the fetcher pulls local laptops but only fetches comps for one model, most candidates come back unpriceable. Match the two sides or the agent has nothing to score.
2. **The agent (the scout).** Fired on a `triage` task, it reads the batch, groups listings by the real thing that sets the price (the exact model and spec, not the wording of the title), prices each buy candidate against the sold comps, applies the hard rules above, and writes a ranked file: what to buy, the comp it is priced against, the spread after costs, and the single biggest risk on each. Then it drops that list for the owner. It appends what it learned to the lessons file so it gets sharper over time.

**Working means:** the owner gets a short, honest ranked list of real buy candidates, each with a named sold comp and a spread that already accounts for fees and shipping, and nothing on it was bought.

**Prove it:** run the fetcher, fire one triage task, and show the owner the actual ranked file the agent wrote. If it found zero worth buying, that is a real and correct answer, not a failure. Show that too.

## The traps that will burn the owner (put these in the agent's prompt)

Every one of these is a real way to lose money on a flip, and the agent should flag them, not ignore them:

- **No proof it works.** "For parts" or "untested" means the item is worth its parts, not its comp. Treat anything not clearly shown working as a parts-price gamble.
- **Locks and bundles.** An account-locked device is near worthless. A bundle photo often hides a working item plus a box of junk; price only the confirmed working piece.
- **Distance eats the spread.** An hour each way is two hours of the owner's day. A thin margin an hour away is a loss dressed as a deal.
- **Shipping kills heavy and fragile items.** A small light item ships cheap; a big or breakable one does not, and insured shipping can erase the whole spread.
- **Old models look like deals.** A last-generation item at a low price is not a find if the comps are just as low. Check the generation before getting excited.
- **Stale listings are priced wrong, not waiting for you.** An item that has sat for weeks is overpriced; the recent, fairly priced ones are the real finds.

## The approval flow (how the owner actually uses it)

The agent's ranked list is a set of recommendations, and every one needs the owner's eyes and click, because the final check (are the photos real, does it truly look working) is human judgment the agent cannot do. Build the owner a simple approve or reject pass over the list, one candidate at a time:

- **Approve** shortlists it for the owner to go act on themselves. It never triggers a purchase; the system does not buy.
- **Reject** drops it and, better, writes a line to the lessons file so the agent learns what the owner turns down and stops resurfacing it. Rejections make the next run smarter.

Showing the listing photo next to each candidate is worth the effort: the owner confirms the model and condition without leaving the list.

## The one honest thing to tell the owner

Flipping is not passive, and a single run showing nothing is not a broken system. Good deals appear and get taken within hours, so the value is in running the fetcher on a schedule (a couple of times a day) and checking a fresh list each time, not in one lucky snapshot. The agent removes the tedious pricing work so the owner only looks at the handful of items actually worth their time. That is the win: not magic money, but their attention spent only on real opportunities.

## The sell side: relisting what they bought (the flip is not done until it sells)

Sourcing is the hard, novel half and it is most of this module. But a flip only counts when the item resells, so the owner needs a repeatable way to turn a bought item back into a listing. Build this second, after the sourcing agent is proving out real buys, and keep it deliberately simple. It is the same crew shape as the Etsy listing agent, pointed at used goods instead of new products.

**What the sell side needs, in order:**

1. **A `draft_resale_listing` agent.** Same template as `04-first-agent.md`: tight prompt, read/write tools only, timeout, output the owner approves. It takes one item the owner bought (make, model, condition, what they paid) and drafts the resale listing: a title with the exact model and key spec, an honest condition description, and a suggested list price. It writes text and images-to-shoot notes, it never posts.
2. **Price it off the SAME sold comps, not a hope.** The agent already knows how to pull sold comps from the sourcing side. Reuse that: list at or just under the median recent SOLD price, not the highest and not the current asking prices. The item was a deal because the owner paid under the sold comp; the resale price is that comp, minus a little to move it faster. Pricing at the top comp is how items sit unsold for months.
3. **Say which platform and why.** The agent recommends where to relist based on the item, and states the tradeoff plainly so the owner picks:
   - **eBay** for anything shippable and searchable (electronics, tools, collectibles). National buyer pool, real sold-price history, but ~13 percent fees and you pack and ship. The fee and shipping already came out of the spread at sourcing, so this is the honest default for most flips.
   - **Local (Facebook Marketplace, OfferUp) for heavy, bulky, or fragile items** where shipping would eat the margin. Cash, no fees, no packing, but a smaller local pool and slower. Same account-safety warning as the buy side applies: do not automate posting from an account the owner cannot afford to lose. Relisting is a hand action the owner does; the agent drafts, the owner posts.
4. **The board gate still holds.** A resale listing is a public post under the owner's name, so it parks on the board exactly like any other risky action. The owner reads the draft, checks the price against the comp, and approves before anything goes live. The agent never posts on its own (rule 3).

**The costs the resale price must already cover** (the agent states these on every draft so the owner sees the real take-home): the platform fee (~13 percent on eBay), shipping if it ships, and the original buy price. Take-home is list price minus all three. If that number is not clearly above what the owner paid, the item was not a flip, and the agent should say so rather than dress up a break-even as a win.

**One honest line to tell the owner:** the money is not made when they buy, it is made when it sells. An item sitting unsold in the garage is capital frozen, not profit. Price to move (median comp, not top comp), and treat a fast sale at a fair spread as better than a slow one chasing the last few dollars.

**Prove it:** draft one real resale listing for something the owner actually bought, show the title, the condition text, the price, the named comp it is priced against, and the take-home after fees and shipping. Do not call the sell side done until the owner has approved and posted one real listing and it is live.

## Where this plugs into the build

- **As a first money-maker** (interview Section C): build it the same function-first way as any other. Brand kit does not apply (there is no storefront); start with the sourcing agent and the approval pass instead. The spend cap still goes in at Step 2 before any tool that could spend. Build the sourcing side first (Steps 3 to 4), and add the sell-side `draft_resale_listing` agent once the owner has actually bought something to relist (their first real win, Step 4 to 5).
- **As a second money-maker** after an Etsy win (`09-after-first-win.md`): it reuses the whole system, just pointed at arbitrage. A good fit for an owner who wants a second income stream that is not another store. The sell side is nearly free here: the Etsy listing agent already exists, so the resale agent is a copy of it with a used-goods prompt.
