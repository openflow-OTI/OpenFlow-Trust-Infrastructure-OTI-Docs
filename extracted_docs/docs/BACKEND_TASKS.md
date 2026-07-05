# OTI — Backend Builder Task List
> Last updated: July 6, 2026 (updated by Manager) | Maintained by: Development Manager
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

---

## 🔴 Your Task Queue — Build In This Exact Order

---

### TASK 7D — Bitcoin Wallet Age Fix
**Phase:** 1 — Bug Fixes
**Priority:** MEDIUM
**Depends on:** Task 3 must be merged first

**Why you are doing this:**
Bitcoin is one of the 5 non-EVM chains OTI supports. Wallet Age carries 25% of the total trust score — the single highest weight of any signal. A Bitcoin wallet created in 2020 should score near 100 on wallet age, giving it 25 points toward the overall score. Instead, a bug in the timestamp parsing returns `walletAgedays: 5`, making every Bitcoin wallet appear brand new regardless of its actual age. This means every Bitcoin score is wrong. The fix is isolated to the Bitcoin chain handler and does not touch `scoring.ts`.

**What to build:**
There is a timestamp parsing bug in the Bitcoin chain handler (`bitcoin.ts` or equivalent). When computing `walletAgedays`, the calculation returns `5` for wallets that are years old (e.g., a wallet created in 2022 returns only 5 days). Investigate the timestamp format returned by the Bitcoin API (likely milliseconds vs seconds confusion, or epoch vs block height conversion error). Fix the calculation so `walletAgedays` correctly reflects the number of days since the wallet's first transaction. Verify with a known old Bitcoin wallet address.

**Definition of done:** A Bitcoin wallet with first activity in 2020 returns a `walletAgedays` value greater than 1,000 days.

---

## ⏳ Future Tasks (Not Yet Active — Manager Will Assign When Ready)

These are coming after the queue above is complete. You do not need to read these in detail now — they are listed so you know what is ahead.

- **Phase 5:** Telegram bot, Discord bot, embeddable widget (`/bots/` directory in this repo)
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
