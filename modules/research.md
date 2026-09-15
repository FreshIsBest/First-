> For the AI building the owner's Agentic OS. Build the research module: three read-only scout agents (niche, keyword, competitor) that gather findings, draft them to the board, and feed the listing and content agents. They report. They never act.

Level: beginner-safe | Time: ~1 evening | Cost: $0 beyond your engine (web/search tools only if the task needs live data)

# Module: research (the scouts)

Every good build starts with a question the owner cannot answer off the top of their head. What should I sell next? What tags actually get found? What is the competition doing this week? The research module answers those. It is a set of scout agents that go look, come back, and write down what they found. They do not decide, buy, publish, or message anyone. They report to the board and the owner decides.

This is the scout pattern from `prompts/prompt-library.md` (section 4) promoted to a first-class module, because every buyer needs it, not just the dropshipping one. I run my own Etsy shop on this: before I draft a single listing, a scout tells me what the niche looks like and what words people search. The same three scouts feed my agency's content pipeline. Research is the cheap step that makes every expensive step land better.

## The one rule that makes research safe

**A scout is read-only. It gathers and drafts. It never takes an action.**

No buying, no listing, no posting, no messaging, no ordering. A scout's entire job ends at "here is what I found, written to the board." That is what makes it safe to run often and cheap: the worst a scout can do is be wrong, and a wrong finding sitting on the board costs nothing until the owner acts on it. Nothing risky auto-runs (rule 3), and research never proposes a risky action directly. It hands facts to the agents that do.

Because a scout only reads and writes its own findings, give it the tightest tool whitelist in the whole system: `Read`, `Write`, and web/search tools ONLY when the specific job needs live data. Never give a scout a publish, spend, or send tool. It has no reason to hold one.

## The three research jobs

Three task types, one shape. Each is a scout with a different question.

```
NICHE RESEARCH   -> what to sell        -> feeds the listing + POD agents
KEYWORD RESEARCH -> what words to use   -> feeds the listing + content agents
COMPETITOR WATCH -> what rivals do      -> feeds the owner (and sometimes the queue)
       |                                        |
       +------------ all draft to the BOARD ----+
                     (read-only, never act)
```

All three write a findings file, park it on the board, and stop. None of them acts on what they find. The owner reads the findings and decides what to build next.

## Job 1: NICHE RESEARCH (`research_niche`)

Find promising product niches with real demand signals, so the owner is not guessing what to sell.

**Role:** You are a market scout for the owner's own business (niche and goal from `PROFILE.md`). You find product niches with signs of real demand. You report findings only. You do not create listings, buy anything, or message anyone.

**Input payload (what the dispatcher hands it):**

```json
{
  "seed": "cozy-season home goods",
  "constraints": { "format": "digital + POD", "budget": "low", "country": "US" },
  "how_many": 5,
  "recent_lessons": ["skip anything needing licensed characters, it gets taken down"]
}
```

**What it looks for (demand signals, not vibes):** search interest that is rising or steady, a healthy number of existing sellers (some competition proves demand; zero sellers usually means zero buyers), price points people actually pay, and seasonality that fits the calendar. It reports the signal AND where it saw it, so the owner can sanity-check.

**Exact output format (write this JSON to `result`, artifacts as files under `board/pending/<task_id>/`):**

```json
{
  "niches": [
    {
      "name": "matte enamel camp mugs",
      "demand_signal": "steady year-round search, spikes Sept-Dec",
      "competition": "many sellers, few with a clear brand voice",
      "price_range": "$14-$22",
      "source": "public marketplace search + trends page",
      "why_it_matters": "fits the cozy niche, low production cost via POD, gift-season lift"
    }
  ],
  "skipped": [{ "name": "licensed fandom mugs", "reason": "IP takedown risk" }]
}
```

**Stop condition:** the requested number of niches (default 5) written to the board with a source and a reason each, plus anything it deliberately skipped. Then stop. Do not draft a listing. Do not pick the winner. That is the owner's call.

**How it feeds the build:** the owner reads the niche findings, picks one, and THAT approved niche becomes the input to `modules/etsy-autopilot.md` (listing drafts), `modules/pod-printify.md` (POD drafts), or the whole `modules/shop-in-a-box.md` generator. Research picks nothing; it fills the owner's shortlist so their pick is informed.

## Job 2: KEYWORD RESEARCH (`research_keywords`)

Find the tags, title words, and SEO terms a listing or post should use so it actually gets found. This is the single highest-leverage scout, because the best product with the wrong words is invisible.

**Role:** You are a keyword scout for the owner's niche (from `PROFILE.md`). You find the search terms real buyers use for a given product or topic. You report a ranked list only. You do not edit any listing or post.

**Input payload:**

```json
{
  "subject": "matte enamel camp mug",
  "surface": "etsy_listing",
  "want": ["13_tags", "title_terms", "long_tail_phrases"],
  "recent_lessons": ["single-word tags waste the 20-char slot, use 2-3 word phrases"]
}
```

`surface` matters: Etsy tags, a blog post, and a Pinterest pin reward different phrasing. Tell the scout which surface so it shapes the terms right.

**Exact output format:**

```json
{
  "tags": ["enamel camp mug", "cozy coffee mug", "matte camping mug", "outdoor gift mug"],
  "title_terms": ["Matte Enamel Camp Mug", "Cozy 12oz Coffee Mug"],
  "long_tail": ["enamel mug for coffee lovers", "campfire mug fall gift"],
  "notes": "Etsy tags cap at 20 chars each; all above fit. 'camp mug' reads higher intent than 'mug'.",
  "source": "public marketplace autocomplete + related searches"
}
```

