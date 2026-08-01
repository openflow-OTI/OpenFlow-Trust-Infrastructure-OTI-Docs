# OTI Economics
> **Document version:** August 1, 2026 — Updated following strategic design session. Supersedes July 29, 2026 version.
> **Regulatory framing:** OTI is a utility access token. All language in this document and in any public content derived from it must reflect utility-first framing. No investment language, no return expectations, no yield framing. See DECISIONS.md for the vocabulary enforcement rule.

---

## What OTI Is

OTI (OpenFlow Trust Infrastructure) is a utility access token. It is the unit of account for interacting with the OpenFlow Trust Infrastructure network — paying for wallet attestations, API subscriptions, analytics access, and infrastructure services.

OTI does not represent equity, ownership, profit rights, revenue rights, or any claim against OpenFlow Labs. It is a software access credential in token form. Its value is determined entirely by demand for the services it unlocks.

---

## Total Supply

**35,000,000 OTI**

- Fixed supply.
- No inflation mechanism.
- No future minting authority.
- No additional token creation under any condition.

The 35,000,000 cap is final at genesis and cannot be altered.

---

## Genesis Mode

Genesis Mode is the initial operational phase of the OTI network. During Genesis, OpenFlow Labs operates the network directly, distributes access fuel through the Ecosystem Whitelist Allocation, builds commercial adoption, and prepares the infrastructure for long-term expansion.

Genesis Mode characteristics:
- One staking system (Founder + internal allocation lockup only)
- No governance layer
- Fixed token supply in force
- All internal allocations locked in smart contracts
- Public access fuel distribution occurs **only** through the Ecosystem Whitelist Allocation

Genesis Mode continues until OpenFlow Labs determines the network has reached the operational maturity required for expanded governance and liquidity phases.

---

## Token Allocation

| Allocation | % | OTI | Contract | Status During Genesis |
|---|---|---|---|---|
| Ecosystem Whitelist | 25% | 8,750,000 | Whitelist Contract | Active — distributed via whitelist program |
| Network Reserve | 20% | 7,000,000 | Allocation Manager | Locked — unassigned until needed |
| Founders | 15% | 5,250,000 | Team Contract | Locked — 5-year linear daily vesting |
| Liquidity | 10% | 3,500,000 | Allocation Manager | Locked — deployed to DEX pool operations |
| Rewards Pool | 10% | 3,500,000 | Allocation Manager | Active — funded by revenue buybacks |
| Strategic Partnerships | 5% | 1,750,000 | Allocation Manager | Locked — released per agreement |
| Marketing & Growth | 5% | 1,750,000 | Allocation Manager | Locked — released per agreement |
| Future Strategic Investment | 5% | 1,750,000 | Allocation Manager | Reserved — unassigned |
| Operations Reserve | 5% | 1,750,000 | Allocation Manager | Locked — released for operational needs |
| **Total** | **100%** | **35,000,000** | | |

---

## Genesis Contract Architecture

At genesis, all 35,000,000 OTI are minted and immediately distributed across three smart contracts. Zero tokens are held in any personal wallet at launch. All locked supply is verifiable on-chain.

### Contract 1 — Whitelist Contract (8,750,000 OTI)

Holds the entire Ecosystem Whitelist Allocation. Manages the DCS bonding curve, ERP reward issuance, and all whitelist participant vesting. Admin-configurable vesting parameters. See Whitelist Program Economics for full detail.

### Contract 2 — Team Contract (5,250,000 OTI)

Holds the Founders allocation exclusively. Fixed 5-year linear daily vesting from genesis date. No early unlock mechanism. No admin override. Founders claim daily unlocked portions via the private `/team` portal by connecting their wallet.

The five-year lockup is non-negotiable and cannot be changed by admin after deployment. This is a permanent on-chain commitment signal.

### Contract 3 — Allocation Manager (21,000,000 OTI)

