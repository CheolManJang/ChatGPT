# Functional Separation: Rule Engine and Task Manager

> [!NOTE]
> **Document baseline:** 2026-08-24. This document defines the approved functional boundary. Implementation status must be verified separately; documentation does not prove that a private operational database has already been migrated.

> [!CAUTION]
> **Use at your own risk.** This is sanitized technical architecture. It excludes production rules, TASK contents, private prompts, operational data, and core financial algorithms. See the [Disclaimer](../DISCLAIMER.md).

## 1. Decision

Rule management and TASK management are separate subsystems.

- The **Rule Engine** owns reusable operational policy.
- The **Task Manager** owns units of work and their execution state.
- Neither subsystem directly changes the other's internal state.
- Integration occurs through stable references, commands, results, and domain events.
- Shared infrastructure is limited to technical primitives such as GUID generation, time, session identity, logging, authorization, and database access helpers.

## 2. Separate Ownership

| Concern | Rule Engine | Task Manager |
|---|---|---|
| Purpose | Define and evaluate approved behavior | Plan, execute, record, and resume work |
| Primary store | `rules.db` | `tasks.db` |
| Main lifecycle | Development → Validated → Live → Replaced/Inactive | Registered → In Progress → Completed/NG/HOLD/Cancelled |
| Versioning | Rule version and promotion History | TASK attempt and result History |
| Lock target | Rule, group, scope, promotion | TASK, execution attempt, dependency |
| Success evidence | Validation and promotion result | Acceptance checks and execution result |
| Retry meaning | Revalidate or repeat an idempotent promotion | Resume or repeat a safe execution attempt |
| Public boundary | Sanitized rule architecture | Sanitized work-management architecture |

A shared table containing both rule and TASK state is prohibited.

## 3. Rule Engine Components

- Rule registry
- Development store
- Live store
- Rule History
- Validation service
- Duplicate, similarity, and conflict service
- Promotion and rollback service
- Rule evaluation interface
- Rule-specific lock namespace

The Rule Engine must not own TASK priority, TASK completion, NG/HOLD workflow, or continuation state.

## 4. Task Manager Components

- TASK registry
- Priority and dependency queue
- Execution-attempt store
- Result and failure History
- NG, HOLD, cancellation, timeout, and user-review workflow
- Continuation and retry-safety service
- TASK-specific lock namespace

The Task Manager must not edit Development or Live rule rows directly.

## 5. Integration Contract

Permitted links include:

- `Related_Rule_GUID` on a TASK
- `Requested_By_Task_GUID` on a rule-change proposal
- `Rule_Version_Used` on a TASK execution attempt
- Correlation ID shared by a command and result
- Immutable events such as `RuleValidationRequested`, `RulePromoted`, `TaskBlockedByRule`, and `TaskCompleted`

A TASK that proposes a rule change sends a command to the Rule Engine. The Rule Engine validates and returns a result. The TASK records that result but does not update the rule store itself.

A rule promotion that requires implementation work can request a new TASK. The Rule Engine does not mark that TASK complete.

## 6. Transaction Boundary

Each subsystem commits only its own database transaction.

A cross-database workflow must use:

1. A correlation ID
2. A durable command or outbox record
3. Idempotent processing
4. A returned result or inbox record
5. Reconciliation when one side commits and the other side is interrupted

A transaction spanning both SQLite files must not be assumed to be atomic merely because the calls occur in one function.

## 7. Separate Interfaces

Recommended command namespaces:

### Rule Engine

- `RULE.REGISTER_DEV`
- `RULE.VALIDATE`
- `RULE.PROMOTE`
- `RULE.ROLLBACK`
- `RULE.LOAD_LIVE`
- `RULE.HISTORY`

### Task Manager

- `TASK.REGISTER`
- `TASK.START`
- `TASK.COMPLETE`
- `TASK.NG`
- `TASK.HOLD`
- `TASK.RESUME`
- `TASK.CANCEL`
- `TASK.HISTORY`

MENU may route to these commands but does not own their business logic.

## 8. Separate Status Models

Rule statuses and TASK statuses must use different enumerations and transition tables.

Examples:

- A Live rule is not “Completed.”
- A TASK is not “Live.”
- Rule validation failure does not automatically mean the requesting TASK is NG; the Task Manager decides whether the result means NG, HOLD, or revision required.
- A TASK cancellation does not delete or deactivate a related rule.

## 9. Shared Infrastructure Boundary

Allowed shared technical services:

- GUID generation
- UTC timestamp provider
- Session identity
- Structured logging
- Authentication and authorization primitives
- SQLite connection factory
- Serialization and validation utilities
- Correlation and idempotency helpers

Prohibited shared business service:

- One Global Function with branches that directly updates both rule and TASK state.

A top-level coordinator may call both subsystems, but each subsystem must expose and enforce its own contract.

## 10. Failure Isolation

- Rule database failure stops rule evaluation or promotion but must not corrupt TASK History.
- Task database failure stops queue or result updates but must not modify Live rules.
- A failed integration message remains reconcilable by correlation ID.
- Retry occurs only after checking both stores and any external side effects.
- NG and HOLD records include the responsible subsystem and continuation point.

## 11. Migration Plan

1. Freeze new mixed-schema additions.
2. Inventory existing rule and TASK tables, functions, statuses, and locks.
3. Classify every field by owning subsystem.
4. Create separate `rules.db` and `tasks.db` schemas.
5. Split Global Functions into Rule and TASK interfaces.
6. Add correlation and idempotency records.
7. Migrate sanitized test data first.
8. Run rule lifecycle and TASK lifecycle tests independently.
9. Run interruption and reconciliation tests.
10. Migrate private operational data only after backup and user-approved validation.
11. Keep a rollback package and migration result record.

## 12. Acceptance Criteria

Functional separation is complete only when:

- Rule and TASK data are stored separately.
- Rule and TASK status transitions are independently enforced.
- No TASK code directly updates rule tables.
- No Rule Engine code directly updates TASK status.
- Cross-system requests are traceable by correlation ID.
- Partial commits can be reconciled safely.
- Independent backup and restore tests pass.
- MENU/CMD routes to separate interfaces.
- Existing operational behavior passes regression tests.
- Migration evidence and rollback instructions are recorded.

## 13. Current Status

The public functional boundary is now defined. This does not claim that the private operational implementation has already completed database and code migration. Implementation must be tracked as a separate TASK with backup, validation, NG/HOLD, and rollback evidence.
