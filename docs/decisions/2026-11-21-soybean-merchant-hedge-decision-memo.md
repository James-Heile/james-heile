# Treasury Decision Memo: EUR/USD Hedging Strategy for 182-Day Soybean Receivable

**To:** Chief Financial Officer  
**From:** James Heile  
**Date:** November 21, 2026  
**Subject:** Hedging Recommendation for €13,500,000 Receivable (182-Day Maturity)  

---

## A. Exposure Summary

Our firm holds a **€13,500,000** trade receivable from a German oilseed crusher settling in **182 days** (`ACT/360`). At the November 20, 2026 ECB EUR/USD reference spot rate of **1.0990**, the unhedged receivable carries a baseline value of **$14,836,500.00**.

Because soybean merchandising operates on disciplined physical basis and processing margins, unhedged EUR/USD volatility directly threatens realized USD gross margin over the next six months. A 5% euro depreciation to **1.04405** reduces unhedged proceeds by **$741,825.00** (to **$14,094,675.00**), while a 5% euro appreciation to **1.15395** adds **$741,825.00** (to **$15,578,325.00**).

---

## B. Hedge Outcomes

Evaluating all structures at $t = 182$ using live November 20, 2026 rates (`R_USD = 4.51%`, `R_FC = 3.00%`, ATM strikes `K_PUT = K_CALL = 1.0990`) and scenario option premiums (`PREM_PUT = 0.0194`, `PREM_CALL = 0.0296`, compounded forward at `R_USD`):

* **Forward Contract (`F0_in = 1.1705`):** Locks in **$15,801,750.00** across all spot outcomes—eliminating FX uncertainty completely and delivering **+$965,250.00** above the unhedged spot baseline (`$14,836,500.00`) if the quoted rate is executable.
* **Money-Market Hedge (3-Step Pipeline):** Borrows **€13,298,308.98** today (`3.00%`), converts to **$14,614,841.57** at `1.0990`, and invests at `4.51%` to lock in **$14,948,068.08** (`+$111,568.08` over the unhedged spot baseline due to positive USD–EUR rate carry), while requiring euro credit capacity and balance-sheet funding execution.
* **Put Option Floor (`K_PUT = 1.0990`):** Establishes a hard net floor of **$14,568,628.53** after compounding the `$261,900.00` upfront premium to **-$267,871.47**, while preserving upside participation (`$15,310,453.53` at `+5%` spot). It trades a guaranteed worst-case floor against upfront premium drag and an upside breakeven hurdle above spot.
* **Written Call Overlay (`K_CALL = 1.0990`):** Generates **$408,711.10** in compounded premium inflows (`$399,600.00` upfront at the bank's bid quote) and caps proceeds at **$15,245,211.10** at or above `1.0990`, but leaves full downside exposure on a euro drop (**$14,503,386.10** at `-5%` spot), making it ill-suited as a standalone downside hedge.

---

## C. Sensitivity Interpretation

Across a **-5% to +5%** terminal spot range (`1.04405` to `1.15395`):

* **EUR Depreciation (`-5%` to `1.04405`):** Certainty wins. Forward (**$15,801,750.00**) and Money-Market (**$14,948,068.08**) insulate cash flows completely, whereas Unhedged (**$14,094,675.00**) and the Written Call (**$14,503,386.10**) suffer steep USD shortfalls. The Put Option holds firm at **$14,568,628.53**, outperforming unhedged by **+$473,953.53** net of compounded premium.
* **EUR Appreciation (`+5%` to `1.15395`):** Flexibility and pricing matter. Unhedged (**$15,578,325.00**) and the Put Option (**$15,310,453.53**) capture higher EUR/USD levels, while the Money-Market (**$14,948,068.08**) and Written Call (**$15,245,211.10**) leave appreciation on the table. Only the static `1.1705` forward quote sits higher (**$15,801,750.00**)—and only if that rate is genuinely available in the market.

---

## D. Recommendation & CIP Parity Check

**Recommended Action — Request an Immediate Bank Requote on the Forward, Execute the Forward if `1.1705` Is Honored, and Otherwise Lock in the Money-Market Hedge (or Put Floor if Bullish EUR):**

1. **CIP Parity Break & Gap:** Using `ACT/360`, the Covered Interest Parity (CIP) implied forward rate is **1.1073** (`1.107264`), which implies **$14,948,068.08** on €13.5M. Compared against the scenario's **1.1705** forward quote (**$15,801,750.00**), there is a **+0.063236 USD/EUR (+632.4 pips, or +5.71%)** rate gap and an **$853,681.92** USD parity gap (`WARN: BASIS_MISMATCH` beyond our `±$5,000.00` tolerance).
2. **What Explains the Gap:** While `S0_in` moved down from `1.1221` to `1.0990` and money-market rates sit at `4.51%` (USD) and `3.00%` (EUR), the `1.1705` forward quote reflects a stale/off-market indicative figure rather than live no-arbitrage pricing (or an extreme cross-currency basis/funding anomaly).
3. **Does It Change the Recommendation?** **Yes—it governs execution discipline.** If the bank actually honors **1.1705**, we should execute the forward immediately because it locks in **$15,801,750.00** across all scenarios. If the bank reprices the forward closer to CIP parity (**~1.1073**), the windfall disappears and we should execute the **Money-Market Hedge** at **$14,948,068.08** for full budget certainty (or the **Put Option Floor** if the CFO wants upside participation above `1.1073` and accepts the `$14,568,628.53` floor).

---

## E. Executive Justification

For a soybean merchandiser, **budget certainty and margin protection** come ahead of directional currency speculation. Locking in USD proceeds via a verified forward or money-market hedge aligns FX cash flows with physical merchandising commitments, avoids unhedged downside shocks if EUR weakens, and preserves liquidity without paying upfront option premiums—provided we confirm live forward pricing with the bank before trading.
