# Rule Management and Work Tracking System

## 1. Overview

This document describes a rule-management and work-tracking architecture designed during the rebuild of a long-running automation workflow.

The system was not created simply to store rules. It was created to solve a broader operational problem: rules, implementation decisions, task progress, failure reasons, and recovery information were spread across conversations and temporary working contexts. When a session stopped, a task failed, or work continued later, it was difficult to answer basic questions reliably:

- What is the current valid rule?
- Was a proposed rule tested before it became active?
- What changed, when, and why?
- Did the previous task finish normally, fail, or stop midway?
- If it failed, what caused the failure?
- What must be continued next?
- Is another session already modifying the same rule?
- Can historical decisions be reconstructed without relying on memory?

The architecture therefore combines two related systems:

1. A **Rule Management System** for Development, Live, History, validation, and promotion.
2. A **Work Management System** for priority, progress, results, failures, holds, and safe continuation.

All examples in this repository are sanitized. Production data, personal information, trading data, credentials, and private operational values are intentionally excluded.

## 2. Why the System Was Built

### Problems with conversation-only rule management

During iterative development, requirements are frequently clarified or changed. Without a structured rule store:

- Old and new rules may both be treated as current.
- A partially discussed idea may be mistaken for an approved rule.
- The reason for a change may be lost.
- A new session may not know which version is authoritative.
- Similar or duplicate rules may be registered repeatedly.
- A failed operation may leave no reliable recovery point.
- Completion may be recorded without explaining what was actually done.

Memory and chat history remain useful context, but they are not sufficient as the authoritative operational database.

### Need for safe experimentation

Directly changing an active rule is risky. A proposed rule may look correct but fail when it interacts with existing rules, reports, notifications, or validation steps.

The system therefore separates:

- **Development rules**, which may be registered and validated
- **Live rules**, which are approved for operational use

A Development rule is promoted only after validation. Promotion may insert a new Live rule or update an existing linked Live rule.

### Need for work continuity

Long-running work may stop because of validation failure, missing data, database conflict, timeout, API failure, user review, session interruption, or a deliberate HOLD decision.

A work item must therefore record both its intended work and its actual result. A simple completed/not-completed flag is not enough.

## 3. Design Principles

1. **One authoritative data source** — Operational rules should be loaded from the shared rule database rather than reconstructed from memory.
2. **Development before Live** — New or changed rules should be validated in Development before promotion.
3. **Traceability** — Every important change should preserve its relationship to the originating rule and previous value.
4. **Explicit results** — Work history must show not only status, but also what was done and why a failure occurred.
5. **Safe continuation** — Interrupted work should contain enough information to resume later without repeating completed work.
6. **Duplicate prevention** — Registration should check for exact duplicates and meaningful similarity.
7. **Concurrency awareness** — Multiple sessions must not silently overwrite each other's work.
8. **Single shared interface** — Common operations should be exposed through a Global Function layer.
9. **Data minimization** — Technical design can be public while private operational data remains separate.

## 4. High-Level Architecture

The architecture consists of:

- Rule registry
- Development rule store
- Live rule store
- Rule history
- Validation and promotion service
- Similarity and duplicate checking
- Lock and session management
- Heartbeat and timeout handling
- Work queue and work history
- Global Function interface

SQLite is the current reference database, while the concepts are applicable to other relational databases.

## 5. Rule Lifecycle

### Registration

A proposed rule is registered in Development. Before insertion, the system should check exact duplicates, similar rules, conflicting active rules, required identifiers, valid mode and state, required relationships, and rule-specific constraints.

### Validation

Validation may include schema, required-field, scope, conflict, duplicate, dependency, test-execution, and expected-result checks. Failed validation must be recorded and must not be interpreted as an inactive or missing rule.

### Promotion to Live

After successful validation, promotion can:

- Insert a new Live rule
- Update the related Live rule
- Preserve the previous Live value in History
- Link Development, Live, and History records
- Record the promotion reason, session, and timestamp

Development and Live may both remain active when operationally useful. Active state is managed by mode.

### Modification and deactivation

A Live modification should preserve the old value, new value, reason, related Development rule, session, timestamp, and result. Deactivation should be explicit; inactive, invalid, deleted, expired, and replaced must not share one ambiguous state.

## 6. Identifier Model

The design uses an auto-increment integer primary key plus three GUID-based identifiers:

- **Create_GUID** identifies the creation lineage of a rule.
- **Group_GUID** groups functionally related rules.
- **Link_GUID** connects Development, Live, and History records.

The integer key supports efficient joins and compact storage. It does not express business relationships.

The semantic boundary between these GUIDs must be strictly documented. Inconsistent use would add complexity without reliable traceability.

## 7. History and Audit

History is not only a backup. A useful record should answer:

- Which rule changed?
- What were the previous and new values?
- Why was the change made?
- Which Development and Live rules are related?
- Which session performed the operation?
- Was it successful?
- If not, where did it stop?
- Can it be resumed safely?

Suggested history categories include creation, update, validation, promotion, activation, deactivation, conflict rejection, duplicate rejection, rollback, interruption recovery, and administrative correction.

Detailed history improves diagnosis but increases storage and query cost. The final retention and archive policy remains under review.

## 8. Work Management

The work-management component exists because task status alone cannot explain whether work finished normally or stopped with an error.

Core concepts include:

- Priority and priority sequence
- Registration, completion, cancellation, NG, and HOLD states
- Registration and completion timestamps
- Work description
- Result description
- Work scope
- Required-order flag
- Duplicate-prohibited flag

Equal priorities are processed in registration order.

### Work description vs. result description

The work description explains what was intended. The result description explains what actually happened.

