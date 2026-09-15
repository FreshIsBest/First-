> For the buyer's AI. How to get the owner set up and moving with as little friction as possible. The owner has read only READ-ME-FIRST.txt; they are talking to you in the Claude Code app right now. You handle every setup step from here and explain each one in plain language. Do not send the owner off to read this file: it is your guide, not theirs.

# Getting the owner set up (your job, not theirs)

The owner just told you to begin. They are a beginner, they are in the Claude Code app, and they have installed nothing else. Your first job, before the interview, is to get their environment right without scaring them. This page tells you how.

## The environment strategy: start easy, upgrade only when needed

There are two places the owner can work, and the honest move is to use both, in order:

- **Now: stay in the Claude Code app.** It is where they already are, and it is perfect for the early build: the interview, their brand, their first listings. That is all conversation and file drafting, which the app handles fine. Do not make a beginner install anything before they have seen a single result. Start here.
- **Later: move to VS Code.** When the build reaches the automation (`03-build-order.md` Step 6, the dispatcher that runs unattended), you need a real terminal to start and watch a long-running process, and to see the file tree. That is when you move them to VS Code, and you explain why at that moment: "Now we are building the part that runs on its own. For this we want VS Code, a free app that shows your files and gives us a terminal in one window. It takes a few minutes and I will walk you through it."

Say this plan to the owner up front, in one or two sentences, so the later move is expected: "We will start right here in the app, get your brand and your store built, and later, when we build the piece that runs everything automatically, I will move you to a free app called VS Code and walk you through it. Nothing to install yet."

This way the beginner never faces a terminal until they are deep enough not to be scared of it, and you narrate the whole way.

## Walking the owner through VS Code (when the time comes)

When Step 6 arrives, or if the owner asks to see their files sooner, set them up. Explain each click as you go (rule 8).

1. Download VS Code from its official site (code.visualstudio.com) and install it with the defaults.
2. Open it, click the Extensions button on the left (four squares), search for Claude Code, install the official one.
3. File, then Open Folder, and pick the Blueprint folder.
4. Open the built-in terminal (Terminal menu, New Terminal). This is where the commands you give them run, in the same window as their files.

**If the owner is on Windows,** the build's commands are written for a Linux-style shell, and Windows opens a different one by default. Set up WSL (below) and then tell them to switch VS Code's terminal to it: the small dropdown next to the plus sign on the terminal, choose WSL or Ubuntu instead of PowerShell. Do this once. Mac and Linux terminals already speak the right language.

## Installing the pieces the build needs (when a step calls for one)

You install these for the owner by giving them the exact command to paste, at the moment the build needs it. Do not front-load them. Confirm the current official install steps in your own knowledge or the tool's docs before handing over a command, because these change.

- **Node** (needed once the build runs scripts): check with `node --version`. If missing, walk them through installing the current LTS (Homebrew on Mac, the official installer or a version manager elsewhere).
- **WSL** (Windows only, for a Linux-style shell): from an admin PowerShell, the current Microsoft install (historically `wsl --install`), then a restart. Have them create a simple Linux username and password and write it down.
- **Any platform tool** (Etsy developer key, Higgsfield, Printify): these are wired at `03-build-order.md` Step 2, not here. When you get there, walk the owner through each, and always have them paste secrets into the `.env` file themselves, never into the chat (rule 11).

## The escape hatch (tell the owner this early)

Set this expectation before anything can break: "If you ever see red text or an error, do not worry. Copy the whole message, paste it to me, and tell me what you were doing. I read errors for a living and I will hand you the fix. You are not expected to know this."

That single habit unblocks almost everything. Make sure they know it before they hit their first error.

## Words the owner may ask about (the glossary)

Define these in one plain line each only if the owner asks or looks lost. Do not lecture.

- **Headless** - a program that runs on its own with no window to click, doing its job in the background.
- **Agent** - one AI worker given a single job and only the tools it needs.
- **Queue** - the to-do list of tasks waiting to be worked, in order.
- **Dispatcher** - the loop that pulls the next task and hands it to a fresh agent.
- **Board** - the approval screen where anything risky waits for a yes or no before it happens.
- **Payload** - the input for a task: the details the agent needs to do it.
- **Guardrail** - a hard limit set in code so the system cannot overspend or publish on its own.
- **API key** - a secret password-like string that proves the system is allowed to use a service; kept private.
- **VPS** - a cheap always-on computer rented in the cloud, so the system keeps running when the owner's machine is off.
- **WSL** - a Linux environment inside Windows, so Windows users can run the same commands as everyone else.
- **SQLite** - a tiny database that lives in one file, where tasks and records are stored.
- **.env** - a private file that holds secret keys, kept out of the code and never shared.
