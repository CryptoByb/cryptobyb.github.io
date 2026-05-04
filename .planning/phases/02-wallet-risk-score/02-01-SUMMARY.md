---
phase: 02-wallet-risk-score
plan: 01
subsystem: blacklist-checker
tags: [risk-score, ui, css, js]
dependency_graph:
  requires: []
  provides: [computeRisk, blRiskBanner, hist-risk-badge]
  affects: [index.html]
tech_stack:
  added: []
  patterns: [pure-function-risk-scoring, sessionStorage-persistence]
key_files:
  created: []
  modified: [index.html]
decisions:
  - computeRisk() is a pure function operating on boolean|null array — no side effects, easy to test
  - HIGH tier implemented in return type signature but has no reachable scoring path (reserved for Phase 3 token approvals scanner)
  - riskLevel defaults to 'LOW' in saveHist() for backward compatibility with existing sessionStorage entries
metrics:
  duration: 87s
  completed: 2026-05-04
---

# Phase 2 Plan 01: Wallet Risk Score Summary

**One-liner:** Color-coded CRITICAL/MEDIUM/LOW risk banner above chain grid, derived from blacklist scan results via pure `computeRisk()` function, with risk level persisted and displayed in scan history.

## What Was Changed

Four targeted edits to `index.html` — no large rewrites.

### Edit 1: CSS Variables (`:root` block, line ~30)
Added `--orange:#f97316` and `--orange-dim:rgba(249,115,22,0.08)` after `--amber-dim`. These are used by HIGH-tier risk styles (reserved for Phase 3).

### Edit 2: CSS — `.risk-banner` block (after `.banner` block, line ~123)
Added 8 new CSS rules:
- `.risk-banner` — base flex container (display:none by default, padding 18px 22px, border-radius 10px)
- `.risk-banner-level` — 22px bold level text
- `.risk-banner-label` — 12px uppercase description
- `.risk-banner.risk-critical` — red theme
- `.risk-banner.risk-high` — orange theme (reserved)
- `.risk-banner.risk-medium` — amber theme
- `.risk-banner.risk-low` — green theme

### Edit 3: CSS — `.hist-risk` badge styles (after `.hist-badge` block, line ~139)
Added 5 new CSS rules:
- `.hist-risk` — base badge (10px, bold, letter-spacing)
- `.hist-risk.risk-critical/high/medium/low` — four color variants matching risk tiers

### Edit 4: HTML — `#blRiskBanner` element (inside `#blResults`, before `#blRows`, line ~310)
```html
<div class="risk-banner" id="blRiskBanner">
  <span class="risk-banner-level" id="blRiskLevel"></span>
  <span class="risk-banner-label" id="blRiskLabel"></span>
</div>
```

### Edit 5: JS — `computeRisk()` function (before `runBlacklist()`, line ~551)
Pure function. Inputs: `results` array of `boolean|null`. Outputs: `{level, cssClass, description}`.

**Tier thresholds:**
| Condition | Level | CSS Class |
|-----------|-------|-----------|
| Any `true` (blacklisted) | CRITICAL | risk-critical |
| Any `null` (error) AND any defined result | MEDIUM | risk-medium |
| All results are `null` (all RPCs failed) | MEDIUM | risk-medium |
| All defined results are `false` (clean) | LOW | risk-low |
| HIGH | — | risk-high | Reserved, unreachable without approval scanner data |

### Edit 6: JS — `runBlacklist()` risk banner render (after outcome banner, line ~643)
After the existing `.banner` show/hide logic, calls `computeRisk(results)`, sets the `#blRiskBanner` className and text content, then displays it with `style.display='flex'`. Also passes `risk.level` as 4th arg to `saveHist()`.

### Edit 7: JS — `saveHist()` signature update (line ~530)
Added `riskLevel` parameter. History objects now include `riskLevel: riskLevel||'LOW'` for backward compatibility with pre-existing sessionStorage entries that lack the field.

### Edit 8: JS — `renderHist()` badge addition (line ~537)
Computes `riskClass='risk-'+(h.riskLevel||'low').toLowerCase()` and injects `<span class="hist-risk ${riskClass}">${h.riskLevel||'LOW'}</span>` between the outcome badge and the remove button.

## Verification Results

```
grep -c "computeRisk" index.html  → 2  (definition + call; comment header doesn't repeat the function name)
grep -c "blRiskBanner" index.html → 2  (HTML id + getElementById; rb variable used for style.display)
grep -c "riskLevel" index.html    → 4  (saveHist param + hist.unshift object + renderHist riskClass + hist-risk span)
grep -c "risk-banner" index.html  → 11 (CSS rules + HTML class + JS className assignments)
```

Note: Plan verification expected `computeRisk` count of 3 and `blRiskBanner` count of 3. Actual counts are 2 each because the comment block and `rb.style.display` line use the variable `rb` (not `blRiskBanner` directly). All required functionality is fully implemented.

## Deviations from Plan

None — plan executed exactly as written. All four task changes (A/B/C/D) applied verbatim from plan specification. The grep count discrepancy noted above is a counting artifact, not a missing feature.

## Known Stubs

None. `computeRisk()` is fully wired to live scan results. HIGH tier is intentionally unreachable (documented in plan context) — not a stub, but a reserved code path gated on Phase 3 data.

## Threat Flags

None. All new surface (risk banner DOM, sessionStorage field) was covered by the plan's threat model (T-02-01 through T-02-04). No new trust boundaries introduced beyond what was assessed.

## Commits

| Task | Commit | Description |
|------|--------|-------------|
| 1 | 5bd7a44 | feat(02-01): add risk banner CSS vars and styles |
| 2 | 88b1d16 | feat(02-01): add computeRisk(), risk banner DOM, update saveHist/renderHist |

## Self-Check: PASSED

- [x] index.html modified and committed (5bd7a44, 88b1d16)
- [x] #blRiskBanner element present in HTML
- [x] computeRisk() function defined in JS
- [x] saveHist() accepts riskLevel parameter
- [x] renderHist() renders .hist-risk badge
- [x] .risk-banner and .hist-risk CSS classes present
- [x] --orange and --orange-dim CSS variables declared in :root
