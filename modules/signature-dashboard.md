> For the buyer's AI. Turn the owner's approval board into a walkable 3D world: a signature ops dashboard that is also filmable content. Three finished worlds come with the purchase as a separate Worlds Pack download (Step 1 walks the owner through getting it), so there is no generation step to start. Build this only after the system works (Step 9), read live board data on top of it, and never give the pretty world any powers the text board did not have.

# The signature 3D dashboard: your ops screen and your best ad, in one

Level: intermediate, optional payoff | Time: ~1-2 evenings | Cost: $0 (the worlds ship with the Blueprint) plus your engine

This is the payoff build. Not the first build. Read the next section before you touch a line of code, because if you build this at the wrong time you are working directly against the whole point of the Blueprint.

## Read this first: it is looks, and looks come last

This is a dashboard. A dashboard is looks, and looks come after function. That is rule 1 in `CLAUDE.md`, the vanity-trap rule, and it does not bend for this file just because the result is cool.

Build this ONLY after the system earns and runs on its own. That is Step 9 in `START-HERE.md`, the "only now, make it look good" step, the same slot where `07-approval-loops.md` builds the flat approval page. The order is fixed: the queue, the dispatcher, the agents, the board, the guardrails, and a first money-maker that produced a real result the owner approved from the command line, ALL of that exists and runs before this world gets loaded. If the owner points at this module on day one and wants to build the walkable city instead of shipping their first listing, give them the honest line from `CLAUDE.md` once:

> "My honest take: get it working first, then make it look great, that is how you actually finish. Every hour on the 3D world before the system earns is an hour it is not earning. But it is your build. Want me to build the world now, or after the first result?"

Then respect the call. Nudge once, do not fight. But do not pretend this is anything other than what it is: the reward for a working system, not a substitute for one. A beautiful empty world dashboarding a business that makes nothing is the vanity trap in its purest form. The world earns its place the day it has real board numbers to show.

If you have read this far and the system is genuinely running, good. Now build the fun part.

## Why this exists: two jobs, one build

Most dashboards do one job: they show the owner their numbers. This one does two, and the second is why it is worth building at all.

**Job one: it is a real ops screen.** The owner opens the world and reads their actual board off it. Pending approvals, task status, today's earnings, which agents are running right now. It is the same data the command-line board and the flat approval page in `07-approval-loops.md` show, rendered as a place instead of a list. Same board, same statuses, same underlying logic from `02-architecture.md`. Only the surface changed, and the surface is a world you can move through.

**Job two: it is filmable content.** This is the unfair part. A screen-recording of a glowing neon harbor where the docks light up as work lands, where a marker pulses over the town hall because three things need the owner's approval, is a piece of content. It is screenshot bait and TikTok bait. It makes the brand recognizable the way the signature 3D world in the Meshy section of `tools/tools.md` makes faceless content compound: same world in every clip, so the moment someone sees it in the feed they know whose it is. Except here you get it for free, because you were going to look at your ops screen anyway. The dashboard you use to run the business is also the content that grows the audience for the business.

That two-in-one is the whole thesis. A normal analytics dashboard is a cost, time spent looking at numbers. This one pays for the time twice: once as the screen the owner runs the business from, once as raw material for `modules/content-engine.md`. Feed a clip of the live world into the Higgsfield and Zernio flow in `tools/tools.md`, the same as any other signature visual, and the ops screen becomes a post. Nothing else on the owner's stack does both.

This is the sibling of the Meshy "signature 3D world" section in `tools/tools.md`. That section builds a custom world from scratch with a paid generator. This module is the instant-start version: a working world comes with the purchase, wired to live data tonight.

## Step 1: pick a world (four come with the purchase)

Three real, finished `.glb` 3D worlds come with the Blueprint. The owner does not generate anything to start. They pick one, you load it, done.

**They are a separate download, on purpose.** The worlds are large (tens of MB each), and they are not needed until this step, which is the last step of the whole build. So they are not inside the Blueprint folder. They are a second file on the owner's purchase, called the **Worlds Pack**, in the same place they downloaded the Blueprint (their Gumroad library, linked from their receipt email).

Walk the owner through it here, at the moment they actually need it, and explain each click (rule 8):

1. Open the download link in their purchase receipt, or their Gumroad library.
2. Download **Agenthusiast-Worlds-Pack.zip**.
3. Unzip it and put the resulting `worlds` folder inside an `assets` folder in the Blueprint folder, so the paths end up as `assets/worlds/harbor.glb` and so on.
4. Tell you when it is done, and you verify the file is there before you write a line of loader code.

