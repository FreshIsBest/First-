> For the AI building the owner's Agentic OS. Scaffold a brand-new Shopify store from scratch: draft products, collections, and pages built through the Admin API and parked under one batch on the board, so the owner reviews the whole store in one pass and nothing goes live without a click.

Level: intermediate | Time: ~2 evenings | Cost: Shopify plan (owner's, external) plus your engine

# Module: Shopify store scaffold

This is the Shopify version of the shop-in-a-box idea: an agent set that stands up a whole store draft on the owner's Shopify account and hands it over as one board item. Draft products, a couple of collections, a few basic pages, all created through the Shopify Admin API, all as drafts, all grouped under one `batch_id` so the owner opens the board once and sees the entire store before anything ships. If the owner already sells on Etsy or print-on-demand, this is how they add a Shopify storefront without hand-building every listing.

The rule that governs this whole module: **nothing publishes.** Every product is created with `status: draft`. Every page and collection is created unpublished. Going live is one deliberate owner decision at the end, not something an agent does on its own. Build that line in before you build anything else, same as `modules/etsy-autopilot.md`.

This assumes the spine is already built (`02-architecture.md`): the queue, the dispatcher, the board, and the guardrails. If it is not built and proven, stop and build it first. This module adds Shopify task types on top of that spine.

## Verify the API before you wire anything

**Before writing a single call, check Shopify's current Admin API docs.** Shopify versions its Admin API by date (for example `2025-01`) and deprecates old versions on a schedule. Endpoints, required fields, the product model, and auth details change between versions. The call shapes below are the pattern, not a promise about today's exact schema. Confirm the current version string, the current product and collection endpoints, and whether Shopify wants you on REST or GraphQL for a given resource before you build against them. A store scaffold that 400s on the first product is a bad first impression, so verify first, then wire.

Also know: Shopify has been steering new development toward the GraphQL Admin API and marking parts of the REST product API as legacy. Check which one the current docs recommend for products at build time and follow that. The REST shapes below are shown because they read clearly; if the docs point you to GraphQL, translate the same steps.

## Step 1: connect the Admin API (get a token)

Shopify's Admin API needs an access token tied to an app installed on the owner's store. For a single owner scaffolding their own store, a **custom app** is the plain path. Walk the owner through it in the Shopify admin, do not try to do it for them, and never ask them to paste the token into chat (rule 12).

Plain steps for the owner:

1. In the Shopify admin, go to Settings, then Apps and sales channels, then Develop apps.
2. Create an app (name it something like "OS Scaffold").
3. Under Configuration, grant Admin API access scopes. For this module the owner needs write access to products, collections/publications, themes, and online-store pages. Grant the least that does the job; do not tick scopes you will not use.
4. Install the app on the store.
5. Reveal the Admin API access token once and copy it straight into the `.env` file. Shopify shows this token once; if it is lost, uninstall and reinstall.

The owner puts these in their `.env`, and you read them from there. Never print them, never commit them:

```
SHOPIFY_STORE_DOMAIN=their-store.myshopify.com
SHOPIFY_ADMIN_TOKEN=shpat_xxxxxxxxxxxxxxxxxxxxxxxx    # Admin API access token, put here by the owner
SHOPIFY_API_VERSION=2025-01                            # confirm the current version in the docs
```

The token goes in one header on every Admin API call: `X-Shopify-Access-Token`. That header alone authenticates the request. A minimal fetch, the whole write layer is this shape:

```js
async function shopify(method, resource, body) {
  const store = process.env.SHOPIFY_STORE_DOMAIN;
  const version = process.env.SHOPIFY_API_VERSION;
  const res = await fetch(`https://${store}/admin/api/${version}/${resource}`, {
    method,
    headers: {
      'X-Shopify-Access-Token': process.env.SHOPIFY_ADMIN_TOKEN,
      'Content-Type': 'application/json',
    },
    body: body ? JSON.stringify(body) : undefined,
  });
  if (!res.ok) throw new Error(`Shopify ${res.status} on ${method} ${resource}: ${await res.text()}`);
  return res.json();
}
```

**Prove the connection before you build on it.** Make one harmless read (for example, fetch the shop record) and confirm it returns the owner's real store name. If that read works, the token and scopes are good. Do not start creating products until a live call has succeeded. Tell the owner in one plain sentence that the connection is live, then move on.

Shopify rate-limits the Admin API (REST uses a leaky-bucket call limit; GraphQL uses a cost budget). Space the scaffold calls out, watch for a 429, and back off when you see one. A store scaffold is a burst of calls, so pace it.

## Step 2: draft the products (status: draft, always)

This is the core of the scaffold. For each product the owner wants, create it through the Admin API with **`status: draft`**. A draft product exists in the admin, is fully editable, and is not visible to shoppers. That is exactly what you want: the owner reviews real products in their real admin, and nothing is buyable until they say so.

If the owner sells print-on-demand, the product images and variants come from the print-on-demand pipeline. **Do not rebuild that here.** Follow `modules/pod-printify.md` for the design-to-mockup pipeline, then feed the resulting mockups and variant data into the Shopify product create. Shopify becomes the storefront; the print-on-demand tool stays the fulfillment source. For digital or handmade goods, the product data comes from the owner's own files and the listing drafting in `modules/etsy-autopilot.md` (the same title, tags, and description work carries over).

The create call, current-version schema permitting:

```js
// Verify the current product create shape in the docs first; REST may be legacy for products.
await shopify('POST', 'products.json', {
  product: {
    title: 'Cozy Season Enamel Mug',
    body_html: '<p>12oz matte enamel mug, built for the third refill.</p>',
    vendor: 'Their Brand',
    product_type: 'Drinkware',
    tags: 'cozy season, enamel mug, slow mornings',
    status: 'draft',                       // NEVER 'active'. This is the guardrail.
    variants: [
      { option1: 'Default', price: '18.00', sku: 'MUG-COZY-12' },
    ],
    images: [
      { src: 'https://.../mockup-front.png' },   // path from the POD pipeline
    ],
  },
});
```

`status: 'draft'` is the single most important field in this module. There is no code path in the scaffold that creates a product as `active`. Publishing is a board decision at the very end, not a create-time default.

Each product create is one task (`shopify_draft_product`), one fresh agent call, one product. Write the created product's id and admin URL into the task `result` so the board can link straight to it. Give this agent write access to the Shopify products scope and nothing else (rule 2).

## Step 3: storefront basics (collections and pages, unpublished)

A store is more than a pile of products. The scaffold also stands up the minimum a storefront needs, and all of it stays unpublished until launch:

- **Collections** to group the products (for example "New Arrivals", "Best For Fall"). Create these as tasks too (`shopify_draft_collection`). A collection created before its products are approved simply has no live items yet; that is fine, it fills in on launch.
- **Basic pages** the store needs: an About page, a Contact page, a simple Shipping/Returns page. Create these through the pages endpoint. For copy, reuse `modules/content-engine.md`: the same content agent that writes posts can draft real About and policy copy from the owner's `PROFILE.md`. Do not invent policies; draft from what the owner tells you and flag anything they must confirm (shipping times, return window).
- **Theme.** Do not build a custom theme and do not touch the live theme. Shopify installs a default theme on a new store; that is enough for a first review. If the owner wants a specific look, note it as a follow-up, do not block the scaffold on design. This is the vanity-trap rule from `CLAUDE.md`: get the store working and reviewable first, make it pretty after the owner has seen it. Nudge once, then respect their call.

Every page and collection is created unpublished or drafted where the API supports it. Where a resource has no draft flag, create it and leave it out of the store's navigation and published channels until launch. The owner should be able to browse the whole scaffold in the admin and find nothing yet visible on the public storefront.

## Step 4: one batch, one board review

Here is the payoff and the reason this reads like shop-in-a-box. Every task in the scaffold (each product, each collection, each page) is created with the **same `batch_id`**. The `batch_id` column is in the canonical tasks schema exactly for this: to group child tasks so the owner reviews them as one unit instead of clicking through twenty separate approvals.

The flow:

1. A parent task (`shopify_scaffold`) reads the owner's plan (how many products, which collections, which pages) and enqueues all the child tasks, stamping each with one shared `batch_id`. Like the manager in `modules/manager.md`, this parent is a conductor: it enqueues and reports, it never creates a product itself.
2. The dispatcher works the children off the queue one fresh agent at a time. Each creates its draft resource and writes its `result` (the created id, the admin URL, and a preview: for a product, the title, price, and mockup path under `board/pending/<task_id>/`).
3. When the batch is complete, the board shows it as **one grouped item**: the whole store, ready to review.

```
$ os board
[batch cozy-store-2026-09-14]  Shopify store scaffold  -> review the whole store (8 items)
   [61] shopify_draft_product    "Cozy Season Enamel Mug   $18 draft"   [mockup]
   [62] shopify_draft_product    "Autumn Tote Bag          $24 draft"   [mockup]
   [63] shopify_draft_collection "Best For Fall (3 products)"
   [64] shopify_draft_page       "About  (draft copy)"
   [65] shopify_draft_page       "Shipping & Returns  (draft copy)"
   ...
   approve whole batch? (y/n)   or open an item to review it
