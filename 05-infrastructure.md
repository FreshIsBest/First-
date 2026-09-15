> Where the system runs: local vs a cheap always-on server. For the AI to route the owner correctly and to explain the tradeoff in plain language, and for a non-coder owner to follow along.

# Infrastructure: where your Agentic OS actually lives

Your Agentic OS is a queue, a dispatcher loop, and some agent calls. That has to run on a computer somewhere. There are two honest options, and the right one depends on the owner's machine. Do not overthink this. Most people should start local.

## Windows note (read this first if the owner is on Windows)

Every code sample and command in this whole Blueprint is written for a bash shell (the shell on macOS and Linux). Many beginners are on Windows, and Windows does not run these directly. Two fixes, both fine:

- **WSL (Windows Subsystem for Linux, recommended).** It gives Windows a real Linux environment where every command in these files runs as written. Install it once, then do the whole build inside it. This is the smoothest path and the one to recommend to a Windows owner.
- **Git Bash.** A lighter option that ships with Git for Windows. It runs most of the bash samples here without a full Linux layer. Fine for a simpler build.

Two hard translations the AI must make on Windows, not just note:

- **Scheduling: use Task Scheduler, never cron.** Cron does not exist on Windows. Anywhere this Blueprint says "cron" or "a scheduled job," the AI sets it up through Windows Task Scheduler instead (or `schtasks` from the command line). Same outcome (a job that fires on a clock), different tool.
- **Paths and scripts.** Translate bash scripts to run in the chosen shell. Inside WSL, use Linux-style paths. In Git Bash, most POSIX paths work but watch for Windows drive letters. The AI translates the sample as needed and proves it runs, per rule 5 in `CLAUDE.md`. Do not hand a Windows owner a raw bash script that errors on the first line.

If the owner is on macOS or Linux, ignore this section, the samples run as written. The interview in `01-interview.md` records the owner's OS, so check `PROFILE.md` and route accordingly.

## First, check what the interview learned

Before you recommend anything, re-read `PROFILE.md`. The interview in `01-interview.md` asked about the owner's computer. You are looking for two things:

1. **How capable is their everyday machine?** A modern laptop or desktop from the last five or so years, with room to spare, is plenty. An old, slow, or already-overloaded machine is not.
2. **Do they need it running when they are asleep or away?** A daily sales digest at 8am, an overnight batch of draft listings, a job that fires every 30 minutes: those only matter if the machine is on and awake when they are supposed to fire.

Then route:

```
   +------------------------------+        +------------------------------+
   |  Decent / modern PC          |        |  Weak / old PC, OR wants      |
   |  (the common case)           |        |  true 24/7 always-on          |
   +------------------------------+        +------------------------------+
                |                                        |
                v                                        v
        PATH A: run local                         PATH B: cheap VPS
     (recommended default)                     (a small rented server)
```

Tell the owner which path you are recommending and why, in one plain sentence. For example: "Your laptop is plenty for this, so we will run it right here to start. If you ever want it running overnight while the laptop is off, we can move it to a small always-on server later without rebuilding anything."

## Path A: run it locally (recommended default)

This is the simplest, cheapest, fastest way to a first result. The whole system lives in a folder on the owner's own computer. The dispatcher loop runs in a terminal window (or as a small background process). No accounts to set up, no server to rent, nothing new to learn. This is where almost everyone should start.

**Why it is the default:**
- Zero extra cost. They already own the computer.
- Nothing to configure. It runs where you built it.
- Fastest path to the first win, which is the whole point (see the vanity-trap rule in `CLAUDE.md`: get it working and earning before anything fancy).

**The one honest tradeoff you must say out loud:**

Anything scheduled only runs while the computer is awake and on. If the owner sets up a daily digest at 8am but the laptop is asleep at 8am, the digest does not run. It does not queue up and fire late. It just misses. Same for a nightly batch job or a "check every 30 minutes" loop. When the machine sleeps, the work pauses. When it wakes, it picks up from wherever it was, but the missed scheduled runs are simply gone.

Say this plainly to the owner. Do not let them discover it by wondering why their morning report never showed up. That exact surprise is a logged gotcha (see `08-gotchas.md`: a local scheduled job silently stopped because the computer slept).

**The simple mitigations (usually enough):**
- **Keep the machine awake.** In the operating system's power settings, set the computer to never sleep while plugged in. On a desktop that runs all day, you are basically done.
- **Keep it plugged in.** A laptop on battery will sleep to save power no matter what. Plugged in, it stays up.
- **Close the lid setting.** On a laptop, set "closing the lid" to do nothing (not sleep) if they want to shut the lid and keep it running.
- **Run on-demand instead of on a clock.** If they do not truly need unattended scheduling, skip the schedule entirely. Have the owner start the dispatcher when they sit down to work, let it clear the queue, and stop it when they are done. No sleep problem exists if there is nothing scheduled to miss.

