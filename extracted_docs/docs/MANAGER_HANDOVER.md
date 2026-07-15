# OTI — Manager Handover Document
> Last updated: July 15, 2026 (session 16 — WOR Phase 2 fully live. BF38/BF39/FF24 closed. BF40 open (Backend Builder hit credit limit before starting it). Frontend Builder active on Task 18 + FF25/FF26/FF27. New Manager and new Backend Builder being onboarded now.)
> **If you are a new Manager reading this: start here. Then read ARCHITECTURE.md, ROADMAP.md, TASKS.md, FIXES.md, and DECISIONS.md in that order.**
> **⚠️ Read `TOKENOMICS.md` before touching anything token-related — price/liquidity sections were deliberately removed at Ahmad's request. Do not add them back or estimate them yourself.**
> **⚠️ D16 (evidence rule, non-negotiable): no signal value or test result may ever be estimated or guessed — only real on-chain data or a disclosed hard cap. A Builder's claim of "verified" is not evidence — check what wallet, what raw API response. Send it back if it's code-inspection only.**

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
- Bug fixes / cleanup / polish are NOT tasks — they go in `FIXES.md`, never folded into an existing task or listed as a numbered Task

---

## The Team

| Role | Status | Notes |
|---|---|---|
| Ahmad (CEO) | Always active | Sole GitHub merge authority |
| Frontend Builder | 🔴 ACTIVE — Task 18 + FF25/FF26/FF27 | New builder onboarded July 15, 2026 (third builder — previous hit credit limit mid-FF24). Combined prompt sent. Awaiting completion report. |
| Backend Builder | ⚠️ NEEDS NEW BUILDER — BF40 unstarted | Hit credit limit July 15, 2026 mid-session before starting BF40. Ahmad is onboarding a new Backend Builder. BF40 prompt is ready (see "Next Things" section below). |
| Development Manager | This account — being replaced now | New Manager being onboarded. |

---

## Current State of Production (July 15, 2026)

