# REVIEW.md: what the review panel checks in wealthfolio (Bryan's fork)

review-gate reads this from the base branch (`bryan-main`) on every PR. Finders propose, a verifier confirms; an unconfirmed finding is dropped.

Scope: Bryan's additions only. Today that is the `ctx.api.alternativeAssets` addon-API patch (`packages/addon-sdk`, `apps/frontend/src/addons/`) and the router docs. Upstream code is out of scope unless a change touches it. The rental-tax addon and its bridge live in `~/finances`, not here, but this fork's addon API is what they depend on.

## Critical (blocks the merge)

- An addon API reachable without its permission: a method added to the host API outside `permissions.ts`, or `alternative-assets` weakened below `riskLevel: high`. No path that bypasses the permission system (bundled `@tauri-apps/api` invoke, global Tauri).
- An addon able to delete, edit metadata or link liabilities on alternative assets. Those stay app-only until a concrete addon need.
- A secret, token, account number, holding or real financial data in the repo, fixtures or a test.
- A push or PR aimed at `upstream`, or `bryan-main` rewritten in a way that drops Bryan's patches on a sync.
- A bulk-save result whose `errors` is discarded: v3 bulk save is all-or-nothing, and total rejection looks like success without them.

## Important (blocks until fixed)

- Payload shapes written against the local `.d.ts` instead of the host app's source at its current version (the installed typings lied about the wire format).
- A private-IP `fetch` or LAN call from an addon: the webview blocks `fetch` and the broker SSRF-blocks private IPs. It needs `ctx.api.network.request` plus HTTPS over Tailscale Serve.
- A change to upstream's `AGENTS.md` below the fork note, or to `.claude/CLAUDE.md` (upstream's, byte-identical, never trimmed), or the fork note removed.
- Bare `npm` or `pnpm` in a script or doc: it is `corepack pnpm`.
- An upstream sync that does not rerun the addon-sdk build, `pnpm type-check` and the frontend vitest, or touches `apps/frontend/src/addons/` without the guard test passing.
- `STATE.md` committed (it is git-excluded).

## Ignore

- Style, naming and formatting.
- Everything upstream's: `crates/`, `apps/tauri`, `apps/server`, `docs/`, lockfiles, vendored assets, e2e fixtures.

## Evidence required

Every finding cites `file:line` and the input that breaks it. No input, no finding.

## Rules learned

Dated one-liners naming the PR that taught them. Add one when a review misses something real or a finding repeats.

- 2026-07-08: v3 networking migration broke the LAN bridge; use the brokered network API over HTTPS, verify shapes against host source, keep bulk `errors` (finances PR #7, see CLAUDE.md "Before any upgrade").
- 2026-07-11: unpinned `^3.0.0` addon SDK resolved to 3.6.1 and removed `RouteConfig.component`; pin exact versions and commit the lockfile (finances).
- 2026-09-16: `.claude/CLAUDE.md` is upstream's byte-for-byte; trimming it conflicts on every pull (commit 53775614e).
