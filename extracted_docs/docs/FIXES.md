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
**Status:** Sent to Backend Builder July 8, 2026, in progress, not yet reviewed. This is the largest and highest-priority fix on the backend list — treat it as the current active item, nothing else goes to Backend Builder until it's verified done.

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

### BF11 — Re-verify "Try It Live" Widget Against Railway Backend 🔴 OPEN
**Priority:** Low-medium, small effort. The docs site's "Try It Live" widget had its API URL reverted during the Task 11 (docs site) remediation, but was never re-checked live after the redeploys. Just needs a live check that it's actually hitting the real Railway backend, not a stale/dev URL. Not yet assigned to a Builder.

### BF12 — Railway Deploys Don't Auto-Run Migrations 🔴 OPEN (optional)
**Priority:** Low, small effort, not urgent. Confirmed July 5, 2026: Railway's deploy pipeline only runs `pnpm install && build && start` — it does **not** run `drizzle-kit push`. Every schema change currently needs Ahmad to manually run the migration against the Railway production DB after deploying (this is how BF3 above was applied). Optional fix: add `drizzle-kit push` to `railway.json`'s `buildCommand` (a one-line change — NOT to `nixpacks.toml`, which stays sacred). Not yet assigned; only worth doing if Ahmad wants to remove the manual step.

### BF13 — DB Never Used as Cache Source — Scores Expire on Every Restart 🔴 OPEN — HIGH PRIORITY
**Priority:** High — directly impacts scale and trust accuracy. Discovered July 11, 2026 via full codebase audit. The DB (`chain_scores` table) is write-only from the cache's perspective — it receives every new score but is never consulted when answering a request. The only cache is a 500-entry in-memory LRU with a 5-minute TTL that is wiped on every Railway restart. Fix: make the score route check the DB first — if a score exists for that wallet+chain within the last 30 days, return it immediately without calling any external API. Ahmad's decision: 30-day validity, admin panel control over the rescore period (so Ahmad can force wallet rescores on a rolling daily basis, not all at once), keep the highest recorded score in sync. This fix is the single biggest lever for handling scale — the majority of repeat requests will never touch external APIs once it is in place.

### BF14 — Dead In-Memory History Write Still Running After BF6 🔴 OPEN
**Priority:** Low, small effort. BF6 fixed the history endpoint to read from the `chain_scores` DB table — but `recordHistory()` (lib/history.ts) is still being called on every fresh score, writing to an in-memory Map that nothing reads anymore. It is dead code burning memory on every request. Fix: remove the `recordHistory()` call from the score route and delete or archive `lib/history.ts`.

### BF15 — Compromised Wallet DB Check Runs on Every Request Including Cache Hits 🔴 OPEN
**Priority:** Medium. The compromised-wallet SELECT against the DB runs before the cache check — meaning even a fully cached request pays for a DB round-trip every single time. Under high load this becomes a constant unnecessary DB tax. Fix: move the compromised-wallet check after the cache check, or maintain a small in-memory set of compromised addresses that refreshes periodically (fast lookup, no DB call per request).

### BF16 — Chain Routing Duplicated Across 4+ Files — No Central Registry 🔴 OPEN
**Priority:** Medium — not urgent now but becomes a real bug risk as new chains are added. Discovered July 11, 2026. Chain routing logic (which fetcher to call for which chain) is a raw if/else block copy-pasted independently in: `score.ts` (routing), `score.ts` (validateRequest), `detect.ts` (auto-detection), `chainFamily.ts` (persistence), and likely `routes/chains.ts`. Adding one new chain today requires editing 4–5 files by hand and keeping them in sync manually. Fix: consolidate into a single chain registry (config map) that all four locations import from — one place to add a chain, everything else derives from it automatically.

### BF17 — Tron Smart Contract Diversity Structurally Broken (to = self address) 🔴 OPEN — HIGH
**Priority:** High — discovered during BF10 live result review, July 12, 2026. Root cause confirmed by full diagnostic audit: Tron's transaction shaping sets `to` and `from` fields to the wallet's own address rather than the actual counterpart/contract address. This means the "unique contract addresses" diversity component is structurally capped at ≤1 regardless of how many distinct contracts the wallet actually used. A wallet with 1 contract call can score 20/20 because the volume-threshold fallback path in scoring.ts drives the score, not real diversity. Fix: extract the actual Tronscan contract address from each transaction and use it for the unique-contract diversity count.

