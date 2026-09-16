# Decision: kereks_sports tool/API spend cap set to $0/day

2026-09-16: Owner set the tool/API spend cap for kereks_sports at $0/day for
now. This is the hard ceiling on what connected services (Zernio, or
anything metered) can spend automatically - enforced value lives in
`config/spend-caps.json`, read by the dispatcher once it exists (Step 6).

This is a SEPARATE number from the owner's own card-buying budget (up to
$200/day when actively flipping). The card-buying budget is the owner's own
money, spent by the owner directly - the system never buys cards on its own
(rule 3 in CLAUDE.md), so that number is not wired into any code.

$0/day works cleanly here because kereks_sports needs no AI-generated
visuals (real phone photography instead), and Zernio's free tier (2
connected accounts, no card required) exactly covers the 2 platforms this
account uses (Instagram + TikTok). See tools/tools.md, Zernio section.
