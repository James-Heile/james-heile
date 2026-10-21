---
type: brief
engagement: fx-hedging
capability: fx-hedging
date: 2026-10-21
status: committed
hypothesis: "I expect a forward hedge to provide the most predictable USD proceeds, while a put option preserves upside at the cost of a premium."
---

# FX Hedging — Executive Brief

**To:** Chief Financial Officer  
**Subject:** Managing EUR/USD Exposure on a €4.5 Million Receivable

## The Exposure and Business Risk

Our firm expects to receive **€4,500,000 in one year** and convert the proceeds into US dollars. Although the euro amount and expected settlement date are fixed, the dollar value is not. Our budget therefore faces uncertainty from movements in the EUR/USD exchange rate.

At an exchange rate of 1.10 USD per EUR, the receivable would generate $4,950,000. If the euro weakens to 1.00, the same receivable would generate only $4,500,000—a $450,000 reduction in dollar proceeds. A 5% decline from 1.10 to 1.045 would reduce proceeds by $247,500. This is a material potential shortfall that could affect cash-flow forecasts, operating budgets, and financial planning.

The business question is how much certainty we need over future dollar proceeds and what we should be willing to pay, or give up, to obtain it.

## Three Hedging Alternatives

**Forward contract or futures.** We can agree today to exchange euros for dollars at a predetermined rate on a future date. This locks in the conversion value and makes budgeting more predictable. The trade-off is that we surrender additional gains if the euro strengthens; counterparty risk, contract terms, and potential margin requirements also need consideration.

**Money-market hedge.** We can borrow against the future euro receivable, convert the euros into dollars today, and invest the dollars until the receivable arrives. This can create a known dollar value using spot and interest rates. However, it requires financing capacity and introduces additional transactions and operational complexity.

**Put option on the euro.** We can purchase the right, but not the obligation, to sell euros at a specified exchange rate. This establishes a minimum conversion value while preserving the opportunity to benefit if the euro strengthens. Like insurance, the protection comes at a price: the option premium is paid whether or not the protection is ultimately needed.

## Assumptions and Hypothesis

For the initial analysis, I assume the firm will receive the full €4.5 million in one year, will convert the entire amount into USD, and has no existing hedge. I also assume the receivable is sufficiently certain to plan around. The initial model will use illustrative exchange rates and financing inputs rather than treating them as current market quotes. With more time, I would verify the settlement date, receivable certainty, available credit capacity, counterparty terms, and live forward and option prices.

**Hypothesis:** I expect a forward hedge to provide the most predictable USD proceeds, while a put option preserves upside at the cost of a premium. The money-market hedge may provide comparable certainty, depending on interest rates and financing costs.

I would reconsider this hypothesis if verified market inputs showed that the money-market hedge produced a materially better net outcome, or that the option's premium was justified by its protection and upside potential under the firm's risk tolerance.

## Next Steps

Stage 2 will specify the model's workbook structure, formulas, named ranges, and validation checks. Stage 3 will build and audit the workbook with AI assistance. Stage 4 will replace illustrative inputs with sourced, timestamped market data. Stage 5 will independently validate the calculations, verify key results, and document the final recommendation in a decision memo.

No hedge is recommended at this stage. The objective is to build a transparent, testable analysis before committing company funds or limiting future currency gains.
