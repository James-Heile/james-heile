# FX Hedging Model Specification: Scenario 14 (Soybean Merchant)

**Document Path:** `capabilities/fx-hedging/spec.md`  
**Engagement:** FX Hedging — Stage 2 Model Specification  
**Author:** James Heile  
**Date:** October 23, 2026  
**Status:** Draft for Review & Audit Specification  

---

## Section 1: Problem Statement

A U.S. soybean merchandiser has sold soybeans to a German oilseed crusher and invoiced the customer in euros. The firm holds a foreign-currency receivable of **€13,500,000**, with payment due in **182 days (6 months)**. 

Because the invoice is denominated in EUR and the firm reports in USD, the firm is exposed to EUR/USD foreign exchange volatility over the 182-day holding period. If the euro depreciates against the U.S. dollar, the firm will realize fewer USD proceeds upon settlement, creating potential cash flow deficits, operating budget shortfalls, and compressed profit margins. 

The objective of this financial model is to quantify the baseline unhedged exposure and evaluate alternative hedging structures—a **Forward Contract**, a **Money-Market Hedge**, a **Put Option Floor**, and a **Call Option/Collar Overlay**—to provide executive leadership with a structured risk-return framework for decision-making.

---

## Section 2: Inputs — The Named-Range Contract

All inputs are stored on the `Inputs` tab and bound to explicit named ranges. All interest rates and spot/forward quotes follow market conventions using an **ACT/360 day-count basis**.

| Named Range | Description | Placeholder Value | Unit | Stage 4 Data Source |
| :--- | :--- | :--- | :--- | :--- |
| `FC_AMT` | Foreign-currency receivable | `13500000` | EUR | Commercial Invoice (Fixed contract parameter) |
| `S0_in` | Spot exchange rate at inception | `1.1221` | USD per EUR | ECB Reference Rate (October 21, 2026) |
| `F0_in` | 182-day forward exchange rate | `1.1705` | USD per EUR | Indicative market quote *(Replaced at Stage 4)* |
| `R_USD` | USD annualized interest rate | `5.30%` | Annual % (ACT/360) | Federal Reserve H.15 6-Month Treasury / SOFR *(Placeholder)* |
| `R_FC` | EUR annualized interest rate | `3.10%` | Annual % (ACT/360) | ECB Main Refinancing Rate / 6M EURIBOR *(Placeholder)* |
| `K_PUT` | Put option strike price | `1.1221` | USD per EUR | Market quote (Set at-the-money spot `S0_in`) |
| `PREM_PUT` | Put option premium per unit of FC | `0.0194` | USD per EUR | Indicative option quote *(Replaced at Stage 4)* |
| `K_CALL` | Call option strike price | `1.1221` | USD per EUR | Market quote (Set at-the-money spot `S0_in`) |
| `PREM_CALL` | Call option premium per unit of FC | `0.0296` | USD per EUR | Indicative option quote *(Replaced at Stage 4)* |
| `T_DAYS` | Time to settlement | `182` | Days | Contract maturity (Fixed contract parameter) |
| `BASIS` | Day-count denominator | `360` | Days | ACT/360 money-market convention |

---

## Section 3: Tab Architecture

The workbook `capabilities/fx-hedging/model.xlsx` consists of six required tabs organized logically to support dynamic calculations and auditing:

1. **`Cover`**: Executive overview, project metadata, author info, scenario definition, and stage status.
2. **`Legend_Key`**: Formatting standards (e.g., blue text / yellow fill for hardcoded inputs, black text for formulas, gray fill for key summary outputs), color codes, and table of named range definitions.
3. **`Inputs`**: Centralized data entry table for all contract named ranges. No hardcoded constants exist outside this tab.
4. **`Hedge_Calculations`**: Active calculation engine detailing the step-by-step logic for Unhedged, Forward Contract, Money-Market Hedge, Put Option, and Call Option strategies.
5. **`Sensitivity`**: Scenario analysis engine modeling terminal spot rate (`S_T`) shifts from -5% to +5% in 1% increments, containing the comparative summary table and visualization chart.
6. **`Notes_Assumptions`**: Institutional documentation of day-count conventions, transaction cost policies, credit assumptions, and mathematical models.

