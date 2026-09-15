> For the buyer's AI. Create the owner's brand identity from scratch: shop names, a logo, a color palette, a brand-voice guide, and a product-design style. Everything is a draft on the board until the owner approves it. The approved kit is written to brand.md and feeds modules/etsy-store-builder.md.

Level: beginner-safe | Time: ~1 evening | Cost: $0 to draft, small image-generation credits for the logo, plus your engine

# Brand kit

This is where the store gets its identity. Before the AI builds a single listing or product, the owner needs a brand: a name, a look, a voice. This module runs a short interview, then generates the whole kit as drafts the owner approves on the board. When they say yes, the AI writes it all to a `brand.md` in their project. Every later step (the store setup, the product designs, the listings, the pins) reads that file and stays on-brand.

This is the first piece of the flagship build. It feeds `modules/etsy-store-builder.md`, which reads the approved `brand.md` and turns it into an actual store. Run this first, or point the store builder at an existing `brand.md` if the owner already has one.

One honest note up front, per the vanity-trap rule in `CLAUDE.md`: branding is looks, and looks normally come after function. The exception is the flagship. A store cannot exist without a name and an identity, so a lean, decided brand IS the function here. Keep it lean. Do not spend three evenings on logo variations. Get a name, a look, and a voice locked, then move to building the store that earns.

## Step 1: the brand interview

Ask the owner these, plainly, one at a time. Keep it short. You are collecting just enough to generate a kit, not running a branding agency.

1. **Niche.** What do you sell, or want to sell? (Cozy home printables, dog-mom mugs, minimalist wall art, gym humor tees.) Get specific enough to design for.
2. **Vibe / aesthetic, in a few words.** How should it feel? (Cozy and warm. Bold and loud. Clean and minimal. Retro 70s. Dark and moody.) Three or four words is plenty.
3. **Who you sell to.** Picture one buyer. (New moms. Gym guys in their 20s. People decorating a first apartment. Coffee-obsessed remote workers.) The voice and look serve them, not the owner's taste.
4. **Any names you already like,** or words, themes, or vibes you want in the name. Also anything to avoid. (Optional. If they have nothing, say so and you will generate from scratch.)

Write the answers into the owner's `PROFILE.md` if they are not already there (per rule 6 in `CLAUDE.md`, the build comes from their real answers). If an answer is vague, ask one follow-up. Do not guess the niche or the audience, those two decide everything downstream.

## Step 2: generate the kit as drafts the owner approves

Now generate the five pieces below. **None of it is final.** Each piece is a draft that waits for the owner's yes before it is saved, exactly like every other lasting action in this system (rule 3 in `CLAUDE.md`). Branding is lasting, so it gets the same gate.

**Timing note (important).** This module is Step 1 of `03-build-order.md`, which is BEFORE the automated queue and board get built (that is Step 6). So do not create tasks or a `needs_approval` board here; there is nothing to create them in yet. "A draft the owner approves" means exactly that: you generate each piece, show the owner the real thing (the actual logo image, the real name list), and wait for their explicit yes before writing it to `brand.md`. Save any generated files (the logo images) into a simple drafts folder in the project so the owner can open them. If you ARE building this later, after the system exists, use the queue and board the normal way; the approval logic is identical either way.

### a. Shop name (3 to 5 options)

Generate 3 to 5 shop-name options that fit the niche, the vibe, and the audience. For each, give the name plus a one-line reason it fits. Aim for names that are short, easy to spell, easy to say, and not boxed into one product (so the shop can grow).

Attach this instruction to the draft, and say it to the owner out loud:

> **You must check availability on Etsy yourself.** Shop names are first-come and I cannot see what is taken. Before you fall in love with one, search it on Etsy, and ideally check that a matching domain and social handle are free too. Pick the first one on your list that is actually available.

Do not claim a name is available. You cannot verify it. The owner does the availability check; you just supply good candidates.

### b. Logo (a couple of options, board-approved)

Generate two logo options with an AI image generator. **Reference the generator generically:** the owner picks a current image tool (there are several good ones, and they change often), and you verify its real API, current model, output formats, and cost in that tool's current docs before wiring anything. Same honest posture as the other tool files in this Blueprint: it is a third-party paid service, credits cost money, tell the owner the cost, and put the API key in `.env`, never in the chat (rule 11).

Design guidance for the prompt:
- A clean, simple mark that reads at small sizes. It becomes the Etsy shop icon (a small circle) and sits on pins and products, so it must survive being tiny.
- Match the vibe words from the interview and the palette from part (c).
- **Do not bake text into the image.** AI image generators garble multi-word text (this is a known failure mode, see `08-gotchas.md`). Generate the mark or symbol clean, then overlay the real shop name yourself in a normal font if a wordmark is wanted. A clean symbol plus real typed text beats a garbled AI wordmark every time.

Generate two distinct directions so the owner has a real choice, render them to files under `board/pending/<task_id>/`, and put the file paths in `result.preview`. **Always preview the real rendered image, never approve a logo from a text description** (per `07-approval-loops.md`). If the owner rejects both, take their note as `owner_feedback` and generate a fresh pair. Do not settle on a logo the owner has not seen with their own eyes.

The shape of a generation step, once you have confirmed the real tool and API:

```bash
# Illustrative only. Confirm the real endpoint, model, fields, and output format
# in your chosen image tool's current docs, and tell the owner the credit cost.
curl -X POST "https://api.your-image-tool.example/v1/images" \
  -H "Authorization: Bearer $IMAGE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "minimal flat logo mark, single cozy coffee-cup symbol, warm terracotta and cream, no text, centered, clean vector style",
    "size": "1024x1024"
  }'
# Save the images under board/pending/<task_id>/ and put the paths in result.preview.
```

### c. Color palette (a few hex colors)

Pick a small palette tied to the vibe: roughly 3 to 5 colors. Give each as a hex code with a plain-language role, so later steps know where to use each one.

A workable default shape:
- A primary (the brand's main color).
- A secondary or accent (for buttons, highlights, pin hooks).
- A dark (for text).
- A light or background (for canvases and mockups).

Tie the choices to the vibe words. "Cozy and warm" leans terracotta, cream, soft brown. "Clean and minimal" leans off-white, charcoal, one quiet accent. "Bold and loud" leans high-contrast and saturated. State the reason in one line so the owner can react to it. Example:

```
Primary   #C4623B  terracotta, warm and cozy, used for the logo mark
Accent    #E8A87C  soft peach, for pin hooks and buttons
Dark      #2E2A26  near-black brown, for all body text
Light     #F5EFE6  warm cream, backgrounds and mockup canvases
```

### d. Brand-voice guide (short)

Write a short voice guide so every later piece of copy (listings, the About section, pins, emails) sounds like the same shop. Keep it to a few lines the owner can actually hold in their head:
- **Tone:** two or three words for how the brand talks (warm and encouraging, dry and funny, calm and premium).
- **Words to use:** a handful of on-brand words and phrases the copy should lean on.
- **Words to avoid:** a handful to stay away from (corporate filler, hype, anything off-vibe).
- **One example line** in the voice (a sample product blurb or tagline), so the owner hears it, not just reads rules about it.

This guide is what keeps a 40-listing shop from sounding like it was written by 40 different people. The listing agent in `modules/etsy-autopilot.md` and the store setup in `modules/etsy-store-builder.md` both read it.

### e. Product-design style

Define the visual direction for the print-on-demand designs themselves (the art that goes on the mugs, tees, prints, totes). This is separate from the logo: it is the recurring look of the products. Cover:
- **Style:** the art direction (hand-lettered quotes, minimalist line art, retro badges, bold typographic, watercolor).
- **Palette use:** which of the brand colors the designs pull from.
- **Motifs or themes:** recurring subjects that tie the line together (coffee, plants, mountains, dogs).
- **What to avoid:** anything that would break the look (clip-art, clutter, off-brand colors).

This is the brief the POD design step in `modules/pod-printify.md` follows so every product looks like it belongs to the same shop. A consistent product style is what turns a pile of listings into a recognizable brand.

## Step 3: write the approved kit to brand.md

Once the owner approves the pieces on the board, write the final kit to `brand.md` in the root of their project (a single file is simplest; a `brand/` folder is fine if the owner wants the logo files kept alongside it). This file is the on-brand source of truth every later step reads. Structure it plainly:

```
# Brand: <chosen shop name>

## Name
Chosen: <name>  (owner confirmed available on Etsy: yes/no + date)
Alternates considered: <the other candidates>

## Logo
File: brand/logo.png  (and any variants)
Notes: <what the mark is, where it is used>

## Palette
Primary  #C4623B  terracotta - logo mark
Accent   #E8A87C  soft peach - buttons, pin hooks
Dark     #2E2A26  near-black brown - body text
Light    #F5EFE6  warm cream - backgrounds

## Voice
Tone: warm, encouraging, unfussy
Use: cozy, simple, everyday, made-for-you
Avoid: luxury, exclusive, hustle, corporate filler
Example line: "The mug that makes the first slow morning feel like a win."

## Product-design style
Style: hand-lettered quotes + simple line motifs
Palette use: dark on light, terracotta accents only
Motifs: coffee, mornings, cozy home
Avoid: clip-art, clutter, neon
```

Copy the approved logo image(s) from `board/pending/<task_id>/` into the project (for example `brand/logo.png`) and reference them by path in `brand.md`. Only the approved pieces go in. Rejected options stay out; if the owner killed a direction outright, note it in `dead-ideas.md` so you do not regenerate the same thing later.

Add one line to `lessons.md` capturing anything the owner reacted strongly to, in plain text, one readable line (for example: `Owner hates the word "premium" in copy, keep it out.`). That memory keeps future copy on-brand without re-asking.

## Prove it and hand off

Per rule 5 in `CLAUDE.md`, do not call the brand done on faith. Confirm, out loud, that:
- `brand.md` exists in the project and reads back correctly.
- The approved logo file(s) are saved and the paths in `brand.md` resolve.
- The owner has confirmed (or been reminded to confirm) the chosen name is available on Etsy.
- Every piece in `brand.md` was actually approved on the board, none slipped in unreviewed.

Then tell the owner in one or two plain sentences what they now have and what happens next: the brand is set, and `modules/etsy-store-builder.md` will read `brand.md` to build the shop identity, the starter products, and the listings, all as drafts under one board review. Nothing here is locked forever; the owner can edit `brand.md` any time and later steps will pick up the change.

---

_Nothing in this kit is final until the owner approves it on the board. Brand results are not typical or guaranteed; a strong brand helps a shop stand out, it does not promise sales. Third-party image tools charge credits for generation, disclose that cost to the owner and keep the API key in `.env`, never in chat._
