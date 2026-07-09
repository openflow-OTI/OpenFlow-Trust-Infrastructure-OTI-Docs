# OTI — Manager Handover Document
> Last updated: July 9, 2026 (session 7 — HANDOVER, previous Manager running low on context, NO development work happened this session. Ahmad spent the whole session refining OTI tokenomics with the previous Manager, then explicitly decided: (1) remove ALL price and liquidity numbers from TOKENOMICS.md — the team will decide price/liquidity later, closer to launch; (2) `TOKENOMICS.md` now only covers supply, allocation, vesting, sale structure (no price), and revenue distribution; (3) **explicitly resume development from Task 11 on** — Ahmad said "we didn't build anything today," so the next Manager should pick up exactly where session 6 left off: check in on Task 11C, finalize/send Task 11E. Do NOT reopen the price/liquidity discussion — it was intentionally deferred, and reconstructing numbers from git history would go against Ahmad's explicit instruction.)
> **If you are a new Manager reading this: start here. Then read ARCHITECTURE.md, ROADMAP.md, and TASKS.md in that order.**
> **⚠️ Read `TOKENOMICS.md` before touching anything token-related — price/liquidity sections were deliberately removed at Ahmad's request July 9, 2026. Do not add them back or estimate them yourself.**

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
| Frontend Builder | New account (July 8, 2026) — previous account hit credit limit, new account imported | Task 8B ✅, 8C ✅, 8D ✅, 8E ✅, 11A ✅, 11B ✅, 11 ✅ all done/live/verified. Docs site fully live at `otiscore.vercel.app/docs/` (see Task 11 section below). Idle, awaiting next task — do not send anything new until Ahmad confirms priority between Task 11E (AI-tell audit, see below) and Task 11C follow-up. |
| Backend Builder | Active | Task 9C fully done and verified. Task 11C (Signal Accuracy Audit) sent — in progress, not yet reviewed. |
| Development Manager | This account | Writes prompts, reviews PRs, owns roadmap |

---

## Current State of Production (as of July 8, 2026 — end of session)

**Live and working:**
- ✅ Backend on Railway: `https://workspaceapi-server-production-5c0c.up.railway.app`
- ✅ Frontend on Vercel: `https://otiscore.vercel.app`
- ✅ 15 chains scoring (10 EVM + TON, Solana, Sui, Bitcoin, Tron; BSC/Base/Optimism intentionally 503)
- ✅ API key + daily quota system (dynamic, DB-driven)
- ✅ Score caching
- ✅ Compromised wallet denylist + red banner UI
- ✅ UI polish, Logo fix, SVG logo (Tasks 1, 2, 2B)
- ✅ Admin endpoints secured with `x-admin-secret` header (Task 3)
- ✅ History endpoint reads from `chain_scores` DB (Task 4)
- ✅ Score response returns weighted signals `{ score, weighted, maxWeight }` per signal (Task 5)
- ✅ `updated_at` column on `subscriptions` table (Task 6)
- ✅ Bitcoin wallet age pagination fix — 40-page cap (Task 7D)
- ✅ Signal bars show `weighted/maxWeight` (Task 7)
- ✅ txCount cap indicator — "1,000+ transactions" (Task 7B)
- ✅ Homepage shows live anonymous daily_limit from DB (Task 7C)
- ✅ Navbar API health status dot (Task 10)
- ✅ Admin Panel fully working — login, Dashboard, API Keys (create/list/edit/delete), Query History, Cache, Plan Configs (Tasks 9, 9-BACKEND)
- ✅ API key reveal modal on creation — shows full key once with copy button (Fix: July 7, 2026)
- ✅ Task 9C — Plan limit enforcement verified: 429 confirmed for free plan, enterprise null = unlimited, all backend ORM schema mismatch bugs fixed (apiKeyAuth.ts, score.ts, admin.ts DELETE + PATCH all use raw SQL)
- ✅ Task 8 — Professional results page redesign: chain-color ring gauge, tier labels, Trust Signals card, truncated wallet address + copy, Share (native share sheet) + Save as Image (3× PNG), "⚑ Report this wallet" mint ghost link, footer, full color system upgrade
- ✅ Task 8B — Professional wallet input page redesign, incl. polish round (logo size/position, zkSync/Linea icon visibility, chain icon size, spacing, report-link styling) — verified live July 8, 2026
- ✅ Task 8C — Anonymous rate-limit cache sync fixed. Real root cause: `setEditId(null)` in the admin mutation's `onSuccess` raced with React 18 batching and unmounted the edit row before the success banner/cache write could be trusted. Fixed by keeping the row open until "Done" is clicked. Verified live on Vercel July 8, 2026 — anonymous limit change to 42 updated the homepage instantly; unlimited (blank field) correctly showed "Unlimited".
- ✅ Task 8D — Homepage visual polish shipped: white placeholder/example-link text, "Try an example" border rebuilt as a GPU-cheap transform-based animation (was a paint-triggering `@property`/conic-gradient causing jank), corrected oversized/zoom sizing, improved spacing and typographic hierarchy. Verified live by Ahmad via screen recording July 8, 2026.
- ✅ Task 8E — Mobile pinch/double-tap zoom fixed: viewport meta set to `maximum-scale=5, minimum-scale=1` (accessibility-conscious compromise, not a full zoom lock) plus a `touch-action: manipulation` CSS backstop for iOS double-tap. Confirmed working smoothly across homepage, results page, and admin dashboard on mobile; desktop zoom unaffected. Verified by Ahmad, July 8, 2026.
- ✅ Task 11A — Marketing homepage live at `/` (Hero, chain row, How It Works, Trust Signals, Use Cases, Get the API, Find Us/Integrations, footer); scoring tool live unchanged at `/score`. Brand consistency confirmed (locked color system, shared components, logo matches `/score` exactly). Verified live by Manager via fresh cache-busted screenshot + JS bundle inspection, and by Ahmad directly, July 8, 2026.
- ✅ Task 11 — Developer Docs Site fully live, July 9, 2026. Standalone Docusaurus site at `https://oti-docs.vercel.app/` (pnpm-based build — npm was abandoned after a real Vercel/npm CLI defect, see Lessons below), proxied onto the main site via `vercel.json` rewrites under `/docs/`. Manager verified via curl: `otiscore.vercel.app/docs`, `/docs/` (trailing slash), and `/docs/api-reference` all return 200 with correct Docusaurus content. Full debugging arc (npm→pnpm switch, baseUrl/routeBasePath conflict, trailing-slash rewrite gap) documented in Lessons Learned section below. Still open: re-verify the "Try It Live" widget calls the real Railway backend (API URLs were reverted earlier but never re-checked live after the redeploys).

**ADMIN_SECRET status:**
- Ahmad set `ADMIN_SECRET` in Railway Variables
- Admin Panel login at `otiscore.vercel.app/admin` confirmed working

**⚠️ OTI COLOR SYSTEM — LOCKED July 7, 2026**
The app palette was upgraded from flat black to a deep-space aesthetic. All future tasks must use these exact values:

| Token | Value |
|---|---|
| Background | `#05080f` |
| Surface | `#0b0f1a` |
| Surface-2 | `#0f1520` |
| Borders | `#1c2535` |
| Body text | `#e8f4ff` |
| Dimmed text | `#7a8fa8` |
| Mint primary | `#00e5a0` |
| Mint gradient | `#3EFFC1` |

Chain colors: Ethereum `#627EEA` · Bitcoin `#F7931A` · Solana `#9945FF` · BNB `#F3BA2F` (all 15 in codebase).
Special effects: navbar `backdrop-filter: blur(14px)`, submit button mint glow, score panel `color-mix()` chain tint — all with plain-value fallbacks. Do NOT revert to pure black `#000000`.

**Critical infrastructure note — Railway subscriptions table real schema:**
The Drizzle schema in the repo does NOT match Railway's actual DB. Real columns:
```
id, api_key, plan, owner_address, created_at, expires_at, updated_at
```
**NO `status` column. NO `email` column.** All backend handlers use raw SQL. Never use Drizzle ORM selects on this table — they will crash with "column does not exist".

**Known open issues:**
- 🟡 "AI-native tell" audit not yet scoped/sent — full pass needed across homepage, docs site, AND whitepaper: copy/tone (remove AI-generated-sounding phrasing), emoji usage (not just Trust Signals/Use Cases — audit everywhere), and any other visual/textual tell. This supersedes and absorbs the old Task 11D scope. Ahmad flagged this directly July 9, 2026 — treat as high priority, but still respect one-task-per-Builder rule.
- 🟡 "Try It Live" widget on docs site — not re-verified against the real Railway backend since the Task 11 redeploys.
- 🟡 Non-EVM signal accuracy — Bitcoin/Solana/TON/Tron/Sui scored with EVM logic (Task 11C — CRITICAL before distribution)
- 🟡 Satoshi genesis wallet shows incorrect age — may be stale cache, flush and retest
- 🟡 BSC/Base/Optimism return 503 — waiting on Ahmad's Etherscan Lite ($49/mo) decision
- 🟡 Dead code: `recordHistory()` in `score.ts` writes to `lib/history.ts` — nothing reads it (future cleanup)
- 🟡 Key expiry (`expires_at`) enforcement not tested — backend reads the column but unknown if it gates requests. Low priority until developer portal exists.

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

### 1. DONE — Task 11B ✅ and Task 11 ✅ fully live and verified

**Task 11B** is ✅ DONE. All 3 fixes confirmed live.

**Task 11** (Developer Docs Site) is ✅ DONE — fully live and verified July 9, 2026. Full arc for reference:
- Original 404 root cause: the Vite dev proxy in `vite.config.ts` only works in Replit's dev environment; on Vercel the SPA catch-all intercepted `/docs/*` before the docs were ever deployed as their own project.
- npm install on the standalone `oti-docs` Vercel project hit a real npm-CLI-on-Vercel defect (deterministic "Exit handler never called!" failure across three independent fix attempts) — resolved by switching the project from npm to pnpm (deleted lockfile/node_modules, generated `pnpm-lock.yaml`). This is a permanent choice — do not switch `oti-docs` back to npm.
- `docusaurus.config.js`: `baseUrl` set to `/docs/`, `docs.routeBasePath` set to `''` (not `'/'`) — this exact combination is required; see Lessons Learned below for why routeBasePath `'/'` breaks the root page.
- Main site `vercel.json` has three rewrite rules for the docs proxy: `/docs`, `/docs/` (trailing slash — this one was missed initially and caused a lingering false-negative), and `/docs/:path*`, all before the SPA catch-all. This was a Manager-approved one-line exception to "never touch vercel.json."
- Standalone deployment lives at `https://oti-docs.vercel.app/`; public-facing URL is `https://otiscore.vercel.app/docs/`. Two separate Vercel projects for the same GitHub repo (main site root, `oti-docs` with Root Directory = `oti-docs`) — intentional, do not consolidate.
- Verified live via curl July 9, 2026: `/docs`, `/docs/`, and `/docs/api-reference` all return 200 with correct Docusaurus content.

**Still open before the whole docs initiative is closed:**
1. Re-verify "Try It Live" widget calls the real Railway backend (not yet re-checked live after the redeploys).
2. NEW — Task 11E: full "AI-native tell" audit across homepage, docs, AND whitepaper (copy/tone, emoji everywhere, any other AI-generated tell). This supersedes the old Task 11D (which only covered Trust Signals/Use Cases emoji). Ahmad raised this directly July 9, 2026 — scope it into a proper task prompt and send per the one-task-per-Builder rule; do not send until Ahmad confirms priority vs. checking in on Task 11C.

### Lesson learned this session (scope creep — historical, resolved)
Ahmad personally authorized the previous Frontend Builder to keep working past the original Task 11 spec, which led to 8 self-directed rounds and several unplanned changes (OG image, Try It Live widget, a privacy audit that deleted internal doc folders, and a custom-domain migration to `api.otiscore.io` that isn't DNS-live). Some of it was valuable (the privacy audit caught a real leak) but it also broke production (docs 404, Try It Live down) without a mid-task check-in. All of this has since been remediated and Task 11 is now fully live. Going forward: even with Ahmad's blanket go-ahead, a Builder should check in with the Manager before migrating live API URLs or touching deployment config, since those changes have production blast radius beyond the original task.

### Lesson learned this session (Vercel/Docusaurus docs proxy debugging)
Full detail in the Manager's persistent memory (`vercel-docs-proxy.md` topic) — summary: (1) Vercel rewrites treat `/docs` and `/docs/` as distinct routes, a wildcard `/docs/:path*` rule does not cover the bare trailing-slash case, always test all three forms via curl; (2) Docusaurus's `baseUrl` only changes link prefixes/router basename, not physical build output folder structure — don't assume a hosting platform's directory settings are misconfigured before checking the actual local `build/` folder structure.

### 0. RESUME HERE — session 7 was a tokenomics-only session, no dev work happened
Ahmad spent session 7 entirely on refining OTI tokenomics with the Manager (liquidity/market-cap ratio math went back and forth several times). He explicitly said "we didn't build anything with you today" and asked to resume from Task 11 onward. **Nothing changed in Task 11C or Task 11E status since session 6 — pick up exactly there:**
- Check in with Backend Builder on Task 11C (Signal Accuracy Audit) progress.
- Finalize the Task 11E prompt (AI-native tell audit) and confirm priority with Ahmad before sending to Frontend Builder (still not sent as of end of session 7).
- `TOKENOMICS.md` is otherwise locked (supply/allocation/vesting/sale/revenue) — price and liquidity were explicitly stripped out per Ahmad's instruction; do not re-add them without him raising it again.

### 2. Task 11C sent to the Backend Builder — awaiting their work
Task 9C is done. Task 11C (Signal Accuracy Audit — non-EVM chains scored with EVM logic, CRITICAL before distribution) was sent to the Backend Builder. Do not send anything further to Backend Builder until 11C is verified done.

### 3. Frontend queue — Task 11 fully closed, Builder idle
Frontend Builder is idle as of July 9, 2026. Next candidate task is 11E (AI-tell audit, see above) — write the task prompt and confirm priority with Ahmad before sending, since he raised it directly and it's broader than anything previously scoped.

### 4. Backend queue after Task 11C (in this exact order, one at a time)
- Tasks 12–15 — Distribution channels (Telegram Bot, Chrome Extension, Widget, Firefox Extension)

### Lesson learned this session (read before trusting any "it's done" report)
A live-site check of Task 11A initially looked broken (`/` showed the old scoring form, not the marketing homepage) — turned out to be a stale/cached first screenshot, not a real bug. A second cache-busted screenshot (`?t=<random>` query param + checking `x-vercel-cache`/`age` headers) confirmed it was genuinely live and correct. **Always re-verify with a cache-busted check before reporting a problem to Ahmad** — false alarms cost trust too.

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
- Task 8B (Wallet Input Page Redesign) is ✅ DONE, verified live.
- Task 8C (Anonymous Rate Limit Cache Sync Fix) is ✅ DONE, verified live by Ahmad on Vercel (real root cause was `setEditId(null)` racing with React 18 batching in the admin mutation's `onSuccess`).
- Task 8D (homepage polish: contrast, animated CTA border rebuilt to be GPU-cheap, zoom/sizing, spacing, typography) is ✅ DONE, verified live by Ahmad via screen recording.
- Task 8E (disable mobile pinch/double-tap zoom, accessibility-conscious `maximum-scale=5` compromise) is ✅ DONE, confirmed working smoothly by Ahmad.
- Task 11A (Marketing Homepage at `/`, scoring tool moved intact to `/score`) is ✅ DONE. Manager double-checked live via cache-busted screenshot + JS bundle inspection after an initial false alarm from a stale first check — genuinely live and brand-consistent. Confirmed by Ahmad directly too.
- The entire Task 8 frontend redesign arc (8, 8B, 8C, 8D, 8E) plus Task 11A are now closed out. Frontend Builder is free for the next task.
- Task 11B (Whitepaper) is ✅ DONE — all 3 fixes confirmed live (white body text, no mobile horizontal scroll, Roadmap removed + renumbered).
- Task 11 (Developer Docs Site — Docusaurus) is ✅ DONE — confirmed fully live July 9, 2026 (see "Next Things" section above for full arc and verification detail).
- Task 11C (Signal Accuracy Audit) is with the Backend Builder, in progress, not yet reviewed.
- NEW Task 11E (AI-native tell audit — homepage/docs/whitepaper copy, tone, emoji) is scoped but NOT yet sent to any Builder — confirm priority with Ahmad first.
- One task at a time per Builder — do not send Task 11E to Frontend Builder without Ahmad's priority confirmation; do not send anything after 11C to Backend Builder until 11C is verified done.

**Rule before ending ANY Manager session:**
1. If a task was confirmed done: tell the Builder to mark it ✅ in THEIR OWN task file AND their own TASKS.md — the Manager's copy of these files is physically separate from each Builder's copy (different account, different repo checkout). Updating the Manager's copy never updates a Builder's copy. This instruction must be sent explicitly, every time, even if the Manager's own file already shows the task as done.
2. If a new task was added: add to TASKS.md first, then tell Builder to add to their file AND TASKS.md (same separate-copies caveat as above)
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
