# Multi-Session Synchronization and Locking Lessons

> [!NOTE]
> **Document baseline:** 2026-08-24. Reference environment: individual ChatGPT Plus account using multiple Chat, Project, Work, or scheduled-task contexts without direct OpenAI API calls. This is sanitized architecture and an experiment retrospective. Behavior may change by account, plan, permissions, context, and product rollout.

> [!CAUTION]
> **Use at your own risk.** This document does not expose production rules, private TASKs, memory contents, prompts, raw data, credentials, or financial algorithms. See the [Disclaimer](../DISCLAIMER.md).

## 1. Problem

A user can open a new conversation while another conversation is still working. Both sessions may believe they are operating on the same system, while each actually holds different:

- Conversational context
- Recalled memory
- Loaded source version
- Rule snapshot
- TASK snapshot
- File copy
- Validation evidence
- Tool and connector state
- Continuation point

Without synchronization and ownership checks, simultaneous work can create a split-brain condition: two sessions independently make valid-looking decisions from different state.

## 2. Why Conversational Continuity Was Insufficient

A new chat cannot safely assume that it inherited:

- The previous chat's complete context
- The same Memory interpretation
- The same loaded Library or Drive file
- An active database transaction
- A valid lock
- The latest TASK result
- An external side effect
- The previous session's unrecorded reasoning

A statement such as “continue” is useful to the user, but it is not a concurrency protocol.

## 3. Failure Scenarios Observed or Identified

### Same rule, different versions

Session A loads rule version A. Session B later loads version B. Session A completes after B and silently overwrites or reports using the older state.

### Same TASK, two owners

Two chats start the same pending TASK. Both send a message, create a file, update a rule, or mark completion.

### Source changes during execution

One session updates a Library or Drive source while another session continues processing the earlier version.

### OPEN without shared ownership

The retired `OPEN` command could declare a session started, but another chat could also issue `OPEN`. A conversational command could not enforce exclusive ownership across contexts.

### CLOSE after another session changed state

A delayed `CLOSE` could write a summary or result based on a stale snapshot after another session had already advanced the work.

### Interrupted owner

A session acquires ownership and then stops because of context limit, timeout, tool failure, browser closure, or user interruption. The next session cannot assume the previous operation had no side effects.

### Memory synchronization illusion

Both sessions may receive similar remembered context, but similarity is not proof of identical, complete, or current state.

## 4. Critical Boundary: Chat Lock vs. Data Lock

ChatGPT conversations themselves are not treated as lockable database sessions.

The project cannot guarantee a product-level lock on:

- A chat window
- Saved Memory
- Hidden model context
- A Project's transient state
- A Work execution context
- A scheduled task's hidden runtime state

Locking must therefore occur in deterministic state owned by the system:

- Rule Engine database
- Task Manager database
- Registered source manifest
- File or package version record
- External action idempotency register
- Delivery attempt record

A message saying “locked” is not evidence. The lock record must exist in the authoritative store and be verified before a critical action.

## 5. Session Identity

Every modifying attempt receives a new Session GUID.

A session record should contain:

- Session GUID
- Context type: Chat, Project, Work, or scheduled task
- Owning subsystem
- Target rule, TASK, source, or delivery package
- Expected source version or hash
- Lock acquisition time
- Last heartbeat
- Lease or timeout policy
- Current operation
- Correlation and idempotency keys
- Status and stop reason
- Continuation reference

The Session GUID identifies an attempt, not a person and not permanent authority.

## 6. Lock Ownership Model

Before changing state:

1. Read the authoritative current version.
2. Request ownership for the exact target and operation.
3. Store the Session GUID, expected version, and acquisition time.
4. Confirm that no conflicting owner is active.
5. Re-read the lock and version after acquisition.
6. Perform bounded work.
7. Verify ownership again immediately before commit.
8. Commit only when Session GUID and expected version still match.
9. Record result and release or expire ownership.

A mismatch produces NG or HOLD. It must not be resolved by silently overwriting the newer state.

## 7. Lock Scope

Different subsystems require separate lock namespaces.

| Target | Possible scope | Main risk |
|---|---|---|
| Rule change | Rule, group, scope, or promotion | conflicting Live versions |
| TASK execution | TASK or execution attempt | duplicate work or result |
| Source refresh | source identity and version | mixed old/new population |
| Report run | report run GUID | duplicate or incomplete report |
| Email delivery | delivery package and recipient reference | duplicate send |
| Backup | backup set or manifest | partial inconsistent package |

A global database lock is simpler but reduces concurrency. Fine-grained locks improve throughput but require dependency ordering and deadlock prevention.

