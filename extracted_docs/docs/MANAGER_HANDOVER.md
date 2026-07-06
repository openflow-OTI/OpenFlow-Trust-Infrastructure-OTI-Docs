# OTI — Manager Handover Document
> Last updated: July 6, 2026 (updated by Manager)
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

---

## The Team

| Role | Status | Notes |
|---|---|---|
| Ahmad (CEO) | Always active | Sole GitHub merge authority |
| Frontend Builder | Active | Builds frontend, notifies Manager when work is done |
| Backend Builder | Active | Tasks 3, 4, 5, (6 code done) complete — see Task 6 blocker below |
| Development Manager | This account | Writes prompts, reviews PRs, owns roadmap |

---

## Current State of Production (as of July 6, 2026)

**Live and working:**
- ✅ Backend on Railway: `https://workspaceapi-server-production-5c0c.up.railway.app`
- ✅ Frontend on Vercel: `https://otiscore.vercel.app`
- ✅ 12 chains scoring (10 EVM + TON, Solana, Sui, Bitcoin, Tron = 15 total; 3 intentionally 503)
- ✅ API key + daily quota system (dynamic, DB-driven)
- ✅ Score caching
- ✅ Compromised wallet denylist + red banner UI
- ✅ PNG score card sharing
- ✅ Logo component fix (Task 2 — shipped)
- ✅ SVG logo live — crisp at all sizes, zero blur on Retina (Task 2B — shipped)
- ✅ Signal bars show weighted values `weighted/maxWeight` — black screen crash resolved (Task 7 — shipped)
- ✅ UI polish (Task 1 — shipped)
- ✅ Admin endpoints secured with `x-admin-secret` header (Task 3 — shipped)
- ✅ History endpoint now reads from `chain_scores` DB — persists across restarts (Task 4 — shipped)
- ✅ Score response returns weighted signals `{ score, weighted, maxWeight }` per signal (Task 5 — shipped)
- ✅ `updated_at` column added to `subscriptions` table — `GET /api/admin/keys` verified working (Task 6 — shipped)
- ✅ Bitcoin wallet age pagination fix — `walletAgedays` now returns correct values (Task 7D — shipped, verified on Railway)

**ADMIN_SECRET status:**
- The original ADMIN_SECRET set by the Backend Builder during Task 3 was never shared with Ahmad and cannot be retrieved.
- Ahmad has set a NEW `ADMIN_SECRET` in Railway Variables and in this Replit's secrets. Both now match.
- Admin auth is confirmed working — 401 without header, passes with correct header.

