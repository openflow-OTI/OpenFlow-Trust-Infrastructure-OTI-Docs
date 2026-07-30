# OTI Economics
> **Document version:** July 29, 2026 — Full redesign following advisor session. Previous TOKENOMICS.md content is superseded entirely.
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
- All internal allocations locked
- Public access fuel distribution occurs **only** through the Ecosystem Whitelist Allocation

Genesis Mode continues until OpenFlow Labs determines the network has reached the operational maturity required for expanded governance and liquidity phases.

---

## Token Allocation

| Allocation | % | OTI | Status During Genesis |
|---|---|---|---|
| Ecosystem Whitelist | 25% | 8,750,000 | Active — distributed via whitelist program |
| Network Reserve | 20% | 7,000,000 | Locked |
| Founders | 15% | 5,250,000 | Locked — 5-year linear vesting |
| Strategic Partnerships | 10% | 3,500,000 | Locked — released as needed |
| Liquidity | 10% | 3,500,000 | Locked — released for liquidity operations only |
| Rewards Pool | 10% | 3,500,000 | Active — funded by revenue buybacks |
| Future Strategic Investment | 5% | 1,750,000 | Reserved |
| Operations Reserve | 5% | 1,750,000 | Locked |
| **Total** | **100%** | **35,000,000** | |

---

### Ecosystem Whitelist Allocation — 8,750,000 OTI (25%)

The Ecosystem Whitelist Allocation is the **only** allocation intended for public distribution during Genesis.

It covers the entire scope of the Ecosystem Whitelist Node Program:
- Invite-only onboarding (access fuel claims by whitelisted operators)
- Referral reward programs
- Community contributor recognition
- Early ecosystem participant rewards
- Social media and awareness campaign rewards
- Network testing participant rewards

All distribution from this allocation is subject to the 75% Node Collateral Lockup. When a whitelisted operator claims access fuel, 25% is immediately accessible and 75% enters a linear daily vesting schedule. The vesting parameters are configured by OpenFlow Labs via the admin dashboard and may be adjusted operationally.

No tokens from any other allocation may be distributed publicly during Genesis.

---

### Network Reserve — 7,000,000 OTI (20%)

Held by OpenFlow Labs to fund long-term network sustainability.

Covers:
- Infrastructure expansion (new chain integrations, scoring expansions)
- Ecosystem development initiatives
- Strategic technical partnerships
- Long-term operational continuity

Locked during Genesis. Released by OpenFlow Labs decision as specific requirements arise.

---

### Founders — 5,250,000 OTI (15%)

Commitment allocation for OpenFlow Labs founders.

- 100% locked at genesis.
- Five-year linear vesting.
- No cliff period.
- No early unlock mechanism.

The five-year lockup structure aligns founder incentives with long-term network health. Founders cannot exit ahead of the network they are building.

---

### Strategic Partnerships — 3,500,000 OTI (10%)

Reserved for infrastructure-level partnerships that require token-based alignment:
- Exchange and DEX partnerships
- Wallet provider integrations
- Enterprise platform integrations
- Infrastructure and tooling partnerships

Released only when a specific partnership requires it. Locked otherwise.

---

### Liquidity Allocation — 3,500,000 OTI (10%)

Reserved exclusively for liquidity operations:
- DEX liquidity pool seeding
- CEX listing requirements
- Market operations to support utility swap access

Not used for any purpose other than establishing and maintaining liquidity. Locked until liquidity operations begin.

---

### Rewards Pool — 3,500,000 OTI (10%)

Funds staking and ecosystem incentive programs.

The Rewards Pool is replenished through revenue-backed open-market purchases. OpenFlow Labs allocates a portion of platform revenue (see Revenue Distribution) to purchase OTI on the open market. Purchased tokens are transferred into the Rewards Pool.

This means staking rewards and incentive programs are funded by real platform revenue — not by inflation and not by minting new tokens. The total supply cap is never breached.

---

### Future Strategic Investment — 1,750,000 OTI (5%)

Reserved for future fundraising rounds with strategic investors.

Not distributed during Genesis. Reserved only.

---

### Operations Reserve — 1,750,000 OTI (5%)

