# OTI — Architectural Decisions Log
> Last updated: July 28, 2026 (session 20 — D28: Strategic pivot to private sale first; D29: New token distribution model; D30: All team allocations locked daily-claimable; D31: GitHub security — no internal files in public repos; D32: No AI exposure in public content; D33: Free product — remove anonymous rate limits) | Maintained by: Development Manager

---

## What This File Is

This file records the reasoning behind how OTI was built — not what the code does (that's `ARCHITECTURE.md`) but **why** it does it that way. It exists because:

- Some behavior looks like a bug but was a deliberate tradeoff
- Some behavior is a known technical limitation, not an oversight
- Without this record, future Builders fix things that were intentional and break something else in the process

**Access:**
- **Manager** — reads and writes this file. Only the Manager adds or updates entries.
- **Builders** — read this file. If your task or fix touches something documented here, read the relevant entry before writing a single line of code. Never update this file yourself — report your findings to the Manager.
- **Ahmad** — reads this file. If a decision needs reversing, tell the Manager and they will update the entry.

**Entry statuses:**
- `INTENTIONAL` — a deliberate design choice, confirmed by Ahmad or the Manager. Do not change without Ahmad's explicit approval.
- `TECHNICAL LIMITATION` — the API, platform, or data source at the time didn't support a better approach. Watch for newer APIs or approaches that remove the constraint.
- `REVISIT` — was intentional or constrained at the time, but there is now a better path. Still do not change without Manager instruction — it is flagged for future work, not immediate reversal.
- `PENDING ANSWER` — behavior was observed in the diagnostic audit but the reason is unknown. The Builder has been asked. Do not treat as a bug until the answer comes back and the Manager updates the status.

---

## Confirmed Decisions

---

### D1 — CORS Is Fully Open (No Origin Restrictions)
**Status:** INTENTIONAL
**What the code does:** `app.use(cors())` with no origin restrictions. Any domain can call the OTI API from a browser.
**Why:** OTI is a public developer API. Exchanges, DeFi frontends, and wallet apps need to call it directly from their browser-based dashboards. CORS restrictions would break those integrations. API key quotas are the correct tool for access control — CORS is not.
**Confirmed by:** Ahmad (original architecture decision) + Manager (documented July 5, 2026 in ARCHITECTURE.md)
**Implications for fixes:** Never add origin restrictions without Ahmad's explicit instruction. The open CORS + API key auth combination is correct and intentional.
**Future:** Once paid plans exist, per-key origin allowlisting may be added as an opt-in enterprise feature. Low priority, does not change the default.

---

### D2 — `scoring.ts` Is Never Modified
**Status:** INTENTIONAL
**What the code does:** The backend `src/lib/scoring.ts` contains the core trust algorithm. It is treated as a sacred, unmodifiable file across all Builder work.
**Why:** This is OTI's protected intellectual property — the core competitive differentiator. Changes to the algorithm would directly affect every score ever computed and could constitute liability if a change degraded results for a paying enterprise customer. All adaptation work (chain-aware weighting, signal transformers) is done in wrapper files that sit alongside it, never inside it.
**Confirmed by:** Ahmad (explicit, repeated instruction across all sessions)
**Implications for fixes:** Any fix that touches scoring behavior must be implemented in an adapter or transformer layer. Never edit `scoring.ts` itself.

---

### D3 — BSC, Base, and Optimism Return 503 Intentionally
**Status:** INTENTIONAL
**What the code does:** Requests to `?chain=bnb`, `?chain=base`, and `?chain=optimism` return HTTP 503 "This chain requires an Etherscan plan upgrade."
**Why:** These three chains require the Etherscan Lite plan ($49/month). The code is already implemented — gated behind a 503 pending Ahmad's decision to subscribe. Ahmad will subscribe when private sale revenue comes in.
**Confirmed by:** Ahmad (explicit decision, documented across ARCHITECTURE.md and MANAGER_HANDOVER.md)
**Implications:** These chains are NOT bugs. Do not remove or "fix" the 503. When Ahmad subscribes, a Builder removes the gate — that is the only change needed.

---

### D4 — Admin Page Is URL-Only (No Navigation Link)
**Status:** INTENTIONAL
**What the code does:** The admin panel at `/admin` has no link from any public page. It is accessible only if you know the URL.
**Why:** Ahmad's deliberate decision. The admin panel is for his eyes only — no public visibility, no discoverability by ordinary users or crawlers. Security by obscurity is a secondary layer on top of the `x-admin-secret` header requirement.
**Confirmed by:** Ahmad (explicit decision, documented July 7, 2026)
**Implications:** Do not add a navbar link, a footer link, or any discoverable reference to `/admin` on any public-facing page.

---

### D5 — Bitcoin Weight Redistribution (40% Across 3 Remaining Signals)
**Status:** INTENTIONAL
**What the code does:** On Bitcoin, Token Holding Behavior (20%) and Smart Contract Interactions (20%) are excluded. Their combined 40% weight is redistributed proportionally to the three remaining signals: Wallet Age (25% → 41.7%), Transaction Count (20% → 33.3%), Transaction Timing Patterns (15% → 25%).
**Why:** Bitcoin has no native token standard and no smart contract layer. Scoring a Bitcoin wallet as 0/20 on Token Holding and 0/20 on Smart Contract Interactions would penalize the wallet for what its chain was architecturally not designed to support. The correct behavior is to redistribute weight so the score remains on a 0–100 scale and measures only what is genuinely measurable.
**Confirmed by:** Ahmad (explicit decision made during BF10 architectural discussion, July 11, 2026)
**Implications for whitepaper:** The whitepaper must explain this redistribution principle. The weights table (25/20/20/20/15) applies to EVM chains where all 5 signals are available. Bitcoin is the primary exception. Do not name internal files in the whitepaper.
**Implications for other chains:** If any other chain genuinely lacks one or more signal types, the same redistribution principle applies — confirmed by Ahmad as the general policy.

---

### D6 — 30-Day Score Validity Window
**Status:** INTENTIONAL
**What the code does:** A computed score is valid for 30 days. Requests within that window return the cached score from the database without fetching fresh chain data.
**Why:** Ahmad's decision. Two purposes: (1) Scale — repeat requests return from the database instantly, zero external API calls. (2) Trust decay — on-chain behavior is not static. Monthly rescoring catches wallets that turn malicious over time.
**Confirmed by:** Ahmad (explicit decision, July 11, 2026)

---

### D7 — WOR Compromise Reports Are Fully Automated (No Admin Review)
**Status:** INTENTIONAL
**What the code does:** A verified wallet owner submits a compromise report by combining a wallet signature with a pre-registered passkey. On success, the wallet is immediately flagged — no admin queue, no review step.
**Why:** The passkey pre-registration is the differentiator. An attacker who has the private key cannot replicate the passkey because it was registered before the compromise. The combination is sufficient proof that the requester is the original owner.
**Confirmed by:** Ahmad (explicit decision, documented across ROADMAP.md and MANAGER_HANDOVER.md)

---

### D8 — Scroll, Sepolia, and Holesky Are Not Implemented
**Status:** INTENTIONAL
**What the code does:** Requests to these chains return HTTP 400 "Invalid enum value." They do not exist in the codebase.
**Why:** Listed in early documentation as planned but never implemented. Sepolia and Holesky are Ethereum testnets — limited value in a production trust scorer. Scroll requires the same Etherscan Lite plan as BSC/Base/Optimism.
**Status of documentation:** Chain count across all materials must reflect 12 working chains (not 15, not 11 — Sui is broken via BF41 but its features remain in the code pending Ahmad's fix; BSC/Base/Optimism return 503 but their code exists). Do not claim 15 supported chains anywhere.

---

### D9 — "Fantom" Chain Entry Rebranded to Sonic (Chain ID Stays 146)
**Status:** INTENTIONAL (resolved)
**What happened:** `CHAIN_ID.fantom = "146"` queried Sonic Mainnet data and labeled it Fantom Opera. Etherscan V2 no longer serves Fantom Opera under any chain ID — Fantom migrated to Sonic and the legacy ftmscan.com domain no longer resolves. Ahmad's decision: rename the entry to `sonic`, keep chain ID 146. Closed July 12, 2026 (BF22).

---

### D16 — No Signal Value May Be Estimated, Inferred, or Guessed — Only Real On-Chain Data
**Status:** INTENTIONAL — standing policy, applies to all past and future signal work
**What this means:** Every number a signal reports must come directly from real on-chain data. Never guess, infer from a proxy, or default to a placeholder. The only exception is a genuine hard cap imposed by the chain/data source itself — and even then, the cap must be documented, not silently presented as the true value.
**Why:** Directly triggered by the Tron contract-diversity investigation where Backend's own "641 contract interactions" verification turned out to be fabricated by a classification bug. A signal must never look right without being right.
**Implications:** Before reporting a signal fix as verified, the Builder must show the raw on-chain data source proving the number. This is a standing review bar for every future Builder report.

---

### D17 — Phase 2B Uses BAS (BNB Attestation Service) as the Sole Attestation Layer
**Status:** INTENTIONAL
**Why:** Ahmad's explicit decision, July 14, 2026. BNB Chain chosen because: OTI token launches on BSC (one chain for token, payments, and attestation); BNB gas is cheap; BAS is live with 3.5M+ attestations; EVM wallet addresses are the same across chains so one BAS attestation covers the full cross-chain score.
**Confirmed by:** Ahmad, July 14, 2026.

---

### D18 — On-Chain Soulbound NFT Removed From Phase 2B
**Status:** INTENTIONAL
**Why:** Ahmad's explicit decision, July 14, 2026. BAS attestation covers the same use case without rescoring complexity, without per-chain contract deployment, and without gas costs at the user level. Do not reconstruct without Ahmad explicitly reopening it.

---

### D19 — MetaMask Snap Removed From Phase 2B and Phase 5
**Status:** INTENTIONAL
**Why:** Ahmad's explicit decision, July 14, 2026. Requiring users to install the OTI Snap creates friction for marginal display coverage that the widget and extension already cover without installation. Do not reconstruct without Ahmad explicitly reopening it.

---

### D20 — Attestation Fee Is One-Time, Not Recurring
**Status:** INTENTIONAL
**Why:** Ahmad's explicit decision, July 14, 2026. One-time fee removes recurring payment friction, maximises user volume, and is sustainable because the total addressable wallet population is in the hundreds of millions. OTI covers rescoring from attestation fee revenue, widget revenue, API subscriptions, and token treasury.
**Implications:** Fee amount, OTI token discount rate, and the 10M free-tier cap are configured via admin panel — never hardcoded.

---

### D21 — First 10 Million Attestations Are Free
**Status:** INTENTIONAL
**Why:** Ahmad's explicit decision, July 14, 2026. When 10 million wallets carry OTI attestations, partners encounter OTI badges widely enough that integrating OTI's widget becomes a business necessity. The free tier cost is an investment in that network effect.
**Implications:** The 10M threshold and free/paid gate are managed via admin panel — not hardcoded.

---

### D22 — OTI Token Rewards First 1 Million Wallets in OTI's Scoring Database
**Status:** INTENTIONAL
**Why:** Strategic discovery mechanism — OTI sends tokens to wallets that were proactively pre-scored. Owners check their wallet, see unfamiliar tokens, search "OTI," and discover they already have a trust score. Discovery happens as a consequence of the airdrop rather than being a prerequisite for it.
**Confirmed by:** Ahmad, July 17, 2026.
**Critical tracking implication:** The eligibility counter starts when proactive background scoring begins. The tracking list must be built into the background scorer from day one — cannot be reconstructed after the fact.

---

### D23 — Score Source Switcher: Widget Data Source Is Server-Side Configuration
**Status:** INTENTIONAL
**Why:** Changing the data source later must not require partner notification or embed code updates. Server-side configuration means one admin panel change propagates to every embedded widget worldwide within 60 seconds. The embed code must never include a `data-source` attribute — partners never know or care which source their widget reads from.
**Confirmed by:** Ahmad, July 17, 2026.

---

### D24 — Widget Embed Is a Self-Contained Vanilla JS Script Tag With Four Hard Constraints
**Status:** INTENTIONAL
**Four constraints — non-negotiable at implementation time:**
1. Zero external dependencies — vanilla JS and Fetch API only.
2. Scoped styles — all CSS scoped via unique class prefix or Shadow DOM. Must not leak into the partner's page.
3. Graceful empty state — if no score exists, the widget renders nothing silently.
4. No redirect on any interaction — the script must never call `window.location` or navigate the user away.
**Confirmed by:** Ahmad, July 17, 2026.

---

### D25 — Etherscan API Key Rotation: Maximum 10 Free Keys, Hard ToS Ceiling
**Status:** INTENTIONAL
**Why:** Above ~10 free Etherscan accounts is clearly ToS-violating account farming. Etherscan detects same-IP registration patterns, same-server usage, same wallet patterns. 10 is the practical ceiling. Upgrade path beyond 10: Etherscan Standard plan ($199/mo) — swap env var, no code change.
**Confirmed by:** Ahmad, July 19, 2026.

---

### D26 — Ethereum Scores Are Used for the BNB Chain Campaign (BSC Blocker Bypassed)
**Status:** INTENTIONAL
**Why:** Every EVM wallet address (0x...) is the same key pair on Ethereum and BNB Chain. Scoring on Ethereum and attesting on BNB Chain is technically correct — not a shortcut. The Etherscan Lite $49/mo subscription is not required for the XMTP campaign.
**Confirmed by:** Ahmad, July 19, 2026.

---

### D27 — Campaign-First Phase 2B: Revenue Before Full Attestation Stack
**Status:** INTENTIONAL — superseded in priority by D28 (private sale first), but the campaign logic remains valid for after private sale
**Why:** The Revenue Campaign (Tasks 19–22) is a self-contained revenue loop built entirely on existing infrastructure. The full attestation stack can be built after, funded by campaign proceeds.
**Updated context (July 28, 2026):** The XMTP campaign is now the SECOND funding mechanism, after the private sale. Both plans remain valid. Private sale happens first to fund servers and initial development. XMTP campaign follows once funding is secured.
**Confirmed by:** Ahmad, July 19, 2026.

---

### D28 — Strategic Pivot: Private Sale First, XMTP Campaign Second
**Status:** INTENTIONAL
**What this means:** All building operations are paused until a private sale site is designed and built. The private sale raises money first. The XMTP campaign (Tasks 19–22, previously the sole revenue path) runs after private sale money is secured.
**Why:** Ahmad's decision, July 28, 2026. The XMTP campaign is a viable revenue path but requires time to build and a period of uncertainty. A private sale targets committed buyers who understand the product and believe in the vision — faster, more certain, and more substantial funding. The private sale also serves as a forcing function to make the product fully professional and public-facing before any serious money changes hands.
**Implications:** Tasks 19–22 remain written and ready in TASKS.md — they are not cancelled, only deprioritized. The next work is the private sale site and all its prerequisites (whitepaper rewrite, GitHub cleanup, privacy policy, T&Cs, free product access, BNB Chain smart contracts).
**Confirmed by:** Ahmad, July 28, 2026.

---

### D29 — Token Distribution Model: 25% Free Immediately, 75% Auto-Staked Daily Over 5 Years
**Status:** INTENTIONAL
**What the private sale contract does:** On purchase, 25% of the buyer's tokens are sent immediately to their wallet. 75% are automatically staked in the distribution contract and claimable daily over 5 years (1,825 days). Equal daily portions accumulate if not claimed.
**Why:** Ahmad's decision, July 28, 2026. The 75% auto-stake protects price stability in the early growth phase — buyers cannot dump their full allocation immediately. The 25% free portion gives buyers immediate proof of ownership and lets them use or move their tokens right away. The 5-year daily release creates a predictable, transparent unlock schedule that private buyers can model and trust.
**Implications for smart contracts:** The private sale contract on BNB Chain must implement this distribution automatically — no manual steps by Ahmad or any team member after a purchase. The buyer's dashboard must show: immediate balance, staked balance, daily claimable amount, accumulated unclaimed tokens.
**Confirmed by:** Ahmad, July 28, 2026.

---

### D30 — All Team-Controlled Allocations Are 100% Locked, Daily Claimable Over 5 Years
**Status:** INTENTIONAL
**What this means:** Team & Founders, Ecosystem & Partnerships, Revenue-Backed Rewards Pool, and Treasury / Reserve allocations are all fully locked from day one. They release in equal daily portions over 5 years — the same mechanism as the private sale buyers' 75% staked portion.
**Why:** Ahmad's decision, July 28, 2026. This is a trust signal to private buyers: the team cannot dump. If the team's tokens released monthly in large chunks, early buyers would face the risk of team-side sell pressure at any monthly unlock. Daily release is smoother, more predictable, and demonstrates that the team's economic incentives are aligned with buyers over the long term. The only exception is the Liquidity & Market Making bucket (10%, 3M OTI) which must be fully available at listing to provide real DEX trading depth.
**Confirmed by:** Ahmad, July 28, 2026.

---

### D31 — GitHub Security: No Internal Files in Public Repositories
**Status:** INTENTIONAL
**What this means:** The OTI docs repo on GitHub is currently public and contains internal files: FIXES.md, TASKS.md, ARCHITECTURE.md, BUILDER_ONBOARDING.md, MANAGER_HANDOVER.md and others. These must not be publicly visible. They expose internal architecture, bug history, builder processes, key rotation strategy, admin route structure, and other information that should remain private.
**Ahmad's direction, July 28, 2026:**
- The docs repo should be made private OR stripped to only public-facing content before the private sale launches.
- From this point forward, Builders do not push to GitHub. Ahmad pushes. Builders work in their own Replit environments, do the work, and Ahmad pulls and reviews.
- No internal files (task files, fix files, architecture docs, builder onboarding) ever reach any public GitHub repo again.
- The frontend public repo must be audited: no internal comments, no references to key rotation strategy, no admin route structure visible in public code.
**Why:** Private buyers, press, and competitors will check GitHub before engaging. Finding a public bug list and internal architecture document would undermine trust in OTI's professionalism and expose strategic information.
**Confirmed by:** Ahmad, July 28, 2026.

---

### D32 — No AI Exposure in Any Public-Facing Content
**Status:** INTENTIONAL
**What this means:** Nothing in public-facing content — the website, whitepaper, docs site, private sale page, GitHub README, social media — should reveal or suggest that OTI was built with AI assistance. This includes: no emojis in professional text, no AI-native writing patterns (hollow filler sentences, excessive caveats, "it's worth noting that" constructions), no generic-sounding copy that reads as AI-generated.
**Why:** Ahmad's direction, July 28, 2026. Private buyers, developers, and potential enterprise partners evaluate the professionalism and credibility of the team through every piece of public content. Content that reads as AI-generated undermines the perception of a serious, technical team. Every public document should read as if a senior human engineer or technical writer produced it.
**Implications:** The whitepaper rewrite, all updated docs site content, the private sale page copy, and the GitHub README must all be reviewed against this standard before going public. The Manager and Builders must apply this standard to all public-facing output.
**Confirmed by:** Ahmad, July 28, 2026.

---

### D33 — Free Product: Anonymous Rate Limits Removed for Public Users
**Status:** INTENTIONAL
**What this means:** The anonymous rate limit (currently 3 requests/day per IP) on the scoring tool and API will be removed or significantly raised. The product is made genuinely free and open for anyone to use — both the website scoring tool at `/score` and the API for developers.
**Why:** Ahmad's decision, July 28, 2026. Private buyers will test the product before investing. If they hit a 3-request wall, the product does not feel real. A genuinely free product also accelerates the wallet database growth (every scored wallet is permanent value that accumulates) and demonstrates product confidence. Developer API access will move to self-serve — no need to contact Ahmad for a key.
**Implications:** The `plan_configs` table anonymous daily limit needs to be updated (admin panel change — no code deploy). A self-serve developer API key signup flow needs to be built. The key rotation system (10 Etherscan keys) provides the capacity to serve increased request volume at $0 additional cost.
**Implementation timing:** The rate limit change is a single admin panel update Ahmad can make immediately. The self-serve signup flow is a Builder task.
**Confirmed by:** Ahmad, July 28, 2026.

---

### D34 — Whitelist Program Replaces Private Sale — Regulatory Compliance
**Status:** INTENTIONAL — NON-NEGOTIABLE
**What this means:** The entire "private sale / presale / ICO / token sale" direction has been replaced by the "Ecosystem Whitelist Node Program." This is a permanent vocabulary and framing change. Banned vocabulary: token sale, private sale, presale, ICO, buy tokens, invest, ROI, yield, investors, trading, listing. Required vocabulary: Ecosystem Whitelist, Acquire Network Access Fuel, Claim Allocation, Node Collateral Lockup, Linear Network Vesting, Whitelisted Operators, Public Utility Liquidity Pool Seeding.
**Why:** Ahmad's direction, July 29, 2026. Running an open public token checkout while marketing on social media triggers "Unregistered Public Offering" under local and international regulatory frameworks. The whitelist framing converts the launch into a private technical ecosystem onboarding program — a gated, invite-only access system that looks like a locked private network onboarding tool to a public crawler or regulator, not a token sale.
**Scope:** Every piece of public-facing content (website, whitepaper, docs site, GitHub README, social media), every task prompt written to Builders, and every piece of code Builders write must use whitelist vocabulary exclusively.
**Confirmed by:** Ahmad, July 29, 2026.

---

### D35 — Off-Chain Referral Tracking
**Status:** INTENTIONAL
**What this means:** Referral and invite relationships are tracked in the PostgreSQL backend database (`whitelist_invites` table), not on-chain. The admin has full visibility and control over all referral data and can adjust commission records operationally.
**Why:** Ahmad's decision, July 29, 2026. On-chain referral tracking would make all commission relationships and wallet connections permanently public and auditable by anyone, removing operational flexibility and potentially creating regulatory exposure. Off-chain tracking keeps the admin in full control — commissions can be corrected, adjusted, or managed without gas fees and without public visibility.
**Implementation:** `whitelist_invites` table stores invite code, redeemer wallet, contribution amount. Commission calculation and distribution managed by backend.
**Confirmed by:** Ahmad, July 29, 2026.

---

### D36 — All Vesting and Lockup Parameters Are Admin-Configurable
**Status:** INTENTIONAL — NON-NEGOTIABLE
**What this means:** No vesting percentage, lockup duration, or distribution parameter for the whitelist program may be hardcoded in smart contracts or backend logic. All parameters must be configurable by OpenFlow Labs via the admin dashboard. This includes: the 75% Node Collateral Lockup percentage, the linear vesting schedule duration, the daily release rate, and any other distribution parameter.
**Why:** Ahmad's direction, July 29, 2026. Hardcoded vesting means any change requires a contract redeployment — expensive, risky, and requiring public announcement. Admin-configurable parameters mean OpenFlow Labs can adjust the vesting schedule operationally as the ecosystem evolves, without on-chain upgrades.
**Implication for Builders:** Any Builder implementing the whitelist smart contract or backend vesting logic must read parameters from a configurable admin-set source, not from constants. Flag this to the Manager if it creates a technical constraint.
**Confirmed by:** Ahmad, July 29, 2026.

---

### D37 — OTI Total Supply Is 35,000,000 (Fixed)
**Status:** INTENTIONAL
**What this means:** The final, correct total supply of OTI is 35,000,000 tokens. Fixed. No inflation, no future minting, no additional creation under any condition. Previous references to 30M or any other figure are incorrect and superseded.
**Allocation breakdown:** Ecosystem Whitelist 25% (8.75M), Network Reserve 20% (7M), Founders 15% (5.25M), Strategic Partnerships 10% (3.5M), Liquidity 10% (3.5M), Rewards Pool 10% (3.5M), Future Strategic Investment 5% (1.75M), Operations Reserve 5% (1.75M).
**Rewards Pool mechanism:** Funded by revenue-backed open-market OTI purchases (15% of platform revenue) — not by inflation. Total supply cap is never breached.
**Confirmed by:** Ahmad, July 29, 2026 (post-advisor session).

---

### D38 — Referral Commissions Denominated in OTI Token
**Status:** INTENTIONAL
**What this means:** All referral and invitation reward commissions are paid in OTI token, not in BNB or any fiat-equivalent stablecoin.
**Why:** Ahmad's decision, July 29, 2026. Paying commissions in OTI keeps demand for OTI circulating within the ecosystem, strengthens utility-driven demand, and avoids creating a cash-out mechanism that bypasses the token entirely.
**Confirmed by:** Ahmad, July 29, 2026.

---

### D39 — Frontend GitHub Repo Requires Cleanup Before Whitelist Launch
**Status:** REQUIRED ACTION
**What this means:** The public frontend GitHub repo currently contains internal workspace files (TASKS.md, FIXES.md, ARCHITECTURE.md, BUILDER_ONBOARDING.md, etc.) that were pushed alongside source code. The repo must be cleaned to show only actual built source code — no internal documentation, no Builder workspace files, no references to internal project management, no key rotation strategy, no admin route structure details.
**Why:** Ahmad's direction, July 29, 2026. The frontend repo is intentionally public for code transparency. Potential enterprise clients, developers, and press will inspect it before engaging. Finding internal bug lists and architecture docs alongside the source code looks unprofessional and exposes strategic information.
**Task:** Frontend Builder cleanup task must be written and assigned before whitelist launches.
**Confirmed by:** Ahmad, July 29, 2026.

---

### D40 — Dynamic Contribution Scale: Linear Bonding Curve on 7,000,000 OTI
**Status:** INTENTIONAL — CONFIRMED PARAMETERS
**What this means:** The paid-claims portion of the Ecosystem Whitelist Allocation uses a linear bonding curve. Contribution Rate starts at $0.001190/OTI and ends at $0.005952/OTI. 5× multiplier from start to end. Total raise target: $25,000 from 7,000,000 OTI.
**Formula:** `Rate(x) = 0.001190 + (0.005952 - 0.001190) × (x ÷ 7,000,000)` where x = total DCS tokens claimed so far.
**Why:** Ahmad's direction, July 29, 2026. Rewards early operators (lower contribution rate) while creating genuine economic urgency without speculation language. Rate increases are network scarcity mechanics, not investment price movements. The 5× multiplier creates a meaningful benefit for early action without making late-stage rates look punishing.
**Framing:** The curve is the "Dynamic Contribution Scale." Rate increase = "Contribution Tier advances as network capacity fills." Never called "token price" in any public-facing content.
**Confirmed by:** Ahmad, July 29, 2026.

---

### D41 — Ecosystem Rewards Pool: Inverse Bonding Curve Tied to DCS Progress
**Status:** INTENTIONAL — CONFIRMED DESIGN
**What this means:** Reward amounts for referrals and social tasks are not fixed — they decrease as the Dynamic Contribution Scale fills. The reward multiplier = DCS Remaining ÷ 7,000,000. As more DCS tokens are claimed, all reward amounts shrink proportionally.
**Base reward amounts (admin-configurable):** Referral = 3,000 OTI, Post/Tag = 1,000 OTI, Share = 500 OTI, Follow (Twitter/X) = 500 OTI, Follow (Telegram) = 500 OTI. All base amounts are set via admin dashboard — never hardcoded.
**Why:** Ahmad's direction, July 29, 2026. Creates the same early-action urgency on the engagement side as the rising contribution rate creates on the claims side. Both curves are driven by one variable (DCS tokens remaining), so the entire system has a single source of truth. The /whitelist page shows both live counters simultaneously — contribution rate going up, reward amounts going down — creating visible urgency without any speculative language.
**Implementation note:** Reward multiplier must be read from `protocol_state.total_slots_claimed` (or DCS tokens claimed counter) at the moment of reward issuance. Rewards are paid from the ERP sub-pool (1,750,000 OTI), not from the DCS sub-pool. The two sub-pools are independent — rewards do not advance the DCS curve.
**Confirmed by:** Ahmad, July 29, 2026.

---

## Pending Answers — Awaiting Builder Response

---

### D10 — EVM Transaction Fetch Is Capped at a Single Page (1,000 / 500 / 500)
**Status:** REVISIT — confirmed as a bug, not a deliberate rate-limit safeguard
**Fix scope:** Paginate up to Etherscan's real ceiling (10 pages at offset=1000), respecting the 3 req/s throttle. Tracked in FIXES.md.

---

### D11 — EVM Token Holdings Computed From Transfer History, Not Real Balance
**Status:** REVISIT — confirmed as a bug via live counterfactual
**Fix scope:** Use Etherscan's `tokenbalance` action for real balance per token/address. Tracked in FIXES.md.

---

### D12 — Tron and Solana Transactions Use the Wallet's Own Address as `to` Field
**Status:** REVISIT — confirmed as a straightforward bug
**Fix scope:** Pure data-extraction fix — real counterparty data is present in the raw API responses and just needs to be read correctly. Tracked in FIXES.md.

---

### D13 — TON Jetton Holdings Inferred From Outgoing Messages
**Status:** TECHNICAL LIMITATION — resolution path under active discussion. Do not close until Manager updates this entry.

---

### D14 — Timing Pattern Signal Uses Block Timestamps, Not Wall-Clock Time
**Status:** Under review. Do not treat as a bug until Manager updates this entry.

---

### D15 — zkSync Era Contract Interactions May Undercount System Transactions
**Status:** Under review. Do not treat as a bug until Manager updates this entry.

---

### D42 — Whitelist Smart Contracts: Builder Deploys Everything Including Mainnet — Ahmad Not Involved Until Workspace Deletion
**Status:** INTENTIONAL
**What this means:** For Task 28 Part D, the Backend Builder handles the entire smart contract lifecycle without Ahmad's involvement: generates the deployer wallet, funds it, deploys to testnet, runs end-to-end tests, deploys to mainnet, and delivers the complete handover package to the Manager. The Manager relays the package to Ahmad. Ahmad's only action is to receive and save the handover package (deployer private key + contract addresses), then delete the Builder workspace.
**Why:** Ahmad's direction, July 30, 2026. Ahmad does not want to be involved in any intermediate step. The Builder is trusted to handle key generation, funding, deployment, and testing fully independently.
**Implications for Builder:**
- Deployer wallet private key must never appear in any committed file — Replit env vars only
- Builder funds mainnet gas from their own wallet; Ahmad reimburses at handover
- Handover package is delivered to the Manager, not directly to Ahmad
- If the real OTI BEP-20 mainnet token address is not yet available at deploy time, use a placeholder and document clearly — Ahmad updates the contract owner address after handover
**Confirmed by:** Ahmad, July 30, 2026.
