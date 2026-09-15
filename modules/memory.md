> For the AI building the owner's Agentic OS. Build the memory layer: a plain-markdown knowledge vault the whole system reads before it works and writes to after, so agents stop starting from zero every session. Start as flat files. Add a search index only when the vault gets big. Never let memory go stale and lie.

Level: beginner-safe | Time: ~1 evening | Cost: $0 (plain files; the optional reader and index are free)

# Module: memory (the vault)

Every agent in this system wakes up with amnesia. The dispatcher fires a fresh session per task (that is on purpose, it stops state from bleeding between jobs), which means each agent knows only what you put in front of it. Without a memory layer, the owner answers the same questions forever, agents repeat mistakes the owner already corrected, and hard-won facts ("this supplier is slow," "this tag got the listing flagged," "the owner hates the color teal") evaporate the moment the session ends.

The memory layer fixes that. It is a folder of plain text files the system reads before it works and writes to after. That is the whole idea. No database, no service, no new skill for the owner. If you can read a note, you can read this.

This is the same thing I run my own businesses on. Every real decision, every lesson, every owner preference lands in a vault of markdown files, and my agents read the relevant ones before they act. It is the single biggest reason the system gets smarter over weeks instead of resetting every morning. The interview already seeded an empty version of this, and `lessons.md` is the mistakes half of it. This module is the full layer: the vault the whole system reads and writes.

## Two moments: the empty vault, then the full layer

The vault is set up in two passes, on purpose.

- **Early (at the interview).** The moment `PROFILE.md` is written and confirmed, the owner has their first memory note, because that profile *is* the first thing the system knows about them. So the interview close (`01-interview.md`) creates an empty `memory/` folder, drops the profile in, and optionally points the owner at Obsidian so they can watch their system's brain fill up from day one. That is all that happens early: an empty vault with one note. No structure, no discipline, no index yet.
- **Now (Step 2 of `03-build-order.md`, with the tools, before the crew).** This is the moment to build the real layer, and the timing matters: the crew comes next (Step 3) and every creative agent needs to read the owner's brand voice and preferences to stay on-brand. So the vault has to exist and hold those before the crew does. You build the folder structure and the dating discipline here, and load what already exists: the brand voice from Step 1 and the owner's preferences (no-go words, decisions, style). The lessons half fills in on its own later, once agents run and get things rejected (from Step 6 on), so you do not need it populated yet, just the structure and the preferences.

If the empty vault already exists from the interview, you are filling it in here, not starting over. If it does not (the owner skipped Obsidian, or you are retrofitting an older build), create it now.

## What memory is NOT

Be honest with the owner up front so they do not picture something fancier than it is.

- It is **not** a chatbot that "remembers your conversation." It is files on disk.
- It is **not** a giant AI model fine-tuned on their business. Nothing gets trained.
- It is **not** required to be a database. Flat markdown files are the recommended start and are enough for a long time.

Memory here means: **the right facts, written down in plain files, handed to the right agent at the right moment.** That is it, and that plainness is the point.

## The one rule that makes memory trustworthy

**A memory that is wrong is worse than no memory at all.**

If an agent reads "the owner's best-selling product is X" and that stopped being true two months ago, the agent now confidently does the wrong thing, and it does it fast, because it "knows." No-memory makes an agent ask. Bad-memory makes it act on a lie. So the whole module is built around one discipline: every fact is dated, and correcting a wrong fact is a first-class action, not an afterthought.

Three habits enforce this, and you build them in from day one:

- **Date every entry.** A fact without a date cannot be judged stale. `2026-07-14: switched primary POD supplier to Printify` can be trusted or retired later. "switched to Printify" cannot.
- **Correct in place, do not pile up.** When a fact changes, edit the file that holds it. Do not append a contradicting line below the old one and leave both. Two facts that disagree are worse than either alone.
- **When memory and reality disagree, reality wins, and you fix the file.** If an agent finds the live shop has 40 listings but memory says 25, the agent's job includes updating memory, not just noticing the gap.

Put this rule in front of every agent that reads or writes the vault. It is the memory equivalent of the approval board: the thing that makes the feature safe to leave running.

## What goes in the vault (and what does not)

