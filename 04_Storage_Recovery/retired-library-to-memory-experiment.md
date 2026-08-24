# Retired Experiment: Loading Library Rules into ChatGPT Memory

> [!NOTE]
> **Document baseline:** 2026-08-24. Reference environment: individual ChatGPT Plus account using ChatGPT web/Work without direct OpenAI API calls. This is a retrospective on an abandoned project experiment, not a claim about every account or future product behavior.

> [!CAUTION]
> **Use at your own risk.** This document contains sanitized architectural lessons only. It does not disclose production rules, private memory contents, prompts, raw data, financial algorithms, credentials, or personal information. See the [Disclaimer](../DISCLAIMER.md).

## 1. Experiment Status

**Status: Retired / not approved for authoritative operation.**

The project experimented with moving or summarizing frequently used material from a registered ChatGPT Library source into saved memory so ChatGPT could recall operating context more quickly.

The approach was eventually abandoned as an authoritative rule mechanism. Saved memory may still be useful for lightweight preferences or reminders, but it is not used as the sole source of truth for operational rules, TASK state, report masters, or recovery.

## 2. Original Intent

The experiment attempted to solve practical latency and continuity problems:

- Avoid repeatedly searching and opening the same Library files
- Reduce startup time in a new conversation
- Make frequently used rules available without a full source reload
- Preserve high-level operating context across conversations
- Reduce repeated user explanations
- Allow ChatGPT to begin work with a familiar vocabulary and workflow
- Continue work when the original conversation was unavailable

The hypothesis was simple: if stable rules were remembered, work could start faster than retrieving and parsing a file for every request.

## 3. Intended Architecture

The proposed flow was:

1. Keep the detailed source in Library.
2. Extract selected stable rules or summaries.
3. Store those items in ChatGPT memory.
4. Recall memory at the beginning of a later conversation.
5. Use Library only when detailed evidence or recovery was needed.
6. Update memory when the Library source changed.

This appeared efficient, but step 6 required reliable synchronization, identity, versioning, and conflict resolution that saved memory was not designed to provide as a transactional rule database.

## 4. Initial Benefits Observed

The experiment showed some practical advantages:

- Faster conversational recall of familiar terms
- Less repeated explanation for stable user preferences
- Easier continuation of high-level discussions
- Lower apparent startup effort for simple questions
- Useful reminders about recurring workflow boundaries
- More natural interaction when no exact operational value was required

These benefits were real, but they applied mainly to orientation and personalization—not deterministic execution.

## 5. Problems Encountered

### No reliable version identity

A remembered summary did not provide the same version identity, content hash, schema, or source metadata as the registered Library file.

It was difficult to prove:

- Which source version created the memory
- Whether the memory was updated after a source change
- Whether only part of the rule set changed
- Whether a later correction replaced the earlier interpretation
- Whether the active context recalled the same memory

### Synchronization drift

Library content and saved memory could diverge.

Examples of drift included:

- The Library source changed but the remembered summary remained old.
- A conversational correction affected the current chat but not the saved memory.
- A remembered statement was generalized more broadly than the source intended.
- Similar rules from different scopes appeared to conflict.
- A later session could not prove which representation was authoritative.

### Summarization loss

Moving a detailed source into memory normally required compression or summarization. That could remove:

- Exceptions
- Preconditions
- Scope
- Priority
- Version information
- Validation evidence
- Failure handling
- Relationships to other rules
- Exact continuation requirements

A shorter representation was faster to recall but less safe to execute.

### Non-deterministic recall

Saved memory could influence responses without behaving like a deterministic database query. The project could not rely on:

- Exact field retrieval
- Complete rule enumeration
- Stable ordering
- Atomic replacement
- Transaction rollback
- Guaranteed recall in every context
- A formal conflict-resolution result

### Rule mixing

Preferences, high-level context, unfinished ideas, and operational rules could become difficult to distinguish.

A remembered item could be interpreted as:

- A user preference
- A current Live rule
- A historical fact
- A proposed Development rule
- A temporary conversational instruction

This ambiguity was dangerous for automated or repeatable workflows.

### Weak auditability

Memory did not provide the required operational History:

