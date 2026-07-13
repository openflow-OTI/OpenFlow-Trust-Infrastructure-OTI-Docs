---
name: Signal-first scoring QA methodology
description: How to structure manual verification passes over the scoring engine across chains
---

When auditing/testing the scoring engine's output quality, sweep by **signal**,
not by chain: pick one signal (e.g. wallet age, smart-contract diversity,
transaction volume, token holdings), pull its value for test wallets across
every supported chain, and compare them side by side for outliers or
suspicious repeated fingerprints (e.g. identical round numbers, identical
"1" values) before moving to the next signal.

**Why:** Chain-by-chain review missed cross-chain structural bugs. It was only
when smart-contract-interaction counts were compared across chains in one pass
that a suspicious repeated "1 interaction, 20/20" fingerprint stood out on
Solana, TON, and Sui simultaneously — leading to BF33. The same
signal-across-chains comparison surfaced BF34 (TON wallet age returning 0 for
a wallet with 373 real transactions). Reviewing one chain fully before moving
to the next hides these patterns because each chain's oddity looks like a
one-off in isolation.

**How to apply:** When Ahmad or a Builder reports fresh score-card results
(screenshots, live tests, etc.), group them by signal first. Work through all
five scored signals one at a time across every chain with available data
before declaring a testing round complete.
