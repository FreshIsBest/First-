> For the AI building the owner's Agentic OS. The once-daily conductor that turns "a queue you feed by hand" into "an OS that runs your day." On a schedule it reads the digest, the board backlog, lessons.md, and the content calendar, then ENQUEUES the day's work and sends the owner a short morning brief. Like shop-in-a-box it is a conductor: it enqueues and reports, it NEVER performs a risky action itself. Build this only AFTER the core system works.

# Module: the manager agent

Level: advanced | Time: ~1 evening (after the core system is proven) | Cost: $0 beyond the engine (one extra agent run per day)

Up to now the owner has been the one who decides what runs. They open the board, they insert tasks, they kick the day off. That is fine, and it is the right way to start. The manager agent is the piece that removes that daily chore. Once a day, on a schedule, it looks at the state of the business and enqueues the day's work itself, then tells the owner what it did in about five lines. The owner still approves every risky thing. What changes is that the owner no longer has to remember to feed the queue.

Read this clearly before you build it: **the manager is not smarter than the system. It is a conductor, exactly like shop-in-a-box (`modules/shop-in-a-box.md`).** It does not draft a listing, render a pin, or call any platform. It reads what already happened, decides what should happen today, and enqueues the existing task types the owner already built and proved. Every real action still runs in its own fresh agent through the dispatcher, and every risky action still parks on the board. The manager just plans the day and hands out the work.

## When to build this (and when not to)

This is an advanced feature (build after the core works). Do not build it for someone whose core system is not yet earning or running clean. A manager that enqueues work into a broken pipeline just breaks more, faster, and does it while the owner is asleep.

Build this only when all of these are true:

- The spine is built and proven: queue, dispatcher, board, guardrails (`02-architecture.md`, `03-build-order.md`, `04-first-agent.md`).
- At least one money-maker works end to end: a real digest, a real draft on the board, a real approval the owner has clicked (`modules/etsy-autopilot.md`, `modules/content-engine.md`, or whichever the owner runs).
- The owner has run the system by hand for enough days to trust the board and to know roughly what a normal day of work looks like.
- The owner has a content calendar or some standing plan the manager can read (the one shop-in-a-box writes, `data/content-calendar.md`, is a fine source).

If any of those is missing, stop and say so plainly: "The manager runs your day for you, so the day it runs has to already work. Let's get the core system earning and prove you trust the board first, then I'll hand it the keys to the morning." Then go build or prove the missing piece. This is the reward for a working system, not a way to skip building one.

Say the promotion out loud to the owner, because it is the whole point: "Right now you have a queue you feed. After this you have an OS that runs your day. You still approve everything that matters. You just stop being the one who has to start it."

## What the manager does each morning (read, plan, enqueue, brief)

Once a day it runs a single agent task, type `run_manager`, that does four things and then stops:

1. **Reads the state of the business.** The latest digest (yesterday's sales, traffic, whatever the owner's money-makers report), the current board backlog (what is still sitting in `needs_approval` waiting on the owner), `lessons.md` (what the system learned the hard way), and the content calendar (what was planned for today).
2. **Decides the day's work.** From that, it picks the tasks the owner's money-makers need today: draft the listings the calendar calls for, generate the pins and posts due, kick off the research or competitor watch that is scheduled, whatever the plan and the numbers point to.
3. **Enqueues those tasks.** It inserts the existing task types into the queue, with `not_before` times spread across the day so work does not all land at once. It does not do the work. It hands specs to the agents that already know how.
4. **Writes a short morning brief.** About five lines to the owner: what it queued and why, and what is still waiting on the owner's approval from before. That brief is the manager's whole visible output. Everything else is just rows in the queue.

That is the entire job. It reads, it plans, it enqueues, it reports. It touches nothing risky.

The flow, in plain ASCII:

```
  daily schedule fires (local scheduler or VPS)
     |
     v
  run_manager agent  (the conductor: reads state, plans the day, enqueues, briefs, then STOPS)
     |
     +-- reads: latest digest + board backlog + lessons.md + content-calendar.md
     |
     +-- enqueue draft_listing / make_pins / draft_post / research ...  (with not_before spread across the day)
     |
     +-- writes data/morning-brief.md  (~5 lines: what I queued and why, what is waiting on you)
     |
     v
  dispatcher runs each enqueued task in its OWN fresh agent, at or after its not_before time
     |
     v
  safe work (drafts, research, generated assets) finishes to done
  risky work (publish, send, spend) parks in needs_approval on the BOARD
     |
     v
  owner reads the brief, does one review pass over the board, clicks approve on what they want live
```

Notice what the manager is NOT in that diagram. It is not in the path between the board and a live action. It never publishes, sends, or spends. It only sits at the front, deciding what enters the queue.

## The manager task shape (its prompt)

Build this as one task type with a tight prompt, same as every other agent (rule 2: one task, one fresh agent). Here is the shape to write into `prompts/run-manager.md`. Keep it plain and specific to the owner's actual money-makers from `PROFILE.md`.

