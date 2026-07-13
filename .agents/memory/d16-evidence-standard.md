---
name: D16 evidence standard, applied retroactively
description: How to judge whether a Builder's "verified" claim is actually evidence, and why this must be re-checked even after a fix is reported closed.
---

Rule: no signal value, test result, or fix confirmation may rest on an estimate, a guess, or a description of what the code "should do." Only two things count as evidence — a real on-chain wallet with a real, fresh API response, or a disclosed hard cap. This applies to every claim of "verified" or "confirmed," including ones made about work already reported done.

**Why:** twice in one session a Builder's report used the word "verified" but the underlying evidence didn't meet the bar — once it meant "I read the code and it looks right" (no live wallet tested), once it meant "I checked a cached Railway response" (not a fresh call, breaking from the standard the rest of the batch used). Both looked fine on a first read of the report text. The gap only surfaced when the Manager asked a concrete question: "what specific wallet, what specific raw response?"

**How to apply:** before accepting any fix as closed, ask for (or check for) the specific address tested, the raw upstream API response, and the fresh Railway response — not just the Builder's summary sentence. If any of those three is missing, generic, or stale/cached, send it back rather than closing it. This applies even to items a Builder has already marked "done" in a report — re-derive the evidence yourself, don't take the label at face value.
