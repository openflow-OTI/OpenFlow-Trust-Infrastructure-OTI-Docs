# OTI — Master Task List
> Last updated: July 30, 2026 (session 22 — Phase 0 Ecosystem Whitelist Infrastructure task prompts added: Tasks 23–28. OTI Economics confirmed: 35M supply, dual bonding curve DCS + ERP parameters.) | Maintained by: Development Manager

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

## 🔴 Phase 0 — Ecosystem Whitelist Infrastructure

Build order is strict. Each task must be confirmed live by Ahmad before the next begins.
One task per Builder at a time — hard rule.

---

### TASK 23 — Frontend: GitHub Repo Cleanup
**Builder: Frontend Builder | Priority: FIRST in Phase 0**
**No dependencies — start immediately when Ahmad assigns**

Strip all internal workspace files from the public frontend GitHub repo. The public repo must contain only actual source code — nothing internal.

**Why this is task 1:** The whitelist page and all upcoming Phase 0 work goes into the frontend repo. Before anything new is pushed, the repo must be professional and clean. Right now it contains internal Manager files that were pushed alongside code.

**What to remove (delete these files from the repo root and any subdirectory they ended up in):**
- `TASKS.md`
- `FIXES.md`
- `ARCHITECTURE.md`
- `BUILDER_ONBOARDING.md`
- `DECISIONS.md`
- `ROADMAP.md`
- `MANAGER_HANDOVER.md`
- `TOKENOMICS.md`
- `BUSINESS_MODEL.md`
- Any file with a `.md` extension that is not a `README.md`
- Any `docs/` folder containing internal planning files

**What to keep:**
- All `.tsx`, `.ts`, `.css`, `.json`, `.html`, `.js` source files
- `package.json`, `package-lock.json`, `tsconfig.json`, `vite.config.ts`, and all build config
- `vercel.json` (do NOT touch this file — sacred)
- `public/` folder (assets)
- `src/` folder (all source code)
- A clean `README.md` — if one does not exist, create a minimal one: project name, one-line description, live URL (`https://otiscore.vercel.app`), and tech stack only. No internal details.

**How to do it:**
- In your Replit workspace, delete those files locally, then push the deletion to GitHub
- Ahmad handles all GitHub merges — you push, he reviews and merges
- Do NOT use `git filter-branch` or rewrite history — a simple deletion commit is enough

**Evidence required to close:**
Builder lists the files deleted and confirms the push. Ahmad reviews the GitHub repo and confirms only source code is visible. Manager verifies via GitHub repo URL before closing.

---

### TASK 24 — Frontend: Docusaurus Docs Site Audit
**Builder: Frontend Builder | Priority: SECOND in Phase 0**
**Depends on: Task 23 confirmed merged by Ahmad**

Audit the live public developer docs at `https://otiscore.vercel.app/docs/` and clean out anything that reveals sensitive internal information about OTI's infrastructure.

**Why this matters:** The docs site is public. Before the whitelist launches and OTI attracts real traffic, the docs must contain only what a developer integrating the API needs — nothing that reveals how the admin system works, how keys are rotated, or internal architecture decisions.

