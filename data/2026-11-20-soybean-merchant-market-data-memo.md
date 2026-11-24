# Stage 4 Market-Data Memo: Scenario 14 (Soybean Merchant)

**Document Path:** `data/2026-11-20-soybean-merchant-market-data-memo.md`  
**Workbook Populated:** `capabilities/fx-hedging/model.xlsx`  
**Author:** James Heile  
**Market Data Date:** November 20, 2026  
**Exposure:** €13,500,000 receivable settling in 182 days (ACT/360)

---

## 1. Live Market Inputs & Provenance Table

| Named Range | Live Value | Unit | Source & Instrument | Observation / Retrieval Timestamp | Rationale / Assumption Note |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `FC_AMT` | `13500000` | EUR | Scenario 14 Commercial Invoice | Fixed contract parameter | Unchanged from scenario. |
| `T_DAYS` | `182` | Days | Scenario 14 Settlement Term | Fixed contract parameter | Unchanged from scenario (6 months). |
| `BASIS` | `360` | Days | Money-market convention | Fixed convention | ACT/360 day-count basis per `spec.md`. |
| `S0_in` | `1.0990` | USD/EUR | European Central Bank (ECB) Euro Foreign Exchange Reference Rate (`ecb.europa.eu`) | 2026-11-20 16:00 CET (retrieved 2026-11-20 19:06 HST) | Official daily institutional EUR/USD reference fix. |
| `R_USD` | `4.51%` | Annual % | U.S. 1-Year Treasury Constant Maturity Yield (`DGS1` / `I:1YTCMR`, U.S. Treasury via YCharts/FRED) | 2026-11-20 market close (retrieved 2026-11-20 19:06 HST) | 1-year sovereign USD risk-free benchmark per Stage 4 guidance. |
| `R_FC` | `3.00%` | Annual % | Germany 1-Year Government Bond (12-Month Bubill) Yield (Trading Economics) | 2026-11-20 market close (retrieved 2026-11-20 19:06 HST) | 1-year AAA Eurozone sovereign yield matching `R_USD` tenor and credit quality. |
| `F0_in` | `1.1705` | USD/EUR | Scenario 14 Bank Forward Quote | Scenario baseline quote (retained) | Bank quote retained per Stage 4 rule; not overwritten with CIP-implied rate. |
| `K_PUT` | `1.0990` | USD/EUR | At-the-money (ATM) strike (`= S0_in`) | 2026-11-20 | Set at live spot `S0_in` per Scenario 14 convention. |
| `K_CALL` | `1.0990` | USD/EUR | At-the-money (ATM) strike (`= S0_in`) | 2026-11-20 | Set at live spot `S0_in` per Scenario 14 convention. |
| `PREM_PUT` | `0.0194` | USD/EUR | Scenario 14 Indicative Option Quote | Scenario baseline (retained) | **Stated Assumption:** Retail-accessible FX option quotes are unreliable; scenario premium kept. |
| `PREM_CALL` | `0.0296` | USD/EUR | Scenario 14 Indicative Option Quote (Bank Bid) | Scenario baseline (retained) | **Stated Assumption:** Scenario premium kept (bank bid-side quote for written call). |

### Rate Choice Rationale
* **`R_USD` (4.51%):** Chosen from the 1-Year U.S. Treasury Constant Maturity yield to provide a transparent, liquid, sovereign USD benchmark for money-market parity.
* **`R_FC` (3.00%):** Chosen from the 1-Year German Bubill yield because it is the closest Eurozone sovereign equivalent in credit quality and maturity to the 1-Year U.S. Treasury, avoiding cross-tier credit distortion in the CIP test.

---

## 2. Covered Interest Parity (CIP) Check & Bank Quote Gap

Using ACT/360 ($T / 360 = 182 / 360 = 0.505556$):

$$F_0^{\text{CIP}} = S_0 \times \frac{1 + R_{\text{USD}} \times \frac{182}{360}}{1 + R_{\text{FC}} \times \frac{182}{360}} = 1.0990 \times \frac{1 + 0.0451 \times \frac{182}{360}}{1 + 0.0300 \times \frac{182}{360}} = 1.0990 \times \frac{1.022801}{1.015167} = \mathbf{1.1073\text{ USD/EUR}}$$