### BF18 — Sui Wallet Age Returns 0 for Receive-Only Wallets 🔴 OPEN — HIGH
**Priority:** High — discovered during BF10 live result review, July 12, 2026. Root cause confirmed by full diagnostic audit: `getSuiTransactions` queries `suix_queryTransactionBlocks` with `filter: { FromAddress: address }` — it only fetches transactions where the wallet is the SENDER. A wallet that has only ever received coins/objects and never sent a transaction returns zero results from this query, producing walletAgedays: 0 and score 0, even though it holds real assets and demonstrably exists on-chain. Fix: change the Sui transaction query to also fetch inbound transactions (filter by involvement, not just sender) and use the earliest timestamp across both directions.

### BF19 — TON Address Validation: Backend OK, Frontend May Still Reject Valid EQ Addresses 🔴 OPEN — MEDIUM
**Priority:** Medium — updated after full diagnostic audit, July 12, 2026. The backend TON address validator correctly accepts EQ/UQ user-friendly format (48 chars) and normalizes `+/` → `-_`. The live rejection seen during testing was likely coming from the frontend validator only. Fix scope: audit the frontend `validateAddress.ts` for TON to confirm it applies the same acceptance logic as the backend — specifically that it handles both the URL-safe (`-_`) and standard (`+/`) base64 character variants of EQ/UQ addresses. Backend validator appears correct; frontend validator needs verification and possible alignment.

### BF20 — Solana Smart Contract Diversity Structurally Broken (to = self address) 🔴 OPEN — MEDIUM
**Priority:** Medium. Root cause confirmed by full diagnostic audit, July 12, 2026. Same structural problem as BF17 (Tron) but for Solana: every transaction sets `to: walletAddress` (self-referential) rather than the actual program ID invoked. The "unique contract" diversity component is capped at 1 regardless of how many distinct on-chain programs the wallet actually used. Volume-threshold fallbacks in scoring.ts still allow active wallets to score reasonably — nothing is fabricated — but true program diversity is not being measured. Fix: extract the actual Solana program ID from each transaction's account keys and use it for the unique-program diversity count.

### BF21 — TON Jetton Holdings Inferred From Outgoing Messages, Not a Direct Balance Query 🔴 OPEN — MEDIUM
**Priority:** Medium. Root cause confirmed by full diagnostic audit, July 12, 2026. Toncenter v2 (the API in use) has no direct owner→Jetton-holdings endpoint. Holdings are inferred by examining outgoing message destinations — meaning wallets that received but never sent Jettons are undercounted, and wallets that sent Jettons they no longer hold are overcounted. Fix: switch to Toncenter v3 or tonapi.io which provide a direct Jetton holdings query for a given owner address.

### BF22 — Fantom Chain ID Points to Wrong Network (Sonic Instead of Fantom Opera) 🔴 OPEN — CRITICAL
**Priority:** Critical. Discovered during full diagnostic audit, July 12, 2026. `etherscan.ts` sets `CHAIN_ID.fantom = "146"` — but chain ID 146 is Sonic Mainnet, not Fantom Opera. The real Fantom Opera chain ID is 250. Every request for `?chain=fantom` is currently querying Sonic network data and returning it mislabelled as a Fantom score. The reverse lookup table also hardcodes `"250": "fantom"` — so a caller passing the correct Fantom chain ID still gets routed to Sonic. There is currently no way to score a real Fantom Opera wallet through OTI at all. Fix: change `CHAIN_ID.fantom` from `"146"` to `"250"`.

### BF23 — Scroll, Sepolia, and Holesky Listed as Supported But Not Implemented 🔴 OPEN — HIGH
**Priority:** High — documentation integrity issue. Discovered during full diagnostic audit, July 12, 2026. All three chains appear in the live docs and whitepaper as supported chains, but none of them exist anywhere in the codebase — not in the chain ID map, not in the Zod schema enum. Any request to `?chain=scroll`, `?chain=sepolia`, or `?chain=holesky` returns a 400 "Invalid enum value" error. Until they are implemented, they must be removed from all documentation, the docs site chain list, the whitepaper chain list, and the UI chain selector. Chain count across all public-facing materials must drop from 15 to 12 until these three are real.

