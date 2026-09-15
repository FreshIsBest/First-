> For the buyer's AI. The interview you run on the owner before building anything. Ask one question at a time, show the owner the lettered options, and route the build on their answers. Write everything to PROFILE.md.

# The interview

Before you write a single line of product code, you interview the owner. Everything you build later comes from these answers, so get real ones. Each question below has options and a short "why it matters" that tells you how the answer changes the build. Some answers reroute the whole plan, so do not skip.

## How to run this

- Ask **one question at a time**, in order. Show the owner the lettered options. Let them pick a letter or answer in their own words.
- After each answer, give the one-line "why it matters" only if it helps them, do not lecture.
- Write every answer into a new file, `PROFILE.md`, in the owner's project. Update it after each answer so nothing is lost.
- Do not start designing or building. This step only gathers answers and sets the route.

---

## SECTION 0: Starting point (ask these first, they shape everything)

**0a. How comfortable are you with tech?**
- (a) Total beginner, I want the AI to do everything
- (b) I have used no-code tools or edited a template before
- (c) I can read and tweak code, I just want the patterns

*Why it matters:* this sets how I talk to you for the whole build. (a) gets slow, plain, step-by-step hand-holding. (c) gets the short version and fewer basics.

**0b. Do you already have a business to run?**
- (a) Yes, a specific product or niche, ready to go
- (b) An idea, but I have not validated it
- (c) Nothing yet, help me pick one

*Why it matters:* (a) we build straight for it. (b) we pressure-test the idea before spending time on it. (c) I help you choose a niche first (I will lean you toward Etsy or print-on-demand, the safest starts).

**0c. Is your AI tool installed on the computer this will run on?**
- (a) Yes, installed and signed in
- (b) No, but I will set it up now
- (c) No, and I would rather just use a browser chat

*Why it matters:* (b) I walk you through installing it before we build. (c) I stop and explain honestly why browser-only makes this much slower and more error-prone (this build creates many files and runs many commands), and I strongly recommend installing the tool. Do not push a different tool, just the setup.

---

## SECTION A: Where it runs (this routes you to 05-infrastructure.md)

**1. The computer you would run this on:**
- (a) A decent, modern machine that can stay on
- (b) An older or slower machine
- (c) A laptop I close at night, or a machine that sleeps or shuts off
- (d) I do not want this tied to my own computer at all

*Why it matters:* (a) points to running locally, the simplest, cheapest, fastest path to a first win (the tradeoff: scheduled jobs pause when the machine sleeps or is off). (b), (c), and (d) point to a cheap always-on server (a VPS). Note the answer, confirm it when you reach 05-infrastructure.md, and do not force a VPS on someone with a good machine.

**2. Do you need it running 24/7, even while you sleep or are away?**
- (a) Yes, always on
- (b) No, only while my computer is on is fine

*Why it matters:* (a) points to a VPS regardless of the machine. (b) means local is fine.

**2b. What operating system is that computer running?**
- (a) Windows
- (b) Mac
- (c) Linux

*Why it matters:* this single answer changes every script I write. All the code samples in this build are bash. On Mac and Linux they run as-is. On Windows I run everything inside WSL (recommended) or Git Bash, translate scripts accordingly, and use Task Scheduler instead of cron for anything scheduled. Note the answer and carry it into 05-infrastructure.md.

---

## SECTION B: Engine and budget (see 06-cost-and-billing.md)

**3. Which AI engine will power your agents?**
- (a) Claude Code Pro, about $20/mo (recommended to start)
- (b) OpenRouter with an open model, pay as you go, no subscription
- (c) Claude Code Max, for heavy use
- (d) Not sure

*Why it matters:* this is how your agents are powered and your monthly floor. (d) means I recommend (a) and move on, do not stall here.

**4. Monthly budget you are comfortable with (engine, paid tools, any ads)?**
- (a) Under $25, lean (engine only, no ads yet)
- (b) $25 to $75 (engine plus some paid tools)
- (c) $75 or more (room for ads and heavier tools)

