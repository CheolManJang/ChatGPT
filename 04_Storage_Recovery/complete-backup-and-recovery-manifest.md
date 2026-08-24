# Complete Backup and Recovery Manifest

> [!NOTE]
> **Document baseline:** 2026-08-24. Reference environment: individual ChatGPT Plus account using ChatGPT web/Work without direct OpenAI API calls. Architectural principles are general; product behavior and observed limitations are specific to the tested plan, context, permissions, connected apps, and rollout state. Revalidate after material product changes.

> [!CAUTION]
> **Use at your own risk.** This material is provided for technical education and general information only, without warranties. You are responsible for evaluating, testing, securing, and legally complying with any implementation. See the [Disclaimer](../DISCLAIMER.md).


## Adoption Decision Summary

| Field | Project record |
|---|---|
| Introduced because | File-only or chat-only backups could not restore operational behavior. |
| Intended improvement | Back up rules, TASKs, History, manifests, schedules, connectors, tests, and recovery instructions. |
| Main difficulty | Hidden model state, permissions, transient context, and external side effects cannot be cloned exactly. |
| Main advantage | Recovery targets a verified working state rather than a misleading file-presence check. |
| Main disadvantage | More storage, verification, restore ordering, and recurring test cost. |
| Observed result | Complete recovery requirements are documented; exact hidden-state restoration is not claimed. |
| Current status | Development / recovery design. |
| Retest trigger | Schema, provider, permission, schedule, source, model, or system-boundary changes. |

## 1. Why a Normal Backup Is Not a Complete Recovery

Copying one chat, one database, or one master file does not recreate the complete ChatGPT operating system.

Operational state is distributed across:

- Rules
- Work status
- History
- Project instructions
- Files
- Connected sources
- Scheduled tasks
- Memory
- GitHub versions
- Gmail workflow state
- Tool permissions
- External side effects

Some of these can be backed up as files. Others must be recorded as configuration or evidence. Some, such as authorizations and transient execution context, must be recreated rather than restored.

## 2. What Is Usually Lost in Partial Backup

### Chat-only backup

May preserve messages but not:

- Current connected-file bytes
- Active scheduled-task state
- Memory controls
- Plugin authorization
- External send results
- Database locks
- Library file identity and versions
- Project source inventory
- Tool permissions
- Transient workspace state

### Database-only backup

May preserve rules and work records but not:

- Project instructions
- Scheduled prompts
- GitHub commits
- Drive file versions
- Gmail message identity
- Library artifacts
- Test fixtures
- Connector permissions

### File-only backup

May preserve content but not:

- Which file was authoritative
- The rule version that produced it
- Work status
- Source timestamps
- Active sessions
- External delivery state
- Required restore order

## 3. Recovery Objective

The realistic goal is not to recreate invisible model state. The goal is to restore a **verifiable operational state** with:

- The same approved rules
- The same known work position
- The same private source versions
- The same task definitions
- The same expected outputs
- Reauthorized tools
- Recorded external side effects
- A clear continuation point

## 4. Backup Package Contents

### A. System identity

- System name
- Backup ID
- Created timestamp
- Environment or plan reference
- Public documentation version
- Private operational version
- Backup creator
- Reason for backup

### B. Rule system

- Database schema
- Schema version
- Migration scripts
- Live rule export
- Development rule export
- Active-state map
- Create_GUID, Group_GUID, and Link_GUID relationships
- Rule priority and scope
- Validation status
- Last approved change
- Checksums

Use SQL, JSON, or another diffable export in addition to a binary database backup.

### C. Work system

- Registered work
- In-progress work
- HOLD work
- NG work
- Completed-result summaries
- Required-order relationships
- Duplicate-prevention keys
- Priority and sequence
- Safe continuation point
- Session ownership and stale-lock disposition

Do not restore an old active lock as if its process were still alive.

### D. History

- Rule changes
- Previous values
- Change reasons
- Promotion history
- Failure causes
- Recovery actions
- Publication history
- External-action evidence

### E. MENU and CMD

- Canonical identifiers
- Aliases
- Parser version
- Parameter schema
- Explanation text
- Active mode
- Parent-child menu relationships
- Confirmation requirements
- Regression tests

### F. Project instructions

- Project purpose
- Safety boundaries
- Source-of-truth order
- User approval rules
- Output requirements
- Public/private data rules
- Current known limitations

### G. Source inventory

For each important file:

- Logical role
- Provider: Library, Drive, GitHub, or local
- Stable file identity where available
- Path or private locator
- Version
- Last-modified time
- Size
- Checksum
- Required access permission
- Whether it is authoritative
- Backup copy location

Do not put private access links in the public repository.

### H. Library artifacts

- Library file identity
- Current version
- Filename
- Logical purpose
- Checksum of materialized bytes
- Exported recovery copy for critical files

Library version restore helps recover that file, but the manifest is needed to reconnect the file to the system role.

### I. Google Drive sources

- Drive file identity
- Logical role
- Export format
- Current approved version
- Modified time
- Checksum where practical
- Offline/exported recovery copy
- Verification result

For Google-native files, include an exported format suitable for recovery.

