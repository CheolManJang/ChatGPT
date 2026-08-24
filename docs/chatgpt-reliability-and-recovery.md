# ChatGPT Reliability, Rule Drift, and Recovery Procedure

> [!NOTE]
> **Document baseline:** 2026-08-24. Reference environment: individual ChatGPT Plus account using ChatGPT web/Work without direct OpenAI API calls. Architectural principles are general; product behavior and observed limitations are specific to the tested plan, context, permissions, connected apps, and rollout state. Revalidate after material product changes.

## 1. Purpose

A ChatGPT-centered operating system requires explicit reliability controls.

ChatGPT can work successfully for many sessions and then produce an unexpected result. This does not necessarily mean that one component is “broken.” Behavior may change because the active context, remembered information, source files, connected tools, task prompt, model behavior, or rule version is different.

The system must therefore be designed under this principle:

> ChatGPT memory and conversation context are useful recall layers, but they are not the authoritative source for rules that must always apply.

The official ChatGPT memory documentation recommends keeping required guidance in durable project documentation and treating memory as helpful recall rather than the only source of mandatory rules.

## 2. Common Failure Symptoms

Possible signs of rule drift or context conflict include:

- A previously stable report suddenly changes format.
- An old threshold or field reappears.
- A Development rule is treated as Live.
- A completed task is repeated.
- A HOLD task is executed.
- An excluded item returns to the output.
- Missing data is interpreted as a negative result.
- MENU or CMD input resolves differently.
- A new session does not continue from the expected point.
- ChatGPT says an operation succeeded without verifying the external result.
- A connected file appears available but an older version is used.
- Two correct rules conflict because their priority is undefined.

## 3. Why Previously Working Behavior Can Change

### Conversation context

A long conversation can contain obsolete instructions, experiments, corrections, and final decisions. If authority is not explicit, the wrong statement may be selected.

### Saved memory

Memory may contain a useful summary that later becomes incomplete or outdated. Memory is not guaranteed to update immediately, and it should not be the only location for mandatory operating rules.

### Rule duplication

Similar rules may exist in different scopes, modes, or versions. Without precedence, both may appear valid.

### Stale connected data

A Google Drive file or other connected source may be delayed, inaccessible, or older than expected.

### Scheduled-task prompt drift

A scheduled task uses its saved prompt and available context. If the Live rule changes but the task prompt still embeds an older rule, results can diverge.

### Model and product behavior

Model behavior, available tools, account capabilities, and product features may change. A workflow must validate output instead of assuming identical behavior forever.

### Ambiguous command input

A short number or attached description may resolve to the wrong MENU or CMD if namespace and context are unclear.

### Partial external failure

A report may be generated while file access, data retrieval, sending, or delivery fails later. Treating the whole workflow as one status hides the real failure.

## 4. Required Foundation Before Automation

The following work should be completed before relying on scheduled or repeated operation.

### 4.1 Define the authoritative sources

Use a clear authority order:

1. Safety and privacy restrictions
2. User-approved Live rules
3. Current work state and required-order constraints
4. Verified private master data
5. Versioned project documentation
6. Scheduled-task prompt
7. Saved memory
8. Current conversational hints

A lower source must not silently override a higher source.

### 4.2 Separate Development and Live

- New ideas enter Development.
- Validation occurs before activation.
- Only approved rules become Live.
- Previous Live values remain in History.
- Development and Live identifiers remain traceable.
- An unvalidated memory or chat statement cannot directly replace Live.

### 4.3 Define status and result semantics

Define Registered, In Progress, Completed, NG, HOLD, and Cancelled before automation.

A status without a result description is insufficient.

### 4.4 Define a fixed input and output contract

For every recurring workflow, define:

- Required source
- Required fields
- Validation rules
- Output schema
- Sorting
- Terminal states
- Failure behavior
- Approval boundary
- Definition of done

### 4.5 Test manually first

Run the complete prompt and workflow manually before scheduling it. Review multiple representative cases, including failure cases.

### 4.6 Create durable checkpoints

Keep versioned documents for:

- Approved rules
- Current work
- Recent changes
- Known failures
- Next continuation point
- Test cases and expected results

## 5. Memory Settings and Review

The attached settings example shows the ChatGPT memory area, including the memory toggle and memory-summary management.

Use memory for stable preferences and useful continuity, but not as the final authority for operational rules.

### What memory is suitable for

- Stable communication preferences
- Long-term project purpose
- General terminology
- High-level user preferences
- A pointer to the authoritative project source

### What memory should not be trusted to hold alone

- Exact production rules
- Current thresholds
- Current task state
- Credentials
- Private raw data
- Complete database schemas
- Exact report populations
- Destructive-action authorization
- Temporary test rules

### Memory review procedure

1. Open ChatGPT Settings.
2. Open the personalization or memory section available to the account.
3. Confirm whether memory use is enabled.
4. Open memory management.
5. Review saved summaries for outdated, duplicated, ambiguous, or private entries.
6. Compare each operational memory with the current approved Live documentation.
7. Remove or correct only the entries confirmed to be wrong or no longer appropriate.
8. Do not delete all memory as the first troubleshooting step.
9. Start an isolated test with only the authoritative rules and approved source.
10. Compare the isolated result with the failing result.
11. Record the diagnosis and correction in History.

