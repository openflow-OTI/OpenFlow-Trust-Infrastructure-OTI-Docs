# OTI — Backend Builder Task List
> Last updated: July 30, 2026 (session 24 — Task 28 Part D updated: two contracts now required — OTIDCSContribution.sol (multi-coin, D59) + OTIWhitelistVesting.sol. Urgency note added: Railway subscription due today.)\n> (session 22 — Phase 0 Ecosystem Whitelist Infrastructure task added: Task 28. OTI Economics confirmed: 35M supply, dual bonding curve DCS + ERP parameters confirmed.) | Maintained by: Development Manager
> **This file contains your tasks only. Read BUILDER_ONBOARDING.md, ARCHITECTURE.md, DECISIONS.md, and TOKENOMICS.md before starting anything here.**
> **`DECISIONS.md` is especially important before touching any scoring, data-fetching, or chain-handling code — it explains why certain behaviors exist and which ones must not be changed without Ahmad's approval. You never update DECISIONS.md yourself.**
> Build in the exact order listed. Do not skip ahead.
>
> **⚠️ This is YOUR copy of this file, in YOUR own account/repo.** The Manager's copy is separate and does not sync with yours. Only mark a task done here, or add a new task here, when the Manager explicitly tells you to.

---

## Your Active Item Right Now

**⚠️ URGENT — As of July 30, 2026:** Task 28 (Whitelist System) is your active task. Railway subscription is due today at midnight UTC — Parts A, B, and C must be built and tested today. Part D (smart contracts) follows once A–B–C are deployed and confirmed. Read DECISIONS.md D59 for the updated multi-coin DCS contribution contract requirement before starting Part D.

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
**⚠️ All vesting/lockup/reward parameters must be admin-configurable. Nothing token-related is hardcoded. (D43, D53)**

Build the complete backend infrastructure for the Ecosystem Whitelist program: database tables, API endpoints, admin dashboard additions, and BNB Chain smart contracts.

Read DECISIONS.md entries D42–D58 before starting. They explain every design choice in this task.

---

**Part A — Database Schema**

Create all tables below and seed `whitelist_config`. Run via `drizzle-kit push` — Ahmad executes this against Railway after you deploy, not you.

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
    telegram_verified BOOLEAN DEFAULT false,
    telegram_user_id BIGINT DEFAULT NULL,
    x_handle VARCHAR(100) DEFAULT NULL,
    joined_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    status VARCHAR(20) DEFAULT 'active'  -- 'active', 'banned'
);

-- Social and referral task log (auto-verified — no manual review queue, D49)
-- task_type: 'referral', 'post_tag', 'share', 'follow_x', 'follow_telegram'
-- status: 'auto_verified', 'rejected'
CREATE TABLE whitelist_social_tasks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    wallet_address VARCHAR(42) NOT NULL,
    task_type VARCHAR(30) NOT NULL,
    proof_url TEXT DEFAULT NULL,
    oti_reward NUMERIC(20, 6) NOT NULL,
    status VARCHAR(20) DEFAULT 'auto_verified',
    submitted_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- One-time product engagement tasks
-- task_type: 'wor_register', 'wor_report', 'dev_api'
-- UNIQUE (wallet_address, task_type) enforces one-time-per-wallet-per-task
CREATE TABLE whitelist_task_completions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    wallet_address VARCHAR(42) NOT NULL,
    task_type VARCHAR(30) NOT NULL,
    oti_reward NUMERIC(20, 6) NOT NULL,
    completed_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    UNIQUE (wallet_address, task_type)
);

-- Daily wallet scoring rewards (once per calendar day UTC per wallet)
CREATE TABLE whitelist_daily_scores (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    wallet_address VARCHAR(42) NOT NULL,
    score_date DATE NOT NULL,
    oti_reward NUMERIC(20, 6) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    UNIQUE (wallet_address, score_date)
);

-- Whitepaper question pool (100 non-technical questions, admin-managed)
CREATE TABLE whitelist_whitepaper_questions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    question_text TEXT NOT NULL,
    option_a TEXT NOT NULL,
    option_b TEXT NOT NULL,
    option_c TEXT NOT NULL,
    option_d TEXT NOT NULL,
    correct_option CHAR(1) NOT NULL,  -- 'a', 'b', 'c', or 'd'
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Whitepaper progress per wallet (max 10 reward rounds = 30 questions)
CREATE TABLE whitelist_whitepaper_progress (
    wallet_address VARCHAR(42) PRIMARY KEY,
    questions_answered INT DEFAULT 0,
    rewards_claimed INT DEFAULT 0,
    last_round_at TIMESTAMP WITH TIME ZONE DEFAULT NULL
);

