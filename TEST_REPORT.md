# Dashboard (index.html) - Test Report

| Item | Value |
|---|---|
| Tested file | `index.html` (same path that is delivered) |
| **SHA-256 of tested index.html** | `409ef0898269205ad32952ff50705ea28d7f5edd66a493ce05cb30b5e49b27e7` |
| SHA-256 before the run / after the run / delivered file | identical (`409ef0898269205a...`) |
| Browser | HeadlessChrome/153.0.8010.0 (real headless Chromium, puppeteer-core) |
| Data | synthetic only; no real clinic data was available |
| Run (container clock) | 2026-11-22 03:17 UTC |
| Finance reference | 16 real functions extracted from `finance_6.html` (`td, ks, nf, load, normExp, grNorm, paidOf, stOf, gOk, gPaid, exOf, dLeft, st, addPm, fin, gDash`) and executed in a Node VM; Finance's DOM is stubbed |
| Shared starter used for the byte comparison | `01_PW_SHARED_STARTER.html` (the uploaded file; it has no "(4)" in its name) |

## 1. Result

| Total checks | Passed | Failed | Skipped |
|---|---|---|---|
| **326** | **324** | **0** | **2** |

Every check below records an Expected and an Actual value. Nothing is marked as passed unless it was executed. The 2 skipped checks are listed in section 8.

| Group | Checks | Passed | Failed | Skipped |
|---|---|---|---|---|
| 01 Static checks | 9 | 9 | 0 | 0 |
| 02 Supplier Payables vs real Finance gDash() | 17 | 17 | 0 | 0 |
| 03 Differential vs real Finance code (random) | 6 | 6 | 0 | 0 |
| 04 Inventory numeric normalization | 26 | 26 | 0 | 0 |
| 05 Role storage access (7 roles) | 46 | 46 | 0 | 0 |
| 06 Role visibility and Doctor/Staff finance isolation | 31 | 31 | 0 | 0 |
| 07 Section failure isolation | 14 | 14 | 0 | 0 |
| 08 Cross-tab role / permission refresh (two real tabs, same browser profile) | 29 | 27 | 0 | 2 |
| 09 Read-only verification | 3 | 3 | 0 | 0 |
| 10 Empty and corrupt storage | 16 | 16 | 0 | 0 |
| 11 Responsive viewports | 62 | 62 | 0 | 0 |
| 12 Dark / light theme | 66 | 66 | 0 | 0 |
| 13 Delivery integrity | 1 | 1 | 0 | 0 |

## 2. What was changed in this round (3 issues only)

1. **Supplier Payables** now follows Finance `gDash()` exactly: every receipt that passes `gOk()` (Cancelled, Void, Returned excluded; **Pending included**), amount = `total - gPaid(g)`, with the `grNorm` legacy rule (no `payments` and `g.paid > 0` means one payment of `g.paid`). The card sub-line now says "pending included".
2. **Inventory numbers** go through one helper, `num()`: null, "", undefined, NaN, Infinity, junk text and booleans become 0; numeric text such as "5" becomes 5; negative numbers are kept (Finance treats qty <= 0 as Out). It is used for quantity, minimum, batch quantity and cost in the status, the alert table and the stock valuation. The `isFinite(i.qty)` shortcut is gone. Stored data is never modified.
3. **Cross-tab role/permission refresh.** The shared code reads `pwSession` and `pwRolePerms` only once at page load. The Dashboard now remembers what the shared code saw, and when either key differs later it hides the content ("Permissions changed - reloading...") and reloads once (120 ms debounce) so the shared code re-reads them. This check runs on every storage event for those keys, `localStorage.clear()`, visibility change, page restore (`pageshow` persisted), the Refresh button and the 60-second timer. No `sessionStorage`/`localStorage` write is used as a loop guard; the comparison against the values read at load makes a reload loop impossible.

Nothing else was redesigned. Shared CSS/JS, layout, English UI, `.dash-` prefix, role visibility and read-only behavior are unchanged. No other module was touched. The file name is still `index.html`.

### Decision you should confirm: two different "payables" numbers exist inside Finance

For the same data Finance shows two numbers. The Goods Receipt dashboard card **OUTSTANDING** (`gDash`) includes Pending receipts. The separate **Supplier Payables** report counts Received receipts only. For your known case (Received 100,000 / paid 40,000; Pending 50,000 / paid 10,000) the first gives **100,000 Ks** and the second gives **60,000 Ks**. You asked for 100,000 Ks, so the Dashboard now matches `gDash`. This replaces the earlier approved "Received only" rule. The card is still labelled "Supplier payables"; say so if you want it renamed.

## 3. Before / after, executed on the previous build and on this build

| Issue | Previous build (executed) | This build (executed) |
|---|---|---|
| Supplier payables, known case | 60,000 Ks | 100,000 Ks |
| Inventory qty `null` / `""` in the alert table | shows "null Box" and a blank | shows "0 Box" and "0 Box" |
| Admin tab open, another tab sets role to Staff | still shows Admin with finance and patient activity after 2.5 s, 0 reloads | shows Staff, finance and patient activity gone, 1 reload |

## 4. Shared CSS/JS comparison with the starter

| Block | Result | Size | SHA-256 of block |
|---|---|---|---|
| PW-SHARED CSS | **byte-for-byte identical** | 46302 bytes | `2fc33317284d66f7416faa096473b2818d220b5d9ed71d244d124ed9918d1726` |
| PW-SHARED JS (v2) | **byte-for-byte identical** | 5159 bytes | `c9ca34c8d98d5d98ce63540a055b954eb13940c59eecf427028d0552db79c54f` |

The 8 uploaded reference files (starter, Patient, Finance, Services, Communication, 3 documents) have the same SHA-256 as when first read.

## 5. Supplier Payables: Finance `gDash()` vs Dashboard

Finance column = the real `gDash()` run in the VM (OUTSTANDING card value and receipt count). "Finance Supplier Payables report" = the Received-only expression, shown for information only; the Dashboard does not use it.

| Case | Finance gDash | Dashboard | Expected | Finance Supplier Payables report (Received only, info) | Result |
|---|---|---|---|---|---|
| K1 user case: Received 100,000 paid 40,000 + Pending 50,000 paid 10,000 | 100,000 Ks / 2 receipts | 100,000 Ks / 2 receipts | 100,000 Ks | 60,000 Ks | PASS |
| K2 Pending only, unpaid 50,000 | 50,000 Ks / 1 receipts | 50,000 Ks / 1 receipts | 50,000 Ks | 0 Ks | PASS |
| K3 Cancelled + Void + Returned (unpaid 30,000 each) are excluded | 0 Ks / 0 receipts | 0 Ks / 0 receipts | 0 Ks | 0 Ks | PASS |
| K4 Received fully paid | 0 Ks / 0 receipts | 0 Ks / 0 receipts | 0 Ks | 0 Ks | PASS |
| K5 Received unpaid | 80,000 Ks / 1 receipts | 80,000 Ks / 1 receipts | 80,000 Ks | 80,000 Ks | PASS |
| K6 partial payments 10,000 + 15,000 of 100,000 | 75,000 Ks / 1 receipts | 75,000 Ks / 1 receipts | 75,000 Ks | 75,000 Ks | PASS |
| K7 legacy: no payments, g.paid=25,000 of 100,000 | 75,000 Ks / 1 receipts | 75,000 Ks / 1 receipts | 75,000 Ks | 75,000 Ks | PASS |
| K8 legacy: no payments, no paid, Pending 40,000 | 40,000 Ks / 1 receipts | 40,000 Ks / 1 receipts | 40,000 Ks | 0 Ks | PASS |
| K9 over-payment: total 1,000 paid 1,500 (Finance lets it go negative) | -500 Ks / 0 receipts | -500 Ks / 0 receipts | -500 Ks | -500 Ks | PASS |
| K10 mix: user case + Cancelled 70,000 + Void 1 + Returned 2 | 100,000 Ks / 2 receipts | 100,000 Ks / 2 receipts | 100,000 Ks | 60,000 Ks | PASS |
| K11 numeric-string total "100000", payment 40000 | 60,000 Ks / 1 receipts | 60,000 Ks / 1 receipts | 60,000 Ks | 60,000 Ks | PASS |
| K12 empty list | 0 Ks / 0 receipts | 0 Ks / 0 receipts | 0 Ks | 0 Ks | PASS |
| D1 payments is a string "x" (Finance throws) [documented deviation] | TypeError: (g.payments \|\| []).reduce is not a function | 1,000 Ks | 1,000 Ks | - | PASS |
| D2 null entry inside the list (Finance throws) [documented deviation] | load: TypeError | 1,000 Ks | 1,000 Ks | - | PASS |
| D3 total missing (Finance gives NaN) [documented deviation] | 0 Ks | 400 Ks | 400 Ks | - | PASS |

