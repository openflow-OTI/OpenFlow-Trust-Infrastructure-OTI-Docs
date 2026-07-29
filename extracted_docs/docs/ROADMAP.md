# OTI — Product Roadmap
> Last updated: July 29, 2026 (session 21 — Full whitelist pivot applied. All "private sale" / "presale" language replaced with Ecosystem Whitelist framing throughout. Supply updated to 35M. Token distribution model updated to match new OTI Economics. D34 vocabulary enforcement applies to this file and everything derived from it.)
> Source document: OTI Full Distribution & Technical Development Strategy (Founder's Playbook, July 2026) + Ahmad direction sessions 20–21.

---

## The Core Thesis

OTI is not a wallet checker website. It is on-chain behavioral intelligence as a service — the infrastructure layer for trust in Web3. The website is the demo. The API is the product.

**Who buys this:**
- Exchanges and gateways — screen withdrawal destinations, detect compromised wallets (enterprise sale, highest value)
- DeFi protocols — risk-adjust lending and collateral based on wallet trust (API integration)
- NFT marketplaces — seller trust badges, filter bad actors (API integration)
- Payment processors — gate crypto payments behind minimum trust score (API integration)

**Three goals:**
- 90 days after distribution launches: 1,000 active API users via bots and widget
- 6 months: 3 paying business API integrations
- 12 months: first enterprise exchange contract

---

## Current State (July 29, 2026)

**What is live in production:**
- Backend scoring API on Railway, 12 chains working (7 EVM + 5 non-EVM)
- Frontend at otiscore.vercel.app: homepage, /score, /whitepaper, /services, /register, /report, /admin
- Developer docs at otiscore.vercel.app/docs/
- WOR (Wallet Ownership Registry) fully live
- Admin panel secured
- Score sharing PNG cards live
- Anonymous rate limits removed (Ahmad via admin panel, D33)

**What is not yet live:**
- /whitelist page and Ecosystem Whitelist Node Program infrastructure
- Self-serve developer API key signup
- Privacy Policy and Terms & Conditions (whitelist versions)
- Whitepaper rewrite (current version does not reflect 35M supply, new economics, or whitelist framing)
- GitHub frontend repo cleanup (internal workspace files present — needs removal before whitelist launches)
- Docusaurus docs audit (sensitive internal architecture details may be exposed)
- BAS attestation layer and widget
- Bots, extension, and other distribution channels

---

## PHASE 0 — ECOSYSTEM WHITELIST INFRASTRUCTURE
**Owner: Both Builders | Status: NEXT — task prompts being written. Awaiting Ahmad's confirmation of bonding curve target raise figure before final task prompt for whitelist smart contracts.**
**Strategic context (D34):** Ahmad's decision July 29, 2026. The Ecosystem Whitelist Node Program is the primary ecosystem funding mechanism and the launch event for the OTI token. It runs before the XMTP campaign. All "private sale / presale / token sale / ICO / invest / ROI / yield" language is permanently banned. Full vocabulary map in DECISIONS.md D34.

---

### 0A — GitHub Frontend Repo Cleanup (Ahmad + Frontend Builder)
The public frontend GitHub repo currently contains internal workspace documentation files (TASKS.md, FIXES.md, ARCHITECTURE.md, BUILDER_ONBOARDING.md, etc.) that were pushed alongside source code. The repo must look professional — only actual built source code visible.

- **Frontend Builder:** Identify and remove all internal documentation files from the repo. Ensure no internal architecture comments, no key rotation strategy, no admin route structure details appear anywhere in the public codebase or its commit history.
- **From this point forward:** Builders do not push to GitHub. Ahmad pushes only. No internal files ever reach any public repo again.

---

### 0B — Anonymous Rate Limit Removal ✅ — DONE
Ahmad removed the anonymous rate limit directly via the admin panel. No code change needed. Frontend Builder still needs to remove any hardcoded "3 per day" text from the frontend once assigned. Backend Builder still needs to build self-serve API key signup.

---

### 0C — Whitepaper Rewrite (Frontend Builder)
The current whitepaper at `/whitepaper` is outdated — does not reflect the 35M supply, OTI Economics, whitelist framing, or the full technical depth of what has been built.

**The rewrite must cover:**
- The five-signal scoring algorithm: what each signal measures, why it was chosen, how weighted score is computed. Do not name internal files or expose implementation code.
- Infrastructure: 12 chains supported (7 EVM + 5 non-EVM), two-tier cache, keep-highest logic.
- Chain-specific weight redistribution: Bitcoin as the primary example (Token Holding and Smart Contract signals inapplicable — weight redistributed proportionally). Methodological honesty.
- The Wallet Ownership Registry: what it is, why passkey pre-registration is novel, what problem it solves.
- The BAS attestation layer: what on-chain attestation means, why BNB Chain, what the five trust tiers are.
- OTI Economics: 35M fixed supply, allocation breakdown, the Dynamic Contribution Scale (bonding curve) for the Whitelist Allocation, vesting structure. Framed as utility infrastructure, not investment.
- The roadmap: what is live today, what is being built, what the ecosystem growth funds.
- The market: who needs on-chain trust infrastructure, why this problem is unsolved by existing tools.

**Source material:** Merge the current /whitepaper content with the additions in `docs/whitepaper-additions-draft.md`. Note: the chain table in the additions draft is stale — Fantom/Scroll/Sepolia/Holesky entries must be corrected. Current live chains: 12 (7 EVM + 5 non-EVM). Sui (BF41 open) and BSC/Base/Optimism (503) use Ahmad-approved neutral language — do not list as broken.

**Writing standards (D32):**
- Clean professional prose — no emojis, no AI-native patterns
- Direct and precise — no hollow filler sentences
- Written as if a senior technical team produced it
- All numbers and chains must be accurate to the live system

---

### 0D — Privacy Policy and Terms & Conditions (Frontend Builder)
New pages at `/privacy` and `/terms`, accessible from the footer. Delete all previous presale versions entirely. Use the verbatim text Ahmad provided — do not rewrite.

The T&C and Privacy Policy text is documented in full in MANAGER_HANDOVER.md under "The New Direction — Ecosystem Whitelist Program." Copy exactly — no rewriting, no summarising.

---

### 0E — Ecosystem Whitelist Page + System (Frontend Builder + Backend Builder)
The core of Phase 0. The gated entry point for the Ecosystem Whitelist Node Program.

**Entry gate (what unauthenticated visitors see):**
A professional locked portal — "OTI Infrastructure Hub — Private Whitelist Node Platform. Access is restricted to whitelisted node operators and infrastructure partners." No wallet connect buttons, no token charts, no contract details visible to anyone without a valid code.

**After code verification (what whitelisted operators see):**
- Live progress bar: "Total Ecosystem Committed Allocation Tracker" — pulling from `protocol_state` table, showing progress toward Milestone 2 ($5k) and Milestone 3 ($15k)
- Wallet connect + allocation claim flow
- T&C checkbox (mandatory before code redemption)
- Three-milestone roadmap display

**Milestone display:**
- Milestone 1 — Alpha Core Genesis: Launch /whitelist. First batch of whitelisted operators via invite codes.
- Milestone 2 — Phase 1 Liquidity Seeding: At $5,000 committed allocation. Public Utility Liquidity Layer deployed. OTI becomes redeemable externally.
- Milestone 3 — Deep Liquidity Scaling: At $15,000 committed allocation. Secondary AMM pool. Utility swap rate stabilisation for B2B clients.

**Backend requirements:**
- New DB tables: `whitelist_invites` and `protocol_state` (schemas in MANAGER_HANDOVER.md)
- `POST /api/verify-invite` endpoint: validates invite code + wallet + terms acceptance, marks code used, increments `protocol_state` totals
- Batch code generator API

**Smart contracts (BNB Chain):**
- Vesting/lockup contract: 25% immediate access fuel + 75% Node Collateral Lockup with linear daily vesting
- Dynamic Contribution Scale: bonding curve pricing (starting price and multiplier — Ahmad confirms exact figures; see DECISIONS.md when added)
- All vesting/lockup parameters admin-configurable — nothing hardcoded
- Deploy to BNB testnet first, Ahmad confirms end-to-end, then mainnet

**Admin dashboard additions:**
- Batch invite code generator (generates OTI-XXXX-XXXX codes, saves to `whitelist_invites`)
- Code management panel: all codes, redeemer wallets, funding metrics, slots remaining (cap: 10,000)
- Ban toggle: per code/wallet → status 'banned' → frontend immediately blocks
- Live metric override: manual `total_committed_usd` adjustment in `protocol_state`

**Stack reminder:** Sample code in Ahmad's brief used MongoDB syntax — must be translated to PostgreSQL + Drizzle ORM + Express throughout.

---

### 0F — Docusaurus Docs Site Audit (Frontend Builder)
Review and update all developer docs content before the whitelist launches:
- Remove anything that reveals internal architecture: key rotation strategy, how providers work, admin route structure
- Remove Scroll, Sepolia, Holesky from all chain lists
- Update chain count to 12 (7 EVM + 5 non-EVM) throughout
- Sui and BSC/Base/Optimism: neutral language (do not describe as broken)
- Update API reference to match current actual response shape
- All writing must meet D32 standard: no AI-native patterns, no emojis, professional prose

---

## PHASE 1 — PRE-DISTRIBUTION REQUIREMENTS
**Status: COMPLETE**

All foundation work is live. Homepage, scoring tool at /score, developer docs at /docs, whitepaper at /whitepaper, all admin routes secured, WOR live. See MANAGER_HANDOVER.md for full production state.

---

## PHASE 2 — WALLET OWNERSHIP REGISTRY (WOR)
**Status: COMPLETE**

10 endpoints live, wallet_ownership table, EIP-191 signature verification, passkey-protected compromise reports, dual-factor self-report, 4 admin endpoints. Verified end-to-end. See ARCHITECTURE.md for full technical detail.

---

## PHASE 2B — OTI REVENUE CAMPAIGN (XMTP)
**Status: ONGOING PROGRAM — runs when funded. Not cancelled, not second priority. Tasks 19–22 written and ready in TASKS.md.**

This is a continuous wallet acquisition and attestation revenue program. When Ahmad is ready to run it, send Task 19 prompt to the Backend Builder.

### What the campaign is
A targeted XMTP message campaign to Ethereum wallets with OTI score 75 or above. Each message invites the wallet to pay $1 in BNB to mint an OTI Trust Attestation on BNB Chain via BAS. Revenue: $1,000–$5,000 projected from first campaign. Total cost: $7–25.

**The key insight (D26):** No BSC scoring needed. Every EVM 0x address is the same person on Ethereum and BNB Chain. OTI already scores on Ethereum (live). Payment collected on BNB Chain (cheap gas). The $49/mo Etherscan Lite blocker does not apply.

### Four components, strict dependency order

**Task 19 — Etherscan Key Rotation (Backend Builder) — FIRST**
- Add `ETHERSCAN_API_KEYS` Railway env var (comma-separated, up to 10 keys, maximum 10 per D25)
- Round-robin counter in `chainRegistry.ts`'s `etherscanApiKey()` function
- Backward-compatible: falls back to single `ETHERSCAN_API_KEY` if array not set
- 10 keys = 1M calls/day = 200K–333K wallets scored per day

**Task 20 — BAS Schema Registration + Signing Endpoint (Backend Builder) — SECOND**
- Part A: Register OTI attestation schema on BAS — on-chain transaction, Ahmad signs and pays (~$0.01 gas). Schema fields: `address wallet, uint256 score, string tier, uint256 issuedAt, uint256 expiresAt`. Record resulting schema UID — hardcoded into Task 21 contract.
- Part B: New `src/routes/sign.ts`. Railway env var: `OTI_SIGNING_KEY`. Endpoint: `POST /api/sign/score` behind adminAuth. Signs wallet + score + expiry with ethers.js.

**Task 21 — Smart Contract + XMTP Sender Script (Backend Builder) — THIRD**
- Solidity smart contract on BNB Chain (chainId 56). Chainlink BNB/USD oracle. Accepts exactly $1 in BNB, verifies OTI signature, calls `BAS.attest()`, emits `AttestationMinted`.
- XMTP sender script: reads eligible wallets from `chain_scores`, filters via `canMessage()`, sends `wallet_sendCalls` messages. Ahmad runs manually after test confirms end-to-end.

**Task 22 — Conversion Dashboard (Frontend Builder) — FOURTH**
- Moralis Streams webhook on `AttestationMinted` event to `campaign_conversions` table
- Admin-only view: messages sent, attestations minted, conversion %, revenue in BNB + USD

### Ahmad's actions required before Task 19 can start
1. Register 10 Etherscan accounts (separate emails, not all in one session). Get 10 API keys. Pass to Manager as comma-separated list.
2. Manager adds them to Railway as `ETHERSCAN_API_KEYS=key1,key2,...,key10`
3. Send Task 19 prompt to Backend Builder.

### Campaign targeting reality
- Ethereum wallets with score 75 or above: 2–4 million addresses (active DeFi participants)
- XMTP penetration: 5–15% of all EVM wallets
- Real send list from 3M eligibles: ~150K–450K wallets
- Revenue at 0.25% conversion on 200K sends: ~$500
- Revenue at 0.25% conversion on 400K sends: ~$1,000

---

## PHASE 2B — ATTESTATION STACK (After Campaign Revenue)

Fund with campaign proceeds.

**Backend:**
- `GET /v1/badge/:wallet` — widget API endpoint
- `PATCH /admin/score-source` — switch Score Source mode
- Attestation endpoints (issue, status, revoke)
- Attestation scheduler — daily batch rescore of expiring wallets
- Proactive background scoring pipeline (feeds airdrop eligibility tracking list — D22, must track from day one)
- `wallet_attestations` DB table

**Frontend:**
- Attestation claim UI embedded inside widget
- Public `/verify/:address` page
- Five-tier badge visuals (design finalized July 17, 2026 — full specs in ARCHITECTURE.md)
- Admin panel: attestation stats, Score Source selector, fee settings

### Badge tiers (finalized July 17, 2026)

| Tier | Label | Score Band | Color |
|---|---|---|---|
| 1 | HIGHLY TRUSTED | 75–100 | Mint green `#00E5A0` |
| 2 | TRUSTED | 55–74 | Sky blue `#4FC3F7` |
| 3 | NEUTRAL | 35–54 | Cool gray `#90A4AE` |
| 4 | RISKY | 20–34 | Amber `#FFB300` |
| 5 | HIGH RISK | 0–19 | Red `#FF4444` |

Compromised override: any wallet in `compromised_wallets` gets the Compromised badge regardless of score.

---

## PHASE 3 — MONETIZATION INFRASTRUCTURE + FULL TOKEN ECOSYSTEM
**Status: Planned — infrastructure partially ready**

| Feature | Notes |
|---|---|
| Developer self-serve portal | Sign up, get API key, choose plan — partially built via Phase 0 |
| Pro/Enterprise plan tiers | `plan_configs` table ready, needs new rows + payment checkout |
| Fiat payments | Stripe |
| Crypto payments | Coinbase Commerce (hosted checkout) |
| BSC/Base/Optimism unlock | Ahmad subscribes to Etherscan Lite when ecosystem funds arrive |
| OTI token full ecosystem integration | Revenue-Backed Rewards Pool (15% of revenue buys OTI on market), staking, discount on attestation fee |
| Voluntary staking design | Deferred until OTI has real usage and revenue to build around |
| Exchange listing | Post-Phase 3 milestone — Ahmad decides timing |
| Partner attribution tracking | Required from Phase 3 day one — cannot be retrofitted. Every attestation payment record must carry partner attribution metadata. |

---

## PHASE 4 — GROWTH FEATURES
**Status: Future**

| Feature | Notes |
|---|---|
| Score history UI | DB already accumulating data |
| Multi-chain wallet comparison | Same wallet scored across multiple chains |
| Wallet portfolio view | `wallet_links` table infrastructure already built |
| Webhook alerts | Notify integrators when a watched wallet is compromised |
| Enterprise exchange path | Compliance screening, withdrawal risk scoring |
| Etherscan key upgrade (Standard $199/mo) | After campaign revenue and background scoring volume exceeds 10-key capacity. Upgrade = swap env var, no code change. |
| Bot and suspicious-wallet behavioral detection | Deliberately deferred — Ahmad identified this as a primary use case but it is a full new design. Not scoped. Do not start until Ahmad explicitly reopens it. |

---

## PHASE 5 — DISTRIBUTION CHANNELS
**Status: Last — intentional. Begins only after Phases 2, 3, and 4 are complete.**

Distribution channels bring people to the product. The product needs to be fully built first. When mass adoption comes, the full stack will already be there to receive it.

### Channel 1 — Telegram Bot
- Library: Telegraf (Node.js)
- Lives in `/bots/telegram/` inside backend repo
- Railway second process alongside API server
- Commands: `/score [address] [chain]`, `/help`, `/about`

### Channel 2 — Discord Bot
- Library: discord.js (Node.js)
- Lives in `/bots/discord/` inside backend repo
- Railway third process
- Slash commands: `/score`, `/help`

### Channel 3 — Embeddable Widget
Four hard constraints per D24: zero external dependencies, scoped styles, graceful empty state (renders nothing if no score), no redirect on any interaction. Full spec in DECISIONS.md (D24) and ARCHITECTURE.md.

### Channel 4 — Firefox Extension (then Chrome)
- Separate GitHub repo: `oti-firefox-extension`
- Content script auto-detects `0x[a-fA-F0-9]{40}` addresses on Etherscan, OpenSea, BscScan
- Injects score badge next to each address
- Firefox first (free to publish), Chrome after first revenue ($5 one-time fee)

---

## Revenue Milestones

| Milestone | Target |
|---|---|
| Whitelist total raise target | Ahmad to confirm — Dynamic Contribution Scale runs until all 8,750,000 whitelist OTI is claimed or target is reached |
| Milestone 2 trigger | $5,000 committed allocation → Public Utility Liquidity Layer deployed |
| Milestone 3 trigger | $15,000 committed allocation → Secondary AMM pool funded |
| 1,000 active API users | 90 days after distribution launches |
| 3 paying business integrations | 6 months after distribution |
| First enterprise exchange contract | 12 months after distribution |
