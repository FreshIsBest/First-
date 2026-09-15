> For the human buyer (the AI can read it too). Plain-language money page: what this costs to run, the one billing surprise that trips people up, and honest break-even.

# Cost and billing: what this actually costs to run

No fluff. Here is where the money goes, how to watch it, and how long it usually takes to earn it back. Some of this comes with a disclosure duty (third-party tools cost money), so read the whole page before you flip anything on.

## The two kinds of cost

There are only two buckets. Keep them separate in your head.

1. **The AI engine.** The brain that runs your agents. Flat monthly, or pay-as-you-go. You control this.
2. **The metered tools.** Third-party services that charge per use: printing a product, generating a video, posting to platforms. These only cost money when you actually make something or sell something.

That is it. No hidden platform fee, no seat licenses, no per-agent charge. The system itself is plain files and scripts on your own machine (see `05-infrastructure.md`). The files are free. The thinking and the doing cost money.

## Bucket 1: the AI engine (the brain)

You pick one. All three drive the exact same system.

| Engine | What it costs | Best for |
| --- | --- | --- |
| Claude Code Pro | about $20/mo, flat | The recommended starting point. Predictable bill, plenty for a first money-maker. |
| OpenRouter + Hermes | pay-as-you-go, no subscription | You do not want a subscription. You pay per call, cents at a time, and stop whenever. |
| Claude Code Max | higher flat monthly | You are running agents hard, all day, many tasks. Only worth it once you have real volume. |

**Start on Claude Code Pro at about $20/mo.** It is a flat bill, so nothing surprises you, and one subscription is more than enough to build and run your first money-maker. Move up to Max only when you feel the ceiling, not before. If you would rather owe nothing monthly, OpenRouter with a Hermes model bills you by the call: fractions of a cent to a few cents each, and you top up a balance instead of subscribing.

You do not need all three. One engine runs everything.

### Honest note on Claude Pro usage limits

Claude Code Pro is the recommended start, but be straight about one ceiling. A flat subscription is not unlimited. Pro comes with rolling usage windows: you get a chunk of usage over a set period, and when you burn through it the plan pauses you until the window resets. For a person using the AI by hand, you rarely hit it. For a **dispatcher spawning fresh agents all day**, you can, and when you do, the loop starts hitting rate-limit errors and every task fails until the window rolls over.

Two things follow from this:

- **The dispatcher must handle a rate-limit error gracefully, not die on it.** When a call comes back rate-limited, the dispatcher should catch that specific error, pause (back off and wait), and resume when the window resets, rather than marking a pile of tasks `failed`. This is the same back-off behavior in the rate-limit gotcha (`08-gotchas.md`). A rate limit is a "wait," not a "stop." Build it to wait.
- **Heavy daily volume is what Max or pay-as-you-go is for.** If the owner genuinely runs agents hard all day and keeps hitting the Pro ceiling, that is the signal to move up to Claude Code Max (a higher flat plan built for volume) or to OpenRouter pay-as-you-go (billed per call, no window to exhaust). Do not fight a Pro limit with retries. Match the plan to the volume.

Do not quote a specific message count or window length here, because providers change these. **Verify the current Pro limits at the provider's own docs when you build the dispatcher**, and size the owner's expectations to whatever the docs say that day.

## Bucket 2: the metered tools (per use)

These charge only when you use them. Idle costs nothing. This is the part with a disclosure duty, so here it is plainly:

- **Printify (print-on-demand).** Free to list and design. You pay Printify's base price for an item **only when a customer buys one**, and it comes out of the sale, not your pocket up front. If you sell nothing, you pay nothing. Your margin is your price minus their base cost. Covered in `modules/pod-printify.md`.
- **Video generation (Higgsfield).** Buys credits. Each generated video spends credits. See the Higgsfield section of `tools/tools.md`.
- **3D asset generation (Meshy).** Buys credits. Each 3D model or render spends credits. See the Meshy section of `tools/tools.md`.
- **Post-everywhere (Zernio).** Its own plan for pushing one post to many platforms at once. See the Zernio section of `tools/tools.md`.

