> For the human buyer. How this product works, how to set up your AI engine, and the exact first move.

# How to use the Agenthusiast Blueprint

## What you actually bought

This is a guided prompt system, not a code repo. There is no app to install and nothing to compile. It is a folder of instruction files (`.md`) that you hand to an AI tool. The AI reads them, interviews you about your business, and then writes your code for you, tailored to your answers.

You do not need to know how to code. You do need to be willing to paste a few commands when the guide gives them to you, answer the interview honestly, and click approve. That is the real job. You will not write code, but you will run some, by copying and pasting exactly what the guide hands you. And when anything errors, you paste the error back to your AI and it fixes it. That skill, pasting an error and asking for the fix, is the only one you truly need. The Blueprint supplies the plan and the guardrails. Your AI supplies the labor.

## What you get: a branded Etsy store, plus the system that runs it

The headline outcome is simple: the AI builds you a branded Etsy store from scratch. It interviews you, then drafts a shop name, a logo, a color palette and brand voice, a starter line of products, the listings written in that voice, and a launch content plan. Everything lands as drafts you review in one pass, and once you approve, the store is launch-ready.

The way it does all that, and then keeps running the store day to day, is an Agentic OS: a small system of AI agents that runs a real online business mostly on its own, with you approving the moves that matter. Instead of one chatbot you talk to, it is a queue of tasks, a loop that hands each task to a fresh AI agent, and an approval board where anything risky (publishing a listing, spending money, sending a message) waits for your click before it happens. This is the same setup I run my own businesses on: one system runs my web agency (leads, invoices, analytics, outreach) and another runs my Etsy shop (daily sales digest, a print-on-demand pipeline, automatic Pinterest pins). You are going to build your own version.

## What this really is: a build, not a button (read this so you are not surprised)

Be clear-eyed before you start, because this is where eager people get frustrated and quit. You did not buy a button that spits out a store in five minutes. You bought the plan and the guardrails to **build a small system that runs a store**, with your AI doing the actual work. The store is real and it is the goal, but it comes *after* you build the engine that runs it. That order is the whole point, and it is why this works when "just make me a store" tools leave you with something you cannot run or grow.

Here is the honest shape of the journey, so you know where you are the whole time:

1. **Set up your tools** (install the AI, once).
2. **The interview.** Your AI asks about your business and writes it all down. This is what makes everything after it yours and not generic.
3. **Pick where it runs** (your computer or a cheap always-on one).
4. **Your brand, made from scratch.** Name options, a logo, colors, a voice, all as drafts you approve. This is your first real result, and it comes early on purpose.
5. **Connect your tools, set your spending limit, and start your system's memory.** Your Etsy and image tools get wired up and tested, a hard spending cap goes in that the system cannot cross, and your brand voice and preferences get saved into a memory the workers will read.
6. **Your workers.** Your AI builds a small crew, one at a time, each doing one job (writing listings, making images, and so on). They read your memory so they stay in your voice, and each one drafts for you to approve. None can publish or spend on their own.
7. **Your first real listing.** The crew makes one complete listing, image, title, description, price, all as a draft you approve by hand. This proves the whole thing works on one product before you do the rest.
8. **Your full store, and you go live.** The rest of your products and listings in your brand voice, as drafts. You review, click approve, and open your shop. Nothing goes live without your yes.
9. **The system that runs it on its own.** Now the automation gets built: the piece that works through jobs on a schedule so it keeps running while you sleep, with anything risky still waiting for your approval.
10. **Only then, looks**: polish and a nicer dashboard.

