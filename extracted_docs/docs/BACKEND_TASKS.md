# OTI — Backend Builder Task List
> Last updated: July 7, 2026 (updated by Manager — all fixes confirmed done, queue empty) | Maintained by: Development Manager
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

---

## 🔴 Your Task Queue — Build In This Exact Order

### TASK 9C — Verify & Harden Plan Limit Enforcement (All Plans)
**Priority:** HIGH
**Depends on:** Nothing — Plan Configs admin UI is live

**Why:** The Admin Panel now lets Ahmad set daily_limit for any plan.
We confirmed anonymous enforcement works. Free, pro, and enterprise
have never been tested. If apiKeyAuth.ts has a bug, Ahmad could set
a limit that silently does nothing.

**What to do:**
1. Read `src/middlewares/apiKeyAuth.ts` — confirm it reads daily_limit
   from plan_configs dynamically per request (not hardcoded, not cached).
2. Test each plan:
   - Set free plan daily_limit to 2 via Admin Panel → make 3 requests
     with a free API key → 3rd must be rate-limited (429).
   - Restore the limit after testing.
   - Confirm enterprise (daily_limit = null) = unlimited (no 429 ever).
3. Fix anything broken. Do not touch scoring.ts.

**Definition of done:**
- apiKeyAuth.ts reads daily_limit dynamically confirmed.
- Free/pro limits enforced immediately after Admin Panel change.
- Enterprise null = unlimited confirmed.
- Report back with results per plan.

---

## ⏳ Future Tasks (Not Yet Active — Manager Will Assign When Ready)

These are coming after the queue above is complete. You do not need to read these in detail now — they are listed so you know what is ahead.

- **Phase 4 (Pre-Distribution) — Task 11C:** Signal Accuracy Audit & Cross-Chain Fix. CRITICAL. Non-EVM chains (Bitcoin, Solana, TON, Tron, Sui) are being scored with EVM-specific logic. Must be fixed before any distribution channel launches. Full spec will be provided by the Manager when assigned.
- **Phase 5:** Telegram bot (Task 12), Chrome Extension (Task 13), Embeddable Widget (Task 14), Firefox Extension (Task 15)
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