Holds all remaining allocations: Network Reserve, Liquidity, Rewards Pool, Strategic Partnerships, Marketing & Growth, Future Strategic Investment, and Operations Reserve.

**Key design principles:**

- All tokens in the Allocation Manager are **not in circulating supply** until assigned. They exist on-chain but are held by the contract, not any external wallet. Circulating supply trackers (CoinMarketCap, CoinGecko, DexScreener) reflect only tokens held in externally-owned wallets — unassigned tokens in this contract are excluded.
- OpenFlow Labs (admin wallet, secured by multi-sig) is the only authority that can create assignments.
- Every assignment is a public on-chain transaction. OpenFlow Labs publishes the corresponding agreement document alongside the transaction hash for full transparency.
- Unassigned tokens remain permanently locked in the contract until OpenFlow Labs creates an assignment. They do not vest, do not move, and cannot be accessed by anyone.

**Per-assignment configuration (admin sets per deal):**

| Parameter | Options |
|---|---|
| Recipient wallet address | Any valid BEP-20 address |
| Total OTI amount | Any amount within unassigned balance |
| Immediate release % | 0% to 100% |
| Vesting duration | 0 days to any number of days |
| Start date | Assignment date (default) or future date |

Setting 0% immediate + 0 days vesting is equivalent to a direct send. Setting 100% immediate is equivalent to an instant grant. All combinations in between are valid. Recipients claim their daily vested portion via the private `/marketing` or `/partners` portal.

**Admin dashboard for each assignment shows:**
- Recipient wallet
- Total OTI assigned
- Agreement reference name
- On-chain transaction hash (links to BscScan)
- Amount claimed so far / amount remaining
- Daily claim rate
- Printable PDF receipt (includes all above fields + assignment terms)

---

## Allocation Manager — Allocation Details

### Network Reserve — 7,000,000 OTI (20%)

Held for long-term network sustainability. Released by OpenFlow Labs decision only when a specific requirement arises.

Covers:
- Infrastructure expansion (new chain integrations, scoring expansions)
- Ecosystem development initiatives
- Strategic technical partnerships
- Long-term operational continuity

### Liquidity Allocation — 3,500,000 OTI (10%)

Reserved exclusively for DEX and CEX liquidity operations.

Primary use: seeding the OTI/USDT (or OTI/BNB) liquidity pool on PancakeSwap. When OpenFlow Labs creates the initial DEX pool, Liquidity allocation tokens are assigned from the Allocation Manager to the pool creation transaction. LP tokens from the initial seed are locked permanently (burned to a dead address) — the liquidity cannot be removed.

Ongoing use: additional liquidity depth additions as the network grows.

### Rewards Pool — 3,500,000 OTI (10%)

Funds staking and ecosystem incentive programs. Replenished through revenue-backed open-market purchases — 15% of platform revenue is used to buy OTI on the open market and transfer it into the Rewards Pool. No new tokens are minted. Total supply cap is never breached.

### Strategic Partnerships — 1,750,000 OTI (5%)

Reserved for infrastructure-level partnerships requiring token-based alignment:
- Exchange and DEX partnerships
- Wallet provider integrations
- Enterprise platform integrations
- Infrastructure and tooling partnerships

Released only when a specific partnership agreement is executed. Each release is configured via the Allocation Manager with custom vesting terms per deal.

### Marketing & Growth — 1,750,000 OTI (5%)

Reserved for marketing partners, influencers, ambassadors, and growth programs.

Released only when a specific marketing agreement is executed. Recipients access their allocation via the private `/marketing` portal. All Marketing & Growth assignments use 3-year vesting (1,095 days) starting from the assignment date, with daily linear claiming. The immediate release % per deal is set by OpenFlow Labs — can be 0% for pure vesting or a small % for upfront confirmation of the agreement.

If any portion of this allocation is unused over the long term, it remains locked in the Allocation Manager. OpenFlow Labs may reassign unused amounts to the Network Reserve by admin decision, with a public on-chain record.

