# OTI Token — Supply, Sale Structure & Revenue Distribution

> Last updated: July 28, 2026 (session 20 — Complete rewrite. New token distribution model confirmed by Ahmad. Previous 36-month monthly linear vesting model deleted and replaced. All team-controlled allocations now locked with daily linear claim. Private sale structure updated. Price and exact sale token amount deferred to build phase.)
> **Status: Direction confirmed by Ahmad July 28, 2026 — exact price and token amount for private sale to be decided during build.**
> **This is OTI's own independent token — NOT the OpenFlow "FLOW" ecosystem token. OTI has its own supply, its own economics, and can operate with or without OpenFlow.**
> **Launch chain: BNB Smart Chain (BSC). Cross-chain expansion planned later.**
> **Do not add price or liquidity pool design — Ahmad decides these during the build phase.**

---

## 1. Total Supply

**30,000,000 OTI — fixed supply, no inflation, no mint function after launch.**

Fixed supply is a deliberate trust signal. Total dilution is knowable from day one. No bucket can be silently expanded.

---

## 2. Allocation

| Bucket | % | Tokens | Purpose |
|---|---|---|---|
| Private Sale | 15% | 4,500,000 | Funds building OTI — private sale runs before exchange listing |
| Team & Founders | 15% | 4,500,000 | Long-term founder ownership (Ahmad + Musty) |
| Ecosystem & Partnerships | 15% | 4,500,000 | Developer grants, integration incentives, protocol/exchange partnerships |
| Revenue-Backed Rewards Pool | 30% | 9,000,000 | Staking/holder rewards — funded by real OTI API revenue via buyback, not token inflation |
| Liquidity & Market Making | 10% | 3,000,000 | DEX trading pool depth, exchange listing support |
| Treasury / Reserve | 15% | 4,500,000 | Operating costs, security audits, contingency — multisig-controlled |
| **Total** | **100%** | **30,000,000** | |

---

## 3. Token Distribution Model (Ahmad — confirmed July 28, 2026)

This is the core distribution mechanism. Every allocation follows this logic:

### 3.1 Private Sale Buyers

When a buyer purchases OTI tokens in the private sale:

- **25% of their purchased tokens are released immediately** — visible in their wallet or in the OTI private sale dashboard from the moment of purchase. No lock, no wait.
- **75% of their purchased tokens are automatically staked** at the point of purchase. This 75% is claimable daily in equal portions over 5 years (1,825 days). Each day, the buyer can claim their daily portion. There is no penalty for not claiming — unclaimed daily portions accumulate and remain claimable.

**Example:** A buyer purchases 10,000 OTI.
- 2,500 OTI arrive in their wallet immediately.
- 7,500 OTI are auto-staked. Daily claimable amount: 7,500 / 1,825 = ~4.11 OTI/day for 5 years.

**Only the 25% free portion represents immediate available supply.** The 75% staked is locked in the distribution contract and releases gradually — this protects price stability during the early growth phase.

### 3.2 All Team-Controlled Allocations

Every allocation that is under team or protocol control — Team & Founders, Ecosystem & Partnerships, Revenue-Backed Rewards Pool, Treasury / Reserve — is **100% locked** from day one, claimable daily in equal portions over the same 5-year period (1,825 days).

No team member, no founder, no treasury wallet receives a lump sum at any point. The team's tokens release at the same daily rate as private sale buyers' staked portions. This is a deliberate trust signal to private buyers: the team cannot dump.

### 3.3 Liquidity & Market Making — Exception

The Liquidity & Market Making bucket (10%, 3,000,000 OTI) is the one exception. It must be fully available at listing to provide real DEX trading depth. Vesting liquidity would leave the token illiquid at launch.

### 3.4 Available Supply Summary

At the moment of private sale close and exchange listing:

| Source | Tokens | Status |
|---|---|---|
| Private sale — 25% free portions | Up to 1,125,000 OTI (25% of 4.5M sold) | Immediately in buyer wallets |
| Liquidity & Market Making | 3,000,000 OTI | In DEX pool at listing |
| All other allocations | 26,625,000 OTI | Locked, claimable daily over 5 years |

This means at launch, only ~4,125,000 OTI (13.75% of total supply) is liquid — the rest releases gradually. This is intentional.

---

## 4. Private Sale Structure

### 4.1 Private Sale (Runs Before Exchange Listing)

