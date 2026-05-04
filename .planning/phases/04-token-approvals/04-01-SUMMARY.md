---
phase: 04-token-approvals
plan: "01"
subsystem: blacklist-risk
tags: [tron, approvals, risk-scoring, tronscan]
dependency_graph:
  requires: []
  provides: [fetchTronApprovals, lastApprovalData, HIGH-risk-tier]
  affects: [computeRisk, runBlacklist]
tech_stack:
  added: []
  patterns: [fire-and-forget async fetch, silent error swallowing, optional parameter with fallback]
key_files:
  created: []
  modified:
    - index.html
decisions:
  - "fetchTronApprovals() swallows all errors and returns 0 — blacklist display must never block on approval fetch"
  - "unlimitedCount defaults via || 0 (not default param) to match existing code style"
  - "tron variable reused from runBlacklist() scope to gate fetch — no redundant isTron() call"
metrics:
  duration: "~5 minutes"
  completed: "2026-05-04"
  tasks_completed: 2
  tasks_total: 2
---

# Phase 4 Plan 01: TronScan Approval Fetch + HIGH Risk Tier Summary

HIGH risk tier wired to live TronScan data — Tron addresses with unlimited approvals now surface `risk-high` in the blacklist risk banner via a silent fire-and-forget fetch after each scan.

## Tasks Completed

| # | Task | Commit | Files |
|---|------|--------|-------|
| 1 | Add fetchTronApprovals() and lastApprovalData state variable | 23f4efa | index.html |
| 2 | Update computeRisk() + HIGH tier + runBlacklist() wiring | 9a6bad3 | index.html |

## What Was Built

**Task 1** added two things to `index.html`:
- `let lastApprovalData = null;` in the state variable block — stores the raw TronScan JSON response for the Approvals tab to consume in a future plan
- `async function fetchTronApprovals(addr)` — calls `https://apilist.tronscanapi.com/api/account/approve/list?address=...`, checks `Array.isArray(j.data)` before iterating, counts rows where `unlimited === true`, stores the full response in `lastApprovalData`, and returns 0 silently on any error

**Task 2** made three targeted edits:
- `computeRisk()` signature updated to `computeRisk(results, unlimitedCount)` with `unlimitedCount = unlimitedCount || 0` fallback
- HIGH tier branch inserted between CRITICAL and MEDIUM: `if(unlimitedCount>0) return {level:'HIGH', cssClass:'risk-high', ...}`
- `runBlacklist()` now runs `const unlimitedCount = tron ? await fetchTronApprovals(addr) : 0;` before calling `computeRisk(results, unlimitedCount)`

## Risk Tier Precedence (confirmed)

```
CRITICAL  (anyBlacklisted)         — blacklisted on any chain
HIGH      (unlimitedCount > 0)     — unlimited approvals, not blacklisted
MEDIUM    (anyError && defined>0)  — partial chain data
MEDIUM    (defined.length === 0)   — all RPCs failed
LOW       (otherwise)              — clean on all chains
```

## Deviations from Plan

None — plan executed exactly as written.

## Threat Model Compliance

| Threat ID | Mitigation Applied |
|-----------|-------------------|
| T-04-01 | `Array.isArray(j.data)` guard before iteration; only `.unlimited` boolean read — no DOM injection |
| T-04-02 | `lastApprovalData` in-memory only, not persisted to storage |
| T-04-03 | All fetch errors caught, return 0; blacklist display completes independently |

## Known Stubs

- `lastApprovalData` is populated but not yet consumed by any UI — the Approvals tab (plan 04-02) will read it

## Self-Check: PASSED

- `let lastApprovalData = null;` present in index.html
- `async function fetchTronApprovals(addr)` present in index.html
- `apilist.tronscanapi.com/api/account/approve/list` present in index.html
- `function computeRisk(results, unlimitedCount)` — 1 match
- `unlimitedCount>0` — 1 match inside HIGH branch
- `await fetchTronApprovals(addr)` — 1 match in runBlacklist
- `computeRisk(results)` (old single-arg call) — 0 matches
- Commits 23f4efa and 9a6bad3 verified in git log
