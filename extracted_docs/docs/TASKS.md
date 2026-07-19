# OTI — Master Task List
> Last updated: July 19, 2026 (session 17 — Phase 2B Revenue Campaign task queue added: Tasks 19–22. Etherscan key rotation moved up from Phase 4 to Task 19 — prerequisite for campaign wallet pre-scoring at scale.) | Maintained by: Development Manager

---

## How to Read This File

This is the complete history and queue of everything either Builder has built or will build — new pages, new systems, new features. Each Builder also has their own copy of just their tasks (`BACKEND_TASKS.md` / `FRONTEND_TASKS.md`) with full prompt text. Bug fixes and cleanup work are tracked separately in `FIXES.md`.

## Builder Roles

- **Backend Builder** — Node.js/Express API on Railway, scoring engine, database, admin routes, bots
- **Frontend Builder** — React/Vite app on Vercel, scoring UI, marketing site, docs site, admin panel UI

---

## ✅ Completed Tasks

### TASK 8 — Frontend: Professional Results Page Redesign ✅
Full visual rebuild of the wallet score results page to the locked black/mint design system: chain-color ring gauge, score tier labels (HIGHLY TRUSTED → HIGH RISK), Trust Signals card with weighted bars, truncated wallet address with copy button, Share (native share sheet) + Save as Image (3× PNG export), "⚑ Report this wallet" placeholder link (activates once WOR/Phase 2 ships), footer. This redesign also established the OTI color system (see `BUILDER_ONBOARDING.md` for the locked token table) that every subsequent page must use.

### TASK 8B — Frontend: Professional Wallet Input Page Redesign ✅
Matching redesign of the wallet input/landing screen to the same visual system — logo sizing/position, chain icon visibility and sizing (including zkSync/Linea), spacing, report-link styling. Verified live.

### TASK 9 — Frontend: Admin Panel UI ✅
Built the full admin panel: login, Dashboard, API Keys (create/list/edit/delete with a reveal-once creation modal), Query History, Cache management, Plan Configs. URL-only route (no public nav link), per Ahmad's decision.

### TASK 9-BACKEND — Backend: Admin Panel API Routes ✅
Built the `/api/admin/*` route surface backing Task 9 — stats, keys CRUD, plan configs CRUD, query history, cache inspection — all behind the `x-admin-secret` auth (`FIXES.md` BF5).

### TASK 11A — Frontend: Marketing Homepage + Restructure Vercel App ✅
Restructured the single Vercel app into a proper front door: the scoring tool moved intact to `/score`, and `/` became a full marketing homepage — Hero, chain row, How It Works, Trust Signals, Use Cases, Get the API (cURL example), Find Us/Integrations, footer with social links. Crisp.chat chat widget embedded (ID left blank until Ahmad provides it). Live at `otiscore.vercel.app`. Verified by Manager via cache-busted screenshot + JS bundle inspection, and by Ahmad directly.

### TASK 11B — Frontend: Whitepaper Page ✅
Built `/whitepaper` — full-length technical/business document (Executive Summary through remaining sections), sticky table-of-contents sidebar on desktop (accordion on mobile), section numbers in mint, print-to-PDF support via `window.print()`, shares the homepage's nav/footer and color system. Three post-launch rendering issues (body text color, mobile horizontal scroll, Roadmap section removal) were fixed — see `FIXES.md` FF16.

### TASK 16 — Backend: Wallet Ownership Registry (WOR) — Phase 2 ✅
10 endpoints + `wallet_ownership` table. EIP-191 sig verification (ethers v6), bcrypt passkey (cost 12), 15-min challenge replay protection, dual-factor self-report (sig + passkey → instant 0-score, generic 401 on either failure). Activates `wallet_links` and `compromised_wallets` tables. 4 admin endpoints (paginated registry, manual flag/unflag). Score cache invalidated on compromise. Verified end-to-end with 2 real EVM keypairs, full raw response evidence provided. Railway migration (`drizzle-kit push`) confirmed applied July 14, 2026. Two noted deviations: compromised_wallets written via Drizzle ORM (already proven in prod); compromised/revoked re-registration treated as UPSERT not hard 409.