### Future Strategic Investment — 1,750,000 OTI (5%)

Reserved for future fundraising rounds with strategic investors. Not assigned during Genesis. Reserved only.

### Operations Reserve — 1,750,000 OTI (5%)

Operational contingency reserve for infrastructure emergencies, unplanned operational costs, and business continuity requirements. Released by OpenFlow Labs decision only for defined operational needs.

---

## DEX Liquidity Engine

The DEX Liquidity Engine is the automated system that maintains OTI utility swap access and price stability on the DEX. It operates from the first whitelist contribution and runs continuously.

### Liquidity Auto-Seeding (30% rule)

When a whitelist participant contributes funds, the contribution is automatically split:

- **30% → DEX liquidity pool** (adds depth to the OTI/USDT pool on PancakeSwap automatically)
- **70% → Committed Funds Reserve** (held in the whitelist contract for operations and buybacks)

This means the DEX pool deepens with every single whitelist contribution from day one. No phase triggers. No manual action by OpenFlow Labs required.

The initial DEX pool is created by OpenFlow Labs at or before the first whitelist buy, using a small seed of Liquidity allocation tokens paired with minimal BNB/USDT to establish the starting price reference. This is a one-time manual action. All subsequent depth comes automatically from the 30% rule.

### DEX Target Multiplier (DCS-TM)

**Default: 1.5×**

The DCS-TM defines the target relationship between the DEX spot price and the current DCS contribution rate:

```
Target DEX Price = Current DCS Rate × DCS-TM
```

Example: if the DCS rate has reached $0.002, the system targets a DEX price of $0.003.

The DCS-TM is admin-configurable. OpenFlow Labs can raise or lower it at any time based on market conditions.

### Auto-Buyback Engine

The auto-buyback engine monitors the DEX price hourly. When the DEX price falls below the target (DCS Rate × DCS-TM), it automatically uses funds from the Buyback Reserve to buy OTI on the DEX, pushing the price back toward the target.

**Buyback Reserve:** OpenFlow Labs sets the buyback reserve percentage in the admin dashboard — this is the portion of Committed Funds that can be used for buybacks. Example: 20% reserve means if $10,000 has been committed, up to $2,000 is available for buybacks.

The OTI purchased via buybacks is held by the contract — it is not destroyed. It can be recycled into the Rewards Pool or future liquidity operations by admin decision.

**Circuit breaker:** If the Buyback Reserve drops below the circuit breaker threshold, the auto-buyback engine pauses automatically and OpenFlow Labs receives an admin alert. The DEX price finds its natural level until the reserve is replenished by new whitelist contributions. This prevents the system from depleting the operational reserve trying to defend a price the market cannot currently support.

The circuit breaker threshold is admin-configurable (default: 5% of total Committed Funds). OpenFlow Labs can raise or lower this threshold at any time via the admin dashboard — for example, setting it higher during slow periods for more conservative protection, or lower during strong whitelist growth when reserve risk is minimal.

### Admin Dashboard — DEX & Liquidity Stats

The admin dashboard whitelist section displays:

| Metric | Description |
|---|---|
| Total committed (USDT/BNB) | All funds received from whitelist contributions |
| DEX pool depth | Current total value locked in the OTI liquidity pool |
| Current DCS rate | Live position on the bonding curve |
| Current DEX spot price | Live price from PancakeSwap |
| DCS-TM gap | DEX price vs target (green = above target, red = below) |
| Buyback reserve balance | Funds available for auto-buyback |
| Buyback history | Table: timestamp, amount spent, OTI bought, price before/after |
| Auto-buyback status | Active / Paused (circuit breaker state) |

---

## Whitelist Program Economics

The Ecosystem Whitelist Allocation (8,750,000 OTI, 25%) is split into two sub-pools that operate simultaneously:

| Sub-Pool | OTI | Mechanism |
|---|---|---|
| Dynamic Contribution Scale (DCS) | 7,000,000 | Bonding curve — paid claims. Raises $25,000 total. |
| Ecosystem Rewards Pool (ERP) | 1,750,000 | Variable rewards — referrals, social tasks. Rate tied to DCS progress. |
| **Total** | **8,750,000** | |

---

### Sub-Pool 1 — Dynamic Contribution Scale (7,000,000 OTI)

A linear bonding curve. The Contribution Rate starts low and increases continuously as tokens are claimed. Early operators contribute less per unit of Access Fuel — rewarding early network participation.

**Confirmed parameters (July 29, 2026):**
- Total target raise: **$25,000**
- Total tokens through DCS: **7,000,000 OTI**
- Starting Contribution Rate: **$0.001190 per OTI** (founding operator tier)
- Ending Contribution Rate: **$0.005952 per OTI** (final tier)
- Multiplier: **5× from start to end**
- Formula: `Rate(x) = P₀ + (P₁ - P₀) × (x ÷ 7,000,000)` where x = tokens claimed so far

The /whitelist page displays the current rate live, updating in real time as claims are processed. No static price is shown — only the current position on the scale.

**Dynamic Unlock Formula:**

The immediate unlock percentage is not fixed — it scales inversely with DCS progress. Early operators (joining when DCS is empty) receive the maximum immediate unlock. Late operators (joining when DCS is nearly full) receive the minimum.

```
Immediate_Unlock% = max(5%, 25% - (DCS_Progress% × 20%))
```

| DCS Progress | Immediate Unlock | Locked (vesting) |
|---|---|---|
| 0% claimed | 25% | 75% |
| 25% claimed | 20% | 80% |
| 50% claimed | 15% | 85% |
| 75% claimed | 10% | 90% |
| 100% claimed | 5% | 95% |

**Why this works for participants:** Although late buyers receive less immediately, their locked portion is larger — meaning their daily vesting earnings are higher. For the same total OTI received:
- Early buyer (10,000 OTI, 90-day vesting): 2,500 immediate + 7,500 ÷ 90 = **83.3 OTI/day**
- Late buyer (10,000 OTI, 90-day vesting): 500 immediate + 9,500 ÷ 90 = **105.6 OTI/day**

Early adopters are rewarded with the larger upfront amount. Late adopters are rewarded with higher daily income.

**Vesting duration:** Admin-configurable in days via the OpenFlow Labs dashboard. Duration is set per period — when OpenFlow Labs changes the vesting duration, it applies to all future buyers from that point forward. Existing buyers' vesting duration locks in at the time of their claim and is never retroactively changed.

---

### Sub-Pool 2 — Ecosystem Rewards Pool (1,750,000 OTI)

Reward amounts for referrals and social tasks are **not fixed** — they scale inversely with DCS progress. As the DCS fills, reward amounts decrease. This creates the same early-action urgency on the engagement side as the rising contribution rate creates on the claims side.

**Formula:**
```
Reward Multiplier = DCS Remaining ÷ 7,000,000
Current Reward = Base Reward × Reward Multiplier
```

**Base reward amounts (admin-configurable):**
| Action | Base OTI (at 0% DCS claimed) |
|---|---|
| Successful referral (per confirmed whitelist claim) | 3,000 OTI |
| Public post + tag OTI | 1,000 OTI |
| Share whitelist link publicly | 500 OTI |
| Follow OTI on Twitter/X | 500 OTI |
| Follow OTI on Telegram | 500 OTI |

**Example decay:**

| DCS Progress | Referral Bonus | Post/Tag | Follow |
|---|---|---|---|
| 0% claimed | 3,000 OTI | 1,000 OTI | 500 OTI |
| 25% claimed | 2,250 OTI | 750 OTI | 375 OTI |
| 50% claimed | 1,500 OTI | 500 OTI | 250 OTI |
| 75% claimed | 750 OTI | 250 OTI | 125 OTI |
| 90% claimed | 300 OTI | 100 OTI | 50 OTI |

