# LOGICS.md — GPF Plan Monitor Indicator Logic

Version: `0.1.0`
Project: `gpf-plan-monitor`
Indicator name: `GPF Plan Monitor`
Indicator type: TradingView Pine Script indicator
Primary user: personal GPF plan monitoring
Primary platform constraint: TradingView Free, one chart, one custom indicator, mode selector

Implementation decisions locked for reliable v0.1:

- `LOGICS.md` is the primary behavioral authority when project documents conflict.
- Use Pine Script v6.
- Use only completed weekly/monthly bars for scoring.
- Implement full confidence and missing-data handling in v0.1.
- Dashboard RS uses `Up` or `Down`; `Flat` remains future work.

---

## 1. Purpose

`GPF Plan Monitor` is a TradingView indicator for monitoring market conditions related to selected Thai GPF investment plans.

The indicator does not read or analyse official GPF NAV. It does not calculate actual plan returns. It does not make investment decisions for the user.

Its purpose is to answer this question:

> Based on market proxies visible in TradingView, which GPF plan group currently looks strong, neutral, weak, or worth deeper review on weekly and monthly timeframes?

The indicator is a **market proxy regime monitor**, not a GPF performance analyser.

---

## 2. Non-goals

The indicator must not do these in v0.1:

- Do not analyse GPF NAV.
- Do not scrape GPF websites.
- Do not fetch external APIs.
- Do not use TradingView as a backend data service.
- Do not provide direct buy/sell/switch recommendations.
- Do not backtest GPF switching strategies.
- Do not use intraday trading signals.
- Do not use fundamental data in v0.1.
- Do not add alerts in v0.1.
- Do not create a strategy script.

The indicator should only provide structured evidence for the user’s monthly or periodic review.

---

## 3. Core mental model

The indicator has two layers:

1. **Dashboard mode**
   - A radar view.
   - Shows all four monitored plans together.
   - Used to decide which plan deserves deeper inspection.

2. **Detail modes**
   - One detail mode per plan.
   - Shows the internal checklist and score components.
   - Used to confirm whether the dashboard signal is supported by multiple tests.

The user workflow is:

```text
Open any chart
→ Add GPF Plan Monitor
→ Set Mode = Dashboard
→ Identify strong / weak / mixed plans
→ Change Mode to the relevant Detail mode
→ Read checklist and score components
→ Record monthly note
→ Decide independently outside the indicator
```

---

## 4. Investment plan groups

The indicator monitors four plan groups:

1. Gold
2. Global Equity
3. Thai Equity
4. Shariah

These groups are monitored through TradingView proxies.

They are not official representations of GPF portfolios.

---

## 5. Allocation rule awareness

The indicator must be aware of the business rule discussed in planning:

- Shariah is treated as an exclusive review path.
- Gold, Global Equity, and Thai Equity are treated as custom allocation candidates.
- Gold has a known allocation constraint in the user’s intended model.

However, v0.1 does not calculate allocations. It only monitors proxy conditions.

Do not implement allocation simulation in Pine v0.1.

---

## 6. Default proxy symbols

The indicator must use input symbols so the user can override defaults.

### 6.1 Gold proxies

| Role | Default symbol |
|---|---|
| Primary | `OANDA:XAUUSD` |
| Secondary | `COMEX:GC1!` |
| Secondary | `AMEX:GLD` |
| Benchmark | `AMEX:ACWI` |
| Macro | `TVC:DXY` |
| Macro | `TVC:US10Y` |
| Macro | `FX_IDC:USDTHB` |

### 6.2 Global Equity proxies

| Role | Default symbol |
|---|---|
| Primary | `AMEX:ACWI` |
| Secondary | `AMEX:SPY` |
| Secondary | `NASDAQ:QQQ` |
| Secondary | `AMEX:VEA` |
| Secondary | `AMEX:VWO` |
| Macro | `TVC:US10Y` |
| Macro | `TVC:DXY` |
| Macro | `FX_IDC:USDTHB` |

### 6.3 Thai Equity proxies