**How long?** Plan for a few focused sittings, not one afternoon. Some people move faster, some slower, and a couple of steps have short waits outside your control (for example, a new Etsy developer account can take a little while to be approved on Etsy's side). None of that is the system being broken. It is a build, and a build takes a few evenings. If you go in expecting that, you will finish.

The single most important habit: **let it go in order.** You will see your brand fast and your store soon after, which is the whole point. The automation that ties it all together comes last, once the pieces already work. Trust the sequence.

## The first move

1. Get Claude Code running. The app is the simplest way: download the Claude Code app, install it like any other app, and sign in. (See "Bring your own brain" just below for the engine options; if you are new, the Claude Code app on a Pro plan is the answer.)
2. Open this folder in Claude Code.
3. Say exactly this: **"Read START-HERE.md and begin."**

That is it. The AI takes over from there. It interviews you, then builds, and it handles every setup step for you along the way, including installing any extra tools and telling you whether to move to a free app called VS Code or just stay in the Claude Code app. You never follow install steps on your own; your AI walks you through each one when it is needed.

## Bring your own brain (your AI engine)

The Blueprint is the plan. You provide and pay for the AI that runs it. This is why you can run your own AI workforce for as little as ~$20/mo. Pick one of three engines:

| Option | Cost | Best for |
|---|---|---|
| **Claude Code Pro** | ~$20/mo flat | Starting out. Cheapest predictable Claude path. |
| **OpenRouter + Hermes** | Pay as you go | No subscription, open model, pay only for what you use. |
| **Claude Code Max** | Higher monthly | Heavy agent loads once your system is busy. |

**If you are new, pick Claude Code Pro. Do not overthink this.** Flat ~$20/mo, no metering to worry about, and it is the exact tool this Blueprint was built and tested on. Every example in these files is written for it, and the rulebook the AI follows loads automatically only in Claude Code. Install it, sign in with a Pro plan, open this folder, and give it the first-move line above.

**The other two are for later, or for people comfortable with code.** OpenRouter + Hermes has no subscription, but you connect it yourself with an API key and the code samples in this guide will not run exactly as written, you or your AI have to adapt them. That is fine if you know your way around an API, and a real headache if this is all new. Claude Code Max is the same tool as Pro, just a higher tier for when your system is busy. Start on Pro. You can switch later once you know what you are doing, and by then you will. Do not let the engine choice stall you on day one.

## Where it runs (your PC or a cheap server)

You can run this on your own computer, or on a cheap always-on server (a VPS) that stays up 24/7. You do not have to decide right now. Early in the build, your AI asks about your computer and helps you pick: a decent machine that can stay on usually runs it locally (simplest and cheapest), and an older machine or a real need to keep working while you sleep points to a small server. Either way the guide walks you through the setup step by step in `05-infrastructure.md`. No server admin experience required.

## What to expect

- **First, an interview.** The AI asks you about your business, your goal, your time, and your spending caps. One topic at a time. Answer honestly. It writes your answers to a `PROFILE.md` file that drives everything after.
- **Then, function first.** Your STORE gets its brand early, because a shop cannot open without a name and a logo, so those are real work. What waits is the SYSTEM's looks: no dashboards, no themes, no 3D world until the thing actually earns or does real work. It builds the working core (the task queue, the agent loop, the approval board, one real agent doing one real job) before any of that polish.
- **Looks come last, on purpose.** If you ask to make it pretty early, the AI will give you its honest take once (get it working first, then make it great), then do it your way if you still want to. That nudge is not a bug. It is the one habit that separates people who ship from people who spend week one picking colors and never make a sale. It is still your build.
- **You stay in control.** Nothing risky runs on its own. Publishing, spending, and sending all wait for your approval.

## Disclaimer (read this)

- **Results are not typical.** Building an Agentic OS does not guarantee income. What you earn depends on your product, your market, your effort, and factors outside anyone's control. My own results are mine and are not a promise of yours.
- **This is not financial advice.** The optional trading module is advanced and carries real financial risk. You can lose money. Nothing in this product is financial, investment, tax, or legal advice. Consult a licensed professional before putting money at risk.
- **You provide and pay for your own AI engine.** The Blueprint does not include an AI subscription or API credits. Claude, OpenRouter, and any other tools are billed to you by those third parties under their own terms and prices, which can change.
- **You are responsible for what your system does.** You approve the moves. Follow the terms of service of every platform you operate on (Etsy, Printify, ad networks, social platforms) and all applicable laws.
