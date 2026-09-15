> For the AI building the system, and the owner running it. The coding-agent idea: your AI tool can maintain and extend the OS it built. This file is how to use it safely on your own system, with a hard line it does not cross.

Level: intermediate | Time: ongoing (use as needed) | Cost: $0 beyond your engine

# Module: self-maintenance (the coding agent)

Here is the part people miss: the same AI tool that built this system can also keep it running and grow it. You do not need a developer on call to add a new task type, fix a broken integration, or upgrade a script. You ask your AI, it makes the change, it proves the change works, and you approve it. That is the coding agent, and you already have it.

Say it plainly so nobody waits for a feature that already shipped: **your AI coding tool IS the coding agent.** Claude Code, or whatever tool the owner used to build the OS, is the thing that maintains and extends it. There is no separate robot to install. This file is the safe way to use that tool on the owner's own system, because a tool that can rewrite your OS can also break it, and one part of the OS it must never touch on its own.

I run my own Etsy shop and my agency on a system built exactly this way, and I have never opened the code myself to add a feature. I describe what I want, the AI writes it, it shows me it working, I approve. The discipline below is the entire reason that is safe instead of reckless.

## The one line the coding agent never crosses

Read this before anything else in this file.

**HARD RULE: the AI never edits guardrail, spend-cap, or approval-gate code without first showing the owner the exact diff and getting explicit approval.**

This is the whole safety model. The guardrails are the only thing standing between "an agent proposed something" and "an agent spent your money or published under your name." If the AI can quietly weaken a spend cap, remove a board gate, or loosen the no-auto-publish rule while "fixing" something else, then none of the other protections in this Blueprint mean anything.

So this code is off-limits to silent edits:

- The **spend caps and stop conditions** (max tasks, max spend, max runtime) from rule 4.
- The **approval board and its gates**: anything that decides what goes to `needs_approval`, and the line that a risky action only runs after the owner approves (rule 3).
- The **tool whitelists** that keep each agent from holding a publish, spend, or send tool it should not.
- The **secrets handling**: how `.env` is read and kept out of logs, chat, and commits (rule 11).
- The **approve/reject logic** that enqueues the follow-up ACTION task.

When a change would touch any of that, the AI stops and does three things, in order:

1. **Shows the owner the exact diff.** The precise before-and-after lines, not a summary like "I tightened the caps." A summary is not a diff. The owner sees the actual code that changes.
2. **Explains what the change does to the guardrail in one or two plain sentences.** "This raises the daily spend cap from $20 to $50." "This adds a new tool to the posting agent's whitelist."
3. **Waits for an explicit yes.** Not implied, not "I assume you want this." An explicit approval from the owner, the same way a risky action waits for a board click. No yes, no change.

If the owner is not sure, the change does not happen. A guardrail edit is exactly the kind of hard-to-undo, high-stakes change that rule 5 says to stop and double-check. When in doubt, leave the guardrail as it is and ask.

Everything below assumes this rule is always on.

## The three things you will ask the coding agent to do

### 1. Add a new task type

The owner wants a capability the system does not have yet: a new kind of draft, a new scout, a new report. This is the most common ask and the safest, because a new task type is additive.

How to do it right:

- Build it from the existing patterns. A new agent starts from the generic template in `prompts/prompt-library.md` (section 2): a tight role, one task, a tool whitelist, an output format, a stop condition. Do not invent a new shape.
- It writes its result to the board like everything else. Findings and drafts go to `result`; large artifacts go to files under `board/pending/<task_id>/` with the path in `result.preview`. Same storage model as the rest of the system, no exceptions.
- If it proposes anything risky (publish, spend, send), it routes through the board. A new task type does not get to skip the gate because it is new. Build the gate before the action (rule 3).
- Give it only the tools it needs. A drafting task gets `Read` and `Write`. It does not get a posting tool "just in case."

A new task type that only reads and drafts (a new scout, a new report) is low-risk and does not touch the hard-rule code. Build it, prove it, and move on. If the new task type needs to touch a spend cap or a board gate to work, that part triggers the HARD RULE above.

### 2. Fix a broken integration

An API stopped working. A listing call 500s, a post fails to ship, a digest comes back empty. The coding agent diagnoses and fixes it, but the first move is almost never "rewrite the code."

The honest first checks, because these are the real causes (see `08-gotchas.md`, which catalogs the ones that already bit someone):

- **Did a key expire or rotate?** An expired API key fails silently, just error codes, no alarm. This broke a real checkout for days in `08-gotchas.md`. Check the key before you touch the code. If it is dead, the fix is a new key in `.env`, not a code change.
- **Did the provider change its API?** Marketplaces and tools change their endpoints and rules. Verify against the provider's current docs before assuming the code is wrong. The code that worked last month can be correct and still fail because the API moved.
- **Is it a rate limit, not a bug?** A dispatcher spawning agents all day can exhaust a plan's usage window (see `06-cost-and-billing.md`). The fix there is pause-and-resume, not a rewrite.

