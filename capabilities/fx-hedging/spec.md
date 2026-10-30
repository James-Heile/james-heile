# University of Hawaiʻi at Mānoa · Shidler College of Business
# FIN 321: International Finance and Securities
# FX Transaction Hedging Project — Technical Specification

## Soybean Merchant (Scenario 14) — FX Transaction Hedge Model · Technical Specification

Technical specification for the FX transaction hedge model — the named-range contract, calculation flow, and validation checks, precise enough that an AI or a colleague could build (or rebuild) the workbook from this document alone. This spec is the input the AI-assisted build works from.

| Field | Value |
| :--- | :--- |
| **Created by** | James Heile |
| **Updated by** | James Heile |
| **Date Created** | 2026-10-28 |
| **Date Updated** | 2026-10-28 |
| **Version** | 1.0 |
| **LLM Used** | ChatGPT |
| **Role** | Treasury Analyst / FP&A Analyst |
| **Audience** | CFO / Director of Treasury |
| **Companion Workbook** | student build (`model.xlsx`) |

---

## 1. Problem Statement

A U.S. soybean merchandiser has sold oilseed to a German crusher and invoiced the transaction in euros, resulting in a foreign-currency receivable of **€13,500,000** due in **182 days**. Because the firm reports in USD, it faces cash flow and earnings volatility driven by fluctuations in the EUR/USD exchange rate. The primary objective is to protect the firm's USD cash inflows against adverse euro depreciation while evaluating alternative hedging structures (forward, money-market, and put option) to balance budgeting certainty against upside optionality. This analysis is prepared for executive treasury review under board-approved corporate risk management guidelines.

---

## 2. Inputs (Known Variables)

All inputs are exposed as workbook named ranges so the calculation flow reads identically whether implemented in Excel, Python, or an AI prompt. Dates, sources, and access timestamps are recorded in the Notes tab. Market inputs (spot, forward, rates, premia) are the adjustable scenario cells.

### 2.1 Core Inputs

| Standardized Name | Description | Unit | Example / Scenario Value |
| :--- | :--- | :--- | :--- |
| `FC_AMT` | Foreign-currency notional (receivable) | EUR | 13,500,000 |
| `S0_in` | Spot exchange rate at inception | USD per EUR | 1.1221 |
| `F0_in` | Forward rate to settlement | USD per EUR | 1.1705 |
| `R_USD` | USD interest rate to settlement | Annual % | 5.30% (0.0530) |
| `R_FC` | Foreign-currency interest rate to settlement | Annual % | 3.10% (0.0310) |
| `T_DAYS` | Days to settlement | Days | 182 |
| `BASIS` | Day-count denominator (single-value simplification) | Days | 360 (ACT/360 money-market convention) |
| `K_PUT` | Put option strike (receivables) | USD per EUR | 1.1221 (At-the-money spot) |
| `PREM_PUT` | Put premium, USD per 1 FC | USD | 0.0194 |

### 2.2 Derived / Intermediate Values

| Name | Description | Formula / Source |
| :--- | :--- | :--- |
| `DF_USD` | USD accumulation factor | $1 + R_{\text{USD}} \times (T_{\text{DAYS}} / \text{BASIS})$ |
| `DF_FC` | Foreign currency accumulation factor | $1 + R_{\text{FC}} \times (T_{\text{DAYS}} / \text{BASIS})$ |
| `FV_PREM_PUT` | Future value of put premium at settlement | $-\text{PREM\_PUT} \times \text{FC\_AMT} \times (1 + R_{\text{USD}} \times T_{\text{DAYS}}/\text{BASIS})$ |
| `S_T_grid` | Sensitivity spot grid at settlement | Built from $S_{0,\text{in}} \pm 5\%$ in $1\%$ steps |
| `USD_NO_HEDGE` | USD proceeds under no hedge | $S_T \times \text{FC\_AMT}$ |

---

## 3. Assumptions & Constraints

- **Quote convention:** All rates expressed as USD per unit of foreign currency (USD/EUR). A higher quote means euro appreciation.
- **Horizon:** Single-maturity model; $T_{\text{DAYS}} = 182$ days.
- **Day-count basis:** Uses the money-market convention of **ACT/360** ($\text{BASIS} = 360$) for both USD and EUR legs.
- **Parity:** Money-market hedge is assumed to replicate the forward hedge under covered interest-rate parity; any gap is treated as a test of input pricing consistency rather than a model error.
- **Option premium:** Paid upfront in USD, quoted per 1 unit of FC. Expressed as a negative cash flow at $t_0$ and compounded forward at $R_{\text{USD}}$ to align with settlement-date proceeds.
- **Counterparty / credit risk:** Excluded. All derivatives assumed frictionless and creditworthy.
- **Transaction costs & bid-ask spreads:** Excluded from the base case.
- **Tax / accounting treatment:** Excluded. Model reports pre-tax cash outcomes only.
- **Scenario construction:** Future spot $S_T$ is varied deterministically across a $-5\%$ to $+5\%$ grid in $1\%$ increments.

---

## 4. Calculation Flow

Described in named-range pseudocode. Formulas are written for a receivable exposure.

### Step 1 — Derived inputs
1. `DF_USD` $= 1 + \text{R\_USD} \times \text{T\_DAYS} / \text{BASIS}$
2. `DF_FC` $= 1 + \text{R\_FC} \times \text{T\_DAYS} / \text{BASIS}$
3. `FV_PREM_PUT` $= -\text{PREM\_PUT} \times \text{FC\_AMT} \times \text{DF\_USD}$

