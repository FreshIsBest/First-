> For the buyer's AI. The owner does not run a store, they run a MARKETPLACE: other vendors sell on their platform, pay them a fee, and submit products the owner has to approve. Dokan on WooCommerce, Sharetribe, Marketplace on Shopify, or a custom build. Different business, different agents. The money here is vendor count and vendor retention, not product count, and the daily grind is reviewing a queue of other people's work.

Level: intermediate | Time: ~2 to 3 evenings | Cost: $0 beyond the owner's existing platform and your engine

# Marketplace (you host other people's stores)

## Read this first: a marketplace is not a store

If the owner sells their own products, they are in `modules/any-store.md` or one of the store modules. Send them there.

This file is for an owner whose customers are **vendors**. That changes everything about which agents matter:

| A store owner is short on | A marketplace owner is short on |
| --- | --- |
| products to sell | hours to review other people's products |
| traffic to their listings | vendors paying to be on the platform |
| time writing listings | time answering vendor questions |

So do not start by building a listing generator. The owner does not need more products, their vendors make those. Build for the two things that actually run a marketplace: **the approval queue** (their time) and **vendor growth** (their revenue).

Say that to the owner out loud before building, because they will often ask for the store modules by name and be wrong about what will help them.

## The money model, so you build for the right number

Most marketplaces earn from some mix of:

- a **recurring vendor fee** (the reliable one, and usually the one to optimize)
- a **commission** per sale
- listing or featured-placement fees

Ask which, and write it into `PROFILE.md`. It decides what the reports track. If the owner earns a monthly vendor fee, then **vendors onboarded and vendors retained** is the business, and a dashboard bragging about total product count is measuring the wrong thing.

## Step 1: connect, using the same adapter idea

Start from `modules/any-store.md`. That module's rule holds: one adapter file is the only platform-specific code. A marketplace adapter just has more surface, because you also read vendors and the pending queue.

```javascript
// store/adapter.mjs, marketplace flavour. Confirm every endpoint in your
// platform's CURRENT docs before writing it (rule 5).

export async function pendingProducts() {
  // Products vendors submitted that are awaiting the owner's decision.
  // Returns [{ id, vendorId, vendorName, title, description, price, images, category }].
}

export async function listVendors() {
  // Returns [{ id, name, status, joinedAt, productCount, email }].
}

export async function approveProduct(id) { /* ONLY from an approved board card. */ }
export async function rejectProduct(id, reason) { /* ONLY from an approved board card. */ }
export async function recentOrders({ since } = {}) { /* for the payout/fulfilment report */ }
```

**Dokan on WooCommerce**, the most common setup, as the worked example: Dokan is built on top of WooCommerce and is integrated with the WordPress REST API, so you get WooCommerce's `/wp-json/wc/v3/` endpoints plus Dokan's own `/wp-json/dokan/v1/` namespace for vendor-level data. Authenticate the same way as plain WooCommerce (consumer key and secret from **WooCommerce > Settings > Advanced > REST API**, owner generates it, into `.env`, never into chat).

Pending products are ordinary WooCommerce products in a non-published state, filtered to those owned by a vendor. Confirm the exact status value and the vendor field name in Dokan's **current** REST docs before you build the query. Do not trust this paragraph over their docs.

Note for the owner: **Elementor is irrelevant here.** It is a page builder for how the site looks. No agent in this system touches it, and nothing you build depends on it.

## Step 2: the vendor product reviewer (build this first, it is the whole module)

This is the agent that gives the owner their evenings back. Vendors submit products; the owner currently opens each one and decides. The agent does the reading and drafts the decision.

For each pending product it produces a board card with a **recommendation and the reasons**, never a silent action:

- **Images**: present, enough of them, not obviously stolen stock or a competitor's watermarked photo.
- **Title and description**: readable, specific, not keyword spam, not copy-pasted from Amazon.
- **Price**: sane for the category, and compared against similar items already on the marketplace. Flag the absurd ones both directions, a $4 "Rolex" and a $900 phone case are both problems.
- **Category fit**: is it in the right place, does it belong on this marketplace at all.
- **Policy**: does it hit anything on the owner's prohibited list (counterfeits, restricted goods, medical or financial claims, whatever their rules say).

The owner's rules live in a plain file, `marketplace-rules.md`, written in their own words. The agent reads it every run, the same way every other agent reads the memory vault. When the owner changes a rule, they edit that file, not the prompt.

Output per product: `approve`, `reject`, or `needs a human`, plus one short paragraph of why, plus the specific fix if it is rejectable. Card goes to `board/pending/`.

