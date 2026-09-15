> For the AI building the owner's Agentic OS. Build a content agent that turns the owner's real business into a steady stream of short posts, makes the visual, and ships one piece to every platform at once, with a board gate before anything goes public.

Level: intermediate | Time: ~2 evenings | Cost: metered video/3D credits per asset (set a daily cap), plus your engine

# Module: the content engine

Content is how the money-maker gets seen. A product with no traffic makes no sales. This module builds an agent that takes the owner's real business (their products, their sales, their vibe) and turns it into short social posts, makes the visual to go with each one, and ships a single piece to every platform in one shot. The owner approves each post before it goes live.

The model is simple and it is the whole point: **make one piece, ship it everywhere, once a day.** Do not build a per-platform content team. Build one pipeline that posts once and fans out. I run my own Etsy shop on this exact loop, and the same content engine feeds my agency's channels. One piece a day, approved by hand, shipped everywhere.

**Timing note.** If the content channel IS the owner's money-maker, you build this across Steps 3 to 5 of `03-build-order.md`, which is BEFORE the automated queue and dispatcher exist (that is Step 6). In that case, run the content agents BY HAND for now: you produce each post and its visual, show the owner, and they approve it directly. The task/queue language below (`draft_post`, `make_asset`, the dispatcher) describes how it works ONCE the system is built at Step 6; before then, the same steps happen by hand with the owner approving each one. If instead the owner already has a store and is adding content as a second money-maker (`09-after-first-win.md`), the spine already exists and you wire these as real task types from the start. Either way, content needs something real behind it: build or have the money-maker first, then point content at it.

## What you are building

Three task types and one board gate, wired in sequence:

1. **CONTENT** (`draft_post`): an agent that writes the hook and caption from real business data.
2. **ASSET** (`make_asset`): a step that produces the visual (a video, or a signature 3D image).
3. **SCHEDULE/POST** (`ship_post`): a step that pushes the approved piece to every platform at once.

Between the asset and the post is the **board**. Nothing goes public without a click.

```
real business data --> CONTENT agent --> ASSET step --> BOARD --> POST step
  (products, sales)     (hook +          (video or      (owner    (ship to ALL
                         caption +        3D visual)      approves)  platforms once)
                         visual brief)
```

## Step 0: get the accounts and connections ready (do this FIRST)

Just like the Etsy path cannot skip opening the actual shop, the content path cannot skip setting up the actual accounts. The pipeline below is worthless until there is somewhere real to post and something real to make the visuals. This is a manual owner step and it comes before any agent. Walk the owner through the whole thing plainly, do not just say "connect your socials." Nothing here posts anything; this is only plumbing.

**1. Create the channel identity (the "shop" of the content path).** The owner needs one consistent handle and look across platforms, the content equivalent of the brand kit. Pick one handle and check it is free on every platform at once (a name taken on TikTok but free on IG is a problem later). Set the same display name, profile image, and one-line bio everywhere. This is why, on the content path, Step 1 of `03-build-order.md` builds the channel look before anything else: the accounts need it.

**2. Open the platform accounts the owner will actually post to.** Create a free account on each target platform and, crucially, set the right account TYPE, because the automation later depends on it:
  1. **TikTok**: a free account; switch it to a **Business** account (Settings, Account, Switch to Business Account). Business type is what unlocks posting through a tool later.
  2. **Instagram**: a free account, switched to a **Professional/Business** account, and linked to a **Facebook Page** (Instagram requires the Page link for any third-party posting tool to work). This trips people up, so do it now, not later.
  3. **Facebook**: a **Page** (not just a personal profile), created from the personal account. The Page is what tools post to.
  4. **YouTube**: a channel on a free Google account (this is the same Google login used for other tools, keep it straight).
  5. **Pinterest**: a free **Business** account (Pinterest has a one-click convert-to-business option).
  Have the owner log into each one successfully before moving on. A forgotten password here stalls the whole build.

**3. Connect every account to the post-to-all tool (Zernio).** This is the content path's version of wiring Etsy Payments: the one connection that makes shipping-everywhere possible. In the Zernio dashboard (`tools/tools.md`), the owner links each platform account above by logging in through Zernio's connect flow, one platform at a time, and authorizing it. This is an owner action (it is their login and their consent), not something the AI does with a pasted password (never solicit a password in chat). Confirm each platform shows "connected" in Zernio before trusting the post step. If a platform will not connect, it is almost always the account-type miss from step 2 (personal instead of Business/Page).

**4. Set up the visual-generation accounts.** The asset step needs credits somewhere:
  - **Higgsfield** for video (`tools/tools.md`): the owner creates an account and puts a payment method on file, because video generation is metered. This is where the daily spend cap matters, set it before the first generate.
  - **Meshy** for the signature 3D visual (`tools/tools.md`), only if the owner wants the 3D differentiator. Optional; skip it if they are video-only to start.
  Put every API key or token in the owner's `.env` file directly (they place it, never pasted in chat), the same secrets rule as every other tool.

Until all four are done there is nowhere to post, no way to make the visual, and the pipeline below has nothing real behind it. Get accounts and connections green first, then build the agents.