| Role | Default symbol |
|---|---|
| Primary | `SET:SET50` |
| Secondary | `SET:SET` |
| Secondary | `SET:DELTA` |
| Secondary | `SET:PTT` |
| Secondary | `SET:KBANK` |
| Secondary | `SET:ADVANC` |
| Secondary | `SET:KTB` |
| Secondary | `SET:SCB` |
| Secondary | `SET:BBL` |
| Secondary | `SET:GULF` |
| Secondary | `SET:PTTEP` |
| Secondary | `SET:CPN` |
| Benchmark | `AMEX:ACWI` |
| Alternative benchmark | `AMEX:VWO` |
| Macro | `FX_IDC:USDTHB` |

### 6.4 Shariah proxies

| Role | Default symbol |
|---|---|
| Primary | `AMEX:SPUS` |
| Secondary | `NASDAQ:HLAL` |
| Benchmark | `AMEX:ACWI` |
| Benchmark | `AMEX:SPY` |
| Macro | `TVC:US10Y` |
| Macro | `OANDA:XAUUSD` |
| Macro | `FX_IDC:USDTHB` |

---

## 7. Timeframe logic

The indicator must avoid short-term trading noise.

Use these timeframes:

| Timeframe | Purpose |
|---|---|
| Weekly | Main trend signal |
| Monthly | Regime confirmation |
| Daily | Not used for core scoring |
| Intraday | Not used |

### 7.1 Weekly trend

A symbol passes weekly trend if:

```text
weekly_close > EMA(weekly_close, 40)
```

Weekly EMA 40 is the main intermediate trend filter.

### 7.2 Monthly trend

A symbol passes monthly trend if:

```text
monthly_close > EMA(monthly_close, 10)
```

Monthly EMA 10 is the higher timeframe regime filter.

### 7.3 Interpretation

| Weekly | Monthly | Meaning |
|---|---|---|
| Pass | Pass | Strong alignment |
| Pass | Fail | Short-term recovery only |
| Fail | Pass | Pullback within larger regime |
| Fail | Fail | Weak regime |

---

## 8. Dashboard mode logic

Dashboard mode is a compact screening view.

It must not show every internal calculation.

It must display these columns:

| Column | Meaning |
|---|---|
| Plan | Gold / Global / Thai / Shariah |
| Weekly | Strong / Weak |
| Monthly | Strong / Weak |
| RS | Up / Down |
| Score | 0–100 |

### 8.1 Dashboard primary symbol per plan

| Plan | Primary symbol |
|---|---|
| Gold | Gold primary proxy |
| Global | Global primary proxy |
| Thai | Thai primary proxy |
| Shariah | Shariah primary proxy |

### 8.2 Dashboard weekly status

```text
Weekly = Strong if primary weekly close > weekly EMA 40
Weekly = Weak otherwise
```

### 8.3 Dashboard monthly status

```text
Monthly = Strong if primary monthly close > monthly EMA 10
Monthly = Weak otherwise
```

### 8.4 Dashboard relative strength

Use a weekly ratio:

```text
RS ratio = primary proxy / benchmark
RS average = SMA(RS ratio, 12)
```

Then:

```text
RS = Up if RS ratio > RS average
RS = Down if RS ratio < RS average
```

`Flat` is reserved for a future version. v0.1 displays only `Up` or `Down`.

### 8.5 Dashboard score

Use a simple score:

```text
Score = 33 if Weekly Strong
      + 33 if Monthly Strong
      + 34 if RS Up
```

Range:

```text
0, 33, 34, 66, 67, 100
```

### 8.6 Dashboard status interpretation

| Score | Status |
|---|---|
| 67–100 | Strong |
| 34–66 | Neutral |
| 0–33 | Weak |

Dashboard status is only a screening output.

It must never be treated as a final confirmation.

---

## 9. Detail mode logic

Each Detail mode must show the checklist that supports or rejects the Dashboard signal.

Detail modes must display at least these rows:

1. Trend
2. Relative Strength
3. Macro
4. Risk
5. Total
6. Final Status
7. Confidence
8. Note / Interpretation

The exact table layout can be adjusted for Pine limits, but the underlying logic must remain clear.

---

## 10. Detail score structure

Each plan receives a `Detail Score` from 0 to 100.

```text
Detail Score = Trend Score
             + Relative Strength Score
             + Macro Score
             + Risk Score
```

Weights:

| Component | Max score |
|---|---:|
| Trend | 40 |
| Relative Strength | 30 |
| Macro | 20 |
| Risk | 10 |
| Total | 100 |

---

## 11. Trend score logic

Trend Score measures whether the plan’s relevant proxies are in weekly and monthly uptrends.