**ERP Reward Vesting:** All ERP reward OTI is locked for **3 years (1,095 days)** from the date the reward is granted. Linear daily vesting — recipients claim each day's unlocked portion. No immediate unlock for ERP rewards. The 3-year clock starts from the grant date, not from the date of the action that earned it.

**Social task verification:** Users submit their post URL or username on /whitelist. OpenFlow Labs reviews and approves via admin dashboard. Approval triggers the OTI issuance and starts the 3-year vesting clock.

**Referral tracking:** Off-chain in the `whitelist_invites` DB table. The referrer's wallet is linked to each invite code used. Reward issued automatically on claim confirmation.

---

### The Dual-Curve Display (on /whitelist page)

The /whitelist page shows two live counters moving simultaneously:
- **Contribution Rate** (DCS): current rate in $/OTI → going UP as claims fill
- **Referral Bonus** (ERP): current OTI per referral → going DOWN as claims fill

Both driven by a single variable: DCS tokens remaining. No static numbers. Every visitor sees live urgency without any speculative language.

---

### What Happens After the Whitelist Closes

When the DCS reaches 7,000,000 OTI claimed, the whitelist stops accepting new participants. The /whitelist page updates to reflect this. However:

- **Existing participants continue claiming normally.** The vesting contract keeps running indefinitely. Anyone with locked OTI continues visiting the platform and claiming their daily unlock for as long as their vesting runs.
- **ERP vesting continues.** All social reward lockups continue their 3-year vesting regardless of whitelist status.
- **The DEX liquidity engine continues.** Auto-buyback and liquidity operations continue from the Committed Funds Reserve.

The whitelist close is an entry gate closing — not a shutdown. All existing obligations (vesting, claiming, DEX support) run to completion.

---

### Access Controls

- Access gated by single-use invite codes (format: OTI-XXXX-XXXX, admin-generated in batches)
- 10,000 maximum whitelisted operator slots over 12 months
- Geographic restrictions enforced (US, China, sanctioned jurisdictions blocked)
- Ban system: admin can freeze any code or wallet address immediately
- All parameters (base rewards, vesting duration in days, unlock %, DCS-TM, buyback reserve %) configurable via OpenFlow Labs admin dashboard

---

## Staking

Genesis uses one staking system.

**What is staked:** Founder allocation and internal locked allocations are subject to the staking lockup schedule described above.

**What public participants receive:** Whitelisted operators receive access fuel according to the dynamic unlock formula configured by the platform. The Node Collateral Lockup is a network stability mechanism, not an investment product. It protects circulating supply from systemic dumping during early network phases.

**Governance staking:** Not available during Genesis. No governance exists during Genesis.

---

## Governance

Genesis includes no governance layer.

OTI is managed operationally by OpenFlow Labs during the Genesis phase. OpenFlow Labs makes all decisions regarding:
- Vesting parameter configuration
- Reserve allocation decisions
- Liquidity and buyback operations
- Partnership and marketing token releases
- Revenue distribution ratios

Governance mechanisms are a future-phase consideration, not a Genesis feature.

---

## Platform Revenue Sources

OTI generates platform revenue from:

| Revenue Stream | Description |
|---|---|
| Wallet Attestations | Per-attestation fees (BNB Chain BAS) |
| API Subscriptions | Developer monthly/annual API plan subscriptions |
| Enterprise API | High-volume enterprise API contracts |
| Widget Subscriptions | Widget commercial license subscriptions |
| Premium Analytics | Score history, analytics, and reporting access |
| Webhook Subscriptions | Real-time wallet alert subscriptions |
| Future Infrastructure Services | New products and services as OTI expands |

Revenue is generated in fiat-equivalent or BNB, not OTI. Revenue feeds back into the ecosystem through the Revenue Distribution model below.

---

## Revenue Distribution