---

## Section 4: Assumptions & Constraints

1. **Day-Count Convention**: All annualized interest rates (`R_USD`, `R_FC`) use the money-market convention of **ACT/360**. The time fraction is calculated strictly as `T_DAYS` / `BASIS`.
2. **Transaction Costs & Spreads**: Transaction costs, bid-ask spreads, and credit line maintenance fees are assumed to be zero in the baseline model.
3. **Option Premium Timing**: Option premiums (`PREM_PUT`, `PREM_CALL`) are paid upfront at t = 0. To maintain cash-flow comparability at maturity (t = 182), option premiums are compounded to t = 182 using the USD risk-free rate (`R_USD`).
4. **Credit & Borrowing Limits**: The firm is assumed to have sufficient credit capacity to execute euro borrowing without incurring credit spread surcharges or collateral calls.
5. **Tax & Accounting Rules**: Gains/losses across all strategies are evaluated on a pre-tax basis under standard USD functional currency accounting.

---

## Section 5: Calculation Flow

All formulas are specified strictly using named-range notation.

### 1. Baseline Unhedged Exposure at Maturity (t = 182)
`USD_Unhedged` = `FC_AMT` × `S_T`

### 2. Strategy 1: Forward Contract Hedge
`USD_Forward` = `FC_AMT` × `F0_in`

### 3. Strategy 2: Money-Market Hedge (Three-Step Pipeline)
- **Step 1 (Borrow Foreign Currency Today)**:  
  `FC_Borrow` = `FC_AMT` / (1 + (`R_FC` × (`T_DAYS` / `BASIS`)))
- **Step 2 (Convert Foreign Currency to USD at Spot)**:  
  `USD_Now` = `FC_Borrow` × `S0_in`
- **Step 3 (Invest USD Proceeds to Settlement)**:  
  `USD_MM` = `USD_Now` × (1 + (`R_USD` × (`T_DAYS` / `BASIS`)))

### 4. Strategy 3: Put Option Floor Hedge
- **Future Value of Put Premium Paid at t = 182**:  
  `FV_PREM_PUT` = -`PREM_PUT` × `FC_AMT` × (1 + (`R_USD` × (`T_DAYS` / `BASIS`)))
- **Net Proceeds at Settlement**:  
  `USD_PUT`(`S_T`) = MAX(`S_T`, `K_PUT`) × `FC_AMT` + `FV_PREM_PUT`

### 5. Strategy 4: Call Option Strategy (Long Call / Ceiling & Payoff)
- **Future Value of Call Premium Paid at t = 182**:  
  `FV_PREM_CALL` = -`PREM_CALL` × `FC_AMT` × (1 + (`R_USD` × (`T_DAYS` / `BASIS`)))
- **Net Proceeds at Settlement**:  
  `USD_CALL`(`S_T`) = MIN(`S_T`, `K_CALL`) × `FC_AMT` + `FV_PREM_CALL`

---

## Section 6: Sensitivity Plan

The model tests terminal spot rate movements at settlement (`S_T`) across a range of **-5% to +5% relative to inception spot (`S0_in`) in 1% increments** (11 steps total).

- **Independent Variable (`S_T`)**:  
  `S_T` = `S0_in` × (1 + Delta%), where Delta% ranges from -5% to +5% in 1% steps.
- **Evaluated Outputs**: Unhedged Proceeds, Forward Proceeds, Money-Market Proceeds, Net Put Proceeds, and Net Call Proceeds across all 11 scenarios.
- **Visualization Chart**: A line chart titled **"182-Day USD Cash Proceeds by Hedging Strategy vs. Terminal Spot Rate"** plotting `S_T` on the horizontal axis and USD Proceeds on the vertical axis.
- **CFO Decision Insight**: The embedded line chart provides executive leadership with a clear comparative view of risk asymmetry. Specifically, it highlights how the forward and money-market hedges lock in a certain cash flow (disregarding upside), while the put option establishes a hard downside cash floor ($14,879,435.99) while preserving exposure to favorable euro appreciation above `S0_in`. This visual directly supports the treasury recommendation of whether the firm should prioritize budget certainty or upside participation based on board risk tolerance.

