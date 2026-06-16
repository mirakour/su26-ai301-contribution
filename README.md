# CodePath AI301 — Open Source Contribution

**Student:** Kourtney Miranda (@mirakour)
**Course:** CodePath AI301 — Summer 2026
**Status:** Phase III Complete

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

### Reproduction Steps

1. Clone apache/superset and set up the frontend dev environment
2. Start the dev server: `cd superset-frontend && npm run dev`
3. Create or open a dashboard with a Table chart
4. Add a numeric column with a D3 format of `.8%`
5. Ensure the column contains very small values like `-0.00001229`
6. Observe: the cell displays `-0.00001229` instead of `-0.00122900%`
7. Open `superset-frontend/plugins/plugin-chart-table/src/utils/formatValue.ts`
8. Confirm: `Math.abs(value) < 1` routes percentage values to the wrong formatter

### Root Cause

In `formatValue.ts`, the condition for using the small-number formatter is:

```
isNumber && typeof value === 'number' && Math.abs(value) < 1
```

Percentage columns store values as decimals (0–1), so this condition is always true for them. The intended percentage formatter (e.g. `.8%`) is never called.

### Solution Plan

Add an `isPercentageFormat()` helper that detects D3 format strings ending with `%`, and add `!isPercentageFormat(config.d3NumberFormat)` to the condition so percentage columns bypass the small-number formatter.

---

## Phase III — Build and Test (Week 3)

### Implementation

Modified file: `superset-frontend/plugins/plugin-chart-table/src/utils/formatValue.ts`

Added a helper function that checks if a D3 format string is a percentage format (ends with `%`). Updated the `useSmallNumberFormatter` condition to skip percentage columns, ensuring they always use their intended D3 formatter instead of being intercepted by the small-number path.

### Tests

Created new test file: `superset-frontend/plugins/plugin-chart-table/src/utils/formatValue.test.ts`

Test coverage includes:
- Very small negative percentage values (the original bug case from issue #36189)
- Very small positive percentage values
- Normal percentage values that should format correctly
- Non-percentage small numbers that should still use the small-number formatter
- Edge cases: null and undefined values

### Pull Request

PR submitted to apache/superset from branch `fix/percentage-small-number-formatting`

### Key Files Changed

| File | Change |
|------|--------|
| `superset-frontend/plugins/plugin-chart-table/src/utils/formatValue.ts` | Added `isPercentageFormat()` helper and updated condition |
| `superset-frontend/plugins/plugin-chart-table/src/utils/formatValue.test.ts` | New test file covering the bug fix and edge cases |