**When local is good enough (most people):**
- They work from one main computer that is on during the day.
- Their scheduled jobs can run while they are at the machine, or they are fine kicking them off manually.
- They do not need anything to happen at 3am while the house is dark.

If that describes the owner, stop here. Build local, get the first result, and only revisit this chapter if they outgrow it. Do not talk someone with a good laptop into renting a server they do not need.

### Back up the local system too (do not skip this)

Running local does not mean skipping backups. The VPS path has a backup step (Step 5 below) and the local path needs the same protection. Everything the system has done lives in a few files on one computer: a hard-drive failure, a bad reset, an accidental delete, or a spilled coffee erases all of it. Build the backup the same day you build the system.

What to copy, on a schedule (once a day is a fine start):

- **The SQLite database file** (the queue, statuses, results, history). This is the system's memory.
- **`lessons.md`** (the plain-text lessons the board has taught the system over time).
- **`PROFILE.md`** (the owner's interview answers the whole build is based on).

How to do it, plainly:

- **Copy those files somewhere off the computer.** Cloud storage (a synced folder, a storage bucket) or a second physical disk. A copy that only lives on the same drive it is protecting is not a backup, the drive that dies takes both.
- **Keep dated copies, not just the latest.** Name each backup with its date and keep the last 7 to 30. If the database corrupts, yesterday's copy is worthless once it has overwritten the last good one. Multiple dated copies let you roll back to a clean point.
- **Automate it.** On macOS or Linux, a small scheduled job (cron) that runs the copy daily. On Windows, the same thing through Task Scheduler (see the Windows note above). Do not rely on remembering.
- **Test one restore.** Copy a backup back into a fresh folder and confirm the system reads it and lists the same tasks. An untested backup is a guess, not a safety net.

One honest catch that ties back to the tradeoff above: a scheduled local backup only runs while the machine is awake, same as any local job. If the computer is asleep at backup time, that day's copy is missed. Either keep the machine awake for the backup window, or run the backup on-demand when you sit down to work. A wiped database with no backup is a real, logged failure mode (see `08-gotchas.md`), and it does not care whether you were local or on a server.

## Path B: a cheap always-on VPS

Choose this path when the owner's PC is too weak or too busy to host the system, or when they genuinely need it running 24/7 (overnight jobs, an early-morning digest before they wake, a loop that must never pause). This section is a full day-zero walkthrough written for someone who has never touched a server. Take it slowly and do each step with them.

**What a VPS is, in one plain sentence:** a small computer you rent by the month that lives in a data center and stays on all the time, so your system keeps running even when your own laptop is off.

That is the entire pitch. It is a computer that never sleeps, that you reach over the internet instead of by sitting in front of it.

### Step 1: Pick a provider and a small plan

Any of the mainstream low-cost cloud providers work. Look for one that offers a small Linux server (often called a "droplet," "instance," "VM," or "compute") for a few dollars a month. Tell the owner:

- **Pick the cheapest small Linux plan to start.** The system is light. A tiny server (roughly 1 shared CPU and 1 GB of memory) is enough to run the dispatcher and spawn agents. They can size up later in a couple of clicks if it ever feels slow.
- **Choose a recent long-term-support Linux version** when the provider asks (a current LTS release of a common distribution). You do not need to know the difference; pick the default recent LTS option.
- **Pick a data center region near them.** It barely matters for this, but closer is marginally faster.
- **Disclose the cost.** This is a real recurring third-party charge, a few dollars a month, on top of their AI engine. Say the number before they sign up (see `06-cost-and-billing.md`).

During signup the provider will ask how they want to log in. If it offers an **SSH key** option, prefer it (it is more secure than a password). If that is too much friction for a first-timer, a strong password is acceptable to start; you can add a key later. Either way, the provider shows the server's **IP address** (a string of numbers) once it is created. Have the owner copy that down. That address is how you reach the server.

### Step 2: Connect to it (SSH, in plain steps)

SSH is just the way you open a command line on a computer that is somewhere else. It is a text connection to the server. Nothing visual, just a terminal.

1. Open a terminal on the owner's own computer (Terminal on macOS or Linux, or PowerShell / Windows Terminal on Windows; all modern systems include SSH).
2. Type the connect command using the username the provider gave (often `root` on a fresh server) and the IP address they copied:

   ```
   ssh root@YOUR_SERVER_IP
   ```

3. The first time, it will ask if they trust this server. Type `yes`.
4. Enter the password (or it uses the SSH key automatically if they set one up).

They are now typing commands that run on the rented server instead of on their own machine. That is all SSH is. Everything from here happens on the server.

**Housekeeping to do once, right after first login** (walk them through it, explain each line in one plain sentence as you go, per rule 8 in `CLAUDE.md`):
- Update the server's software to current versions.
- Create a normal (non-root) user to run the system, rather than doing everything as the all-powerful root account.
- Because Linux distributions and their tools change over time, **do not trust hard-coded setup commands from any guide, including this one.** Look up the current official install steps for the tools you need on the distribution you actually picked, confirm them, then run them. This chapter teaches the shape of the work, not exact commands frozen in time.

### Step 3: Get the AI engine running on the server

The server needs the same brain the owner's laptop had: their AI engine from the interview.

- **If they use an OpenRouter or Hermes-style API engine:** this is the easy case. It is just API calls over the internet. Install whatever small runtime the scripts need, put the API key in a local `.env` file on the server (never printed, never committed, per rule 11 in `CLAUDE.md`), and the agents can call the model from the server exactly as they did locally.
- **If they use a Claude Code style CLI engine:** install that tool on the server following its current official instructions, then sign in the way that tool requires. Confirm the login actually works by running one tiny test call before you build anything on top of it.

Then copy the owner's system folder (the queue, dispatcher, prompts, and scripts) up to the server. A first-timer can do this with a file-transfer command over the same SSH connection, or by cloning from a private repo if they use one. Once it is on the server, run one task by hand and watch it complete. Do not move on until you have proven a single task runs end to end on the server. Prove it works (rule 5).

### Step 4: Run the dispatcher loop as a background service

On the laptop, the dispatcher ran in a terminal window that stopped the moment you closed it. On a server you want the opposite: the loop should keep running after you log out, and start itself again if the server reboots. That means running it as a **background service**, not as a window you have to keep open.

The concept: instead of launching the loop yourself and babysitting it, you hand it to the operating system's service manager and say "keep this running, restart it if it dies, and start it automatically on boot." Most modern Linux servers have a built-in service manager for exactly this. There are also simple process managers that do the same job.

The kind of setup you are describing to the owner:
- Define the dispatcher as a service: what command to run, which folder to run it in, which user runs it, and the rule "restart automatically if it stops."
- Enable it so it launches on every boot.
- Start it, then check its status and logs to confirm it is actually looping.

Because the exact tool and commands depend on the distribution the owner picked, **do not paste fixed commands from memory.** Look up the current documented way to define and enable a background service on their specific server, confirm it, then set it up. The goal is fixed even though the commands are not: a loop that survives logout, survives a reboot, and restarts itself if it crashes.

Verify it the honest way: start the service, disconnect your SSH session entirely, reconnect a minute later, and confirm the loop is still running and still picking up tasks. If it stopped when you logged out, it is not a service yet. Fix it before you call it done.

### Step 5: Back up the database (do not skip this)

This is the step people skip and regret. Your whole system's memory (the task queue, statuses, results, history) lives in one SQLite database file. If the server gets reset, rebuilt, or wiped, and you have no copy, all of it is gone. This is a real, logged failure mode: a database on storage that got reset was erased with no backup (see `08-gotchas.md`).

The fix is simple and you should build it the same day you build the server:

- **Copy the SQLite file somewhere safe, on a schedule.** Once a day is a fine starting point. "Somewhere safe" means off the server itself: down to the owner's own computer, or up to a cloud storage bucket. A backup that only lives on the same server it is protecting is not a backup.
- **Keep a few days of history, not just the latest copy.** Name each backup with its date and keep, say, the last 7 to 30. If something corrupts the database, yesterday's copy is worthless if it already overwrote the last good one. Multiple dated copies let you roll back to a clean point.
- **Automate it with the same background-service or scheduler approach from Step 4** so it runs without anyone remembering to.
- **Test a restore once.** Actually copy a backup back and confirm the system reads it. An untested backup is a guess, not a safety net.

Say this to the owner in plain words: "Your server holds the only copy of everything the system has done. We are making it copy that file somewhere safe every day, and we are going to prove we can get it back, so a bad reset can never erase your business."

## Which should you pick

| Situation | Pick | Why |
|---|---|---|
| Modern laptop or desktop, works during the day | Path A, local | Free, instant, fastest first win |
| Fine kicking off jobs manually, no overnight needs | Path A, local | No sleep problem if nothing is scheduled |
| Old or slow PC that struggles | Path B, VPS | Offload it to a machine built to stay on |
| Truly needs 24/7 (overnight jobs, pre-dawn digest) | Path B, VPS | A server never sleeps |

Default recommendation: **start local.** Get the system working and producing a real result on the owner's own machine first. That is the win that matters, and it costs nothing.

## You can graduate later without rebuilding

This is the important part, and it should lower the stakes on the whole decision: the system is the same code either way. The queue, the dispatcher, the agents, the board, the guardrails, they do not care whether they run on a laptop or a rented server. Moving from local to a VPS later is just copying the folder up, installing the engine, and starting the loop as a service (Steps 3 and 4 above). Nothing gets rebuilt.

So do not agonize over this now. Start local, get the first result, and if scheduling or 24/7 becomes a real need, come back to Path B and move it in an afternoon.
