> The build sequence. The SHAPE is the same for every money-maker (foundation, then one real result, then scale, then automation, then the reward), but the SPECIFICS of each early step branch by what the owner chose in the interview. Etsy and print-on-demand build a brand and a store; a content channel builds a look and a posting pipeline; flipping skips the brand and goes straight to sourcing. Safety, the owner's approval and spend cap, is on from the first action in every path. For the AI to follow step by step.

# Build order: a real win first, automation once it works, the reward last

Build in this exact order. Each step has a definition of "working" and a way to prove it (rule 5: prove it works, out loud). Do not move on until you have run the current step and shown the owner.

Why this order: most people fail because the payoff is buried behind invisible plumbing, so they quit before they see anything real. This flips that. The owner sees their money-maker take shape early, gets one real result, takes it live, and the automation that runs it all comes last, once the pieces work. The reward (their own visual world) comes at the very end, as the thing they earn for finishing.

## Route by the owner's money-maker (from `PROFILE.md`, interview Section C)

The steps below give a universal goal, then the specifics for the owner's chosen path. Follow the path that matches their pick:

- **Etsy store, or print-on-demand** -> the brand-and-store path (the flagship). Build a brand, then a store.
- **Content channel** -> the content path. Build a channel look and a posting pipeline that makes and ships content.
- **Flipping / resale** -> the sourcing path. No brand, no store. Go straight to finding deals.

## Two rules that never bend, in every path

- **Nothing risky fires without the owner's yes.** Publishing, sending, spending, buying: every one waits for a human click. Early on, before the automated board exists, this is simply you showing the owner each draft and asking. The habit is on from Step 1.
- **The spend cap is set before the first thing that could spend** (Step 2). You never wire a tool that can cost money before the ceiling that limits it.

Everything comes from the owner's `PROFILE.md`. If a step needs a number or choice you do not have, stop and ask.

**Before Step 1:** the interview (`01-interview.md`) is done, `PROFILE.md` is written, where-it-runs (`05-infrastructure.md`) is settled, and the interview seeded an empty `memory/` folder with the profile in it.

**A note on the queue and board.** Through Steps 1 to 5 you do not have the automated queue, dispatcher, or approval board yet; you build those at Step 6. So in these early steps, "a draft the owner approves" means exactly that: you (the AI) produce the thing, show it to the owner, and wait for their yes before it is saved or goes live. Do not build the task/queue machinery early to do this. The plain version, you showing a draft and the owner approving, is the whole safety model until Step 6 formalizes it.

---

## Step 1: Foundation (the first real thing the owner sees)

The universal goal: make the first concrete piece of the owner's money-maker, safely, so they see real progress in the first sitting. What that piece is depends on the path.

- **Brand-and-store path (Etsy / POD):** build the brand kit from `modules/brand-kit.md`: three to five shop-name options, a logo, a color palette, a brand-voice guide. All drafts the owner approves. Safe (a name and logo publish nothing), and it is the identity every later step reads. A store's brand is function, not vanity (rule 1 carve-out).
- **Content path:** build the channel's identity: a channel name and handle direction, a look (colors, a visual style), and a voice. Lighter than a full store brand, but the same idea, the owner approves it, and it is what keeps every post consistent. Use the brand parts of `modules/brand-kit.md` for this.
- **Flipping path:** no brand. The foundation here is deciding WHAT to flip: with the owner, pick the categories they will hunt (electronics, tools, a specific niche), and their buying rules (max price to risk, minimum spread, how far they will drive, conditions they will not touch). Write these to a plain file. This is the equivalent of a brand: the decided rules everything downstream follows. See `modules/flipping.md`.
- **Existing-store path (they already sell somewhere):** no brand kit, they have one. The first piece is the **store adapter** from `modules/any-store.md`: look up their platform's current API, then write the one small file that creates a DRAFT product in their real store. First win is a real draft product visible in their own admin and absent from their public storefront. Do not migrate them to another platform, and do not let an agent edit or delete anything that already exists.

**Working means:** the owner has approved the first foundation piece (a brand kit, a channel identity, a decided set of flip categories and buying rules, or a working store adapter that produced one real draft).

**Prove it:** show the owner the real thing (the logo, the channel look, or the written buying rules) and get their explicit yes.

**Say to the owner:** "That is the foundation of your [store / channel / flipping]. Everything we build next follows it."

---

## Step 2: Connect your tools, set your spend cap, and stand up memory

The universal goal: wire the outside services this money-maker needs (proving each with one real call), put the hard spend cap in code before anything can spend, and stand up the memory vault the crew will read. The tools and what memory holds branch by path.

**Set the spend cap first, in code, before any tool that could cost money is used**, in every path. Use the owner's number from `PROFILE.md`.

**Stand up the memory vault** (`modules/memory.md`) so the crew reads it. What it holds depends on the path:
- Brand-and-store and content: the brand voice and preferences (no-go words, style, decisions).
- Flipping: the buying rules from Step 1, plus the running lessons of what the owner rejected and why, so the scout gets sharper.

