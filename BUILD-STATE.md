# Build state
Updated: 2026-09-15

## Where we are
Build order step: 1 of 7 (foundation / channel identity) - just completed.
Money-maker: content channel, short-form fantasy football clips (see PROFILE.md).
Infrastructure: settled as VPS (owner wants 24/7), actual server setup deferred
to the automation step.

## Done and proven
- Interview complete, PROFILE.md written and confirmed by owner.
- Channel identity approved -> `brand.md` (name: "Flock Fantasy Rewind", palette,
  writing voice). Owner picked the name knowing the risk (see below).
- Memory vault seeded: `memory/README.md`, `memory/lessons.md`, `PROFILE.md`.

## Half done / not started
- Nothing built yet for tools, spend cap, crew, or the pipeline itself.

## Blocked on the owner (read this first next session)
- **Content-sourcing permission (urgent, gates real progress).** The plan is to
  clip content from The Flock League / Flock Fantasy (a 12-team fantasy football
  YouTube ecosystem). Owner does NOT have confirmed permission yet, only that the
  league has talked publicly about wanting clippers. Owner chose to proceed and
  name the channel "Flock Fantasy Rewind" anyway, accepting the affiliation risk.
  Do NOT build the actual video-download/clip/repost pipeline (03-build-order.md
  Step 2 tool-wiring for this money-maker) and do NOT let the owner register
  public social handles or post publicly under this name until they confirm
  permission (reply to the league's clipper callout, join whatever
  program/Discord they referenced, or get a direct yes). Ask about this first
  thing next session if it has not come up.
- **Daily spend cap:** owner deferred picking a number. Need a real number
  before Step 2 (tool wiring / spend cap in code).
- **Budget tension:** Claude Pro (~$20/mo) + a VPS (~$5-6/mo) is at/over the
  owner's stated under-$25/mo cap. Not urgent, but flag again before signing up
  for a VPS.

## Next action
Once the owner confirms clipper permission (or decides to pivot), move to Step 2
of 03-build-order.md: wire the content tools (Zernio for posting, Higgsfield for
video, a male-voice TTS/voiceover tool), set the real daily spend cap in code,
and stand up the fuller memory vault. Do this BEFORE building the crew (Step 3).

If the owner has not resolved permission and wants to keep moving in the
meantime, safe work that does not depend on it: none of the actual sourcing
tools should be wired, but platform account creation groundwork (checking
handle availability across TikTok/Instagram/YouTube, per content-engine.md
Step 0) is fine to help with once the name is finalized.
