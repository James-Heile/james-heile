# Stage 5 Independent Validation & Spec Retrospective: Scenario 14 (Soybean Merchant)

**Document Path:** `analysis/2026-11-21-soybean-merchant-validation-analysis.md`  
**Workbook Audited:** `capabilities/fx-hedging/model.xlsx`  
**Source Documents Provided to Fresh LLM:** `capabilities/fx-hedging/spec.md` and `data/2026-11-20-soybean-merchant-market-data-memo.md`  
**Author:** James Heile  
**Date:** November 21, 2026  

---

## 1. Comparison Table: `model.xlsx` vs. Fresh LLM

All figures below evaluate the **€13,500,000** receivable at **$t = 182$** on an **ACT/360** basis using live November 20, 2026 inputs (`S0_in = 1.0990`, `F0_in = 1.1705`, `R_USD = 4.51%`, `R_FC = 3.00%`, `K_PUT = K_CALL = 1.0990`, `PREM_PUT = 0.0194`, `PREM_CALL = 0.0296`). Terminal spot rates correspond to `-5%` (`1.04405` exact / `1.0441` rounded), `0%` (`1.09900`), and `+5%` (`1.15395` exact / `1.1540` rounded).

| Strategy & Spot Scenario (`Δ%`) | `model.xlsx` Result | Fresh LLM Result | Variance (`LLM - Model`) | Diagnosis |
| :--- | :--- | :--- | :--- | :--- |
| **Unhedged** (`-5%` / `1.04405`) | `$14,094,675.00` | `$14,094,675.00` | `$0.00` | Exact match (LLM used unrounded `1.04405` in calculation). |
| **Unhedged** (`0%` / `1.09900`) | `$14,836,500.00` | `$14,836,500.00` | `$0.00` | Exact match (`OUT_UNHEDGED_BASE`). |
| **Unhedged** (`+5%` / `1.15395`) | `$15,578,325.00` | `$15,578,325.00` | `$0.00` | Exact match (LLM used unrounded `1.15395` in calculation). |
| **Forward Hedge** (`-5%`, `0%`, `+5%`) | `$15,801,750.00` | `$15,801,750.00` | `$0.00` | Exact match (`OUT_FWD_PROCEEDS` across all scenarios). |
| **Money-Market Hedge** (`-5%`, `0%`, `+5%`) | `$14,948,068.08` | `$14,948,068.08` | `$0.00` | Exact match (`OUT_MM_PROCEEDS`; `F_implied = 1.1073`). |
| **Put Option Floor** (`-5%` / `1.04405`) | `$14,568,628.53` | `$14,568,628.53` | `$0.00` | Exact match (`OUT_PUT_NET_FLOOR`; `FV_PREM_PUT = -$267,871.47`). |
| **Put Option Floor** (`0%` / `1.09900`) | `$14,568,628.53` | `$14,568,628.53` | `$0.00` | Exact match (ATM floor net of compounded put premium). |
| **Put Option Floor** (`+5%` / `1.15395`) | `$15,310,453.53` | `$15,310,453.53` | `$0.00` | Exact match (`$15,578,325.00 - $267,871.47`). |
| **Written Call Overlay** (`-5%` / `1.04405`) | `$14,503,386.10` | `$14,503,386.10` | `$0.00` | Exact match (`$14,094,675.00 + $408,711.10`). |
| **Written Call Overlay** (`0%` / `1.09900`) | `$15,245,211.10` | `$15,245,211.10` | `$0.00` | Exact match (`$14,836,500.00 + $408,711.10`). |
| **Written Call Overlay** (`+5%` / `1.15395`) | `$15,245,211.10` | `$15,245,211.10` | `$0.00` | Exact match (capped at `K_CALL = 1.0990` plus `+$408,711.10`). |

