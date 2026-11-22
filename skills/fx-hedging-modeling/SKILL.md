---
name: fx-hedging-modeling
description: Model foreign exchange exposure, compare hedging strategies (unhedged, forward, money-market, put floor, written call), check covered interest parity (CIP), run spot sensitivity scenarios, and structure executive hedging recommendations. Use when analyzing FX receivables or payables, building FX hedging specs or models, checking CIP parity gaps, or drafting CFO currency hedging memos.
---

# FX Hedging Modeling

## Summary
Guide end-to-end FX exposure analysis, strategy comparison, covered interest parity (CIP) checks, sensitivity analysis, and executive hedging recommendations for receivables and payables.

## When to Use
Use this skill when:
- Analyzing foreign exchange exposure on a receivable or payable
- Comparing unhedged, forward, money-market, put option, or written call hedging strategies
- Computing CIP-implied forward rates and diagnosing parity or basis gaps
- Running terminal spot sensitivity scenarios across percentage spot shifts
- Drafting a decision memo or recommendation comparing hedging trade-offs

## Steps
1. **Confirm Exposure and Day-Count Convention:** Identify the foreign currency notional, settlement horizon in days, day-count basis (such as ACT/360), inception spot rate, domestic and foreign interest rates, quoted forward rate, strikes, and option premiums. Separate live market quotes from scenario assumptions.
2. **Compute Baseline and Forward Proceeds:** Calculate unhedged terminal proceeds across terminal spot scenarios and fixed settlement proceeds under the forward contract.
3. **Run the Three-Step Money-Market Hedge:** Compute the foreign-currency amount to borrow or fund at inception, convert to domestic currency at inception spot, and compound or discount to maturity at the domestic interest rate.
4. **Evaluate Option Payoffs and Time Alignment:** Compound or discount upfront option premiums to the settlement date using the domestic interest rate so cash flows align on the same timeline. Model put floors and written call caps across spot scenarios.
5. **Check Covered Interest Parity (CIP):** Compute the theoretical CIP-implied forward rate from spot and interest rate differentials. Compare it against the quoted forward rate to measure the rate gap and dollar notional gap, and note when basis mismatches affect execution or requote strategy.
6. **Synthesize Recommendation and Trade-Offs:** Compare certainty, cost, liquidity, and upside participation across scenarios, then state a clear recommendation tied to the firm's risk posture.

## Gotchas
- Always align upfront option premiums with the settlement date using time value of money rather than adding raw inception cash flows directly to maturity proceeds.
- Keep quoted or assumption rates distinct from CIP-implied rates when a large parity gap appears, and treat wide gaps as a requote or basis check rather than a rounding error.
- State clearly whether terminal spot scenarios use exact unrounded spot shifts or rounded display rates before comparing tables.
