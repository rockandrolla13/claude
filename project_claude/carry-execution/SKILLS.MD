---
name: carry-execution
description: >
  Execute pre-registered carry strategy backtest with full audit trail.
  Use when running carry backtests, computing P&L attribution, or validating
  against kill criteria. Enforces pre-commitment protocol.
---

# Carry Strategy Execution Protocol

## Pre-Conditions (verify before ANY execution)
- [ ] Pre-registration document exists and is frozen
- [ ] Universe specification locked
- [ ] Horizon Δ declared
- [ ] Weighting scheme fixed ex-ante
- [ ] Kill criteria defined

## Strategy Specification

### Universe
- `carry_strategy_structural_v0` on USD BB/BB+ bonds
- SD < 2.5, horizon |Δ|Delta < 21d
- ID, universe, horizon frozen before work

### Carry Definition
```
C_it = coupon accrual + credit roll-down - funding - expected loss
```

### Hypothesis (stated ex-ante)
| Component | Economic Meaning | Falsifier |
|-----------|------------------|-----------|
| Carry compensation | Short-horizon credit provides value transfer from borrower to lender | Must be stated ex-ante; "it works in backtests" invalid |
| Time-passing carry | Higher time-passing carry → higher excess return under structural constraints | Hypothesis written before any data access |
| P&L decomposition | C_it(+) coupon accrual + roll-down - funding - expected loss (PD×LGD) | Must be computable ex-ante, net of funding |

## Test Design A: Cross-Sectional Regression

### Commitment
```
Cross-sectional reg: R_i(t+Δ|Delta) ~ C_i(t,t) + DV01, SDV01, sector, rating, liquidity
```

### Procedure
1. At each date t, compute carry C_it for every bond using only information available at t
2. Measure realized excess return R_i,t+Δ over fixed horizon Δ
3. Run cross-sectional regression each date
4. Aggregate coefficients across time (average slope)

### Decision Rule
- Evidence requires average carry slope POSITIVE and STABLE across time
- Not explained away by controls

### Non-Negotiable
- Reg spec, SEs, winsorization frozen
- Fixed before results are seen

## Test Design B: Tradable Long/Short Portfolio

### Commitment
- Carry-sorted L/S (top vs bottom quantile)
- Weights fixed ex-ante

### Procedure
1. Compute carry C_i(t,t) for all bonds using only info available at t
2. Rank bonds by C_i(t,t)
3. Define long set L_t = bonds in top carry quantile
4. Define short set S_t = bonds in bottom carry quantile
5. Assign pre-defined weights w_i:
   - w_i > 0 for i in L_t
   - w_i < 0 for i in S_t
   - Weighting: equal risk or equal notional (declared ex-ante)
6. Impose ex-ante constraints:
   - Σ_i w_i × DV01_i = 0  (rate neutrality)
   - Σ_i w_i × SDV01_i = 0 (spread neutrality)
7. Optional constraints (declared ex-ante):
   - Sector exposure caps
   - Rating exposure caps
   - Liquidity filters (min size, max bid-ask)
8. Freeze portfolio weights w_i at date t
9. Hold portfolio over horizon Δ
10. Compute realized net excess return:
```
    R_p(t+Δ) = portfolio P&L - transaction costs - funding/repo costs
```
11. Repeat steps 1-10 for all rebalance dates

### Portfolio Rules (immutable)
- Max position: ±1.0 per bond
- Dollar neutral, Beta neutral
- Duration, rating, sector all neutralized
- Max turnover (future use): 10%

## Post-Trade TCA & Attribution

### Commitment
- Return decomposition mandatory
- Use CBBT where available to check execution costs

### Output
- Realized return, realized carry
- Breakdown: % of return from carry vs unexplained
- Validate structural premise holds

### Time-Series Model
```
R_t+Δ = α + βC_t + ε
```
- P&L should be carry + noise
- If β|b ≠ 0 or unstable → weaken hypothesis

## Counterfactual Validation
- High carry must coincide with positive returns
- Low carry with weak returns
- Tests mechanism, not outcome
- If not, reject carry explanation

## Success Criteria (ALL must hold)
- Ann. carry > 1%
- Carry share > 60%
- Neutrality holds
- Fill > 85%
- DD < 1.5%

## OOS Prior
- IS Sharpe is upper bound on OOS Sharpe
- Low-signal, high-noise regime assumed

## Kill Criteria (ANY triggers rejection)
- OOS Sharpe ≤ 0
- Carry < 0.1%
- Funding > carry
- Curve inverted
- Carry explains < 40% of return
- Fill rate < 70%
- Slippage > 50bps

## Model Risk & Drift Controls

### Rule Invalidation Triggers
- If rule relies on roll-down and curve flattens → must deactivate
- If funding rises above carry → must deactivate
- Use tagging for regime classification
- Track drift in attribution over time

## Audit Trail
- Log ALL assumptions, changes, attribution, decisions (YAML format)
- Compute carry per SD
- Apply rule once on validation
- **If not logged, didn't happen — makes it observable**

## Variables & Data Quality

### Frozen Inputs
| Variable | Source | Freeze Point |
|----------|--------|--------------|
| OIS/SOFR curve | Bloomberg | Pre-deployment |
| Credit curve | Internal | Pre-deployment |
| Hazard rate | Model | Pre-deployment |
| Liquidity index | TRACE | Pre-deployment |
| Technicals | Internal | Pre-deployment |

### TRACE/Pricing
- Frozen CUSIP map, BB sector curve
- Forward TR over 21d
- Label type: Simulated forward return
- Roll-down method: BB sector curve -1
- Outlier policy: discard > 3σ
- Pricing source: TRACE; Mapping frozen pre-deployment
- **Any spec change invalidates results**
