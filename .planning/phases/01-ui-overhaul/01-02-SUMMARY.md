---
phase: 01-ui-overhaul
plan: "02"
subsystem: result-display
tags: [css, grid, animation, chain-blocks, status-badges]
dependency_graph:
  requires: [nav-seg, logo-mark, css-variables-v2]
  provides: [chain-grid, chain-block, block-status, block-reveal]
  affects: [index.html]
tech_stack:
  added: []
  patterns: [css-grid, keyframe-animation, sequential-reveal, skeleton-scan-state]
key_files:
  created: []
  modified:
    - StableCoin Blacklist Checker/index.html
decisions:
  - "block.id='row-'+c.id preserved so updateRow() finds elements by existing id convention — no other JS changes needed"
  - "scanning::after shimmer uses ::after pseudo-element to avoid adding DOM nodes for each block"
  - "revealed class added after scan completes; opacity:0 default on .chain-block ensures blocks are invisible until RPC returns"
metrics:
  duration: "~8 minutes"
  completed: "2026-05-04"
  tasks_completed: 2
  tasks_total: 2
---

# Phase 01 Plan 02: Chain Result Grid + Sequential Reveal Summary

One-liner: Replaced list-style result rows with a 3-col CSS Grid of compact chain blocks, each with a per-block shimmer scan state and sequential fade-in+translateY reveal animation triggered as each RPC call returns.

## What Changed

### CSS Removed (old list-row system)

| Rule | Reason |
|------|--------|
| `.res-rows` (bordered container) | Replaced by CSS Grid container |
| `.res-row` | Replaced by `.chain-block` |
| `.chain-badge` | Replaced by `.block-short` |
| `.chain-name` | Replaced by `.block-chain` |
| `.row-status` and all variants (`.scanning`, `.safe`, `.blacklisted`, `.error`) | Replaced by `.block-status` system |
| `.sdot` | Removed — badges are pill-shaped, no dot needed |
| `@keyframes pulse` | Removed — shimmer replaces pulsing dot |

### CSS Added (new block system)

| Rule | Purpose |
|------|---------|
| `.res-rows` (grid) | `display:grid; grid-template-columns:repeat(3,1fr); gap:8px` |
| `.chain-block` | Card container: surface bg, border, 88px min-height, `opacity:0` default |
| `.chain-block.revealed` | Triggers `block-reveal` animation, sets `opacity:1` |
| `@keyframes block-reveal` | `opacity:0 + translateY(6px)` → `opacity:1 + translateY(0)` over 250ms |
| `.chain-block.scanning::after` | Shimmer pseudo-element using gradient animation |
| `@keyframes shimmer` | Horizontal gradient sweep at 1.4s interval |
| `.block-top` | Flex row: chain name left, ticker right |
| `.block-chain` | Chain full name — 11px, weight 600, muted2 color |
| `.block-short` | Ticker pill — surface2 bg, muted color |
| `.block-status` | Badge base — inline-flex, mono, 12px, weight 700, 5px border-radius |
| `.block-status.scanning` | Muted text + inline spinner, no background |
| `.block-status.safe` | Green text + green-dim bg + green border |
| `.block-status.blacklisted` | Red text + red-dim bg + red border |
| `.block-status.error` | Amber text + amber-dim bg + amber border |
| `@media(max-width:1023px)` | `.res-rows` switches to `repeat(2,1fr)` for tablet |
| `@keyframes spin` | Re-added (used by `.block-status.scanning .spin`) |

### JS Changes

**chains.forEach block (row creation inside runBlacklist())**

Before: created `div.res-row` with `.chain-badge`, `.chain-name`, `.row-status.scanning` + `.spin`

After: creates `div.chain-block.scanning` with:
- `.block-top` containing `.block-chain` (full name) and `.block-short` (ticker)
- `.block-status.scanning` containing `.spin` and "Scanning…" text
- `id='row-'+c.id` preserved — no change to updateRow lookup

**updateRow(id, bl, err) function**

Before: queried `.row-status`, set className to `row-status safe/blacklisted/error`, used `.sdot` in innerHTML, set `row.style.background` for blacklisted

After:
- Removes `.scanning` class from block (stops shimmer)
- Queries `.block-status` element
- Sets `st.className='block-status safe|blacklisted|error'` with plain `<span>` text content only (no sdot)
- For blacklisted: sets `block.style.borderColor='rgba(239,68,68,0.25)'` for extra visual signal
- Adds `.revealed` class to trigger the block-reveal keyframe animation

## Deviations from Plan

None — plan executed exactly as written.

## Known Stubs

None. The chain grid wires directly to the existing RPC call pipeline. All status states (CLEAN, BLACKLISTED, ERROR) are hardcoded strings, not derived from RPC content.

## Threat Flags

None. Status badge text (`CLEAN`, `BLACKLISTED`, `ERROR`) is hardcoded in updateRow — not interpolated from RPC response. No new network endpoints or auth paths introduced.

## Self-Check

- [x] `index.html` modified
- [x] Commit `2307f9c` — Task 1: chain grid CSS + block-reveal animation
- [x] Commit `6baed4b` — Task 2: JS row creation + updateRow to chain blocks
- [x] `grep -c 'chain-block' index.html` = 4 (>= 4 required)
- [x] `grep -c 'block-reveal' index.html` = 2 (>= 2 required)
- [x] `grep -c 'block-status' index.html` = 11 (>= 6 required)
- [x] `grep -c '\.res-row{' index.html` = 0 (old list rule removed)
- [x] `grep -c 'row-status' index.html` = 0 (old class fully removed from JS)
- [x] `.res-rows{display:grid;grid-template-columns:repeat(3,1fr)` present
- [x] `@keyframes block-reveal` present
- [x] `@keyframes shimmer` present
- [x] `.block-status.safe{color:var(--green)` present
- [x] `.block-status.blacklisted{color:var(--red)` present
- [x] `.block-status.error{color:var(--amber)` present
- [x] `@media(max-width:1023px)` with `repeat(2,1fr)` present
- [x] JS: `block.className='chain-block scanning'` present
- [x] JS: `block.classList.add('revealed')` present
- [x] JS: no `row.className='res-row'` remaining
- [x] JS: no `row-status` remaining
- [x] JS: `block.querySelector('.block-status')` present
