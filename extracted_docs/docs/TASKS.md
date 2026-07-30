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

Build the `/whitelist` page — the Ecosystem Whitelist Node Program portal.

**Page structure (two states):**

**State 1 — Unauthenticated Gate (default for all visitors):**
- A professional, locked security panel. No wallet connect buttons. No token charts. No contract addresses. No pricing visible.
- Header text: "OTI Infrastructure Hub — Private Whitelist Node Platform"
- Subtext: "Access is restricted to whitelisted node operators and infrastructure partners. If you have received an invite code from the OTI team, enter it below."
- Invite code input field (format hint: OTI-XXXX-XXXX)
- Mandatory checkbox: "I have read and agree to the [Terms & Conditions](/terms) and [Privacy Policy](/privacy) of this program." — links open in new tab
- "Request Access" button — calls `POST /api/verify-invite` (Task 28 endpoint). Disabled until checkbox is checked.
- If code is invalid or expired: show inline error "This code is invalid or has already been used."
- If code is valid: transition to State 2

**State 2 — Authenticated Portal (after valid code + terms accepted):**

- **Live DCS counter (top of page, prominent):**
  - "Current Contribution Rate: $X.XXXXXX per OTI" — live, pulling from `protocol_state`
  - Progress bar toward $25,000 total DCS target
  - Sub-label: "Rate increases as allocation fills — early contributors receive more OTI per dollar"

- **Live ERP counter (directly below DCS, prominent):**
  - "Current Referral Bonus: X,XXX OTI" — live, pulling from `protocol_state`
  - Sub-label: "Referral bonus decreases as DCS fills — claim your allocation early for maximum referral value"
  - Both counters update simultaneously: DCS rate goes UP, ERP bonus goes DOWN — this is intentional and must be visually clear

- **Allocation claim section:**
  - "Connect your wallet to claim your Node Access Allocation"
  - Wallet connect button (MetaMask / WalletConnect)
  - Once connected: show wallet address (truncated), show allocation amount based on invite code
  - "Claim Allocation" button

- **Ecosystem Rewards section (below claim):**
  - "Earn additional OTI through Ecosystem Participation"
  - Three reward cards:
    - Referral: "Share your invite link — earn X,XXX OTI per referred operator" (X = current ERP referral rate, live)
    - Social Post: "Post about OTI and tag @OTI — earn 1,000 OTI (pending admin verification)"
    - Share + Follow: "Share a post and follow OTI — earn 500 OTI each (pending admin verification)"
  - All social rewards: "Reward is credited after manual admin review — typically within 48 hours"

- **Vesting summary (below rewards):**
  - "75% Node Collateral Lockup — releases linearly on a daily schedule"
  - "25% immediately accessible as Access Fuel"
  - Exact lockup/vesting parameters shown here — pulled from admin-configurable values, never hardcoded

**Design requirements:**
- Use the locked OTI color system (dark backgrounds, mint accents)
- Both live counters must be visually prominent — they are the key urgency mechanism
- Fully responsive (375px mobile) — Ahmad's users are primarily on mobile
- No AI-sounding copy anywhere — plain, direct English only (D32)
- Add `/whitelist` to the navbar only after Ahmad explicitly says to — for now it is URL-only, like `/admin`

**Evidence required to close:**
Builder confirms both states render correctly. Ahmad tests a real invite code end-to-end: enters code → sees authenticated portal → both live counters loading → wallet connect works → allocation visible. Manager verifies via screenshot before closing.

---

### TASK 28 — Backend: Whitelist System
**Builder: Backend Builder | Priority: Runs in parallel with Frontend Tasks 23–26**
**Can start as soon as Ahmad assigns — no dependency on frontend tasks**
**⚠️ All vesting/lockup/reward parameters must be admin-configurable. Nothing token-related is hardcoded.**

Build the complete backend infrastructure for the Ecosystem Whitelist program: database tables, API endpoints, admin dashboard additions, and BNB Chain smart contracts.

**Part A — Database Schema (run via drizzle-kit push after Ahmad confirms on Railway):**

