> For the AI building the owner's Agentic OS. Build a print-on-demand pipeline on Printify, connected to their Etsy shop, that turns a design into a DRAFT product and only publishes on approval.

# Module: print-on-demand on Printify

This is the money-maker that pairs with the Etsy autopilot. Etsy sells the listing; Printify makes and ships the physical product (a shirt, mug, tote, sweatshirt) with no inventory and no upfront cost. The owner uploads a design, you create a draft, they approve, and Printify pushes a real listing to their Etsy shop. When it sells, Printify prints and ships it. The owner never touches stock.

Build this the same way as everything else: function first, nothing risky auto-runs. A Printify "publish" pushes a live listing to a real storefront. That is a board action, never automatic.

**Timing note.** If print-on-demand is the owner's money-maker, you build this in the early steps of `03-build-order.md`, BEFORE the automated queue and dispatcher exist (Step 6). Run the POD agents by hand for now: produce each product design and draft, show the owner, they approve by hand. The queue and board language here applies once the system is built at Step 6; before then the same steps happen manually with the owner approving each one. The POD pipeline is the "make the product" half; a storefront (the Etsy autopilot, `modules/etsy-autopilot.md`) is the "run the shop" half. Once the system exists, they share the same board.

## What you are building

A small set of scripts and one agent task type:

1. **A Printify client** (one file) that talks to the Printify API: upload a design, pick a blueprint and provider, create a draft, publish a draft, delete a draft.
2. **A POD agent** task type (`make_pod_product`) that takes a design plus product details and creates a DRAFT product. It never publishes.
3. **A publish step** that only runs after the owner approves the draft on the board.

Keep it hand-rolled. One HTTP client, plain functions, no SDK the owner cannot read.

## Step 1: connect Printify

The owner needs a Printify account with their Etsy shop connected inside Printify (Printify calls this a "sales channel"). Walk them through it:

1. Create a Printify account and connect their Etsy shop under **My Stores** (Printify handles the Etsy OAuth).
2. Generate an API token: **Printify > Settings > Connections > API**. Copy the token.
3. Save it as an environment variable. Never print it, never commit it.

```
PRINTIFY_API_TOKEN=xxxxxxxxxxxxxxxxxxxx
```

All Printify calls use `Authorization: Bearer $PRINTIFY_API_TOKEN` against `https://api.printify.com/v1`.

Confirm the connection before building anything else. List the shops and find the Etsy-connected one:

```
GET https://api.printify.com/v1/shops.json
```

Each shop has an `id`, `title`, and `sales_channel`. Pick the shop whose `sales_channel` mentions `etsy`, and fall back to the first shop if there is only one. Store that shop id. Everything downstream needs it. If this call fails, stop and fix the token before going further. Prove the connection works out loud (rule 5).

## Step 2: the shop concept

The owner's Printify shop is a mirror of their Etsy shop. Every product you create in Printify becomes an Etsy listing when published. So the shop concept is the same as the Etsy shop concept from their `PROFILE.md`: the niche, the vibe, the buyer. **If a `brand.md` exists (from `modules/brand-kit.md`), read it and design in its product-design style and palette**, so new products stay on-brand instead of drifting. Do not invent a second identity. If my own shop sells cozy-season designs on Etsy, the Printify products are cozy-season shirts, mugs, and totes. The design carries the brand; the garment is just the canvas.

Decide up front, from the owner's profile, which product types to offer. Start with one or two, not all of them. Good starter set:

- **Tee** (Gildan 5000, the standard blank).
- **Mug** (11oz ceramic).
- Optionally a **sweatshirt** or **tote** once tees sell.

Do not offer eight product types on day one. One design on one good garment that sells beats ten half-configured drafts.

## Step 3: pick a blueprint and a print provider

Printify calls a base product a **blueprint** (for example "Gildan 5000 Unisex Heavy Cotton Tee"). Each blueprint is offered by several **print providers** (the actual print shops), each with their own prices, quality, and ship times.

Look these up by title, not by hardcoded id. Printify renumbers ids, so matching on the title keeps you correct over time. Sensible defaults:

- **Tee:** match `gildan 5000` or `unisex heavy cotton tee`.
- **Sweatshirt:** match `gildan 18000` or `crewneck sweatshirt`.
- **Mug:** match `11oz` ceramic mug.
- **Tote:** match `cotton tote`.

```
GET /catalog/blueprints.json                                  -> find the blueprint by title
GET /catalog/blueprints/{blueprintId}/print_providers.json    -> pick a provider
GET /catalog/blueprints/{blueprintId}/print_providers/{providerId}/variants.json
```

For the provider, taking the first one in the list is a fine default (Printify orders them roughly by relevance). If the owner cares about cost or ship time, let them pick from the list on the board. The variants call returns every color and size combination, each with a variant id. You need those ids to build the product.

## Step 4: upload the design

Upload the design to Printify before you create the product. Two ways:

- **By URL:** give Printify a public URL and it fetches the image.
- **By base64:** read a local PNG, base64-encode it, and send the contents.

```
POST /uploads/images.json
{ "file_name": "cozy-season.png", "contents": "<base64>" }
```

Use a transparent PNG at print resolution. The response returns an image `id`, plus `width` and `height`. Hold onto that `id`. Design requirements to enforce before upload:

- **Transparent background.** A design with a white box behind it prints a white box on the shirt. Strip the background to transparency first.
- **High resolution.** Aim for roughly 4000px on the long edge. Low-res art prints blurry and gets returns.
- **The visual comes from the content engine.** Do not generate art in this module. The design is produced upstream (see `modules/content-engine.md` and the Higgsfield section of `tools/tools.md` for the asset step). This module takes a finished PNG and puts it on a product.

