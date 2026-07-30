# OTI — Backend Builder Task List
> Last updated: July 30, 2026 (session 22 — Phase 0 Ecosystem Whitelist Infrastructure task added: Task 28. OTI Economics confirmed: 35M supply, dual bonding curve DCS + ERP parameters confirmed.) | Maintained by: Development Manager
> **This file contains your tasks only. Read BUILDER_ONBOARDING.md, ARCHITECTURE.md, DECISIONS.md, and TOKENOMICS.md before starting anything here.**
> **`DECISIONS.md` is especially important before touching any scoring, data-fetching, or chain-handling code — it explains why certain behaviors exist and which ones must not be changed without Ahmad's approval. You never update DECISIONS.md yourself.**
> Build in the exact order listed. Do not skip ahead.
>
> **⚠️ This is YOUR copy of this file, in YOUR own account/repo.** The Manager's copy is separate and does not sync with yours. Only mark a task done here, or add a new task here, when the Manager explicitly tells you to.

---

## Your Active Item Right Now

**As of July 30, 2026 — you have NO active task.** Phase 0 Task 28 (Whitelist System) is written and queued below. **Wait for the Manager to send you Task 28 explicitly before starting anything.** Do not begin on your own initiative.

---

## ✅ Your Completed Tasks (genuine new-build work)

### TASK 3 — Admin Route Authentication ✅
Built `adminAuth.ts` middleware, applied to all `/api/admin/*` routes, requiring a correct `x-admin-secret` header. Swagger updated with the `AdminSecretAuth` security scheme. Verified live on Railway.

### TASK 4 — History Endpoint → Database ✅
Built `GET /api/score/:address/history`, querying the `chain_scores` table with an optional `?chain=` filter (auto-detects chain family when omitted), ordered by `scored_at` DESC, capped at 50 records. OpenAPI spec updated; verified live.

### TASK 5 — Signal Scores → Weighted API Response ✅
Built the `signalWeighting.ts` transformer (sits alongside `scoring.ts`, never touches it) so the score response returns `{ score, weighted, maxWeight }` per signal on both the fresh-compute and cache-hit paths. History endpoint correctly still returns raw signals. Both OpenAPI specs updated. Verified live.

### TASK 6 — subscriptions.updated_at Migration ✅
Added the `updated_at` column to the `subscriptions` table via Drizzle migration; `PATCH /api/admin/keys/:id` now sets it on every update. Railway production migration run manually via the Railway Console (Railway does not auto-run migrations — see BF12 in `FIXES.md`).

### TASK 9C — Plan Limit Enforcement System ✅
Built out the real enforcement path across `apiKeyAuth.ts`, `score.ts`, and `admin.ts` (DELETE + PATCH) — the mechanism that actually gates requests against a key's daily quota and returns 429 once exhausted. Verified live: free plan (limit=2) 200/200/429 on requests 1–3; enterprise (limit=null) never 429s; PATCH edits persist with an updated `updated_at`.

---

## 🔴 Your Task Queue — Phase 0 Ecosystem Whitelist Infrastructure

---

### TASK 28 — Whitelist System
**Priority: FIRST | Start when Manager assigns — no dependency on frontend tasks**
**⚠️ All vesting/lockup/reward parameters must be admin-configurable. Nothing token-related is hardcoded.**

Build the complete backend infrastructure for the Ecosystem Whitelist program.

**Part A — Database Schema**

Create these tables and seed `whitelist_config` with default values. Run via `drizzle-kit push` — Ahmad executes this against Railway after you deploy.

```sql
-- Invite code management
CREATE TABLE whitelist_invites (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    invite_code VARCHAR(50) UNIQUE NOT NULL,
    is_used BOOLEAN DEFAULT false,
    used_by_wallet VARCHAR(42) DEFAULT NULL,
    referred_by_code VARCHAR(50) DEFAULT NULL,
    amount_contributed_usd NUMERIC(10, 2) DEFAULT 0.00,
    status VARCHAR(20) DEFAULT 'active',  -- 'active', 'banned', 'expired'
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Whitelisted operator profiles
CREATE TABLE whitelist_participants (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    wallet_address VARCHAR(42) UNIQUE NOT NULL,
    invite_code_used VARCHAR(50) NOT NULL,
    oti_allocated NUMERIC(20, 6) DEFAULT 0,
    oti_claimed NUMERIC(20, 6) DEFAULT 0,
    joined_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    status VARCHAR(20) DEFAULT 'active'  -- 'active', 'banned'
);

-- Social task submissions (pending admin approval)
CREATE TABLE whitelist_social_tasks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    wallet_address VARCHAR(42) NOT NULL,
    task_type VARCHAR(30) NOT NULL,  -- 'referral', 'post_tag', 'share', 'follow'
    proof_url TEXT DEFAULT NULL,
    oti_reward NUMERIC(20, 6) NOT NULL,
    status VARCHAR(20) DEFAULT 'pending',  -- 'pending', 'approved', 'rejected'
    submitted_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    reviewed_at TIMESTAMP WITH TIME ZONE DEFAULT NULL
);

-- Protocol global state (single-row — serves frontend live counters)
CREATE TABLE protocol_state (
    id INT PRIMARY KEY DEFAULT 1,
    dcs_total_usd_committed NUMERIC(10, 2) DEFAULT 0.00,
    dcs_oti_remaining NUMERIC(20, 6) DEFAULT 7000000.00,
    total_slots_claimed INT DEFAULT 0,
    CONSTRAINT single_row CHECK (id = 1)
);

-- Admin-configurable parameters (key-value — nothing hardcoded)
CREATE TABLE whitelist_config (
    key VARCHAR(100) PRIMARY KEY,
    value TEXT NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

Seed `whitelist_config` with these defaults:
```sql
INSERT INTO whitelist_config VALUES
  ('dcs_total_oti', '7000000', NOW()),
  ('dcs_target_usd', '25000', NOW()),
  ('dcs_start_rate', '0.001190', NOW()),
  ('dcs_end_rate', '0.005952', NOW()),
  ('erp_total_oti', '1750000', NOW()),
  ('erp_base_referral_oti', '3000', NOW()),
  ('erp_base_post_tag_oti', '1000', NOW()),
  ('erp_base_share_oti', '500', NOW()),
  ('erp_base_follow_oti', '500', NOW()),
  ('vesting_lockup_pct', '75', NOW()),
  ('max_whitelist_slots', '10000', NOW());
