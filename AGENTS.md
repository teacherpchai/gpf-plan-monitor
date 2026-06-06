# Agent Instructions

- Read `LOGICS.md` before changing indicator behavior. It is the primary behavioral authority.
- Explicit user decisions override older conflicting documentation.
- Inspect the live repository before editing; agentmemory may provide supporting context but is not a source of truth.
- Keep the project a Pine Script indicator. Do not add strategies, alerts, allocation calculations, NAV analysis, or external APIs without explicit approval.
- Use Pine Script v6 and confirmed weekly/monthly data only.
- Preserve TradingView Free compatibility and document Pine changes in `CHANGELOG.md`.
- Do not claim the indicator is verified until it compiles and its modes are visually checked in TradingView.