---

## Section 7: Validation Rules & Parity Tolerance

The completed Stage 3 workbook must pass the following automated integrity checks:

1. **Covered Interest Rate Parity Check**:  
   Compute implied forward rate:  
   `F_implied` = `S0_in` × (1 + (`R_USD` × (`T_DAYS` / `BASIS`))) / (1 + (`R_FC` × (`T_DAYS` / `BASIS`)))  
   *Parity Gap Evaluation*: The model's CIP-implied forward is 1.1344, whereas the quoted forward `F0_in` is 1.1705, resulting in a valuation gap of $487,516.45 between `USD_MM` and `USD_FWD`. This variance is **not** a rounding artifact; it reflects real-world market basis, counterparty credit spreads, and institutional supply-demand imbalances in the FX swap market.  
   *Tolerance & Action Rule*: The model allows an allowable parity tolerance band of ± $5,000.00$ for purely technical execution discrepancies. Because the $487,516.45 gap severely breaches this tolerance, the model triggers an automated audit warning (`WARN: BASIS_MISMATCH`), and the treasury memo must flag that executing a synthetic money-market hedge rather than an outright forward would leave an unhedged basis exposure.
2. **Formula Integrity**: Zero hardcoded numbers in calculation cells. No spreadsheet error codes (`#REF!`, `#VALUE!`, `#N/A`, `#DIV/0!`).
3. **Option Floor Rule**: For all `S_T` <= `K_PUT`, net put proceeds must equal exactly `K_PUT` × `FC_AMT` + `FV_PREM_PUT` ($14,879,435.99).
4. **Forward Flat Line Rule**: Forward hedge proceeds must remain constant at $15,801,750.00 across all `S_T` sensitivity steps.

> *Note on §6.3 Auditability Checklist: Left unticked until `model.xlsx` is physically compiled and verified against these rules in Stage 3.*

---

## Section 8: Named Summary Outputs

| Output Name | Description | Cell Formatting Standard |
| :--- | :--- | :--- |
| `OUT_UNHEDGED_BASE` | Baseline unhedged proceeds at `S0_in` | Currency (`$#,##0.00`), Gray Fill |
| `OUT_FWD_PROCEEDS` | Fixed dollar proceeds under Forward Hedge | Currency (`$#,##0.00`), Gray Fill |
| `OUT_MM_PROCEEDS` | Fixed dollar proceeds under Money-Market Hedge | Currency (`$#,##0.00`), Gray Fill |
| `OUT_PUT_NET_FLOOR` | Minimum net dollar proceeds under Put Option | Currency (`$#,##0.00`), Gray Fill |
| `OUT_PARITY_GAP` | Absolute dollar variance between Forward and MM | Currency (`$#,##0.00`), Green/Red Flag |

---

## Appendix: Prompt Log & Review Iteration

### AI Review Iteration Log (`prompt-log.md`)

- **Initial Draft Defect Identified**: The initial specification draft omitted the explicit day-count basis for interest rate conversions and failed to compound the option premium to settlement date t = 182, creating an apples-to-oranges cash flow comparison between t = 0 option costs and t = 182 hedge proceeds.
- **AI Feedback Prompt**: *"Audit my §4 and §5 spec logic. Are cash flows properly aligned by time value of money?"*
- **Fix Applied**:  
  1. Updated §4 and §2 to mandate **ACT/360** day-count convention for all rate terms (`T_DAYS` / `BASIS`).
  2. Modified §5 Strategy 3 to compute `FV_PREM_PUT` compounded at rate `R_USD` over `T_DAYS`, ensuring net option proceeds reflect true t = 182 dollars.
