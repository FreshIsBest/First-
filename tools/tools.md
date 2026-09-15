> For the buyer's AI. The outside tools this system wires to its agents. The content trio comes first: Zernio (post once, ship everywhere), Higgsfield (AI video), and Meshy (your signature 3D world). Then the rest of the registry: market data for the analyst and paper-trading agents, sports odds for a paper sportsbook, a freelance assist tool, and a voice ("Jarvis") layer for talking to the manager. Wire each only when a module calls for it, verify current docs, put every key in `.env` (never in chat), and keep every real action behind the approval board. Two lines never move: nothing publishes or acts on its own, and nothing here ever connects to a brokerage or bookmaker with power to move real money.

# The tools

## Zernio: post once, ship everywhere

Zernio is a post-to-all-platforms API. You give it one video or one post and it publishes to up to 15 platforms in a single call: TikTok, YouTube, Instagram Reels, Instagram feed, X, Facebook, LinkedIn, and more. The owner's content channel does not need a human copy-pasting the same clip into fifteen apps. The content agent hands the clip to Zernio and Zernio fans it out.

The reason this matters for this build: Zernio ships an MCP server (280+ tools) that plugs straight into the owner's AI. Once it is wired, the content agent can post everywhere itself, no glue code to maintain. That is the whole point. You are connecting the posting muscle directly to the agents you already built.

This file gets used by `modules/content-engine.md`. When that module needs to publish, it comes here.

## The one guardrail that never bends

Posting is a risky action. It is public, it spends the owner's reputation, and it cannot be un-posted cleanly. So it obeys rule 3 from `CLAUDE.md`: **nothing publishes automatically.**

The flow is always: the content agent generates the post and drops it in `board/pending/`. The owner looks at it on the approval board and clicks approve. Only an approved post gets sent to Zernio. The agent never calls Zernio's publish endpoint on its own. Build the board check before you build the posting call.

Zernio's own board and scheduling features do not replace this. Even if Zernio would let the agent publish directly, you route through the owner's approval first. The owner sees every post before the feed does.

## Step 1: sign up

Send the owner to sign up for Zernio (docs at docs.zernio.com). Facts to tell them plainly:

- **Free tier:** 2 connected accounts, no credit card required. That is enough to prove the whole flow before anyone pays.
- **Paid:** roughly $6 per connected account per month after that. So if the owner wants to post to five platforms, budget about that per account, per month. Confirm current pricing on their site before you quote it as fact; this is a third-party cost, not part of the Blueprint.
- Do not ask the owner to paste their Zernio API key into the chat. Have them put it in the project's `.env` file themselves, and read it from there. (This is a standing rule: secrets go in files, never in the conversation.)

## Step 2: connect the owner's accounts

Inside Zernio's dashboard the owner connects their social accounts (TikTok, YouTube, IG, X, etc.) through Zernio's OAuth. This is a one-time click-through per platform and the owner has to do it; it needs their logins, so it is not something you automate. Walk them through it, then confirm each account shows as connected in the Zernio dashboard before you rely on it.

Start with the two free-tier accounts. Prove the flow works on those before the owner pays to connect more.

## Step 3: the basic "post once, ship everywhere" call

The plain API shape is: one authenticated request with the media and caption plus the list of target platforms. Check docs.zernio.com for the exact endpoint and field names, then build the call to match. The idea:

```bash
# Illustrative. Confirm the real endpoint and fields at docs.zernio.com.
curl -X POST "https://api.zernio.com/v1/posts" \
  -H "Authorization: Bearer $ZERNIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "media_url": "https://.../clip.mp4",
    "caption": "approved caption text here",
    "platforms": ["tiktok", "youtube", "instagram_reels", "x"]
  }'
```

One call, four platforms (or up to fifteen). That is the leverage. But remember: this call only runs on a post the owner already approved on the board. It is not wired into the agent as an auto-action.

## Step 4: plug the MCP server into Claude Code

This is the part that makes the content agent self-sufficient. Zernio's MCP server exposes its 280+ tools to the owner's AI, so the agent can list connected accounts, build a post, and publish (to approved posts only) without you hand-writing HTTP calls.

To wire it into Claude Code, add the Zernio MCP server to the owner's MCP config. Check docs.zernio.com for the current server command and required env vars, then register it. The shape looks like this:

```json
// .mcp.json (or the owner's Claude Code MCP config). Confirm exact server details at docs.zernio.com.
{
  "mcpServers": {
    "zernio": {
      "command": "npx",
      "args": ["-y", "@zernio/mcp-server"],
      "env": { "ZERNIO_API_KEY": "${ZERNIO_API_KEY}" }
    }
  }
}
```