### TASK 11 — Frontend: Developer Docs Site ✅
Docusaurus site covering Getting Started, API Reference (weighted signal shape), Score Explanation, Supported Chains, Rate Limits, and code examples in JS/Python/cURL. Deployed as its own Vercel project (`oti-docs`, pnpm-based build) and proxied onto the main site at `otiscore.vercel.app/docs/` via `vercel.json` rewrites. Confirmed fully live July 9, 2026 — `/docs`, `/docs/`, and `/docs/api-reference` all verified 200 via curl. One open follow-up in `FIXES.md` (BF11 — re-verify "Try It Live" widget against the real backend).

---

### TASK 17 — Frontend: WOR UI — Phase 2 ✅
/register (3-step: address check → MetaMask sign → passkey set) and /report (3-step: status check → sign + passkey → confirm dialog) pages live. "⚑ Report this wallet" link activated on results page. Admin WOR tab added (Registry, Compromised, Manual Override sub-views). Verified end-to-end by Ahmad with real Trust Wallet on July 14, 2026 — registration, self-report, and 0% compromised score all confirmed live. Follow-up UI/UX and ecosystem wiring polish tracked as FF24.

### TASK 18 — Frontend: Services Hub Page (`/services`) ✅ — July 15, 2026
New page at `/services` — a portal/hub that surfaces all OTI services as clickable cards so any user (wallet owner, developer, curious visitor) can immediately find what they need without knowing the URL structure. Cards: Score a Wallet → /score, Wallet Ownership Registry (Register / Report) → /register + /report, API for Developers → /docs, Whitepaper → /whitepaper, plus placeholder cards for future services. Homepage stays unchanged. `/services` link added to the navbar. Page must use the locked OTI color system, be fully responsive (375px mobile), and match the visual quality of the rest of the app.
**Builder's report + Ahmad live confirmation, July 15, 2026:** `Services.tsx` built with MarketingNavbar/Footer chrome, 2-col→1-col (≤720px) card grid, `.marketing-service-*` CSS on locked color system, "Services" in navbar (desktop+mobile). `npm run build` clean, 0 TS errors. Ahmad confirmed live. Closed.

## 🔴 Active Queue — Phase 2B Revenue Campaign

Build order is strict — each task depends on the previous one being confirmed live by Ahmad.
One task per Builder at a time. Do not queue next task until current is verified.

---

### TASK 19 — Backend: Etherscan API Key Rotation
**Builder: Backend Builder | Priority: FIRST — prerequisite for everything below**

Add round-robin Etherscan API key rotation to the scoring pipeline so OTI can pre-score millions of Ethereum wallets for the XMTP campaign without hitting the 100K calls/day single-key limit.

**What to build:**
- Add Railway environment variable `ETHERSCAN_API_KEYS` — comma-separated list of up to 10 Etherscan API keys (e.g. `key1,key2,key3,...`)
- In `chainRegistry.ts` (wherever `etherscanApiKey()` is defined), replace the single-key return with a round-robin counter across the array: keep a module-level integer index, increment on every call (mod array length), return that key
- If `ETHERSCAN_API_KEYS` is set, use the array. If not set, fall back to the existing single `ETHERSCAN_API_KEY` env var — backward compatible, nothing breaks
- No other file changes. No other logic changes. Scope is the `etherscanApiKey()` function only

**Why first:** The XMTP sender script needs a pre-built list of Ethereum wallets with score ≥ 75 in `chain_scores`. Without rotation, scoring 1M wallets takes 30–50 days on a single free key. With 10 keys, it takes 3–5 days.

**Ahmad to provide:** 10 Etherscan API keys (registered from separate email accounts, not all in one session). Manager will pass them to Builder as Railway env var values — never in code.

**Evidence required to close:** Builder deploys, Ahmad confirms via a live API call that rotating keys are being used (Builder logs which key index served each request temporarily, then removes the log).

---

### TASK 20 — Backend: OTI Signing Endpoint + BAS Schema Registration
**Builder: Backend Builder | Priority: SECOND — unblocks smart contract and XMTP script**
**Depends on: Task 19 confirmed live**

Two parts, done in sequence by the same Builder in the same session.

**Part A — BAS Schema Registration (do this first):**
- Register the OTI attestation schema on BNB Chain's BAS (BNB Attestation Service) — this is an on-chain transaction, not a code task
- Schema fields: `address wallet, uint256 score, string tier, uint256 issuedAt, uint256 expiresAt`
- Ahmad must sign and pay for this transaction (small BNB gas cost ~$0.01) — Builder provides the exact BAS schema registration transaction for Ahmad to confirm
- The resulting schema UID (a bytes32 hash) is hardcoded into the smart contract in Task 21 — Builder must record and document it

