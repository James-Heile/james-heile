# Build Audit Analysis: Scenario 14 (U.S. Soybean Merchant)

**Document Path:** `analysis/2026-10-28-soybean-merchant-build-audit-analysis.md`  
**Author:** James Heile  
**Date:** October 28, 2026  
**Status:** Stage 3 Build Audit Completed (≥3 Substantive Findings)  

---

## Executive Summary
This audit report documents the systematic verification, defect remediation, and testing of `capabilities/fx-hedging/model.xlsx` compiled from the Stage 2 specification (`capabilities/fx-hedging/spec.md`). During inspection of the initial AI-generated build, three structural formula defects were identified and corrected before final verification.

---

## Audit Findings & Verification Log

### Finding 1: Off-by-One Column Reference Defect (`#VALUE!` Errors in Money-Market & Options Tabs)
- **What was checked:** Opened `capabilities/fx-hedging/model.xlsx` and inspected cell evaluation across `Hedge_MoneyMarket`, `Hedge_Options`, and `Sensitivity`.
- **What was found:** In `Hedge_MoneyMarket` and `Hedge_Options`, the formulas were placed in Column B (`B4:B7`), while the downstream formulas referenced Column C (`C4`, `C5`, `C6`), which contained text descriptions (`"FC_Borrow (EUR)"`, etc.). This caused cascading `#VALUE!` errors in Steps 2–3, the implied forward rate, option net proceeds, and the corresponding columns of the `Sensitivity` table.
- **Action taken:** Updated all cell references on `Hedge_MoneyMarket` (`=B4*S0_in`, `=B5*(1+(R_USD*(T_DAYS/BASIS)))`, `=B6/FC_AMT`), `Hedge_Options` (`B4`, `B6`), and `Sensitivity` (`Hedge_MoneyMarket!$B$6`, `Hedge_Options!$B$4`, `Hedge_Options!$B$6`) to point to Column B. Confirmed all `#VALUE!` errors resolved cleanly.

### Finding 2: Hardcoded Spot Rate Constant in Baseline Unhedged Proceeds
- **What was checked:** Audited all calculation formulas across the workbook for compliance with Contract Requirement 2 (no hardcoded constants in calculated cells).
- **What was found:** On `Hedge_Unhedged_Fwd!B7` (`Baseline Unhedged Proceeds`), the formula was written as `=FC_AMT*1.1221` instead of referencing the named range `S0_in`.
- **Action taken:** Replaced the hardcoded `1.1221` literal with `=FC_AMT*S0_in` so that changing `S0_in` on the `Inputs` tab dynamically updates `OUT_UNHEDGED_BASE`.

### Finding 3: Missing Live Validation Formulas & Summary Output Named Ranges (§7 & §8)
- **What was checked:** Verified workbook compliance against Contract Requirement 7 (live computed validation checks in the workbook) and Section 8 (`OUT_*` named summary outputs).
- **What was found:** The initial `Notes_Assumptions` tab listed the validation rules only as static text strings rather than live formulas, and the five `OUT_*` summary names (`OUT_UNHEDGED_BASE`, `OUT_FWD_PROCEEDS`, `OUT_MM_PROCEEDS`, `OUT_PUT_NET_FLOOR`, `OUT_PARITY_GAP`) were not registered in Name Manager.
- **Action taken:** Added a live Automated Workbook Validation Checks table to `Notes_Assumptions` computing `F_implied`, `OUT_PARITY_GAP` (`$487,516.45`), the `WARN: BASIS_MISMATCH` trigger rule, the option floor check, and the forward flat-line check, and registered all five `OUT_*` named ranges in the workbook.

---