### 11.1 Per-symbol trend test

For each proxy symbol:

```text
weekly_pass = weekly_close > weekly_EMA_40
monthly_pass = monthly_close > monthly_EMA_10
symbol_trend_score = (weekly_pass + monthly_pass) / 2
```

Where:

```text
true = 1
false = 0
```

### 11.2 Plan trend score

For a plan with `n` proxies:

```text
average_trend = average(symbol_trend_score of all proxies)
Trend Score = average_trend × 40
```

### 11.3 Important rule

Do not let a single secondary proxy dominate the plan score.

Use average scoring unless a future version has official holding weights.

---

## 12. Relative Strength score logic

Relative Strength Score checks whether a plan is outperforming its benchmark.

### 12.1 RS ratio

For each relevant proxy:

```text
RS ratio = proxy weekly close / benchmark weekly close
RS average = SMA(RS ratio, 12)
```

Then:

```text
RS pass = RS ratio > RS average
```

### 12.2 Primary vs secondary weighting

Use this structure:

```text
Primary RS = 20 points
Secondary RS average = 10 points
```

Formula:

```text
Relative Strength Score = primary_rs_pass × 20
                        + average(secondary_rs_passes) × 10
```

Range:

```text
0–30
```

### 12.3 Benchmarks by plan

| Plan | Benchmark |
|---|---|
| Gold | ACWI |
| Global | ACWI or SPY family |
| Thai | ACWI by default |
| Shariah | ACWI by default |

For Global Equity, using ACWI as both primary and benchmark will make the RS test meaningless if implemented naively.

Recommended v0.1 behaviour:

- Global Dashboard may use Global primary trend more than RS.
- Global Detail should compare SPY, QQQ, VEA, VWO against ACWI.
- Do not score `ACWI / ACWI` as true outperformance.
- Reliable v0.1 weights Global Dashboard weekly/monthly trend equally and weights the Global Detail secondary-proxy RS basket across the full RS component.

If implementation cannot avoid this cleanly, assign neutral score for Global primary RS and rely on secondary proxies.

---

## 13. Macro score logic

Macro Score measures whether the environment supports or pressures the plan.

Max score:

```text
20
```

### 13.1 US10Y condition

Use weekly US10Y:

```text
US10Y condition passes if US10Y < SMA(US10Y, 26)
```

Score:

```text
+10 if pass
+0 if fail
```

Reasoning:

- Lower or declining yields tend to relieve pressure on equities and gold.
- Higher or rising yields often pressure long-duration assets.

### 13.2 DXY condition

Use weekly DXY:

```text
DXY condition passes if DXY < SMA(DXY, 26)
```

Score:

```text
+5 if pass
+0 if fail
```

Reasoning:

- A weaker dollar often supports gold and global risk assets.

### 13.3 USDTHB condition

Use weekly USDTHB:

```text
USDTHB condition = USDTHB > SMA(USDTHB, 26)
```

Interpretation differs by plan:

#### For foreign equity-related plans

Foreign equity-related plans include:

- Global Equity
- Shariah

Rising USDTHB can support THB-denominated value of foreign assets.

Score:

```text
+5 if USDTHB > SMA(USDTHB, 26)
```

#### For Gold

Gold currency effect can be complex because actual GPF hedging is unknown.

For v0.1, use conservative logic:

```text
+5 if USDTHB < SMA(USDTHB, 26)
```

This avoids over-crediting currency effects.

#### For Thai Equity

USDTHB interpretation is not direct.

Possible v0.1 logic:

- Neutral default.
- Or use USDTHB weakness as possible foreign outflow pressure.

Recommended implementation:

```text
Thai Equity USDTHB = neutral unless explicitly implemented
```

Do not overstate USDTHB effect on Thai equities.

---

## 14. Risk score logic

Risk Score measures drawdown pressure.

Max score:

```text
10
```

Use the primary proxy of the plan.

### 14.1 Drawdown calculation

```text
highest_52w_close = highest weekly close in last 52 weeks
current_weekly_close = latest weekly close
weekly_drawdown = (current_weekly_close - highest_52w_close) / highest_52w_close
```

Drawdown is usually negative.

### 14.2 Risk score mapping

| Drawdown | Risk score |
|---|---:|
| >= 0 | 10 |
| -25% to 0 | 5 |
| < -25% | 0 |

