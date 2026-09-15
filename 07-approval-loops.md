> For the AI building this system. Deepen the approval board from simple yes/no into real review loops. Build on the board and guardrails in 02-architecture.md.

# Approval loops: making the board actually smart

The board in `02-architecture.md` is the human-in-the-loop gate: risky tasks park as `needs_approval` and wait for a click. That is the floor, not the ceiling. A bare approve/reject board wastes the owner's judgment. When they reject something, that reason is worth capturing. When they kill an idea, the system should never pitch it again. When an agent has earned trust, the owner should be able to loosen its leash a little.

Build the four patterns below on top of the existing board. None of them replace the guardrails. Spend caps, no-auto-publish, and the lessons file (`lessons.md`) still hold. These patterns make the human review faster and less repetitive, they do not remove the human.

## Pattern A: reject-with-feedback (the revision loop)

A plain reject throws away the most useful thing the owner just produced: the reason. "No" tells the agent nothing. "No, the title is too long and it does not mention it is handmade" tells it exactly what to fix.

**Build it like this.** When the owner rejects an item, prompt them for a one-line note (optional, but ask). Then, instead of just marking the task `failed` and stopping, do two things:

1. Mark the original task `rejected` and store the note on it.
2. Create a **new task** of the same type, and carry the owner's note into its `payload` as feedback, along with a pointer to the original attempt.

The new agent runs fresh (rule 2, one task one fresh agent), but its prompt now includes the owner's actual complaint and the prior draft. It fixes the real problem instead of guessing at a different one.

A rejection handler, roughly:

```javascript
function rejectWithFeedback(taskId, note) {
  const original = getTask(taskId);
  db.run(
    "UPDATE tasks SET status='rejected', result=json_set(result,'$.reject_note',?) WHERE id=?",
    note, taskId
  );

  // Only spin a revision if there is something to act on.
  if (note && note.trim()) {
    const payload = JSON.parse(original.payload);
    payload.revision_of = taskId;
    payload.owner_feedback = note.trim();
    payload.prior_attempt = original.result;
    enqueue(original.type, payload);   // fresh task, same type, carries the note
  }
}
```

And the agent's prompt for that task type checks for feedback near the top:

```
If this payload includes owner_feedback, the previous attempt was rejected for
that exact reason. Read prior_attempt, fix the specific complaint in
owner_feedback, and do not change things the owner did not object to.
```

Two more moves make this pay off:
- **Cap the revision loop.** Use the existing `attempts` counter. After a small number of rejections on the same lineage (say 3), stop auto-revising and park it for the owner to rewrite the brief themselves. A revision loop with no cap is just a slower runaway loop, and guardrail 4 (stop conditions on everything) applies here too.
- **Feed the lesson.** If the same kind of complaint shows up across different tasks ("title too long" again and again), write it to `lessons.md` for that task type. Then future first drafts avoid the mistake before the owner ever sees it. That closes the loop from `02-architecture.md`: the board catches it once, the lessons file stops it recurring.

## Pattern B: always preview the real artifact

**Never let the owner approve from a summary or from raw JSON.** A one-line description ("Cozy Fall Mug, $18, 3 tags") or a blob of `result` JSON is not the thing being published. The owner could approve text that looks fine while the actual listing has a broken image, a mangled emoji, a wrong price, or a title that overflows. Approving the description is not approving the artifact.

**Rule: the board renders the actual thing before it asks for a yes.** Whatever is about to go live is what the owner sees.

- **A listing or POD product:** show the mockup image, the real title, the full description as it will read, the tags, and the price. If there is a product image, display the image, not its filename.
- **A social post or pin:** render the actual image with its caption laid over or beside it, exactly as it will appear on the platform. Not the prompt that made it. The output.
- **An email or DM:** show the rendered message, subject and body, formatted as the recipient will get it. Not a summary of what it says.
- **A video (Higgsfield) or 3D asset (Meshy):** play or show the real file. A metered generation is doubly worth previewing, because approving it can also spend credits (see `06-cost-and-billing.md`).

The mechanics: when an agent finishes a risky task, it should write two things into `result`: the machine data (the JSON the follow-up action needs) and a **rendered preview** in `result.preview`. Per the storage model in `02-architecture.md`, large artifacts (images, video, rendered HTML) live as files under `board/pending/<task_id>/` and `result.preview` holds the path. The board displays the preview. A minimal single-file local approval page that renders these is exactly what the approval-page section at the end of `07-approval-loops.md` builds, and it exists so the owner never has to approve blind. Until that page is built, even a text board should at minimum open the real image file and print the full final text, never a truncated summary.