**Wire the tools**, proving each with one real call, key in `.env` (never printed, rule 11):
- **Brand-and-store path:** the Etsy connection, an image maker for product shots, and a print supplier (Printify) if selling POD. Heads-up: a new Etsy developer key can sit pending on Etsy's side for a while; that is a wait, not a bug.
- **Content path:** the owner's social accounts through the posting tool (Zernio), and the video maker (Higgsfield), plus Meshy if they want a signature 3D visual. See `tools/tools.md`. This is where the content tools that will produce their posts get connected.
- **Flipping path:** the eBay developer key for sold-price comps (free, sanctioned), and the buy-side sources. Lead with the safe ones (eBay auctions ending soon, auction sites that want bidders). If the owner accepts the risks, wire a no-login local marketplace fetcher (Facebook, and note Craigslist is a dead end); read the honest warning in `modules/flipping.md` first. This is the "get straight to sourcing" step for the flipping path.

**Working means:** every tool the path needs answers a real test call, the spend cap refuses to be crossed, and the vault holds the right foundation (brand voice, or buying rules) in plain files an agent can read.

**Prove it:** show one successful call per tool, set a low test cap and show the refusal, and open the vault to show the foundation sitting there.

**Say to the owner:** "Everything is connected and tested, you have a hard spending limit I cannot cross, and your system remembers what matters. The workers we build next will use all of it."

---

## Step 3: Your crew (the agents)

The universal goal: build the small crew of agents this money-maker needs, each doing one job, reading memory, using the tools, producing drafts the owner approves, never acting alone. Follow the shape in `04-first-agent.md`: tight prompt, only the tools that job needs, fresh session. Run them by hand for now (no automation yet, so a human is always in front of everything).

- **Brand-and-store path:** a listing writer, an image/design generator, a pricing helper, a manager.
- **Content path:** a content writer (drafts the hook and caption) and an asset maker (turns the brief into a video via Higgsfield, or the signature 3D visual). See `modules/content-engine.md`.
- **Flipping path:** the sourcing scout (reads the fetched listings, prices them against sold comps, ranks the real buys), and optionally a lister for reselling what the owner buys. See `modules/flipping.md`.

**Name the crew (a small thing that makes it feel like theirs).** Before you finish, offer the owner three ways to name their agents. It changes nothing functional; a named crew is just more fun to run and easier to talk about ("Ember queued three listings"). Let them:

1. **Pick a theme** from the four below, and each agent takes the matching name for its role.
2. **Name each one themselves**, whatever they like.
3. **Skip it** and keep plain role names (the listing agent, the scout).

| Role | Hearth | Frontier | Cosmos | Studio |
|---|---|---|---|---|
| Manager (plans, delegates) | Ember | Forge | Nova | Chief |
| Scout (research, trends, deals) | Maple | Ranger | Probe | Scout |
| Writer (listings, captions) | Quill | Scribe | Echo | Copy |
| Maker (images, video, design) | Iris | Chisel | Prism | Art |
| Numbers (pricing, analytics) | Penny | Tally | Orbit | Ledger |

Use only the roles the owner's money-maker actually has (a flipping owner may have just a scout and a numbers agent). Write the chosen names into the memory vault so every agent, and the manager's daily brief later, refers to each other by name. If the owner does not care, do not push it; plain role names are completely fine.

**Working means:** each agent runs manually, reads memory, and produces its draft in the owner's voice or by their rules. None can publish, spend, or buy; they only draft.

**Prove it:** run two or three of them on real inputs and show the outputs.

**Say to the owner:** "That is your crew. Each one drafts, you approve. Nothing goes live, gets sent, or gets bought without your yes."

---

## Step 4: Your first real result (prove the whole chain)

The universal goal: put it all together on ONE real thing, end to end, so the owner sees the whole system work before scaling.

- **Brand-and-store path:** the crew produces one complete listing, image, title, description in the owner's voice, tags, price, as a draft the owner approves by hand.
- **Content path:** the crew produces one complete post, the caption and the real visual (Higgsfield makes the video from the brief), as a draft the owner approves. This is where Higgsfield actually produces their content, on one real piece.
- **Flipping path:** the scout produces one real ranked buy list from live data, each candidate priced against a real sold comp with the spread after costs. The owner reviews it. A list with zero worth buying is a correct result, not a failure.

**Working means:** one real output (a listing, a post, or a ranked buy list), built by the crew, sits ready for the owner's approval.

**Prove it:** show the real artifact as it will actually read or look, not a summary (approving from a summary is a logged mistake, `08-gotchas.md`).

**Say to the owner:** "That is the whole system working on one real [listing / post / set of deals]. Next we scale it."

---

## Step 5: Scale it, and go live

The universal goal: take the proven first result and make it a real, running operation, still approved by hand.

