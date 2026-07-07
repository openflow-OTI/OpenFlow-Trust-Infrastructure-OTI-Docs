# OTI — Backend Builder Task List
> Last updated: July 7, 2026 (session 2 — Task 9C ✅ done and verified, Task 11C is next active task) | Maintained by: Development Manager
> **This file contains your tasks only. Read BUILDER_ONBOARDING.md and ARCHITECTURE.md before starting anything here.**
> Build in the exact order listed. Do not skip ahead.

---

## ✅ Your Completed Tasks

### TASK 3 — Admin Route Authentication ✅
- `adminAuth.ts` middleware created, applied to all `/api/admin/*` routes
- Returns 401 on missing/wrong `x-admin-secret` header
- Swagger updated with `AdminSecretAuth` security scheme
- Verified live on Railway

### TASK 4 — History Endpoint → Database ✅
- `GET /api/score/:address/history` now queries `chain_scores` DB table
- Optional `?chain=` filter; auto-detects chain family when omitted
- Ordered by `scored_at` DESC, limited to 50 records
- OpenAPI spec updated; verified live on Railway

### TASK 5 — Signal Scores → Weighted API Response ✅
- `signalWeighting.ts` transformer added (scoring.ts untouched)
- Score response now returns `{ score, weighted, maxWeight }` per signal
- Both fresh-compute and cache-hit paths updated
- History endpoint correctly still returns raw signals
- Both OpenAPI specs updated (served + frontend codegen)
- Verified live on Railway

### TASK 6 — subscriptions updatedAt Migration ✅
- `updated_at` column added to `subscriptions` table via Drizzle migration
- `PATCH /api/admin/keys/:id` now sets `updated_at: new Date()` on every update
- Railway production DB migration run manually via Railway Console on July 6, 2026
- Verified: `GET /api/admin/keys` no longer returns 500

### TASK 7D — Bitcoin Wallet Age Fix ✅
- Root cause: `getBitcoinTxs()` was fetching only the most recent ~50 txs (single page) — not full history
- Fix: now paginates backwards through mempool.space `/address/:address/txs/chain/:last_seen_txid` until true wallet history start
- Safety cap of 40 pages added to bound latency for hyperactive wallets
- Verified on Railway: old wallets returning walletAgedays 4,896 and 5,002 (previously ~5)

### Fix: GET /admin/stats 500 Error ✅ (July 7, 2026)
- Root cause: stats handler had no error handling — any DB query failure caused unhandled rejection → HTML 500
- Fix: per-query isolation with individual try/catch blocks, each defaulting to 0 on failure
- Result: `/admin/stats` always returns HTTP 200; Admin Panel login now works
- Verified live by Manager: HTTP 200 with real data (today_requests, active_keys, total_compromised)

### Fix: PATCH /admin/plan-configs/:id 404 Error ✅ (July 7, 2026)
- Root cause: handler was doing `WHERE plan_name = UUID` — matched nothing
- Fix: dual lookup — UUID param → WHERE id, name string param → WHERE plan_name
- Both UUID and plan name strings now accepted
- Verified live by Manager: both lookup methods return HTTP 200 and update DB correctly

### Fix: GET /admin/keys + POST /admin/keys — Full Resolution ✅ (July 7, 2026)
- Root cause: Railway subscriptions table predates the Drizzle schema in the repo
- Real columns confirmed via information_schema: id, api_key, plan, owner_address, created_at, expires_at, updated_at (NO status column, NO email column, NO daily_limit column)
- GET fixed: raw SQL SELECT * with column-name fallback mapper; no WHERE filter on status
- POST fixed: INSERT (api_key, plan, owner_address) only — no status, no missing columns
- Stats active_keys fixed: removed WHERE status = 'active' (column does not exist); counts all rows
- Verified live: Ahmad created enterprise, pro, and free keys at 14:15 July 7, 2026 — Edit and Delete buttons confirmed working

### TASK 9C — Verify & Harden Plan Limit Enforcement ✅
**Completed:** July 7, 2026

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

## 🔴 Your Task Queue — Build In This Exact Order

### TASK 11C — Signal Accuracy Audit & Cross-Chain Fix
**Phase:** Pre-Distribution (must be done before Phase 5 launches)
**Priority:** CRITICAL — signals are the core product; wrong data destroys trust
**Depends on:** Nothing — start immediately

