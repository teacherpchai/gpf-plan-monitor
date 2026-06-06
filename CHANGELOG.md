# Changelog

All notable changes to this project will be documented in this file.  The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) and the project adheres to semantic versioning.

## [0.1.0] – 2026-06-06
### Added
* Initial project structure with `README.md`, `SPEC.md`, `CODEX.md`, and this changelog.
* High‑level specification for the GPF Plan Monitor indicator, describing modes, scoring rules, proxies and guiding principles.
* Coding guidelines for CodeX based on the Buddhist Method and Andrej Karpathy’s four principles.
* `LOGICS.md` as the primary behavioral authority.
* Pine Script v6 implementation with Dashboard, Gold, Global Equity, Thai Equity, and Shariah modes.
* Confirmed higher-timeframe requests, full confidence scoring, and missing-data handling.
* Thai breadth/concentration reporting and Shariah proxy-disagreement handling.

### Changed
* Global Equity relative strength excludes meaningless ACWI-versus-ACWI scoring.
* Thai Equity macro scoring is neutral.
* Dashboard data requests execute across historical bars so their contexts are available when the realtime table renders.
