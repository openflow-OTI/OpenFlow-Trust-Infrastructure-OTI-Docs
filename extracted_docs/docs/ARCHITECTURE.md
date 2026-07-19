# OTI — Architecture Ground Truth
> Last updated: July 17, 2026 | Maintained by: Development Manager
> **This file is the source of truth for every piece of infrastructure in OTI. All Builders and all new Manager accounts must read this before touching anything.**
> **After reading this file, read `DECISIONS.md` — it explains why things are built the way they are. This file says what; DECISIONS.md says why.**

---

## ⚠️ Important — Where the Code Lives

**This document describes the production OTI system deployed on Railway (backend) and Vercel (frontend).** The Manager's Replit workspace is used only for documentation, prompt writing, and roadmap management — it does NOT contain the OTI source code.

The OTI source code lives in Ahmad's GitHub repositories:
- **Backend repo:** https://github.com/openflow-OTI/OpenFlow-Trust-Infrastructure-OTI- — **PRIVATE** (contains scoring algorithm IP)
- **Frontend repo:** https://github.com/openflow-OTI/OpenFlow-Trust-Infrastructure-O-T-I-Frontend- — **PUBLIC** (clean React code, no secrets, builds developer trust)
- **Docs repo:** https://github.com/openflow-OTI/OpenFlow-Trust-Infrastructure-OTI-Docs — **Manager's workspace repo**

Builders work in their own Replit accounts against those GitHub repos. The Manager reads them for research but does not push to them.

**GitHub open/closed strategy:** GitHub does not support per-file access control. A repo is either fully public or fully private. The frontend is safe to make public (no secrets, no scoring logic). The backend must stay private — `scoring.ts` is protected IP and the admin routes must not be visible publicly.

---

## How Each Builder Gets These Docs

Builders don't receive docs via email or file sharing. The **OTI docs** zip file lives inside the GitHub repos. When a Builder clones the repo to start work, they extract the OTI docs zip from the project root to get all doc files.

**What Ahmad tells each Builder on day one:**
> "Clone the repo. Find the OTI docs zip file in the project root and extract it. Read `BUILDER_ONBOARDING.md` first, then `ARCHITECTURE.md`, then open your dedicated task file — `BACKEND_TASKS.md` if you are the Backend Builder, `FRONTEND_TASKS.md` if you are the Frontend Builder."

**Manager session update flow:**
- End of session: update `MANAGER_HANDOVER.md` + `TASKS.md` → Ahmad pushes via Replit's Git interface
- Start of new session: clone repo or `git pull` → read `MANAGER_HANDOVER.md` → continue

---

## What OTI Is

**OTI (OpenFlow Trust Infrastructure)** — a blockchain wallet trust scoring API and frontend.

Given a wallet address + chain, OTI returns a 0–100 trust score built from five weighted signals. The score is deterministic, derived purely from on-chain data, and works across 15 blockchains.

**Live URLs:**
| Resource | URL |
|---|---|
| Frontend / Marketing | `https://otiscore.vercel.app` ✅ Live |
| Scoring App | `https://otiscore.vercel.app/score` (after Task 11A restructure) |
| Backend | `https://workspaceapi-server-production-5c0c.up.railway.app` |
| Swagger | `/api/docs` |
| Health | `/api/healthz` |

**Domain status:** `otiscore.vercel.app` — confirmed live July 5, 2026. Ahmad attempted `oti.vercel.app` but it was already claimed by another project. `otiscore.vercel.app` is clean and professional. Medium-term: acquire a real domain like `openflowlabs.io` or `getoti.com` (~$15–30/yr on Porkbun.com) and point it at this Vercel project.

---

## The Five Trust Signals (scoring.ts — SACRED, NEVER TOUCH)

`src/lib/scoring.ts` is the core protected algorithm. It computes five signals, each scored 0–100, then applies weights to produce the final 0–100 overall score.

| Signal | Weight | What it measures |
|---|---|---|
| Wallet Age | **25%** | How long the wallet has existed on-chain |
| Transaction Count | **20%** | Volume and frequency of transactions |
| Token Holding Behavior | **20%** | Diversity and quality of tokens held |
| Smart Contract Interactions | **20%** | Depth of DeFi/contract engagement |
| Transaction Timing Patterns | **15%** | Consistency and naturalness of activity timing |

**Math:** `overall_score = (walletAge × 0.25) + (txCount × 0.20) + (tokenHolding × 0.20) + (contractInteractions × 0.20) + (timingPatterns × 0.15)`

**Example (49% wallet):**
- Wallet Age: raw=100, weighted=25 (100 × 0.25)
- Transaction Count: raw=20, weighted=4 (20 × 0.20)
- Token Holding: raw=22, weighted=4.4 (22 × 0.20)
- Smart Contract: raw=25, weighted=5 (25 × 0.20)
- Timing Patterns: raw=70, weighted=10.5 (70 × 0.15)
- **Total: 48.9 ≈ 49** ✓