*Why it matters:* this becomes a hard cap I enforce in code. (a) means we build the smallest thing that ships and skip anything optional.

---

## SECTION C: Your money-maker (this picks your first module)

**5. What do you want your agents to run first? Pick ONE to start.**
- (a) Build a branded Etsy store from scratch (flagship)
- (b) Print-on-demand products
- (c) A content channel that drives traffic
- (d) A dropshipping store
- (e) Flipping: find underpriced items and resell them (no store, sourcing and arbitrage)
- (f) I already sell somewhere else (WooCommerce, BigCommerce, Wix, Squarespace, Amazon, eBay, my own site), or I run a MARKETPLACE where other vendors sell. Run agents on THAT.
- (g) Sell a service (logos, brand kits, social packs, copywriting) on Fiverr or Upwork
- (h) Not sure, help me pick

*Why it matters:* this chooses the first module we build. (a) -> modules/etsy-store-builder.md, which starts with modules/brand-kit.md (it drafts a shop name, logo, palette, and brand voice, then builds the shop identity, a starter product line, listings, and launch content, all as board-gated drafts under one review, ending launch-ready). (b) -> modules/pod-printify.md, (c) -> modules/content-engine.md, (d) -> modules/dropshipping.md, (e) -> modules/flipping.md, a resale sourcing agent that prices local deals against real sold data and drops a ranked buy list for you to approve; a different shape (no storefront) but the same safe agent system. (f) -> **modules/any-store.md**, the universal build: write one small store adapter for whatever platform they are on and every other module works unchanged. If OTHER VENDORS sell on their platform (Dokan, Sharetribe, a custom marketplace), go to **modules/marketplace.md** instead: different business, different agents, their bottleneck is reviewing vendor submissions and keeping vendors, not making products. (g) -> **modules/fiverr-services.md**, agent-assisted service delivery: the agents do the production (brief to spec, concepts, packaging every file format) and the owner still curates and delivers, because Fiverr requires customized work and the seller's own skill in the loop. Tell them up front there is no Fiverr API and no compliant way to automate the platform itself, so this speeds up the work, it does not run the account. (h) -> I recommend the branded Etsy store. We build ONE money-maker first and prove it, then expand later (09-after-first-win.md). Do not try to build several at once.

**If they picked (f), read this before anything else.** They already have the hardest part of a store: real products and somewhere to sell them. Do NOT suggest migrating to Shopify or Etsy, that is a large risky project the agents do not need. Ask which platform, write it into `PROFILE.md`, and go to `modules/any-store.md`. Also tell them the honest sequencing: an owner who already has products is usually short on **demand**, not on storefront, so after the adapter their highest-value module is almost always `modules/content-engine.md`, not more store tooling.

**5b. Do you want the advanced trading module (bot that trades money on the markets)?**
- (a) No / not now (recommended, this is the safe default)
- (b) Yes, I want it built

*Why it matters:* trading is the one money-maker that can lose your money outright, so it is gated. If you pick (b), before I build a single line of it you must acknowledge this in your own words: "I understand I can lose money and this is not financial advice." Record their exact acknowledgement in PROFILE.md with the date. If they do not give it, do NOT build the trading modules, no matter what else they say, and note in PROFILE.md that the trading gate was not passed. Only if this gate is passed are the trading modules ever built: modules/analyst-agent.md (a monitor-only market watcher) and modules/paper-trading.md (a simulated paper portfolio, the riskier of the two). Neither ever places a real trade.

**6. Where are you starting?**
- (a) I already have a shop, store, or channel with some listings or followers
- (b) Starting cold, nothing live yet

*Why it matters:* (a) we build on top of what exists and improve it. (b) we set up the basics first.

**6b. Which platform accounts do you already have, and what country and currency do you sell in?**
- (a) Etsy
- (b) Printify
- (c) Pinterest
- (d) Shopify
- (e) Something else, type it here (WooCommerce, BigCommerce, Wix, Squarespace, Magento, Amazon, eBay, Walmart, Faire, your own site, anything)
- (f) None of these yet