### BF24 — All 7 Working EVM Chains Have No Transaction Pagination 🔴 OPEN — HIGH
**Priority:** High. Discovered during full diagnostic audit, July 12, 2026. Every working EVM chain (Ethereum, Polygon, Arbitrum, Avalanche, Linea, zkSync, and Fantom once chain ID is fixed) hard-caps at a single page: 1,000 native transactions, 500 token transfers, 500 internal transactions — all fetched oldest-first with no pagination loop. This is the exact same class of bug that BF10 fixed for all 5 non-EVM chains, but it was never applied to EVM. Live confirmed on Ethereum: vitalik.eth returns exactly txCount=1000 (the cap). Any wallet with more than 1,000 lifetime native transactions has all of its more-recent activity silently excluded — transaction count, timing patterns, and contract diversity are all computed from an old, incomplete slice of history. Fix: apply cursor/page pagination to EVM fetching the same way BF10 applied it to non-EVM, with a page cap and time budget for safety.

### BF25 — EVM Rate-Limit Errors Cached as Legitimate Zero-Activity Scores 🔴 OPEN — HIGH
**Priority:** High — data integrity issue. Discovered live during full diagnostic audit, July 12, 2026. Confirmed on Polygon and Linea during back-to-back testing. When Etherscan returns a transient `NOTOK` (rate limit) response, the fetch functions treat any non-array result as "no data" and the result (score: 0, txCount: 0) gets written to cache as if it were a real score. Subsequent requests within the cache TTL window return this false "zero activity" result as legitimate. A real, active wallet can appear as High Risk for minutes after a single rate-limit hit. Fix: detect non-array / NOTOK Etherscan responses and throw an error rather than returning empty data — never cache a result that came from a failed API call.

### BF26 — EVM Wallet Age Returns 0 for Token-Only Wallets (No Native Transactions) 🔴 OPEN — MEDIUM
**Priority:** Medium. Discovered during full diagnostic audit, July 12, 2026. Affects all 7 working EVM chains. Wallet age is computed only from the native transaction list (`txlist`) — it never examines token transfer history (`tokentx`). A wallet that has only ever received ERC-20 tokens (airdrop recipient, for example) with zero native ETH/MATIC/etc. transactions will show walletAgedays: 0 and txCount: 0, even though it has a real, timestamped on-chain history visible in the token transfer log. Fix: when `txlist` returns empty, fall back to the earliest timestamp from `tokentx` for wallet age calculation.

