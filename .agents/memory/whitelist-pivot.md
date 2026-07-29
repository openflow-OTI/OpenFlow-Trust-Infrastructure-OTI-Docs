---
name: Whitelist Pivot
description: OTI switched from "private sale" to "Ecosystem Whitelist Node Program" — regulatory compliance decision. All framing, vocabulary, and legal structure changed.
---

## The Rule
Never use "presale", "private sale", "token sale", "ICO", "invest", "ROI", "yield", or "investors" in any public-facing content, task prompt, or Builder instruction. Use whitelist vocabulary exclusively.

## Vocabulary Map
- Token Sale → Ecosystem Whitelist / Node Testing Program
- Buy Tokens → Acquire Network Access Fuel / Claim Allocation
- Staking Payouts / Yield → Node Collateral Lockup / Linear Network Vesting
- Investors → Whitelisted Operators / Community Contributors
- Trading / Listing → Public Utility Liquidity Pool Seeding

**Why:** Running an open public token checkout while marketing on social media triggers "Unregistered Public Offering" under local regulatory frameworks (SEC Nigeria). The whitelist framing converts the launch into a private technical ecosystem onboarding program.

## Key Parameters
- Total whitelist allocation: 25% of total supply (covers invites, social rewards, community rewards — everything)
- All vesting/lockup percentages: admin-configurable via dashboard, NEVER hardcoded
- Off-chain referral tracking (in DB, not smart contract) — admin has full control
- Commissions paid in OTI token, not BNB/USDT
- Invite code format: OTI-XXXX-XXXX, admin-generated in batches
- Target: 10,000 unique slots over 12 months
- /whitelist on existing Vercel project (not separate domain)