*Why it matters:* let them name every one they already have (more than one letter is fine), plus their country and the currency they price in. Knowing this up front saves a stall mid-build: I will not send you off to create an account halfway through wiring an integration, and I set the right country/currency in listings, pricing, and API calls from the start. Write the list, country, and currency into PROFILE.md.

**(e) is a first-class answer, not a fallback.** Whatever they type, the build is `modules/any-store.md`: one small adapter file for their platform, and every other module (content, marketing, research, manager, memory) runs unchanged. Never tell an owner their platform is unsupported, and never tell them to migrate. The only thing you must do is look up that platform's CURRENT API before building anything (rule 5), and if it has no usable API, use the CSV path in that module.

---

## SECTION D: Time and goal

**7. Realistic time per week for this?**
- (a) A few hours
- (b) Around 10 hours
- (c) Full-time until it works

*Why it matters:* this scopes how much we build at once. (a) means the smallest thing that ships, nothing extra.

**8. Your goal with this?**
- (a) A first sale, proof it works
- (b) Steady side income
- (c) Replace or match a real income

*Why it matters:* it tailors how hard I push on traffic and scaling later. Everyone starts at (a) no matter what, we get one real result before we scale.

---

## SECTION E: Safety and approvals (sets the board, see 07-approval-loops.md)

**9. How tight should approvals be to start?**
- (a) Tight: I approve every publish, every message, every dollar (recommended to start)
- (b) Medium: auto-approve small stuff up to a daily cap, ask me above it
- (c) Loose: run mostly on its own, I check in weekly

*Why it matters:* this sets the board's default gate. (c) is risky early (agents can make mistakes, post the wrong thing, or spend). I will suggest starting at (a) and loosening once you trust it.

**10. Daily cap on anything that costs money?**
- (a) $5 a day (safe starter)
- (b) $10 to $20 a day
- (c) $50 a day
- (d) Set it later

*Why it matters:* this becomes a hard limit in code. An agent that tries to spend past it gets blocked before the money leaves.

**11. Is there anything that must NEVER run automatically without asking you first?**
Open answer. Examples: publishing a listing, sending a message or DM, placing an order, spending money.

*Why it matters:* each of these becomes an approval gate I build before I build the action itself.

---

## SECTION F: What "done for now" looks like

**12. What first win would make this feel worth it?**
- (a) First sale
- (b) First 10 posts published
- (c) First listing or store live
- (d) Something else (describe it)

*Why it matters:* this is our definition of done for round one. We build toward it and then stop, we do not gold-plate past it.

**13. Roughly by when do you want that first win?**
Open answer.

*Why it matters:* it sets the pace and keeps the scope honest.

---

## Close the interview

Once every question is answered and `PROFILE.md` is written:

1. Read `PROFILE.md` back as a short, clear summary: their tech comfort, business, where it runs (and OS), engine and budget, first money-maker, existing accounts and country/currency, starting point, time, goal, approval tightness, spend cap, never-auto list, trading gate (passed or not), and first win.
2. Ask: "Did I get this right? Anything to change before I plan the build?"
3. Fix anything they correct, then save the final `PROFILE.md`.
4. **Start the memory vault now.** `PROFILE.md` is the first thing the system knows about the owner, so it is the first note in their system's memory. Make a `memory/` folder in the project and keep `PROFILE.md` reachable from it. This is the seed of the memory layer (`modules/memory.md`); you are only creating the empty vault and dropping in the profile now, not building the full layer yet. If the owner wants to see and edit what their system knows in a friendly app, this is the moment to point them at Obsidian: install it, "open folder as vault," aim it at the project (or the `memory/` folder). It reads the same plain markdown files the agents will, so nothing is locked in. Optional but recommended, and it means the owner can watch their system's brain fill up from day one. The full vault gets built shortly after, at Step 2 of `03-build-order.md`, with the tools and before the crew, so the agents can read the owner's brand voice and preferences from their first run.
5. **Do not move on until the owner says yes.** When they confirm, go to `05-infrastructure.md` to lock in where it runs, then build in the order in `03-build-order.md`.
