## 0. Strategy Definition (Decision Tree)

### Strategy Claim
If corporate bond carry (carry + rolldown − funding − costs),
measured ex-ante using observable curves and TRACE prices,
is positive and stable cross-sectionally,
then a long-only / long-short portfolio formed on this signal
earns excess returns net of realistic execution costs.

### Decision Tree

1. Universe Selection
   - Include bond *i* at time *t* IF:
     - Rating ∈ {BBB, BB}
     - Time-to-maturity ∈ [X, Y] years
     - Issue size ≥ Z
     - TRACE trading frequency ≥ N trades / month
   - ELSE exclude

2. Signal Construction
   - For each eligible bond *i* at time *t*:
     - Compute carry_i,t = coupon accrual − funding_i,t
     - Compute rolldown_i,t = price impact of Δt roll on curve
     - Total expected carry_i,t = carry_i,t + rolldown_i,t

3. Funding Cost
   - funding_i,t defined as:
     - sector-matched repo proxy OR
     - OIS + spread_s(t)
   - Haircut assumption = H%
   - If funding_i,t > expected carry_i,t → bond rejected

4. Portfolio Formation
   - Rank bonds by expected carry
   - Long top K% (or long–short top vs bottom)
   - Weights = equal / duration-neutral / risk-weighted (choose one)

5. Execution & Costs
   - Slippage model = X bps per trade
   - Fill probability = f(liquidity bucket)
   - If fill rate < 70% → strategy fails

6. Evaluation
   - Holding horizon = Δ
   - Rebalance frequency = monthly
   - Metrics:
     - annualized carry
     - carry / total return attribution
     - turnover
     - slippage