---

## Supported Chains (15 total)

**EVM chains — via Etherscan V2 API (7 live, 3 gated):**
Ethereum, Polygon, Arbitrum, Avalanche, Sonic (formerly Fantom — chain ID 146, renamed July 2026), Linea, zkSync

**Non-EVM chains (5):**
TON, Solana, Sui, Bitcoin, Tron

**Intentionally returning 503 (need Etherscan Lite $49/mo):**
BSC (BNB Smart Chain), Base, Optimism

**Not implemented — removed from all documentation (do not list as supported):**
Scroll, Sepolia, Holesky — were in early docs as planned but were never built. Sepolia/Holesky are Ethereum testnets (limited value in a production trust scorer); Scroll requires the same Etherscan Lite plan as BSC/Base/Optimism. Real live chain count: 12 (7 EVM + 5 non-EVM). Do not claim 15.

---

## Database Tables — Purpose of Each

### `chain_scores`
**Purpose:** The historical ledger — the core of OTI's data moat.
Every score ever computed for any wallet on any chain is stored here. Full record: wallet address, chain, all 5 raw signal scores, overall score, timestamp. This is what makes OTI more valuable over time — the data accumulates.

**Status:** ✅ Working. `persistScore()` writes here on every lookup.

**History endpoint:** Now reads directly from this table. `lib/history.ts` (the in-memory store) was deleted; the dead `recordHistory()` call was removed from the scoring pipeline.

**Two-tier cache — L1 + L2:** Every score request checks the in-memory LRU cache (L1) first, then this table (L2) before making any external API call. A valid L2 hit (within the 30-day rescore window) warms L1 and returns immediately — zero external API calls. Only a true L2 miss triggers Etherscan/Toncenter/etc.

**Keep-highest write logic:** When a fresh score is written here, if a higher score already exists for that wallet, the higher score is preserved and only `scored_at` is updated to reset the rescore window. A wallet's score can only go up permanently — quiet periods do not lower a historically active wallet's score.

---

### `scores`
**Purpose:** Lightweight public lookup cache — given a wallet address, quickly return its last known overall score without querying the heavier `chain_scores` table with full signal data. Designed for simple integrations that only need a single trust number, not the full breakdown.

**Status:** ✅ Being written to. `persistScore()` writes to both `chain_scores` and `scores` simultaneously.

**Note:** This table is NOT redundant. It serves a different read pattern — fast, minimal, no signal detail.

---

### `wallet_links`
**Purpose:** The future portfolio / wallet cluster feature. Allows one user to link multiple wallet addresses as belonging to them. Once a user proves ownership of multiple wallets (via the WOR system, see ROADMAP), OTI can compute an aggregate trust profile across all of them.

**Status:** 🔵 Infrastructure ready. Zero routes use it yet. This is because it depends on the Wallet Ownership Registry (WOR) system which hasn't been built yet. `wallet_links` is the second step — WOR must exist first.

---

### `plan_configs`
**Purpose:** Rate limiting tier system. Different API plans have different daily quotas. `anonymous` = 3/day, `free` = higher limit. `pro` and `enterprise` tiers are planned.

**Status:** ✅ Working. The rate limiter reads `daily_limit` from this table dynamically.

**Planned next:** Self-serve developer portal where developers sign up, get a key, and choose their plan. Also: the frontend should display the current plan's quota dynamically (currently hardcoded "3 per day" text).

---

### `subscriptions` (API keys table)
**Purpose:** Developer access layer — every third party (exchange, DeFi protocol, custody platform) that wants to integrate OTI gets an API key here. Paired with `plan_configs`, these two tables form the complete monetization backend.

**Status:** ✅ Working. Missing `updated_at` column — needs migration when PATCH /admin/keys/:id is used.

---

### `compromised_wallets`
**Purpose:** The trust denylist. Any wallet address in this table gets flagged in every score response regardless of its calculated score. Powers the `"compromised": true` field and the red warning banner in the frontend.

**Status:** ✅ Working. Currently only admin can add wallets here.

**Planned next:** The Wallet Ownership Registry (WOR) system will allow verified wallet owners to self-submit compromise reports, which will automatically add to this table without admin involvement.

---

## Backend — Key Files and Their Purpose

### `src/lib/scoring.ts` ⚠️ SACRED — NEVER MODIFY
The trust scoring algorithm. Five signals, five weights, pure computation. This is OTI's protected IP.

### `src/lib/history.ts` — DELETED ✅
Was: in-memory Map storing score history per wallet+chain key (temporary scaffolding, wiped on every Railway restart). Now deleted entirely. History endpoint reads directly from `chain_scores`. The dead `recordHistory()` call in the scoring pipeline was also removed. Nothing imports this file.

