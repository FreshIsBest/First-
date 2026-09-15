> For the buyer's AI. Stand up one working headless agent that does one real task end to end, and prove it runs before anything else.

# Your first agent

This is the template for the crew you build at Step 3 of `03-build-order.md`. Build ONE agent first, one small task, run it by hand, show the owner real output. That is the whole job of this step, and every other agent is a copy of its shape.

**Timing note.** In the build order, the crew comes BEFORE the automated system (queue, dispatcher, board), which is built at Step 6. So there is no dispatcher spawning this agent yet: you run it BY HAND, and the owner approves its output directly. Where this file mentions the dispatcher spawning fresh agents or writing to `board/pending/`, that is how it works once the system exists at Step 6; before then, you run the agent yourself and save its output to a simple drafts folder the owner can open. The SHAPE of the agent (tight prompt, whitelisted tools, timeout, fresh session, output the owner reviews) is identical either way, which is why you get it right here first.

Do not build five agents at once. Build one, prove it, then build the rest of the crew from this template.

## What an agent is here

An agent is not a chatbot you talk to. It is a single headless call to the owner's AI engine, with a tight prompt and a short list of allowed tools, that runs once, does one task, records its result, and exits. Fresh session every time. No memory of the last task. No open chat window.

How the result is stored (the one storage model, same as `02-architecture.md`): the agent's result is recorded as JSON in the task's `result` column in the DB. Small text output can live right in that JSON. Anything large (an image, a video, a rendered HTML preview) is written as a file under `board/pending/<task_id>/` and its path is stored in `result.preview`. For this first agent the output is short text, so we write a preview file the board can show and point `result.preview` at it.

That is the pattern. Everything below is how to build one correctly.

## Step 1: pick one small real task

Pull one genuinely useful task from the owner's money-maker (read `PROFILE.md`). Small, real, and safe to run. Good first tasks:

- Etsy shop: "Draft one product description for [item]."
- Print-on-demand: "Write 3 title options and a tag list for one design."
- Content channel: "Summarize today's shop stats into 3 plain sentences."
- Any: "Read today's numbers from [file] and write a one-paragraph summary."

Rules for the pick:
- It must produce **text output** you can show the owner. No publishing, no spending, no sending. That is later, and it goes through the board.
- It must be something the owner actually needs. Do not invent a toy task. The point is to prove the system does real work on the first try.
- Keep it to one unit: one description, one summary, one set of titles. Not "write all my listings."

Tell the owner which task you picked and why, in one sentence, before you build it.

## Step 2: the fresh-session-per-task principle

Every agent run starts clean. It gets its instructions and its input in the prompt, does the task, and dies. It does not carry context from the last task into the next one.

Why this matters, and why you must hold it:
- **No context bleed.** Task 2 never inherits task 1's mistakes, half-finished reasoning, or stale data.
- **Cheap and predictable.** A fresh short session costs less and behaves the same every time. A long-lived agent drifts and burns tokens.
- **Recoverable.** If a run dies or hangs, the dispatcher just respawns it. Nothing is half-held in memory.

So: no persistent chat session for agents. Each task = one new headless call. The dispatcher (from `03-build-order.md`) is what pulls the next task and spawns the next fresh agent.

## Step 3: write the agent as one headless call

An agent is a small script the dispatcher runs. It builds a tight prompt, calls the engine headless with a whitelisted tool set and a timeout, captures the output, and writes it where the board can show it.

Here is the shape. This is the Claude Code Pro path (`claude -p`). If the owner is on OpenRouter + Hermes, the same script calls their endpoint instead; the structure is identical.