**Context:**
The 5 scoring signals were designed with EVM chains in mind. Bitcoin, Solana, TON, Tron, and Sui have fundamentally different data models — no ERC-20 tokens, no internal transactions, no smart contracts in the EVM sense. Currently these non-EVM chains are being scored using EVM-specific logic, producing wrong and misleading results. Examples: the Satoshi genesis wallet scores ~51 days wallet age (correct is 5,700+ days); Bitcoin wallets get 0/20 for token holding because ERC-20 tokens don't exist on Bitcoin; "1 smart-contract tx" appears on Bitcoin wallets that have never touched a smart contract. These are not edge cases — they affect every single non-EVM wallet scored by OTI.

**What to audit and fix — per signal:**

**Signal 1 — Wallet Age (25%)**
- EVM: first transaction timestamp from Etherscan — correct
- Bitcoin: Task 7D attempted a fix but the Satoshi genesis wallet may still show ~51 days (possible stale cache). Flush cache and re-verify. If still wrong, re-investigate `bitcoin.ts`.
- Solana: verify first tx timestamp is pulled correctly
- TON, Tron, Sui: verify each individually

**Signal 2 — Transaction Count (20%)**
- EVM: Etherscan tx count, capped at 1,000 — correct
- Non-EVM: verify each chain returns real tx count, not a placeholder or default

**Signal 3 — Token Holding Behavior (20%)**
- EVM: ERC-20 token diversity — correct
- Bitcoin: has NO tokens — currently returns 0/20 which unfairly penalises BTC wallets. Fix: if chain is Bitcoin, skip this signal and redistribute its weight proportionally across the other 4 signals, OR score it as neutral (50/100).
- Solana: has SPL tokens — ensure SPL token diversity is being scored, not defaulting to 0
- TON, Tron, Sui: verify token data source and scoring for each

**Signal 4 — Smart Contract Interactions (20%)**
- EVM: internal tx count from Etherscan — correct
- Bitcoin: has NO smart contracts — currently returns fabricated data. Fix: same approach as Token Holding — redistribute weight or score neutral.
- Solana: program interactions = equivalent of contract interactions — verify this is scored correctly
- TON, Tron, Sui: verify each

**Signal 5 — Transaction Timing Patterns (15%)**
- EVM: uses internal transaction timestamps — correct
- Non-EVM: subtitle currently shows "0 internal transactions" for non-EVM chains. Use actual transaction timestamps for timing analysis. Do not default to 0 or use internal tx counts as a proxy.

**⚠️ SACRED FILES — do not touch:**
- `src/lib/scoring.ts` — ever
- `nixpacks.toml` — ever

**Definition of done:**
- The Satoshi genesis wallet (`1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa`) scores wallet age > 5,000 days
- No signal shows fabricated data for chains where that data type doesn't exist
- Non-applicable signals are either redistributed or scored neutral — never 0 by default
- All 15 supported chains tested with at least one known wallet
- Results documented in a short test log committed to the repo
- Report back to Manager with results before marking done

---

## ⏳ Future Tasks (Not Yet Active — Manager Will Assign When Ready)

These are coming after Task 11C is complete. You do not need to read these in detail now.

- **Phase 5:** Telegram bot (Task 12), Discord Bot (Task 13), Embeddable Widget (Task 14), Firefox Extension (Task 15)
- **Phase 6:** Wallet Ownership Registry (WOR) — Ahmad's flagship feature. Off-chain EIP-191 signing + passkey. New DB tables, new routes.
- **Phase 7:** Monetization — self-serve developer portal, paid plan tiers, BSC/Base/Optimism unlock

---

## Keeping This File Updated

You never update either file on your own initiative. Every update — whether marking a task done or adding a new one — happens only when the Manager explicitly tells you to.

**When you complete a task:**
1. Notify the Manager — do not mark anything done yourself
2. Wait for the Manager to review and confirm your work
3. Only when the Manager explicitly tells you to mark it done:
   - Move it to the ✅ Completed Tasks section in this file
   - Mark it ✅ in `TASKS.md` as well
4. Both files must always match

**When the Manager assigns you a new task:**
1. The Manager will explicitly tell you to add the new task to your file
2. Only then: add it to the bottom of your Queue section in this file AND add it to `TASKS.md`
3. Read it fully before starting — every task has a "Why" and a "Definition of done"
4. Do not start a task that has a ⚠️ dependency warning until the Manager confirms it is cleared

**General rules:**
- Never touch Git, never push, never open a PR — Ahmad handles all of that himself
- Never touch `scoring.ts` — ever
- Never touch `nixpacks.toml` — ever
- If a task is blocked, tell the Manager immediately — do not sit on a blocker
- Never update any file without the Manager's explicit instruction
