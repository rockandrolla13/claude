claude
```

Then prompt:
```
Create a skill at .claude/skills/data-availability-checker/SKILL.md with the following specification:

---
name: data-availability-checker
description: >
  Check data availability for backtesting credit strategies by rating, duration, coupon filters.
  Use when planning a backtest, validating data sufficiency, or determining maximum available history
  for a given universe specification.
---

# Data Availability Checker

## Purpose
Given user-specified filters (rating, duration bucket, coupon range), determine the longest continuous 
history available across all required data sources: reference data, pricing data, and TRACE data.

## User Inputs

Prompt user for:

| Parameter | Example | Notes |
|-----------|---------|-------|
| Rating | IG, HY, BBB, BB+, A- | Single or range (e.g., BBB- to BBB+) |
| Duration bucket | [3,5], [5,7] | Years, using standard buckets |
| Coupon range | [0,5], [5,8] | Percentage, optional |
| Minimum bonds | 50 | Minimum universe size per period |
| Desired period | 2Y, 5Y | Target backtest length |

## Data Sources to Check

For each source, identify:
- Available date range (min, max)
- Coverage within user's filters
- Gaps or dropoffs

### 1. Reference Data
Location: `data/reference/` or `data/bond_master/`
Required fields: ISIN/CUSIP, rating, maturity_date, coupon, issue_date, sector

### 2. Pricing Data  
Location: `data/pricing/` or `data/marks/`
Required fields: identifier, date, price (or yield/spread/OAS)

### 3. TRACE Data
Location: `data/trace/`
Required fields: identifier, trade_date, price, volume

## Analysis Workflow

### Step 1: Load and Filter Reference Data
1. Apply rating filter (map letter ratings to numeric for range queries)
2. Compute duration from maturity_date at each reference_date (or use effective_duration if available)
3. Apply duration bucket filter
4. Apply coupon filter if specified
5. Output: filtered bond universe with identifiers

### Step 2: Check Pricing Data Coverage
For filtered universe:
1. Find date range where pricing exists
2. Count bonds with pricing per month
3. Identify months below minimum bond threshold
4. Calculate: longest continuous period meeting minimum

### Step 3: Check TRACE Data Coverage
For filtered universe:
1. Find date range where TRACE data exists
2. Count bonds with trades per month
3. Identify coverage gaps
4. Calculate: longest continuous period with adequate liquidity

### Step 4: Compute Intersection
Find the longest continuous period where ALL THREE sources have adequate coverage:
- Reference data available
- Pricing data for >= minimum bonds
- TRACE data for >= minimum bonds (or specified liquidity threshold)

## Output

Generate `reports/data_availability_{timestamp}.md`:
```markdown
# Data Availability Report

## Query Parameters
- Rating: [user input]
- Duration: [user input]
- Coupon: [user input]
- Minimum bonds: [user input]
- Desired period: [user input]

## Summary
| Metric | Value |
|--------|-------|
| Maximum available history | X years Y months |
| Recommended backtest period | YYYY-MM-DD to YYYY-MM-DD |
| Universe size (avg) | N bonds |
| Coverage quality | Good/Acceptable/Poor |

## Feasibility
[Can desired period be supported? Yes/No + explanation]

## Per-Source Analysis

### Reference Data
- Date range: YYYY-MM-DD to YYYY-MM-DD
- Bonds matching filters: N

### Pricing Data
- Date range: YYYY-MM-DD to YYYY-MM-DD  
- Coverage: N bonds (avg), M bonds (min)
- Gap months: [list if any]

### TRACE Data
- Date range: YYYY-MM-DD to YYYY-MM-DD
- Coverage: N bonds with trades (avg)
- Gap months: [list if any]

## Intersection Analysis
- Longest continuous period: YYYY-MM-DD to YYYY-MM-DD (X years Y months)
- Limiting factor: [which data source constrains]

## Recommendations
1. [Actionable recommendations]
2. [E.g., "Relax rating filter to BBB- for +6 months history"]
3. [E.g., "TRACE coverage drops before 2018, consider pricing-only backtest"]
```

Also generate `reports/data_availability_{timestamp}.csv` with monthly breakdown:

| month | ref_bonds | priced_bonds | traced_bonds | all_available |
|-------|-----------|--------------|--------------|---------------|
| 2020-01 | 150 | 145 | 120 | Yes |
| 2020-02 | 152 | 148 | 118 | Yes |

