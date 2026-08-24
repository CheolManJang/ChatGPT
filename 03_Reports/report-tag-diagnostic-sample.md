# Sanitized Report TAG Diagnostic Sample

> [!NOTE]
> This is a fictional public sample. Identifiers, values, results, and logic names are illustrative and do not reproduce production report rules.

## Purpose

This sample shows how a temporary Development TAG helps identify the logic responsible for an incorrect report result. It does not define a permanent routing command and must not appear in Live output.

## Before Optimization

```text
ITEM-DEMO-01
RESULT = REVIEW
LOGIC_TAG = CLASSIFY
LOGIC_VERSION = DEV-EXAMPLE-01
VALIDATION = NEEDS-REVIEW
```

Observed problem: the item was marked `REVIEW` even though one required input was missing. Without the TAG, a maintainer might incorrectly change source, sorting, formatting, or delivery logic.

## Focused Diagnosis

```text
TAG            = CLASSIFY
EXPECTED       = INCOMPLETE
ACTUAL         = REVIEW
WEAK_POINT     = missing-input branch evaluated after classification
AFFECTED_LOGIC = classification only
DO_NOT_CHANGE  = source, sorting, formatting, delivery
```

## After Optimization

```text
ITEM-DEMO-01
RESULT = INCOMPLETE
LOGIC_TAG = CLASSIFY
LOGIC_VERSION = DEV-EXAMPLE-02
VALIDATION = PASS
```

## Regression Evidence

```markdown
- [x] Missing input produces INCOMPLETE
- [x] Complete valid input still produces the expected classification
- [x] Sorting output is unchanged
- [x] Formatting output is unchanged
- [x] Delivery readiness is unchanged
- [x] TAG-free Live output matches approved behavior
- [x] Retired TAG does not reappear after an unrelated format change
```

## Live Output

```text
ITEM-DEMO-01
RESULT = INCOMPLETE
```

The useful result is the corrected and regression-tested logic. The diagnostic TAG is removed before Live.
