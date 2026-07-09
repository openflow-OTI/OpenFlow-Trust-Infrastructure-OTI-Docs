---
name: Vercel cross-project docs proxy (rewrite) gotchas
description: Lessons from proxying a second Vercel project (Docusaurus docs) under /docs/ on the main site via vercel.json rewrites.
---

## Trailing slash is a distinct route in Vercel rewrites
`/docs` and `/docs/` are matched as separate literal patterns. A rewrite rule for
`/docs/:path*` does NOT automatically also match the bare `/docs/` (empty path)
request in all cases — you need explicit rules for all three: `/docs`, `/docs/`,
and `/docs/:path*`, all placed before the SPA catch-all rewrite.

**Why:** We shipped `/docs` and `/docs/:path*` rules and verified `/docs` and
`/docs/api-reference` worked via curl, but `/docs/` (the trailing-slash form —
what users actually hit from a nav link) fell through to the main app's SPA
catch-all and showed the main site's own 404, even though the proxy itself was
otherwise working correctly.

**How to apply:** Whenever adding a Vercel rewrite to proxy a path prefix to
another origin, always test the exact prefix with AND without a trailing
slash, plus at least one nested sub-path, via curl — don't assume a wildcard
rule covers the bare-slash case.

## Docusaurus baseUrl vs. output file structure
Setting `baseUrl: '/docs/'` in `docusaurus.config.js` changes generated
`href`/`src` link prefixes and the router basename, but does NOT nest the
physical build output under a `docs/` folder — `index.html`,
`api-reference/index.html`, etc. still land at the build output root. Only
`docs.routeBasePath` affects where doc pages route within routeBasePath must
be `''` (not `'/'`) if you want the docs plugin's pages to live at the site
root when `routeBasePath` is otherwise conflicting with a non-root `baseUrl`.

**Why:** Cost real debugging cycles across two rounds — first assuming
`routeBasePath: '/'` was fine with `baseUrl: '/docs/'` (it isn't — causes the
root/"Getting Started" page to 404), then assuming Vercel's Output Directory
setting must be misconfigured to explain root-level bare paths working
standalone (it wasn't — Output Directory override was off/default; the flat
file structure was expected Docusaurus behavior all along).

**How to apply:** When debugging a "some /docs/X pages work, others don't"
issue on a Docusaurus site behind a path-prefix reverse proxy, check the
*physical* build output structure directly (`ls build/` after a local build)
before assuming the hosting platform's directory settings are wrong.
