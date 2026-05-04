# CryptoByB — Stablecoin Blacklist Checker

## What This Is

A client-side web security tool for crypto users to check if a wallet address has been blacklisted by USDT or USDC issuers, detect dangerous token approvals, trace transactions between wallets, and batch-check multiple addresses — all via direct on-chain RPC calls with no backend and no data storage.

Main file: `StableCoin Blacklist Checker/index.html` (self-contained, all logic inline).

## Core Value

A crypto user pastes a wallet address and immediately knows if it's blacklisted or compromised — no signup, no backend, no trust required.

## Requirements

### Validated

- ✓ Blacklist check for USDT across 6 chains (ETH, Tron, Avalanche, Polygon, Arbitrum, Optimism) — existing
- ✓ Blacklist check for USDC across 6 chains (ETH, Arbitrum, Base, Optimism, Polygon, Avalanche) — existing
- ✓ Token approvals scanner for EVM (ETH, BSC, Polygon, Arbitrum) with revoke links — existing
- ✓ Token approvals scanner for Tron USDT with revoke links — existing
- ✓ TX Tracer — trace USDT/USDC transfers between two wallets on a chosen chain — existing
- ✓ Batch checker — check multiple addresses at once — existing
- ✓ Search history (session storage) — existing
- ✓ Share link via URL params — existing
- ✓ CryptoByB branding — existing

### Active

- [ ] UI overhaul — premium dark design distinctive to CryptoByB, not generic AI-look. Build on existing identity: JetBrains Mono, tight dark theme, blue accent (#4f8ef7). Must look hand-crafted.
- [ ] Wallet Risk Score — single LOW/MEDIUM/HIGH rating after checks complete, combining blacklist + unlimited approvals results
- [ ] Align blacklist chains with approvals chains — add BSC/Polygon/Arbitrum to blacklist tab (currently only in approvals scanner)

### Out of Scope

- Backend / API — intentional, privacy guarantee is that nothing is stored
- User accounts — not needed for this tool
- Generic AI-template UI (soft purples, gradient cards, Inter font everywhere) — explicitly rejected

## Context

- Pure client-side HTML/CSS/JS, single file: `StableCoin Blacklist Checker/index.html`
- `blacklist-checker.html` is an older V1 — `index.html` is the active file
- Deployed on Netlify: `https://spiffy-sawine-0cc0cf.netlify.app`
- RPC calls go directly to public endpoints (llamarpc, cloudflare-eth, publicnode, TronGrid)
- Existing design uses JetBrains Mono + Inter, dark bg (#07080f), blue accent (#4f8ef7)
- User is aware that AI-generated designs look samey — the redesign must feel hand-crafted and distinctive

## Constraints

- **Tech stack**: Pure HTML/CSS/JS only — no build tools, no frameworks, no npm
- **File structure**: Single self-contained HTML file (all CSS + JS inline)
- **Privacy**: No data stored, no backend calls beyond public RPC nodes
- **Branding**: CryptoByB — keep existing name and brand identity

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Single HTML file, no build pipeline | Zero deployment friction, works anywhere | ✓ Good |
| Client-side only, public RPCs | Privacy guarantee — nothing logged server-side | ✓ Good |
| UI overhaul before new features | Tool works well; trustworthiness/shareability is the bottleneck | — Pending |
| Build on existing design identity, not a new template | Avoid generic AI-look; user explicitly wants distinctive design | — Pending |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd-transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd-complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-05-04 after initialization*