**Stop condition:** one ranked keyword set for the requested surface, with a short note on why and where the terms came from, written to the board. Then stop. It does not touch the live listing.

**How it feeds the build:** the keyword findings are input for the drafting agents. The listing agent in `modules/etsy-autopilot.md` takes the `tags` and `title_terms` straight into its draft (13 tags, SEO title). The content agent in `modules/content-engine.md` takes `long_tail` and `notes` into captions and hashtags. Research finds the words; the drafting agents place them; the board approves before anything goes live. A clean handoff: the scout never edits a listing, it hands the terms to the agent that drafts one.

## Job 3: COMPETITOR WATCH (`research_competitors`)

Track a small set of named competitors and flag what they launch or change, so the owner is not caught flat-footed.

**Role:** You are a competitor watcher for the owner's niche (from `PROFILE.md`). You check a short, owner-provided list of competitors and report what is new or changed since last time. You report only. You do not copy their work, contact them, or act on what you see.

**Input payload:** the owner names the competitors. The scout does not go find rivals to stalk; it watches the handful the owner already cares about.

```json
{
  "watchlist": [
    { "name": "example cozy shop", "url": "https://www.etsy.com/shop/examplecozyshop" }
  ],
  "watch_for": ["new_listings", "price_changes", "new_bestsellers"],
  "last_seen": "board/pending/prior-competitor-snapshot.json"
}
```

`last_seen` is how it reports CHANGE instead of a full dump every run: it compares this pass to the last snapshot and flags only what moved.

**Exact output format:**

```json
{
  "changes": [
    {
      "competitor": "example cozy shop",
      "change": "launched a 3-mug bundle at $39",
      "signal": "bundling to lift order value",
      "source": "public shop page, seen 2026-09-14",
      "worth_owner_attention": true
    }
  ],
  "snapshot": "board/pending/57/competitor-snapshot-2026-09-14.json",
  "nothing_changed": []
}
```

**Stop condition:** a change list (which may be empty) plus a fresh snapshot for next time, written to the board. Then stop. It flags; it never copies a listing or launches a matching product. If a change is worth acting on, the owner decides and enqueues the build.

**How it feeds the build:** most competitor findings feed the owner directly as a heads-up. Some become the seed for a new build (the owner sees a bundle working and enqueues a bundle listing draft). The scout never crosses that line itself. Watching is safe; reacting is a decision.

## Honest note on data sources (read before wiring web tools)

Scouts are only as clean as where they look, and this is where a build gets the owner in trouble if you are sloppy.

- **Public information only.** Marketplace search pages, public shop pages, trends and autocomplete, published articles. Things anyone can see in a browser without logging in.
- **No scraping that violates a site's terms.** Many marketplaces restrict automated access, rate-limited crawling, or bulk data collection in their terms of service, even for pages that are public to a human. A scout that hammers a site or pulls data a site forbids is a real account and legal risk, not a clever hack. Before you point a scout at any site with a web tool, check that site's terms and robots rules, and prefer an official API where one exists (Etsy's API for Etsy data, for example, per `modules/etsy-autopilot.md`).
- **Cross-reference the compliance section at the end of `08-gotchas.md`.** The owner is responsible for the platforms they touch. the compliance section at the end of `08-gotchas.md` covers marketplace terms on automation and where to check them. Read it before wiring any live-data scout, and when in doubt, gather less and rely on the owner's own manual read of a competitor page. A slower scout that stays inside the rules beats a fast one that gets an account banned.
- **Cite every finding.** Each result carries where it came from (see the `source` field in all three outputs). A finding with no source is a guess, and a guess dressed as a fact is worse than no finding. If a source is thin, the scout says so instead of padding.

## Guardrails for this module

- **Read-only, always.** No scout ever holds a publish, spend, send, or order tool. Tightest whitelist in the system: `Read`, `Write`, and web/search ONLY when live data is needed.
- **Findings draft to the board; they do not become actions.** A scout writes findings and stops. Turning a finding into a build is a separate, owner-initiated task.
- **Caps still apply.** A scout with a web tool can loop or run long. Give it the same hard timeout and max-runtime stop as every other agent (rule 4). One narrow question per run, per the lessons pattern; a broad question is what makes a scout time out.
- **Fresh session per task.** Each scout run is a new headless call (rule 2). No scout accumulates a running memory of a competitor across runs; the `last_seen` snapshot on disk is its memory, not a long-lived session.
- **Record what worked.** When a scout's findings lead to a good (or bad) build, append one line to `lessons.md` so the next run asks a sharper question. The rejection and dead-end reasons are the most useful thing you can save.

## Prove it works

Before you call this done: run one real scout of each type against the owner's actual niche. Show the owner three real findings files on the board, each with real sources. Then hand one keyword set to the listing agent and confirm the terms land in a real draft. Do not claim the module works until the owner has seen live findings and one clean handoff into a drafting agent (rule 5).

## Note on results

A scout reports signals, not certainties. Demand signals can be noisy, competitor moves can be flukes, and a keyword that looks strong can still underperform. Research tilts the odds; it does not remove the risk. Results are not typical. Use the findings to make a better-informed pick, then judge the pick by what it actually earns.
