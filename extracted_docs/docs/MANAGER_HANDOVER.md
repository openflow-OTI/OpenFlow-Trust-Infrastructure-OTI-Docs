# OTI — Manager Handover Document
> Last updated: July 7, 2026 (updated by Manager — Admin Panel fully working, Plan Configs done, next is Task 8)
> **If you are a new Manager reading this: start here. Then read ARCHITECTURE.md, ROADMAP.md, and TASKS.md in that order.**

---

## ⚠️ Important — Where the Code Lives

This Replit workspace is the **Manager's workspace** — used for documentation, prompt writing, and roadmap management only. It does NOT contain OTI source code.

The actual OTI source code lives in Ahmad's GitHub repositories:
- **Backend repo:** https://github.com/openflow-OTI/OpenFlow-Trust-Infrastructure-OTI- — **PRIVATE**
- **Frontend repo:** https://github.com/openflow-OTI/OpenFlow-Trust-Infrastructure-O-T-I-Frontend- — **PUBLIC**
- **Docs repo:** https://github.com/openflow-OTI/OpenFlow-Trust-Infrastructure-OTI-Docs — **Manager's workspace repo**

---

## About Ahmad

Ahmad is CEO of OpenFlow Labs and sole GitHub merge authority. He does NOT want the Manager to touch code or push to GitHub. The Manager writes task prompts for Builders, reviews PRs with GO/NO-GO decisions, and owns the product roadmap.

- Call him Ahmad (not "sir", not "boss")
- He is not a software engineer — explain things clearly but don't be condescending
- He has strong product vision — trust it, build around it
- He uses the Replit multi-account system (see bottom of this file)
- He works primarily from his phone
- One task at a time per Builder — Ahmad's explicit preference
- Every reply to Ahmad must be in a copy box so he can paste it to Builders

---

## The Team

| Role | Status | Notes |
|---|---|---|
| Ahmad (CEO) | Always active | Sole GitHub merge authority |
| Frontend Builder | Active (new account) | Tasks 1, 2, 2B, 7, 7B, 9, 10, 9-FRONTEND-FIX, Plan Configs tab, desktop layout fix, anonymous limit hook fix all done — Task 8 is next |
| Backend Builder | Active | Tasks 3, 4, 5, 6, 7D, 9-BACKEND, stats-500-fix, plan-configs-PATCH-fix all done — queue empty, standing by |
| Development Manager | This account | Writes prompts, reviews PRs, owns roadmap |

---

## Current State of Production (as of July 7, 2026)

**Live and working:**
- ✅ Backend on Railway: `https://workspaceapi-server-production-5c0c.up.railway.app`
- ✅ Frontend on Vercel: `https://otiscore.vercel.app`
- ✅ 15 chains scoring (10 EVM + TON, Solana, Sui, Bitcoin, Tron; BSC/Base/Optimism intentionally 503)
- ✅ API key + daily quota system (dynamic, DB-driven)
- ✅ Score caching
- ✅ Compromised wallet denylist + red banner UI
- ✅ PNG score card sharing
- ✅ UI polish (Task 1 — shipped)
- ✅ Logo component fix (Task 2 — shipped)
- ✅ SVG logo — crisp at all sizes, zero Retina blur (Task 2B — shipped)
- ✅ Admin endpoints secured with `x-admin-secret` header (Task 3 — shipped)
- ✅ History endpoint reads from `chain_scores` DB (Task 4 — shipped)
- ✅ Score response returns weighted signals `{ score, weighted, maxWeight }` per signal (Task 5 — shipped)
- ✅ `updated_at` column on `subscriptions` table (Task 6 — shipped, manual Railway migration run)
- ✅ Bitcoin wallet age pagination fix — paginates through mempool.space history, 40-page cap (Task 7D — shipped)
- ✅ Signal bars show `weighted/maxWeight` — black screen crash resolved (Task 7 — shipped)
- ✅ txCount cap indicator — "1,000+ transactions" shown when metadata.txCount >= 1000 (Task 7B — shipped)
- ✅ Navbar API health status dot — green/red dot connected to `useHealth` hook (Task 10 — shipped)
- ✅ Admin Panel fully working — login, Dashboard, API Keys, Query History, Cache, Plan Configs (Tasks 9, 9-BACKEND — shipped, verified July 7, 2026)
- ✅ Admin Panel desktop layout — full-width sidebar + content, no cramped layout (shipped July 7, 2026)
- ✅ Plan Configs tab — Ahmad can set daily_limit per plan via Admin Panel UI (shipped July 7, 2026)
- ✅ PATCH /api/admin/plan-configs/:id — accepts both UUID and plan name, writes to DB correctly (shipped July 7, 2026)
- ✅ GET /api/config/anonymous-limit — returns live DB value immediately after PATCH (verified July 7, 2026)
- ✅ Homepage anonymous limit display — useAnonymousLimit hook calls correct public endpoint, shows live value (shipped July 7, 2026)