Random comparison: 400 random datasets (all statuses Received/Pending/Cancelled/Void/Returned; payments missing, empty, partial, full; legacy `g.paid`): Dashboard value and receipt count equal real `gDash()` in **400 of 400** datasets.

Cases D1-D3 are corrupt data where Finance itself throws or returns NaN; the Dashboard stays finite (documented deviation).

## 6. Inventory numeric edge cases

Item: minimum 10, tracked, cost 10, no batches. "Finance raw" = real Finance `load()` + `st()` on the unmodified value.

| Input | Finance raw: status / quantity it holds / valuation | Dashboard: status / quantity shown / valuation | Result | Note |
|---|---|---|---|---|
| qty: null | Out / null / 0 | Out / 0 / 0 | PASS | same as Finance |
| qty: "" | Out / '' / 0 | Out / 0 / 0 | PASS | same as Finance |
| qty: "5" | Low / 5 / 50 | Low / 5 / 50 | PASS | same as Finance |
| qty: undefined | Out / 0 / 0 | Out / 0 / 0 | PASS | same as Finance |
| qty: NaN | Out / 0 / 0 | Out / 0 / 0 | PASS | same as Finance |
| qty: 0 | Out / 0 / 0 | Out / 0 / 0 | PASS | same as Finance |
| qty: -1 | Out / -1 / 0 | Out / -1 / 0 | PASS | same as Finance |
| qty: 5 | Low / 5 / 50 | Low / 5 / 50 | PASS | same as Finance |
| qty: Infinity | Out / 0 / 0 | Out / 0 / 0 | PASS | same as Finance |
| qty: "abc" | Out / 0 / 0 | Out / 0 / 0 | PASS | same as Finance |
| qty: " 7 " | Low /  7  / 70 | Low / 7 / 70 | PASS | same as Finance |
| qty: true | Critical / true / 10 | Out / 0 / 0 | PASS | boolean = invalid number, deliberate deviation (Finance treats true as 1) |

Also executed: the same 12 inputs read through real storage (all but NaN/Infinity/undefined); a 90-combination qty x min grid; a batch edge case (null, "", "abc" and "3" quantities with text cost); a proof that the stored inventory is byte-identical after rendering (qty null stays null in storage).

**Where the Dashboard intentionally differs from raw Finance (all executed and listed):**

- Grid: G2_3 (qty "5", min "abc"): raw Finance Low, normalized OK
- Grid: G2_4 (qty "5", min "10"): raw Finance OK, normalized Low
- Grid: G7_3 (qty " 7 ", min "abc"): raw Finance Low, normalized OK
- Finance compares two text values as text, so `"5" < "10"` is false and a text quantity "5" against a text minimum "10" is not Low. The Dashboard compares numbers as you asked. In the 400 random datasets, 3 item statuses differed from raw Finance and **0** of them had an explanation other than qty and min both stored as text.
- A boolean quantity (`true`) is an invalid number, so the Dashboard shows 0 / Out; Finance treats `true` as 1 (Critical).
- Where raw Finance returns NaN for the stock valuation (a batch without a quantity; 10 of the 400 random datasets) the Dashboard shows a finite number, never NaN.

## 7. Cross-tab role / permission refresh (two real tabs in one browser profile)

| Scenario | Expected | Actual |
|---|---|---|
| Admin -> Doctor | tab reloads once, shows Doctor view, no finance | reloaded: role=doctor, finance=false, patient activity=true, loads +1 |
| Doctor -> Staff | tab reloads once, patient activity gone | role=staff, patient activity=false, visits card=false, loads +1 |
| Staff -> Owner | Dashboard tab leaves index.html (shared guard), no Dashboard data read | moved to /patients.html#appointments; business reads after change: none |
| Owner -> Admin | Dashboard opened again shows the Admin view (an Owner tab cannot hold the Dashboard open) | role=admin, finance=true |
| Admin -> unexpected role (janitor) | No access, sidebar emptied | noAccess=true, sidebar children=0 |
| Unexpected role -> Admin | full Admin view | role=admin, finance=true |
| pwRolePerms: Doctor loses index.html | tab redirected away by shared guard after reload | moved to /patients.html#appointments |
| pwRolePerms changes, Dashboard still allowed | one reload, Doctor view kept | loads +1, role=doctor |
| Burst: 3 writes in a row | one reload, final state correct | loads +1, role=admin |
| Sequential changes | one reload per change | doctor:+1, staff:+1, admin:+1 |
| Missed event + visibility | stale view detected and replaced | staff view, finance=false |
| Missed event + pageshow | stale view detected and replaced | staff view, finance=false |
| Missed event + refresh | stale view detected and replaced | staff view, finance=false |
| 60-second timer (storage handler removed) | stale view replaced within ~60 s | refreshed, role=doctor |

Also checked: at the instant of the change the stale finance content was already replaced by the "Permissions changed - reloading..." message; after the change a Doctor/Staff tab never read `pwTx`, `pwExpenses` or `pwGoodsReceipts` (Staff also not `petworld.records.v1`); a change to `pwAppts` refreshes in place with no reload; `pwTheme` and an identical rewrite cause no reload; a burst of 3 writes causes one reload; the load counter stayed constant for 2.5 s afterwards (no loop); the tab had no JavaScript error.

## 8. Skipped, and not tested

**Skipped (counted above as Skipped):**

- Owner -> Admin as a live refresh of an open Dashboard tab - An Owner tab is redirected away from index.html by the shared guard, so the Dashboard cannot stay open as Owner; tested as "open again after the change" instead
- Real back/forward-cache restore (pageshow persisted=true produced by the browser) - headless Chromium with remote debugging does not use the back/forward cache; the handler was tested with a simulated event only

**Not tested at all:**

- Browsers other than Chromium 153 (Safari/iOS, Firefox, Android WebView) and real touch devices.
- Real clinic data. All numbers come from synthetic datasets.
- A real back/forward-cache restore. Only a simulated `pageshow` event with `persisted=true` was used.
- The Finance start-up seeding of sample inventory (`seedInv`) and Finance `legacyMig`/`batchMig`/`idMig`. They were read in the code, not mirrored: they do not change any number used here, but a Finance that has never been opened keeps an empty inventory.
- The Finance Supplier Payables report and the expense/stock valuation expressions are inline report code, so they were copied into the harness by hand; `gDash`, `fin`, `st`, `load`, `normExp`, `grNorm` were run as real functions.
- Shared stat-card colour contrast (white text on the shared gradients) was not re-measured this round. The previous round computed it from the shared CSS (amber 3.19 to 2.15, green 3.77 to 2.54, cyan 4.10 to 2.43); the shared CSS is not changed.
- Real security. The role check is interface-only; anyone can edit browser storage, and Doctor/Staff can still see `pwInventory` (with costs) in their own browser storage.

## 9. Storage access observed (real browser)

Shared code (unchanged) reads `pwTheme`, `pwSession`, `pwRolePerms` for every role. The Dashboard additionally re-reads `pwSession` and `pwRolePerms` only to detect changes. "Business data" below means Dashboard data reads.

