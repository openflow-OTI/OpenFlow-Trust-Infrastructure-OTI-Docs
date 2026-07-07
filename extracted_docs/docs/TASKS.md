# OTI — Master Task Queue
> Last updated: July 7, 2026 (updated by Manager — Admin Panel + Plan Configs fully done, Task 8 next) | Maintained by: Development Manager
> **Manager:** This is your master record — add all new tasks here first, then instruct Builders.
> **Builders:** You also update this file — but only when the Manager explicitly tells you to (marking a task done or adding a new task). Never update it on your own initiative.
> Never let this file go stale.

---

## How to Read This File

- **TASKS.md = the Manager's master record** (all tasks, all roles, all phases — complete picture)
- **BACKEND_TASKS.md / FRONTEND_TASKS.md** = what each Builder actually reads and works from
- **ROADMAP.md = the strategic context** (why we're building it, the phases, the goals)
- When something is done: Manager confirms with the Builder → Manager explicitly tells Builder to mark it done → Builder marks it ✅ here AND in their own task file
- When a new task is added: Manager adds it here → Manager explicitly tells Builder to add it to their file → Builder adds it to their own file AND to this file

---

## Builder Roles

| Role | Status | Notes |
|---|---|---|
| Frontend Builder | Active (new account) | Tasks 1, 2, 2B, 7, 7B, 9, 10, Plan Configs tab, desktop layout fix, anonymous limit hook fix all done — Task 8 is next |
| Backend Builder | Active | Tasks 3, 4, 5, 6, 7D, 9-BACKEND, stats-500-fix, plan-configs-PATCH-fix all done — queue empty, standing by |

---

## ✅ Completed Tasks

### Task 1 — Frontend UI Polish ✅
- Homepage title, back button style, chain icon sizing, dead CSS removed, `isKnownChain()` guard in `useScore.ts`

### Task 2 — Logo Fix ✅
- `Logo.tsx` → `<img src="/logo.jpg">` replacing spiral SVG math
- `generateScoreCard.ts` → uses `loadImage()` before canvas draw

### Task 3 — Admin Route Authentication ✅
- `adminAuth.ts` middleware applied to all `/api/admin/*` routes
- Returns 401 on missing/wrong `x-admin-secret` header
- Swagger updated; verified live on Railway

### Task 4 — History Endpoint → Database ✅
- `GET /api/score/:address/history` now queries `chain_scores` DB table
- Optional `?chain=` filter with auto-detection; ordered by `scored_at` DESC, limited to 50
- OpenAPI spec updated; verified live on Railway

### Task 5 — Signal Scores → Weighted API Response ✅
- `signalWeighting.ts` transformer added (scoring.ts untouched)
- Score response now returns `{ score, weighted, maxWeight }` per signal
- Both OpenAPI specs updated (served + frontend codegen); verified live on Railway

### Task 2B — Logo: SVG Replace ✅
- Original SVG logo file used directly as `public/logo.svg` — no reconstruction needed
- Crisp at 34px on Retina/high-DPI screens, zero blur
- `generateScoreCard.ts` already pointed to `/logo.svg`

### Task 7 — Frontend: Signal Bars → Weighted Display ✅
- `pnpm codegen` run — `src/api/schema.gen.ts` regenerated from live backend OpenAPI spec
- `SignalBar.tsx` updated: bar fill = `(weighted / maxWeight) × 100%`, label = "weighted/maxWeight" e.g. "25/25"
- Color logic updated to use `weighted / maxWeight` ratio
- `Home.tsx` updated to pass full signal object to SignalBar
- `generateScoreCard.ts` updated — score card PNG shows weighted labels and correct fills
- Verified live against real wallet on Vercel — all 5 bars show weighted values
- Also resolved black screen crash caused by Task 5 API shape mismatch

### Task 7D — Bitcoin Wallet Age Fix ✅
- Pagination bug fixed in `bitcoin.ts` — `getBitcoinTxs()` now paginates backwards through mempool.space history
- Safety cap of 40 pages added to bound latency for hyperactive wallets
- Verified on Railway: old wallets returning walletAgedays 4,896 and 5,002 (previously returning ~5)

### Task 6 — subscriptions updatedAt Migration ✅
- `updated_at` column added to `subscriptions` table via Drizzle migration
- `PATCH /api/admin/keys/:id` now sets `updated_at: new Date()` on every update
- Railway production DB migration run manually via Railway Console on July 6, 2026
- `GET /api/admin/keys` verified returning 401 (auth working) — no longer 500

### Task 7B — Frontend: txCount Cap Indicator ✅
- `src/lib/formatMetadata.ts` — signal bar subtitle shows "1,000+ transactions" when `metadata.txCount >= 1000`
- `src/lib/generateScoreCard.ts` — same logic applied so the shareable PNG score card stays consistent
- Verified live on Vercel by Manager: Ethereum Foundation wallet `0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe` returns `txCount: 1000` from the Railway API, and the deployed JS bundle on `otiscore.vercel.app` contains the "1,000+ transactions" string

### Task 10 — Frontend: Navbar API Health Status Dot ✅
- `src/components/Navbar.tsx` — connected `useHealth` hook, renders 7px status dot (green/red/none) top-right of navbar
- `src/index.css` — `.navbar-status-dot` classes (mint green online, red offline, transparent loading) + `.sr-only` utility
- Verified live on Vercel by Manager: green dot visible on `otiscore.vercel.app`

### Task 9 — Frontend: Admin Panel UI ✅
- `/admin` route with password gate (sessionStorage), Dashboard, API Keys, Query History, Cache, Plan Configs tabs
- All API calls include `x-admin-secret` header from sessionStorage — no client-side secret comparison
- Admin Panel login confirmed working at `otiscore.vercel.app/admin` (July 7, 2026)

### Task 9-BACKEND — Backend: Admin Panel API Routes ✅
- All 5 admin routes live: `GET /stats`, `GET /keys`, `POST /keys`, `PATCH /keys/:id`, `DELETE /keys/:id`, `GET /history`, `POST /cache/flush`, `GET /plan-configs`, `PATCH /plan-configs/:id`
- All behind `adminAuth` middleware — 401 without `x-admin-secret`
- `seedPlanConfigs()` self-heal fix: no longer overwrites existing rows' `daily_limit`
- Verified live by Manager on Railway (July 7, 2026)

### Fix: GET /admin/stats 500 Error ✅ (Backend Builder — July 7, 2026)
- Root cause: stats handler had no error handling — any DB query failure caused unhandled rejection → HTML 500
- Fix: per-query isolation with individual try/catch blocks, each defaulting to 0 on failure
- Result: `/admin/stats` always returns HTTP 200 with valid JSON; Admin Panel login now works
- Verified live by Manager: `curl GET /api/admin/stats` → HTTP 200 with real data

### Fix: PATCH /admin/plan-configs/:id 404 Error ✅ (Backend Builder — July 7, 2026)
- Root cause: handler was doing `WHERE plan_name = :id` — UUID passed by frontend matched nothing
- Fix: dual lookup — detects if param is UUID (WHERE id = :param) or name string (WHERE plan_name = :param)
- Both UUID and plan name strings now accepted as path parameter
- Verified live: `PATCH /api/admin/plan-configs/54a72597-...` → HTTP 200; `PATCH .../anonymous` → HTTP 200

### Plan Configs Tab — Admin Panel Frontend ✅ (Frontend Builder — July 7, 2026)
- New "Plan Configs" tab in admin panel showing all 4 plans (anonymous, free, pro, enterprise)
- Inline edit form per row — sets daily_limit and description via PATCH /api/admin/plan-configs/:id
- After successful save, invalidates public anonymous-limit query so homepage updates immediately
- Verified: Ahmad can set anonymous daily_limit via UI and GET /api/config/anonymous-limit reflects the change

### Fix: Admin Panel Desktop Layout ✅ (Frontend Builder — July 7, 2026)
- Root cause: `app-main` div's `max-width: 720px` was constraining the admin panel on desktop
- Fix: CSS `:has()` selector removes width constraints when `app-main` contains the admin shell
- Sidebar + content now fills full browser width on desktop; mobile layout unchanged

### Fix: useAnonymousLimit Hook ✅ (Frontend Builder — July 7, 2026)
- Root cause: hook was calling `GET /admin/plan-configs` (admin-protected, 401 on public page)
- Fix: hook now calls correct public endpoint `GET /api/config/anonymous-limit`
- Fallback: when API returns null, hook defaults to displaying 3
- Homepage now shows live daily_limit value from DB

---

## 🔴 Queue — Not Started (Build In This Exact Order)

---

### TASK 7C-BACKEND — Backend: Public Anonymous Rate Limit Endpoint
**Owner:** Backend Builder
**Phase:** 2 — Operational
**Priority:** MEDIUM — blocks Task 7C (Frontend)
**Status:** ✅ RESOLVED / CLOSED (July 7, 2026, Ahmad's decision — see resolution below)
**Depends on:** Nothing
**Context:** Frontend Builder correctly flagged that `GET /admin/plan-configs` now requires `x-admin-secret` (Task 3), and sending that secret from public client-side code would expose it to every user in devtools. Do not do that. This task adds a safe public endpoint instead.

**Full prompt for Backend Builder:**
> Add a new public endpoint `GET /api/config/anonymous-limit` that returns `{ "daily_limit": number }` — the `daily_limit` value for the `anonymous` row in the `plan_configs` table. No authentication required (this is not sensitive data — it's the same "3 per day" text already shown publicly on the homepage). Do not add this to the `/api/admin/*` route group — it must stay outside `adminAuth` middleware since the frontend calls it with no secret. Add it to the OpenAPI spec.

**Definition of done:** `curl https://workspaceapi-server-production-5c0c.up.railway.app/api/config/anonymous-limit` returns `{ "daily_limit": 3 }` (or current value) with no headers required, no 401.

**Full history — read this before touching the code again:**
1. First deploy: endpoint returned `{ "daily_limit": null }` in production. Investigated — root cause was that the `anonymous` row's `daily_limit` in the **production** `plan_configs` table was `NULL`. Backend Builder assumed this was accidental corruption (e.g. an unauthenticated write before Task 3/admin-auth shipped) and shipped a "self-heal" fix in `seedPlanConfigs()`: on every server boot, if `daily_limit` is `NULL` for anonymous/free/pro, it resets it to a default (3 for anonymous). Enterprise is excluded since `NULL` is its intended "unlimited" meaning.
2. **Ahmad then clarified: the `NULL` value was intentional.** He set it himself, deliberately, to remove the anonymous limit while he does his own manual testing of the live site (unlimited lookups, no 3/day cap getting in his way). He plans to set a real limit himself once the Admin Panel (Task 9) exists.
3. This creates a genuine conflict with the self-heal fix: the fix is *designed* to force `NULL` back to `3` on every boot — which would silently undo Ahmad's intentional testing setup every time Railway restarts, without telling him.
4. Manager re-tested after the fix deployed — endpoint still returned `null` (fix may not have taken effect yet, or Railway hadn't finished redeploying, or the self-heal isn't matching the row — never fully confirmed either way, since Ahmad asked to pause before further debugging).
5. **Ahmad's explicit instruction: leave the limit unlimited for now. Do not force it back to a number. Pause this task until he's done testing.**

**Resolution (Ahmad's decision, July 7, 2026):** Closed. Ahmad will set the real `anonymous` `daily_limit` himself directly via the Admin Panel (Task 9) once it's live — that is now the intended, permanent way this value gets managed, not a temporary testing state. The self-heal fix in `seedPlanConfigs()` must be changed so it never overwrites an existing row's value (including an intentional `NULL`) — it should only seed a default when a plan row is missing entirely (e.g. first-ever boot / fresh database). This fix is folded into Task 9-BACKEND below. The public endpoint itself (route, schema, auth exclusion) does not need to change.

---

### TASK 7C — Frontend: Dynamic Rate Limit Display
**Owner:** Frontend Builder
**Phase:** 2 — Operational
**Priority:** MEDIUM
**Status:** 🟠 ON HOLD — waiting on Ahmad to set a real `daily_limit` via the Admin Panel (Task 9), not a code blocker
**Depends on:** Task 9-BACKEND (admin routes) so Ahmad has a UI to set the limit

**Frontend work already done (per Builder report, July 7, 2026):**
- `src/hooks/useAnonymousLimit.ts` created — ready, just needs to point at the new endpoint once it exists
- `src/pages/Home.tsx` updated — displays live limit on success, falls back to "Anonymous lookups are limited per day. Choose your wallet carefully." on failure
- Correctly avoided sending the admin secret from the client

**Why this is paused too:** The backend endpoint currently returns `null` for `daily_limit` — intentional, since Ahmad plans to set the real value himself via the Admin Panel once it's live. There is nothing broken on the frontend side. No further frontend action needed until Ahmad sets a real limit.

**Definition of done:** Homepage rate limit text reflects the live `anonymous` plan's daily_limit via the new public endpoint. Changing the value in the database updates what the homepage shows automatically.

---

### TASK 9C — Backend: Verify & Harden Plan Limit Enforcement (All Plans)
**Owner:** Backend Builder
**Phase:** 2 — Operational
**Priority:** HIGH — Admin Panel is live; Ahmad can now set limits for all plans. We must confirm the API actually enforces whatever is set — not just for anonymous but for free, pro, and enterprise keys too.
**Depends on:** Task 9-BACKEND (done), Plan Configs tab (done)
**Status:** 🔴 Not started — assigned July 7, 2026

**Why this exists:**
The anonymous plan limit was fixed and verified end-to-end. But free, pro, and enterprise API key limits have never been tested. The Admin Panel lets Ahmad set any daily_limit for any plan. If `apiKeyAuth.ts` has a bug (hardcoded values, wrong table column, stale cache), Ahmad could set a limit and it would silently not be enforced. This must be confirmed before distribution.

**What to verify and fix:**
1. Read `src/middlewares/apiKeyAuth.ts` — confirm it reads `daily_limit` from the `plan_configs` table dynamically on every request (not hardcoded, not cached in memory).
2. Test enforcement for each plan type:
   - Set free plan `daily_limit` to a low number (e.g. 2) via Admin Panel → make 3 requests with a free API key → third request must be rate-limited (429 or equivalent).
   - Restore free plan limit after testing.
   - Confirm enterprise plan with `daily_limit = null` is treated as unlimited (no rate limiting).
3. If any plan's limit is not enforced correctly — fix it. The fix must not touch `scoring.ts`.
4. Add the fix to the OpenAPI spec if any error response shape changes.

**Definition of done:**
- `apiKeyAuth.ts` confirmed reading `daily_limit` dynamically from `plan_configs` (not hardcoded).
- Setting a limit via Admin Panel → Plan Configs takes effect on the next API request (no restart required).
- Free/pro plan limits are enforced. Enterprise null = unlimited confirmed.
- Report back with confirmation that each plan behaves correctly.

---

### TASK 8 — Frontend: Professional Results Page Redesign
**Owner:** Frontend Builder
**Phase:** 3 — Redesign
**Priority:** HIGH — MVP requirement, Ahmad's explicit request
**Depends on:** Task 7 (signal display logic must be correct first), Task 2B (logo must be SVG before the redesign ships)
**Note on ordering:** This is Phase 3 work placed here deliberately — it depends on Task 7 (Phase 1) and should run after Phase 1 backend tasks are merged. Task 9 (Admin panel, Phase 2) can run in parallel with this since they touch different parts of the codebase.

**Full prompt for Frontend Builder:**
> Redesign the scoring results page (`src/pages/Home.tsx` results section and related components). Keep: the black background, the mint green color scheme, the chain selector (do not touch it), the score gauge color system (amber/gold for low scores, mint green for high scores). Keep the "← Score Another Wallet" button.
>
> Required changes:
>
> 1. **Wallet address:** Truncate the middle — show `0xAb58...eC9B` format (first 6 + last 4 chars). Add a copy-to-clipboard icon button next to it.
>
> 2. **Score tier label:** Add a label below the score gauge based on the score value:
>    - 85–100 → "HIGHLY TRUSTED" (mint green)
>    - 65–84 → "TRUSTED" (light green)
>    - 45–64 → "CAUTION" (amber)
>    - 25–44 → "SUSPICIOUS" (orange)
>    - 0–24 → "HIGH RISK" (red)
>
> 3. **Signal section:** Add a section header "Trust Signals" above the signal bars. Each signal bar now shows `weighted/maxWeight` as the score label (from Task 7). Add visual card/panel separation between the score gauge section and the signals section.
>
> 4. **Share button:** Make it larger and more prominent — mint green border, full width or centered with min-width, not a tiny button buried at the bottom.
>
> 5. **Footer:** Add a minimal footer: "© 2026 OpenFlow Labs · openflowlabs.io"
>
> 6. **Report this wallet:** Add a small ghost link below the signals section: "⚑ Report this wallet" — it should do nothing for now (disabled or href="#") but must be present for the WOR system to connect to later.
>
> 7. **Empty space:** The large void below the share button must be eliminated. Use the footer to anchor the bottom.
>
> Keep all CSS in `src/index.css`. Do not add a new component library. Follow existing class naming conventions.

**Definition of done:** Results page looks professional on both mobile (375px) and desktop. Wallet address is truncated. Score tier label is visible. Signal bars show weighted values. Share button is prominent. Footer exists. "Report this wallet" link is present.

---

### TASK 10 — Frontend: API Health Status Indicator ✅
- `src/components/Navbar.tsx` — connected `useHealth` hook, renders 7px status dot (green/red/none) top-right of navbar, with `title` tooltip, `role="status"`, `aria-live="polite"`, sr-only label
- `src/index.css` — navbar flex layout + `.navbar-status-dot` classes (mint green online, red offline, transparent loading) + `.sr-only` utility
- Verified live on Vercel by Manager: green dot visible via screenshot on `otiscore.vercel.app`; deployed JS bundle confirmed to contain "API online"/"API offline" tooltip strings and `navbar-status-dot` classes

---

### TASK 9 — Frontend: Admin Panel UI
**Owner:** Frontend Builder
**Phase:** 2 — Operational
**Priority:** HIGH — required before bots launch (bot traffic generates signups that need managing)
**Depends on:** Task 3 (admin auth), Task 6 (updatedAt migration)

**Full prompt for Frontend Builder:**
> Add a `/admin` route to the React app. It is URL-only — no navigation link anywhere in the app. On load, it shows a simple password prompt (reads `VITE_ADMIN_SECRET` from Vercel environment variables, stored in sessionStorage after entry). Wrong password = locked view.
>
> Screens to build:
> 1. **Dashboard** — total queries today/week/month, active keys count, cache hit rate (from `GET /admin/stats`)
> 2. **API Keys** — table of all keys with plan, usage today, last used, active/suspended status. Actions: create new key (form: email, plan, daily limit), edit (change plan/limit/status), delete. (`GET /admin/keys`, `POST /admin/keys`, `PATCH /admin/keys/:id`, `DELETE /admin/keys/:id`)
> 3. **Query History** — recent wallet lookups: address, chain, score, timestamp (`GET /admin/history`)
> 4. **Cache** — a single "Flush Cache" button (`POST /admin/cache/flush`)
>
> All API calls must include the `x-admin-secret` header (read from sessionStorage). Style consistently with the existing black + mint design system. Plain HTML tables are fine — no table library required.

**Definition of done:** `/admin` route works. Password gates all screens. All 4 screens render real data from the backend. Cache flush button works.

**Frontend status (July 7, 2026):** Frontend Builder shipped this — password gate, all 4 screens, and secure auth design (backend is sole authority on the secret, no client-side comparison). Manager verified live: gate renders correctly on `otiscore.vercel.app/admin`, but `GET /admin/stats`, `/admin/keys`, `/admin/history`, `/admin/cache/flush` all return **404** on the live Railway backend — routes don't exist yet. Blocked on Task 9-BACKEND below.

---

### TASK 9-BACKEND — Backend: Admin Panel API Routes
**Owner:** Backend Builder
**Phase:** 2 — Operational
**Priority:** HIGH — blocks Task 9 (Frontend), which is built and waiting
**Depends on:** Task 3 (admin auth), Task 6 (updatedAt migration)
**Context:** Frontend Builder already shipped the Admin Panel UI (Task 9) calling these exact routes, but none of them exist on the backend yet — all return 404 in production. This task makes them real.

**Full prompt for Backend Builder:**
> Add these 4 routes under the existing `/admin/*` route group (must stay behind `adminAuth` / `x-admin-secret` middleware from Task 3):
>
> 1. `GET /admin/stats` — returns `{ today_requests, week_requests, month_requests, total_keys, active_keys, cache_hit_rate, requests_by_plan: [{ plan, count }] }` (query from wherever request logs / api_keys currently live).
> 2. `GET /admin/keys` — list all API keys: owner/email, plan, daily limit, usage today, last used, active/suspended status, created date, expires date, last4 of the key (never return the full raw key on list).
>    `POST /admin/keys` — create a new key (body: email, plan, daily limit). Return the full raw key value ONCE in the response (frontend shows it once, never again).
>    `PATCH /admin/keys/:id` — edit plan, daily limit, expires_at, active/suspended status.
>    `DELETE /admin/keys/:id` — delete a key.
> 3. `GET /admin/history` — recent wallet lookups: address, chain, score, timestamp. Paginate or cap at a reasonable recent window (e.g. last 500).
> 4. `POST /admin/cache/flush` — flushes the score cache. Return `{ success: true, cleared: number }`.
>
> Also, while touching this code: fix the `seedPlanConfigs()` self-heal logic from Task 7C-BACKEND — it must only insert a default row when a plan is missing entirely from `plan_configs`. It must NEVER overwrite an existing row's `daily_limit`, including an existing `NULL` value — that's an intentional admin-set state now that the Admin Panel lets Ahmad manage it directly.
>
> Add all 4 routes to the OpenAPI spec.

**Definition of done:** All 4 routes return real data with a valid `x-admin-secret` header and 401 without one. Manager verifies live by logging into `/admin` on `otiscore.vercel.app` with the real secret and confirming all 4 screens load data with no errors. `seedPlanConfigs()` no longer overwrites existing rows' `daily_limit`.

**Status: ✅ DONE — verified live by Manager, July 7, 2026.** All 5 routes (`stats`, `keys`, `history`, `cache/flush`, plus pre-existing `usage`/`plan-configs`) confirmed live in production at `https://workspaceapi-server-production-5c0c.up.railway.app/api/admin/*` — 401 without `x-admin-secret`, consistent with the rest of the API's `/api/*` prefix convention. `seedPlanConfigs()` self-heal fix confirmed via code report (no longer overwrites existing rows).

**Follow-up bug found during verification:** The Frontend Builder's Task 9 code calls `/admin/*` (missing the `/api` prefix) — that's why it returned 404 during initial review. This is a frontend-only fix, tracked separately below as Task 9-FRONTEND-FIX.

---

### TASK 9-FRONTEND-FIX — Frontend: Fix Admin API Base Path
**Owner:** Frontend Builder
**Phase:** 2 — Operational
**Priority:** HIGH — blocks Task 9 from working at all
**Depends on:** Task 9-BACKEND (done)
**Context:** Backend admin routes are live and correct at `/api/admin/*` (consistent with `/api/score`, `/api/healthz`, etc.). The Task 9 admin panel frontend code calls `/admin/*` — missing the `/api` prefix — which is why every screen 404'd during Manager review.

**Full prompt for Frontend Builder:**
> In `src/lib/adminClient.ts` (and anywhere else admin routes are called), change the base path so all admin requests go to `/api/admin/*` instead of `/admin/*`. Confirm `/admin/stats`, `/admin/keys`, `/admin/history`, `/admin/cache/flush`, `/admin/usage`, and `/admin/plan-configs` (if referenced) all use the `/api/admin/` prefix.

**Definition of done:** Logging into `/admin` on `otiscore.vercel.app` with the real secret loads real data on all 4 screens with no errors.

---

### TASK 10 — Frontend: API Health Status Indicator
**Owner:** Frontend Builder
**Phase:** 2 — Operational
**Priority:** LOW — quick win, already built
**Depends on:** Nothing (the hook already exists)

**Full prompt for Frontend Builder:**
> The `src/hooks/useHealth.ts` hook already exists and pings `GET /api/healthz`. Connect it to a small status indicator in the Navbar (`src/components/Navbar.tsx`).
>
> Display: a small colored dot (6–8px circle) in the top-right of the navbar.
> - Green dot = API is responding (`status: "ok"`)
> - Red dot = API is unreachable (error state)
> - No dot / pulsing = loading
>
> Tooltip on hover: "API online" or "API offline". Keep it subtle — it should not distract from the main content.

**Definition of done:** Navbar shows a green dot when Railway API is up, red when it's down. Verified by checking the deployed frontend.

---

### TASK 11A — Restructure Vercel App: Marketing Front Door + Scoring at /score
**Owner:** Frontend Builder
**Phase:** 4 — Pre-Distribution (first priority in this phase)
**Priority:** HIGH — the current Vercel URL is impossibly long; the scoring tool has no marketing context for new visitors
**Depends on:** Task 8 (results page redesign must be done — the scoring tool must look professional before marketing traffic sees it), Task 2B (SVG logo)

**Context for the Builder:**
The current Vercel app IS the front door. We are NOT building a second site. The current app gets a new marketing homepage at `/`, and the scoring tool moves to `/score`. One Vercel project, one URL, two purposes. The Vercel URL is already `otiscore.vercel.app` ✅ — Ahmad confirmed this on July 5, 2026. The domain rename is done. The Builder does not need to change anything about the URL.

---

**Part A — Move the scoring tool to `/score`**

Move the current homepage (wallet address form + results) to the `/score` route. Update `vercel.json` carefully — the SPA rewrite rule `{"source":"/(.*)", "destination":"/index.html"}` must stay intact; do not remove or break it. Update any internal links that currently point to `/`.

---

**Part B — Build the marketing homepage at `/`**

**Brand system — everything derives from the OTI spiral logo:**
- Primary mint: `#3EFFC1` (outer/bright), `#2BD9A4` (inner/deeper)
- Background: `#0A0A0A` deep black
- Typography: Geist Sans or Inter — clean, geometric, modern
- Micro-interaction: the spiral logo rotates subtly on hover (CSS `transform: rotate()`, 2–3s ease, infinite)
- All CTAs, highlights, active states: mint only — no other accent color

**Logo:** Current `logo.jpg` is blurry — Task 2B replaces it with `logo.svg`. Task 11A should depend on Task 2B being merged first. If Task 2B is not yet done, use the JPG as a placeholder with a code comment.

**Sections to build:**

**1. Navbar**
- Left: OTI spiral logo (34px) + "OTI" wordmark in white
- Right: "Score a Wallet" button (mint outline) → `/score`; "API Docs" text link → placeholder `#`
- Mobile: hamburger with same links

**2. Hero**
- Headline: "Know Who You're Transacting With"
- Sub-headline: "OTI scores any blockchain wallet 0–100 using five on-chain trust signals. Instant. Free. API-ready."
- CTAs: "Try It Free" (mint filled → `/score`), "View API Docs" (ghost → `#`)
- Faint watermark spiral behind headline (CSS only, no JS library)
- Chain row below CTAs: small chain icons for ETH, SOL, BTC, TON, MATIC, ARB, SUI, TRX + "+7 more" badge

**3. How It Works (3 steps)**
- Step 1: Enter wallet address + select chain
- Step 2: OTI analyzes 5 on-chain behavioral signals
- Step 3: Get a 0–100 trust score with full signal breakdown
- Style: numbered cards, horizontal row on desktop, vertical stack on mobile

**4. Trust Signals (5 cards)**
- 🕐 Wallet Age (25%) — "How long this wallet has been active on-chain"
- 📊 Transaction Count (20%) — "Volume and frequency of transactions"
- 🪙 Token Holding Behavior (20%) — "Diversity and quality of tokens held"
- 🔗 Smart Contract Interactions (20%) — "Depth of DeFi and protocol engagement"
- ⏱ Transaction Timing (15%) — "Consistency and naturalness of activity patterns"

**5. Use Cases (tile grid — 3 cols desktop, 2 tablet, 1 mobile)**

| Tile | Sector | Body |
|---|---|---|
| 💱 | Exchanges & Gateways | Flag compromised wallets before processing withdrawals. Know the behavioral history of any address instantly. |
| 🏦 | DeFi Protocols | Risk-adjust lending and collateral requirements based on wallet trust. Score borrowers before approving credit. |
| 🖼 | NFT Marketplaces | Display trust badges next to seller listings. Filter suspicious actors before they transact. |
| 💸 | Payment Processors | Require a minimum trust score before processing. Reduce fraud without identity verification. |
| 🎮 | Web3 Gaming | Prevent fresh-wallet farming in Play-to-Earn. Verify behavioral legitimacy before distributing rewards. |
| 🗳 | DAO Governance | Weight votes by wallet trust alongside token balance. Reduce governance attacks from newly created wallets. |
| 🔐 | Custody Services | Score source wallets before crediting accounts. Flag wallets with suspicious timing patterns. |
| 📡 | Bridges & Cross-chain | Score the source wallet before allowing a bridge transaction. Stop compromised wallets at the gate. |
| 🛠 | Developer Tooling | One API call. Any chain. Embed behavioral trust scores in your product with no blockchain infrastructure required. |

**6. Get the API**
- Headline: "Free API Key. No Credit Card."
- Sub-line: "Anonymous: 3 lookups/day. Register free for higher limits."
- cURL example block (copy-pasteable)
- CTA: "Read the Docs" → `#` placeholder

**7. Find Us / Integrations**
Row of icons with labels: Telegram, Discord, Chrome Extension, Firefox Extension — all `#` placeholder links until Phase 5 channels are live.

**8. Footer**
- Left: OTI spiral logo (small) + "© 2026 OpenFlow Labs"
- Center: Score a Wallet · API Docs · GitHub (link to public frontend repo) · Privacy Policy (placeholder) · Terms (placeholder)
- Right: Social icons — Twitter/X, LinkedIn, Telegram, Discord (all `#` until accounts exist)

**Chat support — Crisp.chat:**
Add to `index.html` `<head>`:
```html
<script>window.$crisp=[];window.CRISP_WEBSITE_ID="AHMAD_PROVIDES_THIS";</script>
<script src="https://client.crisp.chat/l.js" async></script>
```
Ahmad signs up at crisp.chat (free), creates a workspace, copies the Website ID (short alphanumeric string), provides it to the Builder. If not yet provided, leave `CRISP_WEBSITE_ID=""` — widget simply won't appear until filled.

**Feedback form — Tally.so:**
Ahmad creates a form at tally.so (free): fields = Name (optional), Email (optional), Type dropdown (Bug / Feature Request / Sales / Other), Message. Ahmad provides the embed snippet. Place it in a modal triggered by "Send Feedback" in the footer. If not yet provided, use `mailto:` fallback.

---

**Definition of done:**
- `/` shows the full marketing homepage — all 8 sections present and styled
- `/score` shows the scoring tool exactly as before (no visual changes to the tool itself)
- Navbar "Score a Wallet" navigates correctly between the two views
- Crisp script is in `<head>` (even if ID is empty placeholder)
- Responsive on mobile (375px) and desktop (1440px)
- Brand system (`#0A0A0A` + `#3EFFC1` mint) is consistent throughout

---

### TASK 11B — Whitepaper Page
**Owner:** Frontend Builder
**Phase:** 4 — Pre-Distribution
**Priority:** HIGH — builds credibility with investors, enterprise partners, and serious developers
**Depends on:** Task 11A (lives inside the same Vercel project)

**Route:** `/whitepaper`

**Nav update required** (update the navbar from Task 11A):
- Desktop: `Logo | Score a Wallet | API Docs | Whitepaper | [Social icons]`
- Mobile hamburger: Score a Wallet → API Docs → Whitepaper → [Social icons]

---

**Design rules:**
- Same `#0A0A0A` black + `#3EFFC1` mint brand system as the rest of the site
- Typography: Geist Sans or Inter. Body text slightly larger than normal (17–18px), high line-height (1.8) for readability — this is a long-form document
- Section numbers are displayed in mint: `01`, `02`, `03` etc.
- Sticky table of contents sidebar on desktop (collapses to a "Jump to section" accordion on mobile)
- Faint spiral logo watermark centered behind the page header — `opacity: 0.04`, CSS only
- "Download as PDF" button in the page header — uses `window.print()` with a `@media print` stylesheet that hides the navbar, sidebar, and footer and formats the content as clean A4

---

**Full whitepaper content — copy this EXACTLY. Do not paraphrase or summarize.**

---

**PAGE HEADER**
Title: `OpenFlow Trust Infrastructure — Whitepaper`
Subtitle: `Version 1.0 · July 2026 · OpenFlow Labs`
Two buttons: `[Download PDF]` and `[Score a Wallet →]` (links to `/score`)

---

**01 — Executive Summary**

OpenFlow Trust Infrastructure (OTI) is an independent on-chain trust verification platform that enables any application, protocol, exchange, or marketplace to instantly assess the behavioral trustworthiness of any blockchain wallet address. Given a wallet address and a supported chain, OTI returns a deterministic 0–100 trust score derived from five on-chain behavioral signals — without requiring identity disclosure, KYC, or off-chain data.

OTI is the first public product developed by OpenFlow Labs, the research and engineering division of OpenFlow, a Nigerian technology company building the foundational infrastructure for the emerging attention economy. OTI is intentionally designed as an independent infrastructure business, serving external developers, protocols, DAOs, marketplaces, and enterprises, while simultaneously acting as the trust verification layer that powers OpenFlow's broader ecosystem.

---

**02 — About OpenFlow**

OpenFlow is a technology company based in Nigeria, focused on building the next generation of digital trust and attention infrastructure for Web3 and beyond. Our mission is to transform attention into a verifiable economic asset by creating systems where value is exchanged only after trust has been established.

Rather than building a single application, OpenFlow is developing an ecosystem of interoperable technologies that enable businesses, creators, advertisers, protocols, and communities to measure, verify, and monetize genuine human participation. Through infrastructure, economic design, and open standards, OpenFlow aims to become a foundational layer for the emerging attention economy.

---

**03 — About OpenFlow Labs**

OpenFlow Labs is the research and engineering division of OpenFlow. It is responsible for designing, developing, and maintaining the technologies that power the OpenFlow ecosystem.

The first product developed by OpenFlow Labs is OTI — an independent trust verification platform designed to help applications distinguish genuine users from bots, Sybil attackers, and low-quality engagement through verifiable on-chain signals.

Although OTI is the first public product, it is only the beginning. OpenFlow Labs is building a portfolio of interconnected infrastructure products that will support the OpenFlow ecosystem over the coming years, including developer tools, trust services, attention verification technologies, reputation systems, APIs, SDKs, and future infrastructure for decentralized commerce and digital participation.

OTI is intentionally designed as an independent infrastructure business. It serves external developers, protocols, DAOs, marketplaces, and enterprises while simultaneously acting as the trust layer that powers OpenFlow itself. This separation allows OTI to generate sustainable revenue from external customers, creating a cross-subsidization model that funds long-term research, infrastructure development, and future OpenFlow products without relying solely on token financing.

---

**04 — The Problem**

The internet is entering a new era where attention is one of the world's most valuable digital assets. But before attention can become valuable, it must first become trustworthy.

Today, billions of dollars are lost every year to bots, fake engagement, Sybil attacks, and unverifiable digital interactions. Existing systems measure activity — but they rarely verify authenticity.

In Web3 specifically, this problem is acute:

- Exchanges process withdrawal transactions to wallets they know nothing about
- DeFi protocols extend credit to wallets with no behavioral history
- NFT marketplaces display listings from wallets created minutes ago
- DAOs accept governance votes from freshly funded addresses
- P2P traders send funds to counterparties they cannot verify
- Launchpads are sybil-attacked by thousands of coordinated fresh wallets

Every one of these platforms faces the same fundamental question: **"Is this interaction genuine?"**

The Web3 ecosystem currently has no open, developer-accessible, chain-agnostic infrastructure to answer that question — until now.

---

**05 — How OTI Works**

OTI analyzes five on-chain behavioral signals for any wallet address on any supported chain. Each signal is scored 0–100, then weighted by its contribution to produce a final overall trust score.

| Signal | Weight | What It Measures |
|---|---|---|
| Wallet Age | 25% | How long this wallet has been active on-chain |
| Transaction Count | 20% | Volume and frequency of lifetime transactions |
| Token Holding Behavior | 20% | Diversity and quality of tokens held over time |
| Smart Contract Interactions | 20% | Depth of DeFi and protocol engagement |
| Transaction Timing Patterns | 15% | Consistency and naturalness of activity over time |

**Scoring formula:**
`Overall Score = (WalletAge × 0.25) + (TxCount × 0.20) + (TokenHolding × 0.20) + (ContractInteractions × 0.20) + (TimingPatterns × 0.15)`

**Trust tiers:**

| Score | Label | Meaning |
|---|---|---|
| 85 – 100 | HIGHLY TRUSTED | Long-established, organically active wallet |
| 65 – 84 | TRUSTED | Solid behavioral history, low risk |
| 45 – 64 | CAUTION | Moderate history, proceed carefully |
| 25 – 44 | SUSPICIOUS | Thin history or unusual patterns |
| 0 – 24 | HIGH RISK | New, inactive, or behaviorally anomalous |

All scoring is performed server-side using on-chain data only. No personally identifiable information is collected or required. All results are deterministic and reproducible.

---

**06 — Supported Infrastructure**

OTI currently supports 15 blockchain networks:

**EVM chains (via Etherscan V2 API):**
Ethereum, Polygon, Arbitrum, Avalanche, Fantom, Linea, Scroll, zkSync, Sepolia, Holesky

**Non-EVM chains:**
Solana, TON, Bitcoin, Sui, Tron

Three additional high-volume chains — BNB Smart Chain, Base, and Optimism — are temporarily unavailable pending an infrastructure upgrade. Full coverage across all 18 networks is planned in a near-term release.

---

**07 — Use Cases**

OTI is applicable across every vertical in Web3 where trust between transacting parties matters.

**Exchanges & Gateways**
Screen withdrawal destinations before processing. Detect compromised wallets before funds leave the platform. Know the behavioral history of any recipient address in milliseconds.

**DeFi Protocols**
Risk-adjust collateral requirements and borrowing limits based on wallet trust. Score counterparties before executing credit decisions. Reduce protocol exposure to thin-history or bot-controlled wallets.

**NFT Marketplaces**
Display verifiable trust badges alongside seller listings. Filter out listings from wallets created hours before a sale. Give buyers confidence about the party on the other side of a high-value transaction.

**Payment Processors**
Require a minimum trust score before processing outbound crypto payments. Reduce fraud exposure without requiring identity documents or KYC workflows.

**Web3 Gaming**
Prevent Sybil attacks and reward farming in Play-to-Earn ecosystems. Verify wallet behavioral legitimacy before distributing tokens or prizes. Maintain fair economic environments for genuine players.

**DAO Governance**
Supplement token-based voting with trust-weighted participation. Reduce governance attacks from freshly funded wallets. Reward long-term ecosystem participants with governance influence proportional to their verifiable history.

**Custody Services**
Score incoming transfer sources before crediting accounts. Flag wallets showing anomalous timing or concentration patterns. Add a behavioral layer to existing compliance workflows.

**Cross-Chain Bridges**
Score the source wallet before allowing a bridge transaction. Stop compromised or high-risk wallets at the bridge gate rather than after funds have moved.

**Developer Tooling & Embeds**
Any Web3 product — portfolio trackers, wallet browsers, analytics dashboards, on-chain explorers — can embed OTI trust scores with a single API call. No blockchain infrastructure required on the integrator's side.

---

**08 — The Wallet Ownership Registry (WOR)**

The Wallet Ownership Registry is OTI's planned flagship trust feature — a system with no equivalent in the market.

**The problem:** When a blockchain wallet is compromised, the attacker has copied the private key. This means the original owner and the attacker both have identical signing authority. Simply connecting a wallet or signing a message does not prove you are the original owner — the attacker can do the same.

**The OTI solution — pre-registration with a passkey:**

1. A wallet owner connects their wallet to OTI before any compromise occurs
2. OTI generates a cryptographic challenge
3. The owner signs off-chain (EIP-191, zero gas cost)
4. OTI verifies the signature and records the registration
5. The owner sets a secret passkey — stored as a hash, never in plain text

If the wallet is later compromised, the original owner can submit a compromise report by connecting the wallet AND entering the pre-registered passkey. OTI verifies both — the wallet signature (which the attacker can also do) AND the passkey hash match (which the attacker cannot replicate, because they were not present for the pre-registration).

On successful verification: the wallet is immediately flagged as `compromised: true`. Its trust score drops to 0. Every API consumer using OTI is instantly aware. No admin review. No blockchain transaction. Fully automated. Real-time.

This is the only system in Web3 where a wallet owner can definitively differentiate themselves from an attacker who holds the same private key.

---

**09 — Distribution**

OTI reaches developers through four zero-infrastructure distribution channels, all planned for post-MVP launch:

**Telegram Bot** — Community users can query any wallet score directly in Telegram groups or DMs. Every bot reply carries OTI branding. Group mode is the primary viral lever: a single reply in a public crypto community reaches hundreds of viewers simultaneously.

**Discord Bot** — Slash command integration for Web3 Discord servers. Rich embed responses with score breakdown, signal bars, and trust tier. DeFi and NFT communities are the primary target — exactly the integrations we want to reach.

**Embeddable Widget** — A single-line JavaScript embed for any website. Site operators drop one `<script>` tag and OTI renders a live trust badge next to any wallet address. Target platforms: NFT marketplaces, OTC trading boards, blockchain analytics sites, portfolio trackers.

**Browser Extension** — A Chrome and Firefox extension that automatically detects wallet addresses on any webpage (Etherscan, OpenSea, BscScan, Blur, and beyond) and injects OTI trust badges inline. Zero effort from the user after installation.

**The conversion funnel:**
Community user sees OTI score in a bot reply → shares the score card (every share carries OTI branding) → developer discovers OTI → visits developer docs → gets a free API key → integrates into their product → product grows → becomes a paying business client → enterprise becomes a direct sales conversation.

---

**10 — Revenue Model**

OTI operates a tiered API access model designed to convert community users into paying business clients over time.

| Tier | Access | Limit | Price |
|---|---|---|---|
| Anonymous | No key required | 3 lookups/day | Free |
| Developer | API key | Higher daily limit | Free |
| Pro | API key | High volume | Paid (planned) |
| Enterprise | Direct contract | Unlimited + SLA | Custom pricing |

Infrastructure revenue generated by OTI API subscriptions creates a sustainable recurring income model. This cross-subsidizes OpenFlow Labs research and future OpenFlow ecosystem products, reducing dependence on token financing at early stages.

The highest-value client category is **exchanges and payment gateways** — institutions that process millions of transactions and face the greatest exposure from unverified counterparties. OTI's enterprise offering for this segment is the primary path to significant revenue.

---

**11 — The OpenFlow Vision**

We believe the internet is entering a new era where attention becomes one of the world's most valuable digital assets. However, before attention can become valuable, it must first become trustworthy.

OTI is the foundation of a larger vision: **the OpenFlow Attention Economy**.

The OpenFlow Attention Economy introduces a new economic model where verified attention becomes an asset that can be measured, priced, exchanged, and rewarded across multiple ecosystems. Rather than treating attention as disposable advertising inventory, OpenFlow transforms genuine participation into economic value through transparent verification and sustainable economic design.

OpenFlow combines two complementary models to support this vision:

**Infrastructure Revenue** — generated by products such as OTI, creating sustainable recurring income through enterprise APIs, verification services, and developer tools. This funds operations and research independently of token markets.

**The OpenFlow Economy** — powered by the FLOW ecosystem, which aligns participants through decentralized incentives and ecosystem growth.

As adoption grows, OpenFlow plans to introduce both private and public token offerings to expand ecosystem participation and accelerate global development. Those fundraising events are intended to strengthen an already functioning ecosystem — not finance an idea. Our objective is to demonstrate real products, real users, and sustainable infrastructure before inviting broader community ownership.

Ultimately, our ambition is not simply to build another Web3 platform, but to establish the global infrastructure where trust enables value, and verified attention powers a new digital economy.

---

**12 — Roadmap**

*Near-term (MVP phase — in progress):*
- Admin endpoint security
- Score history served from database
- Weighted signal scores in API response
- Professional results page redesign

*Short-term (pre-distribution):*
- Marketing homepage and Whitepaper page (this document)
- Developer documentation site
- Admin management panel

*Medium-term (distribution):*
- Telegram bot
- Discord bot
- Embeddable JavaScript widget
- Browser extension (Firefox first, then Chrome)

*Long-term:*
- Wallet Ownership Registry (WOR) — self-serve compromise reporting
- Developer self-serve monetization portal
- Pro and Enterprise plan tiers
- FLOW ecosystem integration
- BSC, Base, and Optimism chain support unlock

---

**13 — Team**

**Ahmad Alhassan (Dan Shila)**
*Co-Founder & Chief Executive Officer*

Ahmad Alhassan is a technology entrepreneur and systems designer from Nigeria with a strong passion for building long-term digital infrastructure. He leads the vision, architecture, product strategy, and economic design behind OpenFlow and OTI. His work focuses on solving fundamental problems of trust, attention verification, and sustainable value creation across decentralized ecosystems.

**Mustapha Abdullahi (Musty)**
*Co-Founder*

Mustapha is the co-founder of OpenFlow and works alongside Ahmad in building the long-term vision of the ecosystem. Together they are focused on creating infrastructure that enables trustworthy digital participation and supports the growth of the OpenFlow Attention Economy.

---

**14 — Contact & Links**

| Resource | Link |
|---|---|
| Score a Wallet | `https://otiscore.vercel.app/score` |
| API Documentation | `https://docs.otiscore.vercel.app` (use `#` with "Coming soon" tooltip until Task 11 is live) |
| GitHub | `[Ahmad to provide — public frontend repo URL]` |
| Telegram Bot | `[Ahmad to provide — Telegram bot invite link, available after Task 12 ships]` (use `#` + "Coming soon" tooltip until then) |
| Discord Bot | `[Ahmad to provide — Discord bot invite link, available after Task 13 ships]` (use `#` + "Coming soon" tooltip until then) |
| Email | `[Ahmad to provide — company email address]` |
| Twitter / X | `[Ahmad to provide — Twitter/X profile link]` |
| Crisp Chat | Embedded widget (bottom right of every page) |

---

**Definition of done:**
- `/whitepaper` route is live, all 14 sections present with exact content above
- Sticky TOC sidebar works on desktop; collapses on mobile
- "Download PDF" button uses `window.print()` with clean `@media print` stylesheet
- Navbar updated across the whole site to include "Whitepaper" as the last item before social icons
- Faint spiral watermark in page header
- Fully responsive on mobile (375px) and desktop (1440px)

### TASK 11 — Developer Docs Site
**Owner:** Frontend Builder
**Phase:** 4 — Pre-Distribution
**Priority:** HIGH — hard dependency before any bot or widget launches
**Depends on:** Task 5 (weighted signal response must be finalized before documenting the API shape)

**Full prompt for Frontend Builder:**
> Set up a Docusaurus documentation site for OTI. Deploy it to Vercel on the same account as the frontend (Vercel auto-detects Docusaurus — zero config needed).
>
> ```bash
> npx create-docusaurus@latest oti-docs classic
> ```
>
> The docs must cover (in order of priority):
> 1. **Getting Started (CRITICAL)** — What OTI is in one paragraph. How to get a free API key. One copy-paste cURL example that works immediately. Link to API Reference.
> 2. **API Reference (CRITICAL)** — Every endpoint, every parameter, every response field (use the new weighted signal shape from Task 5), all error codes.
> 3. **Score Explanation (CRITICAL)** — What 0–100 means. What each signal measures. The 5 tier labels (HIGHLY TRUSTED / TRUSTED / CAUTION / SUSPICIOUS / HIGH RISK). How weighted contributions work.
> 4. **Supported Chains (IMPORTANT)** — All 15 chains, chain IDs, limitations (BSC/Base/Optimism = 503, txCount cap at 1,000).
> 5. **Rate Limits & Plans (IMPORTANT)** — Anonymous = 3/day, Free plan limits, how to upgrade.
> 6. **Code Examples (IMPORTANT)** — JavaScript (fetch), Python (requests), cURL — all copy-paste working examples.
>
> Style: keep OTI branding. The site footer should say "OpenFlow Labs · openflowlabs.io".

**Definition of done:** Docusaurus site is live on Vercel. Getting Started page works — a developer can read it and make their first API call within 5 minutes.

---

---
## ⛔ PHASE 4 GATE — All items below depend on this being complete first

Before any distribution channel (Tasks 12–15) is assigned, confirm ALL of the following are done:
- [x] Task 3 — Admin auth live on Railway ✅
- [x] Task 5 — Weighted signal response in API ✅
- [ ] Task 9 — Admin panel UI live on Vercel
- [ ] Task 11A — Marketing homepage live at `/`, scoring tool at `/score`, Vercel project renamed to `oti`
- [ ] Task 11 — Developer docs site live on Vercel
- [ ] Ahmad: Rename Vercel project to `oti` in dashboard (Project Settings → General → Project Name)
- [ ] Ahmad: Sign up for Crisp.chat (free), provide Website ID to Frontend Builder
- [ ] Ahmad: Create Tally.so feedback form (free), provide embed snippet to Frontend Builder
- [ ] Internal bot API key created via admin panel (Ahmad does this)
- [ ] Widget shared key created via admin panel (Ahmad does this)

**No distribution task starts until every item above is checked.**

---

### TASK 12 — Telegram Bot
**Owner:** Backend Builder
**Phase:** 5 — Distribution (first channel)
**Priority:** High
**Depends on:** Phase 4 Gate fully complete (all items above checked)

See ROADMAP.md Phase 5, Channel 1 for full technical spec, file structure, deployment instructions, and environment variables needed.

---

### TASK 13 — Discord Bot
**Owner:** Backend Builder
**Phase:** 5 — Distribution (second channel)
**Priority:** High
**Depends on:** Phase 4 Gate fully complete. Can be built in parallel with Task 12 if Backend Builder capacity allows — both are standalone processes and do not conflict with each other.

See ROADMAP.md Phase 5, Channel 2 for full technical spec.

---

### TASK 11C — Backend: Signal Accuracy Audit & Cross-Chain Fix
**Owner:** Backend Builder
**Phase:** Pre-Distribution (must be done before Phase 5 launches)
**Priority:** CRITICAL — signals are the core product; wrong data destroys trust
**Depends on:** Task 11B (do this last in pre-distribution, signals must be accurate before bots drive traffic)

**Context:**
The 5 scoring signals were designed with EVM chains in mind. Bitcoin, Solana, TON, Tron, and Sui have fundamentally different data models — no ERC-20 tokens, no internal transactions, no smart contracts in the EVM sense. Currently these non-EVM chains are being scored using EVM-specific logic, producing wrong and misleading results. The Satoshi genesis wallet (1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa) scored 51 days wallet age when the correct answer is ~5,700+ days. Smart contract interactions showing "1 smart-contract tx" on a Bitcoin wallet that has never touched a smart contract. Token holding showing 0/20 for Bitcoin wallets because ERC-20 tokens don't exist on Bitcoin. These are not edge cases — they affect every single non-EVM wallet scored by OTI.

**What to audit and fix — per signal:**

**Signal 1 — Wallet Age (25%)**
- EVM: first transaction timestamp from Etherscan — correct
- Bitcoin: first tx timestamp from mempool.space — Task 7D attempted a fix but the Satoshi genesis wallet still shows 51 days (CACHED — may be a stale cache). Re-verify Task 7D's fix against the genesis wallet after cache flush. If still wrong, re-investigate.
- Solana: verify first tx timestamp is being pulled correctly
- TON, Tron, Sui: verify each one individually

**Signal 2 — Transaction Count (20%)**
- EVM: Etherscan tx count, capped at 1,000 — acceptable
- Non-EVM: verify each chain is returning real tx count, not a placeholder or default

**Signal 3 — Token Holding Behavior (20%)**
- EVM: ERC-20 token diversity — correct
- Bitcoin: has NO tokens — currently returning 0/20 which unfairly penalises BTC wallets. Fix: if chain is Bitcoin, skip this signal and redistribute its weight proportionally across the other 4 signals, OR score it as neutral (50/100) to avoid penalising wallets for a chain limitation.
- Solana: has SPL tokens — ensure SPL token diversity is being scored, not defaulting to 0
- TON, Tron, Sui: verify token data source and scoring for each

**Signal 4 — Smart Contract Interactions (20%)**
- EVM: internal tx count from Etherscan — correct
- Bitcoin: has NO smart contracts — currently returning fabricated data ("1 smart-contract tx"). Fix: same approach as Token Holding — redistribute weight or score neutral.
- Solana: program interactions are the equivalent of contract interactions — verify this is being scored correctly
- TON, Tron, Sui: verify each

**Signal 5 — Transaction Timing Patterns (15%)**
- EVM: uses internal transaction timestamps — subtitle shows "0 internal transactions" for non-EVM chains
- Non-EVM: use actual transaction timestamps for timing analysis. Do not default to 0 or use internal tx counts as a proxy.

**Definition of done:**
- The Satoshi genesis wallet scores wallet age > 5,000 days
- No signal shows fabricated data for chains where that data type doesn't exist
- Non-applicable signals are either redistributed or scored neutral — never 0 by default
- All 15 supported chains tested with at least one known wallet
- Results documented in a short test log committed to the repo

---

### TASK 14 — Embeddable Widget
**Owner:** Backend Builder
**Phase:** 5 — Distribution (third channel)
**Priority:** High
**Depends on:** Phase 4 Gate fully complete (widget key needed + docs site must exist for the attribution link)

See ROADMAP.md Phase 5, Channel 3 for full technical spec.

---

### TASK 15 — Firefox Extension
**Owner:** Backend Builder (separate GitHub repo: `oti-firefox-extension`)
**Phase:** 5 — Distribution (fourth channel)
**Priority:** Medium
**Depends on:** Phase 4 Gate fully complete. This is a separate repo — create it fresh, no relation to the backend repo.

See ROADMAP.md Phase 5, Channel 4 for full technical spec.