After it is registered, restart Claude Code and confirm the Zernio tools show up. Then the content agent can be given a whitelist that includes only the Zernio tools it needs (list accounts, create post, publish) and nothing else, the same way every agent gets a tight tool list in `04-first-agent.md`.

Important: giving the agent the publish tool does not mean the agent publishes freely. The agent's job is to prepare the post and put it in `board/pending/`. The publish tool only fires on an approved item. Keep the human click between the draft and the feed.

## Step 5: prove it, out loud

Do not tell the owner posting works. Show them (rule 5 from `CLAUDE.md`):

- Connect the two free-tier accounts and confirm they show connected.
- Take one real approved post and push it through, either the raw API call or the MCP tool.
- Show the owner the actual response and the live posts on each platform.
- Only after you have seen it land on the real feeds do you call it wired.

If it took two tries or one platform rejected the media, say so plainly and fix it before moving on.

## Recap

- Zernio = one call, up to 15 platforms. Its MCP server plugs the posting muscle straight into the owner's agents.
- Free tier: 2 accounts, no card. Then about $6 per account per month. Confirm current pricing and API details at docs.zernio.com.
- Secrets go in `.env`, never in chat.
- Every post routes through the owner's approval board first. The agent prepares; the owner approves; then it ships. No auto-publish, ever.
- `modules/content-engine.md` is what drives this; come back here when it needs to post.

## Higgsfield: AI video for the content engine

Higgsfield is an AI video generator. You give it a prompt (and often a reference image) and it returns a short video clip: motion, camera moves, faceless b-roll. It is a way to produce raw footage for the owner's content channel without a camera, a face, or a set.

This file feeds `modules/content-engine.md`. The content engine needs clips; Higgsfield makes them. It sits upstream of the Zernio section of `tools/tools.md`: Higgsfield produces the clip, the owner approves it, then Zernio ships it.

## What it is good for (and what it is not)

Good for:
- **Short faceless clips.** 3 to 10 second pieces the content engine can cut into a post.
- **Motion and camera moves.** A slow push-in, a pan, a subtle loop. Movement stops the scroll.
- **B-roll for hooks.** The first second or two of a video, the visual that grabs attention before the message lands.

Not good for:
- Long-form video. Keep it short. Higgsfield is for clips, not two-minute films.
- Anything the owner needs to be exact and repeatable frame for frame. AI generation drifts. Treat each clip as one option, not a precise render.

Do not oversell this to the owner. It makes good short clips fast. It is not a film studio. Set that expectation.

## The basic flow: prompt to clip to posting tool

1. **Write a tight prompt.** Describe the shot: subject, motion, mood, length. Short and concrete beats long and vague, same as agent prompts. If the owner has a reference image or their signature 3D asset (see the Meshy section of `tools/tools.md`), feed it in so the clip stays on-brand.
2. **Generate a short, hook-first clip.** Aim for a strong first second. The opening frame is what decides whether anyone watches. Generate the clip that earns the scroll-stop, not a slow build-up.
3. **Save the clip to the owner's project**, in a drafts folder the board can see (for example `board/pending/`), not straight to a posting queue.
4. **Hand it to the posting tool.** Once the owner approves the clip, the Zernio section of `tools/tools.md` ships it. The content engine ties the clip to its caption and platforms.

Keep clips short and hook-first every time. A long clip with a slow open is a wasted generation. If the first second does not grab, regenerate with a tighter prompt.

## Wiring it up: check the current docs

Higgsfield's exact API, endpoints, model names, and pricing change. Do not build against remembered details. Before you wire anything:

- Look up Higgsfield's **current** API docs and confirm the real endpoint, request fields, auth method, and output format.
- Build the generation call to match what the docs say today, not what this file assumes.
- Higgsfield is a third-party paid tool. Tell the owner it costs money to generate (confirm current pricing on their site) and that generation is metered, so wasted generations cost real money. That is another reason to keep prompts tight and clips short.
- Put the API key in `.env`, read it from there. Never ask the owner to paste a key into the chat.

The shape of a generation step, once you have confirmed the real API:

```bash
# Illustrative only. Confirm the real endpoint and fields in Higgsfield's current docs.
curl -X POST "https://api.higgsfield.ai/v1/generate" \
  -H "Authorization: Bearer $HIGGSFIELD_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "slow push-in on a neon pixel city street at night, looping, 6 seconds",
    "reference_image": "assets/brand-city.png",
    "duration": 6
  }'
# Then save the returned clip into board/pending/ for the owner to approve.
```