All platform revenue is distributed across five operational categories. These are internal operational allocations — they represent how OpenFlow Labs manages the business, not distributions to token holders.

| Category | % | Purpose |
|---|---|---|
| Operations | 35% | Infrastructure, servers, development costs, customer support, business operations |
| Network Reserve | 25% | Long-term sustainability fund, future expansion, infrastructure scaling |
| Rewards Pool | 15% | Open-market OTI purchases — transferred to Rewards Pool, no new tokens minted |
| Team Operations | 20% | Founder and team salaries, operational workforce, long-term development continuity |
| Research & Development | 5% | New infrastructure, security improvements, product research, future protocol development |

**The Rewards Pool allocation (15%) is the mechanism that connects platform revenue to the OTI ecosystem.** Rather than minting new tokens, OpenFlow Labs uses 15% of revenue to buy OTI from the open market and transfer it to the Rewards Pool. This creates utility-backed demand and sustains staking/incentive programs without inflation.

---

## OTI Utility

OTI unlocks access to OTI network services:

| Use Case | Description |
|---|---|
| Wallet Attestations | Pay attestation fees (with discount vs. non-OTI payment methods) |
| API Subscriptions | Pay for developer API plan access |
| Enterprise API | Pay for enterprise API contracts |
| Widget Subscriptions | Pay for commercial widget embedding rights |
| Premium Analytics | Unlock score history, wallet analytics, reporting dashboards |
| Webhook Subscriptions | Pay for real-time wallet alert webhooks |
| Developer Staking | Stake OTI for priority API access and reduced fees |
| Ecosystem Incentives | Earn OTI through early-adopter rewards, referral programs, contributor programs |
| Partner Revenue Share | Partners earn OTI commission on attestation conversions through the embedded widget |
| Future Infrastructure Services | Access new OTI network products as they launch |

OTI has no utility outside the OTI network and its integrations. It is not a general-purpose currency.

---

## Circulating Supply At Key Moments

| Moment | Circulating OTI | % of Total |
|---|---|---|
| Genesis day | ~0 OTI | ~0% |
| First whitelist buy | Immediate unlock of first claim only | <0.1% |
| Whitelist 50% filled | DCS immediate unlocks only (~avg 17.5%) | ~0.5% |
| Whitelist closes (full) | All DCS immediate unlocks (~avg 15%) | ~3% |
| 1 year after close | DCS vesting + ERP partial unlock | ~8-12% |
| 3 years after close | Most DCS vested, ERP fully vested | ~30-35% |
| 5 years after genesis | Founders fully vested, DCS/ERP complete | ~40-45% |

The remainder (55-60% at 5 years) is still locked in Allocation Manager assignments not yet vested or not yet assigned.

---

## Core Principles

1. **Fixed supply.** 35,000,000 OTI. No exceptions.
2. **No inflation.** Rewards are funded by revenue buybacks, not new minting.
3. **No governance during Genesis.** OpenFlow Labs operates the network directly.
4. **On-chain lock verification.** All allocations locked in smart contracts from genesis. Zero tokens in personal wallets at launch.
5. **Revenue-backed ecosystem.** Rewards Pool is funded by real revenue, not by printing tokens.
6. **Utility-driven demand.** OTI demand grows as network usage grows — attestations, API calls, subscriptions.
7. **Infrastructure-first.** Economic model designed for long-term sustainability, not short-term speculation.
8. **Long-term alignment.** Founders locked five years. Marketing partners locked three years. All incentives point toward building.
9. **Transparent by design.** Every allocation assignment is a public on-chain transaction with a published agreement record.

---

## What This Document Is Not

This document describes the operational and technical economics of the OTI utility token. It is not:
- An investment prospectus
- A securities offering document
- A guarantee of future value or returns
- A profit-sharing arrangement
- A financial product of any kind

OTI tokens are software access credentials. All token-related activity is governed by the OTI Terms & Conditions and Privacy Policy.