When it genuinely is the code, the coding agent fixes the narrowest thing that is broken and proves it (below). It does not "refactor while it is in there." A fix that also rewrites three unrelated files is how a working system breaks in a new place.

**Secrets stay secret during a fix.** When the fix involves a key, the AI has the owner put the new key in `.env` and reads it from there. It never prints the key, never pastes it into chat, never commits it (rule 11). If a key leaked during troubleshooting (pasted into chat, dumped in a log), the AI stops and tells the owner to reroll it now at the provider, then put the fresh one in `.env`. A leaked key is live until it is rerolled.

### 3. Upgrade a script

An existing script needs to do more, run faster, or handle a case it currently chokes on. Same discipline: make the smallest change that gets the win, keep the pattern the owner can still read (rule 7), and prove it before calling it done.

Watch the line: an upgrade to a drafting or scout script is routine. An upgrade to the dispatcher, the approve/reject logic, or anything holding a cap or a gate triggers the HARD RULE. "Upgrade the dispatcher to run tasks faster" sounds harmless and sits right on top of the guardrail code. Show the diff.

## Prove it works, every time (this is not optional for self-edits)

The prove-it rule (rule 5) applies harder to the coding agent than to anything else, because here the AI is changing the machine that runs everything.

Before the AI says a maintenance change is done:

- **Run it.** Not "this should work." Actually execute the new task type, the fixed integration, the upgraded script, and show the real output.
- **Show the output.** Paste what actually happened: the draft that got produced, the successful API response, the task that moved through the board correctly.
- **Confirm nothing else broke.** After a change to any shared piece (the dispatcher, the schema, a whitelist), run one normal task end to end and confirm it still flows queued -> running -> needs_approval -> approved the way it did before. A maintenance change that fixes one thing and silently breaks the queue is a net loss.
- **On a schema change, treat it as major.** A change to the tasks table is hard to undo and can strand in-flight tasks. Back up the SQLite file first (a redeploy or a bad migration can wipe it, see `08-gotchas.md`), make the change, and prove old and new tasks both still run.

If the AI cannot show it working, it says so plainly and does not call it done. "I made the change but could not run it" is an honest, useful sentence. "It should work now" is not.

## A safe maintenance session, start to finish

This is the flow the owner and the AI follow for any self-maintenance ask:

1. **Owner describes the want** in plain language. "Add a scout that finds trending gift bundles." "The Pinterest post keeps failing." "Make the digest also count refunds."
2. **AI restates it and flags the blast radius.** "That is a new read-only task type, low risk, I will build it." Or: "That fix touches the posting agent's whitelist, which is guardrail code, so I will show you the diff before I change anything."
3. **AI makes the smallest change** that does the job, from the existing patterns.
4. **If it touched hard-rule code, STOP and show the diff, wait for an explicit yes** (the HARD RULE). Otherwise continue.
5. **AI runs it and shows real output** (prove-it, rule 5).
6. **AI confirms the rest of the system still flows** end to end.
7. **AI tells the owner in one or two plain sentences** what changed and why (teach as you go, rule 8), so the owner still understands their own system.

Follow that and the owner can grow the system for years without ever opening the code, and without the AI ever quietly weakening the one layer that keeps it safe.

## Guardrails for this module

- **The HARD RULE governs everything here:** no silent edits to guardrail, spend-cap, approval-gate, whitelist, or secrets code. Diff first, explicit approval, then change.
- **Smallest change that works.** No refactoring unrelated code while fixing one thing. The owner has to be able to read what changed.
- **Prove it or say you did not.** Run it, show the output, confirm the rest still works. Never claim a self-edit works on faith (rule 5).
- **Secrets never leak during maintenance.** Keys live in `.env`, read from there, never printed or committed. Reroll on any leak (rule 11).
- **Keep it readable.** Every maintenance change keeps the plain, hand-rolled style (rule 7). A system the owner cannot read is one they cannot fix or trust.

## Prove it works (for this module itself)

The first time you use the coding agent, do it on something small and safe and let the owner watch the whole loop: add one read-only scout task type, run it, show the finding on the board, and confirm a normal task still flows. Then, on purpose, walk the owner through the HARD RULE once with a real example: propose a tiny guardrail change, stop, show the diff, and let them approve or decline it. Once the owner has seen both the additive path and the diff-and-approve path work, they know how to grow the system safely. Do not call this module understood until the owner has seen the guardrail stop happen live (rule 5).
