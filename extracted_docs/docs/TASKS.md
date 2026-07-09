# OTI — Master Task Queue
> Last updated: July 8, 2026 (session 4 — Task 11B live but not done, 3 fixes sent to Frontend Builder; Task 11C sent to Backend Builder, awaiting their work) | Maintained by: Development Manager
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
| Frontend Builder | New account (July 8, 2026) — previous account hit credit limit, new account imported | Task 11B ✅ DONE — Whitepaper live at `/whitepaper`, all 3 fixes confirmed (body text white, mobile horizontal scroll eliminated, Roadmap section removed + renumbered). Idle, awaiting next task. |
| Backend Builder | Active | Task 9C done ✅. Task 11C (Signal Accuracy Audit — CRITICAL) sent, in progress. |

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

### Task 7C-BACKEND — Public Anonymous Rate Limit Endpoint ✅ (Backend Builder — July 7, 2026)
- `GET /api/config/anonymous-limit` — public endpoint, no auth required
- Returns `{ "daily_limit": number | null }` from plan_configs table
- `seedPlanConfigs()` fixed: no longer overwrites existing rows — only seeds when row is missing entirely
- Verified: endpoint returns live DB value immediately after Admin Panel PATCH

### Task 7C — Frontend: Dynamic Rate Limit Display ✅ (Frontend Builder — July 7, 2026)
- `src/hooks/useAnonymousLimit.ts` calls `GET /api/config/anonymous-limit`
- Homepage confirmed showing "Anonymous lookups are limited to 20 per day" (July 7, 2026)
- Falls back gracefully when API returns null
- Hook invalidates on Plan Configs save so homepage updates immediately

### Fix: API Keys Tab — Full Resolution ✅ (Backend Builder — July 7, 2026)
- Root cause: Railway subscriptions table predates Drizzle schema — real columns confirmed via information_schema
- Real columns: id, api_key, plan, owner_address, created_at, expires_at, updated_at (NO status, NO email column)
- GET fixed: raw SQL SELECT * with column-name fallback mapper; no WHERE filter on status
- POST fixed: INSERT (api_key, plan, owner_address) only
- Stats active_keys fixed: removed WHERE status = 'active' (column does not exist); counts all rows
- Verified live: Ahmad created enterprise, pro, and free keys at 14:15 July 7, 2026 — Edit and Delete confirmed working

### Fix: Admin Panel API Keys UI Resilience ✅ (Frontend Builder — July 7, 2026)
- "+ New Key" button always visible even when keys list fails to load
- Error shows inline with Retry button; table guarded behind isSuccess

---

### TASK 9C — Backend: Verify & Harden Plan Limit Enforcement ✅
**Owner:** Backend Builder | **Completed:** July 7, 2026

Critical production bug found and fixed during testing — every API request using a valid key was silently returning HTML 500 since launch due to Drizzle ORM schema mismatch (status column defined in schema but missing from Railway DB).

**Fixes applied:**
- `apiKeyAuth.ts` — replaced Drizzle ORM select with raw SQL `SELECT *`; missing columns no longer crash the middleware
- `score.ts` — wrapped compromised-wallets denylist check in try/catch; DB hiccup now returns JSON not HTML 500
- `admin.ts` DELETE `/admin/keys/:id` — replaced Drizzle `.delete().returning()` with raw SQL
- `admin.ts` PATCH `/admin/keys/:id` — replaced Drizzle `.update().set().returning()` with raw SQL via pg pool, status column excluded

**429 test results (live Railway):**
- Free plan (daily_limit=2): Request 1 → 200 ✅, Request 2 → 200 ✅, Request 3 → 429 ✅
- Enterprise (daily_limit=null): 5/5 requests → 200, never 429 ✅
- PATCH edit confirmed → HTTP 200 + updated_at timestamp updated ✅

---

## 🔴 Queue — Not Started (Build In This Exact Order)

---

### TASK 8 — Frontend: Professional Results Page Redesign ✅
**Owner:** Frontend Builder | **Completed:** July 7, 2026

