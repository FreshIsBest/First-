> The architecture the buyer builds. For the AI to build from and to explain to the owner in plain language.

# The architecture: how an Agentic OS actually works

This is the map of the whole system, so you (the AI) can hold the shape in your head and explain it to the owner in plain language. Note on timing: you build the automated core described here (the queue, the dispatcher, the board) at Step 6 of `03-build-order.md`, not first. The agents get built earlier, at Step 3, and run by hand until the automation exists. So read this as the picture of where it all ends up, and build the pieces in the order `03-build-order.md` gives. Do not overcomplicate it. The whole system is five small parts and one rule: nothing risky runs without a human click.

This is the same pattern behind the system that runs my own agency (leads, invoices, analytics, outreach) and the one that runs my own Etsy shop (daily sales digest, print-on-demand pipeline, pin generation). Both are just AI agents wired to a queue and a board. That is all this is.

## The plain-language version (say this to the owner)

"Think of your business as a to-do list that a robot works through for you. Tasks go in a list. A manager pulls one task at a time and hands it to a fresh worker. Each worker does one job and goes away. Anything risky (spending money, posting a listing, sending a message) does not happen on its own. It waits on a board for you to say yes. You stay in charge. The system does the grunt work."

That is the whole thing. Now here are the five parts you build.

## The five parts

### 1. The QUEUE (the to-do list)
A list of tasks waiting to be done. Start with the simplest thing that works: one SQLite table, or a single JSON file if the owner has no database yet. Prefer SQLite because it handles concurrent writes and status updates cleanly.

Each task is a row with, at minimum:
- `id`: unique task id
- `type`: what kind of job (for example `draft_listing`, `draft_post`, `research_product`)
- `payload`: the input data (JSON: which product, which platform, etc.)
- `status`: `queued`, `running`, `needs_approval`, `done`, `failed`, or `rejected`
- `attempts`: how many times it has been tried
- `batch_id`: optional group id (nullable) so child tasks (like a whole shop-in-a-box run) can be reviewed together
- `not_before`: optional ISO time (nullable) a task must not run before, for scheduled or manager tasks
- `created_at`, `started_at`, `finished_at`: timestamps
- `result`: the agent's output as JSON (large artifacts live on disk with their paths stored in `result.preview`, see the storage note below)

A minimal SQLite table (canonical schema, use it verbatim):

```sql
CREATE TABLE tasks (
  id          INTEGER PRIMARY KEY AUTOINCREMENT,
  type        TEXT NOT NULL,
  payload     TEXT,                 -- JSON input
  status      TEXT NOT NULL DEFAULT 'queued',  -- queued|running|needs_approval|done|failed|rejected
  attempts    INTEGER NOT NULL DEFAULT 0,
  batch_id    TEXT,                 -- nullable, groups child tasks for one board review
  not_before  TEXT,                 -- nullable, ISO time a task must not run before
  result      TEXT,                 -- JSON by the agent; large artifacts as paths in result.preview
  created_at  TEXT NOT NULL,
  started_at  TEXT,
  finished_at TEXT
);
```

Adding work to the system is just inserting a row. That is on purpose. The queue is dumb and that is its strength.

**Where results live (one storage model, use it everywhere).** The agent writes its result as JSON into the task's `result` column. That row is the record of truth. Large artifacts (images, video, a rendered HTML preview) are too big for a DB cell, so they get written as files under `board/pending/<task_id>/` and their paths are stored in `result.preview` (and `result.files`). So: small structured output in the DB, big binaries on disk with paths in `result`. Note that `board/pending/` is a folder path, not a task status.

### 2. The DISPATCHER (the manager)
A loop. It is the only thing running continuously, and only ONE copy of it may run at a time: on start it writes a lockfile with its PID and refuses to run if another instance already holds that lock, so two dispatchers can never grab the same tasks. Every cycle it does five things:

1. **Pull one task.** Grab the oldest `queued` task, mark it `running`, stamp `started_at`. Do this in a single atomic update so two cycles never grab the same task.
2. **Spawn ONE fresh agent for it.** Shell out to a single headless `claude -p` call (see part 3). One task, one agent, one fresh session.
3. **Enforce a timeout.** Wrap the agent call in a hard timeout (for example 5 minutes). If it runs longer, kill it and mark the task `failed`. An agent that hangs must never freeze the whole system.
4. **Record the result.** Write the agent's output into `result`, set status to `done`, `failed`, or `needs_approval`, stamp `finished_at`.
5. **Recover orphans.** On startup, and every few cycles, find tasks stuck in `running` with an old `started_at` (the process died mid-task) and reset them to `queued` so they get retried. Cap retries with `attempts` so a poison task cannot loop forever.

The dispatcher is small on purpose, maybe 100 lines. It does not know how to make a listing or write a post. It only knows how to hand work to a worker and clean up after it.

### 3. The AGENTS (the workers)
An agent is not a running service. It is a single headless call to your AI, spawned fresh for one task and then gone. In Claude Code that is `claude -p` (print mode, non-interactive). With OpenRouter and Hermes it is one API call with a system prompt.

