---
phase: 1
title: UI Overhaul
status: ready-to-plan
---

# Phase 1 Context — UI Overhaul

## Locked Decisions

### 1. Result Display
**Chain status blocks** — compact grid, one block per chain.
Each block shows: chain name + token (e.g. "Ethereum USDT"), status badge (CLEAN / BLACKLISTED / ERROR), and scan state while loading.
Desktop: 3 columns. Mobile: 2 columns, full-stack vertical on narrow viewports.

### 2. Status Text
Use **CLEAN / BLACKLISTED / ERROR** — exactly as-is, no changes.
- CLEAN = green
- BLACKLISTED = red
- ERROR = amber/yellow

### 3. Header / Hero
**Minimal tool header** — logo + tagline only. No marketing hero section, no CTA, no "built for crypto professionals" copy.
CryptoByB logo left-aligned, one-line tagline right or below. Compact. Feels like a dashboard, not a landing page.

### 4. Scan Animation
**Sequential reveal** — each chain block flips from "scanning…" → result as its RPC call returns, staggered.
Not a global spinner. Each block has its own loading state. Results appear as they come in, not all at once.
Animation: subtle fade + slight upward translate on reveal. No bouncy/springy motion.

### 5. Mobile Layout
Full-stack vertical on mobile (< 480px). Chain blocks: 2-column grid on mobile (375px viewport — no horizontal scroll).
Touch-friendly inputs (min 44px tap targets). Tabs remain horizontally scrollable row on very small screens if needed.

### 6. Tab Style
**Segmented control** — pill-style container, active tab = solid fill (#4f8ef7 bg, dark text), inactive = ghost (transparent bg, muted text).
Tabs: Blacklist / Approvals / TX Tracer / Batch. No underline tabs, no vertical sidebar tabs.

### 7. Desktop Result Columns
**3 per row** on desktop (≥ 1024px). 2 per row on tablet (768–1023px). 2 per row on mobile.

---

## Design Identity (Non-Negotiable)

- Font: JetBrains Mono — primary throughout. Inter only if needed for body copy.
- Background: `#07080f`
- Accent: `#4f8ef7` (blue)
- Surface: ~`#0d0e17` (cards/inputs), border: ~`#1a1b2e`
- Reference aesthetic: Linear, Vercel dashboard, Etherscan — sharp, data-forward, no decorative gradients
- **Do NOT** use: soft purples, floating gradient cards, rounded-everything, Inter-everywhere
- Must feel hand-crafted and distinctive to CryptoByB

---

## What the Planner Should Do

1. Redesign the page structure in `index.html`:
   - Minimal header (logo + tagline)
   - Segmented control tabs replacing current tab design
   - Chain status block grid (3-col desktop, 2-col tablet/mobile)
   - Sequential reveal animation per block
   - Polished scan state per block (skeleton/pulse while loading)

2. Fix mobile layout — no horizontal overflow at 375px

3. Preserve all existing functionality 100%:
   - All RPC calls and CHAINS config unchanged
   - History (sessionStorage), share URL params
   - Approvals scanner, TX Tracer, Batch check tabs
   - Tron support

4. Single `index.html` — all CSS and JS inline, no external dependencies added

---

## Out of Scope for Phase 1

- Wallet risk score badge (Phase 2)
- New chain coverage for BSC/Polygon/Arbitrum blacklist (Phase 3)
- Dark/light mode toggle (v2 deferred)
- ENS resolution (v2 deferred)
