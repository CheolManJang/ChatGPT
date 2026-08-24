# Rule Management System

> [!NOTE]
> **Document baseline:** 2026-08-24. Reference environment: individual ChatGPT Plus account using ChatGPT web/Work without direct OpenAI API calls. Architectural principles are general; observed behavior depends on the tested plan, context, permissions, connected apps, and rollout. Revalidate after material product changes.

> [!CAUTION]
> **Use at your own risk.** This material is provided for technical education and general information only, without warranties. Evaluate, test, secure, back up, and legally review any implementation. See the [Disclaimer](../DISCLAIMER.md).


## Adoption Decision Summary

| Field | Project record |
|---|---|
| Introduced because | Conversation and Memory mixed proposals, obsolete instructions, and approved operational rules. |
| Intended improvement | Create authoritative Development, Live, and History lifecycles with validation and rollback. |
| Main difficulty | Defining identifiers, conflicts, promotion, locking, and retention without creating another ambiguous layer. |
| Main advantage | Traceable and safer rule changes. |
| Main disadvantage | More schema, validation, migration, and concurrency complexity. |
| Observed result | Functional boundary is defined; private implementation migration remains to be verified. |
| Current status | Architecture approved; implementation incomplete. |
| Retest trigger | Schema, model behavior, concurrency, promotion, or source-boundary changes. |

## Functional Boundary

The Rule Engine uses its own store, lifecycle, locks, validation, History, and command interface. It must not own TASK status, priority, execution attempts, or continuation state. See [Functional Separation: Rule Engine and Task Manager](../01_Architecture/rule-task-functional-separation.md).

## 1. Purpose

The Rule Management System keeps approved operational rules separate from conversation, memory, temporary instructions, and unfinished proposals. It answers:

- Which rule is currently authoritative?
- Is it Development or Live?
- Was it validated before activation?
- What changed, when, why, and by which session?
- Which previous value must be restored during rollback?
- Is a similar or conflicting rule already registered?

TASK records are deliberately excluded from this document and are defined in the separate [Task Management System](../02_Task_Management/task-management-system.md).

## 2. Why It Was Built

Conversation-only rule management can mix ideas, tests, obsolete instructions, and approved decisions. A later session may treat an old statement as current or promote an untested proposal accidentally.

The system therefore treats chat and memory as useful context, but not as the sole operational source of truth.

## 3. Rule Layers

- **Development:** proposed or changed rules awaiting validation.
- **Live:** approved rules permitted for operational use.
- **History:** immutable evidence of previous values, reasons, validation, promotion, rollback, and failure.
- **Safety boundary:** publication, security, privacy, and approval constraints that cannot be bypassed by an ordinary rule.

Development and Live may both exist, but execution precedence and scope must be explicit.

## 4. Rule Lifecycle

1. Register a proposal in Development.
2. Check exact duplicates, meaningful similarity, scope, conflicts, identifiers, and required fields.
3. Validate schema, dependencies, expected behavior, and regression cases.
4. Reject failures without treating them as missing or inactive.
5. Promote a validated rule by inserting or updating the linked Live rule.
6. Preserve the previous Live value in History.
7. Record reason, rule version, session, time, validation evidence, and result.
8. Deactivate, replace, expire, or roll back through explicit transitions.

## 5. Identifier Model

The reference design uses an auto-increment primary key plus:

- **Create_GUID:** creation lineage.
- **Group_GUID:** related rule group.
- **Link_GUID:** relationship across Development, Live, and History.

The integer key is for efficient storage and joins. GUIDs express operational relationships and must not be used interchangeably.

## 6. History Requirements

A useful History record should identify:

- Rule and scope
- Previous and new values
- Change reason
- Development and Live relationship
- Validation result and evidence
- Session and timestamp
- Promotion, rollback, rejection, or correction outcome
- Failure stop point and recovery reference

History is an audit mechanism, not merely a backup.

## 7. Concurrency and Ownership

Each modifying operation receives a Session GUID. Before a critical write or promotion, stored ownership must match the active session. A mismatch produces NG rather than silent overwrite.

Locks may apply at database, scope, group, rule, or promotion level. Heartbeats help detect abandoned work, but timeout alone does not prove that an external side effect did not occur.

## 8. Global Rule Functions

The shared interface may provide:

- Load active rules
- Register Development rule
- Detect duplicate, similar, and conflicting rules
- Validate Development rule
- Promote to Live
- Activate, deactivate, replace, or roll back
- Write and query History
- Acquire, verify, heartbeat, and release ownership

The interface should remain deterministic around database state. ChatGPT inference may assist classification or similarity review, but cannot silently change Live rules.

## 9. Advantages

- One authoritative rule state
- Safer experimentation before Live activation
- Traceable changes and rollback
- Reduced duplicate and conflicting rules
- Better diagnosis across sessions
- Reusable lifecycle for reports, notifications, formatting, and integrations

## 10. Disadvantages and Risks

- More modes, identifiers, transitions, validation, and storage
- Similarity detection can reject valid variants
- Incorrect scope or precedence can create hidden conflicts
- SQLite concurrent writes require careful transaction design
- Detailed History grows over time
- Documentation can drift from the implementation

## 11. Current Limitations

- Final schema and migration versioning remain under review.
- Lock granularity and timeout recovery are not finalized.
- Similarity thresholds are not finalized.
- History retention and archive rules are not finalized.
- Role and authorization separation require further design.
- Concurrent performance has not been measured.
- Public examples must remain separated from private operational rules.

## 12. Next Work

1. Finalize the rule schema and state transitions.
2. Define GUID creation and reuse contracts.
3. Define transaction and promotion boundaries.
4. Create deterministic validation and regression tests.
5. Define similarity review levels.
6. Test conflicts, rollback, timeout, and concurrent promotion.
7. Publish only sanitized implementation examples.
