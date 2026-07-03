# CodePath AI301 — Open Source Contribution

**Student:** Kourtney Miranda (@mirakour)
**Course:** CodePath AI301 — Summer 2026

---

## Issue 1: apache/superset #36189 (Weeks 1–4) — Phase IV Complete

D3 percentage formatting bug in formatValue.ts. PR #41098 submitted and closed (already fixed upstream in #37980). Maintainer acknowledged the work. Documented outcome, learnings, and bot review feedback in README.

---

## Issue 2: apache/burr #607 (Weeks 5+)

**Repository:** [apache/burr](https://github.com/apache/burr)
**Issue:** [#607 — Streaming Event type hint should support union types](https://github.com/apache/burr/issues/36189)
**Fix Branch:** [fix/stream-type-union-support](https://github.com/mirakour/burr/tree/fix/stream-type-union-support)

---

## Phase I — Explore and Select (Week 5)

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