### Discrepancy & Nuance Notes
* **Numerical outputs:** There were **zero dollar variances (`$0.00`)** between `model.xlsx` and the fresh LLM across all 11 strategy/scenario cells, the CIP-implied forward (`1.1073`), and the parity gap (`$853,681.92`).
* **Spec ambiguity / presentation nuance (`1.04405` vs. `1.0441` and `1.15395` vs. `1.1540`):** The fresh LLM noted `1.0441` and `1.1540` as rounded spot values while evaluating `S_T` using exact unrounded `1.04405` and `1.15395`. Because `spec.md` defines `S_T = S0_in * (1 + Delta%)` without specifying whether `S_T` is rounded to 4 decimals before multiplying by `FC_AMT`, a different reader or formula could have rounded `S_T` to 4 decimals first (which would shift unhedged/put/call figures slightly). That is a **spec ambiguity** to tighten in v2.

---

## 2. Hand-Verification Table (Named-Range Arithmetic)

Below is the independent hand calculation using named-range notation and an **ACT/360** day-count fraction:

$$\text{Day-Count Fraction} = \frac{\text{T\_DAYS}}{\text{BASIS}} = \frac{182}{360} \approx 0.50555556$$

| Outcome / Step | Named-Range Formula | Step-by-Step Arithmetic | Hand-Verified Result | Reconciled to `model.xlsx` |
| :--- | :--- | :--- | :--- | :--- |
| **Forward Proceeds** (`OUT_FWD_PROCEEDS`) | `FC_AMT * F0_in` | `13,500,000 * 1.1705` | **`$15,801,750.00`** | Matches `$15,801,750.00` (`$0.00` diff) |
| **Money-Market Step 1** (`FC_Borrow`) | `FC_AMT / (1 + R_FC * (T_DAYS / BASIS))` | `13,500,000 / (1 + 0.0300 * (182 / 360))` = `13,500,000 / 1.01516667` | **`€13,298,308.98`** | Matches `13,298,308.98` EUR (`$0.00` diff) |
| **Money-Market Step 2** (`USD_Now`) | `FC_Borrow * S0_in` | `13,298,308.9803 * 1.0990` | **`$14,614,841.57`** | Matches `$14,614,841.57` (`$0.00` diff) |
| **Money-Market Step 3** (`OUT_MM_PROCEEDS`) | `USD_Now * (1 + R_USD * (T_DAYS / BASIS))` | `14,614,841.5694 * (1 + 0.0451 * (182 / 360))` = `14,614,841.5694 * 1.02280056` | **`$14,948,068.08`** | Matches `$14,948,068.08` (`$0.00` diff) |
| **Put Option Floor** (`OUT_PUT_NET_FLOOR` at `S_T <= K_PUT`) | `MAX(S_T, K_PUT) * FC_AMT - PREM_PUT * FC_AMT * (1 + R_USD * (T_DAYS / BASIS))` | Floor gross: `1.0990 * 13,500,000 = $14,836,500.00`.<br>Compounded put cost: `0.0194 * 13,500,000 * 1.02280056 = $261,900.00 * 1.02280056 = $267,871.47`.<br>Net floor: `$14,836,500.00 - $267,871.47` | **`$14,568,628.53`** | Matches `$14,568,628.53` (`$0.00` diff) |
| **Written Call Cap** (at `S_T >= K_CALL`) | `MIN(S_T, K_CALL) * FC_AMT + PREM_CALL * FC_AMT * (1 + R_USD * (T_DAYS / BASIS))` | Cap gross: `1.0990 * 13,500,000 = $14,836,500.00`.<br>Compounded call inflow: `0.0296 * 13,500,000 * 1.02280056 = $399,600.00 * 1.02280056 = $408,711.10`.<br>Capped proceeds: `$14,836,500.00 + $408,711.10` | **`$15,245,211.10`** | Matches `$15,245,211.10` (`$0.00` diff) |

---

## 3. Spec Retrospective (What Worked, What Was Ambiguous, and `spec.md` v2)

