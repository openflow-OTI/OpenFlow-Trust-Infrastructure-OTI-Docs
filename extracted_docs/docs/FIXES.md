# OTI — Fixes Log
> Last updated: July 10, 2026 (session 8 — created this file. Ahmad's direction: bug fixes, corrections, cleanup, and polish are NOT tasks — a "task" builds new capability, a "fix" repairs or refines something already built. Keeping them separate keeps TASKS.md from bloating with churn and stops confusion about what's actually new work vs. maintenance. All fix-type items previously listed in TASKS.md/BACKEND_TASKS.md/FRONTEND_TASKS.md have been moved here.) | Maintained by: Development Manager

---

## How This File Works

- **A task** (lives in `TASKS.md` / `BACKEND_TASKS.md` / `FRONTEND_TASKS.md`) builds something that didn't exist before — a new page, a new endpoint, a new system.
- **A fix** (lives here) repairs, corrects, hardens, or polishes something that already exists — a bug, a wrong value, a security hole, a copy/tone pass, a visual polish round.
- Each list below is numbered smallest/quickest fix first, largest/most involved last — so it's easy to see what can be knocked out fast vs. what needs real time.
- Same rules as everywhere else: one active item at a time per Builder, Manager adds fixes here first, Builder only updates their own copy when told to explicitly.

---

## Backend Fixes

### BF1 — GET /admin/stats 500 Error ✅
**Fixed:** July 7, 2026. Root cause: stats handler had no error handling — any DB query failure caused an unhandled rejection and returned raw HTML 500. Fix: per-query isolation with individual try/catch blocks, each defaulting to 0 on failure. Result: `/admin/stats` always returns HTTP 200 with valid JSON. Verified live.

### BF2 — PATCH /admin/plan-configs/:id 404 Error ✅
**Fixed:** July 7, 2026. Root cause: handler matched `WHERE plan_name = :id`, but the frontend was sometimes sending a UUID — matched nothing. Fix: dual lookup, detects if the param is a UUID (`WHERE id`) or a name string (`WHERE plan_name`). Both forms now work. Verified live.

### BF3 — subscriptions.updated_at Missing Column ✅
**Fixed:** July 6, 2026. Added `updated_at` column to the `subscriptions` table via Drizzle migration; `PATCH /api/admin/keys/:id` now sets it on every update. Railway production DB migration was run manually via the Railway Console (see the note in MANAGER_HANDOVER.md — Railway does not auto-run migrations on deploy). Verified: `GET /api/admin/keys` no longer 500s.

### BF4 — GET/POST /admin/keys Full Resolution ✅
**Fixed:** July 7, 2026. Root cause: the Railway `subscriptions` table predates the Drizzle schema in the repo — real columns are `id, api_key, plan, owner_address, created_at, expires_at, updated_at` with **no `status` column and no `email` column**. All handlers now use raw SQL instead of Drizzle ORM selects on this table (which crash with "column does not exist"). GET uses a column-name fallback mapper; POST inserts only the columns that actually exist; the `active_keys` stat no longer filters on a `status` column that isn't there. Verified live — Ahmad created enterprise/pro/free keys and confirmed edit/delete worked.

### BF5 — Admin Route Authentication (security) ✅
**Fixed:** July 5, 2026. Every `/api/admin/*` route was callable by anyone on the internet — no auth at all, meaning any visitor could create/delete API keys or flag wallets. Fix: `adminAuth.ts` middleware added, requires a correct `x-admin-secret` header on all admin routes, returns 401 otherwise. This had to ship before anything else touched the admin surface. Verified live on Railway.

### BF6 — Score History Served From Memory Instead of Database ✅
**Fixed:** July 6, 2026. `GET /api/score/:address/history` was reading from an in-memory `Map` (wiped on every Railway restart) instead of the `chain_scores` database table, even though the data was already being written there. Fix: endpoint now queries `chain_scores` directly, with an optional `?chain=` filter (auto-detects chain family when omitted), ordered by `scored_at` DESC, capped at 50 records. Verified live.

### BF7 — Signal Scores Missing Weighted Values in API Response ✅
**Fixed:** July 6, 2026. The API returned raw 0–100 signal scores with no indication of how much each signal actually contributed to the overall score. Fix: added a `signalWeighting.ts` transformer (the protected `scoring.ts` algorithm itself was never touched) so the response now includes `{ score, weighted, maxWeight }` per signal, on both the fresh-compute and cache-hit paths. This was a prerequisite for the frontend's weighted signal bars and for any external developer integrating the API. Verified live.

### BF8 — Bitcoin Wallet Age Returning Wrong Value ✅
**Fixed:** July 6, 2026. `getBitcoinTxs()` was only fetching the most recent ~50 transactions (a single page) instead of full history, so old wallets reported an age of only a few days. Fix: now paginates backwards through mempool.space's `/address/:address/txs/chain/:last_seen_txid` until the true first transaction is found, with a 40-page safety cap to bound latency for hyperactive wallets. Verified on Railway: previously-broken old wallets now correctly return walletAgedays in the thousands.