## Interactive Mode

If coverage is insufficient for desired period, suggest alternatives:
1. "Relaxing duration to [X,Y] adds N months of history"
2. "Removing coupon filter adds N bonds and extends coverage to..."
3. "IG-only (excluding BBB-) has continuous coverage from..."

Offer to re-run with modified parameters.

## Implementation Notes
- Use column-mappings from bond-data-profiler skill for column name variations
- Rating mapping: AAA=1, AA+=2, ..., D=22 (for range queries)
- Duration: compute if not present using (maturity_date - as_of_date) / 365.25
- Handle missing data gracefully; report which sources are unavailable
```

---

## Step 3: Create Rating Mappings Reference
```
Create .claude/skills/data-availability-checker/rating-mappings.md with:

# Rating Mappings

## Letter to Numeric
| Rating | Numeric | Category |
|--------|---------|----------|
| AAA | 1 | IG |
| AA+ | 2 | IG |
| AA | 3 | IG |
| AA- | 4 | IG |
| A+ | 5 | IG |
| A | 6 | IG |
| A- | 7 | IG |
| BBB+ | 8 | IG |
| BBB | 9 | IG |
| BBB- | 10 | IG |
| BB+ | 11 | HY |
| BB | 12 | HY |
| BB- | 13 | HY |
| B+ | 14 | HY |
| B | 15 | HY |
| B- | 16 | HY |
| CCC+ | 17 | HY |
| CCC | 18 | HY |
| CCC- | 19 | HY |
| CC | 20 | HY |
| C | 21 | HY |
| D | 22 | HY |

## Shorthand Mappings
| Input | Expands To |
|-------|------------|
| IG | AAA to BBB- |
| HY | BB+ to D |
| A-rated | A+ to A- |
| BBB | BBB+ to BBB- |

## Agency Mappings
| S&P | Moody's | Fitch |
|-----|---------|-------|
| AAA | Aaa | AAA |
| AA+ | Aa1 | AA+ |
| AA | Aa2 | AA |
...
```

---

## Step 4: Create Data Source Config
```
Create .claude/skills/data-availability-checker/data-sources.md with:

# Data Source Configuration

## Reference Data
Paths (check in order):
- data/reference/
- data/bond_master/
- data/universe/

Required columns:
- Identifier: isin, cusip, figi
- rating, composite_rating, sp_rating, moodys_rating
- maturity_date, maturity
- coupon, coupon_rate
- issue_date
- sector, industry

## Pricing Data
Paths:
- data/pricing/
- data/marks/
- data/valuations/

Required columns:
- Identifier
- date, as_of_date, pricing_date
- price, mid_price, clean_price
- yield, ytm (optional)
- spread, oas, z_spread (optional)

## TRACE Data
Paths:
- data/trace/
- data/trace_enhanced/

Required columns:
- Identifier
- trade_date, execution_date
- price, trade_price
- volume, quantity (optional)
- side, buy_sell (optional)
```

---

## Step 5: Create Slash Command for Easy Invocation
```
Create .claude/commands/check-data.md:

Check data availability for backtest with filters: $ARGUMENTS

Parse arguments for:
- rating: e.g., "IG", "BBB", "BB+ to B-"
- duration: e.g., "3-5Y", "[5,7]"  
- coupon: e.g., "0-5%", "fixed"
- period: e.g., "2Y", "5Y"
- min_bonds: e.g., "50"

Use the data-availability-checker skill.

Example usage:
/project:check-data rating=IG duration=3-5Y period=2Y min_bonds=100
/project:check-data rating=BBB duration=5-7Y coupon=4-6% period=3Y
```

---

## Step 6: Update CLAUDE.md
```
Add to CLAUDE.md:

## Data Availability
- Use data-availability-checker skill to validate backtest feasibility
- Specify: rating, duration bucket, coupon, desired period
- Outputs: maximum continuous history, coverage gaps, recommendations
- Slash command: /project:check-data rating=X duration=Y period=Z
```

---

## Step 7: Test the Skill
```
/clear
```

Then:
```
I want to backtest a BBB-rated credit strategy in the 5-7 year duration bucket.
Target period is 3 years. What's the maximum history available?
```

Or with slash command:
```
/project:check-data rating=BBB duration=5-7Y period=3Y min_bonds=50
