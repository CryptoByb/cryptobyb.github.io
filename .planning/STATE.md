# Project State — CryptoByB Stablecoin Blacklist Checker

## Project Reference

See: .planning/PROJECT.md (updated 2026-05-04)

**Core value:** A crypto user pastes a wallet address and immediately knows if it's blacklisted or compromised — no signup, no backend, no trust required.
**Current focus:** Phase 2 — Wallet Risk Score

## Current Phase

**Phase 2: Wallet Risk Score** — NOT STARTED

After all checks complete, surface a single risk rating (LOW / MEDIUM / HIGH / CRITICAL) that combines all signals into one answer users can act on immediately.

## Phase Status

| # | Phase | Status |
|---|-------|--------|
| 1 | UI Overhaul | ✓ Complete (2026-05-04) |
| 2 | Wallet Risk Score | Not started |
| 3 | Chain Alignment | Not started |

## Key Files

- Main file: `index.html`
- Planning: `.planning/`

## Known Issues (from Phase 1 review)

- CR-01: `--border2` / `--border3` CSS variables undeclared — borders on inputs/buttons may be invisible
- CR-02: Vacuous `every()` produces false CLEAN when all RPCs fail
- CR-03: XSS via `innerHTML` with unsanitised sessionStorage / RPC error messages
- CR-04: Tron address ABI hex extraction wrong — may return incorrect blacklist result