**Other open issues:**
- 🟡 Results page UX needs professional redesign (Ahmad's explicit request)
- ~~🟡 Bitcoin wallet age calculation bug~~ — fixed in Task 7D ✅
- 🟡 BSC/Base/Optimism return 503 — waiting on Ahmad's Etherscan Lite ($49/mo) decision
- 🟡 Dead code: `recordHistory()` in `score.ts` still writes to `lib/history.ts` — nothing reads it anymore (flagged for future cleanup)

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
| Non-EVM scoring heuristics are unchanged | Ahmad confirmed acceptable |
| WOR reports are automated (no admin review) | Ahmad's decision |

---

## Next 3 Things the Manager Must Do (In Order)

### 1. Task 7B (Frontend: txCount Cap Indicator) — Assign to Frontend Builder next
Quick cosmetic win. Full spec in TASKS.md and FRONTEND_TASKS.md.

### 2. Task 7B → 7C → 10 → 9 → 8 → 11A → 11B — queue in that order
All queued for Frontend Builder. One task at a time.

### 3. Task 12 (Backend: Signal Accuracy Audit) — queued for AFTER distribution prep
Non-EVM chains (Bitcoin, Solana, TON, Tron, Sui) are being scored with EVM-specific signals (token holding, smart contract interactions, internal transactions). This produces wrong scores. Satoshi genesis wallet showing 51 days age (should be ~5,700+) confirmed this is still broken despite Task 7D. Schedule Task 12 after Task 11B. Full spec in TASKS.md.

---

## The Replit Multi-Account System

Ahmad uses multiple Replit free-tier accounts to avoid hitting credit limits:
- **Manager account** — writes prompts, manages roadmap
- **Frontend Builder account** — builds frontend, notifies Manager when work is done
- **Backend Builder account** — builds backend, notifies Manager when work is done

When credits exhaust → Ahmad pushes to GitHub via Replit's Git interface → open new account → handover → continue.

**The problem:** Handover documents between Manager accounts were inconsistent, causing context loss and orphaned planned features appearing as "abandoned" code.

**The solution (implemented July 5, 2026):**
All context now lives in the GitHub repository itself — not in AI chat memory:
- `docs/MANAGER_HANDOVER.md` — this file
- `docs/ARCHITECTURE.md` — what every piece of the codebase is
- `docs/ROADMAP.md` — all planned features with status
- `docs/TASKS.md` — master task list (all tasks, all roles — Manager adds tasks, Builders update it only when Manager says so)
- `docs/BACKEND_TASKS.md` — Backend Builder's dedicated task list
- `docs/FRONTEND_TASKS.md` — Frontend Builder's dedicated task list
- `docs/BUILDER_ONBOARDING.md` — onboarding for each Builder role
- `docs/READING_GUIDE.md` — who reads which file and when

---

## How Builders Get These Docs (No Manual File Sharing Needed)

Builders do not receive docs via email or file download. The **OTI docs** zip file lives inside the GitHub repos. When a Builder clones the repo to start work, they extract the OTI docs zip from the project root to get all doc files.

**What Ahmad tells each Builder on day one:**
> "Clone the repo. Find the OTI docs zip file in the project root and extract it. Read `BUILDER_ONBOARDING.md` first, then `ARCHITECTURE.md`, then open your dedicated task file — `BACKEND_TASKS.md` if you are the Backend Builder, `FRONTEND_TASKS.md` if you are the Frontend Builder."

**GitHub visibility:**
- **Frontend repo → PUBLIC** — React code has no secrets, making it public builds developer trust
- **Backend repo → PRIVATE** — scoring algorithm IP, admin routes, API key logic stays private

---

## New Manager Startup Procedure (5 minutes, not 5 hours)
1. Clone the repo (or `git pull` if already cloned)
2. Read `docs/MANAGER_HANDOVER.md` (this file)
3. Read `docs/ARCHITECTURE.md`
4. Read `docs/ROADMAP.md`
5. Read `docs/TASKS.md`
6. Check GitHub for open PRs awaiting review
7. Continue from "Next 3 Things" section above

**Rule before ending ANY Manager session:**
Update the following before closing — no exceptions:
1. If a task was confirmed done: explicitly tell the Builder to mark it ✅ in both their own task file AND `TASKS.md`
2. If a new task was added: add it to `TASKS.md` first, then explicitly tell the Builder to add it to their own file AND to `TASKS.md`
3. Builders do not update any file on their own — they act only when the Manager explicitly tells them to, whether marking done or adding new tasks
4. Update "Current State of Production" and "Next 3 Things" in this file
5. Ahmad pushes everything via Replit's Git interface

This is the lifeline between Manager accounts. 5 minutes of updating now saves hours of re-research later.

---

## ⚠️ Critical Infrastructure Note — Railway Migrations Do NOT Auto-Run

Confirmed July 5, 2026: Railway's deploy pipeline only runs `pnpm install && build && start`. It does **NOT** run `drizzle-kit push` or any DB migration step. This means every future schema change (new column, new table, index) that the Backend Builder adds will need Ahmad to manually run `drizzle-kit push` against the Railway production DATABASE_URL after deploying.

**Recommended fix (optional follow-up task):** Add `drizzle-kit push` to `railway.json`'s `buildCommand` so migrations run automatically on every deploy. The builder flagged this. It's a one-line change to `railway.json` — NOT `nixpacks.toml` (which is sacred). Manager can assign this as a micro-task if Ahmad wants.

---

## Critical Context That Must Never Be Lost

1. **Wallet Ownership Registry (WOR)** is Ahmad's flagship new feature. Off-chain signing + passkey pre-registration. Users can self-report wallet compromise even when attacker also has keys (passkey is the differentiator). No blockchain. No admin review. Fully automated. See ROADMAP.md Phase 4.

2. **`wallet_links` table** was built in anticipation of WOR + portfolio. It is NOT abandoned — it needs WOR to exist first.

3. **`scores` table** is a lightweight fast-lookup cache alongside the full `chain_scores` historical table. NOT redundant — serves a different read pattern.

4. **`useHealth` hook** was designed for a navbar API status indicator dot. NOT unused by mistake — just needs a component connected to it.

5. **History endpoint** reads from memory as temporary scaffolding — the permanent fix is to read `chain_scores` from DB. Data is there.

6. **The 3 unavailable chains** (BSC, Base, Optimism) return 503 intentionally — they need Etherscan Lite subscription. Ahmad is aware.
