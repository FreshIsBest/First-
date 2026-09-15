> For the buyer's AI. Build a service business the owner actually sells on Fiverr (or Upwork, or direct): logo and brand identity, social packs, copywriting, whatever they can deliver. Agents collapse the production time per order from hours to minutes, while the owner stays the designer and the seller of record. This module is deliberately NOT "AI runs your Fiverr while you sleep", and the section on why is the most important part of the file.

Level: intermediate | Time: ~2 to 3 evenings to first delivered order | Cost: $0 platform fee to start (Fiverr takes 20% of each sale), plus image generation credits and your engine

# Fiverr services (agent-assisted, human-delivered)

## The two hard limits, before you plan anything

An owner arrives here imagining a machine that takes orders and delivers logos while they sleep. Tell them the truth in the first minute, because everything below is shaped by it and finding out later means a dead account.

**Limit 1: there is no Fiverr API.** Fiverr publishes no public API for gigs, orders, buyer messages, or delivery. Driving the site with a headless browser to fake one is against their terms and puts the earning account at risk. So the platform actions stay human: the owner reads the order, sends the message, clicks deliver. The agent does the *work*, not the *clicking*.

**Limit 2: Fiverr's AI rules forbid the machine the owner is picturing.** Fiverr does allow AI-generated content, including logos, so this is a real business. But their Community Standards say, in substance:

- sellers must deliver **customized work for each order**, and must not deliver AI-generated content **in bulk, where the same work goes to multiple clients**
- AI must **support the freelancer's own skill and effort, not replace it**
- **generic, unmodified, or reused AI output does not meet their quality standard**
- the seller must hold the rights to what they deliver, and infringing content means removal and **possible permanent account suspension**

Read that list again as a build spec, because that is what it is. It rules out the logo mill. It explicitly permits an owner who uses AI to work faster on custom orders. Confirm the current wording in Fiverr's own Help Center before you rely on this file (rule 5); policies move.

**So what you are building:** the owner's production pipeline, not their storefront. A gig that used to eat three hours of their evening takes twenty minutes, and their judgment is still in every delivery. That is a real, legal, durable business. It is also the version that survives, because "generic unmodified AI output" is precisely what gets accounts removed.

If the owner only wants the hands-off version, tell them plainly that it does not exist on Fiverr, and point them at `modules/etsy-autopilot.md` or `modules/pod-printify.md`, where they sell their own products and nobody's terms of service governs how the work was made.

## Step 1: pick a service the agent genuinely accelerates

Not every gig benefits. The test: **is the deliverable mostly digital creation, and is the brief short?** Score a candidate on three things.

Good fits:

- **Logo and brand identity.** Short brief, high perceived value, $25 to $150+, and huge volume of buyers. The worked example in this file.
- **Social media content packs.** Ten to thirty on-brand posts. Repetitive by nature, which is exactly what kills a human and suits an agent.
- **Copywriting**: product descriptions, About pages, email sequences, ad variations.
- **Pitch decks and one-pagers.** Structure plus writing plus light design.
- **SEO content briefs and articles.** Reuses `modules/research.md` directly.

Poor fits, do not start here:

- Video editing (the file wrangling dominates, and the AI does not do the hard part)
- Voice over (the platform is saturated and buyers want a specific real voice)
- Anything needing account access to the buyer's systems (a support burden, not a product)
- Anything where the buyer is really paying for a conversation, like consulting

Have the owner pick **one** and stay there until it is profitable. A seller with one strong gig and real reviews beats a seller with eight thin ones, because Fiverr's ranking rewards completion rate and review volume inside a category.

## Step 2: the pipeline, and exactly where the agent sits

Six stages. Two are human by necessity, four are the agent's.

```
1. Order lands            HUMAN   (Fiverr notifies the owner)
2. Brief -> spec          AGENT   turn the buyer's messy answers into a build spec
3. Concepts               AGENT   generate several genuinely different directions
4. Owner curates          HUMAN   picks, directs, kills the bad ones. THIS IS THE JOB.
5. Package                AGENT   produce every file format the gig promises
6. Message + deliver      HUMAN   owner sends it on Fiverr
```

Stage 4 is not overhead to be optimized away later. It is the "own skill and effort" the platform requires, it is why the work is customized, and it is what makes reviews good. Build the pipeline so stage 4 is fast and pleasant, never so it can be skipped.

The owner pastes the buyer's brief into the system (or forwards the Fiverr notification email, which the agent can read). Everything from there until delivery runs locally.

## Step 3: the BRIEF agent (turn a vague buyer into a spec)