- **Role.** "You are the daily manager for the owner's business. Your job is to plan today's work and enqueue it. You do NOT do the work yourself, and you NEVER publish, send, post, or spend. You read the current state, decide what should run today, insert those tasks into the queue, and write a short brief. Then you stop."
- **What it reads (inputs handed in the payload, or paths it is told to read).**
  - The latest digest (yesterday's numbers for each money-maker).
  - The current board backlog: every task still in `needs_approval`, so the manager knows what is already waiting on the owner and does not pile on more of the same.
  - `lessons.md`: the plain-text memory. The manager follows these when it plans (if a lesson says a task type keeps failing a certain way, it accounts for that).
  - `data/content-calendar.md`: what was planned for today.
  - `PROFILE.md`: the owner's money-makers, brand voice, spend caps, and daily task cap.
- **What it enqueues.** Only existing, proven task types, inserted as normal `queued` rows with a `payload` and a `not_before` time. For example: the `draft_listing` tasks the calendar calls for today, the `make_pins` / `draft_post` tasks due today, a `research` or competitor-watch task if one is scheduled (`modules/research.md`). It sets `not_before` so the work spreads across the day (see the scheduling note below). It does NOT enqueue any action task that publishes, sends, or spends. Those only ever get created by the owner approving something on the board, never by the manager.
- **The brief format.** After enqueuing, it writes about five lines to `data/morning-brief.md`, plain text, for the owner. Keep it to this shape:

```
Morning brief  2026-07-14

Queued today: 3 listing drafts (fall mugs, per the calendar), 4 pins for yesterday's
  top listing, 1 competitor-watch pass. Spread from 9am to 4pm.
Why: yesterday's digest showed the mug listing outsold the rest 3 to 1, so I front-loaded
  more mug work and pins pointed at it.
Waiting on you: 2 listings and 1 POD draft from yesterday are still on the board unreviewed.
Nothing was published, sent, or spent. All of the above is drafts and plans for your approval.
```

- **The stop condition.** "Once you have enqueued today's tasks and written the brief, you are done. Do not run any of the tasks yourself. Do not loop. Do not enqueue more than the owner's daily task cap. Write your result as JSON (what you queued, and the brief text) and stop." One run per day, one pass, then gone. Like every agent, it is a single fresh call that ends.

Give this agent a deliberately small tool set: read access to the digest, board, lessons, calendar, and profile, and write access to the queue (inserting rows) and the brief file. It does NOT get the tools that publish, send, or spend. It cannot do a risky thing because it is never handed the ability to (least power that does the job, from `02-architecture.md` part 3).

## Scheduling: how the manager runs each day

The manager is the first agent that runs on a clock instead of when the owner starts it. There are two honest ways to schedule it, matching the two runtime paths in `05-infrastructure.md`.

**Local (the recommended default for most owners).** Schedule the daily manager run with the operating system's own scheduler. On Mac or Linux that is cron; on Windows it is Task Scheduler (the AI translates the job accordingly, per `CLAUDE.md` and `05-infrastructure.md`). A single daily entry that runs the manager task once each morning:

```
# cron: run the manager once a day at 8am (Mac/Linux)
0 8 * * *  cd /path/to/os && node enqueue-manager.js >> logs/manager.log 2>&1
```

Here `enqueue-manager.js` does one small thing: it inserts a single `run_manager` task into the queue (with `not_before` set to now), and the dispatcher picks it up on its next cycle and spawns the fresh agent. The scheduler does not run the AI directly. It just drops one task in the queue, and the system the owner already built takes it from there. That keeps one pattern for everything: work is always a row in the queue.

Be honest about the local tradeoff, because it is the exact gotcha from `08-gotchas.md`: **a local scheduled job only runs while the machine is awake.** If the computer is asleep or off at 8am, the manager does not run that morning. For a manager that is often fine (the owner runs it when they sit down, or catches up the next day). If the owner needs it to run every single day no matter what, that is the reason to move to the VPS.

**VPS (for true 24/7, or a weak/old local machine).** On the always-on Linux server from `05-infrastructure.md`, the same cron line runs, but the machine never sleeps, so the manager runs every morning without fail. The dispatcher runs there as a background service (one instance, holding its lockfile, per the single-instance rule in `02-architecture.md`), and the daily cron entry enqueues the manager task on schedule. This is the setup that makes it feel like a real OS: the owner wakes up to a brief that was written whether or not their laptop was open.

Either way, the rule from the architecture holds: only ONE dispatcher runs at a time, and the manager is just another task it pulls. You are not adding a second always-on process. You are adding one scheduled insert.

## Using `not_before` to spread the day

This is what `not_before` on the tasks schema is for (`02-architecture.md`). When the manager enqueues the day's work at 8am, it does not want all of it to run at 8:01. It sets each task's `not_before` to a staggered time, and the dispatcher, which already pulls the oldest `queued` task each cycle, simply skips any task whose `not_before` is still in the future and picks it up once that time passes.

So the manager plans a day like this:

```
enqueue draft_listing   not_before 2026-07-14T09:00
enqueue make_pins       not_before 2026-07-14T11:00
enqueue draft_post      not_before 2026-07-14T13:00
enqueue research        not_before 2026-07-14T15:00
```

The dispatcher does the honoring: when it pulls the oldest `queued` task, it checks `not_before`, and if that time has not arrived yet it leaves the task queued and moves on. Add that one check to the dispatcher's pull step if it is not already there (it is a single condition: only grab `queued` rows where `not_before` is null or in the past). That one column plus that one check is the whole scheduler for a day's work. No new machinery.

Spreading the day this way also keeps the owner's engine happy: a Claude Pro plan has rolling usage windows (`06-cost-and-billing.md`), and firing ten agents in the same minute is exactly how an owner exhausts a window and stalls the loop. Staggering the work across the day keeps the load even.

## Guardrails (all inherited, none relaxed)

The manager makes the system run on its own schedule, which is precisely why every guardrail has to stay exactly as strict. Automation that starts itself is only safe because it still cannot finish a risky thing by itself.

- **Nothing risky auto-runs, still.** The manager only ever enqueues drafting, generating, and research tasks. It never enqueues a publish, send, or spend. Those action tasks are created in only one place in the whole system: when the owner approves something on the board (`02-architecture.md` part 4). The manager sits in front of the queue, not between the board and a live action. It has no code path to a risky action, and it is not even handed the tools to perform one (rule 3).
- **Spend caps and a daily task cap.** The manager counts every task it enqueues against the owner's daily task cap and spend cap from `PROFILE.md`, and it refuses to enqueue past them. If today's plan would exceed a cap, it enqueues what fits, notes the rest in the brief, and stops. A conductor that runs every morning is exactly the thing that could quietly run up a bill or a token window over days, so it gets a hard daily ceiling like every other loop (rule 4).
- **One run per day, hard stop.** The manager task has a single-pass stop condition and does not loop. The scheduler enqueues exactly one `run_manager` task per day. If a manager run is still somehow present or `running`, do not stack a second one (the dispatcher's orphan recovery from `02-architecture.md` handles a crashed run by resetting it; it does not run two at once).
- **It reads state, it does not invent a business.** The manager plans from the digest, the calendar, the backlog, and the owner's profile. It does not decide the owner should enter a new niche or launch a new product line on its own. New directions come from the owner (or from `modules/research.md` proposals the owner approved), not from the manager's morning whim.
- **Fresh session, like everything.** The manager is one fresh `claude -p` call that ends when the brief is written (rule 2). It is not a long-lived brain that accumulates context day over day. Its only memory across days is the same memory the whole system uses: the digest, the calendar, and `lessons.md`. That is deliberate; a manager that carried its own growing private context would drift and get expensive.
- **No secrets in the enqueued payloads.** The manager passes specs (which product, which listing, which niche), never credentials. The child agents read keys from `.env` when they run, exactly as they already do (rule 11).
- **Honest scheduling.** If the owner is on the local path, tell them plainly that a missed morning (machine asleep) means a skipped run, and that the VPS path is the fix if they need it guaranteed. Do not let them believe a sleeping laptop runs their day.

## Prove it works

Do not tell the owner the manager is done until you have run it and watched a full morning behave. Run it once by hand first, before you ever put it on a schedule.

- [ ] Run `run_manager` once manually and read its result. It enqueued a sensible day of work from the real digest, calendar, backlog, and profile.
- [ ] Every task it enqueued is an existing, proven task type, in status `queued`, with a `payload` and a staggered `not_before`. It enqueued no publish, send, or spend task.
- [ ] The dispatcher honors `not_before`: a task with a future `not_before` stays `queued` and is not pulled until its time passes. Confirm by setting one a few minutes out and watching it wait, then run.
- [ ] `data/morning-brief.md` exists, is readable, is about five lines, and honestly states what was queued, why, and what is still waiting on the owner. It ends with the "nothing was published, sent, or spent" line and that is actually true.
- [ ] The manager stopped after one pass. It did not run any task itself and did not loop.
- [ ] Caps hold: give it a day of work that exceeds the daily task cap on purpose and confirm it enqueues only what fits and reports the rest in the brief, rather than blowing past the cap.
- [ ] Only after all of the above, schedule it (cron locally, or cron on the VPS) and confirm the next morning's run produced a fresh brief and a queued day, with the owner still approving every risky item by hand on the board.
- [ ] The owner understands, in plain sentences, what changed: the system now starts their day, and they still approve everything that goes live.

When those are true, the owner has crossed the line from "a queue I feed" to "an OS that runs my day." Keep it plain and keep it gated. The manager earns its keep by removing the morning chore, not by removing the owner's control. Make it look nicer only after it has run a real week and the owner trusts what lands in the brief.

---

_The manager introduces no new selling channel and no new risk surface. It only decides what enters the queue, earlier and on a schedule, behind every gate that already existed. It composes the same modules the owner already built (`modules/etsy-autopilot.md`, `modules/content-engine.md`, `modules/research.md`, and the calendar from `modules/shop-in-a-box.md`). All third-party costs from those modules still apply, and one extra agent run per day is one extra engine call. Results are not typical; a manager that queues a perfect day can still produce zero orders. Keep the owner's spend and task caps enforced, keep the brief honest, and keep every live action one human click away._