```sql
-- Invite code management
CREATE TABLE whitelist_invites (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    invite_code VARCHAR(50) UNIQUE NOT NULL,
    is_used BOOLEAN DEFAULT false,
    used_by_wallet VARCHAR(42) DEFAULT NULL,
    referred_by_code VARCHAR(50) DEFAULT NULL,  -- the invite code that referred this user
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
    proof_url TEXT DEFAULT NULL,  -- URL submitted by user as evidence
    oti_reward NUMERIC(20, 6) NOT NULL,  -- amount at time of submission
    status VARCHAR(20) DEFAULT 'pending',  -- 'pending', 'approved', 'rejected'
    submitted_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    reviewed_at TIMESTAMP WITH TIME ZONE DEFAULT NULL
);

-- Protocol global state (single-row table — serves frontend live counters)
CREATE TABLE protocol_state (
    id INT PRIMARY KEY DEFAULT 1,
    dcs_total_usd_committed NUMERIC(10, 2) DEFAULT 0.00,  -- DCS: total USD contributed so far
    dcs_oti_remaining NUMERIC(20, 6) DEFAULT 7000000.00,  -- DCS: OTI remaining in sub-pool
    total_slots_claimed INT DEFAULT 0,  -- total whitelist slots used
    CONSTRAINT single_row CHECK (id = 1)
);

-- Admin-configurable whitelist parameters (key-value store — nothing token-related is hardcoded)
CREATE TABLE whitelist_config (
    key VARCHAR(100) PRIMARY KEY,
    value TEXT NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
-- Seed with default values:
-- INSERT INTO whitelist_config VALUES ('dcs_total_oti', '7000000', NOW());
-- INSERT INTO whitelist_config VALUES ('dcs_target_usd', '25000', NOW());
-- INSERT INTO whitelist_config VALUES ('dcs_start_rate', '0.001190', NOW());
-- INSERT INTO whitelist_config VALUES ('dcs_end_rate', '0.005952', NOW());
-- INSERT INTO whitelist_config VALUES ('erp_total_oti', '1750000', NOW());
-- INSERT INTO whitelist_config VALUES ('erp_base_referral_oti', '3000', NOW());
-- INSERT INTO whitelist_config VALUES ('erp_base_post_tag_oti', '1000', NOW());
-- INSERT INTO whitelist_config VALUES ('erp_base_share_oti', '500', NOW());
-- INSERT INTO whitelist_config VALUES ('erp_base_follow_oti', '500', NOW());
-- INSERT INTO whitelist_config VALUES ('vesting_lockup_pct', '75', NOW());
-- INSERT INTO whitelist_config VALUES ('max_whitelist_slots', '10000', NOW());
```

**Part B — API Endpoints (new file: `src/routes/whitelist.ts`):**

`POST /api/verify-invite`
- Body: `{ invite_code, wallet_address, terms_accepted: true }`
- Validation: `terms_accepted` must be `true` — reject with 400 if not
- Look up `invite_code` in `whitelist_invites` — reject with 404 if not found, 400 if already used or status ≠ 'active'
- Check `whitelist_participants` for `wallet_address` — reject with 409 if already registered
- Check `total_slots_claimed` in `protocol_state` against `max_whitelist_slots` in `whitelist_config` — reject with 410 if full
- On success:
  - Mark `whitelist_invites.is_used = true`, set `used_by_wallet`
  - Insert row into `whitelist_participants`
  - Increment `protocol_state.total_slots_claimed`
  - If `referred_by_code` is set on the invite, queue a referral reward for the referring wallet (insert into `whitelist_social_tasks` with `task_type = 'referral'`, status = 'approved' — referrals are auto-approved)
  - Return `{ success: true, wallet_address, oti_allocated, current_dcs_rate, current_erp_referral_bonus }`

