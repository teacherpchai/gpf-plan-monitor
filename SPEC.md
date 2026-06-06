# GPF Plan Monitor – Specification

This document describes the design, logic and constraints of the **GPF Plan Monitor** indicator. `LOGICS.md` is the primary behavioral authority when documents conflict. Explicit user decisions override older conflicting documentation. Any implementation change must be recorded in `CHANGELOG.md`.

## Purpose

The indicator helps individual investors track the market conditions of four investment plans offered by Thailand’s Government Pension Fund (GPF) using **proxy price series** and simple technical measures.  It does **not** analyse or display official NAV data.  Its goal is to provide situational awareness so that members can decide when to explore switching plans (allowed up to 12 times per year) based on the behaviour of underlying assets.  It is not a recommendation engine and should never give direct buy/sell advice.

## Guiding Principles

The design follows six **Buddhist Method** principles aimed at improving LLM behaviour: verifying facts instead of pattern‑matching (Kalāma), root‑cause analysis (Yoniso Manasikāra), acting with updated state across long tasks (Sati‑Sampajañña), rewriting rather than patching wrong solutions (Anatta), removing bugs instead of hiding them (Pahāna), and keeping equanimity under pressure (Upekkhā)【588121122412891†L265-L272】.  It also incorporates four **Karpathy guidelines**—Think Before Coding, Simplicity First, Surgical Changes, and Goal‑Driven Execution—which emphasise explicit assumptions, minimal code, touching only what is required, and defining success criteria【451674326895714†L292-L352】.

Implementations must therefore:

* State assumptions explicitly and list unknowns when computing scores.
* Use simple formulas; avoid over‑engineering or adding features not requested.
* Touch only the relevant parts of the script when fixing bugs or adding features.
* Define clear success criteria for each change; verify with TradingView compilation and visual inspection.
* Never claim the indicator is working without evidence (e.g., successful compile and expected behaviour on sample charts).

## Modes

The indicator contains multiple modes selected via an `input.string()` parameter:

| Mode                | Description |
|---------------------|-------------|
| **Dashboard**       | Displays a compact table summarising each plan’s weekly/monthly trend, relative strength against a benchmark and macro context.  Used to decide which plan to inspect further. |
| **GoldDetail**      | Shows checks specific to the Gold plan (XAUUSD and GLD) plus macro factors (DXY, US10Y, USDTHB). |
| **GlobalEquityDetail** | Shows checks for the Global Equity plan (ACWI, SPY, QQQ, VEA, VWO). |
| **ThaiEquityDetail**   | Shows checks for the Thai Equity plan (SET50, SET, major constituents) and comparisons against global indices. |
| **ShariahDetail**      | Shows checks for the Shariah plan (SPUS, HLAL proxies) and macro factors; includes a confidence estimate due to proxy mismatch risk. |

Each detail mode displays a checklist of sub‑tests (price trend, relative strength, macro pressure, risk/drawdown) and derives a **Final Status** (`Confirmed`, `Mixed` or `Not Confirmed`) and a **Confidence** rating (High/Medium/Low).

### Weekly and Monthly Trend Tests

For each proxy symbol, the indicator uses the most recently completed weekly and monthly bars. A proxy passes the **weekly trend** test when its confirmed weekly close is above EMA‑40 and passes the **monthly trend** test when its confirmed monthly close is above EMA‑10. Developing higher-timeframe bars must not affect scores.

### Relative Strength Tests

Relative strength (RS) compares each proxy to a benchmark.  A proxy passes the RS test if the ratio `(proxy / benchmark)` is above its 12‑period weekly simple moving average or slope.  The Global Equity benchmark is ACWI, the Thai Equity benchmark is ACWI or VWO, and the Shariah benchmark is ACWI/SPY.  RS scores contribute 30 % of the plan’s detail score.

### Macro Pressure Tests

Macro factors evaluate whether external conditions support or hinder the plan:

* **US10Y pressure:** yields rising sharply indicate higher risk‑free returns, which typically hurt equities and gold.  The macro test passes if the 10‑year yield is below its 26‑week moving average.
* **DXY (US Dollar Index):** a strong dollar tends to weigh on gold and emerging markets; the test passes if DXY is below its 26‑week SMA.
* **USDTHB:** rising USDTHB supports foreign equity proxy translation, while Gold uses conservative inverse treatment. Thai Equity receives a neutral macro score because the relationship is not direct.

### Risk / Drawdown Tests

This component (10 %) penalises proxies experiencing deep drawdowns.  A proxy fails the risk test if the percentage decline from its highest close in the last 52 weeks exceeds 25 %.  A smaller drawdown yields a higher risk score.  For Shariah, where proxies may not perfectly match the GPF’s holdings, the risk test also downgrades if SPUS and HLAL disagree strongly.

