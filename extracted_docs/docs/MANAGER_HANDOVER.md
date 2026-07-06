# OTI — Manager Handover Document
> Last updated: July 7, 2026 (Manager account exhausted free credit — handover to new Manager account, mid-investigation of Admin Panel login bug)
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

---

## The Team

| Role | Status | Notes |
|---|---|---|
| Ahmad (CEO) | Always active | Sole GitHub merge authority |
| Frontend Builder | Active | Tasks 1, 2, 2B, 7, 7B, 9, 10 done — Task 7C ON HOLD, nothing currently assigned (do NOT reassign Task 9-FRONTEND-FIX — see "Live Investigation" below, it was a false lead) |
| Backend Builder | Active | Tasks 3, 4, 5, 6, 7D, 9-BACKEND done — Task "Fix GET /admin/stats 500" assigned, awaiting report |
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
- ✅ txCount cap indicator — "1,000+ transactions" shown when metadata.txCount >= 1000, in both UI and score card PNG (Task 7B — shipped, verified live July 7, 2026)
- ✅ Navbar API health status dot — green/red dot connected to `useHealth` hook (Task 10 — shipped, verified live July 7, 2026)

**ADMIN_SECRET status:**
- Original secret set by Backend Builder during Task 3 was never shared — cannot be retrieved
- Ahmad set a NEW `ADMIN_SECRET` in Railway Variables. Both Railway and this Replit's secrets match.
- Admin auth confirmed working — 401 without header, passes with correct header