None of these fire on their own. Your board gates them (see `07-approval-loops.md`). An agent that wants to generate a video or push a POD order parks the request and waits for your click. So you are never surprised by a metered charge you did not approve.

## The subscription billing surprise (read this before you panic)

This trips up almost everyone the first week, so understand it now.

If you run your agents on a **subscription** engine (Claude Code Pro or Max), and then you go look at the provider's **API usage dashboard** to see what your agents cost, it can read **near zero, or nothing at all**. Your first thought will be "the agents never ran." That is wrong.

Here is why. A subscription and per-call API billing are two different doors into the same model. When you run on a subscription, your calls are covered by the flat monthly fee. They are **not billed per call**, so they **do not show up on the per-call API usage meter**. That meter only tracks pay-as-you-go API spend, which you are not using. An empty API bill on a subscription plan is exactly what a working system looks like.

So do not panic, and do not go add an API key and start double-paying because the meter looked empty. To confirm your agents actually ran, look at the right place:

- Your **queue and board**: tasks moving from `queued` to `done` or `needs_approval` is proof of work. That is your real activity log.
- Your **result output**: agents write their output to `result`. Real drafts, listings, and posts showing up means it ran.
- Your **run logs**: the dispatcher records what it spawned and when.

The API usage dashboard is for the pay-as-you-go path (OpenRouter, or a raw API key). If you are on a Pro or Max subscription, that dashboard is not your source of truth. Your own queue is.

## How to actually watch your spend

Two habits, and one of them is already built into your system.

**1. The in-code caps (already enforced).** During the interview (`01-interview.md`) the AI asked you for your spend limits and wired them into the code as hard stops. Every loop has a max on tasks, spend, and runtime. An agent cannot run forever or rack up a surprise bill, because the loop stops itself when it hits your ceiling. This is guardrail work, covered in `02-architecture.md`. If you ever want to change a cap, tell the AI the new number and it edits the one place the cap lives. Do not raise a spend cap to "see what happens." Raise it on purpose, in a known amount.

**2. Check the tool dashboards weekly.** The metered tools each have their own dashboard showing credits used or charges rung up. Once a week, glance at:
- Your engine bill (flat, so this is a quick sanity check, or your OpenRouter balance if pay-as-you-go).
- Printify orders and base costs against your sales.
- Higgsfield and Meshy credit balances.
- Your Zernio plan usage.

Five minutes, once a week. That plus the in-code caps means nothing runs away from you.

## Honest break-even (results are not typical)

Straight talk, because you already paid for this and you deserve it.

**Most shops take weeks or months to make their money back, and some never do.** This is not a money button. It is a system that does the grunt work so you can run more of a real business than you could by hand. The business still has to be real: real products people want, real listings, real posts, real demand. The Agentic OS makes you faster and more consistent. It does not manufacture customers.

A realistic shape of the first stretch:

- **Week one:** you are spending on the engine (about $20) and building. Zero revenue. Normal.
- **First sales:** show up when your listings and content have been live long enough to get found. Could be days, could be many weeks. Depends entirely on your niche, your pricing, and your effort.
- **Break-even:** when your monthly revenue covers your engine plus your metered tool costs. For a lean POD or digital-goods shop, the running cost is small (roughly the ~$20 engine plus per-sale printing plus a modest credit spend), so break-even is a low bar. Clearing it still takes real traction.

**Results are not typical.** I run my own Etsy shop on this exact system and it took real time and real iteration to earn back, and plenty of ideas I tried went nowhere. Anyone promising fast guaranteed returns is selling you a feeling. What this system honestly gives you: lower running cost than hiring help, more output than doing it alone, and a machine that keeps working while you sleep (if you run it always-on, see `05-infrastructure.md`). Whether it earns is on the business you point it at.

Keep your costs boring and low while you find what works. One engine, caps on, tools metered and gated. Then scale the thing that earns, which is what `09-after-first-win.md` is for.
