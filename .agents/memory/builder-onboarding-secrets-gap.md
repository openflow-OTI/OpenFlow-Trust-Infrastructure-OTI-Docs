---
name: Builder onboarding secrets gap
description: New Builder Replit environments start with zero env vars/secrets configured; nothing carries over from a previous Builder's environment.
---

When a Builder (Backend or Frontend) is replaced or a new one is onboarded into a fresh Replit environment, none of the previous Builder's secrets exist there — not API keys, not session/admin secrets, nothing. This includes third-party chain-explorer keys (Etherscan, Tronscan, Toncenter, Routescan, zkSync alt-endpoint key) and app secrets (SESSION_SECRET, ADMIN_SECRET).

**Why:** secrets are scoped to the individual Replit environment/account, not to the "Backend Builder" role — Ahmad's Railway/Vercel production secrets are separate from whatever the new Builder's own dev environment needs to test locally.

**How to apply:** whenever a new Builder is onboarded, checklist their environment for every required secret (see BUILDER_ONBOARDING.md / ARCHITECTURE.md for the full list) before assuming any integration will work — don't wait for a live rejection error to discover a key is missing.

**Related gotcha caught in the wild:** an old Etherscan API key was rejected because Etherscan migrated to a unified multi-chain (V2) key format — a pre-migration key doesn't work against the V2 endpoints even though it looks superficially valid. If a key "used to work" and now gets rejected, check whether the provider changed key formats/versions before assuming the key itself is wrong.
</content>
