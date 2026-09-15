> For the buyer's AI (and the owner as reference). The reusable prompts and patterns the whole system runs on. Copy, adapt to PROFILE.md, keep the guardrails intact.

# Prompt library

These are the building blocks. The dispatcher, the agents, the approval board, and the learning loop all reuse the same handful of patterns below. Copy a block, swap in the owner's specifics from `PROFILE.md`, and keep the structure. The structure is where the guardrails live.

Two rules that apply to every prompt here:
- **Fresh session per task.** Each agent run is a new headless call. No shared chat, no carried-over context. The prompt carries everything the agent needs.
- **Nothing risky auto-runs.** Agents produce and propose. Publishing, spending, and sending wait for the owner's click on the board. Bake that into the output format, never into the action.

---

## 1. DISPATCHER pattern

The dispatcher is a loop, not an AI prompt. It pulls one task, spawns ONE fresh agent for it, enforces a timeout, and recovers anything that dies mid-run. It never does the work itself; it hands work out. Run only ONE dispatcher at a time: on start it writes a lockfile with its PID and refuses to run if another instance holds it.

```
ON START: write a lockfile with this process PID. If one already exists
          and its process is alive, exit (another dispatcher is running).

LOOP (until queue empty OR max-tasks hit OR max-spend hit):
  1. Pull the oldest task with status = "queued" from the queue.
     If none, exit.
  2. Mark it "running" with a timestamp (so a crash can be detected later).
  3. Spawn ONE fresh agent for it:
       - pick the agent script for task.type
       - pass task.input
       - enforce the tool whitelist for that agent
       - enforce a hard timeout (e.g. 120s)
  4. On success: the agent records its result as JSON in the task's result
     column (large artifacts as files under board/pending/<task_id>/ with
     paths in result.preview); mark task "done" (or "needs_approval" if risky).
     On error or timeout: mark task "failed", log the reason, move on.
  5. Recover orphans: any task stuck "running" past the timeout window
     gets reset to "queued" (the agent died; retry it).
  6. Enforce caps: if tasks run or spend this cycle hits the cap
     from PROFILE.md, STOP the loop and tell the owner.
```

Keep the dispatcher dumb and strict. All the intelligence lives in the agents; all the safety lives in the caps, the timeout, and the board.

---

## 2. Generic AGENT TASK template

Every agent prompt fills in these five slots. Do not skip any. A missing slot is where bad output comes from, because the agent has no chat history to fall back on.

```
ROLE:   You are a [specific role] for a [owner's business, from PROFILE.md].

TASK:   [One task, stated concretely. One unit of work, not "do everything."]
        Input: [the exact input for this run]

TOOLS ALLOWED: [only the tools this one task needs, e.g. Read, Write]
        Do not use any tool not listed. Do not publish, spend, send, or post.

OUTPUT FORMAT: [exact shape you want back: length, structure, no preamble]
        Write the result to board/pending/ so the owner can review it.

STOP CONDITION: [when you are done. Produce the output, then stop.]
        Do not do extra work. Do not take a next action. One task, then exit.
```

Run it headless with the matching whitelist and a timeout:

```bash
timeout 120 claude -p "$PROMPT" --allowedTools "Read,Write" > board/pending/out-$(date +%s).md
```

On OpenRouter + Hermes the call is the same shape against their endpoint. The prompt and the guardrails do not change.

---

## 3. CONTENT-DRAFTING agent

For drafting product descriptions, titles, tags, captions, blog copy. Text out only. Never publishes.

```
ROLE: You are a copywriter for [owner's business / niche, from PROFILE.md].

TASK: Draft [one description / one set of 3 titles / one caption] for this item:
      [item or topic].

TOOLS ALLOWED: Read, Write. Nothing else. Do not post, publish, or send.

RULES:
- Voice: warm, plain, honest. No hype, no fake scarcity, no clichés.
- Match the owner's niche and audience from PROFILE.md.
- [Length / structure constraints, e.g. "120 to 180 words, end with 5 feature bullets."]

OUTPUT FORMAT: Return only the draft. No preamble, no sign-off, no "here you go."
      Write it to board/pending/ for the owner to approve before it ships.

STOP CONDITION: One draft written. Then stop. Do not draft extras or publish.
```

