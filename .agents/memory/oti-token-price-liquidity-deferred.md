---
name: OTI token price & liquidity deferred
description: Price and liquidity pool design were deliberately removed from TOKENOMICS.md; do not reconstruct or re-add them without Ahmad raising it again.
---

During tokenomics design, price and liquidity pool structure went through many iterations (dollar amounts, FDV vs. circulating market cap, a "10x liquidity-to-market-cap ratio" that kept being interpreted two different ways). Ahmad ultimately decided this was premature and told the Manager to strip all of it out.

**Why:** the underlying disagreement was never really about the math — Ahmad wants the actual price and liquidity pool structure decided by the team later, closer to launch, once real conditions (raised capital, market context) are known. Locking numbers in now created a real risk of publishing figures that don't match what the team eventually decides.

**How to apply:** `TOKENOMICS.md` now covers only total supply, allocation, vesting, sale structure (rounds, no price), and revenue distribution. Do not add price, liquidity pool sizing, market cap targets, or liquidity ratios back into that file — or infer them from git history — unless Ahmad explicitly reopens the topic himself.