- **Brand-and-store path:** build out the full store from the brand (`modules/etsy-store-builder.md`): shop identity, a starter product line, the rest of the listings, launch content, all as drafts under one review. The owner approves and opens the shop (the store-builder has the full account walkthrough: seller account, getting paid, paying Etsy's fees). Read the compliance section of `08-gotchas.md` before anything publishes.
- **Content path:** turn on the daily cadence (`modules/content-engine.md`): one post a day, each drafted, visualized, approved, and shipped everywhere at once through Zernio. The owner approves the first several by hand.
- **Flipping path:** run the sourcing on a schedule so a fresh ranked list arrives regularly, and the owner acts on the first real flip (buys it themselves, the system never buys). Reselling reuses a listing agent or is done by hand at first.
- **Existing-store path:** the adapter works, so the storefront is done. Their bottleneck is almost never more store tooling, it is **demand**. Go straight to `modules/content-engine.md` for the daily cadence, then `modules/marketing.md` for email and ad tests. Add the listing agent from `modules/dropshipping.md` pointed at their adapter only when they actually want new products written. Say this to them plainly: you already have product, what you are short on is traffic.

**Working means:** the money-maker is a running operation the owner approves (a live store, a daily posting pipeline, a recurring buy list they act on, or their existing store now fed by agents).

**Prove it:** show it live: the open shop, the first shipped post on the real feeds, or the recurring list producing real candidates.

**Say to the owner:** "That is your [store / channel / flipping] running for real. Now we make it run on its own."

---

## Step 6: Build the system (the automation that runs it unattended)

**First, move the owner to VS Code if they are still in the Claude Code app.** This is the step that needs it: the dispatcher is a long-running process you start and keep alive, which wants a real terminal, and VS Code shows the owner their files alongside it. Explain why in plain words (`SETUP.md` has the walkthrough): "Now we are building the piece that runs everything on its own. For this we need VS Code, a free app that gives us a terminal and shows your files. A few minutes, and I will walk you through every click." A Mac or Linux owner is ready after the install; a Windows owner also switches VS Code's terminal to WSL (in `SETUP.md`).

The universal goal, same for every path: until now the owner has run each agent by hand and approved each draft in person. That is safe, and it is why nothing could run away. But it does not scale and it does not run while they sleep. Now build the engine that automates it, from `02-architecture.md`:

- **The queue:** a task list (one SQLite table). Adding work is inserting a row.
- **The dispatcher:** a single loop that pulls one task, spawns one fresh agent, times out, and recovers orphans. Only one copy runs at a time (a lockfile guards it).
- **The board:** the real approval surface. Risky tasks stop as `needs_approval` and wait for the owner's click, exactly like the by-hand approval they have been doing, now formalized.

You build this now, not first, because the pieces it runs (the money-maker, the agents, the tools) already exist and are proven. Whatever the path, the automation just wires the same agents to run on their own. Harden the guardrails here too (full spend caps and stop conditions, `06-cost-and-billing.md`) and fold the ongoing lessons into the memory vault.

**Working means:** you queue several tasks, the dispatcher works through them, safe ones finish, risky ones park on the board, a killed task recovers on restart, and a cap stops the system with a clear message.

**Prove it:** queue three tasks, run the dispatcher, show them flow through. Queue a risky one, show it wait. Kill a task mid-run, restart, show it recovered. Set a low cap, show the refusal.

**Say to the owner:** "Now it runs itself. Jobs go in a list, the manager works through them, and anything risky still waits for your yes. This is the part that works while you are away, and it has hard limits it cannot cross."

---

## Step 7: The reward, and looks

The owner has earned this. Their money-maker is built, live, and running on its own. Only now, and as the payoff for finishing, do you build the fun part.

**The reward: their own visual world.** This is the thing that makes the whole system feel like theirs. Build the owner a signature visual world with Meshy (the Meshy section of `tools/tools.md`): their own recurring 3D look. For a content channel, it is the signature backdrop every video carries, the unfair-advantage brand from `tools/tools.md`. For any owner, it is the reward that turns a working system into one with a face. Frame it to them exactly that way: "You finished. This is the part you earned, your own world." It comes last on purpose (the vanity-trap rule): it is the reward for a system that already works, never a substitute for building one.

**And the polish:** a real approval dashboard (`07-approval-loops.md`), a theme, a name for the tool. You are changing how it looks, not whether it works.

**Working means:** the reward and the polish sit on top of a system that already earns and runs. Every proof from Steps 1 to 6 still passes.

After this, `09-after-first-win.md` covers what is next: a second money-maker, scale, or stop.

---

## NUDGE ONCE
This order gives the owner real wins early (their foundation at Step 1, a real result at Step 4, a live operation at Step 5) and saves the reward (their visual world) for the end, so the pressure to jump to the fun part is low. But if the owner still reaches for the reward or a fancy dashboard before the system runs, give them your honest advice once, then respect their call (rule 1).

**Your response, plainly:**

> "My honest advice: the money-maker is what pays, and the automation is what saves you time. Let us finish those first, then build your world as the reward. Every hour on the looks before it runs is an hour it is not working for you. But it is your build. Want me to do it now, or after it runs on its own?"

Say it once, then follow their lead.

## The one rule underneath all of this
Prove each step runs before you move on. No "this should work." Run it, show the real output, then build the next part. A real win first, safety always on, automation once it works, the reward last. Every time.
