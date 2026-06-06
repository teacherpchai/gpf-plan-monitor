# GPF Plan Monitor

The **GPF Plan Monitor** is a Pine Script v6 TradingView indicator that helps users track the market conditions of four GPF (Government Pension Fund of Thailand) investment plans without relying on official NAV data. By monitoring proxy assets and macro indicators on confirmed weekly and monthly bars, it summarises whether each plan group is currently strong, neutral, weak, or worth deeper review.

**Note:** This tool analyses proxy market data (e.g., ACWI, SET50, XAUUSD) and does *not* provide or interpret the actual GPF NAV.  It is not investment advice.

## Features

- **Dashboard** view showing the status and score of each plan (Gold, Global Equity, Thai Equity, Shariah).
- **Detail** modes for each plan revealing the trend, relative strength, macro factors, risk score, final status and confidence.
- Configurable proxy symbols and mode via Pine Script inputs.
- Works on TradingView Free by using a single script with a mode selector.
- Uses only completed weekly/monthly bars to avoid repainting developing higher-timeframe signals.
- Separates market status from confidence in the available evidence.

## Installation

1. Create or open a chart on TradingView (preferably `ACWI`).
2. Open the Pine Editor tab.
3. Copy the contents of `pine/gpf_plan_monitor_v0_1.pine` into the editor.
4. Click “Add to Chart”.
5. In the indicator settings, select the desired `mode` (Dashboard or one of the detail modes) and adjust proxy symbols if needed.

## Modes

| Mode               | Description                                                                       |
|--------------------|-----------------------------------------------------------------------------------|
| **Dashboard**      | Displays a table summarising each plan’s status and score.                        |
| **GoldDetail**     | Checks gold proxies (XAUUSD, GLD) and macro conditions (DXY, US10Y, USDTHB).      |
| **GlobalEquityDetail** | Checks global equity proxies (ACWI, SPY, QQQ, VWO, VEA).                    |
| **ThaiEquityDetail**   | Checks Thai equity proxies (SET50, SET, and major constituents).             |
| **ShariahDetail**      | Checks Shariah‑compliant proxies (SPUS, HLAL) and relevant macro factors.   |

## Limitations

- Does not reflect official GPF performance or asset allocations.
- Macro factors are simplified; Thai USDTHB scoring is neutral and fundamental data is not included in v0.1.
- The indicator cannot make trade recommendations.  It provides situational awareness only.
- Proxy coverage for the Shariah plan may not fully represent GPF's actual Shariah holdings; the confidence measure accounts for this.
- Missing proxy data reduces confidence and prevents a misleading positive score.

## Contributing

Read `LOGICS.md` first for the behavioral authority, then `SPEC.md` and `CODEX.md` for the supporting specification and coding workflow.

## License

MIT