### J. Gmail workflow state

- Workflow name
- Sanitized subject pattern
- Last processed message identity
- Last processed timestamp
- Attachment inventory
- Attachment checksum after persistent save
- Draft/sent/delivery/reply status
- Related work item
- Retry safety

Do not store email credentials or private message content in public backups.

### K. Scheduled tasks

- Task title
- Complete durable prompt
- Schedule
- Timezone
- Enabled or paused state
- Connected tools required
- Source references
- Last successful run
- Last failed run
- Last processed cursor or checkpoint
- Notification behavior
- Automatic-reply boundary
- Stop conditions

Scheduled tasks should be recreated from this manifest and tested before enabling.

### L. Memory review snapshot

Do not attempt to rely on an invisible memory backup alone.

Record privately:

- Whether memory use is enabled
- Which stable categories are intentionally remembered
- Which operational details are prohibited from memory
- Last review date
- Known corrections
- Authoritative sources referenced by memory

After recovery, memory should be reviewed against Live rules rather than treated as restored authority.

### M. Connected tools and permissions

Record:

- Required connector
- Required account role
- Read or write capability needed
- Approval boundary
- Last successful access test
- Reauthorization procedure

Do not back up OAuth tokens in the recovery package.

### N. Tests and expected outputs

- Sanitized fixtures
- Expected parser results
- Expected report schema
- Failure cases
- Missing-data cases
- HOLD and NG cases
- Last-known-good output fingerprints
- End-to-end validation checklist

A restore is not complete until these tests pass.

### O. Public repository state

- GitHub repository
- Branch
- Commit SHA
- Documentation version
- Open Issues
- CHANGELOG
- Publication policy
- Release or tag where used

### P. Recovery runbook

- Restore order
- Validation commands
- Manual approval points
- Rollback point
- External-side-effect checks
- Definition of recovery completion

## 5. Recommended Backup Layers

### Layer 1: Frequent operational checkpoint

Contains:

- Live and Development exports
- Work state
- Continuation point
- Source inventory changes
- Scheduled-task checkpoint

### Layer 2: Daily or milestone snapshot

Contains:

- Database backup
- Diffable exports
- Critical Drive exports
- Critical Library artifacts
- History increment
- Test results

### Layer 3: Release checkpoint

Contains:

- Versioned documentation
- Schema and migrations
- Complete sanitized test suite
- Recovery manifest
- Approved private recovery bundle
- Git tag or release reference

## 6. Restore Order

1. Create an isolated recovery environment.
2. Verify backup manifest and checksums.
3. Restore schema and migrations.
4. Restore Live, Development, work, and History data.
5. Clear or classify stale locks.
6. Restore MENU and CMD definitions.
7. Restore project instructions and safety policy.
8. Reconnect Library and Drive files by stable identity.
9. Reauthorize connected apps; do not restore tokens.
10. Recreate scheduled tasks in a paused state.
11. Restore Gmail processing checkpoints.
12. Review memory settings and summaries.
13. Run sanitized unit and regression tests.
14. Run a read-only end-to-end test.
15. Compare with last-known-good output.
16. Obtain user approval.
17. Enable writes and scheduled tasks gradually.
18. Record the recovery result in History.

## 7. Recovery Completion Criteria

Recovery is complete only when:

- Checksums match or approved differences are documented.
- Live rule version is correct.
- No Development rule was promoted accidentally.
- Work status and continuation point are correct.
- Private master version is verified.
- MENU/CMD tests pass.
- Connected sources pass access checks.
- Scheduled tasks are recreated from the approved prompt.
- Gmail checkpoint prevents duplicate processing.
- No stale lock is active.
- Regression tests pass.
- A read-only end-to-end run matches expectations.
- The user approves resuming high-impact operation.

## 8. What Cannot Be Restored Exactly

Even with a complete manifest, the following may not be reproducible byte-for-byte or behavior-for-behavior:

- Hidden model reasoning
- Transient cloud workspace state
- Expired temporary links
- Exact model behavior after product updates
- Deleted external messages or files
- Unrecorded external side effects
- Old connector permissions no longer available
- Conversation context that was never exported
- Memory transformations that were never reviewed
- Tool results whose source changed

The system therefore restores approved state and verifies behavior rather than claiming perfect historical replay.

## 9. Public and Private Backup Separation

### Public GitHub

- Backup schema
- Generic manifest template
- Recovery procedure
- Sanitized test examples
- Known limitations

### Private recovery bundle

- Actual rules
- Actual work state
- Actual source identities
- Actual scheduled prompts
- Actual Gmail checkpoints
- Private database backups
- Raw operational data

### Secret manager or reauthorization

- Credentials
- OAuth tokens
- Passwords
- Recovery codes

Secrets must not be stored in GitHub, Library documents, or ordinary Drive backups unless a dedicated encrypted secret-management policy explicitly allows it.

## 10. Summary

A complete recovery requires more than copying files.

The system must back up rules, work, History, source identities, task definitions, project instructions, checkpoints, tests, and recovery order.

The correct recovery target is a verified operational state, not an impossible promise to reproduce every hidden or transient part of ChatGPT.