This is intentionally simple for v0.1.

Future versions may use smoother scoring.

---

## 15. Final Status logic

After calculating `Detail Score`:

```text
Final Status =
  Confirmed      if Detail Score >= 75
  Mixed          if 50 <= Detail Score < 75
  Not Confirmed  if Detail Score < 50
```

### 15.1 Meaning of Confirmed

`Confirmed` means:

- Multiple proxy signals align.
- Trend and relative strength are supportive.
- Macro and risk conditions are not severely negative.

It does not mean:

- Switch plan now.
- Buy now.
- Guaranteed outperformance.

### 15.2 Meaning of Mixed

`Mixed` means:

- The evidence is not consistent.
- Some signals are supportive.
- Some signals are weak or conflicting.

User action:

- Watchlist candidate.
- Wait for monthly confirmation.
- Review manually if considering action.

### 15.3 Meaning of Not Confirmed

`Not Confirmed` means:

- Conditions do not support the plan.
- The dashboard signal may be weak or misleading.
- User should not treat the plan as strong based on current proxy logic.

---

## 16. Confidence logic

Confidence measures the reliability of the conclusion, not the attractiveness of the plan.

A plan can have:

```text
Final Status = Confirmed
Confidence = Low
```

Meaning:

```text
Signals look good, but evidence quality is weak.
```

A plan can also have:

```text
Final Status = Mixed
Confidence = High
```

Meaning:

```text
Data is reliable, but conditions are genuinely mixed.
```

### 16.1 Confidence components

| Component | Weight |
|---|---:|
| Proxy Fit | 40 |
| Signal Agreement | 25 |
| Timeframe Agreement | 20 |
| Data Completeness | 15 |

### 16.2 Proxy Fit

Default proxy fit values:

| Plan | Proxy Fit |
|---|---:|
| Gold | 35–40 |
| Global Equity | 35–40 |
| Thai Equity | 30–35 |
| Shariah | 20–25 |

Rationale:

- Shariah has lower fit because SPUS/HLAL may not match actual GPF Shariah holdings.
- Thai Equity has moderate fit because SET50 and top stocks may approximate but not exactly match GPF holdings.
- Gold and Global proxies are broader and cleaner.

### 16.3 Signal Agreement

Check whether primary and secondary proxies agree.

Example:

```text
primary weekly trend pass
secondary weekly trend pass
relative strength pass
```

If most agree:

```text
Signal Agreement = high
```

If they conflict:

```text
Signal Agreement = low
```

### 16.4 Timeframe Agreement

Check whether weekly and monthly signals point in the same direction.

| Weekly | Monthly | Timeframe Agreement |
|---|---|---|
| Pass | Pass | High |
| Fail | Fail | High, but bearish |
| Pass | Fail | Low/Medium |
| Fail | Pass | Low/Medium |

### 16.5 Data Completeness

If any important symbol returns `na`, reduce confidence.

Examples:

- `HLAL` unavailable.
- `SET` symbol missing.
- `USDTHB` unavailable.
- Macro data missing.

### 16.6 Confidence mapping

After summing confidence score:

```text
High   = 75–100
Medium = 50–74
Low    = 0–49
```

Reliable v0.1 implements the full confidence model.

---

## 17. Plan-specific logic

## 17.1 Gold Detail

Gold is not a stock plan. It is evaluated by price and macro conditions.

### Inputs

- XAUUSD
- GC1!
- GLD
- ACWI benchmark
- DXY
- US10Y
- USDTHB

### Checklist

Gold Detail must check:

1. XAUUSD weekly trend
2. XAUUSD monthly trend
3. GLD weekly/monthly confirmation
4. XAUUSD relative strength vs ACWI
5. DXY condition
6. US10Y condition
7. USDTHB condition
8. Drawdown risk
9. Final Status
10. Confidence

### Interpretation

Gold is strongest when:

- XAUUSD passes weekly trend.
- XAUUSD passes monthly trend.
- Gold outperforms ACWI.
- DXY is weak.
- US10Y is weak.
- Drawdown is controlled.

Gold may be mixed when:

- Weekly trend is strong but monthly trend is not confirmed.
- Gold is strong but DXY/US10Y are pressuring it.

Gold is weak when:

- XAUUSD fails weekly and monthly trend.
- Gold underperforms ACWI.
- Macro factors are unfavourable.

