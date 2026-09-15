> For the buyer's AI. ADVANCED, OPT-IN ONLY. Build a simulated (paper) trading portfolio agent with real engineering: a durable account file, a cheap price-marking loop, an expensive reasoning loop that proposes hypothetical trades with stops, hard caps, a drawdown circuit-breaker, and a serious multi-step gate before the owner ever considers real money. This system never touches a real brokerage, and building a bridge to one is the thing it refuses to do.

Level: advanced, opt-in only | Time: ~2 to 3 evenings | Cost: $0 on paper (real trading risks real capital) plus your engine

# Paper-trading portfolio (advanced, opt-in only)

## STOP. Read this to the owner before anything else.

**This is not financial advice. Trading carries real financial risk. You can lose money, including all of it. Nobody, and no agent, can predict the market. Results are not typical, and past performance predicts nothing.**

This is the second half of the trading pair. The first half, `modules/analyst-agent.md`, is a monitor-only watcher that reads and reports and never proposes an action. This file is the paper portfolio: a simulated account that proposes hypothetical trades and tracks how they would have done, with fake money, so the owner can measure whether an approach holds up before a single real dollar is ever at risk. It is the riskiest thing in the whole Blueprint, and it is opt-in for a reason.

## The opt-in gate (check this first, no exceptions)

**Only build this module if `PROFILE.md` shows the interview's trading opt-in was passed.** That is the explicit "I understand I can lose money, this is not financial advice, and I want to build this anyway" acknowledgement from `01-interview.md`. If that gate is not on record, you do not build this. You stop, and you route the owner back through the interview acknowledgement. No opt-in, no build. There is no soft version of this check.

If the owner is here because the safe money-makers feel slow, say it plainly: the safe stuff is where the money actually is for almost everyone, and trading is far more likely to lose their capital than replace it. Steer them, once, to the earning modules first:

- `modules/etsy-autopilot.md` (the flagship, build this first)
- `modules/pod-printify.md` (print-on-demand)
- `modules/content-engine.md` (content and traffic)

Those earn by selling real things and doing real work. A new owner belongs there. This module is a bonus for someone who already understands markets and has chosen the risk with open eyes. Do not let it become the thing they do instead of the work that earns.

## The one hard rule of this module

**No auto-execution of real trades. Ever.**

The agent you build here runs a **simulated** portfolio with fake money. It proposes hypothetical paper trades and records them. It does not place real trades. It does not move real money. It does not connect to a live brokerage with execution rights.

Any real trade is something the owner does by their own hand, in their own brokerage, outside this system entirely. There is no bridge from the board to a live brokerage, and building one is the single thing this module refuses to do. When the owner asks you to "just wire it to my account so it can actually trade," the answer is no, and this file is why. The agent proposes on paper. A human, later, in a different place, may or may not act. That gap stays open forever.

## The core engineering: a state-mutating dual-loop

This is the upgrade over a toy trading agent. The system is two loops with different jobs, different costs, and different tools. Same agent shape as `04-first-agent.md` everywhere: tight prompt, whitelisted tools, hard timeout, output to the board, fresh session per run (rule 2).

**Loop 1: the cheap fetch loop (pure code, no LLM).** This runs often. It pulls the current price for every held position, marks the portfolio to market, recomputes unrealized profit and loss, and writes the updated `portfolio.json`. No reasoning, no model call, no proposals. Just deterministic arithmetic on real numbers. This is the loop that keeps the simulated account honest between decisions, and because it never calls the engine it costs almost nothing to run.

**Loop 2: the expensive reason loop (LLM).** This runs rarely, on a schedule or on demand, and only within the caps. It reads the current marked state, checks the owner's curated universe against the owner's written rules, and proposes at most a few hypothetical paper trades. Every proposal carries an entry, a stop-loss, a size, and the reasoning. It logs each proposal to the journal. It never executes anything, not even on paper, without the caps and the circuit-breaker having cleared in code first.

Keeping these separate is the whole design. The cheap loop maintains truth cheaply. The expensive loop thinks rarely and deliberately. A single loop that both marks positions and reasons on every tick would burn tokens for no reason and drift. Split them.

### portfolio.json: the durable memory of the simulated account

The account lives in one plain file. It is the source of truth for the paper portfolio, the same spirit as every other flat file in this system (rule 7, keep it simple and hand-rolled). A concrete schema:

```json
{
  "mode": "paper",
  "opened": "2026-07-24",
  "starting_cash": 10000.00,
  "cash": 7320.50,
  "positions": [
    {
      "ticker": "ACME",
      "qty": 20,
      "entry": 128.40,
      "stop": 119.00,
      "opened": "2026-07-24",
      "last_mark": 131.10,
      "unrealized": 54.00
    }
  ],
  "realized_pl": 210.75,
  "unrealized_pl": 54.00,
  "equity": 10000.00,
  "peak_equity": 10120.00,
  "history": [
    {
      "date": "2026-07-24",
      "action": "paper_buy",
      "ticker": "ACME",
      "qty": 20,
      "price": 128.40,
      "stop": 119.00,
      "reason": "Held above the 50-day on rising volume; matches the owner's breakout rule."
    }
  ]
}
```

