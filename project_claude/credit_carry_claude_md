# CLAUDE.md

## Project Overview
Credit carry strategy research with pre-registered hypotheses and strict falsification criteria.
Target: Journal of Finance and Data Science submission.

## Code Standards
- Python with type hints, NumPy docstrings
- Vectorised pandas/numpy; no loops over rows
- All dates UTC; identifiers: ISIN > CUSIP > FIGI

## Research Discipline (NON-NEGOTIABLE)

### Pre-Registration Rules
- All specifications frozen BEFORE seeing results
- Regression spec, SEs, winsorization, controls: declared ex-ante
- Universe, horizon Δ, weighting: written and locked before looking at data
- Any spec change invalidates results

### Holdout Protocol
- Single-touch holdout run only
- Second access invalidates experiment
- OOS decay assumption written first

### Kill Criteria (strategy rejected if ANY true)
- Annualized carry < 0.1%
- Carry explains < 40% of return (either way)
- Curve inversion (no roll-down)
- Funding > carry (track cross-sectionally AND sector-wise)
- Fill rate < 70% or slippage > 50bps

### Audit Trail
- Log ALL assumptions, changes, attribution, decisions (YAML)
- "If not logged, didn't happen"
- Compute carry per SD; apply rule once on validation

## Key Paths
| Purpose | Location |
|---------|----------|
| Reference data | `data/reference/` |
| Pricing/TRACE | `data/pricing/`, `data/trace/` |
| Pre-registrations | `research/preregistration/` |
| Results | `reports/` |

## Skills Available
| Skill | Trigger |
|-------|---------|
| `carry-execution` | Running carry backtest with pre-registered spec |
| `preregistration-validator` | Checking compliance before/after results |

## Anti-Patterns
- NO post-hoc specification changes
- NO peeking at holdout before final run
- NO unreported robustness checks
- NO floating-point equality without tolerance