## The guardrail: every clip is a draft

A generated clip is a draft, not a published post. It goes to the approval board and waits for the owner's eyes, same rule as everything else in this system (rule 3 in `CLAUDE.md`).

Why this matters extra with AI video: generation drifts. A clip can come out warped, off-brand, or with an artifact the prompt never asked for. The owner has to see it before it touches a feed. The agent generates and stages; the owner approves; only then does the Zernio section of `tools/tools.md` ship it. Never let a raw generation auto-post.

## Prove it

Do not tell the owner video generation works. Show them:
- Run one real generation with a real prompt.
- Show the actual clip it produced and where it saved.
- Say plainly whether it is usable. If it took a few tries to get a clean clip, say that; AI video often does.

Once the owner has a clip they approve, hand it to the content engine and the posting tool.

## Meshy: your signature 3D world

This is the differentiator. Anyone can generate AI clips. What makes faceless content actually stand out is a **recurring visual the owner owns**: a signature 3D asset or world that shows up in every single video. Mine is a 3D pixel city. It appears in all my content, so the moment someone sees it in their feed, they know whose it is. That is a brand you cannot buy and competitors cannot copy fast.

Meshy AI is how the owner builds that world. It turns text or images into 3D models. You use it to generate the owner's signature assets, assemble them into a scene, and render that scene as the recurring backdrop the content engine drops into every post.

This file feeds `modules/content-engine.md`. The world you build here is the visual spine of everything the content engine publishes.

## Why a 3D world is the unfair advantage

Faceless content has one hard problem: nothing ties the videos together, so nothing compounds. Every post looks like a stranger's. A signature 3D world fixes that:

- **Instant recognition.** Same world in every clip. The scroll stops because the viewer already knows the brand.
- **Consistency for free.** Once the world is built, every video inherits the same look. No art direction per post.
- **Ownable.** It is the owner's world, generated to their taste. A competitor cannot lift it without it obviously being theirs.

This is the point of the whole content play. Make the world once, then reuse it forever. Time spent here pays back on every post the owner ever ships.

One honest note, per the build rules: this is still looks, and looks come after function (the vanity-trap rule in `CLAUDE.md`). Build the world when the content engine is actually posting and earning attention, not on day one instead of shipping. The 3D world makes a working content channel unmistakable; it does not replace having one.

## What Meshy does

- **Text to 3D.** Describe an object, get a 3D model.
- **Image to 3D.** Feed a reference image, get a model that matches it.
- Output is a 3D model file (mesh plus texture) the owner can pose, light, and render in a scene.

Exact formats, model options, API, and pricing change. Meshy is a third-party paid tool. Before you wire anything, check Meshy's **current** docs for the real API, endpoints, output formats, and cost, and tell the owner what generation will cost. Put the API key in `.env`, never in the chat.

## The flow: assets to world to backdrop

1. **Define the world with the owner.** One sentence: what is their signature world? A pixel city, a tiny desk diorama, a floating island, a neon workshop. It should fit their business and be simple enough to recur. Get this decided before generating anything.
2. **Generate the assets.** Use Meshy to make the pieces: the buildings, the props, the recurring character or object. Generate a few options per asset and let the owner pick. Keep a consistent style prompt so the pieces match.
3. **Assemble the scene.** Arrange the assets into one world (in whatever 3D tool the owner is comfortable with, or the pipeline the current Meshy docs support). This is the set every video is shot on.
4. **Render the recurring backdrop.** Produce the reusable renders: a hero shot, a few camera angles, a loop. These are the brand backdrop the content engine reuses across posts. Save them into the owner's project where the content engine and the Higgsfield section of `tools/tools.md` can pull them (for example an `assets/` folder).
5. **Hand renders to the content engine.** The signature render can be a static backdrop, the reference image Higgsfield animates, or the visual the caption sits over. `modules/content-engine.md` wires it into each post.

The shape of a generation step, once you have confirmed the real API:

```bash
# Illustrative only. Confirm the real endpoint, fields, and output formats in Meshy's current docs.
curl -X POST "https://api.meshy.ai/v1/text-to-3d" \
  -H "Authorization: Bearer $MESHY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "low-poly neon pixel city block, night, glowing signs, consistent flat-shaded style",
    "art_style": "pixel"
  }'
# Save the resulting model, assemble the scene, render the reusable backdrop into assets/.
```

## Guardrail and proof

The renders feed the content engine, and posts still go through the owner's approval board before anything ships (rule 3 in `CLAUDE.md`). Generation here is asset creation, not publishing.

