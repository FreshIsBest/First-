> For the AI and the owner both. The battle scars: real failure modes that already bit someone, written so you skip the mistakes instead of repeating them.

# Gotchas: the mistakes already made so you do not have to

Every entry below is a real failure that cost someone time, money, or data on a system exactly like the one you are building. They are grouped by area. Each is written as **symptom -> cause -> fix** so you can spot it fast. AI: when you build the matching piece, apply the fix up front. Owner: when something feels wrong, scan this file first, the answer is probably here.

None of this is theoretical. The whole reason this file exists is that the author hit these live, so you can skip straight past them.

## Data and backups

**The database vanished after a reset.**
- Symptom: your queue, your task history, your `lessons.md` lineage, all gone after a redeploy, a container restart, or a host reset. The system comes back up empty like day one.
- Cause: the SQLite file was sitting on temporary or rebuild-on-deploy storage. Anything on that kind of storage is wiped when the environment resets. The app looked fine right up until the reset erased it.
- Fix: keep the SQLite file on persistent storage, a real disk path that survives restarts, and back it up on a schedule. On the VPS path this is a specific mount, see `05-infrastructure.md`. Never assume "it was there yesterday" means it is safe. Test your restore once, a backup you have never restored is a guess.

**A bulk edit garbled every emoji and accent.**
- Symptom: after a script edited many files or many rows at once, emojis turned into mojibake, accented letters broke, curly quotes became junk.
- Cause: files or text were read and rewritten without forcing UTF-8. The default encoding on the machine mangled every non-ASCII character on the way through.
- Fix: read and write as UTF-8 explicitly, every time, and spot-check the output after any bulk edit. One glance at a product title with an emoji in it tells you if the pass was clean.

## Keys and security

**A checkout (or send, or upload) silently 500'd for days.**
- Symptom: something that used to work just returns errors. No alarm, no email, no crash the owner sees. It only surfaces when a customer complains or the owner happens to look.
- Cause: an API key expired or was rotated at the provider. The code kept calling with a dead key and kept getting rejected. Silence is not health, it was failing the whole time.
- Fix: keys expire and get rotated, plan for it. Add a dead-simple health check that pings each critical integration on a schedule and tells the owner when one goes dark. Do not assume no news is good news.

**A leaked key kept working long after it leaked.**
- Symptom: a key got pasted into chat, printed to a log, or committed to a file, and it still authenticates fine.
- Cause: a key does not stop working just because it leaked. It is live and abusable until someone kills it at the provider.
- Fix: reroll it immediately. Regenerate at the provider so the exposed one dies, then put the new value in `.env`. This is `CLAUDE.md` rule 11: secrets live in a local `.env` that is never printed, never shared, never committed, and a leaked key is a live risk until it is rerolled. Do not wait, do not hope nobody saw it.

## Scheduling and runtime

**The overnight jobs just stopped running.**
- Symptom: scheduled work (morning digest, nightly pin batch, queue drain) quietly does not happen. No error, it simply did not fire.
- Cause: the job was scheduled on a local machine that went to sleep, hibernated, or shut off. Local scheduling only runs while the computer is awake. A closed laptop runs nothing.
- Fix: if you need true 24/7, you need an always-on machine, that is the VPS path in `05-infrastructure.md`. If you are running local by choice (a fine default), know the tradeoff and keep the machine awake for the windows you care about. Do not schedule a 3 AM job on a laptop you close at night.

**A loop with no cap burned through spend fast.**
- Symptom: the bill or the API usage spikes hard, or the same agent keeps churning long after it should have stopped.
- Cause: a loop with no stop condition. It kept pulling tasks, kept spawning agents, kept spending, because nothing told it when to quit.
- Fix: every loop gets a hard stop, max tasks, max spend, max runtime, whichever hits first. This is `CLAUDE.md` rule 4 and it is not optional. An agent that can run forever will eventually run forever on the wrong thing. See also cost detail in `06-cost-and-billing.md`.

**Hit a provider rate limit mid-run and tasks started failing.**
- Symptom: agents that worked one at a time start erroring in bursts when several run close together. Retries make it worse.
- Cause: too many calls to the AI or a tool API in too short a window tripped the provider rate limit. Hammering it with instant retries just deepens the hole.
- Fix: run tasks with a small gap, not all at once, and back off on a limit error (wait longer between each retry, do not retry instantly). Respect the ceiling instead of fighting it.

