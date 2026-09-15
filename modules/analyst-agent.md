> For the buyer's AI. ADVANCED, OPT-IN ONLY. Build a monitor-only market and fund analyst: a decoupled dual-loop agent where cheap code computes every number and a rare LLM run only reads those numbers into a plain-language daily read. It watches and reports. It never trades, never proposes a trade, never touches money. Read the disclaimer to the owner and steer them to the earning modules first.

Level: advanced, opt-in only | Time: ~2 evenings | Cost: $0 on a free market-data source, plus your engine (the reason loop runs rarely, so engine cost stays near nothing)

# Module: analyst agent (monitor-only markets)

## STOP. Read this to the owner before anything else.

**This is not financial advice. Watching a market is not predicting it. Nobody, and no agent, can tell you what a price will do next. Past performance means nothing, and a clean-looking daily read is a description of what already happened, not a forecast.**

This module builds an agent that watches a watchlist of tickers or funds and writes a daily read. That is the whole job. It has no trade proposals in it, on paper or otherwise. If the owner wants hypothetical paper-trade proposals with entries and stops, that is a different, riskier module, `modules/paper-trading.md`, and it stays firewalled off from this one. This file is the safest thing in the trading family because it proposes nothing at all. It reads and reports, like a scout in `modules/research.md` pointed at market data.

**Only build this if `PROFILE.md` shows the interview's trading opt-in gate was passed** (the "I understand I can lose money, this is not financial advice" acknowledgement from `01-interview.md`). No opt-in on record, no build. If the gate is not there, stop, and route the owner back through the interview acknowledgement first. This is the same gate `modules/paper-trading.md` uses, and it is not optional here just because this agent never trades. An owner who is watching markets is one step from acting on what they see, so the acknowledgement comes first.

If the owner reached for this because the earning modules feel slow, say it plainly: a daily market read does not make a dollar. It is a dashboard, not an income. The money in this Blueprint is in the work that sells real things:

- `modules/etsy-autopilot.md` (the flagship, build this first)
- `modules/pod-printify.md` (print-on-demand)
- `modules/content-engine.md` (content and traffic)

Build those before you build this. This module is a bonus for an owner who already follows markets and wants a tidy, screenshot-friendly summary each morning without staring at ten charts. It earns nothing on its own.

## The one hard rule of this module

**Monitor only. It watches and reports. It never acts, never trades, never proposes a trade.**

No order, no paper order, no brokerage connection, no "here is what I would buy." The agent's entire job ends at a written daily read parked on the board or in a report file. A human looks at it, or does not. Nothing downstream happens. There is no approve-then-act path here at all, because there is no act. This is even tighter than `modules/paper-trading.md`: that module at least proposes on paper, and this one refuses to propose anything. Do not build a bridge from this agent to a brokerage, to a paper account, or to any order function. That refusal is the module.

## The core idea: a decoupled dual-loop

This is the engineering that makes the module worth building instead of pasting numbers into a chat. Split the work into two loops that never touch the same job.

```
CHEAP FETCH LOOP                         EXPENSIVE REASON LOOP
runs often (every 15-60 min)             runs rarely (once a day)
pure code, NO LLM                        LLM agent, READ-ONLY
  |                                         |
  fetch prices from a data source           read watchlist-state.json
  compute MAs, %change, RSI, volume   -->   write a plain-language read
  write watchlist-state.json                cite ONLY numbers in the file
  (deterministic, every number here)        (invents nothing)
```

**The cheap fetch loop is pure code and holds every number.** It runs on a schedule, hits a market-data source, and computes all the indicators itself: moving averages, percent change, RSI, volume versus average, whatever the owner wants. It writes those into a small state file, `watchlist-state.json`, and exits. No LLM anywhere in this loop. It is free to run because it is just code, and it is correct because arithmetic in code is deterministic: the same inputs give the same numbers every time.

**The expensive reason loop is an LLM that only reads.** Once a day, an agent opens `watchlist-state.json`, reads the pre-computed numbers, and writes a plain-language summary of what those numbers say. It does not fetch anything. It does not calculate anything. It cannot cite a number that is not already in the state file. That is the whole point.

### Why decouple. Two reasons, both non-negotiable.

**Cost.** The loop that runs constantly (the fetch) is free, because it is code, not model calls. The loop that costs money (the LLM read) runs once a day. If you instead had one LLM agent fetch and reason every 15 minutes, you would burn tokens all day to restate the same thing. Decoupling puts the frequent work where it is free and the expensive work where it is rare. This is the cost discipline from `06-cost-and-billing.md` applied to a live-data agent: the cheap loop can poll all day and never move the meter.

**Correctness. This is the anti-hallucination guarantee.** LLMs make up numbers. Ask a model "what is SPY's RSI" and it will confidently produce a plausible, wrong number, because generating plausible text is what it does. So you never let the model produce a number. The code computes every number and writes it down. The model is handed those numbers and told to describe them in words. If a figure is not in `watchlist-state.json`, the agent physically has nothing to cite, so it cannot invent one. Numbers come from code. Words come from the model. They never cross. That single rule is what makes this agent trustworthy instead of a confident liar.