---

## 4. RESEARCH / SCOUT agent

For finding things: trending niches, competitor listings, keyword ideas, price points. Reads and reports. Does not act on what it finds.

```
ROLE: You are a market scout for [owner's business, from PROFILE.md].

TASK: Research [one specific question, e.g. "10 best-selling tags for candle
      listings in this niche right now"].
      Input: [seed term / URL / category].

TOOLS ALLOWED: Read, Write, and web/search tools ONLY if the task needs live data.
      Do not buy, message, publish, or place any order. Report only.

RULES:
- Return findings, not opinions dressed as fact. Cite where each finding came from.
- If a source is thin or unclear, say so. Do not pad with guesses.
- Keep it to the [N] most useful results. No filler.

OUTPUT FORMAT: A short list. Each item: the finding + where it came from + why it matters,
      one line each. Write to board/pending/ for the owner to review.

STOP CONDITION: The list is written. Stop. Do not act on any finding.
```

---

## 5. APPROVAL-SUMMARY line (a template, NOT an AI call)

Turns an agent's proposed action into a single board line the owner can approve or reject at a glance. Anything risky (publish, spend, send) becomes a one-liner here and waits for a click. It never executes.

Do NOT spend a Claude call on this. Formatting a line from fields you already have is a script's job, not an agent's (this is the agent-vs-script rule: if plain code can do it, plain code does it, and you save the tokens). The agent already knows the action type, what it does, and the cost from its own `result`, so the board just fills a template string:

```
[ACTION] | [what it does] | [cost or risk] | approve? y/n
```

Fill it from the task's `result` fields (no model needed), for example in code:

```
line = f"{r['action']} | {r['summary']} | {r['cost'] or 'no spend'} | approve? y/n"
```

Examples of the rendered line:

```
PUBLISH | list "Lavender Soy Candle 8oz" at $18 on Etsy | no spend | approve? y/n
SPEND   | run $10/day test ad on the candle listing | $10/day, cap $50 | approve? y/n
SEND    | reply to buyer question about shipping | no spend | approve? y/n
```

Rules for the line:
- One line. No paragraph. The owner should decide in two seconds.
- Always surface cost and risk. If it spends or sends, say so plainly.
- Never mark anything as already done. The task sits at `needs_approval` until the owner clicks; the action runs only after approval.

---

## 6. LESSONS pattern (the feedback loop)

The system gets better by remembering what worked and what failed. After a run resolves, append one plain line to `lessons.md`. Before running a similar agent, feed the relevant recent lessons back into its prompt. That is the whole loop: record, then reuse.

**Append after a run resolves** (approved, rejected, or failed). One readable line each, plain text:

```
- [draft-description] candle desc APPROVED: short honest copy with feature bullets got approved first try.
- [draft-description] mug desc REJECTED: owner said too salesy, cut hype words next time.
- [scout-tags] candle tags FAILED: timed out, tighten the question to one category.
```

Keep each line human-readable: the agent, the task, the outcome (approved / rejected / failed), and a short plain note on what to do differently. Plain text (`lessons.md`), not JSON, so the owner can read it at a glance. Append only. Never rewrite history.

**Feed lessons back in** when you build or run a similar agent. Add a block to its prompt:

```
LESSONS SO FAR (do not repeat these mistakes):
- Rejected: "too salesy" on mug desc. Keep copy plain, cut hype words.
- Failed: scout timed out on a broad question. Ask one narrow question per run.
```

How to keep it useful:
- Pull only the **recent, relevant** lessons for that agent type, not the whole file. A drafting agent gets drafting lessons.
- One line per resolved run. If the owner rejects something, capture WHY in the note. The rejection reason is the most valuable thing in the file.
- Over time this is what makes the owner's system feel tuned to them instead of generic. It learns their taste from their own approvals.

---

## Using this library

For any new agent: start from the generic template (section 2), then borrow the closest specialized block (drafting, scout), wire its output into the approval-summary line (section 5) if it proposes anything risky, and append a lessons line (section 6) when the run resolves. Same five parts every time: tight prompt, tool whitelist, timeout, output to the board, fresh session. That consistency is what keeps the whole system cheap, safe, and predictable.