```

The owner opens the batch, reviews the real drafts (in the board preview and, for anything they want to inspect closely, in their own Shopify admin where the drafts actually live), and approves or rejects. Approve is what triggers the launch action; see below. Reject on the batch, or on any child, sets that task to `rejected`, stores the note in `result.reject_note`, and can enqueue a revision task carrying `owner_feedback` in its payload (for example "make the About page shorter, less salesy"). That revision loop is the same one in `07-approval-loops.md`.

Always review the real rendered artifact, never approve a product from raw JSON. For products, the mockup image is the artifact; make the board show it (paths restricted to `board/` per the storage model). For pages, show the drafted copy.

## Step 5: launch is a separate, approved action

Approving the batch does not itself flip the store live. Following the locked approve/reject rule: **approve enqueues a follow-up ACTION task** (`shopify_publish`) that is the thing that actually publishes. That action task, for each approved product, sets the product's `status` from `draft` to `active` and publishes it to the online-store sales channel; for pages and collections, it publishes them and wires them into navigation.

Keep the launch action itself gated and deliberate. Two honest options, tell the owner both:

- **Owner flips it in Shopify (default, zero risk).** The drafts already live in the owner's admin. The owner can select the approved products and publish them by hand in the Shopify admin whenever they are ready. The scaffold's job was to build the store; the owner throws the switch. This is the simplest and safest path, especially for a first store.
- **The publish action task does it.** If the owner wants the system to flip the approved drafts to `active`, the `shopify_publish` task does exactly that, but only after the batch was approved on the board, and only for the items that were approved. Even then, treat the very first launch as something the owner watches happen.

Either way, a human approved the store before anything became buyable. There is no path from "agent created a product" to "product is live" that skips the board.

(Why this module allows an approved `shopify_publish` to set items active while `modules/dropshipping.md` never does: this is a one-time launch event the owner is watching, a batch they just reviewed end to end. Dropshipping is an ongoing catalog where products arrive continuously, so it keeps the stricter always-flip-by-hand rule as a second gate. Same platform, different cadence, deliberate difference.)

## What to do after launch

Once the store is live, it plugs into the rest of the OS:

- **Content.** Point `modules/content-engine.md` at the new Shopify products the same way it points at Etsy listings. New product live means a post to draft and ship (through the board).
- **Traffic.** The Pinterest and posting loops from `modules/etsy-autopilot.md` and the content engine work against Shopify product URLs just as well as Etsy URLs.
- **Marketing.** When the owner is ready to spend on ads or send a launch email, that runs through `modules/marketing.md`, board-gated with spend caps, never automatic.

## Guardrails specific to Shopify (enforce in code)

- **Products are created `status: draft`. Always.** There is no create path that sets `active`. Going live is the separate, approved `shopify_publish` action. This is the single most important rule in this module.
- **Pages, collections, and theme stay unpublished until launch.** The owner sees a full scaffold in the admin with nothing yet visible on the storefront.
- **Never touch the live theme.** The scaffold does not edit or overwrite an existing published theme. It uses the store's default theme for review and leaves design as a post-launch, owner-directed follow-up.
- **One `batch_id` per scaffold.** Every child task carries the same batch id so the owner reviews the whole store in one board pass, not twenty.
- **Least scope.** Grant the custom app only the scopes the scaffold uses. The token lives in `.env`, never in chat, never in a commit (rule 12).
- **Respect the rate limit.** Pace the burst of create calls, back off on a 429. A scaffold is a lot of calls at once.
- **Verify against current docs first.** Confirm the API version, the product/collection/page endpoints, and REST-versus-GraphQL before wiring. Versions change and old ones get deprecated.

## Prove-it-works checklist (run before calling the Shopify scaffold done)

Do not tell the owner "the store is scaffolded" until every one of these is true and you have shown the output:

- [ ] A live Admin API read returned the owner's real store name (connection proven before any writes).
- [ ] Every product was created with `status: draft` and is visible in the admin but not on the public storefront (spot-check one on the live storefront to confirm it is not buyable).
- [ ] Collections and pages were created unpublished, with drafted copy the owner can read.
- [ ] Every scaffold task shares one `batch_id` and the board shows the whole store as a single grouped item.
- [ ] The board preview shows the real artifact (product mockups, page copy), not raw JSON.
- [ ] Reject on the batch or a child sets `rejected`, stores the note, and can spin a revision task.
- [ ] Approve enqueues a `shopify_publish` action task; there is no path that publishes without that approval.
- [ ] The owner has been told, in plain sentences, that everything is a draft and exactly how launch happens.

When all boxes are checked, the scaffold is real: a full store draft the owner reviews in one pass, with launch as one deliberate approved step.

---

_Shopify is a third-party platform with its own subscription cost, paid separately by the owner, and its own terms of service the owner is responsible for following. Its Admin API is versioned and changes; verify against the current docs before wiring, as instructed above. A store going live carries no guaranteed outcome and results are not typical. Keep every product a draft until the owner approves launch._