| | |
|---|---|
| Purpose | Fund building OTI — keeps servers running, funds the XMTP campaign, funds Phase 3 development |
| Chain | BNB Smart Chain (BSC) |
| Accepted currency | BNB / USDT on BSC |
| Token amount for sale | To be decided by Ahmad during build phase |
| Price per token | To be decided by Ahmad during build phase |
| Distribution on purchase | 25% immediate to buyer wallet / dashboard; 75% auto-staked, daily claimable over 5 years |
| Referral system | Yes — buyers who refer others earn a commission. Structure to be designed during build phase. |
| Sale closes | When target raise amount is reached — no fixed date |

The private sale runs through a purpose-built private sale site with a smart contract on BNB Chain handling all distribution automatically. Buyers see their full allocation breakdown in the OTI private sale dashboard: immediate balance, staked balance, daily claimable amount, and accumulated unclaimed tokens.

### 4.2 Post-Listing Sales (As-Needed)

After exchange listing, additional tokens from the Private Sale bucket can be sold in as-needed rounds to fund specific development goals. Each post-listing sale:
- Uses tokens from the remaining Private Sale allocation
- Follows the same 25%/75% distribution model
- Is announced publicly with the date, amount, price, and terms

---

## 5. Revenue Distribution

Once OTI's paid API tiers generate real revenue, that revenue is split monthly — starting from day one after the private sale closes:

| Category | % | What it's for |
|---|---|---|
| Operating Costs | 40% | Hosting (Railway), paid data providers, tools, infrastructure — paid first |
| Revenue-Backed Rewards Pool (buyback) | 25% | Buys OTI on the open market and routes into the Rewards Pool for stakers — **only executes when the market is down**; held as cash reserve otherwise |
| Team | 20% | Cash payout to founders (Ahmad + Musty) — separate from their token allocation |
| Treasury/Reserve top-up | 15% | Cash reinvested into growth: marketing, audits, partnerships, legal |

**Why the buyback is conditional:** Buying tokens regardless of price would mean routinely overpaying when the market is already strong. Restricting buybacks to down-market conditions makes the Rewards Pool a genuine value-accrual mechanism rather than a fixed monthly expense.

---

## 6. Chain & Cross-Chain Plan

- **Launch chain:** BNB Smart Chain (BSC) — low fees, fast settlement, matches private sale mechanics.
- **Cross-chain expansion:** Planned after OTI token is established on BSC. Not yet scoped — will be addressed separately when Ahmad reopens it (likely via LayerZero or Wormhole, not a custom bridge).

---

## 7. Token Utility — What OTI Token Actually Does

OTI token must be a real utility token from creation day. It does not launch as an empty speculative asset.

### 7.1 Pay for Attestation
Users who want an OTI Verified Badge attestation can pay the attestation fee in OTI token. Token payment receives a discount versus BNB or fiat. Fee amount and discount rate are configured via admin panel — not hardcoded.

### 7.2 Staking (Revenue-Backed Rewards Pool)
Token holders who stake OTI receive rewards from the Revenue-Backed Rewards Pool. Rewards are funded by real API and attestation revenue — not by token inflation. Note: the 75% auto-staked private sale allocation is a separate distribution mechanic from voluntary staking. Staking design detail to be finalized once OTI has real usage and revenue.

### 7.3 Early Adopter Rewards — First 1 Million Wallets in OTI's Scoring Database
The first 1 million wallet addresses in OTI's scoring database receive OTI tokens — including wallets OTI has proactively pre-scored, regardless of whether the wallet owner has ever visited OTI. Token source: Ecosystem & Partnerships bucket. Amount per user: Ahmad to decide before Phase 3.

**Tracking requirement (critical):** The eligibility counter starts when proactive background scoring begins (Phase 2B Post-Campaign). This tracking list must be built into the background scorer from day one — it cannot be reconstructed after the fact.

### 7.4 Future Widget / API Access (Planned)
As the ecosystem matures, OTI token may gate or discount access to premium widget tiers and API plans. Documented here as the intended direction — not part of the immediate build.

---

## 8. Explicitly Out of Scope For Now

- **Price and liquidity pool design:** Ahmad decides during the build phase. Do not add, infer, or reconstruct pricing numbers.
- **Staking derivative system:** Deferred until OTI has real usage and revenue to build around.
- **Governance:** No DAO or voting mechanism defined for this phase.
- **Formal legal/securities review:** Ahmad's explicit decision — the raise is small enough not to require this at this stage. This was flagged once by the Manager; it is Ahmad's call.