## 8. Heartbeat and Lease

A heartbeat shows that a session recently reported activity. It does not prove:

- The session is healthy
- The operation is progressing
- An external action did not already occur
- Retry is safe
- The browser or model still has the same context

A stale heartbeat permits investigation, not automatic takeover.

Before takeover:

1. Inspect the target's current version.
2. Inspect the previous attempt and transaction state.
3. Check files, messages, APIs, and other side effects.
4. Determine idempotency and retry safety.
5. Record the takeover reason.
6. Create a new Session GUID.
7. Link the recovery attempt to the abandoned attempt.

## 9. Optimistic Version Check

Exclusive locking alone is insufficient. Every critical write should also compare the version read at start with the current version at commit.

Example logical fields:

- `Expected_Version`
- `Current_Version`
- `Source_Hash`
- `Owner_Session_GUID`
- `Updated_At`

The update succeeds only when the expected version and owner still match. Otherwise the session stops and reloads current state.

## 10. Synchronization Between Sessions

Sessions do not synchronize directly through conversation.

They coordinate through authoritative records:

- Rule version and History
- TASK state and execution attempts
- Source identity, hash, and manifest
- Report run GUID and validation record
- Delivery idempotency record
- Backup manifest
- Correlation events and results

A new session must acquire current state from those records. It must not ask another chat's remembered summary to serve as the synchronization source.

## 11. Cross-Database Coordination

Rule Engine and Task Manager remain functionally separate.

When one TASK requests a rule change:

1. Task Manager records the request and correlation ID.
2. Rule Engine acquires its own rule lock.
3. Rule Engine validates and commits its own state.
4. Rule Engine returns an immutable result.
5. Task Manager acquires its TASK ownership and records the result.
6. If interrupted between commits, reconciliation checks both stores.

One broad function must not pretend that separate databases and external services committed atomically.

## 12. External Side Effects

Database rollback cannot unsend an email, undo every API action, remove every copied file, or reverse a provider operation.

Before repeating an external action, verify:

- Idempotency key
- Previous provider response
- Message, file, or report identity
- Sent or created evidence
- Current expected version
- Whether the result was recorded late
- Whether another session already completed the action

Unknown side effects produce HOLD or NG with “side effect uncertain.”

## 13. Recovery from Split-Brain

When two sessions have modified or acted on the same target:

1. Stop both from further material actions.
2. Capture their Session GUIDs, versions, evidence, and side effects.
3. Identify the authoritative store and latest valid committed version.
4. Do not select a winner based only on the later conversation timestamp.
5. Compare validation, ownership, and external effects.
6. Preserve both attempts in History.
7. Decide whether to accept, reverse, supersede, or manually merge.
8. Create a reconciliation TASK.
9. Run regression and completeness checks.
10. Resume under a new Session GUID and recorded continuation point.

## 14. What Was Retired

The following assumptions were retired:

- A new chat automatically inherits the correct working state.
- Saved Memory synchronizes concurrent sessions.
- `OPEN` alone creates exclusive ownership.
- `CLOSE` proves all stores and contexts were synchronized.
- A stale heartbeat means retry is safe.
- A chat completion message proves database commit or external delivery.
- The newest conversational statement always wins.

## 15. Current Approved Design

- Separate `rules.db` and `tasks.db`
- Separate lock namespaces and state machines
- New Session GUID per attempt
- Expected-version comparison before commit
- Registered source identity and hash
- Heartbeat as evidence only, not proof of safety
- Bounded lease and explicit takeover procedure
- Idempotency for external actions
- NG and HOLD for ownership or side-effect ambiguity
- History with stop point, cause, partial effects, and continuation
- Reconciliation rather than silent last-write-wins
- Memory as orientation only

## 16. Implementation Status

The public architecture is defined. This document does not claim that every private operational database, function, lock, or migration has already been implemented and verified.

Functional completion requires:

- Independent Rule and TASK schemas
- Atomic updates inside each owning database
- Version checks
- Lock and heartbeat tests
- Concurrent-session simulation
- Timeout and takeover tests
- Duplicate external-action tests
- Backup and restore tests
- Split-brain reconciliation tests
- Migration rollback evidence

## 17. Lessons for Developers

- Multiple chats are multiple potentially inconsistent workers.
- Natural-language continuity is not shared transactional state.
- Lock the authoritative data, not the conversation.
- Combine ownership locks with optimistic version checks.
- Heartbeat timeout requires investigation before takeover.
- External actions require idempotency and evidence.
- Every interrupted operation needs an explicit continuation point.
- Concurrency failures should be preserved and studied, not overwritten.
