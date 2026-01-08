# CLAUDE.md

## Project Overview
Credit trading research platform: systematic strategies, backtesting, data analysis.

## Code Standards

### Python
- Type hints required on all functions
- Docstrings: NumPy format
- Formatting: black (88 chars), isort, ruff
- Prefer vectorised pandas/numpy over loops

### Data Conventions
- All dates UTC, format YYYY-MM-DD
- Identifiers priority: ISIN > CUSIP > FIGI > internal_id
- Returns are gross unless explicitly `_net` suffix
- Prices are clean (ex-accrued) unless `_dirty` suffix

## Architecture
```
data/ → src/data/ → src/strategies/ → src/backtest/ → reports/
```

- Config-driven design; no hardcoded parameters
- Idempotent functions; deterministic outputs for same inputs

## Key Paths
| Purpose | Location |
|---------|----------|
| Reference data | `data/reference/` |
| Pricing data | `data/pricing/` |
| TRACE data | `data/trace/` |
| Strategy code | `src/strategies/` |
| Backtest harness | `src/backtest/` |
| Generated reports | `reports/` |

## Skills Available
| Skill | Use When |
|-------|----------|
| `bond-data-profiler` | Analysing new datasets, checking quality |
| `data-availability-checker` | Planning backtests, validating data sufficiency |
| `carry-backtest` | Running carry strategy backtests |
| `research-workflow` | Pre-registering hypotheses, managing holdouts |

## Slash Commands
| Command | Purpose |
|---------|---------|
| `/project:profile-data <path>` | Profile dataset at path |
| `/project:check-data <filters>` | Check data availability |
| `/project:run-backtest <strategy>` | Execute backtest |

## Anti-Patterns
- No look-ahead bias: never use future data in signal construction
- No silent failures: always raise or log errors
- No pandas SettingWithCopyWarning: use `.loc[]` explicitly
- No floating-point equality without tolerance
- No Sharpe calculation on < 252 trading days
- No hardcoded transaction costs: use config

## Testing
- pytest for all core logic
- hypothesis for property-based testing on numerical code
- Integration tests for data pipelines

## Git Workflow
- Conventional commits: `feat:`, `fix:`, `refactor:`, `docs:`
- Squash merge to main
- Pre-commit hooks: black, isort, mypy, ruff

## Documentation References
For detailed methodology, see:
- `docs/architecture.md` — system design
- `docs/data-sources.md` — data feeds, schemas, update schedules
- `docs/trading-logic.md` — signal construction, execution rules