**What is safe to keep (public developer-facing information):**
- Getting Started guide (API key signup instructions)
- API Reference (endpoints, request/response shapes, error codes)
- Supported Chains list — update count to **12 chains** (remove Fantom, Scroll, Sepolia, Holesky if they appear anywhere)
- Rate limits (whatever the current live limits are — do not invent numbers)
- Code examples in JS, Python, cURL
- Score explanation (what the score means, tier labels)
- Any `Try It Live` widget (keep — it's useful)

**What must be removed:**
- Any mention of Etherscan key rotation strategy or the number of keys OTI uses
- Any mention of admin routes, `/api/admin/`, or the `x-admin-secret` pattern
- Any mention of Railway, the backend host URL, or internal service architecture
- Any mention of the scoring algorithm internals, signal names with their exact weights, or anything that could help someone reverse-engineer trust scores
- Any bug fix history, version changelogs, or internal "known issues" content
- Any mention of "presale", "private sale", "invest", "ROI", "yield" — replace with whitelist vocabulary if relevant, or remove entirely

**D32 standard:** Read every sentence with fresh eyes. If it sounds like it was written by an AI language model (generic, hollow, over-structured), rewrite it in plain direct English. No "leverage", "robust", "seamlessly", "harness the power of."

**Chain count correction:** If the docs mention a chain count, it must say **12 chains**. The 12 live chains are: Ethereum, Bitcoin, Solana, BNB Chain, Polygon, Arbitrum, Optimism, Base, Avalanche, Tron, Sui (currently broken — BF41 — do not mention), zkSync, Linea. Do not list broken or planned chains as live.

**Evidence required to close:**
Builder lists every doc page they edited and what was removed. Manager spot-checks the live docs URL before closing.

---

### TASK 25 — Frontend: Whitepaper Rewrite
**Builder: Frontend Builder | Priority: THIRD in Phase 0**
**Depends on: Task 24 confirmed complete**

Rewrite the whitepaper at `/whitepaper` from scratch — merging the existing page content with the additions draft in `docs/whitepaper-additions-draft.md`. The result is a single, authoritative, public-facing technical document that reflects OTI as it exists today.

**Why a full rewrite:** The existing whitepaper was written before Phase 2 (WOR) shipped, before the whitelist pivot, and before the OTI Economics design was confirmed. The additions draft has valuable content but also contains stale chain information and presale vocabulary. Neither file can be used as-is.

**Source material to merge:**
1. The existing `/whitepaper` page content (already live — your starting point)
2. `docs/whitepaper-additions-draft.md` — read this and incorporate any section that is still accurate; discard anything stale

**Corrections required before using the additions draft:**
- Chain table: remove Fantom, Scroll, Sepolia, and Holesky — they are not live. The 12 live chains are: Ethereum, Bitcoin, Solana, BNB Chain, Polygon, Arbitrum, Optimism, Base, Avalanche, Tron, zkSync, Linea.
- Sui note: Sui is present in the code but currently offline (BF41). List it as "integration pending" or omit — do not list as live.
- Any reference to "presale", "private sale", "invest", "ROI", "yield" — replace entirely with whitelist vocabulary (see vocabulary table below)

**Vocabulary enforcement (mandatory throughout):**
| OLD — banned | NEW — required |
|---|---|
| Token Sale / Private Sale / Presale | Ecosystem Whitelist / Node Testing Program |
| Buy Tokens / Invest | Acquire Network Access Fuel / Claim Allocation |
| Staking Payouts / ROI / Yield | Node Collateral Lockup / Linear Network Vesting |
| Investors | Whitelisted Operators / Community Contributors |
| Trading / Listing | Public Utility Liquidity Pool Seeding |

**Required sections in the final whitepaper:**
1. **Executive Summary** — what OTI is, why it matters, the trust infrastructure problem
2. **The Problem** — wallet trust gap in DeFi, B2B compliance, and developer tools
3. **The Solution** — OTI scoring architecture, five-signal methodology
4. **How It Works** — wallet score flow, chain support, cache architecture
5. **Trust Signals** — describe the five signals and their function (do not reveal exact weights)
6. **Wallet Ownership Registry (WOR)** — what it is, why it matters, how self-report works
7. **OTI Economics** — use confirmed figures from TOKENOMICS.md: 35M supply, allocation table, dual bonding curve summary (DCS + ERP), revenue distribution, utility list. Do NOT make up numbers — copy from TOKENOMICS.md only.
8. **Ecosystem Whitelist Program** — Genesis Mode, DCS/ERP overview, how to participate (invite-only gate)
9. **Roadmap** — high-level phases only (Phase 0, 1, 2, 2B, 3+) — no internal task numbers
10. **Legal Disclaimer** — standard utility token disclaimer, geographic restrictions

**D32 standard:** Every sentence must read like a human wrote it. No AI tells. No "robust", "seamless", "harness", "leverage", "delve", "it's worth noting." Direct, factual, plain English.

**Evidence required to close:**
Builder confirms the page is live. Ahmad reads it and confirms no presale vocabulary and all economics figures match TOKENOMICS.md. Manager spot-checks the live URL.

---

### TASK 26 — Frontend: Privacy Policy + Terms & Conditions Pages
**Builder: Frontend Builder | Priority: FOURTH in Phase 0**
**Depends on: Task 25 confirmed complete**

Build two new pages at `/privacy` and `/terms`. The content is verbatim — provided by Ahmad. Do not rewrite, paraphrase, or "improve" any of it.

**Why now:** Both pages are required before the /whitelist page launches. The whitelist flow has a mandatory checkbox that links to both. They must be live before Task 27 starts.

**Terms & Conditions — verbatim content for `/terms`:**

```
OTI Ecosystem Whitelist — Terms & Conditions

1. PURPOSE OF THE WHITELIST
The OTI Whitelist Token Onboarding Program is strictly built to distribute network utility vouchers (Access Fuel) to future network testers, node operators, and B2B developers.

2. NO EXPECTATION OF PROFIT
Participants explicitly acknowledge that OTI tokens are utility tools used for wallet attestation fees and API queries. This program is not an investment, security, or financial contract. There is zero promise of future financial returns, passive yield, or profit.

3. PROTOCOL LOCKUP & VESTING
By accessing this software, the user agrees to the automatic 75% Node Collateral Lockup. Tokens will release linearly on a daily schedule to maintain network stability and protect circulating supply from systemic spamming.

4. GEOGRAPHIC RESTRICTIONS
This program is prohibited to residents, citizens, or IP addresses originating from high-risk or strictly regulated jurisdictions, including but not limited to the United States of America, China, and sanctioned nations.

5. ADMINISTRATIVE RIGHTS
The OTI core administration team reserves the absolute right to delete, freeze, or ban any invite code or wallet address found to be operating maliciously or misrepresenting the technical nature of the protocol.
```

**Privacy Policy — verbatim content for `/privacy`:**

```
OTI Ecosystem Whitelist — Privacy Policy

1. DATA COLLECTION PRINCIPLES
The OTI platform operates under true Web3 data minimization protocols. We do not collect, request, or store your real name, physical address, phone number, or government identity documentation.

2. TYPES OF DATA LOGGED
The system strictly logs decentralized interaction points:
(a) Public crypto wallet addresses used to claim allocations
(b) Validated single-use admin invite codes
(c) Basic on-chain transaction hashes

3. COOKIES & TRACKING
We do not deploy advertising tracker cookies or pixel trackers. Local device session storage may be temporarily used to verify your password token state for active sessions.

4. THIRD-PARTY DISCLOSURE
No collected Web3 identifier data is sold, rented, or passed to corporate advertisers. Data remains siloed in decentralized server instances solely used to manage active whitelist states.
```

**Page requirements:**
- Use the locked OTI color system — same navbar/footer as every other page
- Pages must be fully readable on mobile (375px)
- Add links to both pages in the site footer (`/privacy` and `/terms`) — add them now even though the whitelist isn't live yet
- Clean, simple layout: page title at top, numbered sections, readable body text in `#e8f4ff`
- No extra design flourishes — this is a legal document page, keep it plain and readable

**Evidence required to close:**
Builder confirms both pages are live at `/privacy` and `/terms`. Footer links verified. Ahmad confirms the content is verbatim.

---

### TASK 27 — Frontend: /whitelist Page
**Builder: Frontend Builder | Priority: FIFTH in Phase 0**
**Depends on: Task 26 confirmed complete AND Task 28 backend endpoints live**
**⚠️ Hard dependency: Do not start the invite-code verification flow until Ahmad confirms Task 28's `/api/verify-invite` endpoint is live on Railway. The entry gate and static structure can be built first.**

Build the `/whitelist` page — the Ecosystem Whitelist Node Program portal. This is the most professionally designed page on the entire site. Every section must look polished, premium, and trustworthy. No placeholder styling, no rough layouts. When in doubt, go more refined.

**Add `/whitelist` to the site navbar from day one. (D54) It is visible to all visitors; the locked gate controls access.**

---

**UX FLOW — Three Sequential States (D45):**

**State 1 — Locked Gate (all visitors before wallet connect):**
- Clean, minimal, dark-background panel
- Gate copy (exact): "Access is restricted to whitelisted users."
- Subtext: "If you received an invite code from the OTI team, connect your wallet to continue."
- "Connect Wallet" button — MetaMask / WalletConnect
- Nothing else visible. No prices, no token amounts, no charts.

**State 2 — Wallet Connected, Awaiting Code (after wallet connects, before code accepted):**
- Show truncated wallet address (confirmed connected)
- Invite code input field (format hint: OTI-XXXX-XXXX)
- Mandatory checkbox: "I have read and agree to the [Terms & Conditions](/terms) and [Privacy Policy](/privacy) of this program." — links open in new tab
- "Request Access" button — disabled until checkbox is ticked
- If code is invalid or already used: inline error "This code is invalid or has already been used."
- If referral `?ref=OTI-XXXX-XXXX` param is in the URL: silently pass it as `referred_by_code` in the request body — do not show this field to the user

**State 3 — Authenticated Portal (after valid code + T&C accepted):**
Full portal. Six sections in order:

---

**SECTION A — Live Protocol Counters (top of page, most prominent element on the page)**

- **"Current Committed Rate: $X.XXXXXX per OTI"** — live, polling `GET /api/whitelist/state` every 30 seconds (D48: "committed rate", not "contribution rate")
  - Progress bar: DCS fill toward $25,000 target
  - Sub-label: "Committed rate rises as allocation fills — earlier access means more OTI per dollar"
- **"Current Referral Bonus: X,XXX OTI"** — live, same polling interval
  - Sub-label: "Referral bonus decreases as DCS fills"
- DCS rate goes UP, referral bonus goes DOWN. Both moving simultaneously. Design must make this urgency visually obvious.

---

**SECTION B — Social Identity Verification (must complete before any reward can be claimed)**

Two required connections. Gate copy: "Connect both accounts to unlock all rewards."

1. **Telegram Phone Verification** — "Verify your Telegram account"
   - Button triggers Telegram bot auth flow → calls `POST /api/whitelist/connect-telegram`
   - On success: green badge "Telegram Verified ✓"
2. **X (Twitter) Account** — "Connect your X account"
   - OAuth connect → calls `POST /api/whitelist/connect-x`
   - On success: green badge "X Connected: @handle ✓"

Both must show green ✓ before reward cards in Section D are interactive. Once both are done this section collapses to a "Identity Verified ✓" row.

---

**SECTION C — Allocation Claim**

- Connected wallet address shown
- Allocation amount (from invite code API response)
- "Claim Allocation" button → calls claim endpoint
- Vesting summary below button:
  - "25% immediately accessible as Access Fuel"
  - "75% Node Collateral Lockup — releases linearly on a daily schedule"
  - Exact parameters from admin-configurable values, never hardcoded

---

**SECTION D — Ecosystem Rewards (all require Section B complete)**

Each reward card: task name, description, OTI amount (live from config), status badge (Locked / Available / Claimed ✓ / Claimed Today).

**One-time tasks (once per wallet, ever):**

1. **WOR Wallet Registration**
   - "Register your wallet in the Wallet Ownership Registry"
   - Reward: amount from config key `erp_base_wor_register_oti`
   - "Register Now" → opens /register in new tab → after confirmed, calls `POST /api/whitelist/task/one-time` with `task_type: 'wor_register'`
   - Status: Available → Claimed ✓

2. **WOR Compromise Report**
   - "Report a compromised wallet to the registry"
   - Reward: amount from config key `erp_base_wor_report_oti`
   - "Report Now" → opens /report in new tab → after submission, calls `POST /api/whitelist/task/one-time` with `task_type: 'wor_report'`
   - Status: Available → Claimed ✓

3. **Developer API Key**
   - "Create a Developer API key"
   - Reward: amount from config key `erp_base_dev_api_oti`
   - "Get API Key" → opens developer portal → after key created, calls `POST /api/whitelist/task/one-time` with `task_type: 'dev_api'`
   - Status: Available → Claimed ✓

4. **Follow on X (Twitter)**
   - "Follow OTI on X"
   - Reward: amount from config key `erp_base_follow_x_oti` (adjusted by ERP multiplier)
   - Auto-verified via connected X account
   - Status: Available → Claimed ✓

5. **Follow on Telegram**
   - "Join the OTI Telegram channel"
   - Reward: amount from config key `erp_base_follow_telegram_oti` (adjusted by ERP multiplier)
   - Auto-verified via Telegram connection
   - Status: Available → Claimed ✓

6. **Share a Post**
   - "Share about OTI and submit the link"
   - Reward: amount from config key `erp_base_share_oti` (adjusted by ERP multiplier)
   - User pastes URL → backend auto-verifies → calls `POST /api/whitelist/task/social` with `task_type: 'share'`
   - Status: Available → Claimed ✓

7. **Post + Tag OTI**
   - "Post about OTI, tag @OTI, and submit the link"
   - Reward: amount from config key `erp_base_post_tag_oti` (adjusted by ERP multiplier)
   - User pastes URL → backend auto-verifies → calls `POST /api/whitelist/task/social` with `task_type: 'post_tag'`
   - Status: Available → Claimed ✓

**Referral (unlimited — one reward per referred user who activates):**

8. **Refer a Node Operator**
   - "Share your invite link — earn X OTI per operator you refer"
   - Reward per referral: amount from config key `erp_base_referral_oti` (adjusted by ERP multiplier, shown live)
   - Referral link displayed: `otiscore.vercel.app/whitelist?ref=[their_invite_code]` — one-click copy button
   - Note shown: "Referred users still need their own invite code to enter the portal"
   - Reward auto-credited when a referred user successfully activates their own code

**Daily repeatable (once per calendar day UTC, no lifetime limit):**

9. **Daily Wallet Score**
   - "Score your wallet once per day to earn OTI — like daily mining"
   - Reward: amount from config key `erp_base_daily_score_oti`
   - "Score My Wallet" button → calls backend to run a score on the connected wallet → then calls `POST /api/whitelist/task/daily-score`
   - Show streak count: "X-day streak 🔥"
   - Show countdown: "Next reward available in: HH:MM:SS" (countdown to midnight UTC reset)
   - Status: Available → Claimed Today (resets next calendar day UTC)

---

**SECTION E — Whitepaper Engagement**

- Header: "Read the OTI Whitepaper — answer questions to earn OTI"
- Progress display: "X / 30 questions answered · X / 10 rewards earned"
- Link to /whitepaper — opens in new tab
- "Start Questions" button → calls `GET /api/whitelist/whitepaper/questions` → shows 3 random multiple-choice questions
- Each question: non-technical, about OTI's mission, use case, and vision (not about code or architecture)
- Answer all 3 correctly → calls `POST /api/whitelist/whitepaper/submit` → reward credited, progress updated
- One reward of `erp_base_whitepaper_round_oti` OTI per 3-question round answered correctly
- Maximum 10 reward rounds (30 questions total) per wallet — after that: "You've completed all whitepaper rounds ✓"
- If any answer is wrong: show which were incorrect, allow the user to retry the same round

---

**SECTION F — Milestones**

Three milestones as a horizontal progress tracker:
- Milestone 1: "Whitelist live — First node operators onboarded" (active when whitelist is live)
- Milestone 2: "$5,000 committed → Liquidity layer deployed" (DCS counter triggers)
- Milestone 3: "$15,000 committed → Secondary AMM pool live" (DCS counter triggers)

---

**Design requirements:**
- Dark backgrounds, mint/teal accents — consistent OTI color system
- Section A counters are the most visually dominant element on the page — design them that way
- Everything premium: spacing, typography, card edges, badge states — no rough finishes
- Fully responsive (375px mobile minimum) — Ahmad's users are primarily on mobile
- No AI-sounding copy anywhere — plain, direct English only (D32)
- Section B verification badges: satisfying visual completion states
- Reward cards: consistent layout, always show reward amount, always show status

**Evidence required to close:**
Builder confirms all three states render correctly. All nine reward cards in Section D display. Live counters in Section A load from API. Section B shows Telegram + X connection states with correct badge behavior. Daily score task shows streak count and countdown timer. Section E shows whitepaper question flow. Ahmad tests end-to-end: connect wallet → enter code → portal → Section B connects → reward cards available → daily score claimed → whitepaper questions answered. Manager verifies via screenshots before closing.

---

### TASK 28 — Backend: Whitelist System
**Builder: Backend Builder | Priority: Runs in parallel with Frontend Tasks 23–26**
**Can start as soon as Ahmad assigns — no dependency on frontend tasks**
**⚠️ All vesting/lockup/reward parameters must be admin-configurable. Nothing token-related is hardcoded. (D43, D53)**

Build the complete backend infrastructure for the Ecosystem Whitelist program: database tables, API endpoints, admin dashboard additions, and BNB Chain smart contracts.

---

**Part A — Database Schema (run via drizzle-kit push after Ahmad confirms on Railway):**

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

-- Social task submissions (auto-verified via submitted URL)
-- task_type: 'referral', 'post_tag', 'share', 'follow_x', 'follow_telegram'
-- status: 'auto_verified', 'rejected'  (no manual review — D49)
CREATE TABLE whitelist_social_tasks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    wallet_address VARCHAR(42) NOT NULL,
    task_type VARCHAR(30) NOT NULL,
    proof_url TEXT DEFAULT NULL,
    oti_reward NUMERIC(20, 6) NOT NULL,
    status VARCHAR(20) DEFAULT 'auto_verified',
    submitted_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- One-time product engagement tasks (wor_register, wor_report, dev_api)
CREATE TABLE whitelist_task_completions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    wallet_address VARCHAR(42) NOT NULL,
    task_type VARCHAR(30) NOT NULL,  -- 'wor_register', 'wor_report', 'dev_api'
    oti_reward NUMERIC(20, 6) NOT NULL,
    completed_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    UNIQUE (wallet_address, task_type)  -- one-time per wallet per task
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

-- Whitepaper questions pool (100 non-technical questions, admin-managed)
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
    rewards_claimed INT DEFAULT 0,  -- each reward = one 3-question round
    last_round_at TIMESTAMP WITH TIME ZONE DEFAULT NULL
);

