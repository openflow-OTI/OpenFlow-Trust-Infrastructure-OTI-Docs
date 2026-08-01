---
name: OTI DEX Liquidity Engine & Genesis Architecture
description: Key decisions confirmed August 1 2026 — three-contract genesis, DEX auto-seeding, buyback engine, dynamic unlock, allocation manager design.
---

## Three-Contract Genesis Architecture

All 35M OTI minted at genesis and split immediately:
- **Whitelist Contract:** 8,750,000 OTI (DCS + ERP)
- **Team Contract:** 5,250,000 OTI — fixed 5-year linear daily vesting, NO admin override after deployment
- **Allocation Manager:** 21,000,000 OTI — all other allocations, full admin flexibility

Zero tokens in personal wallets at genesis. All locked supply verifiable on-chain.

## Allocation Split Change (August 1 2026)
Strategic Partnerships (10%, 3.5M) split into:
- Strategic Partnerships: 5%, 1,750,000 OTI
- Marketing & Growth: 5%, 1,750,000 OTI

## Allocation Manager Design
- Per-assignment config: recipient wallet, amount, immediate release % (0-100%), vesting duration in days (0 = instant)
- Unassigned tokens = NOT in circulating supply
- Every assignment is a public on-chain transaction; OpenFlow Labs publishes agreement alongside tx hash
- Admin dashboard: assignment table, PDF receipt per assignment, BscScan links
- Multi-sig admin wallet required (NOT a single EOA)

## DEX Liquidity Engine
- **30% rule:** 30% of every whitelist contribution auto-routes to DEX liquidity pool; 70% stays as committed funds
- **DCS-TM default: 1.5×** — target DEX price = current DCS rate × 1.5 (admin-configurable)
- **Auto-buyback:** hourly check; if DEX < target, buy OTI from committed funds
- **Circuit breaker:** auto-buyback pauses + admin alert when buyback reserve < 5% of committed funds
- **Buyback reserve %:** admin-configurable (e.g. 20% of committed funds available for buyback)
- No liquidity phases or milestone triggers — starts from first whitelist buy

## Dynamic Unlock Formula
`Immediate_Unlock% = max(5%, 25% - (DCS_Progress% × 20%))`
- 0% DCS: 25% immediate / 75% locked
- 100% DCS: 5% immediate / 95% locked
- Late buyers get more per day (larger locked pool) even though less upfront

## Vesting Rules
- DCS vesting duration: admin-configurable in days; changes apply to future buyers only, never retroactively
- ERP (social rewards): 3-year lock (1,095 days) from grant date, no immediate unlock
- Marketing & Growth assignments: 3-year default, daily linear, admin sets immediate % per deal
- Founders (Team Contract): 5-year fixed daily vesting, non-overridable

## Why: Key Principles
- Multi-sig is non-negotiable before mainnet — single admin wallet over 3 contracts is too risky
- Circuit breaker prevents buyback reserve drain during slow whitelist periods
- On-chain transparency for all assignments is a core credibility mechanism