Each agent gets:
- **A tight prompt.** One job, stated plainly, with the task payload injected. No "and also handle anything else."
- **A whitelisted tool set.** Only the tools that one job needs. A listing-writer agent gets file read/write, not shell and not the ability to spend money. Give the least power that does the job.
- **A fresh session.** No memory of other tasks. No context bleed. This keeps agents cheap, fast, and predictable, and stops one task's mess from poisoning the next.

A dispatcher spawning an agent looks roughly like this:

```bash
claude -p "$(cat prompts/draft-listing.md)

TASK PAYLOAD:
$payload

Write your result as JSON to result.json and stop." \
  --allowedTools "Read,Write" \
  --output-format json
```

The prompt file (`prompts/draft-listing.md`) is the agent's whole brain for that job. You will build a small library of these, one per task type.

Why fresh and single-purpose: a giant do-everything agent that stays alive gets confused, expensive, and dangerous. Ten small agents that each do one thing and disappear are cheap and easy to reason about. This is rule 2 in `CLAUDE.md`: one task, one fresh agent.

### 4. The BOARD (the approval surface)
The human-in-the-loop gate. Anything risky does not execute. The agent instead sets the task status to `needs_approval` and writes what it wants to do into `result`. It stops there. The board reads the `needs_approval` rows and shows `result.preview` (the rendered artifact, never raw JSON) so the owner reviews the real thing. Nothing happens until the owner approves.

Risky means: publishing a listing, spending money, sending an email or DM, placing an order, anything hard to undo.

The board can start as nothing more than a command that lists pending items and lets the owner approve or reject:

```
$ os board
[12] draft_listing  -> "Cozy Fall Mug, $18, 3 tags..."  approve? (y/n)
[13] send_email    -> "Reply to buyer question..."      approve? (y/n)
```

On approve, the task enqueues a `queued` follow-up ACTION task that actually performs the thing (publish, send, spend). On reject, the task is set to status `rejected`, the note is stored in `result.reject_note`, and a revision task may be enqueued carrying the owner's `owner_feedback` in its payload. Later, once the system earns, this same approval logic gets a real UI. Not before. The logic matters; the looks do not yet.

### 5. The GUARDRAILS (the safety rails)
Rules enforced in code, not in good intentions:

- **Spend caps.** Every loop and every money action has a hard ceiling: max spend per day, max tasks per run, max runtime. Ask the owner for these numbers during the interview and enforce them. An agent that could run forever gets a hard stop.
- **No auto-publish.** The board gate above is a guardrail. Publishing and spending route through approval, always. Build the guardrail before you build the action.
- **A lessons file (the system's memory).** A plain text file, `lessons.md`, where the AI writes down mistakes, one readable line each, so it never repeats them. When something goes wrong (a rejected listing, a flagged post, a repeated timeout), it adds a note. Before running a job, the AI reads the recent notes for that job type and follows them. That is the whole "learning" loop: no training, no pipeline, just a running list of what was learned the hard way. Both you and the owner can read it.

Example lines (plain English, human-readable):

```
- [draft_listing] Etsy rejects tags longer than 20 characters. Keep every tag under 20.
- [draft_post] Posts with more than 2 links get flagged. Use one link max.
```

## How a task flows

```
   +---------+     +------------+     +---------+     +---------+     +------+
   |  QUEUE  | --> | DISPATCHER | --> |  AGENT  | --> |  BOARD  | --> | DONE |
   | (tasks) |     |  (1 loop)  |     | (claude |     | (human  |     |      |
   +---------+     +------------+     |   -p)   |     |  click) |     +------+
        ^          pulls one,         +---------+     +---------+
        |          spawns fresh,           |      risky? waits        |
        |          times out,              |      for approval        |
        |          recovers orphans        |                          |
        |                                  |  safe? goes straight     |
        +----------------------------------+-------- to done ---------+
              failed / retry / follow-up action re-enters the queue

   GUARDRAILS wrap the whole thing: spend caps, no auto-publish, lessons file
```

Read it as: work enters the queue, the dispatcher pulls one item and spawns a fresh agent, the agent does the job. If the job is safe (research, drafting, generating), it finishes and is done. If the job is risky (publish, spend, send), it parks on the board and waits for a human click before a follow-up task performs the real action. Failures and approved actions go back into the queue. Guardrails sit over all of it.

## What this is NOT
- It is not a web app you build first. The queue and dispatcher can run headless with no interface at all. The board can be a text prompt.
- It is not one big smart agent. It is a dumb loop plus many small single-use agents.
- It is not autonomous spending. Nothing risky runs without the owner. That is the point.

## What to tell the owner before you build
Say, in plain words: "I am going to build five small parts: a task list, a manager loop, the workers, an approval board, and the safety caps. No dashboard, no logo, no colors yet. Once it is actually making or doing something real, we make it look good. Sound good?" Get a yes, then go to `03-build-order.md` and build in that exact order.
