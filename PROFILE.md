# Owner profile

Finalized. Confirmed by the owner. This file drives every later build decision.

## Section 0: Starting point

**0a. Tech comfort:** Total beginner, I want the AI to do everything.

**0b. Existing business:** Has an idea, not yet validated. (We'll pressure-test it before building, and get the specifics when we reach the money-maker question.)

**0c. AI tool installed:** Yes, installed and signed in.

## Section A: Where it runs

**1. Computer:** Decent, modern machine. Correction from owner: it is never powered off, but it does sleep/rest sometimes (not a true always-on machine while awake).

**2. 24/7 need:** Yes, always on -> VPS. Settled (Step 2 routing decision). Actual server signup and setup happens later, at the automation step (03-build-order.md Step 6 / START-HERE Step 8), per 05-infrastructure.md and SETUP.md. This pick is doubly confirmed since the machine sleeps sometimes, which would pause scheduled jobs if run locally anyway.

**2b. Operating system:** Windows -> commands run through WSL (recommended) or Git Bash; scheduled jobs use Task Scheduler, not cron.

## Section B: Engine and budget

**3. AI engine:** Claude Code Pro (~$20/mo).

**4. Monthly budget:** Under $25 (lean: engine only, no ads yet; build the smallest thing that ships).
Flagged: Claude Code Pro (~$20) + a cheap VPS (~$5-6, needed later for the 24/7 requirement above) puts the owner at or slightly over this cap. Owner has been told; revisit the exact number when we actually provision the VPS (automation step).

## Section C: Your money-maker

**5. First money-maker:** A content channel that drives traffic -> `modules/content-engine.md`.
Specifics: a short-clips content channel across Instagram, TikTok, and YouTube.

**5b. Trading module:** No / not now. Trading gate NOT passed. Do not build `modules/analyst-agent.md` or `modules/paper-trading.md`.

**6. Starting point:** Starting cold, nothing live yet.

**6b. Existing accounts / country / currency:** None yet (no Instagram, TikTok, or YouTube accounts). Country: United States. Currency: USD.

## Section D: Time and goal

**7. Time per week:** A few hours. Build the smallest thing that ships, nothing extra.

**8. Goal:** First real result, proof it works.

## Section E: Safety and approvals

**9. Approval tightness:** Tight (owner initially said Loose; advised of the early risk, owner chose Tight to start and will loosen once they trust the output).

**10. Daily spend cap:** Set it later. MUST be a real number before any spend-capable tool is wired up (Step 4 of the build). Revisit before then.

**11. Never-auto list:** Publishing (posts/videos), spending money, and sending messages/DMs all require the owner's click. Owner initially asked for auto-publish; told this conflicts with `CLAUDE.md` rule 3 (non-negotiable, no exceptions). Owner wants to revisit publish-approval friction (e.g. one-click "approve all pending", or an auto-approve-up-to-N-per-day setting) once they trust the crew's output, but a click will always be required before anything goes live.

## Section F: What "done for now" looks like

**12. First win:** First 10 posts published.

**13. Target timeline:** 1-2 weeks.