## Agent behavior

**One long-running agent got confused and sloppy.**
- Symptom: an agent that handles many tasks in the same session starts mixing up context, applying one task's details to another, or drifting off the prompt.
- Cause: session pollution. Reusing one long-lived agent across many jobs piles unrelated context on top of each other until it cannot tell them apart.
- Fix: one task, one fresh agent, `CLAUDE.md` rule 2. Each job is a clean session with a tight prompt and only the tools it needs. This is the core pattern for a reason: fresh sessions stay cheap, fast, and predictable.

**Tasks stuck on "running" forever after a crash.**
- Symptom: the queue shows tasks marked in-progress that never finish and never fail. They just sit there, and nothing picks them up again.
- Cause: the dispatcher or the machine died mid-task, so the task was never marked done or failed. It is orphaned, flagged running with nothing actually running it.
- Fix: on startup, reset any stale in-progress task back to queued so it gets retried, and cap retries so a genuinely broken task fails instead of looping. A task that has been "running" longer than your timeout is not running, it is dead. Recover it.

## Approvals and output quality

**Approved a listing from the summary, shipped it broken.**
- Symptom: the owner clicked approve on a clean-looking one-line description, and the thing that actually went live had a broken image, a mangled title, a wrong price, or a garbled emoji.
- Cause: approving from a text summary or raw JSON is not approving the artifact. The summary read fine while the real rendered listing was broken.
- Fix: always render and preview the real artifact before approving, the actual mockup, the actual title and description as they will read, the real price and tags. This is Pattern B in `07-approval-loops.md`, and the approval-page section at the end of `07-approval-loops.md` builds a page that shows the real thing. Never approve blind.

**Kept rejecting the same dead idea every week.**
- Symptom: an idea-generating agent keeps proposing something the owner already killed. The owner re-rejects it constantly.
- Cause: a fresh session has no memory of past rejections, so it re-pitches the graveyard.
- Fix: keep a plain `dead-ideas.md` kill list and inject it into idea agents so they do not re-propose killed ideas or close variants. That is Pattern C in `07-approval-loops.md`. Reject means "try again," kill means "never again," keep them separate.

## AI generation quirks

**The AI-generated image had garbled text on it.**
- Symptom: an image generator was asked to put words on the picture, and the words came out as nonsense letters, misspellings, or melted glyphs.
- Cause: image models are bad at rendering multi-word text. They approximate the shape of letters, they do not typeset.
- Fix: generate the image with no text baked in, then overlay the real text yourself as a separate layer. You get a clean visual and correct, readable words every time. This applies to POD designs, pins, thumbnails, anything where the text has to be right.

**The subscription made it look like nothing ran.**
- Symptom: the owner runs agents all day but sees no per-call charges anywhere, and worries the system is not actually doing anything.
- Cause: a flat AI subscription (like Claude Code Pro) does not itemize per-call usage the way a metered API bill does. Quiet billing is not proof of quiet agents.
- Fix: judge whether the system ran by its own logs and outputs, the task history, the finished artifacts on the board, not by the absence of a charge. `06-cost-and-billing.md` covers this billing surprise in full. Do not confuse "no line item" with "nothing happened."

## Storage and the database, deeper

**The disk filled up and the whole system froze.**
- Symptom: agents start failing in weird ways, writes error out, the dispatcher hangs, and nothing obvious is broken in the code. Then you notice the disk is at 100 percent.
- Cause: the system generated artifacts (draft images, video clips, mockups, render files, old task payloads) and never cleaned them up. Every preview parked under `board/pending/` stayed forever. On a small VPS that fills a disk fast.
- Fix: clean up as you go. When a task is approved or rejected and its artifacts are no longer needed, delete them, or run a scheduled sweep that clears `board/pending/` older than a set age. Keep the DB and the lessons, throw away the spent drafts. Watch disk usage the same way you watch spend.

**The database locked up when two things wrote at once.**
- Symptom: intermittent "database is locked" errors, usually when the dispatcher and a dashboard or a second script touch the SQLite file at the same time. Retrying sometimes works, which makes it feel random.
- Cause: default SQLite settings serialize writers hard, so any overlap throws instead of waiting. The more surfaces read and write the queue, the more often you hit it.
- Fix: turn on WAL mode (write-ahead logging) so readers do not block writers, and keep writes short and single-instance (one dispatcher, the lockfile from the orphaned-task fix). Do heavy reads for the dashboard against the WAL-enabled file and it stops fighting the writer. This is a one-line setting you set once at setup, not something to bolt on after it bites.