**Part B — Signing Endpoint:**
- New file: `src/routes/sign.ts`
- Add Railway env var: `OTI_SIGNING_KEY` (private key — Ahmad generates this key pair, stores private key in Railway, gives public key to Builder for smart contract)
- Endpoint: `POST /api/sign/score` — protected by existing `adminAuth.ts` middleware
- Input: `{ wallet_address, chain, expiry_timestamp }`
- Logic:
  1. Check `compromised_wallets` — return 403 if flagged
  2. Look up `chain_scores` for this wallet — return 403 if score < 75 or not found
  3. Sign payload using `ethers.solidityPackedKeccak256(['address','uint256','uint256'], [wallet_address, score, expiry_timestamp])` then `signingWallet.signMessage(hash)`
  4. Return `{ wallet_address, score, expiry_timestamp, signature }`
- The signing key lives only in Railway env vars — never in code, never logged

**Evidence required to close:** Builder calls the endpoint with a real Ethereum wallet that has a score ≥ 75 in chain_scores, pastes the raw JSON response. Ahmad verifies the endpoint rejects a wallet with score < 75 and rejects a compromised wallet.

---

### TASK 21 — Backend: Smart Contract (BNB Chain) + XMTP Sender Script
**Builder: Backend Builder | Priority: THIRD**
**Depends on: Task 20 confirmed live + BAS schema UID from Task 20 Part A**

Two parts, done in sequence.

**Part A — Smart Contract (Solidity, BNB Chain):**
- Language: Solidity 0.8.x
- Chain: BNB Chain (chainId 56) — deploy to BNB testnet first, verify end-to-end, then mainnet
- Deploy cost: ~$5–20 in BNB (Ahmad funds this)
- Contract logic:
  1. Store OTI public key (from Task 20 key pair) and BAS schema UID (from Task 20 Part A) as immutable constructor args
  2. Store Chainlink BNB/USD price feed: `0x0567F2323251f0Aab15c8dFb1967E4eaA47d42aEE`
  3. `getRequiredBNB()` — returns `1e26 / latestRoundData()` (exactly $1 in BNB wei)
  4. `mintAttestation(uint256 score, uint256 expiry, bytes memory sig) external payable`
     - `require(msg.value >= getRequiredBNB(), "Send exactly $1 in BNB")`
     - `require(score >= 75, "Score below threshold")`
     - `require(expiry > block.timestamp, "Signature expired")`
     - `require(!minted[msg.sender], "Already minted")`
     - Verify OTI signature: `ecrecover(keccak256("\x19Ethereum Signed Message:\n32", keccak256(abi.encodePacked(msg.sender, score, expiry)))) == OTI_PUBLIC_KEY`
     - `require(verifyOTISignature(...), "Invalid OTI signature")`
     - `minted[msg.sender] = true`
     - Call `bas.attest(buildAttestationRequest(msg.sender, score))` — uses BAS schema UID
     - `emit AttestationMinted(msg.sender, score, attestationUID, msg.value)`
  5. `withdraw()` — owner only, sweeps BNB revenue to Ahmad's wallet
- Verify on BscScan after mainnet deploy

**Part B — XMTP Sender Script (Node.js):**
- Runtime: Node.js — lives outside the Railway API server (standalone script, run locally or on a separate process)
- SDK: `@xmtp/node-sdk` v5+
- Dependencies: existing Railway PostgreSQL connection (read-only), `@xmtp/node-sdk`
- Logic:
  1. Query `chain_scores` for Ethereum wallets with `overall_score >= 75` and `scored_at > NOW() - INTERVAL '30 days'`
  2. Batch `canMessage()` check — filter to XMTP-enabled wallets only
  3. For each eligible wallet: call `POST /api/sign/score` to get signed payload (expiry = 24 hours from now)
  4. Construct `wallet_sendCalls` XMTP message with the signed payload embedded as calldata, `chainId: 56` (BNB Chain), contract address, and `$1 in BNB` value
  5. Message content (what user sees in Coinbase Wallet):
     ```
     Your OTI Trust Score: [score] / 100 — HIGHLY TRUSTED

     You qualify for an OTI Trust Attestation — a permanent on-chain
     record of your wallet's trustworthiness, verified across Ethereum.

     This attestation is recognised by DeFi protocols for reduced
     collateral requirements and whitelist access.

     → Approve $1 in BNB to mint your attestation permanently on BNB Chain.

     [Transaction Request: OTI Attestation Contract · ~$1 · BNB Chain]
     ```
  6. Rate: 3,000 messages per 5-min window per sender wallet
  7. Track sent wallets in a local SQLite or flat file to avoid re-sends