- Previous value
- New value
- Change reason
- Approver
- Validation result
- Promotion evidence
- Session ownership
- Failure point
- Rollback record

Without that evidence, a wrong result could not be diagnosed reliably.

### Incomplete backup and recovery

Backing up Library files did not back up every remembered interpretation or transient influence. Conversely, reviewing saved memory did not reconstruct:

- Library file identity and version
- Chat context
- TASK progress
- Scheduled prompts
- Connector permissions
- Tool results
- Hidden model state
- External side effects

A full return to the previous behavior could not be guaranteed.

### Context and product boundaries

Memory behavior could vary with settings, account, plan, context, project, workspace policy, and product changes. A result observed in one chat did not prove identical behavior in ChatGPT Work, a Project, or a scheduled task.

## 6. Retired OPEN/CLOSE Synchronization Commands

As the synchronization problem became visible, the project experimented with explicit `OPEN` and `CLOSE`-style commands.

### Intended purpose

The commands were intended to create a clear user-visible boundary around a work session:

- `OPEN`: begin a controlled session, identify the expected source, and prepare the working context.
- `CLOSE`: finish the session, record results, synchronize approved changes, and leave a known continuation state.

The exact syntax and internal behavior changed during experimentation. The important design intent was to prevent an undefined conversational session from silently reading, changing, or carrying forward operational rules.

### Why the idea appeared useful

An explicit start and end seemed capable of providing:

- A visible synchronization point
- A place to compare Library and remembered state
- A place to acquire and release session ownership
- A place to save changes and History
- A clear boundary for continuation in another chat
- A way to stop use of stale context after work completed

### Problems discovered

The commands were conversational instructions, not product-level transaction hooks. They could request behavior, but could not guarantee that ChatGPT had actually:

- Loaded every intended memory item
- Excluded stale or conflicting memory
- Replaced memory atomically
- Persisted every approved change
- Removed temporary context
- Synchronized another Chat, Project, Work session, or scheduled task
- Prevented a concurrent session from using an older state
- Completed `CLOSE` after a timeout, interruption, or context-length limit

An `OPEN` response could therefore look successful while the underlying context remained incomplete. A `CLOSE` response could look complete while Library, memory, TASK state, History, or another session remained unsynchronized.

### Failure scenarios

- `OPEN` used a remembered summary older than the registered source.
- A Library change occurred after `OPEN` but before `CLOSE`.
- The conversation ended before `CLOSE`.
- `CLOSE` recorded a summary but not every required rule or result.
- Two sessions were opened against different source versions.
- A user interpreted “closed” as proof that memory was cleared or synchronized.
- A later session resumed without being able to verify the previous close result.

### Retirement decision

`OPEN` and `CLOSE` were retired as Memory synchronization commands because they created a false sense of transactional control over a product feature that did not expose the required deterministic guarantees.

The names must not be documented as proof that ChatGPT Memory can be opened, locked, committed, synchronized, or closed like a database.

### What replaced them

The useful intent was preserved at the deterministic system layer:

- Rule Engine session acquisition and version verification
- Task Manager execution attempts and continuation records
- Explicit source fetch and validation
- Separate `TASK.START`, `TASK.COMPLETE`, `TASK.NG`, and `TASK.HOLD` operations
- Rule validation, promotion, and History commands
- Session GUID, heartbeat, lock ownership, idempotency, and reconciliation
- A final result record that states what was actually verified

A user-facing session command may later coordinate these functions, but it must report the verified state of the underlying stores. It must never claim that ChatGPT Memory itself was transactionally opened or closed.

## 7. Why the Approach Was Retired

The approach optimized apparent speed by weakening certainty.

For casual continuity, this trade-off can be acceptable. For operational rules, report masters, TASK state, automated actions, and recovery, it was not.

The experiment was retired because it could not guarantee:

- One authoritative version
- Complete synchronization
- Deterministic loading
- Exact scope and priority
- Reliable conflict handling
- Transactional updates
- Auditable History
- Complete backup and restoration
- Identical behavior across contexts

The decision was not that memory has no value. The decision was that memory must not be promoted into an authoritative operational database.