Operational contingency reserve.

- Infrastructure emergencies
- Unplanned operational costs
- Business continuity requirements

Locked. Released only under OpenFlow Labs decision for defined operational needs.

---

## Staking

Genesis uses one staking system.

**What is staked:** Founder allocation and internal locked allocations are subject to the staking lockup schedule described above.

**What public participants receive:** Whitelisted operators receive access fuel according to the 25%/75% vesting structure configured by the platform. The 75% Node Collateral Lockup is a network stability mechanism, not an investment product. It protects circulating supply from systemic dumping during early network phases.

**Governance staking:** Not available during Genesis. No governance exists during Genesis.

---

## Governance

Genesis includes no governance layer.

OTI is managed operationally by OpenFlow Labs during the Genesis phase. OpenFlow Labs makes all decisions regarding:
- Vesting parameter configuration
- Reserve allocation decisions
- Liquidity timing
- Partnership token releases
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

## Core Principles

1. **Fixed supply.** 35,000,000 OTI. No exceptions.
2. **No inflation.** Rewards are funded by revenue buybacks, not new minting.
3. **No governance during Genesis.** OpenFlow Labs operates the network directly.
4. **One staking system.** Founder lockup + internal allocation lockup only. No public speculation layer.
5. **Revenue-backed ecosystem.** Rewards Pool is funded by real revenue, not by printing tokens.
6. **Utility-driven demand.** OTI demand grows as network usage grows — attestations, API calls, subscriptions.
7. **Infrastructure-first.** Economic model designed for long-term sustainability, not short-term speculation.
8. **Long-term alignment.** Founders locked five years. Internal allocations locked during Genesis. All incentives point toward building.

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

**Vesting:** 25% of claimed OTI is immediately accessible as Access Fuel. 75% enters Node Collateral Lockup — linear daily vesting schedule. All vesting parameters are admin-configurable via the OpenFlow Labs dashboard. Nothing is hardcoded.

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

**Reward vesting:** All reward OTI is subject to the same 75% Node Collateral Lockup as direct claims. 25% immediately accessible, 75% vests daily on the same schedule.

**Social task verification:** Users submit their post URL or username on /whitelist. OpenFlow Labs reviews and approves via admin dashboard. Approval triggers the OTI issuance. No third-party social API integration required.

**Referral tracking:** Off-chain in the `whitelist_invites` DB table. The referrer's wallet is linked to each invite code used. Reward issued automatically on claim confirmation.

---

### The Dual-Curve Display (on /whitelist page)

The /whitelist page shows two live counters moving simultaneously:
- **Contribution Rate** (DCS): current rate in $/OTI → going UP as claims fill
- **Referral Bonus** (ERP): current OTI per referral → going DOWN as claims fill

Both driven by a single variable: DCS tokens remaining. No static numbers. Every visitor sees live urgency without any speculative language.

---

### Milestone Triggers

These are operational commitments. They describe infrastructure OpenFlow Labs deploys as the ecosystem commits allocation — not investment return promises.

- **Milestone 2 — Phase 1 Liquidity Seeding:** At $5,000 committed allocation → Public Utility Liquidity Layer deployed on decentralized protocols. OTI becomes redeemable via utility swap.
- **Milestone 3 — Deep Liquidity Scaling:** At $15,000 committed allocation → Secondary AMM pool funded. Utility swap rates stabilised for B2B clients.

---

### Access Controls

- Access gated by single-use invite codes (format: OTI-XXXX-XXXX, admin-generated in batches)
- 10,000 maximum whitelisted operator slots over 12 months
- Geographic restrictions enforced (US, China, sanctioned jurisdictions blocked)
- Ban system: admin can freeze any code or wallet address immediately
- All parameters (base rewards, vesting %, milestones) configurable via OpenFlow Labs admin dashboard

---

## What This Document Is Not

This document describes the operational and technical economics of the OTI utility token. It is not:
- An investment prospectus
- A securities offering document
- A guarantee of future value or returns
- A profit-sharing arrangement
- A financial product of any kind

OTI tokens are software access credentials. All token-related activity is governed by the OTI Terms & Conditions and Privacy Policy.
