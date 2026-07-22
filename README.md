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
**Pull Request:** [apache/burr #845](https://github.com/apache/burr/pull/845) — Open, awaiting review

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

### Environment Setup

**Setup approach:** Cloned `apache/burr`, created a virtualenv, and installed dev dependencies with `pip install -e ".[dev,pydantic]"`. Ran the existing Pydantic integration tests with `pytest tests/integrations/test_burr_pydantic.py` to confirm baseline passes before making changes.

**Challenges encountered:**
- The repo uses optional extras and the Pydantic tests require `pip install -e ".[pydantic]"` explicitly — just `pip install -e .` missed the dependency. Resolved by checking `pyproject.toml` for the correct extras group.
- Python 3.10+ is required to reproduce the union type bug with `X | Y` syntax. Confirmed environment was on Python 3.11.

### Reproduction

With the unpatched code, the following raises a `TypeError` at decoration time:

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
    stream_type=ModelA | ModelB,  # raises TypeError
)
def my_action(state):
    ...
```

**Expected behavior:** The decorator should accept `ModelA | ModelB` as a valid `stream_type` value, since it is a valid Python 3.10+ type expression.

**Actual behavior:** `TypeError` is raised because `ModelA | ModelB` produces a `types.UnionType` object, which is not in `Union[Type[BaseModel], Type[dict]]`.

### Root Cause

`PartialType = Union[Type[pydantic.BaseModel], Type[dict]]` in `burr/integrations/pydantic.py` (line ~47) does not include `types.UnionType`. In Python 3.10+, the `X | Y` syntax creates a `types.UnionType` object (distinct from `typing.Union`), which the current annotation rejects.

### Solution Plan (UMPIRE)

**Understand:** The `PartialType` alias controls what `stream_type` values pass the internal validation. It needs to accept `types.UnionType` on Python 3.10+.

**Match:** Similar version guards already exist in the Python standard library ecosystem (e.g., `isinstance` checks gated on `sys.version_info`). No existing helper in burr handles this.

**Plan:**
1. Add `import sys` to `burr/integrations/pydantic.py`
2. Add a version guard expanding `PartialType`:
   ```python
   if sys.version_info >= (3, 10):
       PartialType = Union[Type[pydantic.BaseModel], Type[dict], types.UnionType]
   else:
       PartialType = Union[Type[pydantic.BaseModel], Type[dict]]
   ```
3. Update `_validate_and_extract_signature_types_streaming` to use `Optional[PartialType]`
4. Update the `stream_type` annotation in `burr/core/action.py` to use `Any` to reflect the expanded range

**Implement:** See Phase III.

**Review:** Checked that the change is backward-compatible (Python < 3.10 uses the original `PartialType`).

**Evaluate:** Three regression tests added and passing.

### Implementation Status

Fix implemented in `burr/integrations/pydantic.py` on branch [fix/stream-type-union-support](https://github.com/mirakour/burr/tree/fix/stream-type-union-support):
- Added `import sys`
- Updated `PartialType` with `sys.version_info >= (3, 10)` guard to include `types.UnionType`
- Updated `_validate_and_extract_signature_types_streaming` signature to use `Optional[PartialType]`

---

## Phase III — Implement Fix & Write Tests (Week 7)

**Branch:** [fix/stream-type-union-support](https://github.com/mirakour/burr/tree/fix/stream-type-union-support)

### Changes Implemented

**File 1: `burr/integrations/pydantic.py`** (commit `0804ca3`)
- Added `import sys` to detect Python version at runtime
- Added version guard for `PartialType`:
  - Python 3.10+: `PartialType = Union[Type[pydantic.BaseModel], Type[dict], types.UnionType]`
  - Python <3.10: `PartialType = Union[Type[pydantic.BaseModel], Type[dict]]` (original)
- Updated `_validate_and_extract_signature_types_streaming` signature to use `Optional[PartialType]`

**File 2: `burr/core/action.py`** (commit `b9ca822`)
- Updated the `stream_type` parameter annotation in `streaming_action.pydantic()` from `Union[Type["BaseModel"], Type[dict]]` to `Union[Type["BaseModel"], Type[dict], Any]`

**File 3: `tests/integrations/test_burr_pydantic.py`** (commit `154305d`)
- Added 3 regression tests for issue #607

### Tests Added

All tests guarded with `@pytest.mark.skipif(sys.version_info < (3, 10), reason="Union type syntax requires Python 3.10+")`:

1. `test_validate_streaming_signature_accepts_union_stream_type` — verifies `_validate_and_extract_signature_types_streaming` correctly accepts a `types.UnionType` stream_type and returns it unchanged
2. `test_pydantic_streaming_action_accepts_union_stream_type` — verifies the `@pydantic_streaming_action` decorator succeeds without TypeError when given `stream_type=ModelA | ModelB`
3. `test_streaming_action_pydantic_decorator_accepts_union_stream_type` — same check for the higher-level `@streaming_action.pydantic` decorator

**Test suite status:** Existing tests pass. The new tests skip cleanly on Python < 3.10 and pass on Python 3.10+.

**Testing approach:** Manual verification by running `pytest tests/integrations/test_burr_pydantic.py -v` before and after the fix. Confirmed the three new tests pass and no existing tests regress.

### Challenges Faced

- **raw.githubusercontent.com caching**: After committing changes to `action.py`, the raw file URL continued showing the old content for several minutes due to CDN caching. Worked around this by verifying the commit via the GitHub API (`api.github.com/repos/.../contents/...`) which bypasses the cache.
- **UMPIRE plan documentation gap**: Initial write-up of the solution plan was a plain bullet list rather than a structured framework. Restructured using the Understand/Match/Plan/Implement/Review/Evaluate format.

### Implementation Progress

| File | Change | Commit |
|---|---|---|
| `burr/integrations/pydantic.py` | Added `types.UnionType` to `PartialType` via version guard | `0804ca3` |
| `burr/core/action.py` | Updated `stream_type` annotation to accept `Any` | `b9ca822` |
| `tests/integrations/test_burr_pydantic.py` | Added 3 regression tests | `154305d` |

---

## Phase IV — Submit PR & Iterate (Weeks 8–10)

**Pull Request:** [apache/burr #845 — fix: allow union types (X | Y) for stream_type in streaming_action.pydantic](https://github.com/apache/burr/pull/845)
**Status:** Open, awaiting maintainer review
**Submitted:** July 22, 2026

### PR Summary

The PR targets `apache/burr` main branch. It includes changes to 3 files (pydantic.py, action.py, test file) across 3 commits. The PR description references `Closes #607` and follows the project's PR template. The fix is backward-compatible — no behavior change on Python < 3.10.

### Maintainer Feedback Log

| Date | Feedback | Response | Commit |
|---|---|---|---|
| Awaiting review | — | — | — |

### Learnings & Reflections

**What I learned:**
- Python's `types.UnionType` (from `X | Y` syntax in 3.10+) is a distinct object from `typing.Union[X, Y]`. Libraries need version guards to handle both gracefully.
- CDN caching on raw.githubusercontent.com can mislead you into thinking a commit didn't apply — always verify via the GitHub API or the UI diff view.
- Open source contribution requires reading existing test patterns closely. The burr test suite uses `AppStateModel` as a fixture base; writing tests that fit the existing style took more time than the fix itself.
- The UMPIRE framework is genuinely useful for structuring a solution plan — it forces you to distinguish between the symptom and the root cause before writing any code.

**What I'd do differently:**
- Set up the full dev environment (including CI) before starting Phase II to avoid environment surprises.
- Write the structured solution plan (UMPIRE) from the start rather than restructuring it later.
- Cache the GitHub API response check into a helper script to speed up verification.
