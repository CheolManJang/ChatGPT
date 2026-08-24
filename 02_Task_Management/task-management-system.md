# Task Management System
> [!NOTE]
> **Document baseline:** 2026-08-24. Reference environment: individual ChatGPT Plus account using ChatGPT web/Work without direct OpenAI API calls. Architectural principles are general; observed behavior depends on the tested plan, context, permissions, connected apps, and rollout. Revalidate after material product changes.
> **기준:** 2026년 8월 24일. 참조 환경: ChatGPT 웹/Work를 사용하며 직접 OpenAI API를 호출하지 않는 개인 ChatGPT Plus 계정. 아키텍처 원칙은 일반적이지만, 관찰된 동작은 테스트한 플랜, 맥락, 권한, 연결된 앱 및 배포 상태에 따라 달라집니다. 중요한 제품 변경 후 다시 검증하십시오.

> [!CAUTION]
> **Use at your own risk.** This material is provided for technical education and general information only, without warranties. Evaluate, test, secure, back up, and legally review any implementation. See the [Disclaimer](../DISCLAIMER.md).
> **사용 시 주의.** 본 자료는 교육 및 일반 정보 제공 목적이며, 어떠한 보증도 제공하지 않습니다. 모든 구현을 직접 평가·테스트하고, 보안과 백업을 확인하며, 필요한 법적 검토를 수행하십시오. [면책 조항](../DISCLAIMER.md)을 참조하십시오.


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
| Observed result | **Project observation:** TASK records were used to preserve priority, status, result/NG reason, HOLD, ownership, and continuation instead of relying on chat memory alone. The private queue and operational values are not published. |
| Current status | Implemented for the project's bounded workflow; schema hardening, recovery tests, and external-side-effect reconciliation remain in Development. |
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

## 2.1 Actual Implementation Experience

This subsystem was not only discussed as a future architecture. A private work-management store was used to register and update real project work. The implementation experience included:

- Priority and a separate priority sequence, with equal-priority work falling back to registration order
- Registration, start, completion, cancellation, NG, and HOLD states
- Work description and a separate result description so “finished” also records what was done
- Work area, required-order, and duplicate-prohibited controls
- Session ownership and heartbeat information for interrupted or concurrent work
- Continuation records so a later chat or Work session can resume the same TASK
- NG records that preserve the stop point and cause rather than silently appearing complete
- HOLD or Waiting for User Review when a decision must be made by the user

**Observed improvement:** later work could distinguish normal completion from NG, identify what had already changed, and find the next continuation point without treating the last chat response as the authoritative execution record.

**Observed difficulty:** a stopped ChatGPT response does not prove that database, file, connector, or message side effects also stopped. Ownership and heartbeat helped identify stale work, but side effects still required verification before retry.

See the [sanitized TASK implementation sample](task-implementation-sample.md) for a reconstructed record and state transition.

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

- The implemented private schema is not published and may continue to evolve.
- Not every extended status has completed transition and recovery testing.
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
6. Extend the sanitized TASK example with timeout, duplicate, and external-side-effect recovery cases.
7. Verify continuation in a new session without conversational reconstruction.