Running an uncoached fresh LLM against only `spec.md` and the Stage 4 market-data memo showed that the core calculation flow, named-range formulas, and written-call cash-flow signs are clear enough to reproduce every strategy figure accurately. At the same time, placing `spec.md` side by side with the Stage 4 memo exposed several areas where the specification had stale placeholder values or left room for interpretation:

1. **Stale Stage 2/3 placeholder numbers still embedded in `spec.md`:** Sections 2, 5.5, and 7 of `spec.md` still reference the old October 21 placeholder spot (`1.1221`), placeholder interest rates (`5.30%` and `3.10%`), and the old parity gap (`$487,516.45` with implied forward `1.1344`), while the Stage 4 memo uses `1.0990`, `4.51%`, `3.00%`, and `$853,681.92`. An independent reader has to reconcile two sets of numbers across the two documents rather than reading a single authoritative formula specification.
2. **Contradictory notes on option premiums in Section 2:** Section 2's `Stage 4 Data Source` column still labels `PREM_PUT` and `PREM_CALL` as `Indicative option quote *(Replaced at Stage 4)*`, even though Section 5.5 and the Stage 4 memo establish that scenario premiums are **retained** as a stated assumption because retail FX option quotes are unreliable.
3. **Tab-count inconsistency between Sections 2.4 and 3:** The Stage 2 revision note in `spec.md` mentions a 6-tab architecture in one spot while Section 3 lists an 8-tab workbook (`Cover`, `Legend_Key`, `Inputs`, `Hedge_Unhedged_Fwd`, `Hedge_MoneyMarket`, `Hedge_Options`, `Sensitivity`, `Notes_Assumptions`).
4. **Unspecified rounding precision on intermediate spot rates (`S_T`) and rates:** The LLM noted `1.0441` and `1.1540` as rounded spot rates while computing with unrounded `1.04405` and `1.15395`. In `spec.md` v2, I would state explicitly that all intermediate calculations retain full precision and round only for display (`$#,##0.00` for USD, `0.0000` for FX rates, `0.00%` for yields), and I would separate abstract formula rules from stage-specific market snapshots so `spec.md` does not carry outdated placeholder figures once Stage 4 data is active.

---

## 4. Appendix: Raw Fresh LLM Prompt & Output

### Prompt Used (Fresh Session, No Coaching)
> Using only the attached model specification and market-data memo, compute the USD settlement proceeds at t = 182 for all hedging strategies (Unhedged, Forward, Money-Market, Put Option, and Written Call Option) across terminal spot rates S_T of -5%, 0%, and +5% relative to S_0, check Covered Interest Parity, and recommend a hedging strategy for the CFO. Show your step-by-step calculations.

