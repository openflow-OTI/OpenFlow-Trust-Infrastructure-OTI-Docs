---
name: Whitelist Dual Bonding Curve
description: Confirmed parameters for the OTI Ecosystem Whitelist dual-curve system — DCS (paid claims) and ERP (rewards), both confirmed July 29 2026.
---

## Dynamic Contribution Scale (DCS) — 7,000,000 OTI
- Target raise: $25,000
- Starting rate: $0.001190/OTI
- Ending rate: $0.005952/OTI
- Multiplier: 5× from start to end
- Formula: Rate(x) = 0.001190 + (0.005952 - 0.001190) × (x ÷ 7,000,000)
- Decisions: D40

## Ecosystem Rewards Pool (ERP) — 1,750,000 OTI
- Inverse curve: reward amounts decrease as DCS fills
- Formula: Current Reward = Base Reward × (DCS Remaining ÷ 7,000,000)
- Base rewards (admin-configurable, not hardcoded):
  - Referral (per confirmed claim): 3,000 OTI
  - Post + tag OTI: 1,000 OTI
  - Share whitelist link: 500 OTI
  - Follow Twitter/X: 500 OTI
  - Follow Telegram: 500 OTI
- Decisions: D41

## Key rules
- DCS and ERP are independent sub-pools — rewards do NOT advance the DCS curve
- Both sub-pools draw from Whitelist Allocation (8,750,000 OTI total = 25% of 35M supply)
- /whitelist page shows two live counters: DCS rate going UP, ERP referral bonus going DOWN
- All vesting params admin-configurable (75% Node Collateral Lockup, linear daily vesting)
- Social task verification: manual admin approval (no third-party social API)

**Why:** Ahmad's direction July 29 2026. Dual opposing curves create self-evident urgency without speculation language — visitors see contribution rate rising and reward amounts falling simultaneously.
