# OTI — Manager Handover Document
> Last updated: July 14, 2026 (session 15 — Phase 2B fully finalised. All architecture decisions locked. BUSINESS_MODEL.md created. TOKENOMICS.md token utility added. DECISIONS.md D17–D22 added. Both Builders idle. Phase 2 (WOR) is the immediate next build. Phase order: 2 → 2B → 3 → 4 → 5.)
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
| Frontend Builder | Active, idle | BF13 ✅, FF22 ✅, FF23 ✅ all closed July 14, 2026. No active task — awaiting next item from Ahmad. |
| Backend Builder | New builder onboarded July 14, 2026 (third Builder — previous hit usage limit). BF13 backend already done by previous Builder. New Builder oriented but has no active task. Do not send anything until Ahmad decides next backend item. |
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

**Everything is clean as of July 14, 2026:**
- All backend fixes BF1–BF37 are ✅ (BF36 = won't-fix by decision, BF12 = open/optional)
- All frontend fixes FF1–FF23 are ✅
- Signal accuracy audit (BF10 + all children BF17–BF37) fully complete with live evidence
- BF13 DB cache system live — system_settings table migrated manually by Ahmad via Railway Console
- Chain count is correctly 15 in all public materials (12 working, 3 gated: BSC/Base/Optimism)
- "Fantom" fully renamed to "Sonic" across all code and public materials

**All fixes complete as of July 14, 2026:**
- BF1–BF37 all ✅ (BF36 = won't-fix by decision)
- FF1–FF23 all ✅
- No open fixes remain. Both Builders idle.

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

### 1. ✅ DONE — All fixes complete (July 14, 2026)
BF1–BF37 and FF1–FF23 all closed. Signal accuracy audit fully done. BF13 DB cache system live. Both Builders idle.

### 2. ✅ DONE — BF12 closed (July 14, 2026)
`railway.json` `buildCommand` updated to append `drizzle-kit push` after build. Plain push used deliberately (not push-force). Confirmed and closed.

### 3. Ahmad's action — create two API keys (Phase 1 task 1D)
Via admin panel at otiscore.vercel.app/admin:
- Internal bot key: high daily limit, for Telegram/Discord bots
- Widget shared key: for the embeddable widget (Task 14)
Once done, Phase 1 is fully closed and Phase 5 (bots + widget + extension) is unblocked.

### 4. After 1D done — start Phase 2 (WOR)
**Confirmed strategic order (Ahmad, July 14, 2026): Phase 2 → Phase 2B → Phase 3 → Phase 4 → Phase 5.**

**Immediate next Manager action:** Design Phase 2 (WOR) fully — every backend endpoint, every frontend flow, every admin dashboard connection — before writing a single Builder prompt. Ahmad wants complete design first, then prompts.

**Phase 2 (WOR) — what needs designing before prompts:**
- `wallet_ownership` DB table schema
- `POST /api/wallet/register` — EIP-191 sign + passkey registration flow
- `POST /api/report/compromised` — wallet sign + passkey → instant 0-score flag
- EIP-191 signature verification (backend)
- Registration UI (wallet connect + sign + passkey set)
- Report UI (wallet connect + passkey entry + submit)
- `wallet_links` table connection to WOR
- Admin dashboard: WOR registry view, compromised wallets list, manual override

**Phase 2B (OTI Verified Badge) — fully designed, architecture locked:**
- BAS attestation on BNB Chain only — no soulbound NFT, no MetaMask Snap
- First 10M attestations free → one-time fee after (admin panel managed)
- OTI auto-rescores every 30 days — no user action needed
- Widget (partner-side) + Extension (user-side) are the display layers
- First 1M attestation users receive OTI token reward before token launch
- Badge visual design + score thresholds confirmed during build, not before
- Full design: ROADMAP.md Phase 2B | All decisions: DECISIONS.md D17–D22

**Phase 3 pillars:**
- Fiat/crypto payment infrastructure (self-serve portal, Stripe, Coinbase Commerce)
- OTI token: create + deploy BSC + ecosystem integration + presale ($10k raise)
- Exchange listing: post-Phase-3, after revenue is live. Ahmad decides timing.

**Privacy policy + T&C:** deferred until full product built (Ahmad, July 14, 2026).

### 5. Whitepaper additions (when ready)
Draft exists at docs/whitepaper-additions-draft.md.
Send to Frontend Builder once Ahmad gives the go-ahead.

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
