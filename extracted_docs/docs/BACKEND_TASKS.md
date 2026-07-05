# OTI — Backend Builder Task List
> Last updated: July 5, 2026 (updated by Manager) | Maintained by: Development Manager
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

---

## 🔴 Your Task Queue — Build In This Exact Order

---

### TASK 6 — subscriptions updatedAt Migration
**Phase:** 1 — Bug Fixes
**Priority:** MEDIUM
**Depends on:** Task 3 must be merged first

**Why you are doing this:**
The Admin Panel (being built by the Frontend Builder in Task 9) will display a table of all API keys including a "last modified" column. That column needs a timestamp from the database. The `subscriptions` table currently has no `updated_at` column, so every edit to an API key has no timestamp recorded. This migration adds that column and wires it up so the Admin Panel has the data it needs when it ships.

⚠️ **Coordination point — Frontend Builder:**
The Frontend Builder's Task 9 (Admin Panel UI) depends on this migration being merged and deployed first. Once your PR is merged and Railway has run the migration, notify the Manager so the Frontend Builder can proceed.

**What to build:**
Add an `updated_at` column to the `subscriptions` table via a Drizzle migration. The column should be `timestamp`, nullable, defaulting to null (so existing rows aren't affected). Update the `PATCH /api/admin/keys/:id` handler in `src/routes/admin.ts` to set `updated_at: new Date()` on every update. Update the Drizzle schema file for the `subscriptions` table to include the new column.

**Definition of done:** Migration runs cleanly. PATCH endpoint updates `updated_at`. Column visible in database.

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
