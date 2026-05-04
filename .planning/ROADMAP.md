# Roadmap — CryptoByB Stablecoin Blacklist Checker

**3 phases** | **10 requirements mapped** | All v1 requirements covered ✓

## Phase Overview

| # | Phase | Goal | Requirements | Success Criteria |
|---|-------|------|--------------|-----------------|
| 1 | ~~UI Overhaul~~ | ~~Make the tool look like a real product users trust and share~~ | UI-01 – UI-06 | ✓ Complete (2026-05-04) |
| 2 | ~~Wallet Risk Score~~ | ~~Give users one clear answer: is this wallet safe?~~ | RISK-01 – RISK-03 | ✓ Complete (2026-05-04) |
| 3 | Chain Alignment | Complete coverage — blacklist check matches approvals scanner | CHAIN-01 | 2 criteria |

---

## Phase 1: UI Overhaul

**Goal:** Redesign `index.html` with a premium dark aesthetic that is distinctive to CryptoByB — hand-crafted feel, not a generic AI template. Preserve and strengthen the existing identity.

**Requirements:** UI-01, UI-02, UI-03, UI-04, UI-05, UI-06

**UI hint**: yes

**Design constraints:**
- Must NOT look like a generic Claude/AI-designed site
- Build on existing identity: JetBrains Mono, #07080f bg, #4f8ef7 accent
- No soft purples, no floating gradient cards, no Inter-everywhere
- Reference: Linear, Vercel dashboard, Etherscan — sharp, purposeful, data-forward

**Key changes:**
- Hero / header section with clear value proposition
- Tighter, more intentional layout with better visual hierarchy
- Improved tab design — active state clearly distinct
- Result rows redesigned — status readable at a glance
- Mobile layout fixed — no horizontal overflow, touch-friendly inputs
- Scan animation polished — not just a spinner, feels alive

**Success criteria:**
1. User can read blacklist result (CLEAN/BLACKLISTED) within 1 second of scan completing — no squinting
2. Tool looks credible enough to share publicly on Twitter/Telegram
3. Mobile layout renders correctly on 375px viewport with no horizontal scroll
4. Tab switching feels instant and visually clear
5. Loading states communicate progress without feeling like a stock spinner

**Plans:** 3 plans

Plans:
- [x] 01-01-PLAN.md — Header + segmented tab control + CSS variable overhaul (UI-01, UI-02, UI-04)
- [x] 01-02-PLAN.md — Chain result grid blocks + scan skeleton + sequential reveal animation (UI-05, UI-06)
- [x] 01-03-PLAN.md — Mobile responsive CSS + human visual verification checkpoint (UI-01, UI-03)

---

## Phase 2: Wallet Risk Score

**Goal:** After checks complete, surface a single risk rating (LOW / MEDIUM / HIGH / CRITICAL) that combines all signals into one answer users can act on immediately.

**Requirements:** RISK-01, RISK-02, RISK-03

**Scoring logic:**
- CRITICAL: Blacklisted on any chain
- HIGH: 1+ unlimited approval detected
- MEDIUM: Limited (non-zero) approvals detected, or partial errors
- LOW: No blacklist, no approvals, all chains responded

**Key changes:**
- Risk badge displayed prominently after scan (above result rows)
- Badge color: CRITICAL=red, HIGH=orange, MEDIUM=amber, LOW=green
- History items include risk badge so past lookups are scannable

**Plans:** 1 plan

Plans:
- [x] 02-01-PLAN.md — CSS risk-banner styles + computeRisk() + history badge (RISK-01, RISK-02, RISK-03)

**Success criteria:**
1. Risk score appears within 500ms of all chain checks completing
2. CRITICAL correctly triggers when any chain returns blacklisted=true
3. History list shows risk badge for all past checks

---

## Phase 3: Chain Alignment

**Goal:** The blacklist check tab covers the same chains as the approvals scanner — BSC, Polygon, Arbitrum added to blacklist tab.

**Requirements:** CHAIN-01

**Key changes:**
- Add BSC USDT/USDC contract addresses + RPC endpoints to CHAINS config
- Add Polygon USDT/USDC to blacklist tab (already in approvals, add to main check)
- Arbitrum already in blacklist — verify parity with approvals config
- Update "Covered chains" info card to reflect expanded coverage

**Success criteria:**
1. Blacklist tab checks BSC, Polygon, Arbitrum for both USDT and USDC
2. "Covered chains" info section accurately lists all checked chains

---

## Execution Order

Phase 1 → Phase 2 → Phase 3

Phase 1 is the highest-impact change (visual credibility). Phase 2 adds intelligence on top of the existing data. Phase 3 is a targeted config expansion.