**The hard rule:** the agent never approves or rejects anything itself. A rejection is a real message to a paying vendor, and a wrong approval puts a bad product in front of the owner's buyers. The owner clicks; the adapter call fires only from the approved card. This is rule 3, and on a marketplace it protects a paying customer relationship, so it is stricter than usual, not looser.

**Where the leverage actually is:** even a purely advisory version of this pays for itself. The owner still reviews everything, but each card arrives pre-read with a recommendation and the reasons, so a queue that took an evening takes twenty minutes.

### Rejections are customer service, not moderation

When the owner approves a rejection, the vendor should get a message that tells them **exactly what to fix**, in the owner's voice. "Rejected" with no reason creates a support ticket and churns a paying vendor. Draft that message with the card, so approving the rejection and sending the explanation are one action.

## Step 3: the vendor growth agents (this is the revenue)

If vendors pay monthly, then every vendor is recurring revenue and every churned vendor is a permanent hole. Two agents:

**Vendor recruiter.** Reuses `modules/research.md`. It finds people already selling what this marketplace is for (on other marketplaces, on social, at local level) and drafts a personalized outreach message per prospect, with the reason this marketplace fits them specifically. The owner sends them. Do not automate the sending, and read the cold-outreach caution in `modules/marketing.md` first: bulk identical outreach gets an account or domain flagged, and the owner's reputation is the asset.

**Vendor health watcher.** A cheap, code-only loop over `listVendors()` and recent orders that flags the things a human forgets to look at:
- a vendor who has not listed anything in 30 days (about to churn)
- a vendor whose products keep getting rejected (needs help, or needs removing)
- a vendor with rising orders (worth a thank-you, a feature slot, or an upsell)
- a new vendor who signed up but never listed (stalled onboarding, the most recoverable loss there is)

Each becomes a board card with a drafted message. This is the highest-value cheap agent on a marketplace, because vendor churn is silent, and by the time the owner notices, the vendor is gone.

## Step 4: traffic, which is still the answer

A marketplace with no buyers loses vendors, and vendor churn is what kills marketplaces. So `modules/content-engine.md` applies exactly as written, with one twist: **feature the vendors.** Vendor spotlight posts do double work, they pull in buyers and they are the single best recruiting material for new vendors, because a prospective vendor sees what being on the platform gets them.

`modules/marketing.md` runs the buyer newsletter and, separately, a vendor newsletter. Keep those lists apart, they are different audiences with different offers.

## Step 5: what auto-runs and what never does

Auto-runs (reads, drafts, reports, risks nothing):

- reading the pending queue and drafting review recommendations
- the vendor health scan and its flags
- drafting vendor outreach, spotlights, and newsletters
- payout, order, and vendor-count reports

Always waits for a human click:

- **approving or rejecting any product** (money and a customer relationship)
- **any message that reaches a vendor** (rejections, outreach, warnings)
- suspending or removing a vendor
- changing fees, commissions, or marketplace rules
- anything touching payouts

Never built at all: writing to payouts, editing a vendor's live products, or deleting anything. The owner's platform holds other people's businesses. An agent that edits a vendor's listing is editing someone else's storefront, and there is no undo.

## Step 6: prove it

Rule 5, on real data:

- Run the reviewer against the **real pending queue** and show the owner the actual cards, with recommendations and reasons, next to the real products in their admin.
- Ask them to grade it: how many did it get right? Where it disagrees with them, that is a rule missing from `marketplace-rules.md`, so add it and rerun. Do this once out loud, it is how the file gets good.
- Show the vendor health scan naming a real stalled or at-risk vendor they had not noticed.
- Try to approve a product from an unapproved card and show it refuse.
- Show the negative: no code path can pay out, remove a vendor, or edit a vendor's live listing.

Then say it plainly:

"Your agents now read every product your vendors submit and hand you a recommendation with reasons, so the queue is a few minutes instead of an evening. They watch for vendors going quiet before those vendors quit. Nothing gets approved, rejected, or sent to a vendor unless you click it, because those are your customers."

## Recap

- Marketplace owners are short on review hours and vendors, not products. Build for those.
- Adapter first (`modules/any-store.md` pattern), extended with `pendingProducts` and `listVendors`.
- The vendor product reviewer is the module. Advisory-only still saves the evening.
- Vendor health watching protects recurring revenue, and it is cheap code, not an LLM loop.
- Every vendor-facing action is a human click. They are paying customers, not content.
- Dokan sits on the WooCommerce and WordPress REST APIs, and Elementor is irrelevant to all of this. Confirm every endpoint in the current docs before building.