-- Session fingerprinting for multi-account detection (D58)
CREATE TABLE whitelist_fingerprints (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    wallet_address VARCHAR(42) NOT NULL,
    ip_address VARCHAR(45) NOT NULL,           -- IPv4 or IPv6
    fingerprint_hash VARCHAR(64) NOT NULL,     -- SHA-256 of UA + Accept-Language + screen data
    user_agent TEXT DEFAULT NULL,              -- raw UA stored for admin review
    event_type VARCHAR(30) NOT NULL,           -- 'register', 'connect_telegram', 'connect_x', 'claim_reward'
    flagged BOOLEAN DEFAULT false,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Flagged account log (populated automatically by fingerprint checks)
CREATE TABLE whitelist_flags (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    wallet_address VARCHAR(42) NOT NULL,
    flag_reason VARCHAR(100) NOT NULL,         -- e.g. 'ip_collision:3_wallets', 'fingerprint_collision'
    related_wallets TEXT[] DEFAULT '{}',       -- other wallets sharing the same IP/fingerprint
    status VARCHAR(20) DEFAULT 'open',         -- 'open', 'reviewed', 'cleared', 'banned'
    flagged_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
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
  ('dcs_total_oti',               '7000000',  NOW()),
  ('dcs_target_usd',              '25000',    NOW()),
  ('dcs_start_rate',              '0.001190', NOW()),
  ('dcs_end_rate',                '0.005952', NOW()),
  ('erp_total_oti',               '1750000',  NOW()),
  ('erp_base_referral_oti',       '3000',     NOW()),
  ('erp_base_post_tag_oti',       '1000',     NOW()),
  ('erp_base_share_oti',          '500',      NOW()),
  ('erp_base_follow_x_oti',       '500',      NOW()),
  ('erp_base_follow_telegram_oti','500',       NOW()),
  ('erp_base_daily_score_oti',    '100',      NOW()),
  ('erp_base_wor_register_oti',   '500',      NOW()),
  ('erp_base_wor_report_oti',     '300',      NOW()),
  ('erp_base_dev_api_oti',        '500',      NOW()),
  ('erp_base_whitepaper_round_oti','200',     NOW()),
  ('vesting_lockup_pct',          '75',       NOW()),
  ('max_whitelist_slots',         '0',        NOW()),
  ('flag_ip_threshold',           '3',        NOW()),
  ('flag_fingerprint_threshold',  '2',        NOW());
  -- max_whitelist_slots: 0 = unlimited. Ahmad sets a cap via admin dashboard when ready. (D53)
  -- flag_ip_threshold: flag all wallets sharing the same IP if count >= this value
  -- flag_fingerprint_threshold: flag all wallets sharing the same fingerprint hash if count >= this value
```

Seed `protocol_state`:
```sql
INSERT INTO protocol_state (id) VALUES (1);
```

---

**Part B — API Endpoints (new file: `src/routes/whitelist.ts`):**

**`POST /api/verify-invite`**
- Body: `{ invite_code, wallet_address, terms_accepted: true, referred_by_code?, fingerprint_data? }`
  - `fingerprint_data`: optional object from frontend — `{ screen_resolution, timezone_offset, canvas_hash }` — used to build the fingerprint hash server-side
- `terms_accepted` must be `true` — 400 if not
- Look up `invite_code` in `whitelist_invites` — 404 if not found, 400 if used or status ≠ 'active'
- Check `whitelist_participants` for `wallet_address` — 409 if already registered
- Check `max_whitelist_slots` from `whitelist_config`: if value > 0 and `total_slots_claimed` >= that value → 410 (full). If value = 0: no cap, proceed always.
- **Fingerprint capture and collision check (D58):**
  - Build `fingerprint_hash` = SHA-256 of `(ip_address + user_agent + accept_language + fingerprint_data)`
  - Insert into `whitelist_fingerprints` (wallet_address, ip_address, fingerprint_hash, user_agent, event_type='register')
  - Count distinct wallets in `whitelist_fingerprints` with same `ip_address` → if >= `flag_ip_threshold` config value: insert flag row into `whitelist_flags` for all involved wallets, reason = `'ip_collision'`
  - Count distinct wallets with same `fingerprint_hash` → if >= `flag_fingerprint_threshold`: insert flag rows for all involved wallets, reason = `'fingerprint_collision'`
  - Flagging does NOT block registration — it only surfaces to admin. Never tell the user they were flagged.
- On success:
  - Mark `whitelist_invites.is_used = true`, set `used_by_wallet`
  - If `referred_by_code` in body (from `?ref=` URL param): set `whitelist_invites.referred_by_code`
  - Insert into `whitelist_participants`
  - Increment `protocol_state.total_slots_claimed`
  - If invite has `referred_by_code` set: auto-credit referral reward to referring wallet — insert into `whitelist_social_tasks` (task_type='referral', status='auto_verified'), add reward to referring wallet's `oti_allocated`
  - Return `{ success: true, wallet_address, oti_allocated, current_dcs_committed_rate, current_erp_referral_bonus }`

**`GET /api/whitelist/state`**
- Public endpoint — no auth
- Returns live `protocol_state` + computed values from `whitelist_config`:
  - DCS committed rate: `dcs_start_rate + (dcs_end_rate - dcs_start_rate) × ((dcs_total_oti - dcs_oti_remaining) / dcs_total_oti)`
  - ERP referral bonus: `erp_base_referral_oti × (dcs_oti_remaining / dcs_total_oti)`
  - All current `erp_base_*` values (for reward card display)
- Frontend polls every 30 seconds

**`GET /api/whitelist/participant/:wallet_address`**
- Returns full participant record including telegram_verified, x_handle, oti_allocated, oti_claimed

**`GET /api/whitelist/tasks/:wallet_address`**
- Returns completion status for every task type for this wallet:
  - One-time tasks: completed or not (from `whitelist_task_completions`)
  - Social tasks: completed or not (from `whitelist_social_tasks`)
  - Daily score: whether scored today + current streak count (from `whitelist_daily_scores`)
  - Whitepaper progress: questions_answered, rewards_claimed (from `whitelist_whitepaper_progress`)

**`POST /api/whitelist/connect-telegram`**
- Body: Telegram login widget auth data (hash, id, first_name, username, auth_date)
- Verify hash using Telegram bot token (HMAC-SHA256 per Telegram docs)
- Check `auth_date` not older than 15 minutes — 401 if expired
- Update `whitelist_participants`: set `telegram_verified = true`, `telegram_user_id = id`
- Reject if telegram_user_id already used by a different wallet — 409
- Return `{ success: true }`

**`POST /api/whitelist/connect-x`**
- Body: X OAuth callback data
- Complete OAuth 2.0 flow, retrieve X handle
- Update `whitelist_participants`: set `x_handle`
- Reject if x_handle already used by a different wallet — 409
- Return `{ success: true, x_handle }`

**Reward gate helper (internal — used by all reward endpoints below):**
Before crediting any reward, check in this order:
1. `whitelist_participants.status = 'active'` — 403 `{ error: 'banned' }` if not
2. `whitelist_participants.telegram_verified = true` — 403 `{ error: 'telegram_required' }` if not
3. `whitelist_participants.x_handle IS NOT NULL` — 403 `{ error: 'x_required' }` if not
4. Check `whitelist_flags` for any open flag on this wallet — if found, still allow the claim but log the fingerprint event again (do NOT block silently — flagging is for Ahmad's review, not for auto-blocking)
- Also on every reward claim: insert a row into `whitelist_fingerprints` (event_type='claim_reward') and re-run the IP/fingerprint collision check. If new collisions are found, insert new flag rows. Ahmad reviews these in the admin panel.

**`POST /api/whitelist/task/daily-score`**
- Runs reward gate check
- Check `whitelist_daily_scores` for `(wallet_address, CURRENT_DATE UTC)` — 409 if already claimed today
- Trigger scoring API call for `wallet_address` (call internal score endpoint)
- Insert row into `whitelist_daily_scores` with `oti_reward` from config key `erp_base_daily_score_oti`
- Add reward to `whitelist_participants.oti_allocated`
- Return `{ success: true, oti_reward, streak_days }` where `streak_days` = consecutive daily entries ending today

**`POST /api/whitelist/task/one-time`**
- Body: `{ task_type }` — one of: `'wor_register'`, `'wor_report'`, `'dev_api'`
- Runs reward gate check
- Check `whitelist_task_completions` for `(wallet_address, task_type)` — 409 if already completed
- Verify the underlying action occurred:
  - `wor_register`: check `wallet_ownership` table for this wallet (existing WOR table from Phase 2)
  - `wor_report`: check `compromised_wallets` table for a report by this wallet (existing table from Phase 2)
  - `dev_api`: check `subscriptions` table for an active API key owned by this wallet
- If action not verified: 422 with clear message explaining what the user needs to do first
- Insert into `whitelist_task_completions`, add reward to `oti_allocated`
- Return `{ success: true, oti_reward }`

**`POST /api/whitelist/task/social`**
- Body: `{ task_type, proof_url }` — task_type one of: `'post_tag'`, `'share'`
- Runs reward gate check
- Check `whitelist_social_tasks` for existing completed entry for this wallet + task_type — 409 if already done
- Auto-verify `proof_url`: check URL is reachable (HTTP HEAD request, expect 200) — 422 if unreachable
- Apply ERP multiplier: `reward = base_reward × (dcs_oti_remaining / dcs_total_oti)`
- Insert into `whitelist_social_tasks` (status='auto_verified'), add reward to `oti_allocated`
- Return `{ success: true, oti_reward }`

**`GET /api/whitelist/whitepaper/questions`**
- Requires wallet_address param or auth header (wallet must be a registered participant)
- Runs reward gate check
- Check `whitelist_whitepaper_progress`: if `questions_answered >= 30` → 410 (all rounds complete)
- Select 3 random active questions from `whitelist_whitepaper_questions` that this wallet has not yet answered
- Track which question IDs were served (store temporarily — session or signed token)
- Return `{ questions: [{ id, question_text, option_a, option_b, option_c, option_d }] }` — do NOT include correct_option

**`POST /api/whitelist/whitepaper/submit`**
- Body: `{ answers: [{ question_id, selected_option }] }` (3 items)
- Runs reward gate check
- Verify answers match the served questions for this wallet (anti-cheat)
- Check all 3 answers against `correct_option` in DB
- If any wrong: return `{ success: false, incorrect: [question_ids] }` — allow retry of same round
- If all 3 correct:
  - Update `whitelist_whitepaper_progress`: increment `questions_answered` by 3, `rewards_claimed` by 1
  - Add `erp_base_whitepaper_round_oti` to `whitelist_participants.oti_allocated`
  - Return `{ success: true, oti_reward, questions_answered, rewards_claimed }`

---

**Part C — Admin Dashboard Additions (new "Whitelist" tab in existing admin panel):**

Three sub-views (Social Task Review Queue is removed — tasks are auto-verified, D49):

1. **Code Management Panel**
   - `POST /api/admin/whitelist/generate-codes` — generates N unique `OTI-XXXX-XXXX` codes (uppercase alphanumeric, 4+4), inserts into `whitelist_invites` with status 'active'. No maximum N limit (D53).
   - Table: all codes, status, used_by_wallet (truncated), amount_contributed_usd, created_at
   - Filter by status (active / used / banned / expired)
   - Per-row "Ban" button: `PATCH /api/admin/whitelist/codes/:id` → status = 'banned'
   - Per-row "Expire" button: → status = 'expired'

2. **Whitepaper Questions Manager**
   - Table: all questions with is_active toggle
   - "Add Question" form: question_text, option_a–d, correct_option (a/b/c/d), is_active
   - `POST /api/admin/whitelist/questions` — add question
   - `PATCH /api/admin/whitelist/questions/:id` — edit or toggle active
   - `DELETE /api/admin/whitelist/questions/:id` — remove question
   - Pre-seed 100 non-technical questions at deploy time (see question list below)

3. **Flagged Accounts Panel**
   - Table: all rows from `whitelist_flags` — wallet_address, flag_reason, related_wallets, flagged_at, status
   - Filter by status (open / reviewed / cleared / banned)
   - Per-row actions:
     - "Ban Wallet" → sets `whitelist_participants.status = 'banned'` for that wallet + all related_wallets listed on the flag, sets flag status = 'banned'
     - "Clear Flag" → sets flag status = 'cleared' (false positive — no action taken)
     - "Mark Reviewed" → sets flag status = 'reviewed' (Ahmad looked at it, no action yet)
   - Show IP and fingerprint hash on flag detail expand (for Ahmad to assess manually)
   - `GET /api/admin/whitelist/flags` — returns all flags, filterable by status
   - `PATCH /api/admin/whitelist/flags/:id` — updates flag status and optionally bans related wallets

4. **Protocol Config Override**
   - Input field for every `whitelist_config` key, prefilled with current DB value
   - "Save" button: `PUT /api/admin/whitelist/config` — updates any set of config keys
   - This is how Ahmad adjusts all parameters (reward amounts, DCS rates, vesting %, slot cap, flagging thresholds) without a code deploy
   - Include `max_whitelist_slots` — label: "Max whitelist slots (0 = unlimited)"
   - Include `flag_ip_threshold` and `flag_fingerprint_threshold` — label clearly what each controls

**100 Whitepaper Questions (seed data — non-technical, about OTI's mission and use case):**

Builder writes 100 questions in this format and seeds them via a migration script or seed file. Questions must be readable by anyone — no blockchain jargon, no code references. Topics: what OTI does, why trust scoring matters for crypto, what the WOR registry is for, what node operators do, why OTI's scoring is on-chain, what wallets are, how OTI helps exchanges, why ecosystem participation matters, what the whitepaper covers. Example questions:

```
Q: What is the main purpose of OTI?
a) To create a new cryptocurrency exchange
b) To provide a trust score for cryptocurrency wallets  ✓
c) To replace existing blockchain networks
d) To offer crypto lending services