- `mode` is always `"paper"`. If any code path can set it to anything else, you built the thing this module forbids. There is no live mode.
- `cash` plus the marked value of `positions` is `equity`. The cheap loop recomputes this every run.
- `peak_equity` is the high-water mark. The drawdown circuit-breaker measures against it.
- `history` is the append-only ledger of every paper action, so the owner can audit exactly what the account did and when.

### The curated universe (not the whole market)

The agent does not roam the entire market looking for something to do. The owner picks a small list of tickers they actually understand and want to track, and it lives in `PROFILE.md`. The reason loop only ever considers that universe. A tight list keeps the reasoning focused, the data cheap to fetch, and the results interpretable. An agent scanning thousands of symbols is not smarter, it is just noisier and more expensive.

### Every proposal requires a stop-loss (refuse otherwise)

A paper proposal with no defined stop is a broken proposal. The stop is the exit that caps the loss, and a strategy without one is not a strategy. Enforce this in code, not just in the prompt: after the reason loop returns a proposal, validate that a numeric `stop` is present and is on the correct side of the entry. If it is missing, the agent does not get to shrug and propose anyway. Drop the proposal, log the refusal, and write plainly that the agent tried to propose without a stop and was blocked. Make the failure loud.

### The drawdown circuit-breaker

If the paper account draws down past a set percentage from its peak equity (the owner sets the percent in `PROFILE.md`), the reason loop stops proposing. It does not double down. It does not "try to make it back." It writes, in plain words, "the strategy is losing on paper, stop and rethink," and it halts new proposals until the owner reviews. Losing strategies do not get to keep running on autopilot, even with fake money, because the point of paper trading is to catch a losing approach cheaply, not to simulate the exact psychology that blows up a real account.

Check the breaker in code before the reason loop runs, using `equity` and `peak_equity` from `portfolio.json`. If tripped, skip the LLM call entirely and write the halt notice. Cheap, deterministic, and impossible for the model to talk its way past.

### Hard caps, set before the agent runs once

Set these in `PROFILE.md` and enforce them in code before the first run (rule 4, spend caps and stop conditions on everything):

- **Daily proposal cap.** A small number of proposals per day, maximum. Overtrading is a losing pattern; do not let the paper agent simulate it. Count today's `history` entries before proposing.
- **Max position size cap.** No single paper position may exceed a set fraction of equity. Enforce it when sizing, so the paper results reflect a sane strategy and not a fantasy of going all-in.
- **Drawdown percent.** The circuit-breaker threshold above.
- **Hard timeout** on every run, like every agent in this system.

These are code gates, checked before the reasoning happens. The model never decides whether a cap applies. The code decides, and the model runs only inside what the code allows.

### lessons.md: learn from the paper losses cheaply

This is the honest payoff of paper trading. When a logged proposal would have hit its stop and lost, record why in `lessons.md`, the same feedback file used across the system (`07-approval-loops.md`, Pattern A). "Bought the breakout, it faded on low volume, the setup lacked confirmation." Then feed the accumulated lessons back into the reason loop's prompt, so future proposals avoid the same mistake before the owner ever sees them. Learning what does not work, with zero money at risk, is the entire reason this thing exists.

## The real-money graduation gate (this is the differentiator)

Suppose the paper record is good and the owner starts eyeing real money. This is the most dangerous moment in the Blueprint, and it gets a hard, deliberate, multi-step gate. Frame it to the owner exactly as what it is: **a gate against their own excitement.** Good paper results feel like permission. They are not permission. They are one input, and the gate exists precisely because a hot streak is when people size up and get hurt.

The gate has five parts, and all of them must clear:

1. **A minimum paper track record.** No real-money conversation happens before the paper account has run for a set minimum (default 30 days) with a real number of logged proposals. A week of luck is not a track record. Check the `opened` date and the `history` length.
2. **Multiple typed confirmations, not clicks.** The owner types out the risk acknowledgement in full, by hand, more than once. Not a button. Typing "I understand I can lose all of the money I put in, this is not financial advice, and no one can predict the market" forces them to read their own words. A one-click "I agree" does not.
3. **A cooling-off period.** After the confirmations, a hard wait (default 24 hours) before any real-money action is even discussed again. Record the timestamp; refuse to proceed until it has elapsed. Excitement is loudest in the moment. The wait lets it cool.
4. **Restate that the system still never executes.** Even after every part of this gate clears, the SYSTEM still does not place trades. Passing the gate does not unlock a live mode, because there is no live mode. A real trade remains the owner's own manual action, in their own brokerage, on their own hands. The gate is about the owner's readiness to go do that thing elsewhere, not about the system gaining a power it never has.
5. **The drawdown, tax, and regulatory warning.** State plainly, in writing: real trading can lose everything; real gains and losses carry tax consequences the owner is responsible for; and depending on their situation and jurisdiction there may be regulatory rules (pattern-day-trader thresholds, account minimums, reporting) that are theirs to understand. This module does not and cannot advise on any of it. Verify nothing about the owner's specific tax or legal situation from memory; tell them to confirm it with a professional.

