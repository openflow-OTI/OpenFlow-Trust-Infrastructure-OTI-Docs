# OTI — Manager Handover Document
> Last updated: July 7, 2026 (end-of-session update — API Keys fully working, Task 9C in progress with new Backend Builder, Task 8 on hold until 9C done)
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
| Frontend Builder | Active (new account) | All tasks through API Keys UI resilience fix done — Task 8 (Results Page Redesign) is next, ON HOLD until Task 9C done |
| Backend Builder | New account (July 7, 2026) | Previous account hit credit limit mid-fix. Task 9C in progress — code review done, awaiting live 429 test |
| Development Manager | This account | Writes prompts, reviews PRs, owns roadmap |

---

## Current State of Production (as of July 7, 2026 — end of session)

**Live and working:**
- ✅ Backend on Railway: `https://workspaceapi-server-production-5c0c.up.railway.app`
- ✅ Frontend on Vercel: `https://otiscore.vercel.app`
- ✅ 15 chains scoring (10 EVM + TON, Solana, Sui, Bitcoin, Tron; BSC/Base/Optimism intentionally 503)
- ✅ API key + daily quota system (dynamic, DB-driven)
- ✅ Score caching
- ✅ Compromised wallet denylist + red banner UI
- ✅ PNG score card sharing
- ✅ UI polish, Logo fix, SVG logo (Tasks 1, 2, 2B)
- ✅ Admin endpoints secured with `x-admin-secret` header (Task 3)
- ✅ History endpoint reads from `chain_scores` DB (Task 4)
- ✅ Score response returns weighted signals `{ score, weighted, maxWeight }` per signal (Task 5)
- ✅ `updated_at` column on `subscriptions` table (Task 6 — manual Railway migration run)
- ✅ Bitcoin wallet age pagination fix — 40-page cap (Task 7D)
- ✅ Signal bars show `weighted/maxWeight` (Task 7)
- ✅ txCount cap indicator — "1,000+ transactions" (Task 7B)
- ✅ Homepage shows live anonymous daily_limit from DB — confirmed showing "20 per day" (Task 7C)
- ✅ Navbar API health status dot (Task 10)
- ✅ Admin Panel fully working — login, Dashboard, API Keys (create/list/edit/delete), Query History, Cache, Plan Configs (Tasks 9, 9-BACKEND)
- ✅ Admin Panel desktop layout — full-width, no cramped layout
- ✅ API Keys create/list/edit/delete confirmed working — Ahmad created enterprise, pro, free keys at 14:15 July 7, 2026
- ✅ PATCH /api/admin/plan-configs/:id — accepts UUID and plan name
- ✅ GET /api/config/anonymous-limit — returns live DB value

**ADMIN_SECRET status:**
- Ahmad set `ADMIN_SECRET` in Railway Variables
- Admin Panel login at `otiscore.vercel.app/admin` confirmed working

**Critical infrastructure note — Railway subscriptions table real schema:**
The Drizzle schema in the repo does NOT match Railway's actual DB. Real columns confirmed July 7, 2026:
```
id            uuid       NOT NULL
api_key       text       NOT NULL
plan          text       NOT NULL
owner_address text       NOT NULL
created_at    timestampz NOT NULL
expires_at    timestampz nullable
updated_at    timestampz nullable
```
**There is NO `status` column and NO `email` column.** All backend handlers use raw SQL or column-name fallback mappers to accommodate this. This mismatch is the root cause of all admin/keys bugs fixed today.

**Known open issues:**
- 🔴 Task 9C in progress — Backend Builder doing live 429 enforcement test. Free plan key available (ending ...f648). See Task 9C below.
- 🟡 Non-EVM signal accuracy — Bitcoin/Solana/TON/Tron/Sui scored with EVM logic (Task 11C — CRITICAL before distribution)
- 🟡 Satoshi genesis wallet still shows 51 days age despite Task 7D (may be stale cache — flush and retest)
- 🟡 BSC/Base/Optimism return 503 — waiting on Ahmad's Etherscan Lite ($49/mo) decision
- 🟡 Results page UX needs professional redesign — Task 8 (Frontend Builder, ON HOLD until Task 9C done)
- 🟡 Dead code: `recordHistory()` in `score.ts` writes to `lib/history.ts` — nothing reads it (future cleanup)
- 🟡 Anonymous row in plan_configs: if missing, API treats anonymous users as unlimited — confirm the row exists in production

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

### 1. Finish Task 9C — Backend: Plan Limit Enforcement Live Test (IN PROGRESS)
New Backend Builder has completed the code review step. Awaiting live 429 test.
- Give Backend Builder a free-plan API key (Ahmad has one ending in ...f648 — get full key from Admin Panel → API Keys)
- Tell him to: set free plan daily_limit to 2 via Admin Panel → make 3 requests with that key → confirm 3rd = 429 → restore limit → report per-plan results
- Once confirmed, mark Task 9C done in TASKS.md and BACKEND_TASKS.md
- Tell Backend Builder to also mark it in both files

### 2. Send Task 8 to the Frontend Builder — Results Page Redesign
Task 8 is ON HOLD until Task 9C is confirmed done. Once 9C is done:
- Task 8 full prompt is in FRONTEND_TASKS.md (next item in queue)
- No dependencies unmet (Task 7 ✅, Task 2B ✅)
- Tell Frontend Builder to start Task 8

### 3. Frontend queue after Task 8 (in this exact order, one at a time)
- Task 11A — Marketing Homepage + Move scoring to /score
- Task 11B — Whitepaper Page

### 4. Backend queue after Task 9C (in this exact order, one at a time)
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

1. Read `docs/MANAGER_HANDOVER.md` (this file) — full picture
2. Read `docs/ARCHITECTURE.md` — system structure
3. Read `docs/ROADMAP.md` — strategic context
4. Read `docs/TASKS.md` — every task ever, full history
5. Jump straight to "Next Things the Manager Must Do" above and start from step 1

**What is happening right now:**
- Task 9C is IN PROGRESS with the new Backend Builder. Code review step is done. Awaiting live 429 enforcement test. Give the Backend Builder a free API key (get it from Admin Panel → API Keys — the one ending in ...f648) and tell him to run the test.
- Task 8 (Frontend Results Page Redesign) is ON HOLD. Do not send it to the Frontend Builder until Task 9C is confirmed done and marked.
- Both Builders are waiting. Be precise — one task at a time per Builder, no exceptions.

**Rule before ending ANY Manager session:**
1. If a task was confirmed done: tell the Builder to mark it ✅ in their own task file AND in TASKS.md
2. If a new task was added: add to TASKS.md first, then tell Builder to add to their file AND TASKS.md
3. Builders never update files on their own initiative — only when Manager explicitly instructs
4. Update "Current State of Production" and "Next Things" in this file before closing
5. Ahmad pushes everything via Replit's Git interface
6. One task at a time per Builder — Ahmad's hard rule, never break it

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