### `src/routes/admin.ts` ✅ SECURED (BF5)
Full admin CRUD: API keys, plan configs, compromised wallets, usage stats. Protected by `adminAuth.ts` middleware — every `/api/admin/*` request must include the correct `x-admin-secret` header matching the `ADMIN_SECRET` Railway environment variable. Wrong or missing secret returns a generic 401. Verified live.

### `src/middlewares/apiKeyAuth.ts`
Daily quota logic. Reads from `plan_configs` table dynamically — not hardcoded.

### `src/middlewares/rateLimiter.ts`
IP-level rate limiter. 10 requests per minute per IP via express-rate-limit. Separate from API key quota.

### `lib/db/src/persist.ts`
`persistScore()` — writes to both `chain_scores` and `scores` tables. Called after every successful score computation.

### `src/lib/validateAddress.ts` (backend)
Server-side address format validation per chain.

### `src/lib/generateScoreCard.ts`
640×800px PNG canvas image — the shareable score card. Draws all 5 signals, footer says `openflowlabs.io`. This is the virality engine — every share is a free brand impression.

### `nixpacks.toml` ⚠️ SACRED — NEVER TOUCH
Railway build configuration. Breaking this breaks all deployments.

---

## Frontend — Key Files and Their Purpose

### `src/lib/scoring.ts` — NOT IN FRONTEND
Scoring logic is backend-only. Frontend receives computed scores via API.

### `src/api/client.ts`
OpenAPI-fetch client pointed at `VITE_API_BASE_URL`. Strongly typed via auto-generated schema.

### `src/api/schema.gen.ts`
Auto-generated TypeScript types from the live OpenAPI spec. Run `pnpm codegen` to regenerate. Never manually edit this file.

### `src/hooks/useScore.ts`
TanStack Query hook — fetches the score for a given wallet+chain. Handles loading, error, and cached states.

### `src/hooks/useHealth.ts`
Pings `GET /api/healthz` to check if the backend is reachable. **Currently unused in any component.** Designed to power a live status indicator in the navbar. Needs to be connected.

### `src/lib/validateAddress.ts` (frontend)
Client-side address format validation — gives instant format errors before hitting the API.

### `src/components/Logo.tsx`
`<img src="/logo.jpg" width={34} height={34}>`. Task 2 replaced the spiral SVG with this `<img>` tag (shipped ✅). However, a JPG rendered at 34px is blurry on high-DPI screens — a higher-resolution image or an SVG version is still needed. **The logo component change is done; the logo asset quality is still an open issue.**

### `src/pages/Home.tsx`
Single-page component that handles both the homepage (search form) and the results page (score display), switched by URL params.

### `vercel.json` ⚠️ SACRED — NEVER TOUCH
SPA routing: `{"rewrites":[{"source":"/(.*)", "destination":"/index.html"}]}`. Breaking this breaks all Vercel routing.

---

## Current API Response Shape (KNOWN BUG — signal scores need weighted values)

**Current (incorrect for developers):**
```json
{
  "address": "0x...",
  "chain": "ethereum",
  "cached": false,
  "compromised": false,
  "score": 49,
  "signals": {
    "walletAge": 100,
    "transactionCount": 20,
    "tokenHoldingBehavior": 22,
    "smartContractInteractions": 25,
    "transactionTimingPatterns": 70
  },
  "metadata": { ... }
}
```

**Planned (correct — with weighted contributions):**
```json
{
  "signals": {
    "walletAge":               { "score": 100, "weighted": 25.0,  "maxWeight": 25 },
    "transactionCount":        { "score": 20,  "weighted": 4.0,   "maxWeight": 20 },
    "tokenHoldingBehavior":    { "score": 22,  "weighted": 4.4,   "maxWeight": 20 },
    "smartContractInteractions":{ "score": 25, "weighted": 5.0,   "maxWeight": 20 },
    "transactionTimingPatterns":{ "score": 70, "weighted": 10.5,  "maxWeight": 15 }
  }
}
```

The frontend signal bars must be updated to use `weighted / maxWeight` for bar fill width, and display `weighted/maxWeight` as the score label (e.g., "25/25", "4/20"). This fix must happen in BOTH backend (API response shape) and frontend (bar rendering logic).

---

## Two-Tier Cache Architecture (L1 + L2)

OTI uses a two-tier cache for all score requests — no external API call is made unless both tiers miss:

- **L1 — In-memory LRU cache:** 500 entries, 5-minute TTL. Unchanged from original design. A hit returns instantly.
- **L2 — `chain_scores` database table:** On every L1 miss, the backend checks whether a valid score for this wallet+chain exists in `chain_scores` within the configured rescore window (default: 30 days). A valid L2 hit warms L1 and returns the result — zero external API calls.
- **External API call:** Only triggered when both L1 and L2 miss. On a fresh compute, the result is written to both L1 and L2.

