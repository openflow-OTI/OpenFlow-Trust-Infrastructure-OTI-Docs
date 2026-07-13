---
name: Contract addresses scored like any wallet
description: Product decision on whether smart-contract addresses should be excluded from wallet trust scoring, and why the underlying bot/scam-risk concern is handled separately.
---

Decision: smart-contract addresses (multisigs, smart-wallet accounts, token contracts, etc.) are not gatekept or rejected by address type — they are scored exactly like any normal wallet address.

**Why:** the scoring product never verifies identity for any address in the first place, so "there might not be a person directly behind this contract" isn't a meaningfully different trust bar than a normal EOA, where a person is also just assumed. Many contract addresses genuinely are how a real person or org holds funds (multisig/smart-account wallets), and on some chains (e.g. TON) every wallet is technically implemented as a contract — rejecting by address type would misclassify large swaths of legitimate real usage. Scoring a contract's real on-chain history doesn't fabricate anything.

**How to apply:** don't build or resurrect contract-detection/rejection logic for scoring purposes. The actual underlying concern (bots, scam contracts, wash-trading, non-human malicious patterns) isn't unique to contracts — a plain EOA can exhibit the same patterns — and is intentionally deferred to a future dedicated bot/suspicious-wallet behavioral-detection design phase, tracked separately in the roadmap. Don't try to solve that risk by re-introducing address-type gatekeeping.
