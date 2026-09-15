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

**Content source and rights (important, read before building the sourcing/asset pipeline):**
The intended content is clips pulled from an existing YouTube ecosystem: "The Flock League" / "Flock Fantasy," a 12-team fantasy football league with a group channel (daily videos) plus 12+ individual creator channels for each team owner.

The owner does NOT currently have direct, confirmed permission to clip and repost this content. The owner reports the league/creators have talked publicly about wanting clippers, but the owner has not personally reached out or been confirmed into any clipper program.

**This is a hard gate, not a formality.** Downloading and republishing another creator's video content without permission is very likely copyright infringement (Content ID claims, takedowns, platform strikes, possible channel bans -- hard to undo). The AI declined to build an automated scrape-and-repost pipeline against unconfirmed permission and will not build the content-sourcing / asset-pulling tooling for this specific source until the owner has an actual confirmed yes (replying to the league's clipper callout, joining whatever program/Discord they referenced, or a direct DM/agreement).

**What is NOT gated and can proceed now:** the channel's own identity (name/handle, look, voice) -- this does not depend on the rights question and is safe to build today (Step 1 of 03-build-order.md).

**Decision (updated):** owner could not reach Mason Dodd directly, and the only
confirmation obtained was a DM "yes" from an unnamed team manager, which the AI
flagged as too narrow/unverifiable to authorize clipping the full league's
content. Owner agreed to pivot: instead of clipping real Flock League footage,
the content pipeline uses **real fantasy football stats/facts + AI-generated
visuals** (the content-engine.md module's actual default design - the asset
step generates video from a text brief, it does not require lifting anyone's
existing footage). This sidesteps the rights question entirely for the
pipeline itself. The channel can still reference real, public fantasy
outcomes/stats as talking points (facts are not copyrightable); it must not
download or repost anyone's actual video footage.
Standing note: the channel name "Flock Fantasy Rewind" still implies
affiliation and was not covered by this decision either - see the Name entry
in brand.md.

## Section C-2: ACTIVE money-maker (pivoted, same day)

Owner paused the above and pivoted to an EXISTING sports card account instead:
- Handle: kereks_sports (Instagram and TikTok - already exists, dormant).
- Niche: mainly football cards, some basketball.
- Vibe: genuine hobbyist/collector showing off his own collection, with a
  secondary goal of flipping cards for profit. Not purely business-flip
  content, and not purely PC (personal collection) showcase - both.
  This is entirely the owner's own real cards/photography/video - no
  third-party content rights questions at all, unlike the fantasy football
  path above.
- The actual problem: not content quality, just inactivity - no posts in
  about 4 years (since college).
- Owner does NOT want to address the gap as a "comeback" narrative - just
  start posting fresh going forward, no explanation of the hiatus.
- Everything else from the original interview (Section 0, A, B, D, E, F)
  still applies unless the owner says otherwise: total-beginner tech comfort,
  Windows, VPS routing decision, Claude Pro engine, a few hours/week, tight
  approvals, first win = first 10 posts. Daily spend cap still not set, but
  likely moot for this path since real card content needs no paid AI
  generation tools (it is the owner's own photography/video).

**Gathered details (2026-09-15, after the pivot, before building anything):**
- "Better content" means, in the owner's own words: production quality and
  growth/engagement. Not just posting volume, and not purely about closing
  sales - though the flip side still matters (see content mix above).
- Old posts from ~4 years ago stay up as-is. No comeback narrative or
  explanation of the gap; new content just starts appearing.
- Gear: phone only, no lighting/backdrop/ring-light setup yet.
- Current account size: small but real (roughly a few hundred to a couple
  thousand followers) - worth re-engaging, not a cold start.
See `brand-kereks-sports.md` for the approved channel identity (palette,
voice, platforms) built from these answers.

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