**ADMIN_SECRET status:**
- Ahmad set the `ADMIN_SECRET` in Railway Variables
- Admin auth confirmed working — 401 without header, 200 with correct header
- Admin Panel login at `otiscore.vercel.app/admin` confirmed working

**Known open issues:**
- 🟡 Non-EVM signal accuracy — Bitcoin/Solana/TON/Tron/Sui scored with EVM logic (Task 11C will fix — CRITICAL before distribution)
- 🟡 Satoshi genesis wallet still shows 51 days age despite Task 7D (may be stale cache — flush cache and retest before assuming it's still broken)
- 🟡 BSC/Base/Optimism return 503 — waiting on Ahmad's Etherscan Lite ($49/mo) decision
- 🟡 Results page UX needs professional redesign — Task 8 will fix this (NEXT TASK for Frontend Builder)
- 🟡 Dead code: `recordHistory()` in `score.ts` still writes to `lib/history.ts` — nothing reads it (flagged for future cleanup)
- 🟡 /admin/keys handler has inner api_usage try/catch but outer catch returns 200 [] — functional but logs DB errors silently. Low priority.

---

## Active Decisions — Do Not Reverse Without Ahmad's Approval

| Decision | Reason |
|---|---|
| `scoring.ts` is SACRED — never modify | Core IP, protected algorithm |
| `nixpacks.toml` is SACRED — never touch | Railway build config |
| `vercel.json` is SACRED — never touch | SPA routing |
| CORS is fully open | Intentional — OTI is a public developer API |
| BSC/Base/Optimism are 503 | Waiting for Etherscan Lite decision |
| Admin page is URL-only (no nav link) | Ahmad's decision |
| WOR reports are automated (no admin review) | Ahmad's decision |
| One task at a time per Builder | Ahmad's explicit preference |
| All Manager replies to Ahmad in copy boxes | Ahmad's preference — he pastes them to Builders |

---

## ⚠️ Task Numbering — Read This

- **Task 11C** = Backend Signal Accuracy Audit (Pre-Distribution, Phase 4) — added July 6, 2026
- **Task 12** = Telegram Bot (Phase 5, Distribution) — pre-existing
- **Task 13** = Chrome Extension (Phase 5)
- **Task 14** = Embeddable Widget (Phase 5)
- **Task 15** = Firefox Extension (Phase 5)

The Signal Accuracy Audit was originally labelled "Task 12" by mistake — renamed to Task 11C to avoid collision.

---

## Next Things the Manager Must Do (In Order)

### 1. Send Task 8 to the Frontend Builder — Results Page Redesign
This is the next task. Task 8 prompt is fully written in TASKS.md and FRONTEND_TASKS.md. No dependencies are unmet (Task 7 ✅, Task 2B ✅).

### 2. Frontend queue after Task 8 (in this exact order, one at a time)
- Task 11A — Marketing Homepage + Move scoring to /score
- Task 11B — Whitepaper Page
- Task 7C — Dynamic rate limit display (ON HOLD — unblock once Ahmad confirms the limit is set to a real value via Admin Panel. The frontend hook is already built and working. No code change needed — just confirm it's showing the right number.)

### 3. Backend queue (standing by — nothing assigned)
- Task 11C — Signal Accuracy Audit & Cross-Chain Fix (CRITICAL — must ship before distribution)
- Tasks 12–15 — Distribution channels (Telegram Bot, Chrome Extension, Widget, Firefox Extension)

---

## The Replit Multi-Account System

Ahmad uses multiple Replit free-tier accounts to avoid hitting credit limits:
- **Manager account** — writes prompts, manages roadmap
- **Frontend Builder account** — builds frontend, notifies Manager when work is done
- **Backend Builder account** — builds backend, notifies Manager when work is done

When credits exhaust → Ahmad pushes to GitHub via Replit's Git interface → opens new account → handover → continue.

**The solution (implemented July 5, 2026):**
All context lives in the GitHub repository — not in AI chat memory:
- `docs/MANAGER_HANDOVER.md` — this file
- `docs/ARCHITECTURE.md` — what every piece of the codebase is
- `docs/ROADMAP.md` — all planned features with status
- `docs/TASKS.md` — master task list (Manager adds tasks, Builders update only when Manager says so)
- `docs/BACKEND_TASKS.md` — Backend Builder's dedicated task file
- `docs/FRONTEND_TASKS.md` — Frontend Builder's dedicated task file
- `docs/BUILDER_ONBOARDING.md` — onboarding for each Builder role
- `docs/READING_GUIDE.md` — who reads which file and when

---

## How Builders Get These Docs

Builders do not receive docs via email or file download. The **OTI docs zip file** lives inside the GitHub repos. When a Builder clones the repo, they extract the zip to get all doc files.

**What Ahmad tells each Builder on day one:**
> "Clone the repo. Find the OTI docs zip file in the project root and extract it. Read BUILDER_ONBOARDING.md first, then ARCHITECTURE.md, then open your dedicated task file."

**GitHub visibility:**
- **Frontend repo → PUBLIC** — React code has no secrets
- **Backend repo → PRIVATE** — scoring algorithm IP, admin routes, API key logic

---

## New Manager Startup Procedure (5 minutes)

1. Read `docs/MANAGER_HANDOVER.md` (this file)
2. Read `docs/ARCHITECTURE.md`
3. Read `docs/ROADMAP.md`
4. Read `docs/TASKS.md`
5. Jump straight to "Next Things the Manager Must Do" above — the active bug investigation is CLOSED, Admin Panel is fully working. Next task is Task 8 for the Frontend Builder.

**Rule before ending ANY Manager session:**
1. If a task was confirmed done: tell the Builder to mark it ✅ in their own task file AND in TASKS.md
2. If a new task was added: add to TASKS.md first, then tell Builder to add to their file AND TASKS.md
3. Builders never update files on their own initiative — only when Manager explicitly instructs
4. Update "Current State of Production" and "Next Things" in this file before closing
5. Ahmad pushes everything via Replit's Git interface

---

## ⚠️ Critical Infrastructure Note — Railway Migrations Do NOT Auto-Run

Confirmed July 5, 2026: Railway's deploy pipeline only runs `pnpm install && build && start`. It does **NOT** run `drizzle-kit push`. Every future schema change needs Ahmad to manually run `drizzle-kit push` against the Railway production DATABASE_URL after deploying.

**Optional fix:** Add `drizzle-kit push` to `railway.json`'s `buildCommand` — one-line change, NOT `nixpacks.toml` (sacred). Manager can assign as a micro-task if Ahmad wants.

---

## Critical Context That Must Never Be Lost

1. **Wallet Ownership Registry (WOR)** is Ahmad's flagship feature. Off-chain signing + passkey pre-registration. Users can self-report wallet compromise even when attacker has keys (passkey is the differentiator). No blockchain. No admin review. Fully automated. See ROADMAP.md Phase 4.

2. **`wallet_links` table** was built in anticipation of WOR + portfolio. NOT abandoned — needs WOR to exist first.

3. **`scores` table** is a lightweight fast-lookup cache alongside the full `chain_scores` historical table. NOT redundant — serves a different read pattern.

4. **The 3 unavailable chains** (BSC, Base, Optimism) return 503 intentionally — need Etherscan Lite ($49/mo). Ahmad is aware and has not yet decided.

5. **Task 11C numbering** — the Signal Accuracy Audit is Task 11C, not Task 12. Task 12 is the Telegram Bot (Phase 5). Do not confuse them.

6. **Non-EVM signal accuracy is a known critical bug** — every non-EVM wallet is currently scored with EVM-specific logic. Bitcoin wallets get 0/20 for token holding (no ERC-20 tokens exist on Bitcoin), fabricated smart contract interaction data, and wrong timing subtitles. This is Task 11C and must be fixed before any distribution channel launches.

7. **PATCH /api/admin/plan-configs/:id** accepts both UUID and plan name string (e.g. "anonymous") — dual lookup implemented July 7, 2026. Both work correctly.

8. **useAnonymousLimit hook** in the frontend calls `GET /api/config/anonymous-limit` (public endpoint, no auth). When the endpoint returns null, the hook falls back to displaying 3. When it returns a real number, that number is shown. The hook also invalidates on Plan Configs save so the homepage updates immediately.
