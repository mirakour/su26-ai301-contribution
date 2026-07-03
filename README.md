# CodePath AI301 — Open Source Contribution

**Student:** Kourtney Miranda (@mirakour)
**Course:** CodePath AI301 — Summer 2026
**Status:** Phase IV Complete

---

## Issue Selected

**Repository:** [apache/superset](https://github.com/apache/superset)
**Issue:** [#36189 — D3 percentage format breaks for very small negative numbers](https://github.com/apache/superset/issues/36189)

---

## Phase I — Explore and Select (Week 1)

### Understanding the Issue

When a table column uses a D3 percentage format string like `.8%`, values that are very small (e.g. `-0.00001229`) are displayed as the raw number instead of the formatted percentage. The expected output would be something like `-0.00122900%`.

### Codebase Research

The relevant file is `superset-frontend/plugins/plugin-chart-table/src/utils/formatValue.ts`.

The `formatColumnValue` function routes numeric values through a small number formatter when `Math.abs(value) < 1`. Percentage columns store their values as decimals (e.g. 5% is stored as 0.05), so this condition fires for every value in a percentage column.

---

## Phase II — Reproduce and Plan (Week 2)

### Fix Branch

Branch: [fix/percentage-small-number-formatting](https://github.com/mirakour/superset/tree/fix/percentage-small-number-formatting)

### Root Cause

In `formatValue.ts`, the condition `isNumber && typeof value === 'number' && Math.abs(value) < 1` always fires for percentage columns since their values are stored as decimals (0–1). The intended percentage formatter is never called.

### Solution Plan

Add an `isPercentageFormat()` helper to detect D3 format strings ending with `%`, and guard the small-number path with `!isPercentageFormat(config.d3NumberFormat)`.

---

## Phase III — Build and Test (Week 3)

### Implementation

Modified `superset-frontend/plugins/plugin-chart-table/src/utils/formatValue.ts` — added `isPercentageFormat()` helper and updated the `useSmallNumberFormatter` condition to skip percentage columns.

### Tests

Created `superset-frontend/plugins/plugin-chart-table/src/utils/formatValue.test.ts` with coverage for:
- Very small negative and positive percentage values (the original bug)
- Normal percentage values
- Non-percentage small numbers (should still use small-number formatter)
- Edge cases: null and undefined values

---

## Phase IV — Submit & Iterate (Week 4)

### Pull Request

[apache/superset #41098](https://github.com/apache/superset/pull/41098) — fix(table): use percentage formatter for small values in percentage columns

### Outcome

PR was reviewed and closed by maintainer **rusackas** (Apache Superset core contributor). The issue had already been resolved in a separate PR ([#37980](https://github.com/apache/superset/pull/37980)), which closed the original issue before this contribution was submitted.

Maintainer response: "Ahh, looks like this was already fixed by #37980, which closed #36189. I think we can close this one out. Thanks for the work @mirakour!"

### Automated Review Feedback

Two suggestions from the codeant-ai bot were flagged:
1. The `isPercentageFormat()` helper only checks for `%` but D3 also has a `p` type for percentage formatting — extending detection to cover `.2p` style formats would make the fix more complete.
2. The check only inspects `config.d3NumberFormat` but the effective formatter can also come from a datasource-level saved format — the fix may not cover that path.

These are valid edge cases that would improve the fix in a real merge scenario.

### What I Learned

- Duplicate fixes happen in active open source projects — always check if an issue is truly open before starting work.
- Even a closed PR provides real maintainer feedback and a real code review, which is valuable experience.
- The fix itself was technically sound; the bot reviews identified genuine improvements worth noting.
- Engaging with the codebase, writing tests, and opening a PR are the core skills — whether or not it merges.