**Live and working:**
- Backend on Railway: `https://workspaceapi-server-production-5c0c.up.railway.app`
- Frontend on Vercel: `https://otiscore.vercel.app` (marketing homepage `/`, scoring tool `/score`, whitepaper `/whitepaper`, admin panel `/admin`)
- Developer docs: `https://otiscore.vercel.app/docs/` (standalone: `https://oti-docs.vercel.app/`)
- 15 chains scoring (10 EVM + TON, Solana, Sui, Bitcoin, Tron; BSC/Base/Optimism intentionally return 503 pending Ahmad's Etherscan Lite decision)
- API key + daily quota system (dynamic, DB-driven, real 429s)
- Score caching live
- **WOR (Wallet Ownership Registry) fully live** — /register, /report, admin WOR tab (Registry, Compromised, Manual Override), compromised-wallet warning on score page. Verified end-to-end by Ahmad with Trust Wallet on July 15, 2026.
- Admin Panel fully working — Dashboard, API Keys, Query History, Cache, Plan Configs, WOR tab

**Fix status as of July 15, 2026:**
- BF1–BF39: all ✅. BF40 is ⏳ AWAITING AHMAD LIVE VERIFICATION — new Backend Builder self-reported the two counts already match in production (no code change made), but this has not been independently confirmed by Ahmad or the Manager yet. Do not mark closed until Ahmad checks the actual dashboard + WOR Compromised sub-view live.
- FF1–FF27: FF25/FF26/FF27 are 🔴 ACTIVE (Frontend Builder working on them with Task 18)
- Task 18 (/services hub page): 🔴 ACTIVE (Frontend Builder working on it)

**ADMIN_SECRET status:** Set in Railway Variables. Admin Panel login at `otiscore.vercel.app/admin` confirmed working.

**⚠️ OTI COLOR SYSTEM — LOCKED July 7, 2026.** Full token table lives in `FRONTEND_TASKS.md`. Do not revert to pure black `#000000`.

**Critical infrastructure note — Railway `subscriptions` table real schema:** Real columns: `id, api_key, plan, owner_address, created_at, expires_at, updated_at`. **NO `status` column, NO `email` column.** All backend handlers use raw SQL on this table — never use Drizzle ORM selects here, they crash with "column does not exist."

---

## What Happened This Session (July 15, 2026)

**WOR go-live and debugging (BF38, BF39, BF40):**
- BF38: Admin WOR Compromised view was returning 0 — root cause was a missing `GET /admin/wor/compromised` route (only `GET /admin/wor/registrations?status=compromised` existed). Added the route. Closed ✅.
- BF39: After BF38 fix, Compromised view still missed orphaned wallets — wallets in `compromised_wallets` with no `wallet_ownership` row. Fixed by rewriting the route to query `compromised_wallets` as primary source with LEFT JOIN `wallet_ownership`. Deployed. Ahmad confirmed working ✅.
- BF40: Dashboard FLAGGED WALLETS stat queries `wallet_ownership WHERE status='compromised'`, but WOR Compromised view now queries `compromised_wallets`. Mismatch causes dashboard to show 1 while Compromised view shows 0. Fix: stats endpoint must query `compromised_wallets` directly. **Not started — Backend Builder hit credit limit.** Prompt is ready below.
- FF24: WOR UI/UX polish — all 5 items verified live by Ahmad. Closed ✅.
- FF25/FF26/FF27: New fixes from Ahmad's verification — WOR entry point ordering, OTI logo in WalletConnect DApp modal, OTI logo in /register success screen. Frontend Builder working on these with Task 18.
- Task 18: New `/services` hub page — a portal where all OTI services are surfaced as cards. Ahmad's decision: keep homepage unchanged, add hub at `/services` with navbar link. Frontend Builder working on it.

**Key architectural decision — /services hub (July 15, 2026):**
Ahmad wants a unified service portal at `/services`. Current homepage stays. `/services` lists all services as cards: Score a Wallet, WOR (Register/Report), API Docs, Whitepaper, and Coming Soon placeholders. Navbar gets a "Services" link. This is the structure for adding future services cleanly.

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
| One task at a time per Builder | Ahmad's explicit, hard preference |
| All Manager replies to Ahmad in copy boxes | Ahmad's preference — he pastes them to Builders |
| Fixes are never folded into tasks or given task numbers — they live in `FIXES.md` | Ahmad's explicit correction, July 10, 2026 |
| Price/liquidity design intentionally absent from `TOKENOMICS.md` | Deferred by Ahmad, decide later, do not reconstruct |
| DECISIONS.md is Manager-write, Builder-read — Builders never update it | Ahmad's direction |
| Whitepaper additions held until backend fully verified | Content draft exists at docs/whitepaper-additions-draft.md |
| Contract addresses are NOT rejected from scoring — scored like any other address | OTI never verifies identity; bot/scam-risk deferred to Phase 4 |
| D16 (evidence standard) applies retroactively, including to "verified" reports | Always ask for the specific wallet address and raw API response |
| compromised_wallets is the source of truth for flagged wallets | Both score endpoint and admin WOR view read from it — never split this |
| Homepage at / stays unchanged — services hub is at /services | Ahmad's decision, July 15, 2026 |

---

## Next Things the Manager Must Do (In Order)

### 1. 🔴 Send BF40 to new Backend Builder (FIRST THING)

As soon as the new Backend Builder is onboarded, send them this prompt:

---
BF40 — Dashboard FLAGGED WALLETS Count Mismatches WOR Compromised View

Small fix. Do not touch scoring.ts or nixpacks.toml.

THE BUG

Dashboard FLAGGED WALLETS card shows 1. WOR admin Compromised sub-view
shows 0 results. They are reading from different sources.

After BF39, GET /admin/wor/compromised correctly queries
compromised_wallets as the source of truth. The GET /api/admin/stats
"flagged_wallets" count is still querying wallet_ownership WHERE
status = 'compromised' — so the two are out of sync whenever a wallet
ends up in one table but not the other.

THE FIX

In the admin stats handler, change the flagged_wallets count to query
compromised_wallets directly:

  SELECT COUNT(*) FROM compromised_wallets

This makes it consistent with the Compromised sub-view and with the
score endpoint — all three now read from the same source of truth.

DEFINITION OF DONE

1. Confirm current flagged_wallets count in GET /api/admin/stats via
   curl against production
2. Confirm current row count in compromised_wallets via psql
3. Apply fix
4. Confirm the two numbers now match
5. Confirm dashboard FLAGGED WALLETS card and WOR Compromised sub-view
   show the same count

Report back with the curl output and psql COUNT before and after.
Deploy to Railway. No drizzle-kit push needed.

Update your copy of FIXES.md: BF40 → 🔴 ACTIVE (in progress).
---

### 2. ⏳ Wait for Frontend Builder to report Task 18 + FF25/FF26/FF27 complete

The Frontend Builder is working on a combined prompt covering:
- **Task 18:** New `/services` hub page (service cards, navbar link)
- **FF25:** Register as primary WOR entry point throughout the site
- **FF26:** OTI logo in WalletConnect DApp connect modal (manifest + iconUrl)
- **FF27:** OTI logo in /register success screen (circle treatment)

When they report back, verify:
1. `/services` loads, all cards link correctly, mobile-responsive at 375px
2. "Services" appears in the navbar
3. Homepage WOR section: Register button is first, Report is second
4. WalletConnect DApp modal shows OTI logo (Ahmad tests this with Trust Wallet after deploy)
5. `/register` step 4 success screen shows OTI logo in a circle
6. `npm run build` clean, zero TS errors

Then tell Frontend Builder to mark FF25 ✅, FF26 ✅, FF27 ✅ in their FIXES.md and Task 18 ✅ in their FRONTEND_TASKS.md and TASKS.md.

### 3. ⏳ Ahmad action — verify BF39 with a fresh self-report

Ahmad confirmed BF39 is deployed and active on Railway. But the Compromised tab showed 0 because all test wallets had been removed. Ahmad needs to:
1. Self-report one of his registered wallets with a fresh challenge
2. Confirm the wallet appears immediately in Admin → WOR → Compromised
3. Optionally: Remove Flag, confirm it disappears

Once confirmed, mark BF39 ✅ in FIXES.md and tell the Backend Builder to mark it ✅ in their copy.

### 4. Ahmad's action — create two API keys (Phase 1 task 1D) — still pending

Via admin panel at otiscore.vercel.app/admin:
- Internal bot key: high daily limit, for Telegram/Discord bots
- Widget shared key: for the embeddable widget (Task 14)
Once done, Phase 1 is fully closed and Phase 5 (bots + widget + extension) is unblocked.

### 5. After Task 18 + BF40 + BF39 all confirmed ✅ → Design Phase 2B

Phase 2B (OTI Verified Badge) architecture is fully locked in ROADMAP.md and DECISIONS.md D17–D22. No Builder prompts written yet — Manager designs the build sequence first, then prompts go out.

Key Phase 2B facts:
- BAS attestation on BNB Chain only — no soulbound NFT, no MetaMask Snap
- First 10M attestations free → one-time fee after (admin panel managed)
- OTI auto-rescores every 30 days — no user action needed
- Widget (partner-side) + Extension (user-side) are the display layers
- First 1M attestation users receive OTI token reward before token launch
- Badge visual design + score thresholds confirmed during build, not before
- Full design: ROADMAP.md Phase 2B | All decisions: DECISIONS.md D17–D22

### 6. Phase 3 pillars (after Phase 2B)
- Fiat/crypto payment infrastructure (self-serve portal, Stripe, Coinbase Commerce)
- OTI token: create + deploy BSC + ecosystem integration + presale ($10k raise)
- Exchange listing: post-Phase-3, after revenue is live. Ahmad decides timing.

### 7. Whitepaper additions (when ready)
Draft at docs/whitepaper-additions-draft.md. Send to Frontend Builder once Ahmad gives the go-ahead.

---

## Lessons Learned (carry forward every session)

**Builder handoffs:** New Builder starts with zero API keys/secrets — re-add all of them. Always give a full onboarding pass before resuming work, even if code is the same.

**D16 evidence standard:** "Verified" means real wallet, real raw API response, real psql output — not code inspection. Ask "which wallet, which response?" every time before accepting a close. This caught multiple fabricated verifications this project.

**Railway deploys don't auto-run migrations:** Railway only runs install + build + start. Every schema change needs Ahmad to manually run `drizzle-kit push` against Railway production DATABASE_URL via the Railway Console. Config: `cd /app/lib/db` then `drizzle-kit push`.

**compromised_wallets is the single source of truth** for flagged wallets. The score endpoint, the admin WOR Compromised view, and the dashboard stats must ALL query `compromised_wallets` — never split them across `compromised_wallets` and `wallet_ownership.status`. BF38/BF39/BF40 all trace back to this being inconsistent.

**WalletConnect challenge TTL is 15 minutes.** Any self-report test that takes longer than 15 minutes from challenge generation will silently hit the 400 (challenge expired) branch. Always do the full register → challenge → sign → submit flow in one sitting.

**Builder file copies never auto-sync.** Manager's files and each Builder's files are physically separate. Every status change must be explicitly relayed with instruction to update their own copy.

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
- `docs/DECISIONS.md` — why things exist the way they do; Manager-write, Builder-read
- `docs/BACKEND_TASKS.md` / `docs/FRONTEND_TASKS.md` — each Builder's dedicated task file
- `docs/BUILDER_ONBOARDING.md` — onboarding for each Builder role

---

## How Builders Get These Docs

The **OTI docs zip file** lives inside the GitHub repos. When a Builder clones the repo, they extract the zip to get all doc files.

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
6. Read `docs/DECISIONS.md` — why things exist the way they do
7. Jump straight to "Next Things the Manager Must Do" above and execute in order

**Rule before ending ANY Manager session:**
1. If a task or fix was confirmed done: tell the Builder to mark it ✅ in THEIR OWN copy — the Manager's copy and Builder's copy are physically separate, never auto-sync.
2. If a new task or fix was added: add it to `TASKS.md` or `FIXES.md` first, then tell the Builder to add it to their own copy.
3. Update "Current State of Production" and "Next Things" in this file before closing.
4. One task at a time per Builder — Ahmad's hard rule, never break it.
5. Fixes never get task numbers — they go in `FIXES.md` only.

---

## ⚠️ Critical Infrastructure — Railway Migrations Do NOT Auto-Run

Railway only runs install + build + start. It does **NOT** run `drizzle-kit push`. Every schema change needs Ahmad to manually run `drizzle-kit push` against the Railway production `DATABASE_URL` after deploying. Run from: `cd /app/lib/db` then `drizzle-kit push`.

---

## Critical Context That Must Never Be Lost

1. **WOR (Wallet Ownership Registry)** is Ahmad's flagship feature — off-chain EIP-191 signing + bcrypt passkey, no blockchain, no admin review, fully automated. Live at /register and /report.
2. **compromised_wallets is the source of truth** — score endpoint, WOR admin view, and dashboard stats ALL must query this table. Never query wallet_ownership.status for flagged-wallet counts.
3. **wallet_links table** — built in anticipation of WOR + portfolio features. Not abandoned.
4. **scores table** — lightweight fast-lookup cache alongside chain_scores historical table. Not redundant.
5. **3 unavailable chains** (BSC, Base, Optimism) return 503 intentionally, pending Ahmad's Etherscan Lite ($49/mo) decision.
6. **Task numbering:** 12 = Telegram Bot, 13 = Discord Bot, 14 = Widget, 15 = Firefox Extension, 16 = WOR Backend, 17 = WOR Frontend, 18 = /services Hub (active). All Phase 5 tasks (12–15) are gated on Phase 1D (create operational API keys).
7. **subscriptions table real schema** — `id, api_key, plan, owner_address, created_at, expires_at, updated_at`. NO `status`, NO `email`. Use raw SQL on this table only.
8. **Main app uses npm. oti-docs/ uses pnpm.** Never mix them.
9. **WalletConnect challenge TTL = 15 minutes** — full sign flow must complete within this window or the self-report hits the 400 (expired) branch silently.
10. **Phase order confirmed by Ahmad:** Phase 2 → Phase 2B → Phase 3 → Phase 4 → Phase 5.