### BF9 — Plan Limit Enforcement Hardening ✅
**Fixed:** July 7, 2026. A production-wide bug was found during testing: every API request using a valid key was silently returning raw HTML 500 since launch, caused by a Drizzle ORM schema mismatch (a `status` column defined in the repo's schema but missing from the actual Railway DB). Fixed across four call sites — `apiKeyAuth.ts`, `score.ts` (compromised-wallet check), and both the DELETE and PATCH handlers in `admin.ts` — all switched from Drizzle ORM to raw SQL for this table. Verified live: free plan correctly 429s on the 3rd daily request, enterprise (unlimited) never 429s, PATCH edits confirmed persisting.

### BF10 — Signal Accuracy Audit & Cross-Chain Fix ✅ DONE — July 12, 2026
**Status:** Audit and initial fix pass completed and verified July 12, 2026. Remaining chain-specific issues surfaced by this audit were split out into their own tracked items (BF17–BF34 below) rather than left bundled here — this entry covers the original 5-signal audit and Bitcoin redistribution fix only.

**The problem:** the 5 scoring signals were designed around EVM chains. Bitcoin, Solana, TON, Tron, and Sui have fundamentally different data models — no ERC-20 tokens, no internal transactions, no smart contracts in the EVM sense — but are currently being scored with EVM-specific logic, producing wrong and misleading results for every single non-EVM wallet. Confirmed examples: the Satoshi genesis wallet (`1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa`) scores ~51 days wallet age when the correct answer is 5,700+ days; Bitcoin wallets score 0/20 on Token Holding because ERC-20 tokens don't exist on Bitcoin; Bitcoin wallets show a fabricated "1 smart-contract tx" despite Bitcoin having no smart contracts.

**What's being audited/fixed, per signal:**
- **Wallet Age (25%)** — re-verify the Bitcoin fix (BF8) against the Satoshi wallet after a cache flush; verify Solana/TON/Tron/Sui each pull the correct first-tx timestamp.
- **Transaction Count (20%)** — verify every non-EVM chain returns a real count, not a placeholder/default.
- **Token Holding Behavior (20%)** — Bitcoin has no tokens; instead of scoring 0/20 (which unfairly penalizes BTC wallets), either redistribute this signal's weight across the other 4 or score it neutral (50/100). Verify Solana SPL token diversity is actually being scored, not defaulting to 0. Verify TON/Tron/Sui individually.
- **Smart Contract Interactions (20%)** — Bitcoin has no smart contracts; stop fabricating data, apply the same redistribute-or-neutral fix. Verify Solana program interactions (the equivalent concept) are scored correctly. Verify TON/Tron/Sui.
- **Transaction Timing Patterns (15%)** — non-EVM chains currently show "0 internal transactions" as a proxy; must use real transaction timestamps instead.

**Definition of done:** Satoshi wallet scores > 5,000 days wallet age; no signal fabricates data for a chain where that data type doesn't exist; non-applicable signals are redistributed or neutral, never a default 0; all 15 chains tested with at least one known wallet; a short test log is committed to the repo; Backend Builder reports results to the Manager before this is marked done.

**Sacred files — do not touch while fixing this:** `src/lib/scoring.ts`, `nixpacks.toml`.

---

### BF11 — Re-verify "Try It Live" Widget Against Railway Backend ✅
**Fixed:** July 14, 2026. API_BASE in oti-docs/src/components/WalletScorer.jsx confirmed correct (https://workspaceapi-server-production-5c0c.up.railway.app/api). Live test: Vitalik's wallet scored 96/100, cached:false — widget hitting real Railway backend. Backend confirmed shared fetch client uses setBaseUrl() at runtime, no hardcoded URL at library level.

### BF12 — Railway Deploys Don't Auto-Run Migrations 🔴 OPEN (optional)
**Priority:** Low, small effort, not urgent. Confirmed July 5, 2026: Railway's deploy pipeline only runs `pnpm install && build && start` — it does **not** run `drizzle-kit push`. Every schema change currently needs Ahmad to manually run the migration against the Railway production DB after deploying (this is how BF3 above was applied). Optional fix: add `drizzle-kit push` to `railway.json`'s `buildCommand` (a one-line change — NOT to `nixpacks.toml`, which stays sacred). Not yet assigned; only worth doing if Ahmad wants to remove the manual step.

### BF13 — DB Never Used as Cache Source — Scores Expire on Every Restart 🟡 IN PROGRESS — Frontend Builder (backend ✅ confirmed July 14, 2026)
**Priority:** High. Discovered July 11, 2026. The DB (chain_scores table) is write-only — never consulted when answering requests. Only cache is a 500-entry in-memory LRU (5-min TTL, wiped on restart).

**Full scope (expanded per Ahmad, July 14, 2026):**
- L1 (in-memory LRU) + L2 (DB, 30-day per-wallet window) cache architecture
- Keep-highest score logic — never overwrite a higher score with a lower one
- Configurable rescore window (stored in DB, not hardcoded at 30 days)
- Global cache ON/OFF toggle — when OFF, every request bypasses both L1 and DB and hits the on-chain API directly (live data only, not written to cache)
- Admin endpoints: DELETE /api/admin/cache/:address, DELETE /api/admin/cache (all), PATCH /api/admin/rescore-window, PATCH /api/admin/cache/toggle, GET /api/admin/cache/stats
- Frontend Builder wires admin dashboard UI after backend is confirmed done (new task)

### BF14 — Dead In-Memory History Write Still Running After BF6 ✅
**Fixed:** July 14, 2026. Removed recordHistory() call from score route, deleted lib/history.ts. Confirmed no other imports existed. Score route verified working after removal.

### BF15 — Compromised Wallet DB Check Runs on Every Request Including Cache Hits ✅
**Fixed:** July 14, 2026. Cache key now built immediately after PLAN_UPGRADE_REQUIRED check. Cache hit returns immediately with zero DB calls. resolveChainFamily + compromised DB query only run on cache miss. Annotated BF15 in code for traceability.

### BF16 — Chain Routing Duplicated Across 4+ Files — No Central Registry ✅
**Fixed:** July 14, 2026. New file: src/lib/chainRegistry.ts — exports SUPPORTED_CHAINS, ALL_CHAIN_ZOD_ENUM, CHAIN_FAMILY_MAP, PLAN_UPGRADE_REQUIRED. score.ts and routes/history.ts now import from registry. chainFamily.ts left independent (separate workspace package — importing from api-server would create circular dependency). Adding a new chain now requires editing one file only.

### BF17 — Tron Smart Contract Diversity Structurally Broken (to = self address) 🔴 OPEN — HIGH
**Priority:** High — discovered during BF10 live result review, July 12, 2026. Root cause confirmed by full diagnostic audit: Tron's transaction shaping sets `to` and `from` fields to the wallet's own address rather than the actual counterpart/contract address. This means the "unique contract addresses" diversity component is structurally capped at ≤1 regardless of how many distinct contracts the wallet actually used. A wallet with 1 contract call can score 20/20 because the volume-threshold fallback path in scoring.ts drives the score, not real diversity. Fix: extract the actual Tronscan contract address from each transaction and use it for the unique-contract diversity count.

### BF18 — Sui Wallet Age Returns 0 for Receive-Only Wallets ✅
**Priority:** High — discovered during BF10 live result review, July 12, 2026. Root cause confirmed by full diagnostic audit: `getSuiTransactions` queries `suix_queryTransactionBlocks` with `filter: { FromAddress: address }` — it only fetches transactions where the wallet is the SENDER. A wallet that has only ever received coins/objects and never sent a transaction returns zero results from this query, producing walletAgedays: 0 and score 0, even though it holds real assets and demonstrably exists on-chain. Fix: change the Sui transaction query to also fetch inbound transactions (filter by involvement, not just sender) and use the earliest timestamp across both directions.
**Fixed and pushed:** July 13, 2026. Backend reported `walletAgedays: 1162`, cross-checked against a manual calculation of 1161.57 days from the real earliest transaction — match confirmed. Pushed to `main` same day. Not yet independently re-verified live post-deploy by Manager — do that on the next signal sweep.

### BF19 — TON Address Validation: Backend OK, Frontend May Still Reject Valid EQ Addresses ✅
**Priority:** Medium — updated after full diagnostic audit, July 12, 2026. The backend TON address validator correctly accepts EQ/UQ user-friendly format (48 chars) and normalizes `+/` → `-_`. The live rejection seen during testing was likely coming from the frontend validator only. Fix scope: audit the frontend `validateAddress.ts` for TON to confirm it applies the same acceptance logic as the backend — specifically that it handles both the URL-safe (`-_`) and standard (`+/`) base64 character variants of EQ/UQ addresses. Backend validator appears correct; frontend validator needs verification and possible alignment.
**Closed:** July 12, 2026, via FF20 — Frontend's `validateAddress.ts` widened to accept both base64 variants. Verified live.

### BF20 — Solana Smart Contract Diversity Structurally Broken (to = self address) 🔴 OPEN — MEDIUM
**Priority:** Medium. Root cause confirmed by full diagnostic audit, July 12, 2026. Same structural problem as BF17 (Tron) but for Solana: every transaction sets `to: walletAddress` (self-referential) rather than the actual program ID invoked. The "unique contract" diversity component is capped at 1 regardless of how many distinct on-chain programs the wallet actually used. Volume-threshold fallbacks in scoring.ts still allow active wallets to score reasonably — nothing is fabricated — but true program diversity is not being measured. Fix: extract the actual Solana program ID from each transaction's account keys and use it for the unique-program diversity count.

### BF21 — TON Jetton Holdings Inferred From Outgoing Messages, Not a Direct Balance Query ✅
**Priority:** Medium. Root cause confirmed by full diagnostic audit, July 12, 2026. Toncenter v2 (the API in use) has no direct owner→Jetton-holdings endpoint. Holdings are inferred by examining outgoing message destinations — meaning wallets that received but never sent Jettons are undercounted, and wallets that sent Jettons they no longer hold are overcounted. Fix: switch to Toncenter v3 or tonapi.io which provide a direct Jetton holdings query for a given owner address.
**Fix reported, July 13, 2026:** `getTonJettonBalances()` now calls tonapi.io `/v2/accounts/{addr}/jettons`, filters `is_scam`/zero-balance entries.
**Fresh re-verification, July 13, 2026:** Test wallet `EQAj-peZGPH-cC25EAv4Q-h8cBXszTmkch6ba6wXC8BM4xdo` (largest USDT holder on TON, never queried this session — cache hit impossible). Raw tonapi.io count: 45 non-spam non-zero Jetton balances. Railway response: `cached: false`, `uniqueTokens: 45` — exact match. Closed.

### BF22 — Rename "Fantom" Chain Entry to "Sonic" (Fantom Opera Migrated, No Longer Served by Etherscan) ✅
**Priority:** Critical. Discovered during full diagnostic audit, July 12, 2026. `etherscan.ts` set `CHAIN_ID.fantom = "146"` — chain ID 146 is Sonic Mainnet, not Fantom Opera (250), so every `?chain=fantom` request was querying Sonic and mislabeling it as Fantom.
**Live verification (Builder, July 12, 2026):** Switching to chain ID 250 does not restore real Fantom data — Etherscan V2 no longer serves Fantom Opera under any chain ID (confirmed against the live 64-chain chainlist); Fantom migrated/rebranded to Sonic, and ftmscan.com (the old standalone Fantom explorer/API) no longer resolves. Real, distinct Fantom Opera data is not available from our provider.
**Decision (Ahmad, July 12, 2026):** Do not chase chain ID 250. Rename the chain entry itself: `"fantom"` → `"sonic"`, keep chain ID `"146"`. This serves real Sonic data under its real name instead of mislabeling it as Fantom or dropping the chain.
**Fix:** In `etherscan.ts`, rename `fantom` → `sonic` throughout `CHAIN_ID` and the reverse lookup table (chain ID stays `"146"`). Update any user/API-facing chain name strings ("Fantom" / "Fantom Opera" → "Sonic" / "Sonic Mainnet") in responses (`/api/chains`, `chain` field) and anywhere else the chain is labeled.
**Fixed:** July 12, 2026. Renamed across `etherscan.ts`, `score.ts`, `admin.ts`, `history.ts`, `lib/db/chainFamily.ts` — enums/schemas updated, "fantom" removed everywhere in code, `sonic` mapped to the `evm` family. Verified live: `GET /api/chains` lists `sonic`/146, no `fantom`; `?chain=fantom` now correctly rejected; `?chain=sonic` on `0x21be370D5312f44cb42ce377BC9b8a0cef1a4C83` returns real data (txCount 2), cross-checked against sonicscan.org's public explorer (also shows 2 transactions). Remaining "Fantom" mentions are in non-functional docs only (README.md, CHANGELOG.md, ARCHITECTURE.md, TASKS.md) — doc sweep held pending Manager confirmation, outside BF22's code scope.

### BF23 — Scroll, Sepolia, and Holesky Listed as Supported But Not Implemented ✅
**Fixed:** July 14, 2026. Backend confirmed all three absent from Zod enum, CHAIN_ID map, chainFamily, and detect.ts. GET /api/chains confirmed not listing them (live response verified). Frontend: absent from chains.ts, supported-chains.md, WalletScorer.jsx. Whitepaper EVM chain list cleaned (removed "Scroll, Sepolia, Holesky" from Section 06). Chain count stays at 15 across all public materials — BSC/Base/Optimism fill those slots as gated chains (Ahmad's decision).

### BF24 — All 7 Working EVM Chains Have No Transaction Pagination ✅
**Priority:** High. Discovered during full diagnostic audit, July 12, 2026. Every working EVM chain (Ethereum, Polygon, Arbitrum, Avalanche, Linea, zkSync, and Fantom once chain ID is fixed) hard-caps at a single page: 1,000 native transactions, 500 token transfers, 500 internal transactions — all fetched oldest-first with no pagination loop. This is the exact same class of bug that BF10 fixed for all 5 non-EVM chains, but it was never applied to EVM. Live confirmed on Ethereum: vitalik.eth returns exactly txCount=1000 (the cap). Any wallet with more than 1,000 lifetime native transactions has all of its more-recent activity silently excluded — transaction count, timing patterns, and contract diversity are all computed from an old, incomplete slice of history. Fix: apply cursor/page pagination to EVM fetching the same way BF10 applied it to non-EVM, with a page cap and time budget for safety.
**Fixed and pushed:** July 13, 2026. `etherscan.ts` now paginates up to 10 pages × 1,000 = 10,000 records per fetcher, returns `hitApiCap: true` when the ceiling is hit, surfaced to the client as `historyTruncatedByApiCap: true`. Verified live on `0x000000000000000000000000000000000000dead` (burn address): txCount 10,000 exactly (cap hit), `historyTruncatedByApiCap: true`, walletAgedays 3,395 (correct — burn address active since ~2015). Pre-fix this wallet would have shown ≤1,000.

### BF25 — EVM Rate-Limit Errors Cached as Legitimate Zero-Activity Scores ✅
**Priority:** High — data integrity issue. Discovered live during full diagnostic audit, July 12, 2026. Confirmed on Polygon and Linea during back-to-back testing. When Etherscan returns a transient `NOTOK` (rate limit) response, the fetch functions treat any non-array result as "no data" and the result (score: 0, txCount: 0) gets written to cache as if it were a real score. Subsequent requests within the cache TTL window return this false "zero activity" result as legitimate. A real, active wallet can appear as High Risk for minutes after a single rate-limit hit. Fix: detect non-array / NOTOK Etherscan responses and throw an error rather than returning empty data — never cache a result that came from a failed API call.
**Fixed and pushed:** July 13, 2026. Backend reported a live captured log line showing a genuine "3/sec" NOTOK mid-pagination auto-retrying instead of being cached as zero — real reproduction of the failure path being handled correctly, not a synthetic test. Pushed to `main` same day. Not yet independently re-verified live post-deploy by Manager.

### BF26 — EVM Wallet Age Returns 0 for Token-Only Wallets (No Native Transactions) ✅
**Priority:** Medium. Discovered during full diagnostic audit, July 12, 2026. Affects all 7 working EVM chains. Wallet age is computed only from the native transaction list (`txlist`) — it never examines token transfer history (`tokentx`). A wallet that has only ever received ERC-20 tokens (airdrop recipient, for example) with zero native ETH/MATIC/etc. transactions will show walletAgedays: 0 and txCount: 0, even though it has a real, timestamped on-chain history visible in the token transfer log. Fix: when `txlist` returns empty, fall back to the earliest timestamp from `tokentx` for wallet age calculation.
**Fixed and verified, July 13, 2026:** Test wallet `0x238a2bbc89e402df5a4513687c0bb7dbff6676aa` (top LOOKS airdrop recipient — confirmed zero native txs via Blockscout `items: []`, `next_page_params: null`; real token transfers present with earliest timestamp 2026-06-21 10:38:47 UTC). Railway response: `cached: false`, `walletAgedays: 22` — exact match with June 21 → July 13 = 22 days. Pre-fix this wallet would have returned `walletAgedays: 0`. Closed.

### BF27 — EVM Token Holdings Reflect Transfer History, Not Real Current Balance ✅
**Priority:** Medium. Discovered during full diagnostic audit, July 12, 2026. Affects all 7 working EVM chains. Token holding diversity is computed by counting distinct contract addresses in the `tokentx` transfer event log — not by querying actual current balances. A wallet that received 100 tokens and then sent all of them away still registers as "holding" 100 distinct tokens. The signal measures historical transfer breadth, not genuine current portfolio diversity. Fix: use a real token balance query (e.g. Etherscan's `tokenbalance` endpoint or a multicall) to determine which tokens the wallet actually currently holds, rather than inferring from transfer history.
**Fixed and pushed:** July 13, 2026. `getCurrentTokenBalances()` queries Etherscan's `tokenbalance` endpoint per contract (capped at 30 lookups), response now includes `currentTokenHoldings` filtered to nonzero real balances. Verified live on `0x1a9C8182C09F50C8318d769245beA52c32BE35BC` (Uniswap deployer): 25 real live balances returned (USDT, UNI, others), not derived from transfer history.

### BF28 — Solana Smart Contract Diversity: `to` Field Set to Wallet's Own Address 🔴 OPEN — MEDIUM
**Priority:** Medium. Full root cause from diagnostic audit, July 12, 2026. This is the exact same structural issue as BF20 (Solana) — confirmed and detailed here separately for clarity. Every Solana transaction in the current codebase sets `to: walletAddress` rather than extracting the actual program ID from the transaction's account keys. Unique contract count is structurally 1 for any wallet. Volume thresholds in scoring.ts keep active wallets scoring reasonably but genuine program diversity is not measured. Fix: extract the program ID (last account key, or the first non-wallet program key) from each Solana transaction and use it as the `to` value for diversity counting. (Note: this item supersedes and expands on BF20 for the Solana side — BF20 covers both Solana and Tron together; this entry is for tracking the Solana fix specifically.)

### BF33 — Smart Contract Diversity `to=self` Bug Also Suspected on TON and Sui ✅
**Priority:** High. Discovered via score-card screenshot review, July 12, 2026. BF17 (Tron) and BF20/BF28 (Solana) already confirmed that the "unique contract" diversity count is structurally capped at 1 because `to`/`from` gets set to the wallet's own address instead of the real counterpart/program/contract. Screenshot evidence now shows the exact same fingerprint — "Smart Contract Interactions: 1 [smart-contract tx(ns)], 20/20" — on Solana, TON, and Sui score cards for three completely unrelated wallets, alongside genuinely varying counts (0, 1, 2, 5) on EVM chains for the same test wallet, which argues the EVM path is fine and this is specific to the non-EVM chains. Not yet root-caused for TON/Sui specifically — may be the same `to=self` pattern, or a separate fallback-to-1 default. Fix: when doing the BF17/BF20/BF28 fix pass, also audit TON's and Sui's transaction-shaping code for the same self-referential `to` field (or any hardcoded/fallback "1" default) and fix all four chains (Tron, Solana, TON, Sui) together since it's very likely the same root cause repeated per-chain.
**Backend report (July 13, 2026):** Fix implemented on all 4 chains — root cause on Tron/Sui matched the write-up (`to: walletAddress` hardcoded); Solana needed a new sampling step (parsed transaction detail for up to 300 signatures, extracting real program IDs); TON's bug was disguised (`in_msg.destination` is also self-referential for the wallet's own account) and fixed to prefer `out_msgs[0].destination`. Backend's own test wallets: Tron 641, Solana 3, TON 2, Sui 2 — all off the stuck-at-1 fingerprint.
**Manager live verification (July 13, 2026), signal-by-signal sweep across 11 chains:** Solana, TON, and Sui confirmed fixed on live score cards — real varying counts (2, 92, 2) for wallets outside Backend's own test set. Tron flagged for re-check: wallet `TVj7RNVHy6thbM7BWdSe9G6gXwKhjhdNZS` showed "1 smart-contract txns" matching the pre-fix fingerprint.
**Tron re-investigation (Backend, July 13, 2026):** `TVj7RN...dNZS` is not a wallet — it's the KLV token's own smart contract address (confirmed via Tronscan `accountType: 2`). A contract's own tx history is calls made *to* it, so it structurally can't show >1 "contract it called." `contractInteractions: 1` is honest for this address, not a bug. Separately flagged as its own question (not in BF33's scope): should contract addresses even be scoreable as if they were wallets? Tracked as BF36 below.
**Second real bug found and fixed during this re-check:** the Tron "is this a contract call" filter was `contractType !== 1`, which also counted native Tron protocol ops (bandwidth/energy delegation, freeze/unfreeze, voting — types 57/58/etc.) as smart-contract interactions, using the delegation target as a fake "contract." This is how Backend's earlier reported figure of 641 for wallet `TAzVMjYGNLs5NxRSnnALDLdrKkUHJwajPg` was produced — re-verified on-chain, 100% of that wallet's sampled transactions were resource-delegation ops, zero real contract calls; the 641 was fabricated diversity, not genuine dApp usage. Fixed: restricted the check to `contractType === 31` (TriggerSmartContract, the only type that means "called an arbitrary smart contract"). Re-verified: `TAzVMjYGN...` now correctly `contractInteractions: 0`; `TVj7RN...dNZS` still `1` (correct, contract-address reason above); a real EOA `TSkPaxomBrMRc5a6GwhXBhsb1DBGMGA9iV` confirmed calling a live DEX-type contract → `contractInteractions: 6`, verified against real distinct `toAddress` values on genuine TriggerSmartContract transactions pulled directly from Tronscan.
**Closed:** July 13, 2026. All 4 chains (Tron, Solana, TON, Sui) confirmed fixed with real, verified on-chain diversity — no fabricated defaults.

### BF36 — Contract Addresses: Decided NOT to Reject, Score Like Any Other Address 🟢 CLOSED (won't-fix, by decision)
**Priority:** N/A — resolved by product decision, not a code fix. Surfaced during BF33 re-investigation, July 13, 2026, when `TVj7RNVHy6thbM7BWdSe9G6gXwKhjhdNZS` (the KLV token's own smart contract, `accountType: 2` per Tronscan) was found scoreable like a normal wallet.
**Decision (Ahmad, July 13, 2026, reversed from the same-day initial call to reject):** OTI does not verify identity for any address today, so "no person behind a contract" isn't a meaningfully different bar than a normal wallet address, where a person is also just assumed. Many contract addresses genuinely are how a real person or org holds funds (multisig/smart-wallet accounts; on TON, *every* wallet is technically a contract). Scoring a contract doesn't fabricate anything — it reports real on-chain history for that address, same as any wallet. Decision: do not gatekeep by address type. Score everything.
**How this connects to the still-open bigger gap:** the actual risk this was trying to guard against (bots, scam contracts, non-human/malicious patterns) isn't unique to contracts — a plain EOA can be a bot too. That risk is intentionally deferred to the future bot/suspicious-wallet behavioral detection phase (see `ROADMAP.md`), not solved by a blanket contract rejection now.
**No Backend work required.** Do not implement contract-detection/rejection logic for this item.

### BF35 — "Internal Txns" Label Misleading on Non-EVM Chains (Timing Patterns Signal) ✅
**Priority:** Low, cosmetic/data-labeling only — scores look correct. Discovered via signal-by-signal screenshot sweep, July 13, 2026. The Transaction Timing Patterns signal displays "X internal txns" on Bitcoin, Solana, Tron, TON, and Sui score cards, but none of these chains have an EVM-style internal transaction. Values don't track sensibly with tx count or score: Tron shows exactly 300, identical to its total transaction count; Sui shows 0 while still scoring a perfect 15/15. Likely a leftover label/field from before BF10's real-timestamp fix. Not assigned yet — low priority, does not appear to affect actual scoring.
**Root cause and fix (July 13, 2026):** the Solana shaper pushed one synthetic `internalTxs` entry per SPL token account held (using the mint address as `to`), silently inflating `scoreContractInteractions`'s `internalCount` by the number of tokens held — unrelated to real program calls. Removed that push from `shapeSolanaData()`; real program interactions still flow through `regularTxs`/`contractCount` independently. Post-fix state per chain: Bitcoin 0 (correct, no such concept), EVM real count (Etherscan `txlistinternal`), TON real count (genuine `out_msgs`), Tron real count (`TriggerSmartContract` calls only), Sui 0 (correct), Solana 0 (was token-count, now correct).

### BF34 — TON Wallet Age Returns 0 Days for Wallets With Real Transaction History ✅
**Priority:** High. Discovered via score-card screenshot review, July 12, 2026. TON wallet `EQAOobN5eCwKmqhNV-l_TLePKHharXyh2-tFaYrNP14Ew5aK` (the one Backend independently verified as real and active, txCount 370+) scores `walletAgedays: 0` on the live "Try It Live" widget despite having 373 real transactions. This is the same class of bug as BF18 (Sui wallet age 0 for receive-only wallets, root cause: only querying sender-side transactions) — likely TON's age calculation is also filtering by one direction only (e.g. only outgoing messages) and missing the earliest real transaction. Fix: audit TON's wallet-age query the same way BF18 audits Sui's — confirm it examines both inbound and outbound transaction history for the earliest timestamp, not just one direction.
**Fixed and pushed:** July 13, 2026. Backend reported "confirmed fixed with raw evidence" (first item closed in this round, before BF18). Pushed to `main` same day. Not yet independently re-verified live post-deploy by Manager — do that on the next signal sweep.

### BF29 — Sui Token Diversity Counts All Owned Object Types, Not Just Coin Objects ✅
**Priority:** Medium. Discovered during full diagnostic audit, July 12, 2026. `suix_getOwnedObjects` returns every object the wallet owns — including NFTs, capability objects, and any other Move object type — and the current code treats all distinct object types as "tokens" for diversity scoring purposes. A wallet holding 30 NFTs of the same collection and 1 coin would show higher "token diversity" than a wallet holding 5 different coins. Fix: filter the owned objects query to `Coin<T>` types only, or use `suix_getAllCoins` which returns coin objects exclusively, before computing token diversity.
**Fixed and pushed:** July 13, 2026. `getSuiCoins()` now uses `suix_getAllCoins`, fully paginated. Verified live on `0x2ccb2cef019695d24474e5d32126ff3a26c5fb876f2e3fd0c96e7ae419f8c6f8`: direct RPC call returned 7 distinct coin types, Railway response `uniqueTokens: 7` — exact match.

### BF30 — Bitcoin P2WSH Address Format Rejected by Validator ✅
**Priority:** Low. Discovered during full diagnostic audit, July 12, 2026. The Bitcoin address validator covers P2PKH (`1...`), P2SH (`3...`), P2WPKH bech32 (`bc1q...`, 38–39 chars after prefix), and P2TR taproot (`bc1p...`). P2WSH (pay-to-witness-script-hash, used for multisig) is also a valid bech32 `bc1q` address — but with a 32-byte script hash rather than a 20-byte key hash, making the address longer (59 characters after the `bc1q` prefix). The current regex's length range does not cover this form, so valid P2WSH addresses are rejected. Fix: extend the bech32 regex to accept the longer P2WSH length (62 chars total for `bc1q` + 59 chars).
**Fixed and pushed:** July 13, 2026. Added `/^bc1q[ac-hj-np-z02-9]{58}$/i` to `isBitcoinAddress` in `bitcoin.ts`. Verified live on a real, currently-valid P2WSH address (`bc1quhruqrghgcca950rvhtrg7cpd7u8k6svpzgzmrjy8xyukacl5lkq0r8l2d`, 62 chars, regex-matched locally): Railway returned HTTP 200 with score 50/txCount 649/walletAgedays 2, versus HTTP 400 pre-fix.

### BF37 — TON Wallet-History Cap Not Disclosed to Client ✅
**Priority:** Medium. Discovered July 13, 2026 during BF34 re-verification: a wallet's Railway-reported age (1,031 days) fell 54 days short of its true oldest transaction (1,085 days, via tonapi.io full pagination). Root cause was not "no pagination" — a prior round (labeled BF10 in code comments) already added cursor-based pagination to `toncenter.ts` (100/page, 20-page cap = 2,000 tx ceiling, 15s time budget, `ageIsLowerBound` flag set when the cap is hit). The actual gap: `ageIsLowerBound` was computed but never surfaced to the client — unlike BF24's EVM disclosure (`historyTruncatedByApiCap`), so a TON wallet with >2,000 lifetime transactions silently reported an understated age with no indication it was a lower bound.
**Fixed and pushed:** July 13, 2026. `routes/score.ts` now sets the shared `historyTruncatedByApiCap` flag when `tonAgeIsLowerBound === true`, matching the EVM disclosure pattern. Verified live on `EQDtFpEwcFAEcRe5mLVh2N6C0x-_hJEM7W61_JLnSF74p4q2`: txCount 2,000 (cap hit, 20×100), `historyTruncatedByApiCap: true` now present, walletAgedays 1,031 (correctly disclosed as a lower bound, true age 1,085 confirmed independently via tonapi.io).

### BF31 — TON Raw Address Format Rejected by Validator ✅
**Priority:** Low. Discovered during full diagnostic audit, July 12, 2026. The TON backend validator accepts user-friendly format addresses (EQ/UQ/kQ/0Q, 48 chars) but rejects the "raw" format (`workchain:hex64`, e.g. `0:83dfd552...`) which is commonly displayed by explorers and wallets as an alternate representation of the same address. Fix: extend the TON validator to also accept the raw `0:hex64` and `-1:hex64` (masterchain) formats.
**Fixed:** July 12, 2026. `isTonAddress` in `toncenter.ts` extended to accept raw `workchain:hex64` format alongside EQ/UQ/kQ/0Q. Backend independently derived and verified two real, live TON addresses (EQ and UQ forms) against Toncenter to confirm correctness on genuine data.

### BF32 — Tron Hex Address Format Rejected by Validator ✅
**Priority:** Low. Discovered during full diagnostic audit, July 12, 2026. The Tron validator accepts only the standard base58check format (`T` + 33 chars). Tron also has an internal hex format (`41` + 40 hex chars, 42 chars total) used by some tools and APIs. This format is less commonly seen by end users but is a valid Tron address representation. Fix: extend the Tron validator to also accept the `41`-prefixed hex format.
**Fixed and pushed:** July 13, 2026. `TRON_HEX_RE` added to `isTronAddress()`; `toTronscanAddress()` converts hex → base58check via sha256d checksum before calling Tronscan. Verified live on `41a614f803b6fd780986a42c78ec9c7f77e6ded13c` (hex form of the well-known USDT-TRC20 contract): checksum conversion independently produced `TR7NHqjeKQxGTCi8q8ZY4pL8otSzgjLj6t` (matches the real, publicly known USDT contract address), Railway returned HTTP 200 with real data instead of a 400.

---

## Frontend Fixes

### FF1 — Logo Rendering as Broken Spiral Math ✅
**Fixed:** July 5, 2026. `Logo.tsx` replaced spiral SVG math with a plain `<img src="/logo.jpg">` tag; `generateScoreCard.ts` updated to use `loadImage()` before drawing to canvas.

### FF2 — Logo Blurry at 34px (SVG Replace) ✅
**Fixed:** July 6, 2026. Switched to the original SVG logo file directly as `public/logo.svg` — crisp on Retina/high-DPI screens, zero blur. No reconstruction needed, the asset already existed.

### FF3 — txCount Displayed as Raw, Sometimes Misleading Number ✅
**Fixed:** July 7, 2026. `formatMetadata.ts` and `generateScoreCard.ts` both now show "1,000+ transactions" once `txCount >= 1000`, instead of a raw capped number that looked like the true count. Verified live.

### FF4 — Homepage Scrollbar/Layout Bug ✅
**Fixed:** July 7, 2026. `html`/`body`/`root` given proper `min-height` so the browser scrollbar sits at the true far-right edge instead of floating mid-page.

### FF5 — useAnonymousLimit Hook Called Wrong Endpoint ✅
**Fixed:** July 7, 2026. Hook was calling the admin-protected `/admin/plan-configs` route (401 on a public page) instead of the correct public `GET /api/config/anonymous-limit`. Fixed, with a fallback to displaying 3 when the API returns null.

### FF6 — API Key Not Shown Correctly on Creation ✅
**Fixed:** July 7, 2026. POST response field name mismatch — frontend read `data.apiKey`, backend returned `data.api_key`. Fixed the field name and the TypeScript interface; the reveal modal now correctly shows the full key once with a copy button and a "never shown again" warning.

### FF7 — Admin Panel Desktop Layout Constrained to Mobile Width ✅
**Fixed:** July 7, 2026. The app's `max-width: 720px` container rule was also constraining the admin panel on desktop. Fixed with a CSS `:has()` selector that removes the width constraint specifically when the admin shell is present; mobile layout unaffected.

### FF8 — API Keys Tab Broke Entirely When the List Failed to Load ✅
**Fixed:** July 7, 2026. The "+ New Key" button and the whole tab used to disappear if the keys list failed to fetch. Fixed: the button is now always visible, errors show inline with a Retry button, and the table only renders once the fetch actually succeeds.

### FF9 — Rate Limit Display Hardcoded Instead of Live ✅
**Fixed:** July 7, 2026. Homepage showed a hardcoded "3 per day" regardless of the real configured value. `useAnonymousLimit.ts` now calls the live public endpoint and displays the real `daily_limit`, invalidating and refreshing automatically after an admin change.

### FF10 — API Health Not Visible Anywhere in the UI ✅
**Fixed:** July 7, 2026. The `useHealth` hook existed but was wired into zero components. Connected it to a small status dot in the navbar (green/red/loading), with a tooltip and proper `aria-live`/`role="status"` accessibility attributes.

### FF11 — General UI Polish Pass ✅
**Fixed:** July 5, 2026. Homepage title styling, back button style, chain icon sizing, removed dead CSS, added an `isKnownChain()` guard in `useScore.ts` to avoid crashes on unrecognized chains.

### FF12 — Admin API Calls Hitting Wrong Base Path ✅
**Fixed:** July 7, 2026. Every admin panel screen 404'd because the frontend called `/admin/*` while the real backend routes live at `/api/admin/*` (consistent with the rest of the API's `/api/*` prefix). Fixed in `adminClient.ts` and everywhere else admin routes are called.

### FF13 — Anonymous Rate Limit Badge Not Updating After Admin Change ✅
**Fixed:** July 8, 2026. A full audit found the real root cause: `setEditId(null)` inside the Plan Configs mutation's `onSuccess` raced with React 18 batching and unmounted the edit row before the cache update could be trusted — compounded by there being no `invalidateQueries` fallback at all, a closure-based (not server-response-based) value being written to cache, and "unlimited" (`null`) being silently displayed as "3". Fixed by consolidating all cache updates into the mutation's global `onSuccess`, always calling `invalidateQueries`, using the actual PATCH response value, and correctly displaying "Unlimited" for a null `daily_limit`. Verified live by Ahmad: changing the limit to 42 updated the homepage instantly; setting it to unlimited correctly showed "Unlimited".

### FF14 — Mobile Pinch/Double-Tap Zoom Breaking Carefully-Sized Layouts ✅
**Fixed:** July 8, 2026. Every page is precisely sized for mobile, but users could still pinch/double-tap zoom and break the intended layout. Fixed with the accessibility-conscious compromise Ahmad chose: viewport `maximum-scale=5, minimum-scale=1` (not a full `user-scalable=no` lock, which would violate WCAG 1.4.4) plus a `touch-action: manipulation` CSS backstop to curb iOS's accidental double-tap-zoom specifically. Deliberate pinch-zoom still works up to 5x; desktop zoom completely unaffected. Verified across homepage, results page, and admin dashboard.

### FF15 — Homepage Visual Polish: Contrast, Animated CTA Jank, Spacing, Density ✅
**Fixed:** July 8, 2026. Four issues bundled into one pass: (1) input placeholder text was too dim to read, set to near-white; (2) the "Try an example →" moving border caused real jank on mobile Chrome because it animated an expensive `@property`/conic-gradient repaint every frame — rebuilt using a GPU-cheap `transform: rotate()` on a masked pseudo-element, respecting `prefers-reduced-motion`; (3) the whole page felt oversized/zoomed on mobile — audited and reduced sizing across every element; (4) spacing felt cramped, especially below the rate-limit badge — reclaimed space from the sizing fix was redistributed as breathing room without adding page height. Verified live by Ahmad via screen recording.

### FF16 — Whitepaper Rendering Issues ✅
**Fixed:** July 8, 2026. Three issues found during Manager's live verification of the whitepaper page: (1) body/paragraph text was rendering in the dimmed grey token instead of white — fixed, section numbers stay mint; (2) mobile horizontal scroll/shift — fixed, now fits one column at 375px; (3) Ahmad requested the Roadmap section be removed from the whitepaper entirely — removed, all following sections/TOC renumbered sequentially.

### FF18 — Fantom → Sonic Rename Across Frontend, Docs, and Whitepaper ✅
**Fixed:** July 12, 2026. Companion to backend BF22 (Fantom Opera migrated/rebranded to Sonic under Etherscan V2, chain ID stays 146). Renamed "Fantom"/"Fantom Opera" → "Sonic"/"Sonic Mainnet" throughout `chains.ts`, `ChainIcon.tsx`, swapped `fantom.svg` → `sonic.svg`, `schema.gen.ts`, `Whitepaper.tsx`, and the docs site (`api-reference.md`, `supported-chains.md`). Verified live.

### FF19 — Sui and TON "Try It Live" Example Addresses Were Invalid/Wrong Wallet ✅
**Fixed:** July 12, 2026. Docs site `WalletScorer.jsx` example addresses were bad: the Sui example used an EVM address (Vitalik's) instead of a real Sui address, and the TON example was a stale/broken 47-character address unparseable by Toncenter (not a validator bug — confirmed dead example data). Replaced with two real, live-verified wallets: Sui `0xac5bceec1b789ff840d7d4e6ce4ce61c90d190a7f8c4f4ddf0bff6ee2413c33c` (scores 91, Highly Trusted) and TON `EQAOobN5eCwKmqhNV-l_TLePKHharXyh2-tFaYrNP14Ew5aK` (373 real transactions). Verified live.

### FF20 — TON Address Validator Rejected Standard (`+/`) Base64 Variant ✅
**Fixed:** July 12, 2026. `validateAddress.ts` only accepted the URL-safe base64 character set for TON EQ/UQ addresses, rejecting otherwise-valid addresses using the standard `+/` variant. Regex widened to accept both. Verified live.

### FF21 — Docs Site Vercel Build Config Broken for Subproject ✅
**Fixed:** July 12, 2026. `oti-docs/vercel.json` build configuration fixed so the docs subproject builds/deploys correctly on Vercel. Verified live.

### FF17 — "AI-Native Tell" Cleanup: Copy, Tone, and Emoji Across Homepage, Docs, and Whitepaper ✅
**Fixed:** July 14, 2026. Full read-through of all three surfaces. No emoji found anywhere (Lucide icons already in use). Homepage: 4 copy rewrites (How It Works steps, Use Cases). Whitepaper: 9 copy rewrites across Sections 02, 03, 04, 09, 10, 11, 12 — AI boilerplate compressed and grounded. Docs site: already clean, no changes needed.

---

## Adding a New Fix

1. Decide fix vs. task first: does this repair/polish something that already exists (→ fix, goes here) or build something new (→ task, goes in `TASKS.md` / `BACKEND_TASKS.md` / `FRONTEND_TASKS.md`)?
2. Add it to the correct builder's list here, ordered by size — small/quick fixes near the top of their section, larger ones lower.
3. Tell the Builder to add it to their own copy of this file — same rule as tasks, copies never auto-sync.
