# CodePath AI301 — Open Source Contribution

**Student:** Kourtney Miranda (@mirakour)
**Course:** CodePath AI301 — Summer 2026

---

## Issue 1: apache/superset #36189 (Weeks 1–4) — Phase IV Complete

D3 percentage formatting bug in formatValue.ts. PR #41098 submitted and closed (already fixed upstream in #37980). Maintainer acknowledged the work. Documented outcome, learnings, and bot review feedback in README.

---

## Issue 2: apache/burr #607 (Weeks 5+)

**Repository:** [apache/burr](https://github.com/apache/burr)
**Issue:** [#607 — Streaming Event type hint should support union types](https://github.com/apache/burr/issues/607)
**Fix Branch:** [fix/stream-type-union-support](https://github.com/mirakour/burr/tree/fix/stream-type-union-support)

---

## Phase I — Explore and Select (Week 5)

### Problem Summary

The `stream_type` parameter in `@streaming_action.pydantic` only accepts a single Pydantic model or `dict`, but not union types like `ModelA | ModelB`. This matters because real-world streaming actions often need to yield different model types depending on the step, and the current restriction forces workarounds like wrapper models or `Any`. I chose this issue because it involves Python's type system and version compatibility (Python 3.10+ `types.UnionType`), which aligns with my learning goal of understanding how libraries handle generic type annotations safely across Python versions.

### Why I Chose This Issue

I chose burr #607 because it sits at the intersection of Python's evolving type system and a real usability gap in a production-grade library. The fix requires understanding `types.UnionType` (introduced in Python 3.10), `typing.Union`, and version guards — skills directly relevant to writing robust, version-compatible Python. It is also clearly scoped (two files, one type alias) and has a maintainer comment confirming the bug is real, making it a good fit for an open source contribution.

### Understanding the Issue

The `stream_type` parameter in `@streaming_action.pydantic` accepts only a single Pydantic model or `dict`. Passing a union of models (e.g. `MyModel1 | MyModel2`) fails with a type error because the current annotation `PartialType = Union[Type[pydantic.BaseModel], Type[dict]]` does not include `types.UnionType` (Python 3.10+ union syntax).

### Codebase Research

Two files affected:
- `burr/integrations/pydantic.py` — `PartialType` type alias and `_validate_and_extract_signature_types_streaming` signature
- `burr/core/action.py` — `stream_type` parameter in `SingleStepStreamingAction.pydantic()`

---

## Phase II — Reproduce and Plan (Week 6)

### Reproduction

With the unpatched code, the following raises a type error at the annotation level:

```python
from pydantic import BaseModel
from burr.core import action

class ModelA(BaseModel):
    value: str

class ModelB(BaseModel):
    count: int

@action.streaming_action.pydantic(
    reads=[],
    writes=[],
    state_input_type=ModelA,
    state_output_type=ModelA,
    stream_type=ModelA | ModelB,  # type hint says this is invalid
)
def my_action(state):
    ...
```

Type checkers (mypy, pyright) flag `stream_type=ModelA | ModelB` as invalid because `ModelA | ModelB` is `types.UnionType`, which is not in `Union[Type[BaseModel], Type[dict]]`.

### Root Cause

`PartialType = Union[Type[pydantic.BaseModel], Type[dict]]` in `burr/integrations/pydantic.py` does not include `types.UnionType`. In Python 3.10+, the `X | Y` syntax creates a `types.UnionType` object, which the current annotation rejects.

### Solution Plan

Update `PartialType` with a version guard:

```python
import sys

if sys.version_info >= (3, 10):
    PartialType = Union[Type[pydantic.BaseModel], Type[dict], types.UnionType]
else:
    PartialType = Union[Type[pydantic.BaseModel], Type[dict]]
```

Also update `_validate_and_extract_signature_types_streaming` to use `Optional[PartialType]` instead of the inline union, and update `burr/core/action.py` similarly.

### Implementation Status

Fix implemented in `burr/integrations/pydantic.py` on branch [fix/stream-type-union-support](https://github.com/mirakour/burr/tree/fix/stream-type-union-support):
- Added `import sys`
- Updated `PartialType` with `sys.version_info >= (3, 10)` guard to include `types.UnionType`
- Updated `_validate_and_extract_signature_types_streaming` signature to use `Optional[PartialType]`

Next: update `burr/core/action.py` and write tests before opening PR.

## Phase III: Implement Fix & Write Tests

**Branch:** [fix/stream-type-union-support](https://github.com/mirakour/burr/tree/fix/stream-type-union-support)

### Changes Implemented

**File 1: `burr/integrations/pydantic.py`** (already done in Phase II)
- Added `import sys` to detect Python version at runtime
- Added version guard for `PartialType`:
  - Python 3.10+: `PartialType = Union[Type[pydantic.BaseModel], Type[dict], types.UnionType]`
  - Python <3.10: `PartialType = Union[Type[pydantic.BaseModel], Type[dict]]` (original)
- Updated `_validate_and_extract_signature_types_streaming` signature to use `Optional[PartialType]`

**File 2: `burr/core/action.py`**
- Updated the `stream_type` parameter annotation in `streaming_action.pydantic()` from `Union[Type["BaseModel"], Type[dict]]` to `Union[Type["BaseModel"], Type[dict], Any]` to reflect that union types are now accepted

### Tests Added

**File: `tests/integrations/test_burr_pydantic.py`**

Added 3 regression tests for issue #607, all guarded with `@pytest.mark.skipif(sys.version_info < (3, 10), ...)`:

1. `test_validate_streaming_signature_accepts_union_stream_type` — verifies `_validate_and_extract_signature_types_streaming` correctly accepts a `types.UnionType` stream_type and returns it unchanged
2. `test_pydantic_streaming_action_accepts_union_stream_type` — verifies the `@pydantic_streaming_action` decorator succeeds without TypeError when given `stream_type=ModelA | ModelB`
3. `test_streaming_action_pydantic_decorator_accepts_union_stream_type` — same check for the higher-level `@streaming_action.pydantic` decorator

### Status

Phase III complete. Fix branch is ready for PR to apache/burr.