If the owner cannot find it or the download fails, that is a support question, not a build question: point them at the Discord in `READ-ME-FIRST.txt` rather than trying to work around it. Do not substitute a placeholder cube and call the step done.

The four, and the vibe each carries:

- **`assets/worlds/harbor.glb`** (neon harbor). Docks, water, hard neon at night. Reads as busy, coastal, a little cyberpunk. Good for a brand that wants energy and edge. (Mine is a neon harbor: it shows up in everything I post, and that consistency is the point.)
- **`assets/worlds/homestead.glb`** (dusk farm). Barns, fields, warm low light. Reads as grounded, calm, made-by-hand. Good for a craft shop, a maker, a homestead or food brand.
- **`assets/worlds/agent-city.glb`** (3D city). Streets, blocks, buildings. Reads as a whole operation, a lot of moving parts. Good for a brand that runs several agents and wants the world to feel like a small metropolis.
- **`assets/worlds/station.glb`** (orbital station). Modules, docking arms, hard light against black. Reads as precise, technical, a little sci-fi. Good for a brand that is software, AI, or tooling rather than physical goods, and it is the easiest of the four to film, because a slow orbit against an empty background needs no set dressing to look expensive.

Have the owner pick the one that fits their brand, not the one that looks coolest in isolation. The world is going to show up in their content, so it should match how they want to be recognized. There is no wrong pick and it is not permanent: they can swap the file later, and if they outgrow the shipped worlds they can generate a fully custom one through the Meshy section of `tools/tools.md` and drop it in the same slot. Start with what is in the box. It works out of the box, which is the whole advantage over generating first.

One honest note on the files: these `.glb` worlds are large, tens of MB each. That is why they download separately, and it matters for how you load them (see Step 6). Pick one and serve one; do not load all four.

## Step 2: map the geometry to meaning

A world with nothing wired to it is a screensaver. The value comes from mapping the buildings, zones, and docks in the chosen world to the owner's actual agents and business surfaces, so that looking at a place tells the owner something true about their system.

Do this mapping WITH the owner, from their real setup in `PROFILE.md` and the task types they actually run (rule 6, build from real answers). A rough mapping, using the harbor as the example:

- **A dock = a money-maker agent.** One dock is the Etsy shop agent (`modules/etsy-autopilot.md`), another is print-on-demand (`modules/pod-printify.md`), another is the content engine (`modules/content-engine.md`). A dock lights up or shows cargo when that agent has work moving.
- **A building = a surface or a support agent.** The tall building is the content agent, the warehouse is research (`modules/research.md`), the workshop is the maintenance script.
- **The town hall (or harbor master, or the farmhouse) = the approval board.** This is the center of the world and the most important marker, because it is where the owner acts. When something needs approval, this is what glows.
- **Zones = states.** A lit zone means active work. A dark zone means that agent is idle. A pulsing marker means something there wants the owner's attention.

Keep the mapping legible. The owner should be able to glance at the world and know, without reading a single label, that the shop is busy and the approval hall needs them. If the mapping needs a legend to understand, it is too clever. Simple and readable beats a perfect one-to-one of every task type. Pick the handful of surfaces that matter and map those.

Write the mapping down as plain config (a small JSON file: which mesh or coordinate maps to which agent or status), so it is easy to read and change later, same spirit as the plain files everywhere else in the system (rule 7). When the owner adds a fourth money-maker, they add a line to the map, not a rewrite.

## Step 3: surface the live board data inside the world

Now the world tells the truth. The data it shows is the exact same board data everything else reads, straight from the tasks table in `02-architecture.md`. Nothing new gets computed and nothing new gets stored. The world is one more reader of state.

The live signals worth surfacing, and where each comes from:

- **Pending-approval count.** Count the rows at `needs_approval`. Render it as a glowing marker over the town hall (the approval surface). Three pending, the marker pulses and reads "3". Zero pending, it goes quiet. This is the single most useful thing on the screen, because it is the one that means "the owner needs to do something."
- **Task status per agent.** For each mapped dock or building, read the current status of that agent's task type: running, queued, done, failed. Light the zone accordingly. A failed task should read as clearly wrong (a red marker, a dead dock), because a quiet failure is the thing the owner most needs to catch.
- **Today's earnings.** If a money-maker writes results the owner tracks (a sale, an order), sum today's and float it over the relevant dock. Read it from the same result data the board already has; do not build a second books.
- **Agents currently running.** The dispatcher marks tasks `running`. Show those as active: a lit dock, a moving indicator, whatever fits the world. The owner sees the system working in real time.