- Ahmad runs this script manually after end-to-end test confirms everything works

**Evidence required to close:** Builder deploys contract to testnet. Ahmad sends $1 test BNB on testnet → attestation minted → visible on BNB testnet BAS explorer. Then mainnet deploy + same test on mainnet. Builder pastes BscScan contract address and BAS attestation UID from the test mint.

---

### TASK 22 — Frontend: Campaign Conversion Dashboard
**Builder: Frontend Builder | Priority: FOURTH — revenue happens without this**
**Depends on: Task 21 smart contract deployed to mainnet (need contract address + ABI)**

Build a simple admin-only conversion dashboard inside the OTI Assessment Replit artifact that tracks campaign performance in real time.

**What to build:**
- Set up Moralis Streams webhook on the `AttestationMinted` contract event — writes to a lightweight `campaign_conversions` table (wallet, score, attestation_uid, amount_paid_bnb, amount_paid_usd, timestamp)
- Dashboard view (React/Vite, admin-only, no public link):
  - Total messages sent (manual input or flat file)
  - Total attestations minted
  - Conversion rate %
  - Revenue in BNB + USD equivalent (live BNB price)
  - Live-updating — no page refresh needed
  - Simple table of recent conversions (wallet truncated, score, timestamp, USD value)
- Moralis free tier: 40K compute units/day — sufficient for campaign volume

**Evidence required to close:** Ahmad sees at least the testnet `AttestationMinted` event from Task 21 appearing in the dashboard live. No mainnet conversions required to close the task — the infrastructure just needs to be working.

---

## ⛔ PHASE 5 GATE — Distribution Channel Tasks Below Depend On Phase 1 Being Fully Closed

Every bot reply, widget badge, and extension popup links back to the marketing homepage, docs site, and whitepaper. Do not start any task below until Ahmad confirms Phase 1 (see `ROADMAP.md`) is fully closed — specifically task 1D (create operational API keys via admin panel).

### TASK 12 — Backend: Telegram Bot
Telegraf (Node.js) bot in `/bots/telegram/` inside the backend repo, deployed as a second Railway process. Commands: `/score [address] [chain]`, `/help`, `/about`. Env vars needed: `TELEGRAM_BOT_TOKEN`, `OTI_BOT_API_KEY`. See `ROADMAP.md` Phase 5 for full context.

### TASK 13 — Backend: Discord Bot
discord.js bot in `/bots/discord/`, deployed as a third Railway process. Slash commands `/score`, `/help`, rich embed responses. Env vars needed: `DISCORD_BOT_TOKEN`, `DISCORD_CLIENT_ID`, `OTI_BOT_API_KEY`.

### TASK 14 — Backend: Embeddable Widget
Vanilla JS, zero-dependency widget served from Railway at `GET /widget.js`. One-line integration snippet for site owners. Uses a shared widget API key.

### TASK 15 — Frontend/Extension: Firefox Extension (then Chrome)
Separate GitHub repo (`oti-firefox-extension`). Content script auto-detects wallet addresses on Etherscan/OpenSea/BscScan and injects a score badge. Firefox first (free to publish), Chrome after first revenue. No backend changes required.

---

## Not Yet Scoped Into Task Prompts

These exist in `ROADMAP.md` as phases but have not yet been broken into individual task prompts for a Builder:

- **Phase 2 — Wallet Ownership Registry (WOR):** `wallet_ownership` table, registration/report endpoints, EIP-191 verification (Backend); registration + report UI (Frontend).
- **Phase 3 — Monetization infrastructure:** self-serve developer portal, Pro/Enterprise plan rows + checkout, Stripe + Coinbase Commerce integration.
- **Phase 4 — Growth features:** score history UI, multi-chain wallet comparison, wallet portfolio view, webhook alerts, enterprise exchange compliance path.

---

## Keeping This File Updated

Both Builders update this file — but only when the Manager explicitly tells them to (marking a task ✅, or adding a newly assigned task). Builder copies (`BACKEND_TASKS.md`, `FRONTEND_TASKS.md`) are separate physical files that never auto-sync with this one or with each other — every status change must be explicitly relayed to the relevant Builder.
