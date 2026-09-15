> For the buyer's AI. THE UNIVERSAL STORE MODULE. The owner sells somewhere this Blueprint does not name: WooCommerce, BigCommerce, Squarespace, Wix, Magento, Amazon, eBay, Walmart, Faire, a headless build, or their own custom cart. Build them the same agent-run store as the flagship modules by writing ONE small adapter and reusing everything else unchanged. Nothing here is platform-specific except that adapter.

Level: intermediate | Time: ~1 to 2 evenings for the adapter, then the normal module time | Cost: $0 beyond the owner's existing store and your engine

# Any store (the universal build)

## When you are in this file

The owner answered the platform question with something other than Etsy, Printify, or Shopify. Maybe they already run a real store on WooCommerce with their own vendors and inventory. Maybe it is BigCommerce, Wix, a marketplace, or something hand-built.

Do not send them away, and do not tell them to migrate to Shopify. They already have the hardest part of a store: real products and a place to sell them. Migrating is a huge, risky, pointless project when the agents do not care what the storefront is.

**The Blueprint is universal by design.** The store modules (`modules/etsy-autopilot.md`, `modules/shopify-store.md`, `modules/pod-printify.md`, `modules/dropshipping.md`) are worked examples of one pattern, not the only platforms that work. This file is that pattern, stated directly.

## The core idea: only ONE piece is platform-specific

Look at what an agent-run store actually does:

- research a niche or product
- write the listing (title, description, tags, price)
- make the image or video
- publish the product
- drive traffic with content
- watch orders and answer customers
- report what happened

Exactly **one** of those touches the store: publish the product (and read orders). Everything else is writing, thinking, and posting, and none of it knows or cares whether the storefront is Etsy or WooCommerce.

So you do not rebuild the system per platform. You write a **store adapter**: one small file with a handful of functions. Every other module in this Blueprint calls the adapter and keeps working untouched.

This is the whole module. The rest is how to write that adapter safely.

## Step 1: find out what the platform can actually do (do not guess)

Rule 5 applies harder here than anywhere else, because you are working against an API this file has never seen. **Do not build from memory or assumption.** Before writing a line:

1. Ask the owner exactly what platform they are on, and whether they are the admin (they need permission to create API keys).
2. Look up that platform's **current** REST API docs. Confirm four things and write them into `PROFILE.md`:
   - **Auth**: API key pair, OAuth, bearer token, or a plugin-generated secret?
   - **Create a product**: endpoint, method, required fields.
   - **Draft state**: what field keeps a product unpublished? This one matters most, see Step 3.
   - **Read orders**: endpoint, and whether their plan allows it.
3. If the platform has **no usable API**, that is fine and common. Go to the CSV path in Step 5.

Tell the owner plainly what you found, including limits. "Your plan does not expose the orders endpoint, so the fulfillment agent cannot run, everything else can" is a real and useful answer. Discovering that on day one beats discovering it three agents deep.

## Step 2: write the adapter (the only platform-specific code)

One file, `store/adapter.mjs`, exposing a tiny fixed interface. Keep it this small; the temptation to wrap the platform's whole API is a trap.

```javascript
// store/adapter.mjs, the ONLY file that knows which platform the owner uses.
// Swap this file, and every agent in the system works against a different store.
// Illustrative shape: confirm the real endpoints and fields in your platform's
// CURRENT docs before writing it (rule 5).

export async function createDraftProduct(p) {
  // p = { title, description, price, sku, tags, images: [url] }
  // MUST create it UNPUBLISHED. Returns { id, adminUrl }.
}

export async function listProducts({ limit = 50 } = {}) {
  // Returns [{ id, title, price, status }]. Used to avoid duplicates and to report.
}

export async function publishProduct(id) {
  // Flips one already-approved draft live. Called ONLY from an approved board card.
}

export async function recentOrders({ since } = {}) {
  // Returns [{ id, placedAt, total, status, items }]. Optional: skip if the
  // platform or plan does not expose it, and say so plainly in the report.
}
```

Four functions. That is the entire platform surface. `modules/dropshipping.md`, `modules/shop-in-a-box.md`, the listing agent, and the fulfillment watcher all call these instead of a Shopify or Etsy client.

### Worked example: WooCommerce

WooCommerce is the most common "other" and a good template for the rest. It ships a full REST API (`/wp-json/wc/v3/`), authenticated with a consumer key and secret the owner generates in **WooCommerce > Settings > Advanced > REST API**. Have them create a key with Read/Write and put it in `.env` themselves; never ask for it in chat (rule 11).