The hard rule over all of it: **the dashboard READS state. It never writes, publishes, or spends.** This is a window onto the system, not a lever on it. That is rule 3 in `CLAUDE.md` and the guardrail spine of the whole Blueprint, and the pretty world gets zero exceptions. A glowing approval marker is not an approve button that acts. When the owner wants to actually approve something, the world links out to the real approval flow: the flat approval page from `07-approval-loops.md`, or the command-line board, wherever the owner already approves. The click that changes state happens there, through the mechanism that already works and already has the guardrails on it. The world can deep-link to it (click the town hall, it opens `approval.html` filtered to the pending items). It does not reinvent it.

Say this plainly to the owner so there is no confusion: "This world shows you everything. It does not do anything. When you want to approve, it takes you to the same board you already use. No pretty button ever spends your money or posts for you." The no-auto-publish rule does not get a loophole because the UI got nicer, exactly as `07-approval-loops.md` says about the flat page: a prettier button is still just a human click, and this world does not even have the button, it has a link to the button.

## Step 4: the concrete build

Small web app, owner's machine, no framework sprawl. Here is the shape.

**Load the world.** Use three.js with `GLTFLoader` to load the chosen `.glb`, or react-three-fiber if the owner already lives in React (`modules` code elsewhere leans plain, so plain three.js is the lighter default). Add `OrbitControls` for a look-around view, or simple WASD/pointer-lock controls if the owner wants to walk through it. Orbit is less to build and reads fine for a dashboard; offer walk controls as the upgrade once orbit works.

**Overlay the live data as HTML, not 3D text.** The markers (pending count, earnings, status) are plain HTML/CSS elements positioned over the canvas, or three.js `CSS2DRenderer` labels anchored to world coordinates. HTML overlay is far easier to style, read, and update than rendering text into the 3D scene, and it keeps the data layer separate from the world layer so you can restyle numbers without touching geometry.

**Poll the board, do not stream it.** Every few seconds, fetch the current board state from the same local endpoint the flat approval page uses (`/api/pending` and a small status endpoint from Path 1 in `07-approval-loops.md`, reading the same tasks table). Update the markers from the response. Use plain polling, not websockets. This is a dashboard the owner glances at, not a trading terminal; a 3 to 5 second poll is simpler, has no connection to keep alive, and is plenty fresh. Keep it simple (rule 7).

**Owner-only, HTTP Basic auth.** This screen shows the owner's earnings and their whole operation. It is exactly the access posture the rest of the system holds: owner-only, local. Serve it from the owner's own machine behind HTTP Basic auth so a wandering browser cannot open it. If the owner later wants it reachable from their phone, that is the `05-infrastructure.md` conversation, same as the flat page, not something you expose casually. Do not host this publicly with real numbers on it. (A separate, faked-number version for filming is fine and even smart, see Step 8, but the live one stays private.)

An illustrative sketch of loading the world and hanging one data marker on it. This is a shape to build from, not a copy-paste: **verify the current three.js docs** for the exact import paths and API, because three.js moves its module layout and `GLTFLoader`/controls locations between versions.

```javascript
// Illustrative only. Confirm current three.js module paths and API in today's docs.
import * as THREE from "three";
import { GLTFLoader } from "three/addons/loaders/GLTFLoader.js";
import { OrbitControls } from "three/addons/controls/OrbitControls.js";

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(60, innerWidth / innerHeight, 0.1, 1000);
camera.position.set(0, 15, 30);

const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(innerWidth, innerHeight);
document.body.appendChild(renderer.domElement);

const controls = new OrbitControls(camera, renderer.domElement);
scene.add(new THREE.HemisphereLight(0xffffff, 0x223344, 1.2));

// Load the shipped world ONCE. It is tens of MB, so this is slow the first time.
new GLTFLoader().load("assets/worlds/harbor.glb", (gltf) => {
  scene.add(gltf.scene);
});

// The town-hall marker is a plain HTML element positioned over the canvas.
const approvalMarker = document.getElementById("approval-marker");

// Poll the SAME board the flat approval page reads. Never write, only read.
async function refresh() {
  const tasks = await fetch("/api/pending").then((r) => r.json());   // needs_approval rows
  const n = tasks.length;
  approvalMarker.textContent = n ? `${n} waiting` : "clear";
  approvalMarker.classList.toggle("pulse", n > 0);   // glow only when action is needed
  // Clicking the marker opens the REAL approval flow. The world does not approve.
  approvalMarker.onclick = () => (location.href = "/approval.html");
}
refresh();
setInterval(refresh, 4000);   // plain polling, a few seconds is plenty fresh

renderer.setAnimationLoop(() => {
  controls.update();
  renderer.render(scene, camera);
});
```