Prove it, out loud (rule 5): show the owner the actual generated assets and the assembled render, not a description. If the style drifted across assets, say so and regenerate until the world is consistent, because consistency is the entire value.

## Recap

- The owner's own 3D world = their unfair advantage on the feed. Same signature visual in every video is what makes faceless content recognizable and compounding.
- Meshy turns text or images into 3D models. Generate assets, assemble a scene, render the recurring backdrop, hand it to `modules/content-engine.md`.
- Build it when the content channel is working, not before (vanity-trap rule).
- Check Meshy's current docs for real API and pricing. Key in `.env`, never in chat.
- Want a signature world without generating one first? `modules/signature-dashboard.md` comes with four ready-made 3D worlds (harbor, homestead, agent-city, station) the owner can pick and run, and it wires the live approval board onto the world so it works as both an ops dashboard and filmable content. They arrive as a separate Worlds Pack download from the owner's purchase; that module's Step 1 walks them through it. Meshy here is the path for owners who later want to generate their own custom world instead.

## Market data: the price source for the trading agents

> For the buyer's AI. A READ-ONLY price feed for `modules/analyst-agent.md` (monitor-only) and `modules/paper-trading.md` (simulated portfolio). It reads the market; it never touches it. There is no execution here and no brokerage connection, ever.

The owner will say "Robinhood, so my agent can trade stocks." Tell them the truth plainly:

- Robinhood has **no official public API**. The unofficial libraries drive your real logged-in account, which violates Robinhood's terms and can get the account locked.
- More important: this system has a hard rule (see `modules/paper-trading.md`). It **never connects to a brokerage with trade rights.** The paper-trading agent proposes hypothetical trades on fake money and stops there. A real trade is something the owner does by their own hand, in their own brokerage, outside this system.

So you do not wire Robinhood, or any broker, for execution. You wire a read-only **data** provider so the agents can see prices. That is the only thing they need from the outside.

Read-only data providers to choose from (confirm current free tiers, endpoints, and auth before you build, rule 5, and before you quote pricing as fact):

- **Alpaca Market Data** (alpaca.markets): clean docs, free tier, stocks and crypto. Use the **market data endpoints only**. Do NOT wire Alpaca's trading client, not even its paper sandbox: paper and live share one API surface, one client library, and one key format, and differ only by the base URL. Wiring the sandbox puts a broker client one string away from live in a codebase whose owner is told to verify safety by grepping for exactly that (`modules/paper-trading.md`, "the absence is the feature"). The paper portfolio marks positions from price data and simulates its own fills in `portfolio.json`. It needs no broker connection of any kind.
- **Finnhub**, **Alpha Vantage**, or **Polygon**: free tiers for quotes and history. Good enough for the cheap fetch loop that marks positions.

Wiring: put the data key in `.env`, read it there, never print it or paste it in chat (rule 11). The cheap loop in `modules/paper-trading.md` calls your `getPrice()` against this provider. Prove it out loud (rule 5): show a real `portfolio.json` marked against live prices, then grep the code in front of the owner to prove there is no execution path. The absence is the feature.

## Sports odds: data for a paper sportsbook

> For the buyer's AI. Odds and scores DATA for a SIMULATED sports-pick agent. Same stance as trading: it tracks hypothetical picks against real outcomes with fake stakes. It never places a real bet, and there is no path that could.

Build a sports agent exactly like the trading pair, disciplined and paper-only. Be straight with the owner up front: **this is not betting advice, gambling carries real risk, and you can lose money if you ever wager for real.** No consumer sportsbook offers an API to place bets programmatically; automating real-money betting would break their terms and, depending on the owner's state and jurisdiction, the law. So the agent keeps a paper record and grades it. That is the whole thing. Gate it opt-in, the same way `modules/paper-trading.md` gates trading.

Data providers (confirm current docs and free tiers):

- **The Odds API** (the-odds-api.com): live lines and odds across books, with a free tier. This is the "market data" of the sports agent.
- **ESPN's unofficial endpoints**: schedules and final scores to grade picks against reality. Unofficial means they can change without notice, so verify them and handle failure gracefully rather than trusting a remembered URL.

Flow, mirroring the trading module: pull the lines, let a reason loop propose paper picks that each carry a stake (capped) and written reasoning, log every pick, grade against the real result, and keep a `lessons.md` of what did not work. Same hard caps, same drawdown breaker on the paper bankroll, same board, same proof: show the paper record, then prove there is no path to a real bookmaker. The point is to measure whether an approach holds up with zero money at risk, not to simulate the psychology that empties a real account.

## Fiverr and freelance platforms: an assist agent, not an autopilot