---

## 17.2 Global Equity Detail

Global Equity monitors broad global equity conditions.

### Inputs

- ACWI
- SPY
- QQQ
- VEA
- VWO
- US10Y
- DXY
- USDTHB

### Checklist

Global Equity Detail must check:

1. ACWI weekly trend
2. ACWI monthly trend
3. SPY weekly/monthly trend
4. QQQ weekly/monthly trend
5. VEA weekly/monthly trend
6. VWO weekly/monthly trend
7. SPY/QQQ/VEA/VWO relative strength vs ACWI
8. US10Y condition
9. DXY condition
10. USDTHB condition
11. Drawdown risk
12. Final Status
13. Confidence

### Interpretation

Global Equity is strongest when:

- ACWI passes weekly and monthly trend.
- SPY and QQQ support the move.
- VEA and VWO are not failing badly.
- US10Y is not rising aggressively.
- USDTHB supports foreign asset translation.

Global Equity is mixed when:

- ACWI passes weekly but not monthly.
- US tech is strong but international markets are weak.
- VWO diverges sharply from developed markets.

Global Equity is weak when:

- ACWI fails weekly and monthly trend.
- SPY/QQQ/VEA/VWO mostly fail.
- US10Y/DXY pressures are strong.

---

## 17.3 Thai Equity Detail

Thai Equity monitors SET50, SET, and major Thai equity proxies.

### Inputs

- SET50
- SET
- DELTA
- PTT
- KBANK
- ADVANC
- KTB
- SCB
- BBL
- GULF
- PTTEP
- CPN
- ACWI benchmark
- VWO optional benchmark
- USDTHB

### Checklist

Thai Equity Detail must check:

1. SET50 weekly trend
2. SET50 monthly trend
3. SET weekly/monthly trend
4. Number of major Thai proxies passing weekly trend
5. Number of major Thai proxies passing monthly trend
6. SET50 relative strength vs ACWI
7. SET50 relative strength vs VWO, optional
8. Concentration warning if only DELTA passes
9. USDTHB condition, optional/neutral
10. Drawdown risk
11. Final Status
12. Confidence

### Interpretation

Thai Equity is strongest when:

- SET50 passes weekly and monthly trend.
- SET confirms SET50.
- Several major Thai stocks pass together.
- SET50 outperforms ACWI or VWO.
- Strength is not only from one stock.

Thai Equity is mixed when:

- SET50 is strong but breadth is weak.
- DELTA dominates the signal.
- SET50 outperforms locally but not globally.

Thai Equity is weak when:

- SET50 fails weekly and monthly trend.
- Most large Thai stocks fail.
- SET50 underperforms ACWI/VWO.

---

## 17.4 Shariah Detail

Shariah is evaluated with proxy caution.

The indicator must always assume lower confidence for Shariah because SPUS and HLAL are only external Shariah-compliant ETF proxies, not actual GPF Shariah holdings.

### Inputs

- SPUS
- HLAL
- ACWI
- SPY
- US10Y
- XAUUSD
- USDTHB

### Checklist

Shariah Detail must check:

1. SPUS weekly trend
2. SPUS monthly trend
3. HLAL weekly trend
4. HLAL monthly trend
5. SPUS relative strength vs ACWI
6. HLAL relative strength vs ACWI
7. SPUS relative strength vs SPY
8. US10Y condition
9. XAUUSD risk-off support
10. USDTHB condition
11. Drawdown risk
12. Proxy confidence penalty
13. Final Status
14. Confidence

### Interpretation

Shariah is strongest when:

- SPUS passes weekly and monthly trend.
- HLAL does not conflict badly.
- SPUS/HLAL outperform ACWI.
- SPUS is not underperforming SPY badly.
- US10Y pressure is low.
- USDTHB supports foreign asset translation.

Shariah is mixed when:

- SPUS is strong but HLAL is weak.
- Weekly trend passes but monthly trend does not.
- SPUS outperforms ACWI but not SPY.
- Macro pressure is high.

Shariah is weak when:

- SPUS and HLAL fail weekly and monthly trend.
- SPUS/HLAL underperform ACWI.
- US10Y pressure is strong.

### Shariah-specific confidence rule

Even when Shariah Detail Score is high, confidence should usually be capped at Medium unless multiple proxies confirm and data completeness is high.

