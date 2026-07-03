# CodePath AI301 — Open Source Contribution

**Student:** Kourtney Miranda (@mirakour)
**Course:** CodePath AI301 — Summer 2026

---

## Issue 1: apache/superset #36189 (Weeks 1–4)

D3 percentage formatting bug — fixed in formatValue.ts. PR #41098 submitted and closed (already fixed upstream in #37980). Full write-up in the sections below.

**Status:** Phase IV Complete

---

## Phase I — Explore and Select (Week 1)

**Issue:** [apache/superset #36189](https://github.com/apache/superset/issues/36189) — D3 percentage format breaks for very small negative numbers

When a table column uses a D3 percentage format string like `.8%`, values that are very small (e.g. `-0.00001229`) are displayed as the raw number. Root cause: `Math.abs(value) < 1` in `formatValue.ts` always fires for decimal-stored percentage values, routing them to the wrong formatter.

---

## Phase II — Reproduce and Plan (Week 2)

Branch: [fix/percentage-small-number-formatting](https://github.com/mirakour/superset/tree/fix/percentage-small-number-formatting)

Fix plan: Add `isPercentageFormat()` helper and guard the small-number path with `!isPercentageFormat(config.d3NumberFormat)`.

---

## Phase III — Build and Test (Week 3)

Implemented `isPercentageFormat()` in `formatValue.ts`. Created `formatValue.test.ts` with coverage for the bug scenario and edge cases.

PR: [apache/superset #41098](https://github.com/apache/superset/pull/41098)

---

## Phase IV — Submit & Iterate (Week 4)

PR reviewed and closed by maintainer rusackas — issue already fixed in #37980. Maintainer said "Thanks for the work @mirakour!" Bot review flagged two valid edge cases (D3 `p` format type; datasource-level formatters). Documented outcome in README.

**What I learned:** Check issue status carefully before starting. A closed PR still earns real maintainer interaction and code review experience.

---

## Issue 2: apache/burr #607 (Week 5+)

**Status:** Phase I — Explore and Select

**Repository:** [apache/burr](https://github.com/apache/burr)
**Issue:** [#607 — Streaming Event type hint should support union types](https://github.com/apache/burr/issues/607)

### Understanding the Issue

When using `@streaming_action.pydantic`, the `stream_type` parameter only accepts a single Pydantic model or `dict`. Passing a union of models (e.g. `MyModel1 | MyModel2`) causes a type error because the current annotation is:

```python
PartialType = Union[Type[pydantic.BaseModel], Type[dict]]
```

In Python 3.10+, `MyModel1 | MyModel2` creates a `types.UnionType` object, which is not covered by this annotation.

### Codebase Research

Two files need updating:

1. **`burr/integrations/pydantic.py`** — `PartialType` type alias and `_validate_and_extract_signature_types_streaming` function signature
2. **`burr/core/action.py`** — `stream_type` parameter annotation in `SingleStepStreamingAction.pydantic()`

### Solution Plan

Extend `PartialType` to also accept `types.UnionType` (Python 3.10+) with a version-compatible guard:

```python
import sys
import types as builtin_types

if sys.version_info >= (3, 10):
    PartialType = Union[Type[pydantic.BaseModel], Type[dict], builtin_types.UnionType]
else:
    PartialType = Union[Type[pydantic.BaseModel], Type[dict]]
```

Update both function signatures to use the expanded type, and ensure runtime validation handles union types gracefully.