-- Session fingerprinting for multi-account detection (D58)
CREATE TABLE whitelist_fingerprints (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    wallet_address VARCHAR(42) NOT NULL,
    ip_address VARCHAR(45) NOT NULL,
    fingerprint_hash VARCHAR(64) NOT NULL,  -- SHA-256 of IP + UA + Accept-Language + client fingerprint_data
    user_agent TEXT DEFAULT NULL,
    event_type VARCHAR(30) NOT NULL,        -- 'register', 'connect_telegram', 'connect_x', 'claim_reward'
    flagged BOOLEAN DEFAULT false,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Flagged account log (auto-populated by collision checks)
CREATE TABLE whitelist_flags (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    wallet_address VARCHAR(42) NOT NULL,
    flag_reason VARCHAR(100) NOT NULL,      -- 'ip_collision', 'fingerprint_collision'
    related_wallets TEXT[] DEFAULT '{}',    -- other wallets sharing same IP or fingerprint
    status VARCHAR(20) DEFAULT 'open',      -- 'open', 'reviewed', 'cleared', 'banned'
    flagged_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    reviewed_at TIMESTAMP WITH TIME ZONE DEFAULT NULL
);

-- Protocol global state (single-row)
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

Seed `whitelist_config`:
```sql
INSERT INTO whitelist_config VALUES
  ('dcs_total_oti',                '7000000',  NOW()),
  ('dcs_target_usd',               '25000',    NOW()),
  ('dcs_start_rate',               '0.001190', NOW()),
  ('dcs_end_rate',                 '0.005952', NOW()),
  ('erp_total_oti',                '1750000',  NOW()),
  ('erp_base_referral_oti',        '3000',     NOW()),
  ('erp_base_post_tag_oti',        '1000',     NOW()),
  ('erp_base_share_oti',           '500',      NOW()),
  ('erp_base_follow_x_oti',        '500',      NOW()),
  ('erp_base_follow_telegram_oti', '500',      NOW()),
  ('erp_base_daily_score_oti',     '100',      NOW()),
  ('erp_base_wor_register_oti',    '500',      NOW()),
  ('erp_base_wor_report_oti',      '300',      NOW()),
  ('erp_base_dev_api_oti',         '500',      NOW()),
  ('erp_base_whitepaper_round_oti','200',       NOW()),
  ('vesting_lockup_pct',           '75',       NOW()),
  ('max_whitelist_slots',          '0',        NOW()),
  ('flag_ip_threshold',            '3',        NOW()),
  ('flag_fingerprint_threshold',   '2',        NOW());
  -- max_whitelist_slots 0 = unlimited (D53)
  -- flag thresholds: auto-flag when same IP/fingerprint shared by >= N wallets (D58)
```

Seed `protocol_state`:
```sql
INSERT INTO protocol_state (id) VALUES (1);
```

---

**Part B — API Endpoints (new file: `src/routes/whitelist.ts`)**

---

**`POST /api/verify-invite`**
Body: `{ invite_code, wallet_address, terms_accepted: true, referred_by_code?, fingerprint_data? }`
- `fingerprint_data` is optional from frontend: `{ screen_resolution, timezone_offset, canvas_hash }`
- `terms_accepted` must be `true` — 400 if not
- Look up `invite_code` in `whitelist_invites` — 404 if not found, 400 if used or status ≠ 'active'
- Check `whitelist_participants` for `wallet_address` — 409 if already registered
- Slot cap check: read `max_whitelist_slots` from config. If value > 0 and `total_slots_claimed >= value` → 410. If value = 0 → no cap.
- **Fingerprint capture + collision check (D58):**
  - Build `fingerprint_hash` = SHA-256(`ip_address + user_agent + accept_language + JSON.stringify(fingerprint_data)`)
  - Insert into `whitelist_fingerprints` (wallet_address, ip_address, fingerprint_hash, user_agent, event_type='register')
  - Count distinct wallets in `whitelist_fingerprints` with same `ip_address` — if >= `flag_ip_threshold`: insert flag rows into `whitelist_flags` for all involved wallets, reason = 'ip_collision'
  - Count distinct wallets with same `fingerprint_hash` — if >= `flag_fingerprint_threshold`: insert flag rows, reason = 'fingerprint_collision'
  - Flagging never blocks registration. Never tell the user they were flagged.
- On success:
  - Mark `whitelist_invites.is_used = true`, `used_by_wallet = wallet_address`
  - If `referred_by_code` present in body: set `whitelist_invites.referred_by_code`
  - Insert into `whitelist_participants`
  - Increment `protocol_state.total_slots_claimed`
  - If invite has `referred_by_code` set: auto-credit referral reward to referring wallet — insert into `whitelist_social_tasks` (task_type='referral', status='auto_verified'), add reward to referring wallet's `oti_allocated`
  - Return `{ success: true, wallet_address, oti_allocated, current_dcs_committed_rate, current_erp_referral_bonus }`

---

**`GET /api/whitelist/state`**
- Public endpoint — no auth
- Returns `protocol_state` + computed values:
  - `current_dcs_committed_rate` = `dcs_start_rate + (dcs_end_rate - dcs_start_rate) × ((dcs_total_oti - dcs_oti_remaining) / dcs_total_oti)`
  - `current_erp_referral_bonus` = `erp_base_referral_oti × (dcs_oti_remaining / dcs_total_oti)`
  - All current `erp_base_*` values (so frontend can show live reward amounts on every card)
- Frontend polls every 30 seconds

---

**`GET /api/whitelist/participant/:wallet_address`**
Returns full participant record including `telegram_verified`, `x_handle`, `oti_allocated`, `oti_claimed`, `status`.

---

**`GET /api/whitelist/tasks/:wallet_address`**
Returns all task completion statuses for this wallet:
- One-time tasks: completed or not (from `whitelist_task_completions`)
- Social/referral tasks: completed or not (from `whitelist_social_tasks`)
- Daily score: scored today or not + current streak count (from `whitelist_daily_scores`)
- Whitepaper: `questions_answered`, `rewards_claimed` (from `whitelist_whitepaper_progress`)

---

**`POST /api/whitelist/connect-telegram`**
Body: Telegram Login Widget auth data (`id`, `first_name`, `username`, `auth_date`, `hash`)
- Verify `hash` using HMAC-SHA256 with Telegram bot token (per official Telegram docs)
- Check `auth_date` is not older than 15 minutes — 401 if expired
- Check `telegram_user_id` not already used by a different wallet — 409 if collision
- Update `whitelist_participants`: `telegram_verified = true`, `telegram_user_id = id`
- Insert `whitelist_fingerprints` row (event_type='connect_telegram'), run collision checks
- Return `{ success: true }`

---

**`POST /api/whitelist/connect-x`**
OAuth 2.0 flow for X (Twitter):
- Complete OAuth callback, retrieve verified X handle
- Check `x_handle` not already used by a different wallet — 409 if collision
- Update `whitelist_participants`: `x_handle = handle`
- Insert `whitelist_fingerprints` row (event_type='connect_x'), run collision checks
- Return `{ success: true, x_handle }`

---

**Reward gate helper (internal — called at the start of every reward endpoint):**
1. `whitelist_participants.status = 'active'` — 403 `{ error: 'banned' }` if not
2. `whitelist_participants.telegram_verified = true` — 403 `{ error: 'telegram_required' }` if not
3. `whitelist_participants.x_handle IS NOT NULL` — 403 `{ error: 'x_required' }` if not
4. Insert `whitelist_fingerprints` row (event_type='claim_reward'), re-run IP + fingerprint collision checks. Flag if new collisions found. Do NOT block the claim — flagging is for admin review only.

---

**`POST /api/whitelist/task/daily-score`**
- Run reward gate
- Check `whitelist_daily_scores` for `(wallet_address, CURRENT_DATE AT TIME ZONE 'UTC')` — 409 if already scored today
- Trigger internal scoring API call for `wallet_address`
- Read `erp_base_daily_score_oti` from config — insert row into `whitelist_daily_scores`, add reward to `oti_allocated`
- Calculate streak: count consecutive daily entries ending today
- Return `{ success: true, oti_reward, streak_days }`

---

**`POST /api/whitelist/task/one-time`**
Body: `{ task_type }` — one of: `'wor_register'`, `'wor_report'`, `'dev_api'`
- Run reward gate
- Check `whitelist_task_completions` for `(wallet_address, task_type)` — 409 if already completed
- Verify the underlying action was actually done:
  - `wor_register`: query `wallet_ownership` table — check wallet_address exists (Phase 2 table)
  - `wor_report`: query `compromised_wallets` table — check a report by this wallet exists (Phase 2 table)
  - `dev_api`: query `subscriptions` table — check an active API key owned by this wallet exists
- If action not verified: 422 with `{ error: 'action_not_completed', message: '<clear instructions>' }`
- Read reward amount from config key corresponding to task_type — insert into `whitelist_task_completions`, add to `oti_allocated`
- Return `{ success: true, oti_reward }`

---

**`POST /api/whitelist/task/social`**
Body: `{ task_type, proof_url }` — task_type: `'post_tag'` or `'share'`
- Run reward gate
- Check `whitelist_social_tasks` for existing 'auto_verified' entry for this wallet + task_type — 409 if already done
- Auto-verify `proof_url`: HTTP HEAD request to URL, expect 200-299 — 422 `{ error: 'url_unreachable' }` if not
- Apply ERP multiplier: `reward = base_reward × (dcs_oti_remaining / dcs_total_oti)`
- Insert into `whitelist_social_tasks` (status='auto_verified'), add to `oti_allocated`
- Return `{ success: true, oti_reward }`

---

**`GET /api/whitelist/whitepaper/questions`**
Query param: `wallet_address`
- Run reward gate
- Check `whitelist_whitepaper_progress`: if `questions_answered >= 30` → 410 (all rounds complete)
- Fetch all question IDs this wallet has already answered (derived from progress rounds — store answered IDs in progress or a separate answered-questions log)
- Select 3 random active questions from `whitelist_whitepaper_questions` not yet answered by this wallet
- Sign the served question IDs into a short-lived JWT (15 min expiry, signed with `SESSION_SECRET`) — return the token alongside the questions so the submit endpoint can verify they were genuinely served
- Return `{ session_token, questions: [{ id, question_text, option_a, option_b, option_c, option_d }] }` — never include `correct_option`

---

**`POST /api/whitelist/whitepaper/submit`**
Body: `{ session_token, answers: [{ question_id, selected_option }] }` (exactly 3 items)
- Run reward gate
- Verify `session_token` JWT — reject 401 if expired or tampered
- Confirm the 3 `question_id` values in answers match those encoded in the session token (anti-cheat)
- Look up `correct_option` for each question_id in DB
- If any answer wrong: return `{ success: false, incorrect: [question_ids] }` — user may retry same round (do not advance progress)
- If all 3 correct:
  - Upsert `whitelist_whitepaper_progress`: `questions_answered += 3`, `rewards_claimed += 1`, `last_round_at = NOW()`
  - Read `erp_base_whitepaper_round_oti` from config, add to `oti_allocated`
  - Return `{ success: true, oti_reward, questions_answered, rewards_claimed }`

---

**Part C — Admin Dashboard ("Whitelist" tab — four sub-views)**

No Social Task Review Queue — social tasks are auto-verified (D49).

**1. Code Management Panel**
- `POST /api/admin/whitelist/generate-codes`: generates N unique OTI-XXXX-XXXX codes (uppercase alphanumeric, 4+4 chars), inserts into `whitelist_invites` with status 'active'. No maximum N (D53).
- Table: all codes with status, used_by_wallet (truncated), amount_contributed_usd, created_at
- Filter by status (active / used / banned / expired)
- Per-row "Ban" → `PATCH /api/admin/whitelist/codes/:id` → status = 'banned'
- Per-row "Expire" → status = 'expired'
- Generated codes displayed in a copyable list after generation

**2. Whitepaper Questions Manager**
- Table: all questions with is_active toggle, question preview, correct option
- "Add Question" form: question_text, option_a–d, correct_option (a/b/c/d)
- `POST /api/admin/whitelist/questions` — add
- `PATCH /api/admin/whitelist/questions/:id` — edit or toggle is_active
- `DELETE /api/admin/whitelist/questions/:id` — remove
- Pre-seed 100 non-technical questions at deploy time (see seed file)

**3. Flagged Accounts Panel**
- Table: `whitelist_flags` — wallet_address, flag_reason, related_wallets, flagged_at, status
- Filter by status (open / reviewed / cleared / banned)
- Per-row "Ban Wallet" → sets status = 'banned' on participant + all related_wallets + flag status = 'banned'
- Per-row "Clear" → flag status = 'cleared' (false positive)
- Per-row "Mark Reviewed" → flag status = 'reviewed'
- Expandable row detail: IP address + fingerprint_hash for Ahmad to assess
- `GET /api/admin/whitelist/flags` — all flags, filterable
- `PATCH /api/admin/whitelist/flags/:id` — update status, optionally ban related wallets

**4. Protocol Config Override**
- Input fields for every `whitelist_config` key, prefilled from DB
- "Save" button: `PUT /api/admin/whitelist/config`
- Labels: "Max whitelist slots (0 = unlimited)", "IP flag threshold", "Fingerprint flag threshold", plus all reward amount fields
- This is the only place Ahmad touches parameters — never a code deploy

---

**Part D — Smart Contracts (BNB Testnet → Mainnet — you handle everything, Ahmad not involved until workspace deletion)**

See D42 in DECISIONS.md. Ahmad is not involved at any point during this part.

**⚠️ UPDATED Session 24 — Two contracts required (D59):**

**Contract 1 (NEW): `OTIDCSContribution.sol` — BSC only, Layer 1**
Handles BNB + BSC stablecoins only. Do NOT attempt to accept ETH, SOL, TON, XRP, BTC or any other native coin here — those live on separate chains and are handled via Layer 2 backend addresses (see D59).
- Accepted tokens on BSC: BNB (native), USDT-BEP20, USDC-BEP20, BUSD-BEP20
- BNB/USD Chainlink on BSC mainnet: `0x0567F2323251f0Aab15c8dFb1967E4eaA47d42aEE`
- USDT, USDC, BUSD: stablecoins — no oracle, $1 = $1
- `setAcceptedToken(address token, address chainlinkFeed, bool isStable, bool enabled)` — owner only. Pass `address(0)` for chainlinkFeed on stablecoins.
- `contribute(address token, uint256 amount) external payable` — BNB via `msg.value` with `token = address(0)`; BEP-20 via `IERC20.transferFrom` (frontend must call approve first)
- BNB pricing: call Chainlink `latestRoundData()` on BNB/USD feed
- Emit `Contribution(address indexed wallet, address indexed token, uint256 amount, uint256 usdEquivalent)`
- `withdraw(address token)` — owner only, sweeps collected tokens/BNB to Ahmad's wallet
- Verify Chainlink feed address is live on BSC testnet before using on mainnet

**Layer 2 (non-BSC coins) — backend + DB, not a contract:**
ETH, BTC, SOL, TON, XRP, MATIC each have a dedicated OTI receiving address on their own chain. Part A adds `whitelist_contribution_addresses` (one row per coin/chain) and `whitelist_contributions` (each verified payment). Part C admin panel adds a "Pending Contributions" queue where Ahmad pastes a tx_hash → backend auto-fetches price from CoinGecko + verifies → records. See DECISIONS.md D59 for the full chain/API table.

**Contract 2 (original): `OTIWhitelistVesting.sol`** — unchanged design (see steps below)

**Step 1 — Generate deployer wallet:**
```js
const { ethers } = require("ethers");
const wallet = ethers.Wallet.createRandom();
console.log("Address:", wallet.address);
console.log("Private key:", wallet.privateKey);
```
Save as Replit env vars: `DEPLOYER_ADDRESS`, `DEPLOYER_PRIVATE_KEY`. Never put keys in any file.

**Step 2 — Fund the deployer wallet:**
- Testnet: https://testnet.bnbchain.org/faucet-smart → paste DEPLOYER_ADDRESS (free, no Ahmad needed)
- Mainnet: **you cannot fund this yourself.** Send DEPLOYER_ADDRESS to the Manager immediately after Step 1. The Manager relays it to Ahmad. Ahmad sends BNB (~0.05 BNB). Do not proceed to mainnet deploy until the Manager confirms Ahmad has funded the address. Verify the balance on BscScan before continuing.
- Do NOT proceed to mainnet with any contract until you see the BNB balance confirmed.

**Step 3 — Deploy `MockOTI.sol` — testnet only:**
- Standard ERC-20/BEP-20, `constructor(uint256 initialSupply)` mints to deployer
- Use `35000000 * 10**18` as supply
- BNB testnet (chainId 97) only. Save address as `MOCK_OTI_ADDRESS`.

**Step 4 — Deploy `OTIWhitelistVesting.sol` to testnet, then mainnet:**
- Testnet (chainId 97): owner = DEPLOYER_ADDRESS, token = MOCK_OTI_ADDRESS
- Mainnet (chainId 56): owner = DEPLOYER_ADDRESS, token = placeholder (D57 — no real OTI BEP-20 exists yet; use address(0) or a clearly documented stub)
- **Required contract functions:**
  - `vest(address participant, uint256 total_oti_amount)` — owner-only. 25% immediate, 75% linear daily over `vesting_duration_days`
  - `claimVested(address participant)` — callable by participant. Transfers unlocked OTI.
  - `setVestingDuration(uint256 days_)` — owner-only. Future vests only, not retroactive.
  - `setTokenAddress(address token_)` — owner-only. **Required** — lets Ahmad update the token address after the real OTI BEP-20 is deployed (D57).
  - `getVestingStatus(address participant)` — view. Returns total_allocated, total_claimed, currently_claimable, vesting_start, vesting_end.
  - `banParticipant(address participant)` — owner-only. Freezes remaining locked tokens.
- After testnet deploy: call `MockOTI.approve(vestingContractAddress, large_amount)` so the vesting contract can transfer tokens
- Verify all contracts on BscScan (testnet + mainnet) after each deploy

**Step 5 — End-to-end test on testnet:**
- `vest(testWallet, 1000 * 10^18)` → confirm 250 OTI immediately claimable (25%)
- Advance time → confirm linear daily unlock
- `claimVested(testWallet)` → confirm OTI transferred
- `banParticipant(testWallet)` → confirm remaining tokens frozen
- Paste raw output as evidence before proceeding to mainnet

**Step 6 — Handover package (deliver to Manager in one message at close):**
Manager relays to Ahmad. Do not send directly to Ahmad.
- `DEPLOYER_ADDRESS`
- `DEPLOYER_PRIVATE_KEY` — Ahmad saves this immediately and securely
- `MOCK_OTI_ADDRESS` (BNB testnet only)
- `VESTING_CONTRACT_ADDRESS_TESTNET`
- `VESTING_CONTRACT_ADDRESS_MAINNET`
- BscScan links for all contracts (testnet + mainnet)
- Note: "Mainnet vesting deployed with placeholder token. Call `setTokenAddress(real_OTI_address)` before going live."
- Gas cost incurred (BNB) — for Ahmad to reimburse

Ahmad saves the package, then deletes this workspace.

---

**Evidence required to close:**
- All DB tables created on Railway (Ahmad runs drizzle-kit push — you confirm the schema is ready)
- `/api/verify-invite`: valid code → success JSON; used code → 400; missing → 404 (paste raw responses)
- `/api/whitelist/state`: returns computed DCS committed rate + all current ERP bonus values
- `/api/whitelist/connect-telegram`: Telegram auth verified, participant.telegram_verified = true
- `/api/whitelist/connect-x`: OAuth completes, x_handle stored
- `/api/whitelist/task/daily-score`: first call succeeds with streak count; second call same day → 409
- `/api/whitelist/task/one-time`: 422 if action not done; success after action; 409 on duplicate (test all three task_types)
- `/api/whitelist/task/social`: URL auto-verified, reward credited; 409 on duplicate
- `/api/whitelist/whitepaper/questions`: returns 3 questions without correct_option, returns session_token
- `/api/whitelist/whitepaper/submit`: correct answers → reward credited + progress updated; wrong answers → returns incorrect IDs
- Fingerprint collision: register two wallets from same IP → confirm flag row appears in whitelist_flags
- Admin Whitelist tab: code management works (generate + ban + expire), questions CRUD works, flagged panel shows flag rows with ban/clear/review actions, config save updates DB values
- 100 whitepaper questions seeded and visible in admin questions manager
- BNB testnet contracts deployed and verified — paste BscScan testnet links
- BNB mainnet vesting contract deployed and verified — paste BscScan mainnet link
- vest() and claimVested() end-to-end on testnet — paste raw output
- setTokenAddress() callable by owner on mainnet contract — confirm in BscScan
- Full handover package delivered to Manager in one message
- Manager verifies all evidence and relays package to Ahmad before closing

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