This avoids false precision.

---

## 18. Display logic

The indicator must be usable on TradingView Free.

Therefore:

- Use one chart.
- Use one indicator.
- Use mode selector.
- Use a table display.
- Do not require multiple layouts.

### 18.1 Dashboard table

Suggested columns:

```text
Plan | Weekly | Monthly | RS | Score
```

Suggested rows:

```text
Gold
Global
Thai
Shariah
```

### 18.2 Detail table

Suggested columns:

```text
Check | Result | Score / Note
```

Suggested rows:

```text
Trend
Relative Strength
Macro
Risk
Total
Final Status
Confidence
Note
```

### 18.3 Colour meaning

| Meaning | Colour |
|---|---|
| Pass / Strong / Confirmed | Green |
| Mixed / Neutral / Warning | Orange |
| Fail / Weak / Not Confirmed | Red |
| Score / neutral numeric value | Blue or default |

Do not rely on colour only. Text must also communicate the result.

---

## 19. TradingView Free constraints

The implementation must respect these constraints:

- Single indicator script.
- One mode visible at a time.
- No requirement for multiple layouts.
- No heavy multi-chart UI.
- Avoid too many `request.security()` calls.
- Start with v0.1 logic before adding holdings/fundamentals.

If the script becomes too heavy:

1. Remove low-priority secondary proxies.
2. Reduce Thai holdings count.
3. Reduce macro checks.
4. Do not add fundamentals yet.

---

## 20. Implementation warnings for CODEX

CODEX must not silently alter these rules.

### 20.1 Do not change the problem

Do not transform this into:

- Trading strategy
- Buy/sell signal
- Rebalancing bot
- NAV analyser
- Webapp backend
- Fundamental screener

### 20.2 Do not over-engineer

Do not add:

- Classes
- Complex custom data models
- Dynamic arrays beyond what Pine requires
- Alerts
- Labels all over the chart
- Excessive colours
- Extra dashboards

### 20.3 Do not claim success without compile

The script is not verified until:

- It compiles in TradingView Pine Editor.
- It can be added to a chart.
- Dashboard mode shows a table.
- Each detail mode shows expected rows.
- No obvious `na` or symbol errors break display.

---

## 21. Test checklist

After any implementation, test manually in TradingView.

### 21.1 Compile test

Expected:

```text
No Pine syntax errors.
```

### 21.2 Dashboard test

Set mode:

```text
Dashboard
```

Expected:

```text
A table appears with four rows:
Gold, Global, Thai, Shariah.
```

### 21.3 Detail mode tests

Test each mode:

```text
GoldDetail
GlobalEquityDetail
ThaiEquityDetail
ShariahDetail
```

Expected:

```text
Each mode displays Trend, Relative Strength, Macro, Risk, Total, Final Status, Confidence.
```

### 21.4 Symbol override test

Change at least one input symbol.

Expected:

```text
The table still updates without script failure.
```

### 21.5 Missing data test

If a symbol returns missing data:

Expected:

```text
The indicator should show weaker confidence or neutral/fail status.
It should not misrepresent missing data as strong signal.
```

---

## 22. Current known weakness of v0.1

These are accepted weaknesses:

- Shariah proxies are imperfect.
- Thai holdings may be too many for Pine if all are requested.
- Fundamental data is not included.
- Global primary RS against ACWI must avoid self-comparison issue.
- USDTHB treatment is simplified.
- Macro scoring is coarse.
- Risk scoring uses a hard drawdown threshold.

These weaknesses must be documented, not hidden.

---

## 23. Future version ideas

Only consider these after v0.1 compiles and displays correctly:

### v0.2

- Fix confidence scoring to match full model.
- Add missing-data handling.
- Improve Thai breadth logic.
- Add Global secondary outperformance logic.

### v0.3

- Add optional fundamental data using TradingView-supported functions.
- Add separate Pine Screener workflow.
- Add alerts only after user confirms exact trigger logic.

### v0.4

- Create a small local companion note/report system outside TradingView.
- Keep TradingView as visual market monitor.

---

## 24. One-sentence logic summary

`GPF Plan Monitor` scores each monitored GPF plan proxy by combining weekly/monthly trend, relative strength, macro condition, and drawdown risk, then separates the result into `Final Status` and `Confidence` so the user can see both market condition and evidence quality before doing any manual decision-making.