Fiverr buyers write briefs like "modern logo for my coffee shop, something clean". The first agent's job is to turn that into something a generator can execute, and to surface what is missing before any work is done.

Its output is a `spec.json` on the board:

```json
{
  "business": "Ridgeline Coffee",
  "industry": "specialty coffee roaster, sit-down cafe",
  "audience": "commuters and remote workers, 25 to 45, small mountain town",
  "adjectives": ["clean", "warm", "outdoorsy", "not corporate"],
  "avoid": ["cartoon mascots", "coffee cup with steam", "generic script fonts"],
  "colorDirection": "muted earth tones, one warm accent",
  "logoType": "wordmark with a simple mark",
  "deliverables": ["SVG", "transparent PNG", "favicon", "one-page brand sheet"],
  "openQuestions": ["do you want the mark to work standalone, without the name?"]
}
```

Two details that matter more than they look:

**The `avoid` list is where quality comes from.** Every coffee logo on Fiverr is a steaming cup. Instruct the agent to fill `avoid` with the clichés of that specific industry, not just what the buyer said. This single field is most of the difference between output that reads as generic AI and output that reads as design.

**`openQuestions` gets asked, not guessed.** If the brief is genuinely ambiguous on something structural, the owner sends one short question before generating. Buyers experience that as professionalism, and it prevents a revision round.

## Step 4: the CONCEPT agent (and the vector problem, which is the real work)

Generate **three to five genuinely distinct directions**, not five versions of one idea. Give the generator the spec plus an explicit instruction that each concept must differ in approach, for example: one wordmark-led, one abstract mark, one literal-but-fresh, one monogram.

### The vector problem

This is where most people building a logo gig fail, so handle it head on. **Image generators produce raster (PNG). Logo buyers need vector (SVG, and often EPS or AI).** A vector scales to a billboard; a PNG does not. Delivering only PNGs gets you a one-star review from any buyer who knows the difference, and the ones who do not know will come back angry when their printer rejects it.

Three workable paths, in order of quality:

1. **Generate the SVG directly.** For geometric marks, monograms, and wordmarks, have the model output SVG source. Modern models are decent at clean geometric SVG, and the output is genuinely editable and genuinely vector. Best quality, most control, needs the concept to be geometric rather than painterly.
2. **Raster first, then trace.** Generate a high-contrast black-and-white concept image, then vectorize it (`potrace`, `vtracer`, or an equivalent). Then the owner cleans up the paths. Good for organic or hand-drawn marks. The cleanup step is real work; do not pretend it is free.
3. **Type plus a simple mark, composed in code.** Many strong logos are a well-chosen typeface plus a small geometric element. Compose the SVG programmatically: text element, licensed font, a shape. Fastest and most reliable, and it sidesteps generation drift entirely.

Path 3 is where a new owner should start. It is the least glamorous and it produces the most sellable output.

### Font licensing (the trap that ends accounts)

Fiverr requires the seller to hold the rights to what they deliver, and a logo is exactly where this goes wrong. **Fonts are licensed software.** Most free-for-personal-use fonts do not permit commercial logo use, and delivering one to a buyer who then trademarks it is a real problem for the owner.

Build a small `fonts.md` in the project listing only fonts cleared for commercial embedding and logo use (the SIL Open Font License family is the safe default, and Google Fonts is largely OFL). The concept agent may only use fonts from that list. Put the font name and its license in the delivered brand sheet, so the buyer knows what they own. This is a five minute setup that prevents the one mistake that suspends an account.

### Trademark sanity

The agent is not a lawyer and must not act like one. But a cheap check is worth doing: before delivery, search the proposed mark and name against existing brands in that industry, and flag anything obviously close to a known logo. Put the finding on the card for the owner to look at. Say plainly to the buyer, in the gig description and the delivery, that trademark clearance is their responsibility. Never claim the mark is legally clear.

## Step 5: the PACKAGE agent (deliver like a studio, not a hobbyist)

Once the owner picks a direction, this agent produces everything the gig promised, from the one approved source file. This is pure code, no model call, and it is where a $25 gig starts feeling like a $150 one.

- **Vector**: SVG (and PDF, which most buyers can open; export EPS only if the gig promised it)
- **Raster**: transparent PNG at 500, 1000, 2000 px
- **Variants**: full color, all black, all white (reversed), and a horizontal plus a stacked lockup
- **Favicon**: 32 px, plus a 512 px app icon
- **Social**: a square profile crop and a banner
- **Brand sheet**: one PDF or PNG with the logo, the hex colors, the font names and their licenses, and a short "here is how to use this" note
- Everything in a tidy folder structure, zipped, named for the buyer's business