Memory is for what is **true and not obvious from the code or the live platform.** If an agent can look it up in thirty seconds (current listing count, today's orders, the live price), it does not belong in memory, it belongs in a fresh read. Memory holds the things that took a decision or a mistake to learn.

Good vault entries:

- **Decisions and why.** "Chose Printify over Printful because of faster US shipping." The why is what stops the system from re-litigating it.
- **Owner preferences.** Voice, palette, no-go words, brands they will not touch, how blunt they want drafts. These drive every creative agent.
- **Lessons from failures.** The old `lessons.md`, folded in here. "Tags over 20 chars get truncated." "This niche saturates in Q4."
- **Facts about the business that are stable but not in code.** Supplier lead times, which platforms the owner actually operates on, seasonal patterns they have noticed.
- **Reference pointers.** Where the real thing lives: the shop URL, the analytics dashboard, the ticket the owner is tracking. A link, not a copy.

Keep out of the vault:

- **Anything a live read answers.** Current inventory, today's sales, this week's traffic. Read it fresh, do not cache it into a file that will lie tomorrow.
- **Secrets.** No API keys, tokens, or passwords in the vault, ever (rule 11 in `CLAUDE.md`). Memory is plain text the owner browses; secrets live in `.env` and nowhere else.
- **Raw dumps.** Do not paste a whole page in. Write the one line that is worth remembering and link to the source.

## Build it: start flat, one file per fact

Do not build a database. Build a folder.

```
memory/
  INDEX.md          <- one line per note: title + a five-word hook
  decisions/
    pod-supplier.md
    pricing-model.md
  preferences/
    brand-voice.md
    no-go-words.md
  lessons/
    listing-tags.md
  reference/
    dashboards.md
```

Each note is tiny: a title, a date, the fact, and why it matters. One fact per file. Small files are easy for an agent to read whole, easy for the owner to edit, and easy to retire when they go stale.

`INDEX.md` is the trick that makes this scale without a database. It is one line per note, a title plus a short hook, and it is the only memory file an agent loads *every* time. The agent reads the index, sees which few notes are relevant to the task in front of it, and opens only those. A hundred notes, and the agent still reads five. This is exactly how the build already treats `lessons.md`: relevant lines fed into the prompt, not the whole history dumped in.

**Working means:** an agent, before doing a task, reads `INDEX.md`, pulls the two or three relevant notes, and its output visibly reflects them (uses the owner's voice, avoids a no-go word, respects a past decision). After the task, if it learned something durable, it writes or updates a note.

**Prove it:** put a real preference in the vault ("the owner never uses exclamation marks"). Run a listing-draft agent. Show the draft obeys it without you re-stating it in the prompt. Then reject something for a new reason, and show the agent wrote a fresh lesson note on its own.

## Give the owner a way to see it: Obsidian (free, optional, recommended)

The vault is just markdown files, so the owner can open them in any text editor. But a plain folder of notes is hard to browse and connect. **Obsidian** is a free app that reads a folder of markdown files exactly as they sit on disk and turns it into a browsable, linkable knowledge base. No lock-in, no import, no new format: it points at the same `memory/` folder your agents read and write. The owner edits a preference in Obsidian, the next agent run picks it up, because it is the same file.

Why recommend it, in one honest sentence: it gives a non-coder a real window into what their system knows and lets them fix a wrong fact by clicking a note and editing a line, instead of hunting through folders.

Set it up plainly: install Obsidian, "open folder as vault," point it at `memory/`. Done. The `[[double-bracket]]` links between notes that Obsidian shows are also just plain text in the files, so they cost nothing and help the owner (and you) see how facts connect.

This is optional. The system works on the raw files without it. But for an owner who wants to actually see and steer their system's memory, it is the free upgrade worth making, and it is what I use to run my own.

## The optional upgrade: a search index (only when the vault gets big)

`INDEX.md` plus small files carries you a long way, a couple hundred notes at least. Past that, an agent scanning the index for "what is relevant" starts to strain, because a keyword match misses a note that means the same thing in different words.

When and only when that happens, add a **semantic index**: a tool that reads the whole vault and lets an agent ask "what do we know that relates to *this*?" and get back the right notes even when the wording differs. This is what a tool like Graphify does, build a searchable index over the folder, refreshed when the notes change, so retrieval stays sharp as the vault grows.

Treat this exactly like the VPS decision in `05-infrastructure.md`: **do not build it first.** It is a graduation step, not a starting point. Flat files and `INDEX.md` are the right build for every new system. Add the index the day the owner has enough notes that keyword lookup misses things, and not a day sooner. Building a semantic search layer over eleven notes is the vanity trap wearing a clever hat (rule 1 in `CLAUDE.md`).

If you do add it: the index is derived data, not the source of truth. The markdown files are the memory. The index is a fast way to search them and can be rebuilt from scratch any time. So it never needs its own backup, it needs the vault backed up (which Step 5 of `05-infrastructure.md` already covers).

## Back it up with everything else

The vault is the system's memory, so it is exactly as important as the database, and it gets the same protection. `05-infrastructure.md` already lists `lessons.md` and `PROFILE.md` in the daily backup. Widen that to the whole `memory/` folder. Everything the system has learned lives in those files. A wiped vault is a system with amnesia again, back to answering the same questions from scratch. Copy the folder somewhere off the machine, keep dated copies, and test one restore, same as the database.

## Where this plugs into the build

- **Two passes (see "Two moments" above).** The empty vault plus the profile is seeded at the interview close (`01-interview.md`). The full layer is built at Step 2 of `03-build-order.md`, with the tools and before the crew, so every agent can read the owner's brand voice and preferences from its first run. The ongoing lessons fold in later, from Step 6 on, as agents run. Seeding the empty vault early is fine and encouraged; the structured layer with the preferences loaded is the Step 2 job.
- **Feeds every creative and decision agent.** The listing agent reads brand voice and no-go words. The research scouts (`modules/research.md`) write findings worth keeping into `reference/`. The manager (`modules/manager.md`) reads decisions before it delegates. Wire the vault read into the agents that benefit; not every agent needs it.
- **Extends, does not replace, `PROFILE.md`.** `PROFILE.md` is the owner's fixed interview answers from day one. The vault is everything the system has learned *since*. Keep `PROFILE.md` as the stable base; let the vault grow.

## Say it to the owner plainly

> "Right now every agent forgets everything the second it finishes. This gives your system a memory: a folder of plain notes it reads before it works and writes to after. It remembers your decisions, your preferences, and its own mistakes, so it stops asking you the same things and stops repeating errors you already fixed. You can read and edit all of it yourself in a free app. And the one rule is that a wrong note gets fixed fast, because a system that remembers something false is worse than one that just asks."
