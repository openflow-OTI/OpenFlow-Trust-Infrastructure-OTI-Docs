# OTI — Manager Handover Document
> Last updated: July 13, 2026 (session 11 — handover due to Manager credit exhaustion, mid-batch. Closed a large consolidated backend fix batch: BF18, BF24, BF25, BF27, BF29, BF30, BF32, BF34, BF35, BF37 all confirmed fixed with real live evidence. BF33 (contract diversity, 4 chains) closed. BF36 resolved by product decision (score all address types, no gatekeeping). Two items sent back to Backend Builder and awaiting response: BF26 (needs a real token-only test wallet, not code inspection) and BF21 (fix looks right but was verified against a cached response, needs a fresh non-cached re-check). DO NOT mark BF26/BF21 closed until the Builder's next reply comes back with real evidence — see "Next Things" below.)
> **If you are a new Manager reading this: start here. Then read ARCHITECTURE.md, ROADMAP.md, TASKS.md, FIXES.md, and DECISIONS.md in that order.**
> **⚠️ Read `TOKENOMICS.md` before touching anything token-related — price/liquidity sections were deliberately removed at Ahmad's request. Do not add them back or estimate them yourself.**
> **⚠️ D16 (evidence rule, non-negotiable): no signal value or test result may ever be estimated or guessed — only real on-chain data or a disclosed hard cap. This applies retroactively too: if a Builder reports something "verified" via code-reading only (no live wallet, no real API response), it is NOT verified. Send it back. This is exactly what happened with BF26 this session and why it's still open.

---

## ⚠️ Important — Where the Code Lives

This Replit workspace is the **Manager's workspace** — used for documentation, prompt writing, and roadmap management only. It does NOT contain OTI source code.

The actual OTI source code lives in Ahmad's GitHub repositories:
- **Backend repo:** https://github.com/openflow-OTI/OpenFlow-Trust-Infrastructure-OTI- — **PRIVATE**
- **Frontend repo:** https://github.com/openflow-OTI/OpenFlow-Trust-Infrastructure-O-T-I-Frontend- — **PUBLIC**
- **Docs repo:** https://github.com/openflow-OTI/OpenFlow-Trust-Infrastructure-OTI-Docs — **Manager's workspace repo**

---

## About Ahmad

Ahmad is CEO of OpenFlow Labs and sole GitHub merge authority. He does NOT want the Manager to touch code or push to GitHub. The Manager writes task prompts for Builders, reviews PRs with GO/NO-GO decisions, and owns the product roadmap and documentation.

- Call him Ahmad (not "sir", not "boss")
- He is not a software engineer — explain things clearly but don't be condescending
- He has strong product vision — trust it, build around it
- He uses the Replit multi-account system (see bottom of this file)
- He works primarily from his phone
- One task at a time per Builder — Ahmad's explicit, non-negotiable preference
- Every reply to Ahmad must be in a copy box so he can paste it to Builders
- Bug fixes / cleanup / polish are NOT tasks — they go in `FIXES.md`, never folded into an existing task or listed as a numbered Task. This was an explicit correction from Ahmad on July 10, 2026 after the Manager had previously folded a fix into an existing task's number.

---

## The Team

| Role | Status | Notes |
|---|---|---|
| Ahmad (CEO) | Always active | Sole GitHub merge authority |
| Frontend Builder | Active, idle | All build tasks (8, 8B, 9, 11A, 11B, 11) done and verified live. Awaiting Ahmad's priority call on `FIXES.md` FF17 (AI-native tell cleanup) before receiving a new item. |
| Backend Builder | Active — **this is the SECOND Backend Builder** (first hit its usage limit mid-batch and was replaced; onboarded fresh, zero pre-existing secrets, re-onboarded via full BUILDER_ONBOARDING.md + D16 walkthrough). Currently has exactly ONE active item outstanding: a combined follow-up covering BF26 (real test wallet needed) and BF21 (fresh non-cached re-check needed). Do not send anything else until both come back with real evidence. | |
| Development Manager | Previous account exhausted credits mid-session, July 13, 2026 — this handover exists because of that, not because work was paused deliberately. | Writes prompts, reviews PRs, owns roadmap and docs |

---

## Current State of Production

**Live and working:**
- Backend on Railway: `https://workspaceapi-server-production-5c0c.up.railway.app`
- Frontend on Vercel: `https://otiscore.vercel.app` (marketing homepage at `/`, scoring tool at `/score`, whitepaper at `/whitepaper`, admin panel at `/admin`)
- Developer docs live at `https://otiscore.vercel.app/docs/` (standalone deployment: `https://oti-docs.vercel.app/`)
- 15 chains scoring (10 EVM + TON, Solana, Sui, Bitcoin, Tron; BSC/Base/Optimism intentionally return 503 pending Ahmad's Etherscan Lite decision)
- API key + daily quota system (dynamic, DB-driven, enforced with real 429s)
- Score caching, compromised-wallet denylist with red banner UI
- Admin Panel fully working — Dashboard, API Keys (create/list/edit/delete + reveal-once modal), Query History, Cache, Plan Configs
- Full black/mint visual redesign live across scoring app, homepage, and whitepaper (see the locked color system in `FRONTEND_TASKS.md`)
- BF10 (non-EVM signal accuracy audit) is now effectively DONE — all its child items (BF17/BF18/BF19/BF20/BF28/BF33 and this session's BF24/BF25/BF27/BF29/BF30/BF32/BF34/BF35/BF37) are closed with live evidence. Only BF26 and BF21 remain (see below) before the whole signal-accuracy effort can be signed off as complete.

**What's still open right now — only 2 backend items, both awaiting the Builder's reply:**
- **BF26** (EVM wallet age = 0 for token-only wallets) — sent back to Backend Builder because their "verification" was code inspection only, no real wallet tested. Waiting on them to find a real token-only address (e.g. an airdrop-only EOA) and re-test live.
- **BF21** (TON Jetton holdings via tonapi.io) — fix looks correct but was verified against a *cached* Railway response, breaking from the fresh-test standard used everywhere else in the batch. Waiting on a fresh, non-cached re-check.
- A combined follow-up prompt covering both was already sent to the Backend Builder this session (see chat log for exact wording) — **do not resend or duplicate it**; just wait for their reply and grade it against D16.

**What's still open beyond those two** — full detail in `FIXES.md`:
- BF11 — re-verify "Try It Live" docs widget hits the real Railway backend (never assigned, still open)
- BF23 — Scroll/Sepolia/Holesky listed as supported but not implemented (real chain count = 12, not 15) — not yet assigned
- FF17 — AI-native tell cleanup across homepage/docs/whitepaper (open, high priority, not yet sent) — Frontend Builder is idle and waiting for Ahmad's priority call on this
- Doc sweep: "Fantom" → "Sonic" rename (BF22) left stale mentions in README.md/CHANGELOG.md/ARCHITECTURE.md/TASKS.md — non-functional, held pending Manager confirmation
- Whitepaper + FAQ update — draft ready at docs/whitepaper-additions-draft.md; NOT sent to Frontend Builder until backend is fully verified correct (now much closer — just BF26/BF21 away)

**⚠️ Chain count across ALL public materials is wrong:** docs, whitepaper, and UI all say 15 chains. Real working count is 12. Scroll, Sepolia, Holesky are not implemented. Fantom is querying the wrong network. This must be corrected before any public-facing update ships.

**ADMIN_SECRET status:** Ahmad set `ADMIN_SECRET` in Railway Variables. Admin Panel login at `otiscore.vercel.app/admin` confirmed working.

**⚠️ OTI COLOR SYSTEM — LOCKED July 7, 2026.** Full token table lives in `FRONTEND_TASKS.md`. Do not revert to pure black `#000000`.

**Critical infrastructure note — Railway `subscriptions` table real schema:** the Drizzle schema in the repo does NOT match Railway's actual DB. Real columns: `id, api_key, plan, owner_address, created_at, expires_at, updated_at`. **NO `status` column, NO `email` column.** All backend handlers use raw SQL on this table — never use Drizzle ORM selects here, they crash with "column does not exist." (Full history of this bug and its fixes: `FIXES.md` BF4/BF9.)

---

## What Changed This Session (Documentation Reorganization)

Ahmad's directive: bug fixes/cleanup/polish are not tasks. Create `FIXES.md`, move every fix-type item there out of `TASKS.md`/`BACKEND_TASKS.md`/`FRONTEND_TASKS.md`, number small fixes first, split by Builder. Then rewrite everything fresh rather than edit incrementally.

**What now lives where:**
- `TASKS.md`, `BACKEND_TASKS.md`, `FRONTEND_TASKS.md` — only genuine new-build tasks (new pages, new systems, new features)
- `FIXES.md` — every bug fix, correctness fix, security fix, hardening pass, and polish/cleanup item, split into a Backend list and a Frontend list, each numbered smallest/quickest first
- `ROADMAP.md` — unchanged in structure, updated only to reference `FIXES.md` where relevant
- `READING_GUIDE.md` — added a row/section for `FIXES.md`
- `PENDING_BACKEND_PROMPT.md` — deleted. It was a stale, superseded draft for what became Task 11C/BF10 (already correctly sent and in progress under the right identity)

**Fix vs. task rule going forward:** a task builds something that didn't exist before; a fix repairs, corrects, hardens, or polishes something that already exists. When in doubt, ask: "if this were never done, would the product be missing a capability (task) or just imperfect (fix)?"

**The former Task 11C (Signal Accuracy Audit) is now `FIXES.md` BF10** — it corrects existing scoring behavior for non-EVM chains rather than building a new feature. Its status, priority, and "in progress" state are unchanged — only its home file changed. The former Task 11E (AI-native tell audit) is now `FIXES.md` FF17, same situation.

---

## Active Decisions — Do Not Reverse Without Ahmad's Approval

| Decision | Reason |
|---|---|
| `scoring.ts` is SACRED — never modify | Core IP, protected algorithm |
| `nixpacks.toml` is SACRED — never touch | Railway build config |
| `vercel.json` is SACRED — never touch (one narrow exception already used for the docs proxy rewrite) | SPA routing |
| CORS is fully open | Intentional — OTI is a public developer API |
| BSC/Base/Optimism are 503 | Waiting for Etherscan Lite decision |
| Admin page is URL-only (no nav link) | Ahmad's decision |
| WOR reports are automated (no admin review) | Ahmad's decision |
| One task at a time per Builder | Ahmad's explicit, hard preference |
| All Manager replies to Ahmad in copy boxes | Ahmad's preference — he pastes them to Builders |
| Fixes are never folded into tasks or given task numbers — they live in `FIXES.md` | Ahmad's explicit correction, July 10, 2026 |
| Price/liquidity design intentionally absent from `TOKENOMICS.md` | Deferred by Ahmad, decide later, do not reconstruct |
| DECISIONS.md is Manager-write, Builder-read — Builders never update it | Ahmad's direction, July 12, 2026 — before treating any behavior as a bug, check DECISIONS.md first |
| Whitepaper additions held until backend fully verified | Content draft exists; publishing wrong technical claims would be worse than publishing nothing |
| Contract addresses are NOT rejected from scoring — scored like any other address (BF36) | OTI never verifies identity for any address; "no person behind a contract" isn't a meaningfully different bar than a normal wallet (multisigs, smart accounts, and all TON wallets are contracts). The bot/scam-risk concern this was meant to address is deferred to the future bot/suspicious-wallet behavioral detection phase (ROADMAP.md Phase 4), not solved here. |
| D16 (evidence standard) applies retroactively, including to "verified" reports | A Builder's own report claiming something is verified is not sufficient if it turns out to be code-inspection-only or based on cached/stale data — re-check the actual evidence given, not just the claim of verification, every time. |

---

## Next Things the Manager Must Do (In Order)

### 1. Grade the Backend Builder's reply on BF26 + BF21 (this is what you're waiting for right now)
A combined follow-up prompt is already sent, asking for: (a) BF26 — a real token-only test wallet (empty `txlist`, real `tokentx`) with actual Railway output showing non-zero wallet age; (b) BF21 — a fresh, non-cached re-verification of the TON Jetton fix. When the reply comes in:
- Apply D16 strictly: code inspection, a description of what the code "should do," or a cached response does NOT count as evidence. Only a real address + real raw API response + real Railway response counts.
- If BF26 still has no real wallet, send it back again — do not accept "couldn't find one, but logically it should work."
- If BF21 is still cached, send it back again for a fresh call.
- Once both are genuinely verified, update `FIXES.md`: flip BF26 to ✅ with the evidence, flip BF21's header from "FIX REPORTED — PENDING FRESH VERIFICATION" to ✅, and add the fresh evidence line.
- Only after BOTH close should you queue this Backend Builder's next item — remember the hard one-task-at-a-time rule.

### 2. Once BF26 + BF21 close — the whole BF10 signal-accuracy audit is done
This is a real milestone: every child item (BF17–BF37, minus the won't-fix BF36 and the still-unassigned BF11/BF23) will be closed. Tell Ahmad explicitly when this happens — it's worth flagging as "signal accuracy audit fully complete."

### 3. Then pick the next backend item — likely candidates, in rough priority
- **BF23** — Scroll/Sepolia/Holesky removal from docs/UI (chain count 15→12 everywhere) — confirmed bug, no ambiguity, ready to scope now if Ahmad wants it before the milestone above closes.
- **BF11** — re-verify "Try It Live" docs widget hits real Railway backend — small, never assigned.
- Doc sweep — stale "Fantom" mentions in README/CHANGELOG/ARCHITECTURE/TASKS (non-functional, cosmetic, low priority).

### 4. Once all backend fixes verified live — finalise whitepaper additions
Whitepaper additions draft is at docs/whitepaper-additions-draft.md. Update the signal applicability table and chain count once all fixes are confirmed. Then send to Frontend Builder as their next task.

### 5. Get Ahmad's priority call on FF17 (Frontend Builder is idle)
AI-native tell cleanup across homepage, docs, and whitepaper — scoped, not yet sent. Frontend Builder is waiting. This can run in parallel with backend fix work since they are fully independent.

### Lesson learned (scope creep during Task 11 — historical, resolved)
Ahmad personally authorized a previous Frontend Builder to keep working past the original Task 11 spec, leading to 8 self-directed rounds and some unplanned production breakage (docs 404, "Try It Live" pointed at a non-existent custom domain). Some of the extra work was valuable (a privacy audit caught a real leak), but it shipped without a mid-task check-in. Fully remediated; Task 11 is now live. Going forward: even with a blanket go-ahead, a Builder should check in with the Manager before migrating live API URLs or touching deployment config — those changes have production blast radius beyond the original task.

### Lesson learned (Vercel/Docusaurus docs proxy debugging)
Full detail in the Manager's persistent memory (`vercel-docs-proxy.md` topic). Summary: (1) Vercel rewrites treat `/docs` and `/docs/` as distinct routes — a wildcard `/docs/:path*` rule does not cover the bare trailing-slash case, always test all three forms via curl; (2) Docusaurus's `baseUrl` only changes link prefixes/router basename, not the physical build output folder structure — check the actual local `build/` folder before assuming a hosting platform's directory settings are misconfigured.

### Lesson learned (verifying "done" reports)
A live-site check of Task 11A initially looked broken (`/` showed the old scoring form) — turned out to be a stale/cached screenshot, not a real bug. A cache-busted check (`?t=<random>` + checking `x-vercel-cache`/`age` headers) confirmed it was genuinely live and correct. Always re-verify with a cache-busted check before reporting a problem to Ahmad — false alarms cost trust too.

### Lesson learned (D16 evidence standard — this session, July 13, 2026)
When a Builder reports a fix as "verified," that word alone means nothing — check what evidence backs it. This session: BF26 was sent back because "verified" turned out to mean reading the code, not testing a real wallet; BF21 was sent back because "verified" turned out to mean a cached Railway response, breaking from the fresh-test standard the rest of the batch used. Both looked fine on first read of the report text — the gap only showed up when the Manager asked "what specific wallet, what specific raw response?" Always ask that question before accepting a close.

### Lesson learned (Backend Builder handoffs mid-batch)
Twice this project a Backend Builder has hit its usage/credit limit mid-task and had to be replaced. Each time: (1) work-in-progress must be confirmed pushed to a branch/commit before the old Builder's environment is abandoned; (2) the new Builder starts with zero secrets/API keys — re-add all of them; (3) give the new Builder a full onboarding pass (project overview, sacred files, D16 with a concrete example of a past fabrication) before resuming the task, don't assume continuity of understanding just because the code is the same.

---

## The Replit Multi-Account System

Ahmad uses multiple Replit free-tier accounts to avoid hitting credit limits:
- **Manager account** — writes prompts, manages roadmap and docs
- **Frontend Builder account** — builds frontend, notifies Manager when work is done
- **Backend Builder account** — builds backend, notifies Manager when work is done

When credits exhaust → Ahmad pushes to GitHub via Replit's Git interface → opens new account → handover → continue.

**All context lives in the GitHub repository, not in AI chat memory:**
- `docs/MANAGER_HANDOVER.md` — this file
- `docs/ARCHITECTURE.md` — what every piece of the codebase is
- `docs/ROADMAP.md` — all planned features with status
- `docs/TASKS.md` — master list of genuine new-build tasks
- `docs/FIXES.md` — master list of bug fixes, corrections, hardening, and cleanup, split by Builder
- `docs/DECISIONS.md` — why things exist the way they do; confirmed decisions, technical limitations, pending answers. Manager-write, Builder-read. Check before treating anything as a bug.
- `docs/BACKEND_TASKS.md` / `docs/FRONTEND_TASKS.md` — each Builder's dedicated task file
- `docs/BUILDER_ONBOARDING.md` — onboarding for each Builder role
- `docs/READING_GUIDE.md` — who reads which file and when

---

## How Builders Get These Docs

Builders do not receive docs via email or file download. The **OTI docs zip file** lives inside the GitHub repos. When a Builder clones the repo, they extract the zip to get all doc files.

**What Ahmad tells each Builder on day one:**
> "Clone the repo. Find the OTI docs zip file in the project root and extract it. Read BUILDER_ONBOARDING.md first, then ARCHITECTURE.md, then open your dedicated task file — and check FIXES.md for anything assigned to you there too."

**GitHub visibility:**
- **Frontend repo → PUBLIC** — React code has no secrets
- **Backend repo → PRIVATE** — scoring algorithm IP, admin routes, API key logic

---

## New Manager Startup Procedure

1. Read `docs/MANAGER_HANDOVER.md` (this file) — full picture
2. Read `docs/ARCHITECTURE.md` — system structure
3. Read `docs/ROADMAP.md` — strategic context
4. Read `docs/TASKS.md` — every genuine build task, current queue
5. Read `docs/FIXES.md` — every open/in-progress fix, per Builder
6. Read `docs/DECISIONS.md` — why things exist the way they do; check before treating any behavior as a bug
7. Jump straight to "Next Things the Manager Must Do" above and start from step 1

**Rule before ending ANY Manager session:**
1. If a task or fix was confirmed done: tell the Builder to mark it ✅ in THEIR OWN copy of the relevant file AND their own `TASKS.md`/`FIXES.md` — the Manager's copy is physically separate from each Builder's copy (different account, different repo checkout). Updating the Manager's copy never updates a Builder's copy. Send this instruction explicitly, every time.
2. If a new task or fix was added: add it to `TASKS.md` or `FIXES.md` first, then tell the Builder to add it to their own file too (same separate-copies caveat).
3. Builders never update files on their own initiative — only when the Manager explicitly instructs.
4. Update "Current State of Production" and "Next Things" in this file before closing.
5. Ahmad pushes everything via Replit's Git interface.
6. One task at a time per Builder — Ahmad's hard rule, never break it.
7. Fixes never get folded into tasks or given task numbers — they go in `FIXES.md`.

---

## ⚠️ Critical Infrastructure Note — Railway Migrations Do NOT Auto-Run

Confirmed July 5, 2026: Railway's deploy pipeline only runs `pnpm install && build && start`. It does **NOT** run `drizzle-kit push`. Every future schema change needs Ahmad to manually run `drizzle-kit push` against the Railway production `DATABASE_URL` after deploying. Optional one-line fix tracked as `FIXES.md` BF12 (add `drizzle-kit push` to `railway.json`'s `buildCommand` — not `nixpacks.toml`, which stays sacred).

---

## Critical Context That Must Never Be Lost

1. **Wallet Ownership Registry (WOR)** is Ahmad's flagship feature — off-chain signing + passkey pre-registration, no blockchain, no admin review, fully automated. See `ROADMAP.md` Phase 2.
2. **`wallet_links` table** was built in anticipation of WOR + portfolio features. Not abandoned — needs WOR to exist first.
3. **`scores` table** is a lightweight fast-lookup cache alongside the full `chain_scores` historical table. Not redundant — serves a different read pattern.
4. **The 3 unavailable chains** (BSC, Base, Optimism) return 503 intentionally, pending Ahmad's Etherscan Lite ($49/mo) decision.
5. **Task numbering:** Task 12 = Telegram Bot, 13 = Discord Bot, 14 = Widget, 15 = Firefox Extension (all Phase 5). The former "Task 11C" and "Task 11E" are now `FIXES.md` BF10 and FF17 respectively — they are fixes, not numbered tasks, and this renumbering-out-of-TASKS.md is intentional, not a mistake to undo.
6. **Non-EVM signal accuracy is a known critical bug** (`FIXES.md` BF10) — every non-EVM wallet is currently scored with EVM-specific logic. Must be fixed before any distribution channel launches.
7. **`PATCH /api/admin/plan-configs/:id`** accepts both a UUID and a plan-name string (e.g. "anonymous") — dual lookup, both work correctly.
8. **`useAnonymousLimit` hook** calls the public `GET /api/config/anonymous-limit` endpoint (no auth). Falls back to displaying 3 only on a genuine fetch failure; a real `null` from the server means "Unlimited," and is displayed as such. Invalidates on every Plan Configs save so the homepage updates immediately.
