# Task Management System

> [!NOTE]
> **Document baseline:** 2026-08-24. Reference environment: individual ChatGPT Plus account using ChatGPT web/Work without direct OpenAI API calls. Architectural principles are general; observed behavior depends on the tested plan, context, permissions, connected apps, and rollout. Revalidate after material product changes.

> [!CAUTION]
> **Use at your own risk.** This material is provided for technical education and general information only, without warranties. Evaluate, test, secure, back up, and legally review any implementation. See the [Disclaimer](../DISCLAIMER.md).


## Adoption Decision Summary

| Field | Project record |
|---|---|
| Introduced because | Completed/not-completed flags could not explain results, failures, partial effects, or continuation. |
| Intended improvement | Make priority, ownership, NG, HOLD, results, retry safety, and continuation explicit. |
| Main difficulty | External side effects and interrupted sessions are difficult to reconcile safely. |
| Main advantage | Reliable continuation and auditable execution results. |
| Main disadvantage | More states, records, ownership checks, and recovery work. |
| Observed result | Separate Task Manager architecture is defined; the private live queue is not published. |
| Current status | Architecture approved; implementation incomplete. |
| Retest trigger | Status-model, lock, retry, external-action, or context changes. |

## Functional Boundary

The Task Manager uses its own store, lifecycle, locks, execution attempts, results, and command interface. It must not directly update Development or Live rules. See [Functional Separation: Rule Engine and Task Manager](../01_Architecture/rule-task-functional-separation.md).

## 1. Purpose

The Task Management System records what work must be done, its priority and dependencies, what actually happened, whether it completed normally, and where a later session should continue.

Rules and TASKs are different:

| Rule | TASK |
|---|---|
| Defines required or prohibited behavior | Defines a unit of work |
| Development, Live, History lifecycle | Registered, In Progress, Completed, NG, HOLD, Cancelled lifecycle |
| Reused across many operations | Has a beginning, progress, result, and terminal or waiting state |
| Changes require validation and promotion | Execution requires ownership, evidence, and completion recording |

Rule definitions are maintained separately in the [Rule Management System](../02_Rule_Management/rule-management-system.md).

## 2. Why It Was Built

A completed flag cannot explain what was done. A generic error cannot explain the stop point, partial effects, retry safety, or continuation point. Long-running work may also stop because of missing data, permission failure, validation failure, session interruption, user review, or deliberate HOLD.

The system must allow a new session to continue safely without relying on conversational memory.

## 3. Core TASK Fields

A public logical model includes:

- TASK number or stable identifier
- Title and work description
- Scope and category
- Priority and priority sequence
- Registration time and target order
- Status and status reason
- Owner Session GUID
- Dependency and predecessor references
- Duplicate-prohibited and required-order flags
- Start, heartbeat, completion, cancellation, and update times
- Result description
- Validation evidence
- Failure stop point
- Partial effects
- Retry safety
- Continuation point
- User decision required flag
- Related rule, document, Issue, or external action reference

Private operational values are not published.

## 4. Status Model

Minimum states:

- **Registered:** accepted but not started.
- **In Progress:** owned by an active session.
- **Completed:** work and validation finished with a recorded result.
- **NG:** execution failed; cause and continuation data are required.
- **HOLD:** intentionally paused because a dependency or decision is unresolved.
- **Cancelled:** explicitly stopped without completion.

Useful extended states include Waiting for Validation, Waiting for User Review, Blocked, Timed Out, Retrying, and Superseded.

Allowed transitions must be defined. A TASK must not jump to Completed merely because processing stopped.

## 5. Priority, Ordering, and Duplicates

Priority determines importance; priority sequence resolves order inside the same priority. Equal items default to registration order unless an approved dependency changes execution order.

Before registration, check:

- Exact duplicate TASK
- Same target already In Progress
- Superseding or predecessor TASK
- Required earlier TASK
- Conflicting external action
- Existing Completed result that makes the work unnecessary

## 6. Work Description and Result Description

The work description states the intended outcome and acceptance criteria.

A successful result records:

- What changed
- What was verified
- Evidence or output produced
- Side effects
- Remaining follow-up
- Final continuation state

An NG result records:

- Exact stop point
- Direct cause
- Partial changes or external effects
- Whether retry is safe
- Required repair or decision
- Next safe continuation point

A HOLD result records the unresolved dependency, responsible decision maker, required input, and resume condition.

## 7. Sessions, Locks, and Heartbeats

Each active attempt receives a new Session GUID. The session must own the TASK before modifying shared state or declaring completion.

A heartbeat indicates activity but cannot prove that an external operation is incomplete. On timeout, verify database transactions, files, messages, APIs, and other side effects before retrying.

Ownership mismatch, ambiguous side effects, or missing evidence produces NG or HOLD.

## 8. User Review Boundary

A TASK enters Waiting for User Review or HOLD when it:

- Changes an approved architecture or Live rule
- Requires a material design decision
- Needs user-specific information
- May expose private operational data
- Sends a sensitive external communication
- Cannot determine a safe, bounded action

Straightforward factual corrections and bounded approved work may proceed autonomously, with the result recorded.

## 9. Continuation Across Sessions

A resumable TASK must contain enough information to answer:

- What was already completed?
- What evidence was checked?
- What failed or remains?
- Were there partial side effects?
- Which source and version were used?
- Is retry idempotent?
- What must the next session do first?
- Does the user need to approve anything?

“Continue TASK” must load this record rather than reconstruct state from memory alone.

## 10. Advantages

- Clear queue and execution order
- Reliable result and failure evidence
- Safe continuation after interruption
- Better distinction between NG, HOLD, cancellation, and completion
- Reduced repeated work and duplicate external actions
- Traceability between GitHub Issues, rules, work, and validation

## 11. Disadvantages and Risks

- More status transitions and recordkeeping
- Stale ownership and heartbeat recovery are complex
- External side effects may not be transactionally reversible
- Poor result descriptions still produce ambiguous History
- Overly broad TASKs become difficult to validate and resume
- Automated retries can duplicate actions unless idempotency is proven

## 12. Current Limitations

- Final TASK schema and state-transition table are not finalized.
- Timeout thresholds and stale-session recovery require testing.
- The official private TASK source is not published.
- Cross-context automatic ownership transfer is not guaranteed.
- External delivery and connector side effects require provider-specific verification.
- Public GitHub shows sanitized architecture, not the private live queue.

## 13. Next Work

1. Finalize TASK fields and allowed transitions.
2. Define priority, dependency, duplicate, and supersession rules.
3. Define ownership, heartbeat, timeout, and takeover procedures.
4. Create Completed, NG, HOLD, Cancelled, and Waiting for User Review tests.
5. Define idempotency evidence for external actions.
6. Add a sanitized TASK example and History record.
7. Verify continuation in a new session without conversational reconstruction.