## Scheduling and runtime, deeper

**A task fired hours off from when it should have.**
- Symptom: the morning digest lands at 3 in the afternoon, or a "post at 9am" task goes out at the wrong hour. The schedule looks right in the code.
- Cause: a timezone mismatch. The machine's clock is on UTC (common on a VPS) while the owner thinks in their local time, or the `not_before` timestamps were written in one zone and compared in another.
- Fix: pick one timezone and be explicit about it everywhere: store timestamps in UTC, convert to the owner's local time only when you show them. When you set a `not_before` for "tomorrow 9am the owner's time," compute it against their zone, not the server's. State the zone in the code so future-you is not guessing.

**The scheduled job ran on time and did nothing.**
- Symptom: cron or Task Scheduler fires the job, there is no error, but nothing happens. Run the same command by hand and it works fine.
- Cause: the scheduled job started in the wrong working directory or without the environment the command needs. It could not find the `.env`, the database path, or the script, so it failed silently or ran against nothing.
- Fix: in the scheduled command, use absolute paths and set the working directory explicitly, and make sure the `.env` is loaded the same way it is when you run it by hand. Test the job through the scheduler once, not just from your terminal. A command that works in your shell but not from cron is almost always a path or environment difference.

## Duplicates and idempotency

**The same post went out twice.**
- Symptom: a listing publishes twice, a customer gets two identical emails, the same pin posts to a board two times. Embarrassing and, for email, a compliance risk.
- Cause: the same task got enqueued twice (a retry that did not check, a scheduler that fired twice, a double-click) and nothing stopped the duplicate from running.
- Fix: make risky tasks idempotent. Give each one a natural key (for example a hash of the listing id plus the action) and refuse to enqueue or run a second task with the same key while the first is pending or done. A duplicate that cannot enqueue cannot double-post. Cheap to add, painful to skip.

## Keys and security, deeper

**A key ended up in a log the owner pasted back to me.**
- Symptom: while debugging, the owner copies an error or a log and pastes it into the chat, and a secret is sitting right there in the text.
- Cause: something printed the key. An agent echoed its environment, a verbose error dumped the request headers, a debug line logged the full config. Once it is in a log, it travels wherever that log goes.
- Fix: never print secrets, not even in debug output. Redact keys in any log line (show the last four characters at most). And when the owner pastes a log with a live key in it, tell them plainly and reroll it, same as any leak (`CLAUDE.md` rule 11). A key you saw is a key that leaked.

## Platform and machine

**The agent refused to run, or ran as root and behaved oddly, on the VPS.**
- Symptom: things that work on the owner's laptop act differently on the Linux VPS. A tool refuses to run, warns about running as root, or an AI CLI balks at the environment.
- Cause: on a fresh VPS you are often logged in as root, and some tools refuse to run as root or change behavior in that context. The environment is not the same as a normal desktop user account.
- Fix: create and use a normal (non-root) user for running the system on the VPS, not root. It is safer anyway (a compromised agent should not be root), and it avoids the class of "won't run as root" refusals. `05-infrastructure.md` has the day-zero setup; do the user step, do not just live as root because it was the first login.

## AI generation quirks, deeper

**An overnight update broke every agent at once.**
- Symptom: the whole system worked yesterday and is broken this morning with no change from you. Errors point at a tool, an API, or the AI CLI itself.
- Cause: a third-party thing updated underneath you. A model was deprecated, an API changed a field, a CLI shipped a breaking version. Your code did not change, the ground under it did.
- Fix: this is why every third-party integration in this Blueprint carries a "verify current docs before wiring" note, and why the escape hatch is "paste the error back to your AI." When something breaks with no change from you, suspect an upstream update first: check the tool's changelog or status, read the new error, and adapt the call. Where a tool lets you pin a version, pinning buys you the choice of when to upgrade instead of being surprised at 3am.

## Dashboard and heavy assets