If any part fails, the gate does not open, and the owner stays on paper. This is not friction for its own sake. It is the difference between a tool that respects the risk and one that funnels an excited person toward losing their savings.

## What auto-runs vs what always waits

Auto-runs (safe, produces text and simulated state, risks nothing):

- the cheap fetch loop marking positions and updating `portfolio.json`
- checking the owner's rules against the curated universe
- generating paper proposals within the caps and logging them
- the daily paper-journal digest
- tripping the drawdown circuit-breaker and writing the halt notice

Always waits for a human, and stays outside this system entirely:

- **any real trade** (never routes through an agent action at all; the owner does it themselves in their brokerage)
- passing the graduation gate
- changing the caps, the universe, or the strategy

Note the difference from the store modules. In `modules/etsy-autopilot.md` and the others, an approved board click can trigger a follow-up task that performs the real action (publish, order, send). **Here it does not.** There is no "approve, then place the trade" path. An approved board card here means the owner has read a hypothetical and nothing more. The card is where the agent's involvement ends. Do not build a bridge from the board to a live brokerage. That bridge is the whole thing this module refuses to build.

## Illustrative code: the cheap loop marking to market

Illustrative only. Verify the current docs for whatever price source the owner uses before you wire it; data-source APIs and their auth change, and rule 5 means you prove it against the live source, not against this sketch.

```javascript
// mark-positions.mjs: cheap loop. No LLM. Marks paper positions to market.
// Illustrative. Verify your current price-source docs before relying on it.
import { readFileSync, writeFileSync } from "node:fs";

const pf = JSON.parse(readFileSync("portfolio.json", "utf8"));
if (pf.mode !== "paper") throw new Error("Refusing to run: mode is not paper.");

let positionsValue = 0;
for (const p of pf.positions) {
  const price = await getPrice(p.ticker);      // your verified data source
  p.last_mark = price;
  p.unrealized = +((price - p.entry) * p.qty).toFixed(2);
  positionsValue += price * p.qty;
}

pf.unrealized_pl = +pf.positions.reduce((s, p) => s + p.unrealized, 0).toFixed(2);
pf.equity = +(pf.cash + positionsValue).toFixed(2);
pf.peak_equity = Math.max(pf.peak_equity ?? pf.equity, pf.equity);

writeFileSync("portfolio.json", JSON.stringify(pf, null, 2));
console.log(`Marked ${pf.positions.length} positions. Equity: ${pf.equity}`);
```

Any data-source credentials go in `.env` and are read from there, never printed and never pasted into chat (rule 11). If a key ever lands in a log or the chat, tell the owner to reroll it at the provider immediately.

### The reason loop prompt shape (tight, refuses without a stop)

```
You are a disciplined paper-trading analyst for a SIMULATED account with fake money.
UNIVERSE: only these tickers: {universe}. Ignore everything else.
STATE: {portfolio_json}. RULES: {owner_written_rules}. LESSONS: {lessons_md}.
TASK: Propose at most {daily_cap} hypothetical paper trades that match the owner's rules.
For EACH proposal you MUST give: ticker, entry price, stop-loss price, size (respecting the
max position cap), and one-paragraph reasoning tied to the rules.
HARD RULE: every proposal MUST include a stop-loss. If you cannot set a sane stop, do not
propose that trade. A proposal without a stop is invalid.
Nothing here is real. You place no trades. You only propose on paper.
OUTPUT: a JSON array of proposals, nothing else.
```

The code that receives this output validates the stop on every proposal and drops any that lack one, per the refuse rule above. The prompt asks; the code enforces.

## Prove it works (rule 5), and prove there is no path to real money

Do not tell the owner this works. Show them, out loud:

- Show the real `portfolio.json` after a cheap-loop run, with positions marked to real current prices and equity recomputed.
- Show a real paper proposal from the reason loop with its entry, stop, size, and reasoning, and show the daily-cap and max-size checks that ran in code before it was allowed.
- Show the journal tracking how the hypothetical proposals would have performed over time, wins and losses both.
- Trip the drawdown circuit-breaker on purpose (feed it a drawn-down state) and show it refusing to propose and writing the halt notice.
- Show the missing-stop refusal firing when a proposal comes back without a stop.
- Then prove the negative: there is no brokerage connection with trade rights, no execution path, nothing in the code that can move real money. Grep the codebase for it in front of the owner if that makes it concrete. The absence is the feature.

Then say it plainly:

"This runs a pretend portfolio with fake money so you can see if an approach actually holds up, with zero dollars at risk. It cannot and will not place a real trade or touch your money, and there is no setting that turns that on. If you ever trade for real, you do that yourself, in your own account, knowing you can lose all of it. This is a measurement tool, not a money machine, and it is the riskiest thing in this whole Blueprint. The reliable money is in the other modules."

If the paper record ever tempts the owner toward real money, walk them through the graduation gate above, slowly, and remember what the gate is for: protecting them from their own excitement. And if that gotcha about approving from a summary instead of the real artifact ever tempts you to cut a corner here, `08-gotchas.md` applies to this module too. Show the real state, never a flattering summary of it.

Unless the owner has a specific reason to keep building here, point them back to the work that actually earns.
