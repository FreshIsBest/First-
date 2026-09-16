# Build state
Updated: 2026-09-15

## Where we are
Build order step: 2 of 7 (tools + spend cap + memory) for kereks_sports, IN
PROGRESS. Step 1 (foundation / channel identity) is DONE for the active path.
Money-maker: ACTIVE = kereks_sports, real sports card content (Instagram +
TikTok, already exist, dormant ~4 years). See PROFILE.md Section C-2.
Flock Fantasy Rewind (fantasy football) is PAUSED, not abandoned - see the
"PAUSED PATH" section below, preserved as-is.
Infrastructure: settled as VPS (owner wants 24/7), actual server setup
deferred to the automation step (Step 6).

## Done and proven (kereks_sports, active path)
- Gathered real details before building anything (rule 6): what "better"
  means (production quality + growth/engagement), old posts stay up with no
  comeback narrative, phone-only gear, small-but-real existing audience.
  Recorded in PROFILE.md Section C-2.
- Channel identity built and approved -> `brand-kereks-sports.md`: palette
  (red/white/blue, owner's own choice, concrete hex values given), voice
  guide (approved as drafted: knowledgeable/genuine tone, PC + flip example
  lines), platforms scoped to IG + TikTok only (not the 5-platform default).
  No brand name/handle decision needed - kereks_sports already exists.
- Spend cap set: $0/day for tool/API spend, in `config/spend-caps.json`.
  Confirmed separate from the owner's own card-buying budget (up to
  $200/day when actively flipping - owner's own money, spent by the owner,
  never automated, per rule 3). $0 works because this path needs no
  Higgsfield/Meshy (real phone photography) and Zernio's free tier (2
  accounts, no card) exactly covers IG + TikTok.
- `.gitignore` and `.env.example` added so the coming Zernio key never gets
  committed (rule 11). Owner still needs to create the real `.env` file
  themselves once they have the key.
- Fuller memory vault built per modules/memory.md: `memory/INDEX.md` plus
  `decisions/`, `preferences/`, `lessons/`, `reference/` folders, one fact
  per file, dated. Old flat `lessons.md` folded into the new structure and
  removed. Both paths' voice/preferences are in there; kereks_sports is
  marked active throughout.

## Half done / not started (kereks_sports)
- Zernio not wired yet. Free tier fits the $0 cap (2 accounts, no card),
  but signing up and connecting IG + TikTok is an OWNER action (needs their
  logins) - see Next action.
- No Higgsfield/Meshy needed for this path (real photography, not AI-
  generated visuals) - this differs from the blueprint's content-engine
  default and is noted in brand-kereks-sports.md so it is not mis-wired
  later.
- Crew (Step 3) not started - waiting on Zernio being connected first.

## Next action (blocked on the owner)
1. Owner signs up for Zernio (docs.zernio.com, free tier, no card needed)
   and connects the kereks_sports Instagram + TikTok accounts through
   Zernio's OAuth flow. This needs the owner's own logins, cannot be done
   for them.
2. Owner puts the real ZERNIO_API_KEY into a `.env` file they create from
   `.env.example` (never paste the key in chat).
3. Once both accounts show "connected" in the Zernio dashboard, confirm
   that with the owner (a "connected" status is enough proof for this step -
   defer an actual test post to Step 4, so the account's first real content
   in 4 years is real content, not filler).
4. Then move to Step 3: build the crew (a content writer + a
   caption/shot-list agent, no asset-generation agent needed for this path).

## PAUSED PATH (2026-09-15): Flock Fantasy Rewind - preserved, not deleted
Everything below was true when the owner paused this path to pivot to
kereks_sports. Nothing here was undone; resume it later if the owner wants.

- Interview complete, PROFILE.md written and confirmed by owner (Section C
  original answers).
- Channel identity approved -> `brand.md` (name: "Flock Fantasy Rewind",
  palette, writing voice, Name/Handle naming convention).
- Memory vault seeded: `memory/README.md`, `memory/lessons.md`, `PROFILE.md`.
- Shared login email created: flockfantasyrewind@gmail.com. Separate
  Facebook admin login: flockfantasyrewindfb@gmail.com (see brand.md).
- YouTube channel created (@FlockFantasyRewind - needs lowercase fix).
- TikTok account created (lowercase flockfantasyrewind).
- Facebook Page: created, but name stuck on the owner's personal name
  instead of "Flock Fantasy Rewind". Unresolved when the owner paused.
- Instagram: not started, blocked on the Facebook Page fix above.
- Pinterest: not started.
- Profile pictures: not decided. Confirmed: do NOT reuse The Flock League's
  own logo/photo (see memory/lessons.md).

**Still gating this path if it resumes:** clipper permission from The Flock
League. Owner could not reach Mason Dodd directly; the only confirmation
obtained was a DM "yes" from an unnamed team manager, flagged as too
narrow/unverifiable to authorize clipping the league's footage. Decision
made: if resumed, the pipeline uses real fantasy stats/facts + AI-generated
visuals (Higgsfield), NOT lifted footage - this sidesteps the rights
question. The name "Flock Fantasy Rewind" still implies affiliation and was
never covered by that decision either; do not register public handles or
post publicly under this name until permission is actually confirmed.

**Budget tension (still real if both paths ever run together):** Claude Code
Pro (~$20/mo) + a VPS (~$5-6/mo) is at/over the owner's stated under-$25/mo
cap. Not urgent for kereks_sports alone (little to no metered spend), but
flag again before provisioning a VPS if Flock Fantasy Rewind resumes too.