## The state file: `watchlist-state.json`

This is the contract between the two loops, the same way the input and output payloads are the contract in `modules/research.md`. The fetch loop writes it. The reason loop reads it. Keep it small, flat, and dead simple.

```json
{
  "generated_at": "2026-07-24T08:00:00Z",
  "source": "free market-data API (verify current docs before wiring)",
  "disclaimer": "Descriptive only. Not advice. Not a prediction.",
  "tickers": [
    {
      "symbol": "SPY",
      "price": 548.12,
      "prev_close": 545.30,
      "pct_change_day": 0.52,
      "pct_change_week": -1.14,
      "ma_50": 541.88,
      "ma_200": 512.44,
      "rsi_14": 61.3,
      "volume": 68420000,
      "avg_volume_30d": 74100000,
      "above_ma_50": true,
      "above_ma_200": true,
      "as_of": "2026-07-24T07:59:00Z"
    }
  ],
  "notes": []
}
```

Every field here is produced by code. `pct_change_day` is arithmetic. `ma_50` is an average of closes. `rsi_14` is a fixed formula. `above_ma_50` is a comparison the code already did, so the model does not have to reason about greater-than and get it wrong. Pre-computing the booleans is a small move that removes a whole class of model mistakes: the agent reads "above_ma_50: true" instead of eyeballing two numbers.

## The cheap fetch step (illustrative code)

Illustrative only. **Verify the current docs of whatever data source the owner picks before you wire it** (rule 5 mindset: prove the endpoint and its fields are what you think, do not trust this sketch as live). Free market-data sources change their endpoints, rate limits, and field names, and many restrict automated access in their terms, so read the terms and the current API reference first, exactly like the data-source caution in `modules/research.md` and the compliance section of `08-gotchas.md`.

```python
# tools/fetch_watchlist.py
# CHEAP LOOP. Pure code, no LLM. Fetches prices, computes indicators,
# writes watchlist-state.json. Runs on a schedule (see 05-infrastructure.md).
# Illustrative: verify your data source's real endpoint, fields, and terms first.

import json, datetime
# import your chosen data source here (a free market-data API, a yfinance-style
# library, whatever the owner picked). Confirm its current docs before trusting it.

WATCHLIST = ["SPY", "QQQ", "VTI"]   # from PROFILE.md, owner's tickers/funds

def rsi(closes, period=14):
    # standard RSI formula, computed in code so it is deterministic
    gains, losses = [], []
    for i in range(1, len(closes)):
        d = closes[i] - closes[i - 1]
        gains.append(max(d, 0)); losses.append(max(-d, 0))
    avg_gain = sum(gains[-period:]) / period
    avg_loss = sum(losses[-period:]) / period
    if avg_loss == 0:
        return 100.0
    rs = avg_gain / avg_loss
    return round(100 - (100 / (1 + rs)), 1)

def build_entry(symbol):
    closes = get_daily_closes(symbol, days=250)   # your data source call
    price, prev = closes[-1], closes[-2]
    ma50, ma200 = sum(closes[-50:]) / 50, sum(closes[-200:]) / 200
    return {
        "symbol": symbol,
        "price": round(price, 2),
        "prev_close": round(prev, 2),
        "pct_change_day": round((price - prev) / prev * 100, 2),
        "ma_50": round(ma50, 2),
        "ma_200": round(ma200, 2),
        "rsi_14": rsi(closes),
        "above_ma_50": price > ma50,
        "above_ma_200": price > ma200,
        "as_of": datetime.datetime.utcnow().isoformat() + "Z",
    }

state = {
    "generated_at": datetime.datetime.utcnow().isoformat() + "Z",
    "source": "your data source (verify docs)",
    "disclaimer": "Descriptive only. Not advice. Not a prediction.",
    "tickers": [build_entry(s) for s in WATCHLIST],
    "notes": [],
}
with open("watchlist-state.json", "w") as f:
    json.dump(state, f, indent=2)
print("Wrote watchlist-state.json for", ", ".join(WATCHLIST))
```

If the owner is on Windows, this runs through WSL or Git Bash and is scheduled with Task Scheduler, not cron (rule 12, detail in `05-infrastructure.md`). Any API key the data source needs lives in a local `.env` the owner fills in themselves, never pasted into chat or committed (rule 11).

## The reason loop (the read-only LLM agent)

Same agent shape as `04-first-agent.md`: tight prompt, whitelisted tools, hard timeout, fresh session, output the owner reviews. The whitelist is the tightest possible: it reads the state file and writes its summary. **No web tool, no execution tool, no data-fetch tool.** If the agent could fetch, it could invent; take the ability away.

