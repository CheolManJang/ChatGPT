# Feature Documentation Standard

> [!NOTE]
> **Standard baseline:** 2026-08-24. This standard applies to every public feature, subsystem, optimization, experiment, integration, and retired approach in this repository.

## Purpose

A feature document must help a reader decide not only **how** something works, but also **why it was introduced, whether it improved the system, what it cost, where it failed, and whether it should be used now**.

A usage description without adoption context is incomplete.

## Required Sections

Every feature document must answer:

1. **Why it was introduced**
   - What situation triggered the feature?
   - What was difficult or unsafe before it existed?

2. **Problem to solve**
   - What precise failure, delay, ambiguity, cost, or risk was targeted?
   - Who or what was affected?

3. **Expected improvement**
   - What should become faster, safer, clearer, more accurate, easier to recover, or easier to maintain?

4. **How it works**
   - Components, inputs, outputs, states, source boundary, interfaces, and dependencies.

5. **Adoption difficulties**
   - What was difficult while designing, migrating, testing, or operating it?
   - Which assumptions proved wrong?

6. **Problems and risks**
   - Failure modes, synchronization issues, security risks, privacy concerns, regressions, and provider dependencies.

7. **Advantages**
   - Confirmed or expected benefits, with evidence class and baseline.

8. **Disadvantages and trade-offs**
   - Added latency, complexity, storage, maintenance, review, permissions, or recovery burden.

9. **Observed result**
   - What actually improved?
   - What did not improve?
   - Which result is measured, observed, inferred, partial, or unverified?

10. **Current status and baseline**
    - Development, testing, Live, HOLD, retired, or replaced.
    - Verification date, environment, context, permissions, and retest trigger.

11. **Limitations**
    - What is not guaranteed or not supported?
    - What remains incomplete?

12. **Rejected, retired, or alternative approaches**
    - What else was tried?
    - Why was it not selected or why was it removed?

13. **Rollback, removal, and recovery**
    - How is the feature disabled, reverted, migrated, or recovered?
    - What state cannot be restored exactly?

14. **Next work and acceptance criteria**
    - What must be completed and verified before broader use or Live promotion?

15. **Publication safety**
    - Which data, logic, prompts, credentials, links, recipients, or operational details must remain private?

## Required Adoption Summary

Near the beginning of each feature document, include a compact summary:

| Field | Required content |
|---|---|
| Introduced because | Original need or failure |
| Intended improvement | Expected benefit |
| Main difficulty | Adoption or implementation challenge |
| Main advantage | Most important benefit |
| Main disadvantage | Most important trade-off |
| Observed result | Actual result and evidence class |
| Current status | Development, Live, HOLD, Retired, or Replaced |
| Retest trigger | Change that requires verification again |

## Evidence Labels

Use one or more of:

- **Official documentation**
- **Project observation**
- **Controlled test**
- **Inference**
- **Not yet measured**
- **User-confirmed decision**

Do not present an inference as a confirmed product capability.

## Status Vocabulary

- **Proposed:** not approved.
- **Development:** under design or optimization.
- **Testing:** implementation exists but acceptance is incomplete.
- **Live:** approved and verified for the defined scope.
- **HOLD:** intentionally paused.
- **Retired:** removed and not approved for current use.
- **Replaced:** superseded by a named alternative.
- **Unknown:** evidence is insufficient.

## Change Rule

Any material feature change must update:

1. The feature document
2. The adoption summary
3. Current status
4. Verification baseline
5. Advantages, disadvantages, and limitations if changed
6. Regression and recovery requirements
7. CHANGELOG
8. Related Issue or TASK when applicable

A small change can reintroduce retired AI behavior. Every material change is therefore a regression trigger.

## Publication Gate

Do not publish a feature document until:

- Required sections are present
- Evidence labels are accurate
- Current status is explicit
- Measured and unmeasured claims are separated
- Advantages and disadvantages are both stated
- Failure and recovery are documented
- Private data and reconstructable core algorithms are removed
- Links and document baseline are verified