```javascript
// store/adapter.mjs for WooCommerce. Confirm current endpoints/fields in Woo's docs.
const BASE = `${process.env.WOO_URL}/wp-json/wc/v3`;
const auth = 'Basic ' + Buffer.from(
  `${process.env.WOO_KEY}:${process.env.WOO_SECRET}`
).toString('base64');

export async function createDraftProduct(p) {
  const res = await fetch(`${BASE}/products`, {
    method: 'POST',
    headers: { Authorization: auth, 'Content-Type': 'application/json' },
    body: JSON.stringify({
      name: p.title,
      description: p.description,
      regular_price: String(p.price),
      sku: p.sku,
      status: 'draft',            // THE SAFETY LINE. Never 'publish' here.
      catalog_visibility: 'hidden',
      images: (p.images || []).map((src) => ({ src })),
    }),
  });
  if (!res.ok) throw new Error(`Woo create failed ${res.status}: ${await res.text()}`);
  const d = await res.json();
  return { id: d.id, adminUrl: `${process.env.WOO_URL}/wp-admin/post.php?post=${d.id}&action=edit` };
}
```

`status: 'draft'` is the equivalent of Shopify's `status: draft` and Printify's publish gate. Every platform has one. **Find it before you write the create call**, because it is what keeps a half-written agent from putting a live product in front of real customers.

For other platforms the shape is the same and only the names move: BigCommerce uses `is_visible: false`, Wix and Squarespace have their own draft or hidden states, marketplaces like eBay and Amazon have a separate "inactive" or "incomplete" listing state. Look yours up.

## Step 3: the safety rule, restated for a store you did not build

The owner's store is a **live business** with real customers and real money. That makes this module more dangerous than the from-scratch ones, where a mistake hits an empty shop nobody has seen.

Non-negotiables, all of them rule 3 in `CLAUDE.md`:

- **Every product the agent creates is a draft.** No exceptions, no "just this once", no flag that skips it.
- **The agent never edits or deletes an existing product.** It only creates new drafts. Touching live listings is how an agent quietly wrecks a working catalog, and there is no undo on someone else's storefront.
- **Publishing is a human action** on an approved board card, never a step the agent chains on its own.
- **Prices are drafted, never pushed.** A price change on a live store is a money decision. Draft it to the board with reasoning; the owner applies it.
- **Never touch orders, refunds, or customer data as a write.** Read them for reports if the API allows. Writing there is real money and real people.

Say all of this to the owner out loud before you build. On an existing store, trust is the product.

## Step 4: reuse everything else, unchanged

Once the adapter exists, the rest of the Blueprint applies with no platform work at all:

- **`modules/content-engine.md`** drives traffic. Completely platform-agnostic. For most owners with an existing store, this is the single highest-value module in the whole Blueprint, because they already have product and are short on demand.
- **`modules/marketing.md`** drafts emails, newsletters, and ad tests, and reads last week's numbers.
- **`modules/research.md`** scouts niches, keywords, and competitors. On an existing store it is also how you decide what to stock next.
- **`modules/dropshipping.md`** listing agent writes the product pages. Point its publish step at your adapter instead of Shopify.
- **`modules/manager.md`** runs the daily digest across all of it.
- **`modules/memory.md`**, **`modules/brand-kit.md`**, **`modules/signature-dashboard.md`**: no changes at all.

Do not rebuild any of these here. Build the adapter, then work through those modules normally.

## Step 5: no API? the CSV path still works

Plenty of stores either have no API, gate it behind a higher plan, or the owner cannot get admin access. This is not a dead end.

Every major store host imports products from CSV. Change `createDraftProduct` so that instead of calling an API, it appends one row to `board/approved/products-import.csv` using the host's exact column headers, with the status column set to draft or unpublished. The owner uploads the file in their admin.

The agent builds the file, the owner does the upload and the go-live. Same safety rule, no API required. Everything else in the system is unaffected, because the adapter is still the only thing that changed.

## Step 6: prove it, out loud

Rule 5, and be strict about it on a live store:

- Show the owner the **real draft product in their own admin**, opened in their browser, not a JSON blob and not your description of it.
- Show that it is genuinely unpublished: it does not appear on the storefront. Load the public shop page in front of them and show it missing.
- Show `listProducts()` returning their actual catalog, so they can see the connection reads correctly too.
- Try to publish from an unapproved card and show it refuse.
- If the orders endpoint is unavailable on their plan, say that plainly now, not later.

Then say it in plain words:

"Your agents can now create products in your own store, but only ever as drafts. Nothing goes live, nothing gets edited, and nothing gets deleted unless you click approve. The rest of the system, content, marketing, research, works the same as it does for anyone else, because none of it cares what your storefront runs on."

## Recap

- The Blueprint is platform-universal. Only ONE file, `store/adapter.mjs`, knows the platform.
- Four functions: create a draft, list, publish an approved one, read orders. That is the whole surface.
- Look up the platform's CURRENT API before writing anything, and find its draft state first.
- On an existing live store: create drafts only, never edit or delete, never write orders or prices.
- No API is fine, write CSV rows instead.
- Everything else (content, marketing, research, manager, memory) is reused with zero changes, and for an owner who already has products, content is usually the module that actually makes them money.
