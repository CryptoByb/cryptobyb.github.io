# Requirements — CryptoByB Stablecoin Blacklist Checker

## v1 Requirements

### UI & Design
- [x] **UI-01**: Tool renders with a premium dark design that feels hand-crafted and distinctive to CryptoByB — not a generic AI-generated template *(Validated Phase 1)*
- [x] **UI-02**: Existing brand identity is preserved and strengthened: JetBrains Mono, dark bg (#07080f), blue accent (#4f8ef7) *(Validated Phase 1)*
- [x] **UI-03**: UI is fully responsive and works well on mobile (current version breaks on small screens) *(Validated Phase 1)*
- [x] **UI-04**: Tab navigation (Blacklist / Approvals / TX Tracer / Batch) is clear and consistent *(Validated Phase 1)*
- [x] **UI-05**: Loading states, scan animations, and result transitions feel polished and purposeful *(Validated Phase 1)*
- [x] **UI-06**: Result cards/rows communicate status (CLEAN / BLACKLISTED / ERROR) at a glance without requiring the user to read *(Validated Phase 1)*

### Wallet Risk Score
- [ ] **RISK-01**: After all checks complete, a single risk rating (LOW / MEDIUM / HIGH / CRITICAL) is displayed prominently
- [ ] **RISK-02**: Risk rating is derived from: blacklist status + unlimited approval count + partial error states
- [ ] **RISK-03**: Risk badge is visible on history items so past checks are immediately readable

### Chain Coverage
- [x] **CHAIN-01**: Blacklist tab checks BSC, Polygon, Arbitrum in addition to current chains (align with approvals coverage) *(Closed as already satisfied — Phase 3)*

## v2 Requirements (Deferred)

- Export results as PDF or CSV
- Dark/light mode toggle
- ENS / domain name resolution (enter `vitalik.eth` instead of `0x…`)
- Webhook alerts when a saved address gets blacklisted
- Embeddable widget for third-party sites

## Out of Scope

- Backend / server-side logic — privacy guarantee requires client-only
- User accounts / saved wallets — not needed for v1
- Generic AI-template redesign (soft purples, gradient cards) — explicitly rejected by user
- Build tools / npm / frameworks — single HTML file constraint

## Traceability

| REQ-ID | Phase |
|--------|-------|
| UI-01 – UI-06 | Phase 1: UI Overhaul |
| RISK-01 – RISK-03 | Phase 2: Wallet Risk Score |
| CHAIN-01 | Phase 3: Chain Alignment |
