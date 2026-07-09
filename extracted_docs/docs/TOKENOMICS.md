# OTI Token — Tokenomics, Sale, Liquidity & Revenue Distribution

> Last updated: July 9, 2026 (session 6) | Maintained by: Development Manager
> **Status: Ahmad-approved design, locked.** This is OTI's own independent token — it is NOT the OpenFlow "FLOW" ecosystem token discussed in earlier planning. OTI can live with or without OpenFlow, so it has its own token, its own supply, and its own economics. Launch chain: BNB Smart Chain (BSC) first, with cross-chain expansion planned later.
> Regulatory note: Ahmad has explicitly decided this raise is small enough not to require formal securities/regulatory review at this stage. This decision was flagged to him once by the Manager and is his call to make.

---

## 1. Total Supply

**30,000,000 OTI — fixed supply, no inflation, no mint function after launch.**

Fixed supply is a deliberate trust signal: no bucket can be silently expanded, and total dilution is knowable from day one.

---

## 2. Allocation

| Bucket | % | Tokens | Purpose |
|---|---|---|---|
| Sale (Pre-Listing + Future Discount Sales) | 15% | 4,500,000 | Funds building OTI; sold in one pre-listing round plus later as-needed discounted rounds |
| Team & Founders | 15% | 4,500,000 | Long-term founder ownership (Ahmad + Musty) |
| Ecosystem & Partnerships | 15% | 4,500,000 | Developer grants, integration incentives, protocol/exchange partnerships that drive real API usage |
| Revenue-Backed Rewards Pool | 30% | 9,000,000 | Staking/holder rewards — funded by real OTI API revenue via buyback, not token inflation |
| Liquidity & Market Making | 10% | 3,000,000 | DEX trading pool depth, exchange listing support |
| Treasury / Reserve | 15% | 4,500,000 | Operating costs, security audits, contingency — multisig-controlled |
| **Total** | **100%** | **30,000,000** | |

---

## 3. Vesting

**Universal rule:** every bucket except Liquidity vests linearly over **36 months**, releasing 1/36th each month, starting from that bucket's (or that specific sale's) own release date. No cliff, no lump-sum unlock — a steady monthly drip for 3 years.

- **Liquidity & Market Making — the one exception.** Fully unlocked at listing, with no vesting. Liquidity has to be tradeable from day one; vesting it would leave the token illiquid at launch.
- **Sale round tokens** each get their own independent 36-month clock starting from their own purchase date — the pre-listing round starts at listing; each later discount sale starts its own clock from when that sale happens.
- **Team, Ecosystem, Rewards Pool, Treasury** — all 36-month monthly linear from their respective start dates.
- **Revenue-Backed Rewards Pool** has a second gate on top of vesting: even as tokens vest into availability, actual payout to stakers still depends on real API revenue funding the pool (see Section 6). Vesting caps the *maximum* release rate; revenue availability governs whether a payout actually happens that month.

---

## 4. Token Sale Structure

### 4.1 Pre-Listing Sale Round (the only round before launch)

| | |
|---|---|
| Target raise | $10,000 |
| Price | $0.02 per OTI |
| Tokens sold | 500,000 OTI (≈1.7% of total supply) |
| Vesting | 36-month monthly linear, from listing date |
| Accepted currency | BNB / USDT on BSC |

This is a deliberately small, single public round — there is no separate private/KYC'd round, since Ahmad does not currently have private investors lined up. All buyers go through the same public terms.

### 4.2 Post-Listing Discount Sales (as-needed, not scheduled)

These are **not** a second fixed round. They happen only when OTI genuinely needs additional capital, at any point after listing:

- **Price:** set at a discount to the live market price at the time of that sale (typically 20–30% below market — exact discount decided per sale, since market price varies)
- **Source:** drawn from the ≈4,000,000 OTI remaining in the Sale bucket after the pre-listing round
- **Vesting:** 36-month monthly linear, starting from that specific sale's own purchase date
- **Transparency:** every future discount sale is logged publicly with its date, amount raised, price, and vesting terms — no ambiguity about how much of the Sale bucket has been used vs. still reserved

**Why this shape:** committing to only one round now, and treating every later raise as need-based rather than calendar-based, avoids selling supply the business doesn't yet need — and the discount-for-lockup exchange means every future raise adds capital without creating instant sell pressure on the newly raised tokens.

---

## 5. Initial Liquidity & Liquidity-to-Market-Cap Ratio

**Why this ratio matters:** a token's initial DEX liquidity pool determines how much can be bought or sold before the price swings sharply (slippage). A common industry rule of thumb is to target liquidity pool value at roughly **10–20% of fully diluted valuation (FDV)** for a reasonably tradeable token. Below that, even small trades cause large price swings; that erodes early buyer trust.