| Role | Keys read on load | Business-data keys read | Writes |
|---|---|---|---|
| admin | pwTheme, pwSession, pwRolePerms | petworld.records.v1, pwAppts, pwClinic, pwExpenses, pwGoodsReceipts, pwInventory, pwIp, pwTx | none |
| doctor | pwTheme, pwSession, pwRolePerms | petworld.records.v1, pwAppts, pwClinic, pwInventory, pwIp | none |
| staff | pwTheme, pwSession, pwRolePerms | pwAppts, pwClinic, pwInventory, pwIp | none |
| owner | pwTheme, pwSession, pwRolePerms | none | none |
| janitor | pwTheme, pwSession, pwRolePerms | none | none |
| constructor | pwTheme, pwSession, pwRolePerms | none | none |
| (no pwSession: shared default admin) | pwTheme, pwSession, pwRolePerms | petworld.records.v1, pwAppts, pwClinic, pwExpenses, pwGoodsReceipts, pwInventory, pwIp, pwTx | none |

Owner reads no business data (it is redirected by the shared guard). An unexpected role (`janitor`, `constructor`) reads no business data, has its sidebar emptied and sees "No access". Doctor and Staff never read `pwTx`, `pwExpenses` or `pwGoodsReceipts`; Staff never reads `petworld.records.v1`. The only write observed in any test was `pwTheme` after the theme button was clicked. The whole `localStorage` was byte-identical before and after load, render, Refresh, a storage event and a visibility change.

## 10. Recorded, not changed

- The unchanged shared menus (`MENU_OWNER` and `MENU_ALL`) still list Campaigns and Notes for Owner. This conflicts with the current Communication decisions. It was not touched in this work and no Communication integration was added. It must be fixed in the shared starter.
- The Dashboard cannot stay open as Owner: the shared guard redirects an Owner tab away from `index.html`.

## Appendix: every check with Expected and Actual

### 01 Static checks