A successful result should state what changed, what was verified, what output was produced, and whether follow-up remains.

An NG result should state where processing stopped, the direct cause, partial effects, retry safety, and the recommended continuation point.

### Status model

The minimum states are Registered, In Progress, Completed, NG, HOLD, and Cancelled. Possible future states include Waiting for Validation, Waiting for User Review, Blocked, Timed Out, Retrying, and Superseded.

## 9. Concurrency, Sessions, and Locks

Two sessions may load the same rule and attempt different updates. Without ownership verification, a later commit may silently overwrite a valid change.

Each operation therefore receives a new Session GUID. The lock record may contain session ID, target, acquisition time, last heartbeat, operation, expected owner, and timeout threshold.

Before a critical update or completion, the stored Session ID must match the active session. A mismatch produces an NG result.

### Heartbeat and recovery

A heartbeat distinguishes active long-running work from an abandoned lock. However, timeout does not prove that an original operation is safe to replace: an external action may have completed before its result was recorded.

Recovery may require idempotency checks, external side-effect verification, transaction inspection, manual review, and a recovery record linked to the original session.

### Lock granularity

Possible scopes include the whole database, rule scope, rule group, individual rule, work item, or promotion operation. A broad lock is simpler but reduces concurrency; fine-grained locking improves concurrency but increases deadlock and recovery complexity. The final choice remains under review.

## 10. Global Function Layer

The shared layer may provide operations to load active rules, register and validate Development rules, check duplicates, promote to Live, write History, manage locks and heartbeats, register work, record completion or failure, and resume pending work.

Benefits include consistent behavior, centralized validation, less duplicate implementation, easier testing, and easier migration. The risk is that one Global Function can become too broad, so internal services and commands should remain clearly separated.

## 11. Advantages

- Clear authoritative current state
- Safer Development-to-Live changes
- Better debugging through History and result descriptions
- Reliable continuation across interrupted sessions
- Auditable change relationships
- Reduced duplicate and conflicting rules
- Reusable design for reports, notifications, formatting, classification, and API workflows
- Public value for developers facing lifecycle, audit, concurrency, and recovery problems

## 12. Disadvantages and Trade-offs

- More tables, modes, GUIDs, states, locks, and operational complexity
- The management system itself requires policies for promotion, validation, timeout, retry, duplication, and retention
- Additional database activity and storage
- Risk of inconsistent identifier use
- External side-effect recovery cannot always be automated
- SQLite requires careful design for concurrent writes
- Documentation must remain synchronized with implementation

## 13. Difficulties Encountered

### Separating ideas from approved rules

Conversation contains questions, suggestions, tests, and final decisions. An explicit Development/Live distinction was needed to identify authority.

### Defining meaningful History

Old values alone were insufficient. Reasons, results, sessions, relationships, and continuation information were also needed.

### Defining completion and failure

A completion flag did not explain what was completed. A generic error did not explain the stop point, cause, partial effect, or safe retry position.

### Supporting interrupted sessions

Work may continue after the original context is unavailable. The database must carry enough state for safe continuation.

### Avoiding duplicates without rejecting valid variants

Exact duplicate checking is simple. Semantic similarity is difficult because similar wording may apply to different scopes or modes.

### Balancing Development and Live activation

Both modes may remain active, but execution precedence must not become ambiguous.

### Concurrency and stale locks

Locks require reliable ownership, heartbeat, timeout, and recovery rules.

### History size management

Detailed records are valuable for diagnosis but expensive over time.

## 14. Current Limitations

- Final schema is still being refined.
- Lock granularity is not finalized.
- Heartbeat timeout and recovery policy are not finalized.
- Similarity detection is not finalized.
- History retention and archive policy are not finalized.
- Automatic retry criteria are not finalized.
- Status transitions need a formal state model.
- Authorization and role separation are not specified.
- Migration and schema versioning are not specified.
- Concurrent performance has not been measured.
- Disaster recovery and corruption handling need test plans.
- External side-effect verification remains domain-specific.
- Public samples must remain separated from private operational data.

## 15. Next Steps

1. Finalize the logical schema.
2. Define statuses and allowed transitions.
3. Define exact GUID creation and reuse rules.
4. Specify transaction boundaries.
5. Select lock granularity.
6. Define heartbeat, timeout, and recovery behavior.
7. Define duplicate and similarity levels.
8. Define Development validation requirements.
9. Define atomic Development-to-Live promotion.
10. Define History retention and archival.
11. Create sanitized SQLite schema examples.
12. Create Delphi interfaces for the Global Function layer.
13. Create success, NG, HOLD, timeout, and recovery tests.
14. Publish unresolved questions as GitHub Issues.
15. Add versioned implementation notes as decisions change.

## 16. Questions for Other Developers

1. What lock granularity works best for a SQLite rule engine used by multiple sessions?
2. How should a stale heartbeat be distinguished from a slow but valid operation?
3. What is the safest recovery method when an external side effect may have occurred before a database result was recorded?
4. How can semantic similarity be detected without rejecting legitimate variants?
5. Should Development and Live share one table with a mode column, or use separate tables?
6. What History retention strategy preserves useful auditability without excessive growth?
7. Which status model best distinguishes NG, blocked, HOLD, cancelled, timed out, and retrying work?
8. How should promotion be versioned so rollback remains simple and reliable?

## 17. Summary

A rule system is not only a collection of conditions. A reliable operational system must manage lifecycle, validation, history, concurrency, failure, recovery, and continuation.

Work management is equally important. It provides evidence of whether work completed normally, what was done, why it failed, and where the next session should continue.

This architecture remains a work in progress. Publishing both successful decisions and unresolved limitations is intentional: the goal is to help other developers and invite informed technical feedback.