- Score panel in its own bordered card — ring gauge color matches chain brand color (all 15 chains have exact brand hex)
- Score number color reflects trust tier; tier label shown beneath gauge: HIGHLY TRUSTED / TRUSTED / CAUTION / SUSPICIOUS / HIGH RISK
- Trust Signals in separate bordered card with "Trust Signals" heading; each signal shows label, metadata, weighted fraction, colored bar
- Wallet address truncated (0xAb58...eC9B) with one-click copy button; chain icon + name displayed above
- Share button — native OS share sheet (navigator.share) with clipboard fallback
- Save as Image — 3× scale PNG (1920×2580px), chain-color ring arc, tier label, mirrors live UI exactly
- "⚑ Report this wallet" ghost link in mint — placeholder for WOR system (Phase 6)
- Footer: "© 2026 OpenFlow Labs · openflowlabs.io"
- Full color theme upgrade to OTI deep-space palette (see Color System below)
- Verified live on Vercel by Manager — July 7, 2026

---

### OTI COLOR SYSTEM — LOCKED July 7, 2026 (All future tasks must use these values)

| Token | Value | Usage |
|---|---|---|
| Background | `#05080f` | Page background |
| Surface | `#0b0f1a` | Card/panel backgrounds |
| Surface-2 | `#0f1520` | Inner elements, bar tracks |
| Borders | `#1c2535` | All card/panel borders |
| Body text | `#e8f4ff` | Main text |
| Dimmed text | `#7a8fa8` | Metadata, secondary labels |
| Mint primary | `#00e5a0` | CTAs, highlights, active states |
| Mint gradient | `#3EFFC1` | Gradient highlights |

Chain brand colors: Ethereum `#627EEA` · Bitcoin `#F7931A` · Solana `#9945FF` · BNB `#F3BA2F` (all 15 in codebase)
Special effects: navbar frosted glass (`backdrop-filter: blur(14px)`), submit button mint glow, score panel chain-color tint via `color-mix()`, all with plain-value fallbacks for older browsers.

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

**Status: ✅ DONE — verified live by Manager, July 7, 2026.** Password gate, all 4 screens (Dashboard, API Keys, Query History, Cache, Plan Configs), and secure auth design confirmed working on `otiscore.vercel.app/admin`. All screens load real data from the backend.

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

**Status: ✅ DONE — verified live by Manager, July 7, 2026.** All admin API calls use `/api/admin/*` prefix. All 4 screens load real data with no errors.

---

### TASK 8B — Frontend: Professional Wallet Input Page Redesign ✅
**Completed:** July 8, 2026, incl. one polish round (logo size/position, zkSync/Linea icon visibility, chain icon size, spacing, report-link styling). Verified live by Manager via screenshot.

---

### TASK 8C — Frontend: Fix Anonymous Rate Limit Cache Sync Bug ✅
**Completed:** July 8, 2026. Real root cause found on the second audit pass: `setEditId(null)` inside the mutation's `onSuccess` raced with React 18 batching and unmounted the edit row before the success banner/cache update could be trusted. Fixed by keeping the row open until the user clicks "Done". Verified live on Vercel by Ahmad — anonymous limit change to 42 updated the homepage badge instantly, and unlimited (blank field) correctly showed "Unlimited".

---

### TASK 8D — Frontend: Homepage Visual Polish (Contrast, Animated CTA, Spacing, Density) ✅
**Completed:** July 8, 2026. Placeholder contrast fixed, "Try an example" border animation rebuilt from a paint-triggering animated `@property`/conic-gradient to a GPU-cheap transform-based rotation, oversized/zoom sizing corrected, spacing and typographic hierarchy improved. Verified live by Ahmad via screen recording.

---