**Known open issues:**
- ✅ Task 7C-BACKEND is RESOLVED/CLOSED (Ahmad's decision, July 7, 2026): Ahmad will set the real `anonymous` `daily_limit` himself via the Admin Panel once it's fully working. The self-heal fix was corrected as part of Task 9-BACKEND so it never overwrites an existing row's value again. Task 7C (frontend) stays ON HOLD until Ahmad sets a real limit — that's expected, not a bug.
- ✅ Task 9-BACKEND is done and verified live (July 7, 2026) — all admin routes live at `/api/admin/*`.
- ✅ Task 9 (Admin Panel frontend) and Task 9-FRONTEND-FIX are BOTH actually done and correct — see "🔴 LIVE INVESTIGATION" section below for the full story. Do not reassign either.
- 🔴 **ACTIVE BUG — Admin Panel login fails ("Access denied — wrong secret") even with the correct secret.** Root cause fully diagnosed, NOT a frontend or auth bug — see "🔴 LIVE INVESTIGATION IN PROGRESS" section immediately below. Fix is assigned to Backend Builder, awaiting their report.
- 🟡 Non-EVM signal accuracy — Bitcoin/Solana/TON/Tron/Sui scored with EVM logic (Task 11C will fix)
- 🟡 Satoshi genesis wallet still shows 51 days age despite Task 7D (may be stale cache — flush cache and retest before assuming it's still broken)
- 🟡 BSC/Base/Optimism return 503 — waiting on Ahmad's Etherscan Lite ($49/mo) decision
- 🟡 Results page UX needs professional redesign — Task 8 will fix this
- 🟡 Dead code: `recordHistory()` in `score.ts` still writes to `lib/history.ts` — nothing reads it (flagged for future cleanup)

---

## 🔴 LIVE INVESTIGATION IN PROGRESS — Admin Panel Login Bug (read this before doing anything with Task 9/Admin Panel)

**Symptom:** Ahmad enters the correct `ADMIN_SECRET` (confirmed matching Railway's Variables tab exactly, even after typing a brand-new value fresh and redeploying) into `otiscore.vercel.app/admin`, and gets "Access denied — wrong secret."

**Full diagnostic trail (do NOT repeat these steps — they're already done):**
1. Confirmed the Task 9 frontend admin panel code is fully correct: right path (`/api/admin/*`, not `/admin/*` — an earlier false lead, Task 9-FRONTEND-FIX, is NOT needed, the fix was already live), right header name (`x-admin-secret`), no client-side secret comparison, raw value sent unmodified (verified directly in the deployed JS bundle).
2. Confirmed backend `adminAuth` middleware code is correct: reads `process.env.ADMIN_SECRET` fresh per-request (no caching), does a plain `!==` string comparison, no trimming/hashing bugs.
3. Confirmed CORS preflight is clean: `Access-Control-Allow-Headers` includes `x-admin-secret`, origin `*` allowed, all methods allowed.
4. Backend Builder curl-tested the real secret directly against `GET /api/admin/stats` → got **HTTP 500**, not 401. This proves auth passes fine with the correct secret — the crash happens *after* auth, inside the stats handler itself.
5. **Root cause found:** the frontend's login probe (function `Ra` in the built JS) only unlocks the panel if the probe request to `/api/admin/stats` returns exactly HTTP 200. Since that endpoint currently crashes with a 500 (even with the correct secret), the frontend can't distinguish "wrong password" from "server crashed" and shows the same misleading "wrong secret" message either way.

**Conclusion:** This is a single bug, not three. The Admin Panel login will work immediately once `GET /api/admin/stats` stops returning 500. No frontend fix is needed — the frontend behavior (treating non-200 as failure) is arguably still worth hardening later to show a distinct "server error" message, but that's a nice-to-have, not required to unblock Ahmad.

**Current status:** Assigned to Backend Builder as a CRITICAL task in `BACKEND_TASKS.md` ("Fix GET /admin/stats 500 error"). Awaiting their report.

**What the next Manager must do:**
1. Wait for/check in on the Backend Builder's report on the `/admin/stats` 500 fix.
2. When they report done, verify independently — do NOT trust self-report:
   - `curl` `GET /api/admin/stats` with the real secret (ask Ahmad to relay it directly to you in chat if needed, or better, have the Backend Builder confirm 200 themselves and you verify via the live Admin Panel UI instead, so the secret isn't repeated across multiple chats unnecessarily) and confirm HTTP 200 with valid JSON.
   - Take a screenshot of `otiscore.vercel.app/admin`, ask Ahmad to try logging in, or verify via the JS bundle if the stats shape matches.
3. Once confirmed, mark the task done in `BACKEND_TASKS.md` and `TASKS.md`, and mark Task 9 (Admin Panel, frontend) fully ✅ done at last — it was actually complete this whole time, just blocked by this backend bug.
4. Then resume the normal queue (see "Next Things To Do" below).

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

---

## ⚠️ Task Numbering — Read This

There is a numbering note to be aware of:
- **Task 11C** = Backend Signal Accuracy Audit (Pre-Distribution, Phase 4) — added July 6, 2026
- **Task 12** = Telegram Bot (Phase 5, Distribution) — pre-existing
- **Task 13** = Chrome Extension (Phase 5)
- **Task 14** = Embeddable Widget (Phase 5)
- **Task 15** = Firefox Extension (Phase 5)

The Signal Accuracy Audit was originally labelled "Task 12" by mistake — renamed to Task 11C to avoid collision. The Backend Builder has been notified and has added it to BACKEND_TASKS.md as Task 11C.

---

## Next Things the Manager Must Do (In Order)

### 0. FIRST — resolve the Admin Panel login bug (see section above)
Do not start anything else until this is closed out. Backend Builder is already working the fix.

### 1. Once Admin Panel is confirmed fully working, send Task 8 to the Frontend Builder
Task 7C (frontend) and Task 7C-BACKEND (backend) remain paused at Ahmad's explicit request until he sets a real `daily_limit` himself via the (now-working) Admin Panel. Do not resume either without Ahmad's go-ahead.

### 2. Frontend queue after Task 8 (in this exact order, one at a time)
- Task 11A — Marketing Homepage + Move scoring to /score
- Task 11B — Whitepaper Page
- (Task 7C resumes whenever Ahmad gives the go-ahead — see TASKS.md, the design question is resolved, just waiting on Ahmad to actually set the limit)

### 3. Backend queue after all Phase 4 Frontend tasks complete
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
5. Check GitHub for open PRs awaiting review
6. Jump straight to "🔴 LIVE INVESTIGATION IN PROGRESS" and "Next Things the Manager Must Do" above — there is an active bug being fixed by the Backend Builder, check in on it first

**Rule before ending ANY Manager session:**
1. If a task was confirmed done: tell the Builder to mark it ✅ in their own task file AND in TASKS.md
2. If a new task was added: add to TASKS.md first, then tell Builder to add to their own file AND TASKS.md
3. Builders never update files on their own initiative — only when Manager explicitly instructs
4. Update "Current State of Production" and "Next 3 Things" in this file before closing
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

4. **`useHealth` hook** was designed for the navbar API status dot. NOT unused — just needs Task 10 to connect a component to it.

5. **The 3 unavailable chains** (BSC, Base, Optimism) return 503 intentionally — need Etherscan Lite ($49/mo). Ahmad is aware and has not yet decided.

6. **Task 11C numbering** — the Signal Accuracy Audit is Task 11C, not Task 12. Task 12 is the Telegram Bot (Phase 5). Do not confuse them.

7. **Non-EVM signal accuracy is a known critical bug** — every non-EVM wallet is currently scored with EVM-specific logic. Bitcoin wallets get 0/20 for token holding (no ERC-20 tokens exist on Bitcoin), fabricated smart contract interaction data, and wrong timing subtitles. This is Task 11C and must be fixed before any distribution channel launches.