**FDV at listing price:** 30,000,000 OTI × $0.02 = **$600,000 FDV**.

**The honest constraint:** the pre-listing round only raises $10,000 total. Fully pairing the entire 3,000,000-token Liquidity bucket at $0.02/token would require $60,000 of matching BNB/USDT — far more capital than this raise produces. Rather than promise a healthy ratio the business cannot yet afford, the plan below is scaled to what's actually raised, with an explicit path to grow it over time.

### 5.1 Initial Launch Pool (day one)

- $8,000 of the $10,000 raised is allocated to seed liquidity (the remaining $2,000 goes to immediate operating costs — see Section 6).
- $8,000 in BNB/USDT is paired with an equal $8,000 worth of OTI from the Liquidity bucket → 400,000 OTI (at $0.02/token).
- **Initial pool size: ~$16,000 total** (400,000 OTI + $8,000 BNB/USDT).
- **Initial liquidity-to-FDV ratio: ~2.7%** ($16,000 / $600,000) — well below the healthy 10–20% target. This is expected and disclosed upfront, not hidden: a $10k raise cannot bootstrap deep liquidity on day one, and pretending otherwise would be misleading to early buyers.
- The remaining 2,600,000 OTI in the Liquidity bucket stays reserved, unpaired, ready to be added to the pool as capital becomes available.

### 5.2 Growing the Pool Toward the Healthy Target

A fixed share of future capital is directed at deepening liquidity until the 10–20% FDV target is reached:

- **From post-listing discount sale proceeds:** 30% of every future discount sale's raised capital is earmarked for liquidity top-ups (paired with unpaired OTI from the reserved Liquidity bucket), until the pool reaches at least 10% of FDV at the time.
- **From API revenue:** none of the revenue distribution (Section 6) is earmarked for liquidity by default — this keeps the two funding sources cleanly separated. If liquidity growth stalls, this can be revisited.
- Once the 10–20% FDV target is reached and sustained, liquidity top-ups from discount sales can be reduced or stopped, and proceeds redirected fully to operations/team/treasury per Section 6.

**Bottom line:** OTI launches with thin but real, fully-disclosed liquidity, and has an explicit, funded mechanism to grow it — rather than either overpromising day-one depth it can't afford, or ignoring the problem.

---

## 6. Revenue Distribution

Once OTI's paid API tiers (Pro/Enterprise, per the existing Revenue Model in TASKS.md) begin generating real revenue, that revenue is split every month as follows — **starting from day one, immediately after the pre-listing sale round completes** (not gated behind any revenue threshold):

| Category | % | What it's for |
|---|---|---|
| Operating Costs | 40% | Hosting (Railway), any paid data providers/APIs, tools, infra — always covered first |
| Revenue-Backed Rewards Pool (buyback) | 25% | Buys OTI on the open market and routes it into the Rewards Pool for stakers — **only executes when the market is down**; held as cash reserve otherwise, deployed opportunistically |
| Team | 20% | Cash payout to the founders (Ahmad + Musty) — split decided internally between them. This is separate from the Team's fixed token allocation (Section 2), which is equity-like, not cash |
| Treasury/Reserve top-up | 15% | Cash reinvested into growth — marketing, audits, partnerships, legal. Separate from the fixed Treasury token allocation; this is a cash reserve, not tokens |

**Why the buyback is conditional, not automatic:** buying back tokens every month regardless of price would mean routinely overpaying when the market is already strong. Restricting the buyback to down-market conditions makes the Rewards Pool a genuine value-accrual mechanism (buy low) rather than a fixed monthly expense.

---

## 7. Chain & Cross-Chain Plan

- **Launch chain:** BNB Smart Chain (BSC) — chosen for low fees and fast settlement, matching the presale mechanics above (BNB/USDT accepted).
- **Cross-chain expansion:** planned for later, once OTI's token and liquidity are established on BSC. Not yet designed in detail — when this becomes active work, it should be scoped as its own task (likely via a standard bridging solution such as LayerZero or Wormhole, rather than a custom-built bridge, consistent with the "don't reinvent solved infrastructure" principle applied to vesting).

---

## 8. Explicitly Out of Scope For Now

- **Staking/derivative system (e.g., a "locked OTI" derivative used as collateral):** discussed as a future phase, not part of this design. To be scoped separately once OTI has real usage and revenue to build the mechanism around.
- **Governance:** not part of this design. No DAO or voting mechanism defined yet for OTI token holders.
- **Formal legal/securities review:** flagged once to Ahmad; his explicit decision is that the raise is small enough not to require this at this stage.
