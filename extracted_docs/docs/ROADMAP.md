# OTI — Product Roadmap
> Last updated: July 28, 2026 (session 20 — Strategic pivot: Phase 0 (Private Sale Infrastructure) added as new highest priority. Phase 2B XMTP campaign moved to On Hold — runs after private sale. New token distribution model (25%/75% daily) reflected throughout. GitHub security, free product access, no AI exposure decisions added.)
> Source document: OTI Full Distribution & Technical Development Strategy (Founder's Playbook, July 2026) + Ahmad direction session 20.

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

## Current State (July 28, 2026)

**What is live in production:**
- Backend scoring API on Railway, 12 chains working
- Frontend at otiscore.vercel.app: homepage, /score, /whitepaper, /services, /register, /report, /admin
- Developer docs at otiscore.vercel.app/docs/
- WOR (Wallet Ownership Registry) fully live
- Admin panel secured
- Score sharing PNG cards live

**What is not yet live:**
- Private sale site and smart contracts
- Self-serve developer API key signup
- Free product access (anonymous limit currently 3/day — to be raised)
- Privacy policy and terms and conditions
- Whitepaper rewrite (current version undersells technical depth and does not reflect new token structure)
- BAS attestation layer and widget
- Bots, extension, and other distribution channels

---

## PHASE 0 — PRIVATE SALE INFRASTRUCTURE
**Owner: Both Builders | Status: NEXT — full design required. No Builder starts until Ahmad answers open questions in MANAGER_HANDOVER.md.**
**Strategic context (D28):** Ahmad's decision July 28, 2026. The private sale is the primary funding mechanism. It runs before the XMTP campaign. Money raised keeps servers running and funds continued development.

---

### 0A — GitHub Security (Ahmad action — no Builder)
The docs repo is currently public and contains internal files (FIXES.md, TASKS.md, ARCHITECTURE.md, BUILDER_ONBOARDING.md, MANAGER_HANDOVER.md). Ahmad must make the docs repo private OR strip it to only public-facing content before the private sale launches.

From this point forward, Builders do not push to GitHub. Ahmad pushes only. No internal files (task files, fix files, architecture docs) ever reach any public repo.

---

### 0B — Anonymous Rate Limit Removal (Ahmad admin panel + Builders)
The scoring product must be genuinely free to use before private buyers arrive.

- **Ahmad (admin panel — no code deploy needed):** Update the `anonymous` plan in the `plan_configs` table to a high or unlimited daily limit. This is a database value Ahmad controls directly from the admin panel.
- **Frontend Builder:** Remove or update any hardcoded "3 per day" text in the frontend to reflect the real new limit.
- **Backend Builder:** Build self-serve API key signup flow so developers can obtain a key without contacting Ahmad.

---

### 0C — Whitepaper Rewrite (Frontend Builder)
The current whitepaper exists at `/whitepaper` but does not reflect the technical depth of what has been built or the new token/presale structure.

**The rewrite must cover:**
- The five-signal scoring algorithm methodology: what each signal measures, why it was chosen, how the weighted score is computed. Do not name internal files or expose implementation code.
- The infrastructure: chains supported and how scoring works across different chain architectures (EVM, Bitcoin, Solana, TON, Tron). Explain the two-tier cache and keep-highest logic in plain terms — these are trust signals for buyers, not just technical details.
- Chain-specific weight redistribution: Bitcoin is the primary example — Token Holding and Smart Contract signals are inapplicable to Bitcoin and their weight is redistributed to the three remaining signals. This is honest and shows methodological rigor.
- The Wallet Ownership Registry: what it is, why the passkey pre-registration model is novel, why it solves what other compromise-detection systems don't.
- The BAS attestation layer: what on-chain attestation means, why BNB Chain was chosen, what the five trust tiers are and how they map to scores.
- The OTI token: 30M fixed supply, allocation breakdown, the distribution model (25% free immediately to buyers, 75% auto-staked daily over 5 years, all team allocations locked daily claimable over 5 years — the team cannot dump).
- The roadmap: what is live today, what is being built, what the private sale funds.
- The market: who needs on-chain trust infrastructure, what the total addressable market looks like, why this problem is unsolved by existing tools.

**Writing standards (D32):**
- Clean, professional prose — no emojis, no AI-native writing patterns
- Direct and precise — no hollow filler sentences
- Written as if a senior technical team produced it
- All chain counts must be accurate: 12 chains currently scored (7 EVM + 5 non-EVM). Do not mention Scroll, Sepolia, or Holesky. For Sui (broken) and BSC/Base/Optimism (503), use language Ahmad approves — see open questions in MANAGER_HANDOVER.md.

---

### 0D — Privacy Policy and Terms & Conditions (Frontend Builder)
Both pages must be live before the private sale opens. New pages at `/privacy` and `/terms`, accessible from the footer.

**Privacy Policy must cover:**
- What data OTI processes: on-chain public blockchain data only (wallet addresses, transaction histories, token balances). No personally identifiable information is collected or stored.
- How scores are computed and stored
- What data is shared with third parties (none, except as required by law)
- User rights: how to request data deletion
- Cookie and analytics policy

**Terms and Conditions must cover:**
- Use of the scoring product
- API developer terms: acceptable use, rate limits, prohibited uses (scraping, reselling without authorization)
- WOR terms: what registration and compromise reports commit the user to
- Private sale terms: what buyers are purchasing, the token distribution schedule (25%/75%), the risks of purchasing a pre-exchange token, the non-guarantee of returns or exchange listing
- Referral program terms: eligibility, commission calculation, payout schedule
- Limitation of liability

---

### 0E — Private Sale Site (Frontend Builder + Backend Builder)
The core of Phase 0. A purpose-built private sale experience that converts informed visitors into buyers.

**Information that must be present:**
- What OTI is and what it's building — the product, the technical infrastructure, the market
- Why this is the right time to buy: what the private sale funds, what milestones unlock
- Token economics: supply (30M fixed), allocation breakdown, the private sale terms
- Distribution model clearly explained: 25% free to your wallet immediately, 75% auto-staked and claimable daily over 5 years. The team's tokens work exactly the same way — the team cannot dump.
- A live dashboard for buyers: see your immediate balance, your staked balance, your daily claimable amount, your accumulated unclaimed tokens
- Referral system: your referral link, how commissions work, your earnings
- Complete FAQ: thorough and honest answers to every objection a serious buyer would raise
- Roadmap: what is live, what is being built, what the money funds

**Technical requirements:**
- Smart contracts on BNB Chain: sale contract (accepts BNB/USDT), distribution contract (25%/75% split executed on purchase, daily claim function), referral tracking, admin controls (pause, parameters, withdraw)
- All contracts deployed and tested on BNB testnet before mainnet — Ahmad confirms testnet behavior before mainnet deploy
- Smart contracts hold real money — no shortcuts in testing
- The sale closes automatically when the target raise is reached (cap enforced in contract)
- Design standard: professional, matches OTI's black/mint visual system, no emojis, reads like a serious investment opportunity

---

### 0F — Docusaurus Docs Site Audit (Frontend Builder)
Review and update all docs content:
- Remove any content that reveals internal architecture (key rotation details, how providers work, admin route structure)
- Remove Scroll, Sepolia, Holesky from all chain lists
- Update chain count to accurate figure
- Sui and BSC/Base/Optimism: use Ahmad-approved language (see open questions in MANAGER_HANDOVER.md)
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
**Status: ON HOLD — runs after private sale money is secured**

Tasks 19–22 are written and ready in TASKS.md. The campaign is not cancelled — it is the second funding mechanism. When Ahmad is ready to run it, the next Manager sends Task 19 prompt to the Backend Builder.

Full campaign detail preserved below for continuity.

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

Fund with campaign proceeds. Full specifications in ROADMAP.md (prior version) — preserved in TASKS.md.

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
| BSC/Base/Optimism unlock | Ahmad subscribes to Etherscan Lite when presale pays |
| OTI token full ecosystem integration | Revenue-Backed Rewards Pool, staking, discount on attestation fee |
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
| Private sale target raise | Ahmad to confirm — sale runs until target is reached |
| 1,000 active API users | 90 days after distribution launches |
| 3 paying business integrations | 6 months after distribution |
| First enterprise exchange contract | 12 months after distribution |