```bash
#!/usr/bin/env bash
# agents/draft-description.sh
# One task: draft a single product description. Fresh session. Text only. No publishing.

set -euo pipefail

ITEM="$1"                              # the input for this run
TASK_ID="$2"                           # the task id from the queue
OUT="board/pending/${TASK_ID}/draft.md"    # preview file for this task
mkdir -p "board/pending/${TASK_ID}"

# Portable hard timeout. `timeout` is GNU coreutils: it exists on Linux and in
# WSL, and does NOT exist on a stock Mac, where this script would die with
# "command not found" on the owner's very first agent run. Use whichever is
# present, and otherwise run a plain background watchdog, which needs nothing
# installed and works everywhere.
run_capped() {
  local secs="$1"; shift
  if command -v timeout  >/dev/null 2>&1; then timeout  "$secs" "$@"; return $?; fi
  if command -v gtimeout >/dev/null 2>&1; then gtimeout "$secs" "$@"; return $?; fi
  "$@" &
  local pid=$!
  ( sleep "$secs"; kill -TERM "$pid" 2>/dev/null ) &
  local watcher=$!
  wait "$pid" 2>/dev/null
  local code=$?
  kill -TERM "$watcher" 2>/dev/null
  return $code
}

PROMPT="You are a product copywriter for an Etsy shop.
TASK: Write one product description for this item: ${ITEM}.
RULES: 120 to 180 words. Warm, plain, honest. No hype, no fake scarcity.
End with a 5-line bullet list of the key features.
OUTPUT: return only the description. No preamble, no sign-off."

# Headless. Whitelisted tools only. Hard timeout so it can never hang forever.
run_capped 120 claude -p "$PROMPT" \
  --allowedTools "Read,Write" \
  > "$OUT"

echo "Wrote draft to $OUT"
```

What each guardrail is doing, and why you keep all of them:

- **`claude -p "..."`** runs headless: one prompt in, one answer out, no interactive chat. This is what makes it an agent and not a conversation.
- **Tight prompt.** Role, task, rules, output format, all spelled out. The agent has no chat history to lean on, so the prompt carries everything. Vague prompt = vague output.
- **`--allowedTools "Read,Write"`** is the whitelist. This agent can read files and write its result. It cannot run shell commands, hit the web, or touch anything else. Give an agent only the tools its one task needs. A drafting agent does not get network or execution tools. (See `prompts/prompt-library.md` for the generic template.)
- **`run_capped 120`** is the hard stop. If a run hangs, it dies at 120 seconds instead of running forever and burning tokens. Every agent gets a timeout. No exceptions. The helper exists because the obvious version, plain `timeout 120`, is a GNU coreutils command that a stock Mac does not have, so on a Mac the owner's first ever agent run would fail with "command not found" and look like the whole system is broken. Check the owner's OS answer from `PROFILE.md`, and if they are on macOS, run the script once in front of them to confirm the fallback path actually fires.
- **`> "$OUT"`** writes the preview to `board/pending/<task_id>/`, where the approval board can show it to the owner, and that path is what goes in the task's `result.preview`. The agent's job is to produce, not to publish. The owner decides what happens next.

## Step 4: run it and capture the output

Run the agent for real, with a real input:

```bash
bash agents/draft-description.sh "hand-poured lavender soy candle, 8oz" 1
```

Then read the file it wrote (task id 1 here):

```bash
cat board/pending/1/draft.md
```

If it produced a clean draft: good. If it errored, timed out, or wrote junk, fix the prompt or the tool list and run it again. Do not move on with a broken agent.

## Step 5: show the owner and prove it ran

This is rule 5 from `CLAUDE.md`, and it is not optional. Do not tell the owner the agent works. Show them.

- Show the **actual command you ran** and its exit (success or the error).
- Show the **real content of the output file**, pasted in, not described.
- Say plainly whether it worked. If it took two tries, say that too.

Never say "this should work" or "the agent is done" on faith. If you did not run it and read the output, it is not done. Run it, show the output, then call it done.

Then tell the owner, in one or two plain sentences, what you just built: "This is your first agent. It takes an item, drafts a description, and drops it in your approval pile. Nothing publishes on its own. You approve what ships."

## Step 6: this shape scales to every agent

You just built the template. Every future agent is the same five parts:

1. **A tight prompt** (role, task, rules, output format, stop condition).
2. **A whitelist** of only the tools that one task needs.
3. **A timeout** so it can never run away.
4. **An output written where the board can see it** (`board/pending/`).
5. **Fresh session per run**, spawned by the dispatcher.

To add the next agent, copy this file, swap the prompt and the input, adjust the tool whitelist to the new task, keep the timeout and the output path. A research agent gets read/web tools. A stats agent gets read only. A posting agent still writes to `board/pending/` and never sends on its own; the board and the owner's click do that.

Do not build a giant do-everything agent. Many small, single-purpose agents, each proven the way you just proved this one. That is the system.

When this first agent runs clean and the owner has seen its real output, move on to their money-maker module in `modules/`.
