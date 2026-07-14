---
name: Phase 2B Final Architecture
description: All locked decisions for the OTI Verified Badge — attestation layer, chains, pricing, display, removed components.
---

# Phase 2B — OTI Verified Badge: Final Architecture

**Why:** Ahmad redesigned Phase 2B entirely in session 15 (July 14, 2026). Several components from the original proposal were removed. Future sessions must not reconstruct them without Ahmad explicitly reopening.

## Locked Decisions

**Attestation layer:** BAS (BNB Attestation Service) on BNB Chain only. No Ethereum EAS, no other chains, no OTI-owned contract. One BAS attestation per wallet covers all EVM chains (wallet address is chain-agnostic).

**Removed — do not rebuild without Ahmad:**
- On-chain soulbound NFT (ERC-5192 / ERC-5114) — too complex to rescore, gas at scale, unnecessary
- MetaMask Snap — only MetaMask supports it, user must install, widget + extension cover the same ground

**Pricing:**
- First 10 million attestations: FREE (deliberate network effect investment)
- Post-10M: one-time fee per wallet (not recurring, not a subscription)
- OTI token discount for users who pay in OTI token
- Fee amount + discount: admin panel managed, never hardcoded

**Rescoring:** OTI handles all rescoring automatically every 30 days. User does nothing after initial attestation. Daily rolling batch — wallets rescored as their 30-day windows expire.

**Display layers:**
- Widget (partner-side): partner embeds OTI widget → widget reads BAS → shows badge in partner UI
- Extension (user-side): user installs once → detects wallet addresses on every website → shows badge everywhere
- Neither the attestation nor the soulbound NFT displays the badge — the widget and extension do.

**Token integration:**
- First 1 million attestation users receive OTI token before token launch (from Ecosystem bucket)
- Token pays for attestation fee at a discount

## Open Decisions (before task prompts can be written)
1. Badge tier visual design — 5 distinct designs, Ahmad to finalise
2. Score thresholds per tier — confirm from codebase
3. OTI signing key management/rotation policy
4. Attestation fee amount — Ahmad decides, admin panel manages

**How to apply:** Any Builder prompt for Phase 2B must reference BAS SDK integration, not EAS, not soulbound contracts. Check DECISIONS.md D17–D22 before writing any Phase 2B prompt.