Note what that sketch does and does not do. It reads `/api/pending` (the same endpoint `07-approval-loops.md` built) and it sends the owner to `/approval.html` to act. It has no approve call, no publish call, no spend call. It cannot, by construction. That is the guardrail spine holding: the world is a reader and a link, never an actor.

## Step 5: the .glb files are big, load them right

The shipped worlds are real 3D scenes, tens of MB each. If you load one naively over a slow path the owner stares at a blank canvas and thinks it broke. Handle it:

- **Serve the file locally and efficiently.** Serve it off the owner's own machine (the same local process serving the dashboard), not fetched from somewhere remote every load. Local disk to local browser is fast.
- **Load once, then cache.** The first load is slow, that is physics, not a bug. So load the world a single time and keep it. Let the browser cache it (proper cache headers on the local server) so a refresh does not re-download tens of MB. If the owner is on react-three-fiber, its loader caches by default; on plain three.js, do not call `GLTFLoader.load` on every poll (the sketch above loads once, then only the markers update).
- **Show a loading state.** While the world streams in, show a simple "loading the world..." overlay so the first slow load reads as intentional, not frozen. Drop it when the model's load callback fires.
- **Only the markers update on the poll, never the geometry.** The world loads once and sits there. The 4-second poll updates HTML numbers and glows, which is cheap. Never reload the `.glb` to refresh data. Keeping the heavy thing static and the light thing live is the entire performance trick here.

## Step 6: prove it works

Rule 5, same as everywhere: do not tell the owner it is done. Show them, against their real system.

The proof for this build is specific and it is two things:

1. **The world loads with their real board numbers on it.** Open the dashboard, let the world load, and show the owner the actual pending-approval count, the actual agent statuses, today's actual earnings, floating on the actual world they picked. Not placeholder numbers. Their numbers. If three things are genuinely pending, the town hall marker reads three. Cross-check one number against the command-line board so the owner sees the world and the board agree. If they disagree, the world is lying and you fix it before you call it done.
2. **One screenshot or clip that could go on the feed.** Capture the live world, a screenshot or a short screen recording, that is good enough to actually post. This is the second job proving itself: the ops screen produced content. Hand that clip toward `modules/content-engine.md` and the Higgsfield/Zernio flow in `tools/tools.md` the same as any other signature visual. If the capture is not postable, the content half of the thesis is not real yet, so say so plainly and tighten the look until a clip earns the scroll-stop.

Then also prove the guardrail, out loud: click the approval marker in front of the owner and show that it OPENS the real approval flow rather than approving anything. Show them that the world has no button that spends or posts. That is the reassurance that the pretty thing did not quietly gain powers, and it is worth demonstrating, not just asserting.

If the world took a few tries to load cleanly, or one status marker was wired to the wrong dock at first, say so. Do not sell it as flawless.

## The guardrail spine, one more time

Because this is the shiniest thing in the Blueprint, it is the one most likely to tempt someone into letting it act. Do not.

- **It shows; it does not do.** Every risky action (approve, publish, spend, send) still routes through the board and the owner's real click, exactly as `02-architecture.md` and `07-approval-loops.md` define. The world reads that board and links to it. Nothing more.
- **No new powers because the UI got nicer.** Rule 3 holds at full strength. A glowing marker is a status light, not a trigger. The confidence ramp, auto-approve, batch actions: none of that lives here. If the owner wants any of that, it is the deliberate, separate decision in `07-approval-loops.md`, not something a 3D button does quietly.
- **Read-only, owner-only, local.** The dashboard polls state behind HTTP Basic auth on the owner's machine. It does not expose the board to the public and it does not write to it.
- **It came last, on purpose.** This is the payoff for a system that already works (`09-after-first-win.md`, the "make it look good" reward). It is not a reason to build less of the real system. If the machine underneath is not genuinely done and earning, this world is decoration on nothing, and the vanity-trap rule says build the machine first.

Build it when the system has earned it. Then enjoy it, because you get a genuinely great ops screen and a genuinely great piece of content out of one evening's work, and almost nobody else's dashboard does both.
