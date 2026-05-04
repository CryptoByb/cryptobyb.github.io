# CryptoByB — Stablecoin Blacklist Checker

## Project

- **Main file:** `index.html` (single self-contained file — all CSS + JS inline)
- **Old V1:** `blacklist-checker.html` (reference only, do not modify)
- **Planning:** `.planning/` (PROJECT.md, ROADMAP.md, REQUIREMENTS.md, STATE.md)

## Stack

Pure HTML/CSS/JS — no build tools, no npm, no frameworks. Single file deployment.

## GSD Workflow

This project uses GSD (Get Shit Done) for structured planning and execution.

**Always check `.planning/STATE.md` first** to understand the current phase and status.

**Commands:**
- `/gsd-progress` — check where we are
- `/gsd-discuss-phase 1` — discuss approach before planning
- `/gsd-plan-phase 1` — create execution plan for a phase
- `/gsd-execute-phase 1` — execute the plan
- `/gsd-ui-phase 1` — generate UI design contract (use for Phase 1)
- `/gsd-code-review` — review changes before shipping

## Design Rules (Phase 1 constraint — CRITICAL)

- **DO NOT** produce generic AI-template UI (soft purples, gradient cards, Inter font everywhere)
- **DO** build on existing identity: JetBrains Mono, `#07080f` bg, `#4f8ef7` blue accent
- Reference aesthetic: Linear, Vercel dashboard — sharp, purposeful, data-forward
- The redesign must feel hand-crafted and distinctive to CryptoByB

## Key Decisions

- Client-side only — no backend, no data stored. This is the core privacy guarantee.
- Single HTML file — all CSS and JS stay inline in `index.html`
- CryptoByB branding — keep existing name and identity