> For the buyer's AI. A drafting agent for the owner's freelance gigs (Fiverr, Upwork). It writes proposals and replies in the owner's voice; the owner sends them. There is no full automation, and saying so is honesty, not a shortcoming to paper over.

The reality, told plainly: Fiverr has no open API for automating gigs, proposals, or buyer messages, and driving a headless login against it risks the account that earns. Upwork's API is limited and gates proposal submission. So the durable, terms-safe build is an **assist** agent:

1. The owner pastes a buyer's brief or message, or forwards the notification email (the agent can read a forwarded email).
2. The agent drafts a tailored proposal or reply in the owner's voice and drops it in `board/pending/`.
3. The owner reviews it and sends it themselves on the platform.

This still removes the real work, writing the sharp tailored response fast, without breaking a platform's rules or risking a suspension. Wire nothing that logs in as the owner. Prove it (rule 5) by drafting one real proposal from a real brief and showing the owner the draft, not a claim that it works.

Why it earns its place: this is how the owner turns a marketplace into a lead source without living in the inbox, and it points straight at the customer-getting problem most owners actually have.

## Voice: talk to your agent (the "Jarvis" layer)

> For the buyer's AI. A voice loop so the owner can speak to the manager agent and hear it answer, instead of typing. It is an INTERFACE on top of the system you already built. It gives the agents no new power to act, and every real action still routes through the approval board.

Three layers, each with a free start and a paid upgrade:

1. **Speech to text (hear the owner).** Start free with the browser's built-in **Web Speech API** (`SpeechRecognition`), no key, runs right in the signature dashboard. For higher accuracy, non-Chrome browsers, or offline, use **OpenAI Whisper** (local or hosted) or another STT provider.
2. **The brain.** The transcribed text goes to the manager agent (`modules/manager.md`) exactly as a typed command would: same tight prompt, same whitelisted tools, same board. Voice changes the input, nothing else.
3. **Text to speech (talk back).** Start free with the browser's **SpeechSynthesis**. For a natural, on-brand voice, use **ElevenLabs** or **OpenAI TTS** (key in `.env`, confirm current pricing before you quote it).

Wire it as **push-to-talk** in `modules/signature-dashboard.md`: hold a key, speak, see the transcript appear, watch the manager respond in text and read it back in voice. That makes the whole system feel like Jarvis without pretending it is magic.

The rule that does not bend: **voice is input, not authority.** Saying "post that" still only drops the item in `board/pending/`; the owner still approves it on the board. A spoken command is identical to a typed one and never bypasses rule 3. Do not let "it is faster by voice" quietly become "it just does things now." Prove it (rule 5): speak one real command in front of the owner and show the transcript, the drafted board card, and the spoken reply, with the approval step still standing between the words and the action.

## Tool map: which agent reaches for which tool

A quick index so every agent's outside tools are explicit. The content trio and the four above are written up in this file; the platform tools marked `[wire when built]` get their own sections as their modules come online, and confirm which provider the owner actually uses before documenting one as fact.

- `modules/content-engine.md` -> Zernio (post), Higgsfield (video), Meshy (world)
- `modules/analyst-agent.md`, `modules/paper-trading.md` -> Market data (read-only)
- paper sportsbook (opt-in, same shape as paper-trading) -> Sports odds
- freelance / services agent -> Fiverr and Upwork assist
- `modules/manager.md` -> Voice (Jarvis)
- `modules/etsy-autopilot.md`, `modules/etsy-store-builder.md` -> Etsy API `[wire when built]`
- `modules/pod-printify.md` -> Printify API `[wire when built]`
- `modules/shopify-store.md`, `modules/dropshipping.md` -> Shopify API and a supplier API `[wire when built]`
- `modules/any-store.md` -> whatever the owner already sells on (WooCommerce, BigCommerce, Wix, Amazon...). ONE adapter file; look up that platform's current API before building `[wire per owner]`
- `modules/marketplace.md` -> the marketplace's vendor and approval API. Dokan sits on the WooCommerce `/wp-json/wc/v3/` and Dokan `/wp-json/dokan/v1/` namespaces `[wire per owner]`
- `modules/fiverr-services.md` -> NO platform API exists. An image generator plus a vectorizer (`potrace` or `vtracer`) plus a commercially licensed font set (OFL). The owner does every Fiverr action by hand.
- `modules/marketing.md` -> an email sender (Resend or Gmail) `[wire when built]`
- `modules/research.md` -> web search and scrape `[wire when built]`

Every one of these, without exception: key in `.env` not chat, confirm the current docs before building, route real actions through the board, and prove it out loud against the live service.