### Final Status

The detail score (0–100) is the weighted sum of the four components above.  Interpretation:

* **Confirmed (≥ 75):** Most signals support the plan; the proxy series are in up‑trends and outperforming benchmarks with favourable macro context.
* **Mixed (50–74):** Signals are inconsistent; some proxies or macro factors align while others do not.  Caution is warranted.
* **Not Confirmed (< 50):** Signals predominantly negative; the plan is likely in a weak regime.

### Confidence

Confidence reflects the reliability of the signals given the quality of the proxies and the internal agreement of tests.  It ranges High > Medium > Low and is derived by combining:

* **Proxy Fit (40 %):** how representative the selected symbol is for the actual GPF plan.  Shariah proxies receive lower scores because SPUS and HLAL may differ from the fund’s holdings.
* **Signal Agreement (25 %):** how consistently multiple proxies and timeframes point in the same direction.
* **Timeframe Agreement (20 %):** whether weekly and monthly tests agree.
* **Data Completeness (15 %):** whether data are available for all proxies without `na` values.

Confidence is displayed alongside final status. Missing data reduces confidence and prevents a misleading positive total. Shariah confidence is normally capped at Medium because its proxies do not represent official holdings.

## Dashboard Mode Logic

In `Dashboard` mode the indicator condenses the status of all four plans in a single table without listing the internal tests.  For each plan it computes:

* **Weekly Status:** “Strong”, “Neutral” or “Weak” based solely on whether its primary proxy (XAUUSD, ACWI, SET50, SPUS/HLAL) is above the weekly EMA‑40.
* **Monthly Status:** same but using monthly EMA‑10.
* **Relative Strength:** “Up” if the primary proxy ratio is above its 12-week average and “Down” otherwise. `Flat` is future work.
* A simple **Score** combining these high‑level signals to rank the plans.

Dashboard results direct the user to the relevant detail mode for further confirmation, but are insufficient on their own to decide plan changes.

## Proxies and Inputs

Implementations must define input variables for all proxy symbols with sensible defaults.  Users can override these inputs at runtime.  The defaults should be:

| Plan               | Primary proxy    | Additional proxies         | Benchmarks    | Macro inputs |
|--------------------|------------------|----------------------------|--------------|--------------|
| **Gold**           | `OANDA:XAUUSD`  | `COMEX:GC1!`, `AMEX:GLD`  | ACWI (for RS) | `TVC:DXY`, `TVC:US10Y`, `FX_IDC:USDTHB` |
| **Global Equity**  | `AMEX:ACWI`     | `AMEX:SPY`, `NASDAQ:QQQ`, `AMEX:VEA`, `AMEX:VWO` | ACWI        | US10Y, DXY, USDTHB |
| **Thai Equity**    | `SET:SET50`     | `SET:SET`, leading Thai stocks (DELTA, PTT, KBANK, ADVANC, KTB, SCB, BBL, GULF, PTTEP, CPN) | ACWI or VWO | USDTHB |
| **Shariah**        | `AMEX:SPUS`     | `NASDAQ:HLAL`              | ACWI/SPY     | US10Y, XAUUSD, USDTHB |

Future versions may allow additional proxies or dynamic weighting, but v0.1 must hard‑code equal weighting across proxies within a plan.

## Version 0.1 Scope

The first version (`v0.1`) will implement the Dashboard mode and all four detail modes using the scoring rules above.  It will not include:

* Fundamental metrics (revenue, EPS, debt ratios, etc.) due to data availability and complexity.
* Alerts or trading signals.  Users must interpret the results themselves.
* Automatic updates or scraping of GPF NAV.
* Integration with external APIs.  Only data available natively in TradingView may be used.

## Success Criteria

* The Pine Script v6 indicator compiles without errors and can be added to any chart.
* In Dashboard mode it displays a table summarising the status of all four plans using default proxies.
* In each detail mode it displays a checklist with at least eight rows: weekly trend, monthly trend, relative strength tests, macro tests, risk/drawdown tests, a final status and a confidence estimate.
* Scores use completed weekly/monthly bars and do not change while those bars are developing.
* Global Equity does not score ACWI against itself; Thai mode reports breadth/concentration; Shariah reports proxy disagreement.
* Missing symbols reduce confidence and never become positive signals.
* All calculations are transparent and traceable to the inputs; no hidden global state or undocumented magic numbers.

## Future Work

Later versions (`v0.2`, `v0.3`, …) may add:

* Simple fundamental signals via TradingView’s `request.financial()` where available.
* A screener for watchlists of individual holdings using Pine’s screener features.
* Scenario simulation and allocation calculators outside Pine (in a separate local app).
* Enhanced weighting of proxies based on published GPF holdings.