| Metric | Value | Notes |
| :--- | :--- | :--- |
| Bank Forward Quote (`F0_in`) | `1.1705` | Retained from scenario |
| CIP-Implied Forward (`F_implied`) | `1.1073` (`1.107264`) | Computed from live Nov 20 spot & 1Y sovereign yields |
| **Rate Gap (`F0_in - F_implied`)** | **`+0.0632` USD/EUR** | Bank quote is +632.4 pips (+5.71%) above CIP |
| Forward Proceeds (`OUT_FWD_PROCEEDS`) | `$15,801,750.00` | `13,500,000 × 1.1705` |
| Money-Market Proceeds (`OUT_MM_PROCEEDS`) | `$14,948,068.08` | `13,500,000 × 1.1072643` |
| **USD Parity Gap (`OUT_PARITY_GAP`)** | **`$853,681.92`** | Breaches ±$5,000 tolerance → **`WARN: BASIS_MISMATCH`** |

**Interpretation for Stage 5:** No input was altered to force parity to pass. The parity gap widened from `$487,516.45` (placeholder stage) to `$853,681.92` because live EUR/USD spot fell from `1.1221` to `1.0990` while the scenario's bank forward quote (`1.1705`) remained static. In live treasury operations, a +632 pip off-market quote is stale and requires a bank requote before execution.

---

## 3. Populated Model Outputs & Verification

Expected named-range outputs in `capabilities/fx-hedging/model.xlsx` under live November 20, 2026 inputs:

- **`OUT_UNHEDGED_BASE` (at `S0_in = 1.0990`):** `$14,836,500.00`
- **`OUT_FWD_PROCEEDS`:** `$15,801,750.00`
- **`OUT_MM_PROCEEDS`:** `$14,948,068.08` *(Step 1 Borrow: `€13,298,308.98` → Step 2 Spot USD: `$14,614,841.57` → Step 3 Invested USD: `$14,948,068.08`)*
- **`OUT_PUT_NET_FLOOR`:** `$14,568,628.53` *(Upfront premium `$261,900.00`, compounded `FV_PREM_PUT = -$267,871.47`)*
- **Written Call Capped Proceeds (at `S_T >= 1.0990`):** `$15,245,211.10` *(Upfront premium `$399,600.00`, compounded `FV_PREM_CALL = +$408,711.10`; breakeven spot where upside give-up overtakes premium = `1.1293`)*
- **Sensitivity Grid & Chart:** Recalculated across `-5%` (`1.0441`) to `+5%` (`1.1540`) around live `S0_in = 1.0990`.

### Structural Fixes During Population
No structural fixes were needed during Stage 4 population. Only the market inputs on `Inputs` and the cover/source text changed from Stage 3; all formulas, named ranges, and sensitivity rows recalculated cleanly with zero formula errors.

---

## 4. FX Hedging Lab Cross-Check

| Strategy | `model.xlsx` Output | FX Hedging Lab Output | Difference | Status |
| :--- | :--- | :--- | :--- | :--- |
| Unhedged (at `S0_in`) | `$14,836,500.00` | Not displayed in Lab view | N/A | Not shown in Lab |
| Forward Hedge | `$15,801,750.00` | `$15,801,750` | `$0.00` | Exact match |
| Money-Market Hedge | `$14,948,068.08` | `$14,948,068` | `+$0.08` *(display rounding)* | Match (rounding only) |
| Put Option Floor | `$14,568,628.53` | Not displayed in Lab view | N/A | Not shown in Lab |
| Written Call (Capped) | `$15,245,211.10` | Not displayed in Lab view | N/A | Not shown in Lab |

**Money-Market Steps & Premium / Difference Notes:**
- **Lab Money-Market Steps:** Step 1 Borrow FC: `13,298,309 EUR` (vs. `€13,298,308.98` in `model.xlsx`) → Step 2 Convert at spot: `$14,614,842` (vs. `$14,614,841.57`) → Step 3 Invest in USD: `$14,948,068` (vs. `$14,948,068.08`)[cite: 7].
- **Difference Explanation:** Forward and money-market outputs match `model.xlsx` aside from whole-dollar/integer rounding in the Lab display (`$14,948,068` vs. `$14,948,068.08`)[cite: 7]. Both tools compute an implied forward of `1.1073` against the `1.1705` quoted forward (`-$853,682` in Lab `MM - Forward` terms, matching the `-$853,681.92` difference between `$14,948,068.08` and `$15,801,750.00`)[cite: 7, 8]. Unhedged, put floor, and written call outputs were not displayed in the Lab's results panel, so no option-premium comparison was observed from the Lab[cite: 7, 8].

**Cross-Check Resolution:** Forward and money-market hedge mechanics and CIP gap figures are verified across `model.xlsx` and the FX Hedging Lab[cite: 7, 8]. The `$853,681.92` parity gap reflects the retained `1.1705` bank forward quote versus live November 20 market rates rather than a workbook formula error, and option figures remain governed by `model.xlsx` since the Lab UI did not show option outputs[cite: 7, 8].