### TASK 8E — Frontend: Disable Mobile Pinch/Double-Tap Zoom Across the App ✅
**Completed:** July 8, 2026. Viewport meta updated to `maximum-scale=5, minimum-scale=1` (accessibility-conscious compromise — deliberate zoom still allowed, only accidental/runaway pinch-zoom and iOS double-tap-zoom curbed via a `touch-action: manipulation` CSS backstop). Verified working across homepage, results page, and admin dashboard on mobile, with desktop zoom confirmed unaffected.

---

### TASK 11A — Restructure Vercel App: Marketing Front Door + Scoring at /score ✅
**Completed:** July 8, 2026. Marketing homepage live at `/` (Hero, chain row, How It Works, Trust Signals, Use Cases, Get the API, Find Us/Integrations, footer), scoring tool live unchanged at `/score`. Brand consistency confirmed — locked OTI color system, shared components, logo matches `/score` exactly. Verified live by Manager via fresh cache-busted screenshot and JS bundle inspection, and by Ahmad directly.

<details><summary>Original spec (for reference)</summary>

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

**⚠️ Brand consistency is non-negotiable for this task.** This is not a fresh design — it must look and feel like it belongs to the exact same product as `/score` (the results/scoring pages) and the admin dashboard. A visitor moving from `/` to `/score` should never feel like they landed on a different site. Before writing any code:
1. Open the actual live `/score` page and the results page and note the real, current values in use — border radius, card padding, shadow/glow treatments, button states (hover/active/disabled), font weights, and spacing rhythm between sections — don't guess or approximate them.
2. Reuse existing shared components/CSS classes/tokens from the scoring app wherever possible (buttons, card containers, badges, chain icons) instead of rebuilding equivalents from scratch. This avoids visual drift and keeps future updates in sync.
3. Any new component introduced for the homepage (that doesn't exist in the scoring app) must still derive from the same tokens below — never introduce a one-off color, font, radius, or shadow value.

**Brand system — use the LOCKED OTI color system exactly (see Color System section above), zero deviation:**
- Background: `#05080f` · Surface: `#0b0f1a` · Surface-2: `#0f1520` · Borders: `#1c2535`
- Mint primary: `#00e5a0` · Mint gradient: `#3EFFC1`
- Body text: `#e8f4ff` · Dimmed text: `#7a8fa8`
- Chain brand colors: use the same per-chain hex values already defined in the scoring app (Ethereum `#627EEA`, Bitcoin `#F7931A`, Solana `#9945FF`, BNB `#F3BA2F`, all 15 total) — do not invent new ones for the homepage's chain row
- Typography: same font family already in use across the scoring app (Geist Sans or Inter) — match weights and sizing scale, not just the family name
- Same special effects already established: navbar frosted glass (`backdrop-filter: blur(14px)`), mint glow on primary buttons, `color-mix()` tint techniques where relevant — with plain-value fallbacks for older browsers
- Micro-interaction: the spiral logo rotates subtly on hover (CSS `transform: rotate()`, 2–3s ease, infinite)
- All CTAs, highlights, active states: mint only — no other accent color
- Do NOT revert to pure black `#000000` anywhere

**Logo:** Use `logo.svg` (the current, non-blurry logo already live on the scoring app — Task 2B is done) at the same aspect ratio and visual treatment as it appears in the scoring app's navbar. It must look identical in the homepage navbar/footer as it does on `/score` — same crispness, same proportions, same hover behavior if any exists there.

**Overall bar:** this must read as a fully polished, professional product website — not a placeholder or template. Treat spacing, alignment, hover states, and responsive behavior with the same level of care and polish that Task 8/8B/8D delivered on the scoring app. Screenshot both `/` and `/score` side by side before calling this done and confirm they feel like one coherent product.

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
- Brand system uses the locked OTI color system (Background `#05080f`, Mint `#00e5a0`/`#3EFFC1`) throughout, with zero one-off/invented colors, fonts, or component styles anywhere on the page
- Logo renders identically (crispness, proportions) to how it appears on `/score`
- Side-by-side screenshot of `/` and `/score` submitted to Manager showing visual consistency before marking done

</details>

---

### TASK 11B — Whitepaper Page
**Owner:** Frontend Builder
**Phase:** 4 — Pre-Distribution
**Priority:** HIGH — builds credibility with investors, enterprise partners, and serious developers
**Depends on:** Task 11A (lives inside the same Vercel project)
**Status: 🟡 LIVE BUT NOT DONE** — built and deployed at `/whitepaper`, but Manager's live verification (July 8, 2026, session 4) found body text rendering in the dimmed grey token instead of white; Ahmad also flagged a mobile horizontal-scroll bug and requested the Roadmap section be removed entirely. Fix list sent back to Frontend Builder:
1. All body/paragraph text → white (`#e8f4ff`), keep section numbers mint (`#00e5a0`)
2. Eliminate mobile horizontal scroll/shift — must fit one column at 375px
3. Remove the Roadmap section entirely; renumber all following sections/TOC sequentially

**Route:** `/whitepaper`

**Nav update required** (update the navbar from Task 11A):
- Desktop: `Logo | Score a Wallet | API Docs | Whitepaper | [Social icons]`
- Mobile hamburger: Score a Wallet → API Docs → Whitepaper → [Social icons]

---

**Design rules:**
- Use the locked OTI color system (Background `#05080f`, Surface `#0b0f1a`, Mint `#00e5a0`/`#3EFFC1` — see Color System section above) — must match the homepage (Task 11A) and scoring app exactly, same tokens, no deviation
- Typography: same font family as the rest of the site (Geist Sans or Inter). Body text slightly larger than normal (17–18px), high line-height (1.8) for readability — this is a long-form document
- Reuse the shared navbar/footer components from Task 11A rather than rebuilding them — this page must feel like a continuation of the same site, not a separate document viewer
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

**Status update (July 9, 2026):** Content is code-complete (all 6 sections built, API values live-verified). Ahmad authorized open-ended follow-on work, which also added: an OG image, a "Try It Live" widget, a privacy audit that removed leaked internal doc references, and a migration of API URLs to a not-yet-live custom domain (`api.otiscore.io`). This broke production: `/docs/` 404s on Vercel (SPA catch-all intercepts it — docs were never deployed as their own Vercel project) and "Try It Live" fails (DNS for `api.otiscore.io` isn't live). Fix plan: revert API URLs to the working Railway URL, deploy `oti-docs/` as its own Vercel project, then add a `vercel.json` rewrite (`/docs/:path*` → that deployment, ordered before the SPA catch-all) so the public URL stays `otiscore.vercel.app/docs/` with no custom domain needed. Full detail in FRONTEND_TASKS.md Task 11.

---

### TASK 11D — Replace emoji icons with a real icon set + copy tone pass
**Owner:** Frontend Builder
**Phase:** 4 — Pre-Distribution
**Priority:** Medium — after Task 11 deployment fixes
**Added:** July 9, 2026

Homepage Trust Signals and Use Cases sections use raw emoji as icons (from the original Task 11A spec) — reads as unpolished for an enterprise sales motion. Replace with a proper icon set (Lucide/Heroicons) in mint. Also do a read-through of the whitepaper and docs copy for AI-sounding phrasing and flag findings to the Manager before rewriting. See FRONTEND_TASKS.md Task 11D for full spec.

---

---
## ⛔ PHASE 4 GATE — All items below depend on this being complete first

Before any distribution channel (Tasks 12–15) is assigned, confirm ALL of the following are done:
- [x] Task 3 — Admin auth live on Railway ✅
- [x] Task 5 — Weighted signal response in API ✅
- [ ] Task 9 — Admin panel UI live on Vercel
- [x] Task 11A — Marketing homepage live at `/`, scoring tool at `/score` ✅
- [ ] Task 11 — Developer docs site live on Vercel (code-complete, deployment/remediation pending — see status update above)
- [ ] Task 11D — Emoji icons replaced, copy tone pass done
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
**Status: 🟡 SENT** — sent to Backend Builder July 8, 2026 (session 4), in progress, not yet reviewed.

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