This is guardrail-level, not a nicety. The gotcha in `08-gotchas.md` about approving from a summary and shipping a broken listing is this pattern being skipped. Do not skip it.

## Pattern C: the kill list (dead ideas stay dead)

Agents that propose ideas (products to make, posts to write, niches to try) will re-propose things the owner already said no to, because a fresh session has no memory. Rejecting the same bad idea every week is a tax on the owner's attention. Fix it with a small, permanent kill list.

**Build it as a plain, readable file, the same spirit as `lessons.md`.** Call it `dead-ideas.md`, one line per killed idea, in the owner's words:

```
- [product] Scented candles. Owner killed it: does not want to deal with wax melt liabilities.
- [post] "Top 10" listicles. Owner killed it: off-brand, too generic.
- [niche] Wedding favors. Owner killed it: too seasonal, tried before, flopped.
```

When the owner kills an idea from the board, append a line here (grab a short reason if they give one). Then wire two checks:

1. **Before an idea-proposing agent runs,** inject the current `dead-ideas.md` into its prompt: "Do not propose anything on this kill list, and do not propose close variants of it." Fresh session, but now it knows the graveyard.
2. **When a proposal lands on the board,** the dispatcher can do a cheap check against the kill list and auto-drop obvious repeats before they ever reach the owner.

Keep kill separate from reject. A **reject** is "not this version, try again" and it feeds Pattern A's revision loop. A **kill** is "never this idea again" and it goes here. Give the owner both buttons on the board so they can say which one they mean. Killing should be rare and deliberate, so a single misfire does not permanently wall off a whole category. If the owner ever wants something back, they delete its line from the file. Plain text, easy to undo, easy to read: same philosophy as the rest of the system.

## Pattern D: the confidence ramp (loosen a trusted gate, carefully)

Some agents earn trust. If the owner has approved a given agent's output many times in a row with no edits and no rejections, making them keep clicking yes on that same safe task is friction with no payoff. Offer, do not impose, a lighter gate for that one agent.

**How to build it safely:**

- **Track a clean-streak per task type.** Count consecutive approvals with zero edits and zero rejections. A single reject or edit resets the streak to zero. Store it wherever you keep task-type config.
- **When the streak passes a threshold** (say 20 clean approvals in a row), surface it to the owner as an offer, never a default: "Your `draft_post` agent has been approved 20 times straight with no edits. Want to auto-approve its drafts from now on, or keep reviewing each one?" The owner opts in. The system never loosens a gate on its own.
- **If they opt in,** that task type can skip the manual click and go straight through, but it still writes a rendered preview to `result` and still logs every action, so the owner can spot-check the history any time and re-tighten with one word.

**Hard limits on the ramp, no exceptions:**
- **Never ramp a spend action.** Anything that costs money (POD orders, video or 3D credit spend, ads) stays gated forever, no matter the streak. Guardrail 4 and `06-cost-and-billing.md` do not bend for a good track record.
- **Never ramp an irreversible or external-facing send** without the owner explicitly choosing it, and even then be conservative. Publishing a listing, sending an email or DM, placing an order: these are the "hard to undo" category from `02-architecture.md`. A high streak can justify auto-approving a draft that stays internal (a written post saved for the owner to schedule); it does not justify auto-firing something at a customer or a marketplace.
- **Any reject collapses the ramp.** One bad output and that task type snaps back to manual review, streak reset. Trust is easy to lose here, on purpose.

The confidence ramp is the only pattern that removes a click, so it is the one to build last and most carefully. Function and safety first (that is the whole `CLAUDE.md` order). Only loosen a gate once the owner has watched an agent be right many times and asks for it themselves.

## How these tie back to the board

All four patterns are additions to the single `needs_approval` gate, not replacements for it:

- The board still parks every risky task and waits for a human. That never changes.
- **Reject-with-feedback** turns a dead-end "no" into a fix.
- **Always-preview** makes sure the "yes" is an informed one.
- **The kill list** stops the board from asking the same dead question twice.
- **The confidence ramp** lets the owner retire the clicks that stopped teaching them anything, within strict limits.

Build A and B first: they cost the owner nothing and prevent the most common failures. Add C when idea-agents start repeating themselves. Offer D only after real trust exists, and never for spend or irreversible sends. The guardrails in `02-architecture.md` sit over all of it, unchanged.

## The approval page (the optional screen, built last)

This is the first piece of interface in the whole build, and it comes late on purpose. Do not build it until the system already functions: the queue, the dispatcher, at least one real agent, the board logic, the guardrails, and a first money-maker that has produced a real result the owner has approved from the command line. That is Step 7 in `03-build-order.md`, the "only now, polish" step. Everything before this had to work as plain text first, and it does.