### Raw Model Output
> Live Market Inputs & Terminal Spot Scenarios (t = 182)
> From the Stage 4 Market-Data Memo (2026-11-20-soybean-merchant-market-data-memo.md) and Model Specification (spec.md), the active parameters under an ACT/360 day-count convention are:
> - Receivable (FC_AMT): €13,500,000
> - Tenor (T_DAYS / BASIS): 182 / 360 = 0.505556 years
> - Inception Spot (S0_in): 1.0990 USD/EUR
> - USD Interest Rate (R_USD): 4.51% (1 + R_USD × 182/360 = 1.02280056)
> - EUR Interest Rate (R_FC): 3.00% (1 + R_FC × 182/360 = 1.01516667)
> - Bank Forward Quote (F0_in): 1.1705 USD/EUR
> - ATM Option Strikes (K_PUT, K_CALL): 1.0990 USD/EUR
> - Option Premiums: PREM_PUT = 0.0194 USD/EUR (paid upfront); PREM_CALL = 0.0296 USD/EUR (received upfront at bank bid)
>
> Terminal Spot Rates (S_T = S_0 × (1 + Δ%)):
> - −5% Scenario: S_T = 1.0990 × 0.95 = 1.04405 USD/EUR (1.0441 rounded)
> - 0% Scenario: S_T = 1.0990 × 1.00 = 1.09900 USD/EUR
> - +5% Scenario: S_T = 1.0990 × 1.05 = 1.15395 USD/EUR (1.1540 rounded)
>
> 2. Step-by-Step Settlement Calculations (t = 182)
> A. Unhedged Exposure (USD_Unhedged = FC_AMT × S_T)
> - At −5% (1.04405): €13,500,000 × 1.04405 = $14,094,675.00
> - At 0% (1.09900): €13,500,000 × 1.09900 = $14,836,500.00 (OUT_UNHEDGED_BASE)
> - At +5% (1.15395): €13,500,000 × 1.15395 = $15,578,325.00
>
> B. Forward Contract Hedge (USD_Forward = FC_AMT × F0_in)
> - All Scenarios (−5%, 0%, +5%): €13,500,000 × 1.1705 = $15,801,750.00 (OUT_FWD_PROCEEDS)
>
> C. Money-Market Hedge (Three-Step Pipeline)
> 1. Borrow EUR at t = 0 (FC_Borrow): FC_Borrow = €13,500,000 / (1 + 0.0300 × 182/360) = €13,500,000 / 1.01516667 = €13,298,308.98
> 2. Convert Borrowed EUR to USD at Spot (USD_Now): USD_Now = €13,298,308.98 × 1.0990 = $14,614,841.57
> 3. Invest USD to t = 182 (USD_MM): USD_MM = $14,614,841.57 × (1 + 0.0451 × 182/360) = $14,614,841.57 × 1.02280056 = $14,948,068.08
> - All Scenarios (−5%, 0%, +5%): Locked in at $14,948,068.08 (OUT_MM_PROCEEDS).
>
> D. Put Option Floor Hedge (USD_PUT = MAX(S_T, K_PUT) × FC_AMT + FV_PREM_PUT)
> 1. Compounded Upfront Put Premium (FV_PREM_PUT):
> - Upfront Cost = −0.0194 × €13,500,000 = −$261,900.00
> - FV_PREM_PUT = −$261,900.00 × 1.02280056 = −$267,871.47
> 2. Net Proceeds by Terminal Spot:
> - At −5% (1.04405 < 1.0990 — Option Exercised): max(1.04405, 1.0990) × €13,500,000 − $267,871.47 = $14,836,500.00 − $267,871.47 = $14,568,628.53
> - At 0% (1.09900 = 1.0990 — At-the-Money Floor): max(1.09900, 1.0990) × €13,500,000 − $267,871.47 = $14,836,500.00 − $267,871.47 = $14,568,628.53
> - At +5% (1.15395 > 1.0990 — Option Expires Worthless): max(1.15395, 1.0990) × €13,500,000 − $267,871.47 = $15,578,325.00 − $267,871.47 = $15,310,453.53
>
> E. Written Call Option Overlay (USD_CALL = MIN(S_T, K_CALL) × FC_AMT + FV_PREM_CALL)
> 1. Compounded Upfront Call Premium Received (FV_PREM_CALL):
> - Upfront Revenue = +0.0296 × €13,500,000 = +$399,600.00
> - FV_PREM_CALL = +$399,600.00 × 1.02280056 = +$408,711.10
> 2. Net Proceeds by Terminal Spot:
> - At −5% (1.04405 < 1.0990 — Buyer Lets Call Expire): min(1.04405, 1.0990) × €13,500,000 + $408,711.10 = $14,094,675.00 + $408,711.10 = $14,503,386.10
> - At 0% (1.09900 = 1.0990 — At-the-Money Cap): min(1.09900, 1.0990) × €13,500,000 + $408,711.10 = $14,836,500.00 + $408,711.10 = $15,245,211.10
> - At +5% (1.15395 > 1.0990 — Call Exercised Against Firm): min(1.15395, 1.0990) × €13,500,000 + $408,711.10 = $14,836,500.00 + $408,711.10 = $15,245,211.10
>
> 3. Comparative Summary Table (t = 182 USD Proceeds)
> - Unhedged: −5% Spot (1.04405) = $14,094,675.00 | 0% Spot (1.09900) = $14,836,500.00 | +5% Spot (1.15395) = $15,578,325.00 | Linear exposure; unprotected downside
> - Forward Hedge: −5% Spot = $15,801,750.00 | 0% Spot = $15,801,750.00 | +5% Spot = $15,801,750.00 | Fixed rate (1.1705); highest across all cases
> - Money-Market Hedge: −5% Spot = $14,948,068.08 | 0% Spot = $14,948,068.08 | +5% Spot = $14,948,068.08 | Synthetic fixed rate (1.1073)
> - Put Option Floor: −5% Spot = $14,568,628.53 | 0% Spot = $14,568,628.53 | +5% Spot = $15,310,453.53 | Hard floor at $14.57M; retains upside
> - Written Call Overlay: −5% Spot = $14,503,386.10 | 0% Spot = $15,245,211.10 | +5% Spot = $15,245,211.10 | +$408.7k buffer; capped at $15.25M (breakeven 1.1293)
>
> 4. Covered Interest Parity (CIP) Check
> - F_0^CIP = S_0 × (1 + R_USD × 182/360) / (1 + R_FC × 182/360) = 1.0990 × 1.02280056 / 1.01516667 = 1.107264 USD/EUR (1.1073)
> - Rate Discrepancy: F_{0,in} − F_0^CIP = 1.1705 − 1.107264 = +0.063236 USD/EUR (+632.4 pips, or +5.71% above theoretical parity).
> - USD Parity Gap (OUT_PARITY_GAP): $15,801,750.00 − $14,948,068.08 = $853,681.92
> - Audit Status: Because $853,681.92 exceeds the model's ±$5,000.00 tolerance threshold, the workbook triggers WARN: BASIS_MISMATCH. As noted in the memo, the bank forward quote (1.1705) remained static while live spot dropped from 1.1221 to 1.0990, making the quoted forward stale/off-market.
>
> 5. CFO Hedging Recommendation
> 1. Primary Action — Attempt Immediate Execution of the Forward Contract ($15,801,750.00), Subject to Requote Verification:
> - If the bank will honor the 1.1705 forward quote, the Forward Hedge strictly dominates every other strategy across all scenarios—delivering $15,801,750.00, which beats the Money-Market Hedge by $853,681.92 and even outperforms an unhedged +5% EUR rally ($15,578,325.00) with zero downside risk.
> 2. Fallback Action Upon Bank Requote — Execute the Money-Market Hedge ($14,948,068.08) or Put Option Floor ($14,568,628.53 minimum):
> - Because a +632.4 pip deviation above CIP is stale in live institutional FX markets, the bank will almost certainly reprice the forward near F_0^CIP ≈ 1.1073 (≈$14,948,068) upon a live dealing request.
> - If the CFO prioritizes budget certainty and margin protection (standard for physical commodity merchandisers operating on tight processing spreads), lock in the Money-Market Hedge at $14,948,068.08 (or the refreshed forward if quoted at or above 1.1073). This eliminates all FX volatility and beats the 0% spot unhedged baseline ($14,836,500.00) by $111,568.08 due to the positive USD–EUR interest rate carry (4.51% vs. 3.00%).
> - If management has a strong bullish EUR conviction and wants upside participation beyond S_T > 1.1188, use the Put Option Floor, which guarantees a worst-case floor of $14,568,628.53 while preserving upside ($15,310,453.53 at +5%). Avoid the Written Call Overlay as a standalone hedge, as it leaves the firm exposed to unhedged downside below 1.0990 ($14,503,386.10 at −5%) in exchange for a capped upside.