The variants matter. The single most common revision request on a logo order is "can I get a white version for my dark background". Ship it before they ask and the revision never happens.

## Step 6: the MESSAGE agent (drafts, the owner sends)

Three templates the owner edits and sends by hand on Fiverr:

- **Order received**: confirms the brief in the owner's own words, asks any `openQuestions`, sets the delivery expectation. Sending this within an hour of the order does more for reviews than almost anything else.
- **Delivery**: what they are getting, what each file is for, how to ask for a revision.
- **Revision received**: acknowledges the change and restates it, so a mismatch surfaces before the rework.

Keep them short and in the owner's voice (`modules/brand-kit.md` has the voice guide, and it applies to the owner's own business here). Never auto-send: these are messages to a paying customer on a platform where a bad exchange costs the account's rating.

## Step 7: pricing and packages

Structure it so the agent's speed becomes margin instead of a race to the bottom.

| Package | Contains | Typical |
| --- | --- | --- |
| Basic | 1 concept, PNG only, 1 revision | $25 to $40 |
| Standard | 3 concepts, full vector pack, 2 revisions | $60 to $100 |
| Premium | 3 concepts, vector pack, brand sheet, social kit, source files, unlimited revisions in scope | $120 to $250 |

Notes that matter:

- **Fiverr takes 20%.** A $25 gig nets $20. Price the Basic tier so it is worth doing at all, and treat it as the entry point that sells the Standard.
- **Most of the value is in the package step**, which costs the owner almost nothing once built. That is why Standard and Premium are where the money is.
- **Delivery speed is a ranking and conversion lever.** Being genuinely able to deliver in 24 hours, because the pipeline is fast, is a real advantage over sellers who need three days.
- Cap revisions explicitly in the gig description. "Unlimited revisions" on a logo gig attracts the buyers who will consume an entire evening for $60.

## Step 8: what auto-runs and what never does

Auto-runs (local, produces files, risks nothing):

- parsing the brief into a spec
- generating concepts
- packaging the approved direction into every format
- drafting buyer messages
- the trademark and font-license flags

Always a human:

- **choosing and directing the concept** (this is the skill the platform requires, and the reason the work is customized)
- **every message sent to a buyer**
- **clicking deliver on Fiverr**
- creating, editing, or pausing gigs
- anything touching the account itself

Never built: browser automation against Fiverr, auto-accepting orders, auto-delivering, or reusing a previous buyer's deliverable for a new order. That last one is worth stating to the owner explicitly, because it is the tempting shortcut and it is the one that is explicitly against the rules.

## Step 9: prove it, on a real order

Rule 5, and here it has an unusually honest form: **the owner should run the pipeline on a fake brief before their first real buyer, and then on their first real order, end to end.**

- Feed it a realistic brief and show the actual `spec.json`, including what it put in `avoid`.
- Show the real generated concepts, and be honest about how many are usable. If four of five are weak, say so; that is normal and it is why the owner curates.
- Open the delivered SVG and confirm it is genuinely vector, with sane paths, not a PNG in a wrapper.
- Show the full packaged folder and open the white-on-dark variant.
- Show the font used and its license line on the brand sheet.
- Time it. Compare against how long the owner's manual version took. That number is the whole business case.

Then say it plainly:

"This does the production for you. You still pick the direction and you still hit send, because Fiverr requires the work to be customized and to involve your own skill, and because that is what keeps your reviews good. What changed is that an order takes twenty minutes instead of three hours, so you can take more of them and deliver faster than the sellers you are competing with."

## Scaling, and the trap

When it works, the owner will want to add gigs. The order to do it in:

1. **Raise prices before raising volume.** Faster delivery and a better package justify it, and it is free margin.
2. **Add adjacent deliverables to the same gig** (brand sheet, social kit) as upsells. The package agent already makes them.
3. **Only then add a second gig type**, reusing this same pipeline shape.

The trap: taking so many orders that the owner starts skipping stage 4 and shipping raw generations. That is the exact moment the business becomes the thing Fiverr suspends accounts for, and the reviews go first anyway. If volume ever forces that choice, the answer is higher prices, not thinner work.

## Recap

- No Fiverr API and no compliant automation of the platform. Agents do the work; the owner does the clicking.
- Fiverr allows AI-assisted delivery but requires customized work per order, forbids bulk or reused output, and requires the seller's own skill in the loop. Build to that, not around it.
- Four agents: brief to spec, concepts, package, messages. The owner curates in the middle and that stage is load-bearing.
- For logos: solve vector properly, and clear your fonts. Those two details separate a real gig from a refund.
- The money is in the package step and the price tier, not in doing more orders badly.