Frame it to the owner exactly that way. This is the small taste of a real screen that they earned by getting the system working first, not the shiny thing they start with. If the owner asked for a dashboard earlier and you nudged them off it (the vanity-trap rule in `CLAUDE.md`), this is you keeping the other half of that promise: "get it working first, then we make it nice." Now it works, so now this.

Keep it deliberately minimal. One HTML file. Runs from their own machine by opening the file. No framework, no build step, no server to deploy, no hosting, no account. If you catch yourself reaching for React or a bundler or a login page, stop. That is scope the owner did not ask for and does not need. The value here is that the owner can finally SEE the pending items and their real previews and click Approve or Reject, instead of reading a text list. That is the whole win. Do not inflate it.

## What it is

A single local page that does exactly what `os board` did in `02-architecture.md`, but visually:

- Reads every task sitting at `needs_approval`.
- Shows each one as a card with its **rendered preview**, not raw JSON. This is Pattern B from `07-approval-loops.md`: the owner approves the actual artifact (the mockup image, the formatted post, the real message text), never a summary. The page exists so the owner never approves blind.
- Gives each card an **Approve** and a **Reject** button that update that task's status, exactly as the command-line board did.

It is the same board, same statuses, same underlying logic. Only the surface changed. Nothing about the guardrails, the spend caps, or the no-auto-publish rule moves. A prettier button is still a human click.

## How to build it (choose the simpler path for their setup)

The page needs to read pending tasks and write back a status change. There are two honest ways to do that locally. Pick based on what the owner already has running, and tell them which and why.

**Path 1: reuse the local process they already have.** If their dispatcher or a small local server is already running (most setups have something), add two tiny read/write endpoints to it: one that returns the `needs_approval` tasks as JSON, one that accepts an id plus a decision and updates the row. The HTML page fetches from the first and posts to the second. This is the cleaner path when a local process exists, because the page talks to the same SQLite database through code you already trust.

**Path 2: no server at all, act through the queue.** If the owner has no running local server and you do not want to add one, the page can read a small JSON file the dispatcher writes out (a dump of pending items) and, on a click, write the owner's decision into a second file (or a watched folder) that the dispatcher picks up on its next cycle. The page stays pure static HTML opened from disk. Slightly more moving parts on the dispatcher side, zero server to run. Prefer this only when adding an endpoint is genuinely not wanted.

Default to Path 1 if any local process is already up. It is less to explain and less to break. Whichever you pick, the buttons must end at the same place the command-line board did: an approve routes the task to its follow-up action through the queue, a reject marks it `rejected` and (per Pattern A in `07-approval-loops.md`) can carry a feedback note into a fresh revision task. Do not invent a new approval mechanism. Wire the page to the one that already works.

## The rules this page must not break

- **Preview the real thing.** Every card renders the actual artifact. If it is an image (a listing mockup, a pin, a POD product), show the image, not its filename. If it is text (a post, an email, a caption), show it formatted the way it will ship. If it is a metered generation (video, 3D), link to or embed the real file, and remember approving it may spend credits (`06-cost-and-billing.md`). Approving a summary is the exact gotcha in `08-gotchas.md` about shipping a broken listing. The screen exists to prevent that, so it must never show less than the command-line board did.
- **A click is still a human click.** Nothing on this page auto-approves. It does not batch-approve. It does not remember a choice and apply it to future tasks. That is the confidence ramp (Pattern D in `07-approval-loops.md`), a separate, later, opt-in decision, not something a UI button quietly does.
- **No new powers.** The page can only do what the board could already do: approve, reject, and reject-with-a-note. It does not spend, publish, or send directly. It flips a status and lets the existing dispatcher and guardrails do the rest.
- **Local only.** Opened from the owner's disk or served from their own local process on their own machine. Not hosted, not public, no auth to build because it never leaves their computer. If they later want it reachable from their phone, that is a `05-infrastructure.md` conversation, not part of this file.

## A minimal shape to build from

One file, `approval.html`. Inline the CSS and the small script. Fetch pending items on load, render a card per item with its preview, and post the decision on click. Keep it around this size, not larger:

```html
<!doctype html>
<meta charset="utf-8">
<title>Approvals</title>
<style>
  body { font: 16px system-ui, sans-serif; max-width: 720px; margin: 2rem auto; }
  .card { border: 1px solid #ccc; border-radius: 8px; padding: 1rem; margin: 1rem 0; }
  .card img { max-width: 100%; border-radius: 4px; }
  .meta { color: #555; font-size: 0.9rem; }
  button { font-size: 1rem; padding: 0.5rem 1rem; margin-right: 0.5rem; cursor: pointer; }
  .approve { background: #1a7f37; color: #fff; border: 0; }
  .reject { background: #b42318; color: #fff; border: 0; }
  #empty { color: #555; }
</style>

<h1>Waiting for you</h1>
<div id="board"><p id="empty">Loading...</p></div>

<script>
async function load() {
  // Path 1: read from the local endpoint the dispatcher exposes.
  const tasks = await fetch("/api/pending").then(r => r.json());
  const board = document.getElementById("board");
  board.innerHTML = "";
  if (!tasks.length) { board.innerHTML = "<p id='empty'>Nothing to approve. Clear.</p>"; return; }

  for (const t of tasks) {
    const p = t.preview || {};            // the rendered artifact, written by the agent
    const card = document.createElement("div");
    card.className = "card";
    card.innerHTML = `
      <div class="meta">#${t.id} &middot; ${t.type}</div>
      ${safeSrc(p.image) ? `<img src="${safeSrc(p.image)}" alt="preview">` : ""}
      ${p.title ? `<h2>${escapeHtml(p.title)}</h2>` : ""}
      ${p.body  ? `<p>${escapeHtml(p.body)}</p>` : ""}
      ${p.price ? `<div class="meta">Price: ${escapeHtml(p.price)}</div>` : ""}
      <button class="approve">Approve</button>
      <button class="reject">Reject</button>
    `;
    card.querySelector(".approve").onclick = () => decide(t.id, "approve");
    card.querySelector(".reject").onclick  = () => {
      const note = prompt("Why? (a reason spins a fixed revision; blank just rejects)") || "";
      decide(t.id, "reject", note);
    };
    board.appendChild(card);
  }
}

async function decide(id, action, note = "") {
  await fetch("/api/decide", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ id, action, note })
  });
  load();   // refresh; the item is gone from pending once the dispatcher acts
}

// Never inject agent text as raw HTML. Escape everything the agent wrote.
function escapeHtml(s) {
  return String(s).replace(/[&<>"']/g, c =>
    ({ "&":"&amp;", "<":"&lt;", ">":"&gt;", '"':"&quot;", "'":"&#39;" }[c]));
}

// The preview path is agent-written too, so it is NOT trusted. Only allow a
// relative path under board/ (where previews live per the storage model), and
// escape it before it goes into the src attribute. Anything else returns empty,
// so the card shows no image rather than injecting an attacker-controlled URL.
function safeSrc(path) {
  if (typeof path !== "string") return "";
  if (path.startsWith("board/") && !path.includes("..")) return escapeHtml(path);
  return "";
}

load();
</script>
```

Two things to hold onto in that sketch:

- The card renders `t.preview`, the rendered artifact the agent wrote alongside its machine data (exactly what Pattern B in `07-approval-loops.md` tells the agent to produce). If an agent is only writing raw result JSON and no preview, fix the agent to emit a preview before you build this page, or the page has nothing real to show.
- Escape every piece of agent-written text before it hits the DOM. The agent's output is not trusted markup. The `escapeHtml` helper is not optional decoration, it is the one security detail this page has to get right.
- **Escape the preview image path too, not just the text.** The `p.image` path is written by the agent, so it is exactly as untrusted as the title and body. Dropping it raw into `src="..."` lets a malicious or malformed path break out of the attribute (for example a value containing a quote and an `onerror` handler) and run script on the page. The `safeSrc` helper closes that hole two ways: it only accepts a relative path under `board/` (where real previews live per the storage model in `02-architecture.md`), and it escapes the value before it lands in the attribute. Anything outside `board/` renders no image instead of injecting a URL you did not vet. Escaping the visible text but leaving the image path raw was the actual gap, and this is the one-line fix for it.

The `/api/pending` and `/api/decide` endpoints are the two small handlers from Path 1, reading and writing the same tasks table the command-line board used. For Path 2, swap the two `fetch` calls for reading the dispatcher's pending-items file and writing a decision file it watches. Same page, same buttons, same result.

## Prove it, then hand it over

Same standard as every other step (rule 5 in `CLAUDE.md`). Do not tell the owner it is done until you have opened the page, seen a real pending item render with its actual preview, clicked Approve, and confirmed in the database (or the dispatcher log) that the task moved and its follow-up action fired. Then reject one and confirm it went to `rejected` and, if a note was given, that a revision task appeared. Show the owner the page working against a real item on their board, not a mockup.

Then say it plainly: "This is the same approval board you have been using, now as a screen. Same rules, same safety, it just looks like something now. You earned this by getting the system working first."