Q: What does WOR stand for in the OTI ecosystem?
a) Wallet Ownership Registry  ✓
b) Web3 Operations Relay
c) Wallet Operation Record
d) Web Ownership Rights

Q: Why do exchanges use wallet trust scores?
a) To slow down transaction speeds
b) To identify and reduce risk from suspicious wallets  ✓
c) To charge higher fees
d) To limit the number of wallets on their platform
```

Builder writes all 100 in this format and inserts them. They do not need to be exhaustive — they just need to verify the user read the whitepaper and understands the product at a high level.

---

**Part D — Smart Contracts (BNB Testnet → Mainnet — Builder handles everything, Ahmad not involved until workspace deletion):**

Ahmad is not involved at any point during this part. Builder generates all keys, deploys to testnet, tests fully, deploys to mainnet — all without Ahmad. At close, deliver the complete handover package to the Manager. The Manager relays it to Ahmad. Ahmad's only action is to receive the package and delete the Builder workspace. See D42 in DECISIONS.md.

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
- Testnet: https://testnet.bnbchain.org/faucet-smart → paste DEPLOYER_ADDRESS (free)
- Mainnet: Builder funds from their own wallet (~0.01–0.05 BNB). Ahmad reimburses at handover.
- Confirm balances on both networks before proceeding.

**Step 3 — Deploy mock OTI BEP-20 token (`MockOTI.sol`) — testnet only:**
- Standard ERC-20, `constructor(uint256 initialSupply)` mints to deployer
- Use `35000000 * 10**18` as initial supply
- Deploy to BNB testnet (chainId 97) only
- Save deployed address as `MOCK_OTI_ADDRESS`

**Step 4 — Deploy `OTIWhitelistVesting.sol` to testnet first, then mainnet:**
- Testnet (chainId 97): owner = DEPLOYER_ADDRESS, token = MOCK_OTI_ADDRESS
- Mainnet (chainId 56): owner = DEPLOYER_ADDRESS, token = placeholder address (D57 — no real OTI BEP-20 exists yet; use address(0) or a clearly documented placeholder; contract must include `setTokenAddress(address)` owner-only function so Ahmad can update it after handover before the whitelist goes live)
- Contract functions:
  - `vest(address participant, uint256 total_oti_amount)` — owner-only. 25% immediate, 75% linear daily over `vesting_duration_days`
  - `claimVested(address participant)` — callable by participant. Transfers unlocked OTI.
  - `setVestingDuration(uint256 days_)` — owner-only. Future vests only, not retroactive.
  - `setTokenAddress(address token_)` — owner-only. Allows updating the OTI token address after deployment (required for mainnet — see D57).
  - `getVestingStatus(address participant)` — view. Returns total_allocated, total_claimed, currently_claimable, vesting_start, vesting_end
  - `banParticipant(address participant)` — owner-only. Freezes remaining locked tokens.
- After testnet deploy: call `MockOTI.approve(vestingContractAddress, large_amount)` so vesting contract can move tokens
- Verify all contracts on BscScan (testnet + mainnet) after each deploy

**Step 5 — End-to-end test on testnet:**
- `vest(testWallet, 1000 * 10^18)` → confirm 250 OTI immediately claimable (25%)
- Advance testnet time or wait → confirm linear daily unlock works
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
- BscScan testnet + mainnet links for all contracts
- Note: "Mainnet vesting contract deployed with placeholder token address. Call setTokenAddress(real_OTI_address) after OTI BEP-20 is deployed before going live."
- Gas cost incurred (BNB amount) — for Ahmad to reimburse

Ahmad saves the keys and addresses, then deletes this workspace.

---

**Evidence required to close:**
- All DB tables created on Railway (Ahmad runs drizzle-kit push after Builder deploys)
- `/api/verify-invite`: valid code → success JSON; used code → 400; missing → 404 (paste raw responses)
- `/api/whitelist/state`: returns computed DCS committed rate and all ERP bonus values
- `/api/whitelist/connect-telegram`: Telegram auth verified, participant updated
- `/api/whitelist/connect-x`: X OAuth completes, x_handle stored
- `/api/whitelist/task/daily-score`: first call succeeds, second call same day returns 409
- `/api/whitelist/task/one-time`: returns 422 if underlying action not done, succeeds after action, returns 409 on duplicate
- `/api/whitelist/task/social`: auto-verifies URL, credits reward, 409 on duplicate
- `/api/whitelist/whitepaper/questions`: returns 3 questions without correct_option
- `/api/whitelist/whitepaper/submit`: correct answers → reward credited; wrong answers → returns which failed
- Admin Whitelist tab: code management table shows rows, ban/expire works, questions CRUD works, config save updates DB
- 100 whitepaper questions seeded and visible in admin
- BNB testnet contracts deployed and verified — paste BscScan testnet links
- BNB mainnet vesting contract deployed and verified — paste BscScan mainnet link
- vest() and claimVested() test on testnet confirmed with raw output
- setTokenAddress() function present and callable by owner on mainnet contract
- Full handover package delivered to Manager
- Manager verifies all evidence and relays package to Ahmad before closing

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