Turning memory off for an isolated diagnostic test can help determine whether recalled context contributes to the problem, but it does not repair the underlying Live rule or source data.

## 6. Rule-Conflict Diagnostic Procedure

### Step 1: Freeze high-impact writes

Pause promotion, publication, sending, deletion, and master-data updates until the conflict is understood.

### Step 2: Capture the failure

Record:

- Original input
- Actual output
- Expected output
- Session ID
- Time
- Rule version
- Source-file version
- Scheduled-task identity if relevant
- Connected-tool results
- Last known successful result

### Step 3: Identify the earliest divergence

Separate the workflow into stages and find the first incorrect stage:

- Input parsing
- MENU/CMD resolution
- Rule loading
- Master loading
- External data acquisition
- Classification
- Formatting
- Validation
- Sending
- Delivery
- Reply handling

### Step 4: Inventory every possible rule source

Check:

- Live rule database
- Development rules
- History
- Work record
- README and detailed documents
- Scheduled-task prompt
- Saved memory
- Current chat
- Connected files
- Hard-coded program defaults

### Step 5: Apply authority and recency

For conflicting sources:

1. Reject unauthorized sources.
2. Prefer approved Live over Development.
3. Verify version and timestamp.
4. Check scope.
5. Check Active state.
6. Check whether a later History record superseded it.
7. Ask for user approval when the conflict changes a material decision.

### Step 6: Reproduce with a minimal test

Use fictional or sanitized inputs. Disable unrelated modules and reproduce only the disputed behavior.

### Step 7: Repair in Development

Do not patch Live directly unless the emergency procedure explicitly permits it.

- Correct the rule or prompt in Development.
- Add a regression test.
- Validate expected and failure cases.
- Review side effects.
- Promote only after approval.

### Step 8: Verify the complete workflow

A local fix is not sufficient. Re-run all dependent stages and confirm the final external result.

### Step 9: Record the result

History must state:

- Root cause
- Conflicting sources
- Chosen authority
- Previous and new values
- Tests performed
- Promotion result
- Remaining risks
- Safe continuation point

## 7. “It Suddenly Became Strange” Recovery Checklist

1. Stop automatic external writes.
2. Confirm whether the problem is reproducible.
3. Compare with the last known successful version.
4. Check current Live and Development rules.
5. Review memory summaries for stale or conflicting context.
6. Verify the connected master file version.
7. Inspect the scheduled-task prompt.
8. Confirm MENU/CMD parsing.
9. Check tool and connector errors.
10. Identify the earliest divergent stage.
11. Repair in Development.
12. Add a regression case.
13. Re-run end to end.
14. Promote to Live.
15. Resume automation only after repeated success.

## 8. Preventive Controls

### Rule controls

- Unique canonical rule identity
- Scope and mode
- Active state
- Version
- Effective date
- Priority
- Conflict set
- Validation status
- Approver
- Previous-value History

### Work controls

- Required order
- Duplicate prevention
- Session ownership
- Heartbeat
- Timeout
- NG reason
- HOLD reason
- Continuation point

### Output controls

- Fixed schema
- Required fields
- Population counts
- Source timestamps
- Invariant checks
- Explicit “unverified” state
- No guessed values
- Comparison with representative expected outputs

### Automation controls

- Manual test before scheduling
- Narrow permissions
- First-run review
- Quiet behavior when nothing changed
- Separate generate and send approval where appropriate
- Automatic pause after repeated NG results
- Versioned durable task prompts

## 9. Limits of Memory Management

Reviewing or correcting memory can solve some context problems, but it cannot:

- Repair a wrong Live rule
- Restore a missing Drive file
- Guarantee current external data
- Fix a connector permission failure
- Prove email delivery
- Resolve ambiguous command definitions
- Replace database History
- Guarantee deterministic output

Memory management is one diagnostic layer, not the entire recovery system.

## 10. Current Project Application

This project applies the reliability model through:

- Development and Live rule separation
- Create_GUID, Group_GUID, and Link_GUID
- Work status and result descriptions
- Session GUID and lock ownership
- Heartbeat and timeout design
- Private Google Drive master sources
- Versioned GitHub technical documentation
- Gmail stages separated from report generation
- Fixed MENU/CMD identities
- Full-population completeness checks
- Explicit NG and HOLD results

The remaining work is to complete repeated end-to-end regression tests and automatic recovery boundaries.

## 11. Official Product References

- [Memories](https://learn.chatgpt.com/docs/customization/memories)
- [Settings](https://learn.chatgpt.com/docs/reference/settings)
- [Long-running work](https://learn.chatgpt.com/docs/long-running-work)
- [Scheduled tasks](https://learn.chatgpt.com/docs/automations)
- [Iterate on difficult problems](https://learn.chatgpt.com/use-cases/iterate-on-difficult-problems)

## 12. Summary

Reliable ChatGPT application requires more than a good prompt.

It requires authoritative sources, Development/Live separation, versioning, work results, source verification, memory review, regression tests, and explicit recovery procedures.

The safest design assumes that context can drift and makes drift detectable, diagnosable, and recoverable.