`GET /api/whitelist/state`
- Public endpoint — no auth
- Returns live values from `protocol_state` + current DCS rate + current ERP bonus values computed from `whitelist_config`
- DCS current rate formula: `start_rate + (end_rate - start_rate) × ((dcs_total_oti - dcs_oti_remaining) / dcs_total_oti)`
- ERP current referral bonus formula: `base_referral_oti × (dcs_oti_remaining / dcs_total_oti)`
- Frontend polls this every 30 seconds for live counter updates

`GET /api/whitelist/participant/:wallet_address`
- Protected by existing `apiKeyAuth` middleware (or open — Ahmad decides)
- Returns participant record if exists, 404 if not

**Part C — Admin Dashboard Additions (new tab: "Whitelist" inside the existing admin panel):**

The admin panel already exists (Task 9). Add a new "Whitelist" tab with four sub-views:

1. **Batch Code Generator:**
   - Input: number of codes to generate (default 10, max 500)
   - Button: "Generate Codes"
   - Backend endpoint: `POST /api/admin/whitelist/generate-codes` — generates N unique `OTI-XXXX-XXXX` format codes (uppercase alphanumeric, 4+4 chars), inserts into `whitelist_invites` with status 'active'
   - After generation: display the generated codes in a copyable list

2. **Code Management Panel:**
   - Table: all codes, status, used_by_wallet (truncated), amount_contributed_usd, created_at
   - Filter by status (active / used / banned / expired)
   - Per-row "Ban" button: `PATCH /api/admin/whitelist/codes/:id` — sets status to 'banned'

3. **Social Task Review Queue:**
   - Table: all pending `whitelist_social_tasks` rows — wallet, task_type, proof_url (clickable), oti_reward, submitted_at
   - Per-row "Approve" and "Reject" buttons — `PATCH /api/admin/whitelist/social-tasks/:id`
   - On approve: mark status = 'approved', update `whitelist_participants.oti_allocated` for that wallet
   - On reject: mark status = 'rejected', no allocation change

4. **Protocol State Override:**
   - Input fields for each `whitelist_config` key (prefilled with current values)
   - "Save" button: `PUT /api/admin/whitelist/config` — updates one or more config keys
   - This is how Ahmad adjusts all parameters — vesting %, reward amounts, DCS rates, slot count — without a code change

**Part D — Smart Contracts (BNB Chain — testnet first, mainnet only after Ahmad confirms):**

The whitelist smart contract manages OTI token vesting and lockup for whitelisted participants. This is a separate concern from the XMTP attestation contract (Tasks 20/21).

Scope for Phase 0: **Testnet only for now.** Do not deploy to mainnet until Ahmad explicitly says go.

- Language: Solidity 0.8.x
- Chain: BNB Chain testnet (chainId 97) first
- Contract: `OTIWhitelistVesting.sol`
  - Constructor args: `owner_address`, `oti_token_address` (BEP-20 OTI token — Ahmad provides this address when OTI token is deployed; use a mock address on testnet)
  - `vest(address participant, uint256 total_oti_amount)` — owner-only. Records vesting schedule: 25% immediate, 75% linear daily release over `vesting_duration_days` (admin-configurable, read from contract storage)
  - `claimVested(address participant)` — callable by participant. Transfers any unlocked OTI to their wallet.
  - `setVestingDuration(uint256 days_)` — owner-only. Changes the vesting duration for future vests (not retroactive).
  - `getVestingStatus(address participant)` — view function. Returns `{ total_allocated, total_claimed, currently_claimable, vesting_start, vesting_end }`
  - `banParticipant(address participant)` — owner-only. Freezes remaining locked tokens.

**Evidence required to close:**
- All DB tables created on Railway (Ahmad runs drizzle-kit push after Builder deploys)
- `/api/verify-invite` tested with a real invite code: valid code → success; used code → 400; no code → 404
- `/api/whitelist/state` returns correct live computed DCS rate and ERP bonus
- Admin Whitelist tab renders: code generator works, code table shows rows, config save works
- Smart contract deployed on BNB testnet — Builder pastes testnet contract address
- Ahmad confirms testnet `vest()` and `claimVested()` work end-to-end
- Manager verifies all evidence before closing

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