### BF27 — EVM Token Holdings Reflect Transfer History, Not Real Current Balance 🔴 OPEN — MEDIUM
**Priority:** Medium. Discovered during full diagnostic audit, July 12, 2026. Affects all 7 working EVM chains. Token holding diversity is computed by counting distinct contract addresses in the `tokentx` transfer event log — not by querying actual current balances. A wallet that received 100 tokens and then sent all of them away still registers as "holding" 100 distinct tokens. The signal measures historical transfer breadth, not genuine current portfolio diversity. Fix: use a real token balance query (e.g. Etherscan's `tokenbalance` endpoint or a multicall) to determine which tokens the wallet actually currently holds, rather than inferring from transfer history.

### BF28 — Solana Smart Contract Diversity: `to` Field Set to Wallet's Own Address 🔴 OPEN — MEDIUM
**Priority:** Medium. Full root cause from diagnostic audit, July 12, 2026. This is the exact same structural issue as BF20 (Solana) — confirmed and detailed here separately for clarity. Every Solana transaction in the current codebase sets `to: walletAddress` rather than extracting the actual program ID from the transaction's account keys. Unique contract count is structurally 1 for any wallet. Volume thresholds in scoring.ts keep active wallets scoring reasonably but genuine program diversity is not measured. Fix: extract the program ID (last account key, or the first non-wallet program key) from each Solana transaction and use it as the `to` value for diversity counting. (Note: this item supersedes and expands on BF20 for the Solana side — BF20 covers both Solana and Tron together; this entry is for tracking the Solana fix specifically.)

### BF29 — Sui Token Diversity Counts All Owned Object Types, Not Just Coin Objects 🔴 OPEN — MEDIUM
**Priority:** Medium. Discovered during full diagnostic audit, July 12, 2026. `suix_getOwnedObjects` returns every object the wallet owns — including NFTs, capability objects, and any other Move object type — and the current code treats all distinct object types as "tokens" for diversity scoring purposes. A wallet holding 30 NFTs of the same collection and 1 coin would show higher "token diversity" than a wallet holding 5 different coins. Fix: filter the owned objects query to `Coin<T>` types only, or use `suix_getAllCoins` which returns coin objects exclusively, before computing token diversity.

### BF30 — Bitcoin P2WSH Address Format Rejected by Validator 🔴 OPEN — LOW
**Priority:** Low. Discovered during full diagnostic audit, July 12, 2026. The Bitcoin address validator covers P2PKH (`1...`), P2SH (`3...`), P2WPKH bech32 (`bc1q...`, 38–39 chars after prefix), and P2TR taproot (`bc1p...`). P2WSH (pay-to-witness-script-hash, used for multisig) is also a valid bech32 `bc1q` address — but with a 32-byte script hash rather than a 20-byte key hash, making the address longer (59 characters after the `bc1q` prefix). The current regex's length range does not cover this form, so valid P2WSH addresses are rejected. Fix: extend the bech32 regex to accept the longer P2WSH length (62 chars total for `bc1q` + 59 chars).

### BF31 — TON Raw Address Format Rejected by Validator 🔴 OPEN — LOW
**Priority:** Low. Discovered during full diagnostic audit, July 12, 2026. The TON backend validator accepts user-friendly format addresses (EQ/UQ/kQ/0Q, 48 chars) but rejects the "raw" format (`workchain:hex64`, e.g. `0:83dfd552...`) which is commonly displayed by explorers and wallets as an alternate representation of the same address. Fix: extend the TON validator to also accept the raw `0:hex64` and `-1:hex64` (masterchain) formats.

### BF32 — Tron Hex Address Format Rejected by Validator 🔴 OPEN — LOW
**Priority:** Low. Discovered during full diagnostic audit, July 12, 2026. The Tron validator accepts only the standard base58check format (`T` + 33 chars). Tron also has an internal hex format (`41` + 40 hex chars, 42 chars total) used by some tools and APIs. This format is less commonly seen by end users but is a valid Tron address representation. Fix: extend the Tron validator to also accept the `41`-prefixed hex format.

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

### FF17 — "AI-Native Tell" Cleanup: Copy, Tone, and Emoji Across Homepage, Docs, and Whitepaper 🔴 OPEN — HIGH PRIORITY
**Raised by Ahmad:** July 9, 2026. The largest open frontend fix — a full read-through of all three public-facing surfaces (marketing homepage, developer docs site, whitepaper) for anything that reads as AI-generated rather than a deliberately designed product site:
1. **Copy/tone** — flag and rewrite any AI-sounding boilerplate phrasing.
2. **Emoji usage** — audit and replace/remove emoji everywhere it appears (not just the Trust Signals/Use Cases sections) with a proper icon set (Lucide/Heroicons) in mint where an icon is still needed.
3. **Any other AI tell** — visual or textual, caught during the read-through.

**Status:** Scoped, not yet sent to a Builder — confirm priority with Ahmad before sending (this is broad enough that it may be worth splitting into three smaller passes — homepage, docs, whitepaper — rather than one large one). Not yet assigned an order relative to BF10 on the backend side; they're independent and can run in parallel once both are prioritized.

---

## Adding a New Fix

1. Decide fix vs. task first: does this repair/polish something that already exists (→ fix, goes here) or build something new (→ task, goes in `TASKS.md` / `BACKEND_TASKS.md` / `FRONTEND_TASKS.md`)?
2. Add it to the correct builder's list here, ordered by size — small/quick fixes near the top of their section, larger ones lower.
3. Tell the Builder to add it to their own copy of this file — same rule as tasks, copies never auto-sync.