| # | Check | Expected | Actual | Result |
|---|---|---|---|---|
| 1 | JavaScript syntax check (node --check on all inline script) | exit code 0 | exit code 0 | PASS |
| 2 | Shared CSS block byte-for-byte identical to 01_PW_SHARED_STARTER.html | identical | identical=true, 46302 bytes, sha256 2fc33317284d66f7416faa096473b2818d220b5d9ed71d244d124ed9918d1726 | PASS |
| 3 | Shared JS block byte-for-byte identical to 01_PW_SHARED_STARTER.html | identical | identical=true, 5159 bytes, sha256 c9ca34c8d98d5d98ce63540a055b954eb13940c59eecf427028d0552db79c54f | PASS |
| 4 | Each shared marker appears exactly once | 1 each | markers counted | PASS |
| 5 | No storage-write / network / eval API in Dashboard code (after shared block) | none found | none found | PASS |
| 6 | LS.get keys in Dashboard code are exactly the approved set | pwAppts, pwIp, pwInventory, pwClinic, petworld.records.v1, pwTx, pwExpenses, pwGoodsReceipts (+ pwSession, pwRolePerms f... | petworld.records.v1, pwAppts, pwClinic, pwExpenses, pwGoodsReceipts, pwInventory, pwIp, pwRolePerms, pwSession, pwTx | PASS |
| 7 | No unapproved key name anywhere in index.html | none | none | PASS |
| 8 | data-file="index.html", no placeholders left, file name unchanged (index.html) | true | ok | PASS |
| 9 | Uploaded reference files (starter, Patient, Finance, Services, Communication, docs) unchanged since first read (SHA-256) | all identical | all 8 identical | PASS |

### 02 Supplier Payables vs real Finance gDash()

| # | Check | Expected | Actual | Result |
|---|---|---|---|---|
| 1 | Supplier Payables K1 user case: Received 100,000 paid 40,000 + Pending 50,000 paid 10,000 | Finance=Dashboard=100000 | Finance gDash OUTSTANDING=100000 (2); Dashboard=100000 (2) | PASS |
| 2 | Supplier Payables K2 Pending only, unpaid 50,000 | Finance=Dashboard=50000 | Finance gDash OUTSTANDING=50000 (1); Dashboard=50000 (1) | PASS |
| 3 | Supplier Payables K3 Cancelled + Void + Returned (unpaid 30,000 each) are excluded | Finance=Dashboard=0 | Finance gDash OUTSTANDING=0 (0); Dashboard=0 (0) | PASS |
| 4 | Supplier Payables K4 Received fully paid | Finance=Dashboard=0 | Finance gDash OUTSTANDING=0 (0); Dashboard=0 (0) | PASS |
| 5 | Supplier Payables K5 Received unpaid | Finance=Dashboard=80000 | Finance gDash OUTSTANDING=80000 (1); Dashboard=80000 (1) | PASS |
| 6 | Supplier Payables K6 partial payments 10,000 + 15,000 of 100,000 | Finance=Dashboard=75000 | Finance gDash OUTSTANDING=75000 (1); Dashboard=75000 (1) | PASS |
| 7 | Supplier Payables K7 legacy: no payments, g.paid=25,000 of 100,000 | Finance=Dashboard=75000 | Finance gDash OUTSTANDING=75000 (1); Dashboard=75000 (1) | PASS |
| 8 | Supplier Payables K8 legacy: no payments, no paid, Pending 40,000 | Finance=Dashboard=40000 | Finance gDash OUTSTANDING=40000 (1); Dashboard=40000 (1) | PASS |
| 9 | Supplier Payables K9 over-payment: total 1,000 paid 1,500 (Finance lets it go negative) | Finance=Dashboard=-500 | Finance gDash OUTSTANDING=-500 (0); Dashboard=-500 (0) | PASS |
| 10 | Supplier Payables K10 mix: user case + Cancelled 70,000 + Void 1 + Returned 2 | Finance=Dashboard=100000 | Finance gDash OUTSTANDING=100000 (2); Dashboard=100000 (2) | PASS |
| 11 | Supplier Payables K11 numeric-string total "100000", payment 40000 | Finance=Dashboard=60000 | Finance gDash OUTSTANDING=60000 (1); Dashboard=60000 (1) | PASS |
| 12 | Supplier Payables K12 empty list | Finance=Dashboard=0 | Finance gDash OUTSTANDING=0 (0); Dashboard=0 (0) | PASS |
| 13 | Supplier Payables card shown on the Dashboard for the user case = 100,000 Ks | Supplier payables ... 100,000 Ks | Payables, purchases and stock value Open → Supplier payables 2 receipt(s) with a balance · pending included 100,000 Ks Outstanding expenses 0 partly paid or unp | PASS |
| 14 | Pending Goods Receipt is NOT left out (Pending-only data gives 50,000) | 50,000 | covered by K2: Dashboard=50,000 | PASS |
| 15 | Corrupt data D1 payments is a string "x" (Finance throws): Dashboard keeps working and returns a finite number | 1000 | Finance: TypeError: (g.payments \|\| []).reduce is not a function \| Dashboard=1000 | PASS |
| 16 | Corrupt data D2 null entry inside the list (Finance throws): Dashboard keeps working and returns a finite number | 1000 | Finance: load: TypeError \| Dashboard=1000 | PASS |
| 17 | Corrupt data D3 total missing (Finance gives NaN): Dashboard keeps working and returns a finite number | 400 | Finance: 0 Ks \| Dashboard=400 | PASS |

### 03 Differential vs real Finance code (random)

| # | Check | Expected | Actual | Result |
|---|---|---|---|---|
| 1 | Supplier Payables (value and receipt count) equals real Finance gDash() OUTSTANDING on 400 random datasets | 0 mismatching datasets | 0 mismatching datasets of 400 | PASS |
| 2 | Inventory status, earliest expiry and alert counts equal real Finance st()/exOf()/load() (run on numerically normalized qty, min, batch qty and cost) ... | {} | {} | PASS |
| 3 | Stock valuation equals the Finance valuation expression on numerically normalized data (raw Finance gives NaN in 10 of these datasets, Dashboard never... | 0 | 0 mismatches | PASS |
| 4 | Where RAW Finance differs from numerically normalized Finance (3 item statuses), every case has qty AND min stored as text (Finance then compares them... | 0 unexplained | 0 unexplained of 3 | PASS |
| 5 | fin() (today, month, 6 trend months incl. expense categories) equals real Finance fin() | identical | ok | PASS |
| 6 | Outstanding expenses equals Finance expDash formula (paidOf + normExp) | 0 | 0 mismatches | PASS |

### 04 Inventory numeric normalization

| # | Check | Expected | Actual | Result |
|---|---|---|---|---|
| 1 | num() helper: null,"", " ","5"," 7 ",undefined,NaN,Infinity,-Infinity,0,-1,5,2.5,"abc","1e2",true,false,{},[]  | [0,0,0,5,7,0,0,0,0,0,-1,5,2.5,0,100,0,0,0,0] | [0,0,0,5,7,0,0,0,0,0,-1,5,2.5,0,100,0,0,0,0] | PASS |
| 2 | qty null (min 10): status, shown quantity, valuation, no null/undefined/NaN text | status=Out, shown=0, valuation=0, badText=false | status=Out, shown=0, valuation=0, badText=false \| Finance: status=Out, raw shown=null, valuation=0 | PASS |
| 3 | qty null read through real storage: same status and valuation, no bad text | status=Out, valuation=0, badText=false | status=Out, valuation=0, badText=false | PASS |
| 4 | qty "" (min 10): status, shown quantity, valuation, no null/undefined/NaN text | status=Out, shown=0, valuation=0, badText=false | status=Out, shown=0, valuation=0, badText=false \| Finance: status=Out, raw shown=, valuation=0 | PASS |
| 5 | qty "" read through real storage: same status and valuation, no bad text | status=Out, valuation=0, badText=false | status=Out, valuation=0, badText=false | PASS |
| 6 | qty "5" (min 10): status, shown quantity, valuation, no null/undefined/NaN text | status=Low, shown=5, valuation=50, badText=false | status=Low, shown=5, valuation=50, badText=false \| Finance: status=Low, raw shown=5, valuation=50 | PASS |
| 7 | qty "5" read through real storage: same status and valuation, no bad text | status=Low, valuation=50, badText=false | status=Low, valuation=50, badText=false | PASS |
| 8 | qty undefined (min 10): status, shown quantity, valuation, no null/undefined/NaN text | status=Out, shown=0, valuation=0, badText=false | status=Out, shown=0, valuation=0, badText=false \| Finance: status=Out, raw shown=0, valuation=0 | PASS |
| 9 | qty NaN (min 10): status, shown quantity, valuation, no null/undefined/NaN text | status=Out, shown=0, valuation=0, badText=false | status=Out, shown=0, valuation=0, badText=false \| Finance: status=Out, raw shown=0, valuation=0 | PASS |
| 10 | qty 0 (min 10): status, shown quantity, valuation, no null/undefined/NaN text | status=Out, shown=0, valuation=0, badText=false | status=Out, shown=0, valuation=0, badText=false \| Finance: status=Out, raw shown=0, valuation=0 | PASS |
| 11 | qty 0 read through real storage: same status and valuation, no bad text | status=Out, valuation=0, badText=false | status=Out, valuation=0, badText=false | PASS |
| 12 | qty -1 (min 10): status, shown quantity, valuation, no null/undefined/NaN text | status=Out, shown=-1, valuation=0, badText=false | status=Out, shown=-1, valuation=0, badText=false \| Finance: status=Out, raw shown=-1, valuation=0 | PASS |
| 13 | qty -1 read through real storage: same status and valuation, no bad text | status=Out, valuation=0, badText=false | status=Out, valuation=0, badText=false | PASS |
| 14 | qty 5 (min 10): status, shown quantity, valuation, no null/undefined/NaN text | status=Low, shown=5, valuation=50, badText=false | status=Low, shown=5, valuation=50, badText=false \| Finance: status=Low, raw shown=5, valuation=50 | PASS |
| 15 | qty 5 read through real storage: same status and valuation, no bad text | status=Low, valuation=50, badText=false | status=Low, valuation=50, badText=false | PASS |
| 16 | qty Infinity (min 10): status, shown quantity, valuation, no null/undefined/NaN text | status=Out, shown=0, valuation=0, badText=false | status=Out, shown=0, valuation=0, badText=false \| Finance: status=Out, raw shown=0, valuation=0 | PASS |
| 17 | qty "abc" (min 10): status, shown quantity, valuation, no null/undefined/NaN text | status=Out, shown=0, valuation=0, badText=false | status=Out, shown=0, valuation=0, badText=false \| Finance: status=Out, raw shown=0, valuation=0 | PASS |
| 18 | qty "abc" read through real storage: same status and valuation, no bad text | status=Out, valuation=0, badText=false | status=Out, valuation=0, badText=false | PASS |
| 19 | qty " 7 " (min 10): status, shown quantity, valuation, no null/undefined/NaN text | status=Low, shown=7, valuation=70, badText=false | status=Low, shown=7, valuation=70, badText=false \| Finance: status=Low, raw shown= 7 , valuation=70 | PASS |
| 20 | qty " 7 " read through real storage: same status and valuation, no bad text | status=Low, valuation=70, badText=false | status=Low, valuation=70, badText=false | PASS |
| 21 | qty true (min 10): status, shown quantity, valuation, no null/undefined/NaN text | status=Out, shown=0, valuation=0, badText=false | status=Out, shown=0, valuation=0, badText=false \| Finance: status=Critical, raw shown=true, valuation=10 | PASS |
| 22 | qty true read through real storage: same status and valuation, no bad text | status=Out, valuation=0, badText=false | status=Out, valuation=0, badText=false | PASS |
| 23 | Grid of 90 qty x min combinations: every Dashboard status equals real Finance st() on numerically normalized data | 0 mismatches | 0 mismatches | PASS |
| 24 | Grid: combinations where RAW Finance differs from numeric comparison are only those with qty AND min stored as text (Finance compares text: "5" < "10"... | only text-vs-text combinations | 3 combination(s): G2_3 (qty "5", min "abc"): raw Finance Low, normalized OK \| G2_4 (qty "5", min "10"): raw Finance OK, normalized Low \| G7_3 (qty " 7 ", min "abc"): raw Finance Low, normalized OK | PASS |
| 25 | Batch edge case (qty null / "" / "abc" / "3" with string cost): earliest expiry and valuation equal Finance | same expiry; valuation NaN | expiry=2026-12-02 (Finance 2026-12-02), valuation=21 (Finance NaN) | PASS |
| 26 | Dashboard did not modify the stored inventory data (qty null / min "x" stays exactly as stored) | unchanged=true | unchanged=true | PASS |

### 05 Role storage access (7 roles)

| # | Check | Expected | Actual | Result |
|---|---|---|---|---|
| 1 | admin: business-data keys read are exactly the permitted set | petworld.records.v1, pwAppts, pwClinic, pwExpenses, pwGoodsReceipts, pwInventory, pwIp, pwTx | petworld.records.v1, pwAppts, pwClinic, pwExpenses, pwGoodsReceipts, pwInventory, pwIp, pwTx | PASS |
| 2 | admin: no other key read (stock ledger, Communication, Services, pending bills, archive, finance settings, audit log) | none | none | PASS |
| 3 | admin: zero writes/removals on load | [] | [] | PASS |
| 4 | admin: theme toggle writes only pwTheme | [["set","pwTheme"]] | [["set","pwTheme"]] | PASS |
| 5 | admin: cost-reading function invCosted() runs for Admin | >0 | calls=1 | PASS |
| 6 | admin: re-render and Refresh button write nothing (only the earlier theme click) | ["pwTheme"] | ["pwTheme"] | PASS |
| 7 | admin: no JavaScript errors | none | none | PASS |
| 8 | doctor: business-data keys read are exactly the permitted set | petworld.records.v1, pwAppts, pwClinic, pwInventory, pwIp | petworld.records.v1, pwAppts, pwClinic, pwInventory, pwIp | PASS |
| 9 | doctor: no other key read (stock ledger, Communication, Services, pending bills, archive, finance settings, audit log) | none | none | PASS |
| 10 | doctor: zero writes/removals on load | [] | [] | PASS |
| 11 | doctor: never read pwTx / pwExpenses / pwGoodsReceipts | none read | none read | PASS |
| 12 | doctor: theme toggle writes only pwTheme | [["set","pwTheme"]] | [["set","pwTheme"]] | PASS |
| 13 | doctor: cost-reading function invCosted() is never called | 0 | calls=0 | PASS |
| 14 | doctor: re-render and Refresh button write nothing (only the earlier theme click) | ["pwTheme"] | ["pwTheme"] | PASS |
| 15 | doctor: no JavaScript errors | none | none | PASS |
| 16 | staff: business-data keys read are exactly the permitted set | pwAppts, pwClinic, pwInventory, pwIp | pwAppts, pwClinic, pwInventory, pwIp | PASS |
| 17 | staff: no other key read (stock ledger, Communication, Services, pending bills, archive, finance settings, audit log) | none | none | PASS |
| 18 | staff: zero writes/removals on load | [] | [] | PASS |
| 19 | staff: never read pwTx / pwExpenses / pwGoodsReceipts | none read | none read | PASS |
| 20 | staff: never read petworld.records.v1 | not read | not read | PASS |
| 21 | staff: theme toggle writes only pwTheme | [["set","pwTheme"]] | [["set","pwTheme"]] | PASS |
| 22 | staff: cost-reading function invCosted() is never called | 0 | calls=0 | PASS |
| 23 | staff: re-render and Refresh button write nothing (only the earlier theme click) | ["pwTheme"] | ["pwTheme"] | PASS |
| 24 | staff: no JavaScript errors | none | none | PASS |
| 25 | owner: business-data keys read are exactly the permitted set | none | none | PASS |
| 26 | owner: no other key read (stock ledger, Communication, Services, pending bills, archive, finance settings, audit log) | none | none | PASS |
| 27 | owner: zero writes/removals on load | [] | [] | PASS |
| 28 | owner: shared guard navigated away from index.html | .../patients.html | http://127.0.0.1:33769/patients.html#appointments | PASS |
| 29 | janitor: business-data keys read are exactly the permitted set | none | none | PASS |
| 30 | janitor: no other key read (stock ledger, Communication, Services, pending bills, archive, finance settings, audit log) | none | none | PASS |
| 31 | janitor: zero writes/removals on load | [] | [] | PASS |
| 32 | janitor (unsupported role): sidebar emptied and hidden, hamburger hidden, no content shift | {sbChildren:0,sbDisplay:none,hamDisplay:none,marginLeft:0px} | {"sbChildren":0,"sbDisplay":"none","hamDisplay":"none","marginLeft":"0px"} | PASS |
| 33 | janitor (unsupported role): "No access" message, no dashboard data | No access ... | No access The Dashboard is available to Admin, Doctor and St | PASS |
| 34 | janitor: theme toggle writes only pwTheme | [["set","pwTheme"]] | [["set","pwTheme"]] | PASS |
| 35 | janitor: no JavaScript errors | none | none | PASS |
| 36 | constructor: business-data keys read are exactly the permitted set | none | none | PASS |
| 37 | constructor: no other key read (stock ledger, Communication, Services, pending bills, archive, finance settings, audit log) | none | none | PASS |
| 38 | constructor: zero writes/removals on load | [] | [] | PASS |
| 39 | constructor (unsupported role): sidebar emptied and hidden, hamburger hidden, no content shift | {sbChildren:0,sbDisplay:none,hamDisplay:none,marginLeft:0px} | {"sbChildren":0,"sbDisplay":"none","hamDisplay":"none","marginLeft":"0px"} | PASS |
| 40 | constructor (unsupported role): "No access" message, no dashboard data | No access ... | No access The Dashboard is available to Admin, Doctor and St | PASS |
| 41 | constructor: no JavaScript errors | none | none | PASS |
| 42 | (no pwSession: shared default admin): business-data keys read are exactly the permitted set | petworld.records.v1, pwAppts, pwClinic, pwExpenses, pwGoodsReceipts, pwInventory, pwIp, pwTx | petworld.records.v1, pwAppts, pwClinic, pwExpenses, pwGoodsReceipts, pwInventory, pwIp, pwTx | PASS |
| 43 | (no pwSession: shared default admin): no other key read (stock ledger, Communication, Services, pending bills, archive, finance settings, audit log) | none | none | PASS |
| 44 | (no pwSession: shared default admin): zero writes/removals on load | [] | [] | PASS |
| 45 | (no pwSession: shared default admin): theme toggle writes only pwTheme | [["set","pwTheme"]] | [["set","pwTheme"]] | PASS |
| 46 | (no pwSession: shared default admin): no JavaScript errors | none | none | PASS |

### 06 Role visibility and Doctor/Staff finance isolation

| # | Check | Expected | Actual | Result |
|---|---|---|---|---|
| 1 | admin: clinic name is escaped (no HTML injection) | escaped | escaped | PASS |
| 2 | admin card apptToday | 3 | 3 | PASS |
| 3 | admin card apptPast | 1 | 1 | PASS |
| 4 | admin card ipNow | 2 | 2 | PASS |
| 5 | admin card invTotal | 5 | 5 | PASS |
| 6 | admin card visitsMonth | 2 | 2 | PASS |
| 7 | admin card netSalesToday | 15,000 Ks | 15,000 Ks | PASS |
| 8 | admin card netSalesMonth | 15,000 Ks | 15,000 Ks | PASS |
| 9 | admin card refundsMonth | 9,000 Ks | 9,000 Ks | PASS |
| 10 | admin card grossMonth | 9,300 Ks | 9,300 Ks | PASS |
| 11 | admin card expensesMonth | 5,700 Ks | 5,700 Ks | PASS |
| 12 | admin card profitMonth | 3,600 Ks | 3,600 Ks | PASS |
| 13 | admin card invLow | 2 | 2 | PASS |
| 14 | admin card invExpired | 1 | 1 | PASS |
| 15 | admin card invSoon | 1 | 1 | PASS |
| 16 | admin card invOut | 1 | 1 | PASS |
| 17 | admin: supplier payables 15,000 (Finance gDash rule: Received 6,000 + Pending 9,000, Cancelled excluded), outstanding expenses 5,500, purchases receiv... | payables 15,000 (Received 6,000 + Pending 9,000; Cancelled excluded), expenses 5,500, purchases 10,000, valuation 4,921 | Payables, purchases and stock value Open → Supplier payables 2 receipt(s) with a balance · pending included 15,000 Ks Outstanding expenses 2 partly paid or unpaid 5,500 Ks Purchases received 1 receipt... | PASS |
| 18 | inventory alert order Out > Expired > Critical > Low > Exp.Soon | Ibuprofen\|Out, Amox\|Expired, Vacc\|Critical, Drops\|Low, Oint\|Exp.Soon | ["Ibuprofen\|Out","Amox\|Expired","Vacc\|Critical","Drops\|Low","Oint\|Exp.Soon"] | PASS |
| 19 | doctor: no finance summaries or stock valuation in the interface | absent | finance text absent | PASS |
| 20 | doctor: no cost / selling price / inpatient-fee values in the page | absent | markers absent | PASS |
| 21 | doctor: static check - invSafe, exOf, dLeft, st, invAlerts, num never mention cost or price | no match | no match | PASS |
| 22 | doctor: appointments (3), inpatients (2) and inventory alerts (5) visible | 3/2/5 | 3/2/5 | PASS |
| 23 | doctor: all clinic appointments shown (no doctor filter) | both doctors listed | both doctors listed | PASS |
| 24 | staff: no finance summaries or stock valuation in the interface | absent | finance text absent | PASS |
| 25 | staff: no cost / selling price / inpatient-fee values in the page | absent | markers absent | PASS |
| 26 | staff: static check - invSafe, exOf, dLeft, st, invAlerts, num never mention cost or price | no match | no match | PASS |
| 27 | staff: appointments (3), inpatients (2) and inventory alerts (5) visible | 3/2/5 | 3/2/5 | PASS |
| 28 | staff: all clinic appointments shown (no doctor filter) | both doctors listed | both doctors listed | PASS |
| 29 | doctor: patient activity, Visits This Month = 2, species shown | 2 | visits=2 | PASS |
| 30 | staff: no patient activity, no Visits This Month, no species | absent | absent | PASS |
| 31 | species exactly as stored (Canine and Dog separate, blank = Not recorded) | ok | ok | PASS |

### 07 Section failure isolation

| # | Check | Expected | Actual | Result |
|---|---|---|---|---|
| 1 | renderer topCards throws: that section shows Unavailable, all other sections render | unavailable>=1, 5 others present | unavailable=1, present=5/6 | PASS |
| 2 | renderer apptCard throws: that section shows Unavailable, all other sections render | unavailable>=1, 5 others present | unavailable=1, present=5/6 | PASS |
| 3 | renderer ipCard throws: that section shows Unavailable, all other sections render | unavailable>=1, 5 others present | unavailable=1, present=5/6 | PASS |
| 4 | renderer secPatients throws: that section shows Unavailable, all other sections render | unavailable>=1, 5 others present | unavailable=1, present=5/6 | PASS |
| 5 | renderer secFinance throws: that section shows Unavailable, all other sections render | unavailable>=1, 5 others present | unavailable=1, present=5/6 | PASS |
| 6 | renderer secInv throws: that section shows Unavailable, all other sections render | unavailable>=1, 5 others present | unavailable=1, present=5/6 | PASS |
| 7 | calculation apptStats throws: page keeps rendering, failed part marked unavailable | unavailable>=1, sections>=4 | unavailable=2, sections present=5 | PASS |
| 8 | calculation ipStats throws: page keeps rendering, failed part marked unavailable | unavailable>=1, sections>=4 | unavailable=2, sections present=5 | PASS |
| 9 | calculation patStats throws: page keeps rendering, failed part marked unavailable | unavailable>=1, sections>=4 | unavailable=1, sections present=5 | PASS |
| 10 | calculation finStats throws: page keeps rendering, failed part marked unavailable | unavailable>=1, sections>=4 | unavailable=2, sections present=5 | PASS |
| 11 | calculation invAlerts throws: page keeps rendering, failed part marked unavailable | unavailable>=1, sections>=4 | unavailable=1, sections present=5 | PASS |
| 12 | whole render throws: Unavailable message (not blank) and recovery on next render | Unavailable ... + recovery | Unavailable — the Dashboard could not be displayed. Try Refr | PASS |
| 13 | doctor: inventory renderer failure leaves appointments and patient activity working | {inv:false,appt:true,pat:true} | {"inv":false,"appt":true,"pat":true} | PASS |
| 14 | every injected failure was caught and logged by the Dashboard; no uncaught page error | >0 caught, 0 uncaught | 13 caught, 0 uncaught | PASS |

### 08 Cross-tab role / permission refresh (two real tabs, same browser profile)

| # | Check | Expected | Actual | Result |
|---|---|---|---|---|
| 1 | start: Admin sees finance and patient activity | {admin,fin:true,pat:true} | {"role":"admin","fin":true,"pat":true} | PASS |
| 2 | Admin -> Doctor: open Dashboard tab reloads and shows the Doctor view (finance gone, patient activity kept) | {doctor,fin:false,pat:true} | {"role":"doctor","fin":false,"pat":true} | PASS |
| 3 | Admin -> Doctor: exactly one reload | +1 | loads +1 | PASS |
| 4 | Admin -> Doctor: at the moment of the change the stale finance content was already replaced (no unauthorized content left on screen) | "Permissions changed - reloading..." without finance text | Permissions changed — reloading… | PASS |
| 5 | Admin -> Doctor: after the change the tab never read pwTx / pwExpenses / pwGoodsReceipts | none read | none read | PASS |
| 6 | Doctor -> Staff: tab reloads; patient activity and Visits This Month removed; finance still absent | {staff,pat:false,visits:false,fin:false} | {"role":"staff","pat":false,"visits":false,"fin":false} | PASS |
| 7 | Doctor -> Staff: exactly one reload; no read of petworld.records.v1 or finance keys afterwards | +1; no records/finance reads | loads +1, reads after change: pwSession, pwTheme, pwRolePerms, pwAppts, pwIp, pwInventory, pwClinic | PASS |
| 8 | Staff -> Owner: tab leaves index.html (Owner may not use the Dashboard) | /patients.html... | /patients.html#appointments | PASS |
| 9 | Staff -> Owner: no Dashboard business data read after the change and no Dashboard content on the new page | none | business reads: none | PASS |
| 10 | Owner -> Admin: Dashboard opened after the change shows the full Admin view | {admin,fin:true,pat:true} | {"role":"admin","fin":true,"pat":true} | PASS |
| 11 | Owner -> Admin as a live refresh of an open Dashboard tab | - | SKIPPED: An Owner tab is redirected away from index.html by the shared guard, so the Dashboard cannot stay open as Owner; tested as "open again after the change" instead | SKIPPED |
| 12 | Admin -> unexpected role: tab reloads to "No access", sidebar emptied, no business data read afterwards | {noAccess:true,sidebarChildren:0,businessReads:[]} | {"noAccess":true,"sidebarChildren":0,"businessReads":[]} | PASS |
| 13 | Unexpected role -> Admin: tab reloads and shows the full Admin view again | {admin,fin:true} | {"role":"admin","fin":true} | PASS |
| 14 | pwRolePerms removes index.html for Doctor: open Dashboard tab leaves the Dashboard | /patients.html... | /patients.html#appointments | PASS |
| 15 | after navigating back, Doctor without Dashboard permission is redirected again (shared guard) | /patients.html... | /patients.html#appointments | PASS |
| 16 | pwRolePerms changes but Dashboard stays allowed: exactly one reload, Doctor view kept | +1, doctor, finance false | loads +1, role=doctor, finance=false | PASS |
| 17 | Business-data change in another tab (pwAppts): Dashboard refreshes in place, no reload | 3 -> 1, loads +0 | appointments 3 -> 1, loads +0 | PASS |
| 18 | pwTheme change in another tab: no reload | +0 | loads +0 | PASS |
| 19 | Identical rewrite of pwRolePerms (no change): no reload | +0 | loads +0 | PASS |
| 20 | Burst of 3 changes in a row (session, permissions, session): exactly one reload, final role Admin, no reload loop | +1, admin | loads +1, role=admin | PASS |
| 21 | localStorage.clear() in another tab: Dashboard reloads once and falls back to the shared default (admin) | +1, admin | loads +1, role=admin | PASS |
| 22 | Three role changes one after another (600 ms apart): each causes exactly one reload | doctor:+1, staff:+1, admin:+1 | doctor:+1, staff:+1, admin:+1 | PASS |
| 23 | No infinite reload: load count stays constant for 2.5 s after the last change | unchanged | loads 19 -> 19 | PASS |
| 24 | Tab errors during all cross-tab scenarios | none | none | PASS |
| 25 | Missed storage event, then tab becomes visible (visibilitychange): stale Admin view is replaced by the Staff view | stale admin -> staff, finance false | before=admin (stale, as expected without the event), after=staff, finance=false | PASS |
| 26 | Missed storage event, then page restored (pageshow with persisted=true, event simulated): stale Admin view is replaced by the Staff view | stale admin -> staff, finance false | before=admin (stale, as expected without the event), after=staff, finance=false | PASS |
| 27 | Missed storage event, then Refresh button: stale Admin view is replaced by the Staff view | stale admin -> staff, finance false | before=admin (stale, as expected without the event), after=staff, finance=false | PASS |
| 28 | Real back/forward-cache restore (pageshow persisted=true produced by the browser) | - | SKIPPED: headless Chromium with remote debugging does not use the back/forward cache; the handler was tested with a simulated event only | SKIPPED |
| 29 | Safety-net timer: with the storage handler removed, a visible tab notices the role change within the 60-second check | refreshed within 75 s, role=doctor | document.hidden=false, refreshed=true, role=doctor | PASS |

### 09 Read-only verification

| # | Check | Expected | Actual | Result |
|---|---|---|---|---|
| 1 | admin: whole localStorage is byte-identical before and after load, render, Refresh, storage event and visibilitychange | identical=true, write calls=0 | identical=true, write calls=0 | PASS |
| 2 | doctor: whole localStorage is byte-identical before and after load, render, Refresh, storage event and visibilitychange | identical=true, write calls=0 | identical=true, write calls=0 | PASS |
| 3 | staff: whole localStorage is byte-identical before and after load, render, Refresh, storage event and visibilitychange | identical=true, write calls=0 | identical=true, write calls=0 | PASS |

### 10 Empty and corrupt storage

| # | Check | Expected | Actual | Result |
|---|---|---|---|---|
| 1 | admin empty storage: renders zeros and empty states, no errors | 0, errors=0 | appointments=0, errors=0 | PASS |
| 2 | admin empty storage: finance shows 0 Ks | 0 Ks | 0 Ks | PASS |
| 3 | admin invalid JSON in every key: renders safely | 0 | errors=0 | PASS |
| 4 | admin wrong data shapes: renders safely | 0 | errors=0 | PASS |
| 5 | admin junk records inside arrays: renders, no NaN/undefined/null text, no errors | errors=0, badText=false | errors=0, badText=false | PASS |
| 6 | doctor empty storage: renders zeros and empty states, no errors | 0, errors=0 | appointments=0, errors=0 | PASS |
| 7 | doctor invalid JSON in every key: renders safely | 0 | errors=0 | PASS |
| 8 | doctor wrong data shapes: renders safely | 0 | errors=0 | PASS |
| 9 | doctor junk records inside arrays: renders, no NaN/undefined/null text, no errors | errors=0, badText=false | errors=0, badText=false | PASS |
| 10 | staff empty storage: renders zeros and empty states, no errors | 0, errors=0 | appointments=0, errors=0 | PASS |
| 11 | staff invalid JSON in every key: renders safely | 0 | errors=0 | PASS |
| 12 | staff wrong data shapes: renders safely | 0 | errors=0 | PASS |
| 13 | staff junk records inside arrays: renders, no NaN/undefined/null text, no errors | errors=0, badText=false | errors=0, badText=false | PASS |
| 14 | Visits This Month hidden when no visit has a valid date, and the data note says so | card absent, note shown | card absent, note shown | PASS |
| 15 | Impossible calendar dates (2026-02-31, 04-31, 2025-13-01...) are excluded and counted in the data note | 1, 1, 1,000 Ks; note: 2 appointments, 2 visits, 3 finance records | appointments=1, visits=1, net sales=1,000 Ks | PASS |
| 16 | isD(): 6930 generated dates agree with an independent calendar reference | 0 | 0 disagreements | PASS |

### 11 Responsive viewports

| # | Check | Expected | Actual | Result |
|---|---|---|---|---|
| 1 | responsive admin dark @320px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=320, viewport=320, columns=1 | PASS |
| 2 | responsive admin dark @320px: single column and 26px stat values | columns=1, font=26px | columns=1, font=26px | PASS |
| 3 | responsive admin dark @375px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=375, viewport=375, columns=1 | PASS |
| 4 | responsive admin dark @375px: single column and 26px stat values | columns=1, font=26px | columns=1, font=26px | PASS |
| 5 | responsive admin dark @414px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=414, viewport=414, columns=1 | PASS |
| 6 | responsive admin dark @414px: single column and 26px stat values | columns=1, font=26px | columns=1, font=26px | PASS |
| 7 | responsive admin dark @599px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=599, viewport=599, columns=1 | PASS |
| 8 | responsive admin dark @599px: single column and 26px stat values | columns=1, font=26px | columns=1, font=26px | PASS |
| 9 | responsive admin dark @600px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=600, viewport=600, columns=1 | PASS |
| 10 | responsive admin dark @600px: single column and 26px stat values | columns=1, font=26px | columns=1, font=26px | PASS |
| 11 | responsive admin dark @768px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=768, viewport=768, columns=2 | PASS |
| 12 | responsive admin dark @1100px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=1100, viewport=1100, columns=2 | PASS |
| 13 | responsive admin light @320px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=320, viewport=320, columns=1 | PASS |
| 14 | responsive admin light @320px: single column and 26px stat values | columns=1, font=26px | columns=1, font=26px | PASS |
| 15 | responsive admin light @375px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=375, viewport=375, columns=1 | PASS |
| 16 | responsive admin light @375px: single column and 26px stat values | columns=1, font=26px | columns=1, font=26px | PASS |
| 17 | responsive admin light @414px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=414, viewport=414, columns=1 | PASS |
| 18 | responsive admin light @414px: single column and 26px stat values | columns=1, font=26px | columns=1, font=26px | PASS |
| 19 | responsive admin light @599px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=599, viewport=599, columns=1 | PASS |
| 20 | responsive admin light @599px: single column and 26px stat values | columns=1, font=26px | columns=1, font=26px | PASS |
| 21 | responsive admin light @600px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=600, viewport=600, columns=1 | PASS |
| 22 | responsive admin light @600px: single column and 26px stat values | columns=1, font=26px | columns=1, font=26px | PASS |
| 23 | responsive admin light @768px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=768, viewport=768, columns=2 | PASS |
| 24 | responsive admin light @1100px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=1100, viewport=1100, columns=2 | PASS |
| 25 | responsive staff dark @320px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=320, viewport=320, columns=1 | PASS |
| 26 | responsive staff dark @320px: single column and 26px stat values | columns=1, font=26px | columns=1, font=26px | PASS |
| 27 | responsive staff dark @375px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=375, viewport=375, columns=1 | PASS |
| 28 | responsive staff dark @375px: single column and 26px stat values | columns=1, font=26px | columns=1, font=26px | PASS |
| 29 | responsive staff dark @414px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=414, viewport=414, columns=1 | PASS |
| 30 | responsive staff dark @414px: single column and 26px stat values | columns=1, font=26px | columns=1, font=26px | PASS |
| 31 | responsive staff dark @599px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=599, viewport=599, columns=1 | PASS |
| 32 | responsive staff dark @599px: single column and 26px stat values | columns=1, font=26px | columns=1, font=26px | PASS |
| 33 | responsive staff dark @600px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=600, viewport=600, columns=1 | PASS |
| 34 | responsive staff dark @600px: single column and 26px stat values | columns=1, font=26px | columns=1, font=26px | PASS |
| 35 | responsive staff dark @768px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=768, viewport=768, columns=2 | PASS |
| 36 | responsive staff dark @1100px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=1100, viewport=1100, columns=2 | PASS |
| 37 | responsive staff light @320px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=320, viewport=320, columns=1 | PASS |
| 38 | responsive staff light @320px: single column and 26px stat values | columns=1, font=26px | columns=1, font=26px | PASS |
| 39 | responsive staff light @375px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=375, viewport=375, columns=1 | PASS |
| 40 | responsive staff light @375px: single column and 26px stat values | columns=1, font=26px | columns=1, font=26px | PASS |
| 41 | responsive staff light @414px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=414, viewport=414, columns=1 | PASS |
| 42 | responsive staff light @414px: single column and 26px stat values | columns=1, font=26px | columns=1, font=26px | PASS |
| 43 | responsive staff light @599px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=599, viewport=599, columns=1 | PASS |
| 44 | responsive staff light @599px: single column and 26px stat values | columns=1, font=26px | columns=1, font=26px | PASS |
| 45 | responsive staff light @600px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=600, viewport=600, columns=1 | PASS |
| 46 | responsive staff light @600px: single column and 26px stat values | columns=1, font=26px | columns=1, font=26px | PASS |
| 47 | responsive staff light @768px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=768, viewport=768, columns=2 | PASS |
| 48 | responsive staff light @1100px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=1100, viewport=1100, columns=2 | PASS |
| 49 | responsive janitor dark @320px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=320, viewport=320, columns=0 | PASS |
| 50 | responsive janitor dark @375px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=375, viewport=375, columns=0 | PASS |
| 51 | responsive janitor dark @414px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=414, viewport=414, columns=0 | PASS |
| 52 | responsive janitor dark @599px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=599, viewport=599, columns=0 | PASS |
| 53 | responsive janitor dark @600px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=600, viewport=600, columns=0 | PASS |
| 54 | responsive janitor dark @768px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=768, viewport=768, columns=0 | PASS |
| 55 | responsive janitor dark @1100px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=1100, viewport=1100, columns=0 | PASS |
| 56 | responsive janitor light @320px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=320, viewport=320, columns=0 | PASS |
| 57 | responsive janitor light @375px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=375, viewport=375, columns=0 | PASS |
| 58 | responsive janitor light @414px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=414, viewport=414, columns=0 | PASS |
| 59 | responsive janitor light @599px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=599, viewport=599, columns=0 | PASS |
| 60 | responsive janitor light @600px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=600, viewport=600, columns=0 | PASS |
| 61 | responsive janitor light @768px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=768, viewport=768, columns=0 | PASS |
| 62 | responsive janitor light @1100px: no horizontal page overflow | scrollWidth <= viewport | scrollWidth=1100, viewport=1100, columns=0 | PASS |

### 12 Dark / light theme

| # | Check | Expected | Actual | Result |
|---|---|---|---|---|
| 1 | theme admin dark @320px: body class matches stored theme | light class=false | light class=false | PASS |
| 2 | theme admin dark @375px: body class matches stored theme | light class=false | light class=false | PASS |
| 3 | theme admin dark @414px: body class matches stored theme | light class=false | light class=false | PASS |
| 4 | theme admin dark @599px: body class matches stored theme | light class=false | light class=false | PASS |
| 5 | theme admin dark @600px: body class matches stored theme | light class=false | light class=false | PASS |
| 6 | theme admin dark @768px: body class matches stored theme | light class=false | light class=false | PASS |
| 7 | theme admin dark @1100px: body class matches stored theme | light class=false | light class=false | PASS |
| 8 | theme admin light @320px: body class matches stored theme | light class=true | light class=true | PASS |
| 9 | theme admin light @375px: body class matches stored theme | light class=true | light class=true | PASS |
| 10 | theme admin light @414px: body class matches stored theme | light class=true | light class=true | PASS |
| 11 | theme admin light @599px: body class matches stored theme | light class=true | light class=true | PASS |
| 12 | theme admin light @600px: body class matches stored theme | light class=true | light class=true | PASS |
| 13 | theme admin light @768px: body class matches stored theme | light class=true | light class=true | PASS |
| 14 | theme admin light @1100px: body class matches stored theme | light class=true | light class=true | PASS |
| 15 | theme staff dark @320px: body class matches stored theme | light class=false | light class=false | PASS |
| 16 | theme staff dark @375px: body class matches stored theme | light class=false | light class=false | PASS |
| 17 | theme staff dark @414px: body class matches stored theme | light class=false | light class=false | PASS |
| 18 | theme staff dark @599px: body class matches stored theme | light class=false | light class=false | PASS |
| 19 | theme staff dark @600px: body class matches stored theme | light class=false | light class=false | PASS |
| 20 | theme staff dark @768px: body class matches stored theme | light class=false | light class=false | PASS |
| 21 | theme staff dark @1100px: body class matches stored theme | light class=false | light class=false | PASS |
| 22 | theme staff light @320px: body class matches stored theme | light class=true | light class=true | PASS |
| 23 | theme staff light @375px: body class matches stored theme | light class=true | light class=true | PASS |
| 24 | theme staff light @414px: body class matches stored theme | light class=true | light class=true | PASS |
| 25 | theme staff light @599px: body class matches stored theme | light class=true | light class=true | PASS |
| 26 | theme staff light @600px: body class matches stored theme | light class=true | light class=true | PASS |
| 27 | theme staff light @768px: body class matches stored theme | light class=true | light class=true | PASS |
| 28 | theme staff light @1100px: body class matches stored theme | light class=true | light class=true | PASS |
| 29 | theme janitor dark @320px: body class matches stored theme | light class=false | light class=false | PASS |
| 30 | theme janitor dark @375px: body class matches stored theme | light class=false | light class=false | PASS |
| 31 | theme janitor dark @414px: body class matches stored theme | light class=false | light class=false | PASS |
| 32 | theme janitor dark @599px: body class matches stored theme | light class=false | light class=false | PASS |
| 33 | theme janitor dark @600px: body class matches stored theme | light class=false | light class=false | PASS |
| 34 | theme janitor dark @768px: body class matches stored theme | light class=false | light class=false | PASS |
| 35 | theme janitor dark @1100px: body class matches stored theme | light class=false | light class=false | PASS |
| 36 | theme janitor light @320px: body class matches stored theme | light class=true | light class=true | PASS |
| 37 | theme janitor light @375px: body class matches stored theme | light class=true | light class=true | PASS |
| 38 | theme janitor light @414px: body class matches stored theme | light class=true | light class=true | PASS |
| 39 | theme janitor light @599px: body class matches stored theme | light class=true | light class=true | PASS |
| 40 | theme janitor light @600px: body class matches stored theme | light class=true | light class=true | PASS |
| 41 | theme janitor light @768px: body class matches stored theme | light class=true | light class=true | PASS |
| 42 | theme janitor light @1100px: body class matches stored theme | light class=true | light class=true | PASS |
| 43 | contrast dark: .dash-clinic (16px) | >= 4.5 | ratio 14.98 | PASS |
| 44 | contrast dark: .dash-sec (12px) | >= 4.5 | ratio 7.07 | PASS |
| 45 | contrast dark: .dash-kv span (13px) | >= 4.5 | ratio 12.3 | PASS |
| 46 | contrast dark: .dash-kv b (13px) | >= 4.5 | ratio 12.3 | PASS |
| 47 | contrast dark: .tbl td b (13px) | >= 4.5 | ratio 12.3 | PASS |
| 48 | contrast dark: .tbl td (13px) | >= 4.5 | ratio 12.3 | PASS |
| 49 | contrast dark: .dash-note (11px) | >= 4.5 | ratio 7.07 | PASS |
| 50 | contrast dark: .dash-chips (12px) | >= 4.5 | ratio 5.81 | PASS |
| 51 | contrast dark: .dash-meta span (11px) | >= 4.5 | ratio 6.39 | PASS |
| 52 | contrast dark: .dash-val (10px) | >= 4.5 | ratio 5.81 | PASS |
| 53 | contrast dark: .cd-h h3 (14px) | >= 4.5 | ratio 12.3 | PASS |
| 54 | contrast dark: .dash .mut (11px) | >= 4.5 | ratio 5.81 | PASS |
| 55 | contrast light: .dash-clinic (16px) | >= 4.5 | ratio 16.15 | PASS |
| 56 | contrast light: .dash-sec (12px) | >= 4.5 | ratio 9.33 | PASS |
| 57 | contrast light: .dash-kv span (13px) | >= 4.5 | ratio 17.85 | PASS |
| 58 | contrast light: .dash-kv b (13px) | >= 4.5 | ratio 17.85 | PASS |
| 59 | contrast light: .tbl td b (13px) | >= 4.5 | ratio 17.85 | PASS |
| 60 | contrast light: .tbl td (13px) | >= 4.5 | ratio 17.85 | PASS |
| 61 | contrast light: .dash-note (11px) | >= 4.5 | ratio 9.33 | PASS |
| 62 | contrast light: .dash-chips (12px) | >= 4.5 | ratio 10.31 | PASS |
| 63 | contrast light: .dash-meta span (11px) | >= 4.5 | ratio 9.33 | PASS |
| 64 | contrast light: .dash-val (10px) | >= 4.5 | ratio 10.31 | PASS |
| 65 | contrast light: .cd-h h3 (14px) | >= 4.5 | ratio 17.85 | PASS |
| 66 | contrast light: .dash .mut (11px) | >= 4.5 | ratio 10.31 | PASS |

### 13 Delivery integrity

| # | Check | Expected | Actual | Result |
|---|---|---|---|---|
| 1 | index.html SHA-256 unchanged during the whole test run (tested file = delivered file) | identical | 409ef0898269205ad32952ff50705ea28d7f5edd66a493ce05cb30b5e49b27e7 -> 409ef0898269205ad32952ff50705ea28d7f5edd66a493ce05cb30b5e49b27e7 | PASS |