**Cache toggle:** The entire cache can be disabled from the admin panel (debugging/research only — not for production use). When off, every request fetches live chain data.

**Rescore window:** Configurable via admin panel. Reads from `system_settings` via `getSettings()` with a 60-second in-memory TTL — admin panel changes propagate within one minute, no deploy needed.

---

## `system_settings` Table

Stores global OTI configuration that Ahmad can change from the admin panel without touching code or triggering a deploy:

| Setting | Default | Notes |
|---|---|---|
| Rescore window | 30 days | How long a `chain_scores` entry is valid before a fresh compute is triggered |
| Cache enabled | true | Toggle the two-tier cache — debugging tool, not for production use |
| Score Source | OTI Backend | Controls where the widget API reads badge data from (see Score Source Switcher below) |

Read via `getSettings()` with a 60-second in-memory TTL. Any change Ahmad makes in the admin panel takes effect across all running instances within one minute.

---

## Score Source Switcher

The widget API's data source is a **server-side configuration** in `system_settings` — not hardcoded in the widget embed code. Three modes:

| Mode | Behaviour | When to use |
|---|---|---|
| **OTI Backend** | Widget API reads from `chain_scores` DB directly. No BAS dependency. No attestation required. Available from day one. | **Default during early growth.** |
| **BAS Attestation** | Reads attestation records from BNB Chain via BAS SDK. Trustless, blockchain-verifiable. Only returns data for wallets with an active BAS attestation. | Once attestation adoption is meaningful. |
| **Auto (fallback)** | Tries BAS first. Falls back to OTI's DB score if no BAS attestation exists for the wallet. | **Intended long-term default.** |

When Ahmad changes the setting in the admin panel, every embedded widget worldwide updates within one minute — no partner deploys, no partner notifications, no embed code changes required. **The widget embed code is permanently agnostic to this setting.** Partners only set `data-wallet`, `data-chain`, and `data-key`. Data source routing is entirely server-side.

---

## `chainRegistry.ts` — Single Chain Source of Truth

All chain routing logic — supported chains list, chain-to-family mapping, Etherscan plan-upgrade gate — now lives in a single file: `chainRegistry.ts`. Before this file existed, the same information was scattered across `score.ts`, `history.ts`, and `chainFamily.ts`.

Exports:
- `SUPPORTED_CHAINS` — the authoritative list of all scoring-eligible chains
- `ALL_CHAIN_ZOD_ENUM` — used by route validators
- `CHAIN_FAMILY_MAP` — maps chain name to family (`evm`, `ton`, `solana`, etc.)
- `PLAN_UPGRADE_REQUIRED` — set of chains gated behind Etherscan Lite: BSC, Base, Optimism

**Adding a new chain = editing one file.**

---

## `chainWeighting.ts` — Chain-Specific Weight Redistribution

`scoring.ts` is never touched. Weight redistribution for chains where not all five signals apply lives in `chainWeighting.ts`.

**Bitcoin** is the primary case: Token Holding Behavior (20%) and Smart Contract Interactions (20%) are inapplicable (Bitcoin has no ERC-20 tokens, no smart contracts). Their combined 40% is redistributed proportionally across the three applicable signals:

| Signal | Standard weight | Bitcoin weight |
|---|---|---|
| Wallet Age | 25% | 41.7% |
| Transaction Count | 20% | 33.3% |
| Transaction Timing Patterns | 15% | 25.0% |

The raw signal values `scoring.ts` produces (Token Holding = 0, Contracts = 0 for Bitcoin) are still returned in the API response. Only the final score uses the redistributed weights.

**General policy (Ahmad, confirmed):** This principle applies to any chain where a signal genuinely does not exist — Bitcoin is the primary case, not the only one.

---

## CORS Policy (Intentional — Do Not Change)

`app.use(cors())` with no origin restrictions is **deliberate and correct**.

**Why:** OTI is a public trust scoring API. Exchanges, DeFi frontends, and wallet apps need to call OTI from their browser-based dashboards. CORS restrictions would break those integrations. API key quotas control WHO can call and HOW MUCH — CORS is not the right tool for access control. Open CORS + API key auth = the correct combination for a developer API.

**Future:** Once paid plans exist, restrict CORS to allowlisted origins per paid API key (complex — low priority).

---

## useHealth Hook — Purpose and Status

`src/hooks/useHealth.ts` pings `GET /api/healthz` from the React app. The endpoint returns `{"status":"ok"}` when the backend is running.

**Intended for:** A live status indicator dot in the navbar — green when the API is up, amber when degraded, red when down. This tells users immediately whether OTI has issues, without needing to run a lookup and get a confusing error.

**Current status:** The hook is complete and correct. It is imported by zero components. **Frontend Builder task:** Add a status dot component to the Navbar that uses this hook.