**The 3D dashboard took forever to load and dragged the machine.**
- Symptom: the signature 3D dashboard from `modules/signature-dashboard.md` is slow to open, spikes the machine, or re-fetches a huge file constantly.
- Cause: the world files in `assets/worlds/` are large (tens of megabytes each). Loading one on every poll, or serving it inefficiently, means downloading a big model over and over and re-parsing it each time.
- Fix: load the world once and cache it, then poll only the small board numbers (pending count, status, earnings) on a timer, not the geometry. Serve the `.glb` from local disk, let the browser cache it, and only the lightweight data overlay updates. The world is a backdrop you load once, not something to refetch every few seconds. A dashboard that hammers the machine defeats the point of a dashboard.

## The one habit under all of these

Most of these come from one assumption: silence means everything is fine. It does not. A dead key is silent. A wiped database is silent until you look. A slept-through job is silent. Build the checks, read the logs, preview the real thing, and back up what you cannot afford to lose. The system will not tell you it is broken unless you make it.

## Platform rules that bite (the compliance section)

Read this before your system touches a real platform. It is short, and it will save you a suspended account.

**Plain truth up front: you own what your system does.** Every platform you operate on (Etsy, Printify, Pinterest, Shopify, Google, Meta, your email provider) has its own terms of service, and you agreed to them when you signed up. An AI agent acting on your behalf does not change that. If an agent posts something that breaks a rule, the platform holds you responsible, not your software. So the goal here is simple: know the tripwires, and keep a human (you) in front of anything that publishes, sends, or spends.

**This is not legal advice.** It is a practical heads-up from someone who runs shops on this exact setup. It is not complete, it is not written by a lawyer, and rules change. When something below matters to your business, go read the platform's current terms yourself, or ask a professional.

## The tripwires agents hit most

### Etsy: AI content and made-by-a-seller disclosure

Etsy is a handmade-and-vintage-first marketplace, and it has specific expectations about who made an item and how it was made. If AI is involved in creating your designs or listings, Etsy's rules on disclosure and on what counts as allowed can apply to you. Do not assume a fully AI-generated product is fine to list without checking. Read Etsy's current seller policies on AI-assisted and AI-generated content, and on production-partner and made-by-you disclosures, before you list at scale. Getting this wrong is a top cause of shop suspensions.

### Marketplace terms on automation and bots

Most marketplaces and social platforms limit or forbid automated actions: bulk auto-posting, scripted account activity, scraping, mass messaging. Some allow automation only through their official API, not through faking a human clicking around. Your system should always act through a platform's official API with your own credentials, never by imitating a person in a browser where that is against the rules. Check each platform's terms on automation and API use before you wire an agent to it.

### Ad-platform policies

If you run ads (Google, Meta, or anywhere else), the ad network reviews and can reject your creative, your targeting, and your claims. AI-written ad copy and AI-generated images are still your responsibility to keep truthful and policy-compliant. Prohibited claims, misleading before-and-after imagery, and restricted categories get accounts flagged fast. Read the ad platform's current advertising policies before an agent proposes a single ad.

### Email rules (CAN-SPAM and friends)

If your system sends marketing email, US law (CAN-SPAM) has a short list of hard requirements, and other countries have their own (stricter) versions. The basics: use a real, honest "from" name and address, do not write a deceptive subject line, include a real physical mailing address, and give every recipient a working unsubscribe link that you honor promptly. An agent drafting newsletters does not get to skip these. If you sell or send outside the US, look up the rules where your recipients live too.

## How the board protects you

This is why the whole system is built the way it is. Nothing risky runs on its own. Every publish, send, and spend stops on the approval board and waits for you to click. That pause is your compliance checkpoint. Before you approve, you are the human who confirms the listing follows Etsy's rules, the ad copy is honest, the email has an unsubscribe link. Use it. Do not rubber-stamp the board, and never wire an agent to auto-publish or auto-send around it. The guardrail is the point.

## For the AI building this

- Never build an auto-publish, auto-send, or auto-spend path that skips the board. Every risky action is an approval task first (see `07-approval-loops.md`).
- When you wire any platform integration, use the owner's own API credentials through the official API, and add the same "verify against the platform's current terms and API docs before going live" check you use for third-party tools.
- When you draft marketing email, include the required elements by default: honest sender, honest subject, a physical address placeholder for the owner to fill, and a working unsubscribe. Flag it in the board preview.
- Do not present compliance as settled. Remind the owner in plain language that they are responsible for each platform's current rules, and point them here.