## 8. Current Approved Boundary

| Information type | Approved authoritative source | Memory role |
|---|---|---|
| Live operational rules | Registered rule store | Optional high-level reminder only |
| Development rules | Development rule store | Not authoritative |
| TASK state and results | Task Manager | Not authoritative |
| Yearly Report master | Registered Library master | Never replace source |
| G-Yearly original | Registered Google Drive original | Never replace source |
| History and audit | Domain History records | Not authoritative |
| User preferences | Approved preference source or user confirmation | Useful when current and non-critical |
| General vocabulary | Documentation | Useful orientation |
| Recovery | Verified backup and recovery manifest | Cannot provide full recovery |

## 9. Replacement Architecture

The safer approach is:

1. Keep authoritative data in its registered source.
2. Store source identity, version, and validation evidence.
3. Load the required source for each material operation.
4. Use memory only to locate the source or recall non-critical preferences.
5. Compare recalled context with the authoritative source.
6. Stop with NG or HOLD when source identity or freshness cannot be proven.
7. Record rule and TASK changes in their domain History.
8. Back up files, databases, manifests, tasks, rules, schedules, and recovery instructions.
9. Revalidate after product, permission, or context changes.

Memory can say, “The project uses a registered rule source.” It should not silently supply the complete Live rule set.

## 10. Speed Versus Correctness

The experiment exposed a recurring engineering trade-off:

| Optimization | Benefit | Risk |
|---|---|---|
| Recall from memory | Faster apparent startup | Stale, incomplete, or ambiguous state |
| Reload authoritative source | More latency | Stronger identity, completeness, and auditability |
| Cache verified extraction | Faster repeated use | Requires version binding and invalidation |
| Use concise summaries | Lower context cost | Lost exceptions and relationships |

The approved optimization is a **version-bound verified cache**, not unconstrained remembered operational content.

A cache must include source identity, source version or hash, extraction version, creation time, validation result, scope, and invalidation rule. If those fields are unavailable, the source must be fetched again.

## 11. Diagnostic Procedure for Memory-Related Drift

When behavior becomes inconsistent:

1. Stop material automation and external actions.
2. Identify the expected authoritative source.
3. Record the observed incorrect or conflicting behavior.
4. Review current saved-memory settings and relevant memory summaries.
5. Compare memory-derived statements with Live rules, TASK state, and registered sources.
6. Classify each statement as authoritative, contextual, historical, proposed, or incorrect.
7. Remove or correct misleading memory only through the supported product controls.
8. Reload the authoritative source in an isolated test.
9. run regression scenarios and compare outputs.
10. Resume only after rule version, source identity, and expected behavior are verified.
11. Record the cause, correction, validation, and continuation point.

Deleting memory alone does not restore the system. The authoritative source and operational state must also be verified.

## 12. Lessons for Developers

- Fast recall is not the same as correct state retrieval.
- Personalization memory is not a transactional configuration store.
- Summaries must not replace full rule contracts.
- Every cache requires identity, version, invalidation, and validation.
- Context, memory, files, databases, and external services have different consistency models.
- A probabilistic assistant needs a deterministic operating shell.
- Failure and abandoned approaches should be documented, not hidden.
- Recovery means validated operational behavior, not merely restoring a file.

## 13. What May Be Published

This retrospective may publish:

- The architectural intent
- High-level benefits
- Failure categories
- The retirement decision
- The replacement boundary
- Sanitized diagnostic and recovery procedures

It must not publish:

- Actual memory contents
- Private prompts
- Complete operational rules
- Report master data
- Core financial algorithms
- Real symbols, prices, holdings, accounts, or recipients
- Credentials or connected-app authorization details
- Screenshots that reveal private memory summaries

## 14. Current Conclusion

Saved memory is useful for human-friendly continuity and non-critical preferences. It is fast because it can reduce repeated explanation, but that speed comes without the versioning, transactions, completeness, and audit guarantees required for an operational rule system.

The experiment therefore remains documented as **Retired**. Future reconsideration would require official product capabilities that provide deterministic scope, version identity, controlled synchronization, audit history, backup, and restore behavior suitable for the specific workflow.