```

Also insert the protocol_state seed row:
```sql
INSERT INTO protocol_state (id) VALUES (1);
```

**Part B — API Endpoints (new file: `src/routes/whitelist.ts`)**

`POST /api/verify-invite`
- Body: `{ invite_code, wallet_address, terms_accepted: true }`
- `terms_accepted` must be `true` — reject 400 if not
- Look up code in `whitelist_invites` — 404 if not found, 400 if used or status ≠ 'active'
- Check `whitelist_participants` for `wallet_address` — 409 if already registered
- Check `total_slots_claimed` vs `max_whitelist_slots` — 410 if full
- On success:
  - Mark `whitelist_invites.is_used = true`, set `used_by_wallet`
  - Insert into `whitelist_participants`
  - Increment `protocol_state.total_slots_claimed`
  - If `referred_by_code` is set on the invite: insert auto-approved referral row in `whitelist_social_tasks` for the referring wallet, update their `oti_allocated`
  - Return `{ success: true, wallet_address, oti_allocated, current_dcs_rate, current_erp_referral_bonus }`

`GET /api/whitelist/state`
- Public endpoint — no auth required
- Returns live values from `protocol_state` + computed DCS rate + computed ERP bonuses
- DCS current rate: `dcs_start_rate + (dcs_end_rate - dcs_start_rate) × ((dcs_total_oti - dcs_oti_remaining) / dcs_total_oti)`
- ERP current referral bonus: `erp_base_referral_oti × (dcs_oti_remaining / dcs_total_oti)`
- Frontend polls this every 30 seconds

`GET /api/whitelist/participant/:wallet_address`
- Returns participant record if found, 404 if not

**Part C — Admin Dashboard Additions**

Add a "Whitelist" tab to the existing admin panel. Four sub-views:

1. **Batch Code Generator**
   - Input: number of codes (default 10, max 500)
   - `POST /api/admin/whitelist/generate-codes` — generates N unique `OTI-XXXX-XXXX` codes (uppercase alphanumeric, 4+4), inserts into `whitelist_invites` with status 'active'
   - Displays generated codes in a copyable list

2. **Code Management Panel**
   - Table: all codes, status, `used_by_wallet` (truncated), `amount_contributed_usd`, `created_at`
   - Filter by status
   - Per-row "Ban" button: `PATCH /api/admin/whitelist/codes/:id` → sets status = 'banned'

3. **Social Task Review Queue**
   - Table: all pending `whitelist_social_tasks` — wallet, task_type, proof_url (clickable link), oti_reward, submitted_at
   - Per-row "Approve" / "Reject" buttons: `PATCH /api/admin/whitelist/social-tasks/:id`
   - On approve: status = 'approved', add `oti_reward` to that wallet's `whitelist_participants.oti_allocated`
   - On reject: status = 'rejected', no allocation change

4. **Protocol Config Override**
   - Input fields for each `whitelist_config` key (prefilled with current DB values)
   - "Save" button: `PUT /api/admin/whitelist/config` — updates config keys
   - This is how Ahmad adjusts all parameters without a code deploy

**Part D — Smart Contracts (BNB Testnet → Mainnet — you handle everything end-to-end)**

Ahmad is not involved at any point during this part. You generate all keys, deploy to testnet, test fully, then deploy to mainnet — all without Ahmad. At close, deliver the complete handover package to the Manager. The Manager relays it to Ahmad. Ahmad's only action is to receive the package and delete this workspace. See D42 in DECISIONS.md.

**Step 1 — Generate deployer wallet:**
```js
const { ethers } = require("ethers");
const wallet = ethers.Wallet.createRandom();
console.log("Address:", wallet.address);
console.log("Private key:", wallet.privateKey);
```
Save as Replit env vars: `DEPLOYER_ADDRESS`, `DEPLOYER_PRIVATE_KEY`. Never put keys in any file.
This wallet is the permanent mainnet deployer/owner — Ahmad takes ownership at handover.

**Step 2 — Fund the deployer wallet:**
- Testnet: https://testnet.bnbchain.org/faucet-smart → paste DEPLOYER_ADDRESS → request testnet BNB (free)
- Mainnet: fund from your own wallet with enough BNB to cover contract deployment gas (~0.01–0.05 BNB). Ahmad reimburses at handover.
- Confirm balance on both networks before proceeding.

**Step 3 — Deploy mock OTI BEP-20 token (`MockOTI.sol`) — testnet only:**
- Standard ERC-20, `constructor(uint256 initialSupply)` mints to deployer
- Use `35000000 * 10**18` as initial supply
- Deploy to BNB testnet (chainId 97) only — this mock is never deployed to mainnet
- Save deployed address as `MOCK_OTI_ADDRESS`

**Step 4 — Deploy `OTIWhitelistVesting.sol` to testnet first, then mainnet:**
- Testnet deploy (chainId 97): owner = DEPLOYER_ADDRESS, token = MOCK_OTI_ADDRESS
- Mainnet deploy (chainId 56): owner = DEPLOYER_ADDRESS, token = real OTI BEP-20 address (if not yet available, use a placeholder and document clearly — must be updated before mainnet goes live)
- Functions:
  - `vest(address participant, uint256 total_oti_amount)` — owner-only. 25% immediate, 75% linear daily over `vesting_duration_days`
  - `claimVested(address participant)` — callable by participant. Transfers unlocked OTI.
  - `setVestingDuration(uint256 days_)` — owner-only. Future vests only, not retroactive.
  - `getVestingStatus(address participant)` — view. Returns total_allocated, total_claimed, currently_claimable, vesting_start, vesting_end
  - `banParticipant(address participant)` — owner-only. Freezes remaining locked tokens.
- After testnet deploy: call `MockOTI.approve(vestingContractAddress, large_amount)` so vesting contract can move tokens
- Verify both contracts on BscScan (testnet + mainnet) after each deploy

**Step 5 — End-to-end test on testnet:**
- `vest(testWallet, 1000 * 10^18)` → confirm 250 OTI immediately claimable (25%)
- Advance testnet time or wait → confirm linear daily unlock works
- `claimVested(testWallet)` → confirm OTI transferred
- `banParticipant(testWallet)` → confirm remaining tokens frozen
- Paste raw output as evidence before proceeding to mainnet

**Step 6 — Handover package (deliver to the Manager in one message at close):**
The Manager relays this to Ahmad. Do not send directly to Ahmad.
- `DEPLOYER_ADDRESS`
- `DEPLOYER_PRIVATE_KEY` — Ahmad saves this immediately and securely
- `MOCK_OTI_ADDRESS` (BNB testnet only)
- `VESTING_CONTRACT_ADDRESS_TESTNET`
- `VESTING_CONTRACT_ADDRESS_MAINNET`
- BscScan testnet + mainnet links for all contracts
- Gas cost incurred (BNB amount) — for Ahmad to reimburse you

Ahmad saves the keys and addresses, then deletes this workspace. All conversation history and temp credentials are gone.

**Evidence required to close:**
- All DB tables created on Railway (Ahmad runs drizzle-kit push after you deploy)
- `/api/verify-invite`: valid code → success JSON; used code → 400; missing → 404 (paste raw responses)
- `/api/whitelist/state`: returns correct computed DCS rate and ERP bonus
- Admin Whitelist tab: code generator works, code table shows rows, social task queue renders, config save works
- BNB testnet contracts deployed and verified — paste BscScan testnet links
- BNB mainnet vesting contract deployed and verified — paste BscScan mainnet link
- vest() and claimVested() test on testnet confirmed with raw output
- Full handover package delivered to Manager

---

## ⏳ Future Tasks (Beyond Phase 0)

- **Phase 2B / XMTP Campaign (Tasks 19–22):** Etherscan key rotation → signing endpoint → smart contract + XMTP sender script → conversion dashboard. Prompts fully written in TASKS.md. Runs when Ahmad funds Etherscan accounts.
- **Phase 5 — Distribution channels:** Telegram Bot (Task 12), Discord Bot (Task 13), Embeddable Widget (Task 14).
- **Phase 3 — Monetization infrastructure:** self-serve developer portal backend, Pro/Enterprise plan rows in `plan_configs`, Stripe integration, Coinbase Commerce integration.

---

## Keeping This File Updated

You never update either file on your own initiative. Every update — whether marking a task done or adding a new one — happens only when the Manager explicitly tells you to.

**When you complete a task:**
1. Notify the Manager — do not mark anything done yourself
2. Wait for the Manager to review and confirm your work
3. Only when the Manager explicitly tells you to mark it done:
   - Move it to the ✅ Completed Tasks section in this file
   - Mark it ✅ in `TASKS.md` as well (or in `FIXES.md`, if it's a fix, not a task)
4. All your files must always match the Manager's

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
