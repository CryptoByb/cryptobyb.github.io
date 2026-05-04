---
phase: 01-ui-overhaul
plan: "01"
subsystem: page-chrome
tags: [css, header, nav, segmented-control, branding]
dependency_graph:
  requires: []
  provides: [nav-seg, seg-tab, logo-mark, logo-text, css-variables-v2]
  affects: [index.html]
tech_stack:
  added: []
  patterns: [segmented-control, css-custom-properties, single-file-html]
key_files:
  created: []
  modified:
    - StableCoin Blacklist Checker/index.html
decisions:
  - "Kept data-tab='approvals' (not 'tab-approvals') consistent with existing tab pattern; JS constructs 'tab-'+btn.dataset.tab at runtime"
  - "Responsive breakpoint updated from .nav-tab to .seg-tab with tighter padding (6px) for four-tab fit on mobile"
metrics:
  duration: "~10 minutes"
  completed: "2026-05-04"
  tasks_completed: 2
  tasks_total: 2
---

# Phase 01 Plan 01: CSS Variables + Header + Segmented Nav Summary

One-liner: Hardened CSS variables to solid-hex spec, replaced border-bottom nav with pill segmented control, updated header to compact logo-mark design with four-tab navigation including Token Approvals shell.

## What Changed

### CSS Variables (`:root`)
- `--surface` updated from `#0d0e1a` to `#0d0e17`
- `--surface2` updated from `#13141f` to `#111220`
- `--surface3` updated from `#1a1b2a` to `#1a1b2e`
- `--border` changed from `rgba(79,142,247,0.07)` (transparent blue) to `#1a1b2e` (solid hex)
- `--border-glow` added as `rgba(79,142,247,0.18)` for glow effects
- `--muted` darkened from `#64748b` to `#4a5568`
- `--muted2` darkened from `#94a3b8` to `#718096`
- All `*-dim` alpha values tightened from `0.1` to `0.08`
- `--sans` variable removed (Inter no longer needed as primary font)
- `--r` updated from `10px` to `8px`

### Body
- `font-family` switched from `var(--sans)` to `var(--mono)` — JetBrains Mono is now the primary font throughout

### Layout
- `.wrap` max-width increased from `800px` to `860px`

### Header
- `.hdr` gains `padding-bottom:24px` and `border-bottom:1px solid var(--border)` separator line
- New `.logo-mark` — solid `#4f8ef7` blue square (32×32px, border-radius:6px) with dark `#07080f` "B" text
- New `.logo-text` wrapper with `.logo-name` (15px, weight 600) and `.logo-tag` (9px, 0.12em letter-spacing, uppercase)

### Navigation
- Replaced `.nav` / `.nav-tab` with `.nav-seg` / `.seg-tab` segmented pill control
- `.seg-tab.active` uses solid `var(--accent)` background with `#07080f` dark text (not outline/border style)
- Inactive tabs: transparent background, `var(--muted2)` text
- Responsive: `.seg-tab` font-size reduced to `10px` / padding `8px 6px` at ≤520px

### HTML Structure
- `<header>` uses new `.logo-mark` and `.logo-text` markup
- `<nav class="nav-seg">` replaces `<nav class="nav">`
- Four tabs: Blacklist Check (active), Token Approvals, TX Tracer, Batch Check
- `#tab-approvals` placeholder div added with dashed border "coming soon" message

### JavaScript
- Both `querySelectorAll('.nav-tab')` calls updated to `querySelectorAll('.seg-tab')`
- All other JS untouched — tab switching, blacklist check, TX tracer, batch check fully intact

## New Class Names Introduced

| Class | Purpose |
|-------|---------|
| `.nav-seg` | Segmented tab container (pill background) |
| `.seg-tab` | Individual tab button within segmented control |
| `.seg-tab.active` | Active tab state — solid blue fill, dark text |
| `.logo-mark` | Solid blue square logo icon |
| `.logo-text` | Wrapper for logo name + tagline |

## Removed Class Names

| Class | Replaced By |
|-------|------------|
| `.nav` | `.nav-seg` |
| `.nav-tab` | `.seg-tab` |
| `.logo-icon` | `.logo-mark` |

## Deviations from Plan

### Minor: tab-approvals grep count

The plan's acceptance criterion `grep -c 'tab-approvals'` expected >=2. In the source, `id="tab-approvals"` appears once. The nav button correctly uses `data-tab="approvals"` (matching the pattern of all other tabs: `data-tab="blacklist"` pairs with `id="tab-blacklist"`). The JS handler constructs `'tab-'+btn.dataset.tab` at runtime. This is functionally correct and consistent with the existing pattern — the grep criterion in the plan assumed the full string `tab-approvals` would appear in both places, but the established convention only puts it in the id.

**Impact:** Zero. Tab switching works correctly for all four tabs.

## Known Stubs

| Stub | File | Reason |
|------|------|--------|
| `#tab-approvals` "coming soon" placeholder | `index.html` line ~272 | Token Approvals scanner is Phase 2 scope — intentional shell per plan spec |

## Threat Flags

None. This plan modifies only CSS classes, HTML structure (class names, layout), and a JS selector string. No new network endpoints, auth paths, file access patterns, or schema changes introduced.

## Self-Check

- [x] `index.html` exists and modified
- [x] Commit `e6b9778` — Task 1 CSS variables + header/nav styles
- [x] Commit `5dc03f9` — Task 2 HTML header, four-tab nav, Approvals shell, JS selector
- [x] `.seg-tab.active{background:var(--accent);color:#07080f` present in CSS
- [x] `--border:#1a1b2e` present in `:root`
- [x] `class="seg-tab active" data-tab="blacklist"` present in HTML
- [x] `querySelectorAll('.seg-tab')` present in JS (both occurrences)
- [x] No `class="nav-tab"` or `class="nav"` remaining in file