### Step 2 — Forward hedge (certainty benchmark)
- `USD_FWD` $= \text{FC\_AMT} \times \text{F0\_in}$
- Locked-in at $t_0$; invariant across the $S_T$ grid.

### Step 3 — Money-market hedge (parity check)
1. Borrow $\text{FC\_AMT} / \text{DF\_FC}$ foreign currency today (discounted PV of receivable).
2. Convert to USD at spot: $(\text{FC\_AMT} / \text{DF\_FC}) \times \text{S0\_in}$.
3. Invest USD to maturity: $\text{USD\_MM} = (\text{FC\_AMT} / \text{DF\_FC}) \times \text{S0\_in} \times \text{DF\_USD}$.
4. At maturity, the EUR receivable repays the euro loan exactly, leaving the USD investment proceeds.
- **Parity check:** $\text{USD\_MM} \approx \text{USD\_FWD}$ within rounding.

### Step 4 — Option hedge (floor with upside)
Put-and-hold strategy on the receivable:
- Pay $\text{PREM\_PUT} \times \text{FC\_AMT}$ USD today for a put option with strike `K_PUT`.
- At settlement, for each $S_T$ on the grid:
  - $\text{USD\_PUT}(S_T) = \max(S_T, \text{K\_PUT}) \times \text{FC\_AMT} + \text{FV\_PREM\_PUT}$

### Step 5 — Sensitivity table (rows of the grid)
For each $S_T$ in `S_T_grid`:
- **No hedge:** $S_T \times \text{FC\_AMT}$
- **Forward:** `USD_FWD` (constant)
- **Money market:** `USD_MM` (constant)
- **Option (put):** $\max(S_T, \text{K\_PUT}) \times \text{FC\_AMT} + \text{FV\_PREM\_PUT}$
- **Hedge profit:** $\text{USD}_k - \text{USD\_NO\_HEDGE}$ for each strategy $k$.
- **Winner labels:** `ARGMAX` across strategies.

### Step 6 — Summary metrics (scalar outputs)
- $\text{USD\_FLOOR\_PUT} = \min(\text{USD\_PUT})$ across `S_T_grid`.
- $\text{USD\_BASE\_k} = \text{USD}_k$ evaluated at $S_T = \text{S0\_in}$.

---

## 5. Outputs

| Output Description | Format / Location | Purpose |
| :--- | :--- | :--- |
| **Input panel** | Top of each tab | Single source of truth for named ranges |
| **Strategy summary** | Table above sensitivity grid | Executive at-a-glance baseline comparison |
| **Sensitivity table** | Core table on sensitivity tab | Core analytical evidence across $\pm 5\%$ spot range |
| **Hedge-profit columns** | Sub-table within sensitivity grid | Isolates strategy value-add vs. unhedged exposure |
| **Winner labels** | Label columns | Quick-read decision cue |
| **Sensitivity chart** | Embedded line chart | Visual comparison of payout profiles |

### 5.1 Computed Base-Case Values ($S_T = S_{0,\text{in}} = 1.1221$)

| Strategy | USD Proceeds (Receivable) | Hedge Profit vs. No Hedge |
| :--- | :--- | :--- |
| **No hedge** | $15,148,350.00 | $0.00 |
| **Forward** | $15,801,750.00 | +$653,400.00 |
| **Money market** | $15,314,233.55 | +$165,883.55 |
| **Option (put)** | $14,879,435.99 | -$268,914.01 |

---

## 6. Model Review — What Worked & What to Improve

### 6.1 What Worked
- The three-step money-market pipeline accurately mirrors the borrowing, spot conversion, and reinvestment mechanics.
- Upward compounding of option premium via `DF_USD` correctly places upfront cash flows on a time-equivalent $t = 182$ settlement basis.

### 6.2 What to Improve
- Incorporate live API rate pulls in Stage 4 to reconcile the quoted forward rate (`1.1705`) with covered interest-rate parity implied rates.

### 6.3 Auditability Checklist
- [x] Every input has a standardized named range from §2.1.
- [x] Every formula in §4 uses named ranges — no bare cell references.
- [x] Money-market hedge ties to forward hedge mechanics.
- [x] Put payoff at $S_T = K_{\text{PUT}}$ verified against strike and premium future value.
- [x] Sensitivity grid is symmetric around $S_T = S_{0,\text{in}}$ in 1% steps.
- [x] Notes tab records ECB reference spot sources and access dates.
- [x] Cell colors match formatting standards.

---

## 7. Sensitivity Plan

- **Grid:** `S_T_grid` spans $1.1221 \times (1 \pm 5\%)$ in $1\%$ increments (11 rows total).
- **Strategies plotted:** No hedge, Forward, Money market, Put option.
- **Primary chart:** Line chart with $S_T$ on the x-axis and USD proceeds on the y-axis.
- **Secondary table:** Hedge profit vs. no hedge to quantify risk/reward profiles.

---

## 8. Limitations & Next Steps

This specification treats the hedge statically at $t_0$, excludes credit/counterparty risk, and relies on scenario-driven premium inputs rather than Black-Scholes volatility outputs. Future stages will translate this evidence into the Stage 5 CFO decision memo.

---

## Appendix A — Change Log

| Version | Date | Author | Change |
| :--- | :--- | :--- | :--- |
| 1.0 | 2026-10-28 | James Heile | Initial comprehensive spec for Scenario 14 (Soybean Merchant) |