```bash
#!/usr/bin/env bash
# agents/market-read.sh
# EXPENSIVE LOOP, run rarely (once a day). Reads watchlist-state.json ONLY.
# Writes a plain-language daily read. No fetching, no math, no proposals.
set -euo pipefail

TASK_ID="$1"
OUT="board/pending/${TASK_ID}/market-read.md"
mkdir -p "board/pending/${TASK_ID}"

PROMPT="You are a market monitor for the owner. You are handed a JSON file of
numbers that were already computed by code. Your ONLY job is to describe those
numbers in plain language.

HARD RULES:
- Use ONLY numbers that appear in watchlist-state.json. If a number is not in the
  file, you do not have it and you do not mention it. Never estimate or recall a
  figure from memory.
- Do NOT predict, forecast, or say what will happen. Describe what the numbers
  show, past tense.
- Do NOT propose any trade, buy, sell, or position. That is not your job and not
  in this system.
- Every claim must trace to a field in the file. Cite the symbol and the field.

OUTPUT: a short daily read. For each ticker, 1-2 plain sentences on where it sits
(price vs its moving averages, day and week change, RSI band, volume vs average).
End with a one-line 'what the owner is watching' recap. No advice. No preamble."

# Read-only whitelist. It reads the state file and writes the read. Nothing else.
timeout 120 claude -p "$PROMPT" \
  --allowedTools "Read,Write" \
  < watchlist-state.json \
  > "$OUT"

echo "Wrote daily read to $OUT"
```

The prompt whitelist inside the prompt (use only numbers in the file, no predictions, no proposals) plus the tool whitelist (`Read,Write` only) are two fences around the same rule. Keep both.

## Output: a daily digest card

The reason loop writes one plain-language card and parks it on the approval board as a report the owner reads (or writes it to a simple report file before the board exists, per the timing note in `04-first-agent.md`). Because it proposes nothing, there is no Approve or Reject action on this card. It is informational. The owner reads it and moves on.

This is the module's honest payoff: a clean, screenshot-friendly daily read the owner can glance at with coffee or drop into their content. It ties straight into the content play in `modules/content-engine.md`. "Here is today's market in plain English" is filmable, shareable, and true, because every number in it came from code and every sentence traces to a field. It is a genuinely nice thing to look at. It just is not a trade and never becomes one.

## Hard rules for this module

- **No trade proposals here.** Not on paper, not hypothetically, not "for fun." Proposals live in the separate `modules/paper-trading.md`, behind their own gate. This agent describes; it never suggests a position.
- **No brokerage connection. Ever.** No account with read, trade, or withdrawal rights. This agent has no reason to know an account exists.
- **No auto-anything downstream.** The card is the end. Nothing the owner clicks makes this system act. Nothing risky auto-runs (rule 3), and here nothing runs at all past the read.
- **All numbers from code.** The LLM never computes or fetches a figure. If it is a number, the fetch loop made it. If it is a sentence, the reason loop wrote it.
- **Every claim traces to the state file.** No figure the model "remembers," no market fact from training. If it is not in `watchlist-state.json`, it does not go in the read.
- **Caps and timeouts like everything else** (rule 4). The fetch loop gets a rate limit and a max-runtime so it cannot hammer the data source. The reason loop gets a hard timeout like every agent.
- **Fresh session per run** (rule 2). Each daily read is a new headless call. The state file on disk is the memory, not a long-lived chat.

## Prove it works (rule 5)

Do not tell the owner this works. Show them, in two parts, because the whole design is about separating the numbers from the words.

1. **Run the cheap loop and show the state file.** Run `fetch_watchlist.py` for real against the owner's tickers. Open `watchlist-state.json` and show them the populated numbers: prices, moving averages, RSI, volume, all computed by code, each with an `as_of` stamp. Point out that no model touched any of these.
2. **Run the reason loop and show the read.** Run `market-read.sh`, open the card, and read it next to the state file. Show the owner that every number in the summary appears in the JSON, and that the words add no figure the file did not already hold. Then show that there is no proposal anywhere in it, and no path from this card to a trade or an account.

Say it plainly: "This watches your tickers, does all the math in code so nothing gets made up, and writes you a plain-English read once a day. It cannot predict anything, it cannot propose a trade, and it is not wired to any brokerage or your money. It is a dashboard. If you ever want hypothetical paper proposals, that is a separate build with its own warning."

## Note on results and where the money actually is

A daily read describes the past. It does not tell the owner what to do, and a green day on the card is not a signal to buy anything. Monitoring is not predicting. Markets are noisy, indicators lag, and a tidy summary can make randomness look like a pattern. Nothing in this module tilts the odds of a trade, because this module does not trade.

So unless the owner has a specific reason to keep watching here, point them back to the modules that earn: `modules/etsy-autopilot.md`, `modules/pod-printify.md`, and `modules/content-engine.md`. This is a nice thing to look at. Those are the things that pay.