## Step 1: the CONTENT agent (draft the post)

Build an agent task, `draft_post`, that writes one short post. The rule that makes it good: **it drafts from real data, not from thin air.** Feed it something true about the business and make it write around that.

Good sources of a real hook, pulled from the owner's own system:

- A product ("the cozy-season enamel mug just went live").
- A recent sale or milestone ("hit 50 orders this week", read straight from the Etsy digest in `modules/etsy-autopilot.md`).
- A behind-the-scenes detail ("how this design gets made").
- A seasonal tie-in that matches the niche.

### The agent prompt shape (build it exactly like this)

Keep the same one-task-one-fresh-agent pattern from `04-first-agent.md`. The `draft_post` prompt has four fixed parts. Do not improvise the shape; the exact output format is what lets the next step and the board read it without a human in the loop.

**Role (system line):** "You are the content writer for the owner's own [niche] brand. Write in the brand voice pulled from their real listings. You draft one short social post from one true fact about the business. You do not post anything."

**Input payload (what the dispatcher hands it):** a small JSON object with the topic and the brand context, nothing else.

```json
{
  "topic": { "kind": "product", "name": "Cozy Season Enamel Mug", "detail": "just went live, 12oz, matte finish" },
  "brand_voice": "warm, unhurried, homey. never hypey. sounds like a friend who loves fall.",
  "niche": "cozy-season home goods",
  "recent_lessons": ["captions over ~150 chars get cut on IG", "no more than 5 hashtags reads cleaner"]
}
```

The `recent_lessons` lines come from `lessons.md` (rule: feed the agent its own past mistakes so it stops repeating them).

**Exact output format (the agent must return this and only this):** a single JSON object the board and the asset step both parse.

```json
{
  "hook": "It finally smells like fall in here.",
  "caption": "The Cozy Season enamel mug is live. 12oz, matte, built for the third refill. Link in bio if your mornings need it.",
  "hashtags": ["#cozyseason", "#fallvibes", "#enamelmug", "#slowmornings", "#etsyfinds"],
  "visual_brief": "12oz matte cream enamel mug on a wooden table by a rain-streaked window, steam rising, warm morning light, slow push-in.",
  "platforms": ["tiktok", "instagram_reels", "youtube_shorts", "facebook", "pinterest"]
}
```

**Stop condition:** exactly one post object, or `{"skip": true, "reason": "..."}` if the topic is thin and not worth a post today. One piece, not five. Do not pad. Write the JSON to `result` and set the task done. Drafting posts nothing, so this runs straight from the queue with no approval.

Give this agent read access to the product/sale data and write access to its result file, nothing more (rule 2). Keep the voice tied to the owner's brand from `PROFILE.md`. A cozy Etsy shop does not post like a hustle-bro. Pull the tone from their real listings.

## Step 2: the ASSET step (make the visual)

A caption alone dies in the feed. Every post needs a visual, and the `visual_brief` from Step 1 is the exact instruction for it. Build a `make_asset` task that reads that brief and produces one. Two paths, pick per post:

- **Video.** Short vertical video is the default for reach. Feed the `visual_brief` straight into the Higgsfield section of `tools/tools.md`, which covers AI video generation. In the worked example above, the brief "12oz matte cream enamel mug ... steam rising ... slow push-in" becomes the Higgsfield prompt. Higgsfield returns a 3 to 10 second clip. Store it the way the whole system stores artifacts: the post JSON goes in the task's `result` column, and the clip (a large binary) is written as a file under `board/pending/<task_id>/` with its path recorded in `result.preview` so the board can find it. Video is what the "post once, ship everywhere" platforms (TikTok, Reels, Shorts) reward most.
- **Signature 3D visual.** The differentiator. Instead of the same stock-looking graphics everyone posts, the owner can build their own recurring 3D "world" or object and feature it. Wire this through the Meshy section of `tools/tools.md`. Use the owner's signature render as the reference image Higgsfield animates, or as the backdrop the mug sits in, so every clip carries the same recognizable look. A signature visual that shows up across every post is a brand people start to recognize.

Default to video for volume, and use the 3D visual for the pieces that carry the brand. Do not generate a fresh look every single time; a repeated signature beats novelty for recognition.

The asset step writes the clip path into `result.preview` and carries it on the post object too, so the board has the caption and the visual together (`preview` is the path the board renders):

```json
{
  "hook": "It finally smells like fall in here.",
  "caption": "The Cozy Season enamel mug is live. 12oz, matte, built for the third refill. Link in bio if your mornings need it.",
  "hashtags": ["#cozyseason", "#fallvibes", "#enamelmug", "#slowmornings", "#etsyfinds"],
  "preview": "board/pending/57/cozy-mug-2026-09-14.mp4",
  "platforms": ["tiktok", "instagram_reels", "youtube_shorts", "facebook", "pinterest"],
  "status": "needs_approval"
}
```

Asset generation can cost money or API credits. Treat it as a metered action: cap how many assets get made per day (see guardrails). Generating is not posting, so it does not need board approval by itself, but it does count against the spend cap.

## Step 3: the board gate (approve before anything ships)