## Step 5: create a DRAFT product (never publish here)

Create the product as an unpublished draft. It lands in Printify but is NOT pushed to Etsy. This is the whole safety model: the POD agent produces drafts, the board publishes them.

```
POST /shops/{shopId}/products.json
{
  "title": "...",
  "description": "...",
  "blueprint_id": <blueprintId>,
  "print_provider_id": <providerId>,
  "variants": [ { "id": <variantId>, "price": 2499, "is_enabled": true }, ... ],
  "print_areas": [
    {
      "variant_ids": [ ...all variant ids... ],
      "placeholders": [
        { "position": "front", "images": [ { "id": "<uploadId>", "x": 0.5, "y": 0.5, "scale": 1, "angle": 0 } ] }
      ]
    }
  ],
  "tags": [ "cozy", "fall", "..." ]
}
```

`price` is in cents (2499 = $24.99). Set it to cover the provider's base cost plus the owner's margin; pull the base cost from the variants call. Etsy allows up to 13 tags, so send at most 13.

### Gotcha 1: the 100-variant cap

**Printify caps enabled variants at 100 per product.** A garment with many colors and sizes can exceed 100 combinations (sweatshirts especially). If you enable more than 100, the create call fails.

Enable at most the first 100 variants. Map over the variant list and set `is_enabled: i < 100`. You still pass every variant id in `print_areas.variant_ids` so the print placement applies across the board, but only the first 100 are sellable. This is the real fix I hit; do not skip it.

### Gotcha 2: design placement scale

The `scale` and `x`/`y` position control how the art sits on the print area. `scale: 1` fills the whole print area. **A full-bleed scale of 1 often blows the art up too large**, cropping it at the edges, especially on wide print areas like mugs and totes. Use a scale per product type:

| Product | Scale | Why |
|---|---|---|
| Tee | ~1.0 | Chest print, square-ish area, full size is fine |
| Sweatshirt | ~1.0 | Same as tee |
| Mug | ~0.52 | Wide wrap-around area, a square design must shrink hard or it wraps off the edges |
| Tote | ~0.72 to 0.85 | Sits smaller than a shirt, shrink so it centers cleanly |

`x` and `y` are 0 to 1 (0.5, 0.5 is centered). Default to centered. These numbers are starting points, not gospel. Always look at the returned mockup before approving (next step).

### The output

The create call returns the product with an `id`, the `variants` (check how many came back `is_enabled`), and `images` (Printify auto-generates mockup photos). Write a result the board can show:

```
RESULT { "id": 6889..., "productType": "tshirt", "title": "...", "mockups": ["https://...", ...] }
```

Set the task status to `needs_approval`. This follows the standard storage model from `02-architecture.md`: the JSON above goes in the task's `result` column, and if you download a mockup or generate any local file, it lives under `board/pending/<task_id>/` with its path in `result.preview`. The mockup URLs are the whole point of the review: the owner looks at the shirt render and says publish or reject. Nothing is live yet.

## Step 6: publish only on approval

Publishing pushes the listing to Etsy. This is a board action. It runs only after the owner approves the draft.

On approve, run a follow-up task that calls:

```
POST /shops/{shopId}/products/{productId}/publish.json
{ "title": true, "description": true, "images": true, "variants": true, "tags": true, "keyFeatures": true, "shipping_template": true }
```

That pushes the full listing to the owner's Etsy shop and it goes live for sale. From here the Etsy autopilot (`modules/etsy-autopilot.md`) owns the listing: it can adjust tags, answer buyer questions, and track the sale. The handoff is clean because both sit on the same board and the same queue.

On reject, delete the draft so it does not clutter Printify:

```
DELETE /shops/{shopId}/products/{productId}.json
```

## How the POD agent fits the Etsy autopilot

The two modules are one pipeline with a board in the middle:

```
content engine ─┐
 (makes design) │
                ▼
        make_pod_product agent ──> DRAFT product ──> BOARD (owner approves)
                                                         │
                                                approve  │  reject
                                                         ▼        ▼
                                              publish to Etsy   delete draft
                                                         │
                                                         ▼
                                             Etsy autopilot runs the listing
```

- The **content engine** produces the design PNG.
- The **POD agent** (this module) turns a PNG into a Printify draft and parks it on the board.
- The **owner** approves the mockup.
- The **publish step** pushes it to Etsy.
- The **Etsy autopilot** takes over the live listing.

Give the POD agent only the tools it needs: read the design file, call the Printify client, write a result. It does not need the ability to publish; publishing is a separate follow-up task gated by approval (rule 3).

## Guardrails for this module

- **No auto-publish.** Draft creation is safe and can run from the queue. Publishing is a board action, always. Build the publish step as a separate task that only the board can trigger.
- **Cap drafts per run.** Set a max number of drafts the POD agent creates per day so a loop cannot flood Printify and Etsy with junk listings. Ask the owner for the cap in the interview and enforce it in code.
- **Check the mockup before publish.** The mockup URL exists so a human can catch a design that placed wrong (blown-up art, off-center, cropped). Do not let the owner approve blind; show the mockup on the board.
- **Never print the API token.** Load it from the environment, use it, never log it.

## Prove it works

Before you tell the owner this is done: create one real draft end to end, show them the mockup URL, and confirm the draft exists in their Printify dashboard as unpublished. Then, on their explicit approval, publish exactly one product and confirm the listing appears in their Etsy shop. Do not claim it works until you have seen both. (rule 5)

## Note on money

Print-on-demand has real costs: Printify charges the provider base cost per item when a sale happens, and Etsy charges listing and transaction fees. The owner earns the difference between their retail price and those costs. Set retail prices that clear a real margin after both. Results are not typical; a live listing is not a guaranteed sale.
