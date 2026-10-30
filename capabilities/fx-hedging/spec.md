# FX Hedging Model Specification: Scenario 14 (Soybean Merchant)

**Document Path:** `capabilities/fx-hedging/spec.md`  
**Engagement:** FX Hedging — Stage 2 Model Specification  
**Author:** Student Analyst  
**Date:** October 23, 2026  
**Status:** Draft for Review & Audit Specification  

---

## Section 1: Problem Statement

A U.S. soybean merchandiser has sold soybeans to a German oilseed crusher and invoiced the customer in euros. The firm holds a foreign-currency receivable of **€13,500,000**, with payment due in **182 days (6 months)**. 

Because the invoice is denominated in EUR and the firm reports in USD, the firm is exposed to EUR/USD foreign exchange volatility over the 182-day holding period. If the euro depreciates against the U.S. dollar, the firm will realize fewer USD proceeds upon settlement, creating potential cash flow deficits, operating budget shortfalls, and compressed profit margins. 

The objective of this financial model is to quantify the baseline unhedged exposure and evaluate three distinct hedging strategies—a **Forward Contract**, a **Money-Market Hedge**, and a **Put Option Floor**—to provide executive leadership with a structured risk-return framework for decision-making.

---

## Section 2: Inputs — The Named-Range Contract

All inputs are stored on the `Inputs` tab and bound to explicit named ranges. All interest rates and spot/forward quotes follow market conventions using an **ACT/360 day-count basis**.

| Named Range | Description | Placeholder Value | Unit | Stage 4 Data Source |
| :--- | :--- | :--- | :--- | :--- |
| `FC_AMT` | Foreign-currency receivable | `13500000` | EUR | Commercial Invoice (Fixed contract parameter) |
| `S0_in` | Spot exchange rate at inception | `1.1221` | USD per EUR | ECB Reference Rate (October 23, 2026) |
| `F0_in` | 182-day forward exchange rate | `1.1705` | USD per EUR | Indicative market quote *(Replaced at Stage 4)* |
| `R_USD` | USD annualized interest rate | `5.30%` | Annual % (ACT/360) | Federal Reserve H.15 6-Month Treasury / SOFR *(Placeholder)* |
| `R_FC` | EUR annualized interest rate | `3.10%` | Annual % (ACT/360) | ECB Main Refinancing Rate / 6M EURIBOR *(Placeholder)* |
| `K_PUT` | Put option strike price | `1.1221` | USD per EUR | Market quote (Set at-the-money spot `S0_in`) |
| `K_CALL` | Call option strike price | `1.1221` | USD per EUR | Market quote (Set at-the-money spot `S0_in`) |
| `PREM_PUT` | Put option premium per unit of FC | `0.0194` | USD per EUR | Indicative option quote *(Replaced at Stage 4)* |
| `PREM_CALL` | Call option premium per unit of FC | `0.0296` | USD per EUR | Indicative option quote *(Replaced at Stage 4)* |
| `T_DAYS` | Time to settlement | `182` | Days | Contract maturity (Fixed contract parameter) |

> *Note: All numerical values listed above are indicative placeholders and will be updated with live market data at Stage 4.*

---

## Section 3: Tab Architecture

The workbook `capabilities/fx-hedging/model.xlsx` consists of six required tabs organized logically to support dynamic calculations and auditing:

1. **`Cover`**: Executive overview, project metadata, author info, scenario definition, and stage status.
2. **`Legend_Key`**: Formatting standards (e.g., blue text / yellow fill for hardcoded inputs, black text for formulas, gray fill for key summary outputs), color codes, and table of named range definitions.
3. **`Inputs`**: Centralized data entry table for all ten contract named ranges. No hardcoded constants exist outside this tab.
4. **`Hedge_Calculations`**: Active calculation engine detailing the step-by-step logic for Unhedged, Forward Contract, Money-Market Hedge, and Put Option strategies.
5. **`Sensitivity`**: Scenario analysis engine modeling terminal spot rate ($S_T$) shifts from $-5\%$ to $+5\%$ in $1\%$ increments, containing the comparative summary table and visualization chart.
6. **`Notes_Assumptions`**: Institutional documentation of day-count conventions, transaction cost policies, credit assumptions, and mathematical models.

---

## Section 4: Assumptions & Constraints

1. **Day-Count Convention**: All annualized interest rates (`R_USD`, `R_FC`) use the money-market convention of **ACT/360**. The time fraction is calculated strictly as `T_DAYS / 360`.
2. **Transaction Costs & Spreads**: Transaction costs, bid-ask spreads, and credit line maintenance fees are assumed to be zero in the baseline model.
3. **Option Premium Timing**: Option premiums (`PREM_PUT`, `PREM_CALL`) are paid upfront at $t=0$. To maintain cash-flow comparability at maturity ($t=182$), option premiums are compounded to $t=182$ using the USD risk-free rate (`R_USD`).
4. **Credit & Borrowing Limits**: The firm is assumed to have sufficient credit capacity to execute euro borrowing without incurring credit spread surcharges or collateral calls.
5. **Tax & Accounting Rules**: Gains/losses across all strategies are evaluated on a pre-tax basis under standard USD functional currency accounting.

---

## Section 5: Calculation Flow