The finished piece (hook, caption, hashtags, visual) parks on the board with status `needs_approval`. Show it as a real preview card, not raw JSON, so the owner sees exactly what will go out:

```
[57] ship_post  ->  "It finally smells like fall in here."
                    caption: The Cozy Season enamel mug is live. 12oz, matte...
                    5 hashtags | video: cozy-mug-2026-09-14.mp4 (plays inline)
                    ships to: tiktok, ig reels, shorts, facebook, pinterest
                    approve? (y/n)
```

The owner sees the caption and the visual together and decides: ship it, or reject it. This is not optional and it is not automatic. Posting publicly under the owner's name is a risky, hard-to-undo action, exactly the kind rule 3 says never auto-runs. A bad post can embarrass the brand or break a platform's rules. The human clicks, or it does not go out.

Always let the owner actually watch the clip on the board before approving, never approve from the caption text alone (the asset is where AI generation drifts). On reject, set the task to `rejected` with a note, and append a line to `lessons.md` if the reason is reusable, so the CONTENT agent stops repeating it ("stop over-describing the product, lead with the feeling").

## Step 4: the POST step (ship everywhere, once)

Here is the leverage. Instead of logging into ten apps, the post step ships the approved piece to every platform in one call. Wire this through the Zernio section of `tools/tools.md`, the post-to-all-platforms tool: the one clip and caption go out to all the connected platforms (TikTok, Instagram, YouTube, Facebook, and the rest) at once. Zernio has an MCP server that plugs straight into the agent, so the post step is a single tool call, not a per-platform loop you maintain. The `result.preview` clip path and `caption` and `platforms` from the approved post object are exactly what Zernio takes.

This is the "post once, ship everywhere" model made real. The owner creates one piece a day and it lands everywhere their audience is. That is the whole reason to build a content engine instead of posting by hand.

The post step runs only on approval. It is the follow-up task the board triggers, never a job the queue runs on its own.

## The full worked cycle (one real day)

This is the end-to-end path, start to finish, for one post:

1. 8 AM: the daily job enqueues a `draft_post` with `{"kind":"product","name":"Cozy Season Enamel Mug"}` (pulled from the newest live listing).
2. The CONTENT agent returns the post JSON: hook, caption, 5 hashtags, visual brief, platforms.
3. The `make_asset` task reads the `visual_brief`, calls Higgsfield (the Higgsfield section of `tools/tools.md`), writes the clip under `board/pending/57/cozy-mug-2026-09-14.mp4`, records that path in `result.preview`, and sets the task to `needs_approval`.
4. The owner opens the board, watches the 6-second clip, reads the caption, and clicks approve.
5. The `ship_post` follow-up hands the clip and caption to Zernio (the Zernio section of `tools/tools.md`), which fans it out to all five platforms in one call.
6. Done. One piece, one approval, everywhere. Tomorrow it runs again with the next topic.

That is the whole engine. Everything else is caps and cleanup.

## Cadence: one a day

Set the target cadence to **one post per day.** That is realistic, sustainable, and enough to build presence without burning credits or spamming. It also fits the pipeline cleanly: one topic in, one piece out, one approval, one ship.

Schedule it the simplest way that works: a daily job (cron, a scheduled task, or a "post due" check in the dispatcher) enqueues one `draft_post` task each morning with a fresh topic. The topic can come from a rotating list the owner seeds, or from the newest product or sale in their system. Do not build a complex content calendar on day one. One a day, pulled from real business data, is the whole engine.

If the owner wants to push harder later, raise the cadence then. Start at one. A daily post the owner actually approves beats a firehose of drafts nobody ships.

## Guardrails for this module

- **No auto-post. Ever.** Every public post routes through the board. The post step is a board-triggered follow-up, not a queued job. Build this gate before you build the posting call (rule 3).
- **Spend and rate cap.** Asset generation (video, 3D) costs money or credits. Set a hard daily cap: max assets generated per day and max spend per day. At one post a day the natural cap is one asset a day; enforce it in code so a bug or a retry loop cannot run up a bill. Ask the owner for the numbers in the interview (rule 4).
- **Rate-limit the posting too.** Cap posts per day (default: one) so an accidental re-queue does not blast the same thing five times across every platform.
- **Draft is safe, ship is not.** Drafting and asset generation can run from the queue. Publishing to social cannot. Keep that line sharp.
- **Stay on the right side of each platform.** The owner is responsible for following the terms of service and posting rules of every platform Zernio ships to. Do not build anything that spams, fakes engagement, or violates a platform's automation rules.

## Prove it works

Before you call this done: run one full cycle end to end. Enqueue a `draft_post` for a real product, let the CONTENT agent write it, generate one real asset, park it on the board, and show the owner the caption and the visual. Then, on their explicit approval, ship exactly one post through Zernio and confirm it actually landed on the platforms. Do not claim the engine works until you have seen a real post go live from a real approval (rule 5).

## Note on results

Content is a slow compounding play, not a switch. Posting daily does not guarantee traffic or sales; reach depends on the niche, the quality, the platforms, and timing outside anyone's control. Results are not typical. Build the engine, keep the cadence, and judge it over weeks, not days.
