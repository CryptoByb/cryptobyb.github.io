---
phase: 04-token-approvals
plan: "02"
subsystem: frontend-ui
tags: [approvals, trc20, xss-prevention, dom-api]
dependency_graph:
  requires: [04-01]
  provides: [appr-ui, runApprovals]
  affects: [index.html]
tech_stack:
  added: []
  patterns: [dom-creation-api, textContent-xss-guard, finally-button-reset]
key_files:
  created: []
  modified:
    - index.html
decisions:
  - "Used DOM creation API (createElement/textContent) exclusively for all API-sourced data — no innerHTML with untrusted content"
  - "Used --border (declared) instead of --border2/--border3 (undeclared, CR-01) for all new .appr-table CSS"
  - "Spender truncation implemented inline (slice 0-8 + ellipsis + slice -6) rather than reusing short() helper — short() uses 16-char threshold which is too long for addresses"
metrics:
  duration: "8 minutes"
  completed: "2026-05-04"
  tasks_completed: 2
  tasks_total: 2
---

# Phase 4 Plan 02: Token Approvals UI Summary

Token Approvals tab fully replaced — input field, scan button, loading/empty/error states, and results table with UNLIMITED badges rendering TRC-20 approval data from TronScan API using DOM creation API throughout (zero innerHTML with API data).

## Tasks Completed

| # | Task | Commit | Files |
|---|------|--------|-------|
| 1 | Add CSS classes for approvals table and states | 826fc04 | index.html |
| 2 | Replace #tab-approvals placeholder and add runApprovals() | 5bc788d | index.html |

## What Was Built

**Task 1 — CSS classes (5 new classes):**
- `.appr-table` — full-width table using `--border` (declared variable, avoids CR-01)
- `.appr-unlimited` — orange badge with `var(--orange)` / `var(--orange-dim)` styling
- `.appr-loading` — flex centered loading state with gap for spinner
- `.appr-empty` — centered muted text for zero-result state
- `.appr-err` — red-dim error panel matching existing `.err-box` aesthetic

**Task 2 — UI + function:**
- `#tab-approvals` now contains: `id="apprInput"` (Tron address input), Paste button, `id="apprBtn"` scan button, `id="apprResults"` container
- `runApprovals()` function: validates address with `isTron()` before any fetch, shows loading spinner, fetches `https://apilist.tronscanapi.com/api/account/approve/list`, renders results table or empty/error state
- Enter key listener on `apprInput` wired to `runApprovals()`
- `lastApprovalData` updated after successful scan so blacklist tab risk score stays in sync

## Deviations from Plan

None — plan executed exactly as written.

## Security (Threat Model Compliance)

| Threat ID | Status | Implementation |
|-----------|--------|----------------|
| T-04-04 (XSS) | Mitigated | All API fields set via `.textContent` — `results.innerHTML = ''` only resets container with no API data |
| T-04-05 (Spoofing) | Mitigated | `isTron()` regex validation before any fetch; non-Tron addresses rejected with inline error in `#apprErr` |
| T-04-06 (DoS) | Accepted | Loading spinner visible during fetch; catch block renders error via textContent into `.appr-err`; `finally` re-enables button |
| T-04-07 (Info Disclosure) | Accepted | Spender full address in `title` attribute only — public blockchain data |

## Known Stubs

None. All data paths are wired to the live TronScan API.

## Threat Flags

None — no new network endpoints, auth paths, or trust boundaries introduced beyond those already declared in the plan's threat model.

## Self-Check: PASSED

- index.html exists and contains all required elements
- Commit 826fc04 exists (Task 1 CSS)
- Commit 5bc788d exists (Task 2 UI + function)
- `grep "coming soon"` returns 0 matches
- `grep "id=\"apprInput\""` returns 1 match
- `grep "id=\"apprBtn\""` returns 1 match
- `grep "id=\"apprResults\""` returns 1 match
- `grep "async function runApprovals"` returns 1 match