All formulas are specified strictly using named-range notation.

### 1. Baseline Unhedged Exposure at Maturity ($t = 182$)
$$\text{USD\_Unhedged} = \text{FC\_AMT} \times S_T$$

### 2. Strategy 1: Forward Contract Hedge
$$\text{USD\_Forward} = \text{FC\_AMT} \times \text{F0\_in}$$

### 3. Strategy 2: Money-Market Hedge (Three-Step Pipeline)
* **Step 1 (Borrow Foreign Currency Today)**: Size the euro loan so that maturity principal and interest exactly equal the receivable amount:
  $$\text{FC\_Borrow} = \frac{\text{FC\_AMT}}{1 + \left(\text{R\_FC} \times \frac{\text{T\_DAYS}}{360}\right)}$$
* **Step 2 (Convert Foreign Currency to USD at Spot)**: Convert the borrowed euros to dollars at inception:
  $$\text{USD\_Now} = \text{FC\_Borrow} \times \text{S0\_in}$$
* **Step 3 (Invest USD Proceeds to Settlement)**: Invest USD proceeds at the domestic interest rate to $t=182$:
  $$\text{USD\_MM\_Proceeds} = \text{USD\_Now} \times \left(1 + \left(\text{R\_USD} \times \frac{\text{T\_DAYS}}{360}\right)\right)$$

### 4. Strategy 3: Put Option Floor Hedge
* **Gross Proceeds at Settlement Spot $S_T$**:
  $$\text{USD\_Put\_Gross} = \text{FC\_AMT} \times \max(S_T, \text{K\_PUT})$$
* **Future Value of Premium Paid at $t=182$**:
  $$\text{PREM\_PUT\_FV} = (\text{FC\_AMT} \times \text{PREM\_PUT}) \times \left(1 + \left(\text{R\_USD} \times \frac{\text{T\_DAYS}}{360}\right)\right)$$
* **Net Proceeds at Settlement**:
  $$\text{USD\_Put\_Net} = \text{USD\_Put\_Gross} - \text{PREM\_PUT\_FV}$$

---

## Section 6: Sensitivity Plan

The model tests terminal spot rate movements at settlement ($S_T$) across a range of **$-5\%$ to $+5\%$ relative to inception spot (`S0_in`) in $1\%$ increments** (11 steps total).

* **Independent Variable ($S_T$)**:  
  $$S_T = \text{S0\_in} \times (1 + \Delta\%)$$  
  where $\Delta\% \in \{-5\%, -4\%, -3\%, -2\%, -1\%, 0\%, +1\%, +2\%, +3\%, +4\%, +5\%\}$.
* **Evaluated Outputs**: Unhedged Proceeds, Forward Proceeds, Money-Market Proceeds, and Net Put Option Proceeds across all 11 scenarios.
* **Visualization Chart**: A line chart titled **"182-Day USD Cash Proceeds by Hedging Strategy vs. Terminal Spot Rate"** plotting $S_T$ on the horizontal axis and USD Proceeds on the vertical axis.
* **CFO Decision Insight**: The chart illustrates the downside protection (floor) provided by the put option, the fixed return lock-in of the forward/MM hedges, and the unhedged downside vulnerability.

---

## Section 7: Validation Rules (Check Figures & Audit Criteria)

The completed Stage 3 workbook must pass the following automated integrity checks:

1. **Covered Interest Rate Parity Check**:  
   Compute implied forward rate:
   $$F_{\text{implied}} = \text{S0\_in} \times \frac{1 + \left(\text{R\_USD} \times \frac{\text{T\_DAYS}}{360}\right)}{1 + \left(\text{R\_FC} \times \frac{\text{T\_DAYS}}{360}\right)}$$
   *Validation Rule*: $|\text{USD\_MM\_Proceeds} - (\text{FC\_AMT} \times F_{\text{implied}})| < \$0.01$. If non-zero, trigger audit flag `ERR: PARITY_DISCREPANCY`.
2. **Formula Integrity**: Zero hardcoded numbers in calculation cells. No spreadsheet error codes (`#REF!`, `#VALUE!`, `#N/A`, `#DIV/0!`).
3. **Option Floor Rule**: For all $S_T \le \text{K\_PUT}$, $\text{USD\_Put\_Gross}$ must exactly equal $\text{FC\_AMT} \times \text{K\_PUT}$.
4. **Forward Flat Line Rule**: Forward hedge proceeds must remain constant across all $S_T$ sensitivity steps.

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

* **Initial Draft Defect Identified**: The initial specification draft omitted the explicit day-count basis for interest rate conversions and failed to compound the option premium to settlement date $t=182$, creating an apples-to-oranges cash flow comparison between $t=0$ option costs and $t=182$ hedge proceeds.
* **AI Feedback Prompt**: *"Audit my §4 and §5 spec logic. Are cash flows properly aligned by time value of money?"*
* **Fix Applied**: 
  1. Updated §4 and §2 to mandate **ACT/360** day-count convention for all rate terms (`T_DAYS / 360`).
  2. Modified §5 Strategy 3 to compute $\text{PREM\_PUT\_FV}$ compounded at rate `R_USD` over `T_DAYS`, ensuring net option proceeds reflect true $t=182$ dollars.
