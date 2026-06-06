# Coding Guidelines for CodeX

This document contains rules and expectations for the coding agent responsible for implementing and maintaining the **GPF Plan Monitor** indicator. `LOGICS.md` is the primary behavioral authority. `SPEC.md` provides supporting context. Explicit user decisions override older conflicting text.

## Alignment with Buddhist Method and Karpathy Guidelines

1. **Think Before Coding (Kalāma, Think Before Coding)** – Whenever you receive a task, restate the assumptions, unknowns and success criteria clearly before writing code.  If any requirement is ambiguous, ask targeted questions.  Document all reasoning in comments.  Avoid hidden confusion【451674326895714†L292-L301】.

2. **Simplicity First (Simplicity First)** – Write the minimal amount of code required to satisfy the spec.  Do not introduce unnecessary functions, abstractions or features; follow YAGNI.  Use built‑in Pine functions where possible.  Resist the urge to generalise prematurely【451674326895714†L304-L314】.

3. **Surgical Changes (Surgical Changes)** – Only edit or add code directly related to the task.  Preserve existing style and structure.  Avoid refactoring or reformatting unrelated parts.  Remove only unused code introduced by your change【451674326895714†L317-L333】.

4. **Goal‑Driven Execution (Goal‑Driven Execution)** – Break large tasks into smaller steps with explicit success criteria.  For each step, explain how you will test it (e.g., compile error‑free in TradingView).  After each change, verify the goal before declaring the task complete【451674326895714†L335-L352】.

5. **Verify Facts (Kalāma)** – Use the citations in `SPEC.md` as authoritative references.  If you need to recall external information, search or consult the provided citations.  Do not invent details.  Document all facts you rely on【588121122412891†L265-L272】.

6. **Root‑Cause Analysis (Yoniso Manasikāra)** – When debugging, identify the underlying cause of a problem instead of patching symptoms.  Use logging or print statements to isolate issues before making changes【588121122412891†L265-L272】.

7. **Rewrite Instead of Patch (Anatta)** – If a section of code fails to meet the spec, rewrite it cleanly rather than layering patches.  Ensure the new version fits naturally with the surrounding code【588121122412891†L265-L272】.

8. **Remove Bugs, Don’t Hide Them (Pahāna)** – Do not suppress errors by using try/except patterns or silently ignoring failures.  Fix the underlying bug or clearly signal the error condition【588121122412891†L265-L272】.

9. **Maintain Equanimity (Upekkhā)** – Remain steady when encountering conflicting user demands, scarce data or compile errors.  Follow the guidelines calmly and ask for clarification if necessary【588121122412891†L265-L272】.

## Project‑Specific Rules

* Use **Pine Script v6**. Do not use Pine strategies or alert conditions; implement only indicators.
* Create a single script in `pine/gpf_plan_monitor_v0_1.pine`.  Use inputs to allow the user to choose between modes and to override default proxy symbols.
* Implement only the features described in `SPEC.md` for version 0.1: Dashboard, GoldDetail, GlobalEquityDetail, ThaiEquityDetail, ShariahDetail.  Do not add fundamental data or other modes.
* Weekly and monthly calculations must use confirmed bars only. Use a one-bar expression offset with `lookahead=barmerge.lookahead_on`; do not use unoffset `lookahead_on`.
* Keep each active mode within TradingView Free request limits. Bundle related values in tuple requests and calculate only the selected mode.
* Use a table for display (`table.new()`) and align columns consistently.  Provide column headers and row labels.  Avoid dynamic table sizes—define a fixed number of rows per mode to maintain clarity.
* Document each function with comments.  Include formulas for scoring and threshold values.  Avoid magic numbers; if a constant is used (e.g. drawdown threshold 25 %), define it at the top with a comment referencing `SPEC.md`.
* Do not attempt to fetch external data or GPF NAV.  Use only TradingView symbols available to the user.  Provide sensible defaults matching the spec but allow user override.
* Implement confidence from proxy fit, signal agreement, timeframe agreement, and data completeness. Missing data must not be interpreted as a positive signal.
* Do not claim full verification until the script compiles in TradingView and all five modes are visually checked.
* Update `CHANGELOG.md` with a new entry whenever you modify the Pine script.  Describe the change succinctly and reference the spec version.

## Workflow for Each Task

1. **Analyse the assignment** and reference `SPEC.md` to understand the requirements.  Write down assumptions, success criteria and unknowns.
2. **Plan the changes** in clear, small steps.  For example, “Add a new macro test for USDTHB” or “Fix the scoring calculation for Thai Equity”.
3. **Implement** the changes following Simplicity First and Surgical Changes.  Use descriptive variable names.  Write comments referencing spec sections.
4. **Test** the script by compiling in TradingView or with automated tests if available.  Ensure that tables display correctly and values update on the correct timeframe.
5. **Verify** the result meets the success criteria.  If not, return to step 2.
6. **Record** the changes in `CHANGELOG.md` with version numbers and a brief description.
7. **Ask for clarification** if any part of the task remains ambiguous.  Do not proceed based on guesswork.
