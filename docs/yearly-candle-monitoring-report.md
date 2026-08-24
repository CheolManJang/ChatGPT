# Yearly-Candle Monitoring Report System

> [!NOTE]
> **Document baseline:** 2026-08-24. Reference environment: individual ChatGPT Plus account using ChatGPT web/Work without direct OpenAI API calls. Architectural principles are general; product behavior and observed limitations are specific to the tested plan, context, permissions, connected apps, and rollout state. Revalidate after material product changes.

## 1. Overview

This document describes a real applied-technology use case built around ChatGPT Plus without directly using the OpenAI API: a yearly-candle-based market monitoring and reporting workflow.

The purpose of the project is not to publish investment recommendations or private trading data. The public value is the technical architecture:

- Converting a user-defined analytical method into persistent rules
- Maintaining a private master dataset outside the chat
- Performing full-population validation rather than partial sampling
- Combining multiple market-session sources
- Distinguishing missing data from a negative result
- Producing a fixed-format report
- Delivering it through a connected communication channel
- Recording normal, NG, HOLD, and continuation states
- Continuing the work across different ChatGPT sessions

All market symbols, prices, holdings, recipients, and credentials are excluded from the public repository.

## 2. Why the System Was Built

### Repeated manual analysis

The original workflow required the user to repeatedly inspect long-term yearly charts, compare current prices with preselected monitoring levels, exclude held positions, check listing and suspension risks, and format the results manually.

This created several problems:

- The same rules had to be explained repeatedly.
- A new chat could apply an older or incomplete rule.
- Some symbols could be omitted accidentally.
- A failed market-data lookup could be mistaken for “no matching symbols.”
- Report formatting changed between runs.
- Delivery success could be confused with report-generation success.
- The private master list was not guaranteed to be the latest copy.
- It was difficult to know where an interrupted run should continue.

### Need for an authoritative workflow

The project therefore moved from a conversational checklist toward a rule-driven reporting system with:

- An approved Live rule set
- Development rules for proposed changes
- A private master dataset
- A deterministic validation pipeline
- A fixed report schema
- Work and result records
- Versioned public technical documentation

## 3. Project Goals

The system aims to:

1. Inspect every eligible master item, not a sample.
2. Use the correct market-session value for each item.
3. Distinguish “not reached” from “could not verify.”
4. Exclude private held positions from the public-facing monitoring result.
5. Detect items within a configurable proximity threshold.
6. Include required quality, market-capitalization, dividend, and risk fields.
7. Highlight suspension or critical status clearly.
8. Produce the same approved format every run.
9. Send a report even when the result is explicitly “none,” if the rule requires it.
10. Record generation, validation, delivery, and reply handling separately.
11. Continue safely after interruption.
12. Keep private market data out of the public GitHub repository.

## 4. Analytical Concept

The private analytical method uses long-term yearly-candle structures to define monitoring zones and staged levels.

The public implementation does not disclose real symbols or operational prices. It describes only the generic model:

- One or more pre-approved monitoring levels
- One or more staged target or exit levels
- Long-term box or support-zone qualification
- Exclusion of persistently declining structures
- Optional confirmation from monthly, weekly, or daily charts
- Risk screening before adding a new master item
- A proximity calculation between the decision price and the monitoring level

A conceptual proximity formula is:

```text
proximity_percent = (decision_price / monitoring_price - 1) * 100
```

A configurable threshold classifies items that are above but close to the monitoring level.

## 5. Private Master Dataset

The master dataset is the authoritative list of monitored items and their approved rule values.

It may contain private fields such as:

- Symbol identity
- Monitoring levels
- Staged target levels
- Classification
- Quality grade
- Market-capitalization review
- Dividend information
- Risk status
- Holding or exclusion state
- Last validation metadata

This dataset belongs in private connected storage, not public GitHub.

### Current source boundary as of 2026-08-24

This document describes the **Yearly-Candle Monitoring Report**, whose current authoritative master is the registered ChatGPT Library source.

The separate **G-Yearly Report** uses its Google Drive original as the execution source.

Google Drive also provides:

- Approved raw-data sharing
- Verified backups of Library-managed systems
- Recovery packages
- User-managed source files for systems explicitly assigned to Drive

These source roles must not be merged or silently substituted.

Before a report run, the workflow should verify:

- Correct file identity
- Latest available version
- Expected schema
- Record count
- Required fields
- Duplicate symbols
- Update timestamp where available

If the latest master cannot be verified, the report must stop or return NG. It must not silently use a stale copy.

## 6. End-to-End Workflow

### Stage 1: Start

The workflow begins through:

- A user command
- A MENU or CMD entry
- A scheduled task

A new work record and Session GUID are created.

### Stage 2: Load rules

The system loads the current Live rules, including:

- Eligibility rules
- Proximity threshold
- Sorting
- Required report fields
- Exclusion rules
- Market-session decision rules
- Failure behavior
- Delivery rules

A Development rule must not silently replace a Live rule.

### Stage 3: Load and validate the private master

The latest approved master is loaded from connected storage and checked for completeness.

### Stage 4: Full-population market verification

Every eligible master item is checked.

The current decision model distinguishes:

- Regular-market low
- Regular-market close
- Alternative-session eligibility
- Alternative-session final value after that session closes
- Suspension or listing-risk state
- Missing or conflicting data

A partial lookup is not sufficient for a final population-level result.

### Stage 5: Choose the decision price

The decision price depends on session eligibility and report timing.

Conceptually:

- Use the approved regular-session value for items not eligible for the alternative session.
- Use the approved final alternative-session value when that session applies and has closed.
- Do not substitute an earlier regular close when the rule requires a later final session value.
- Mark the item unverified when required source data is missing.

### Stage 6: Apply exclusions

Private held positions and other rule-defined exclusions are removed before the user-facing monitoring list is produced.

Exclusion and lookup failure are different states and must be logged separately.

### Stage 7: Calculate classifications

The system evaluates:

- Reached level
- Within proximity threshold
- Outside threshold
- Unverified
- Suspended or critical state
- Requires detailed review

### Stage 8: Validate report completeness

Before formatting, the workflow checks:

- Every expected item has a terminal result
- No lookup failure was converted to “not reached”
- Required fields are present
- Sorting is correct
- Exclusions were applied
- Critical states are visible
- Counts match the underlying classified rows

### Stage 9: Generate the report

The approved report contains a stable title and a fixed set of fields, such as:

- Item name and identifier
- Staged monitoring levels
- Staged target levels
- Proximity
- Quality grade
- Market-capitalization assessment
- Dividend field
- Suspension or critical-state display

Private examples are never published.

### Stage 10: Deliver through Gmail

Gmail is the delivery and communication layer.

The workflow separates:

- Report generation
- Report validation
- Gmail availability
- Send request
- Delivery status where available
- Reply receipt
- Reply processing

A successful report generation does not prove that the email was delivered.

### Stage 11: Record results

The work record stores:

- Source version
- Rule version
- Population count
- Verified count
- Excluded count
- Reached count
- Proximity count
- Unverified count
- Generation result
- Delivery result
- NG reason
- Safe continuation point

## 7. Report Rules Developed During the Project

The rule set evolved through repeated review.

Publicly shareable rule categories include:

- Full-population inspection
- Configurable proximity threshold
- Ascending proximity sort
- Exclusion of current private holdings
- Multiple monitoring and target levels
- Quality grade
- Market-capitalization suitability
- Dividend information
- Strong visual treatment for suspension
- Alternative-session final-value selection
- Explicit “no matching items” reporting
- No reuse of an earlier failed result
- No conversion of missing data into a negative result

The specific operational values remain private.

## 8. Main Difficulties

### 8.1 Incomplete market-data acquisition

The largest technical difficulty has been obtaining a complete, reliable set of decision values for every monitored item at the required time.

A population-level report is invalid if only some items were successfully checked.

### 8.2 Multiple trading sessions

Regular-market and alternative-session values may differ. The workflow must determine which items are eligible and wait for the relevant session's final value.

### 8.3 Timing

A report executed too early may use a non-final value. A report executed after all required sessions must still handle delayed or missing sources.

### 8.4 Missing data vs. no result

The most important reliability rule is:

> Failure to retrieve data is not evidence that no item matched.

Earlier attempts exposed the risk of reporting “none” or producing counts when the full population had not been verified.

### 8.5 Master synchronization

A report may be internally correct but still wrong if it uses an outdated master file. Connected-file recognition and synchronization can be delayed.

### 8.6 Rule drift between sessions

Thresholds, fields, exclusions, and formatting were refined over multiple chats. Without Live/Development separation, an older rule could reappear.

### 8.7 Formatting consistency

The report needs a stable approved format. Repeated free-form generation caused field omissions and wording changes.

### 8.8 Holding exclusion

The system must keep private held items out of the monitoring output while still preserving correct internal population accounting.

### 8.9 Generation vs. email delivery

A generated report can exist even when Gmail delivery fails. Earlier workflows risked treating these as one result.

### 8.10 Long-running session interruption

The work can outgrow a single conversation or stop during a long lookup. The next session needs an explicit continuation point.

## 9. Solutions Applied

### Full-population terminal-state check

Each eligible master item must finish in one of a defined set of states:

- Reached
- Near threshold
- Outside threshold
- Excluded
- Suspended or critical
- Unverified
- Failed

The report cannot claim population completeness while any required item remains unresolved.

### Multi-stage source verification

The workflow separates:

1. Regular-session low and close
2. Alternative-session eligibility
3. Alternative-session final value
4. Suspension and listing-risk checks

### Fail closed

If required data is unavailable:

- Do not reuse an older report
- Do not guess counts
- Do not classify the item as outside the threshold
- Return NG or “unable to finalize”
- Record the missing stage
- Preserve a continuation point

### Rule versioning

Confirmed changes move through Development, validation, and Live promotion. Previous values remain in History.

### Private/public separation

- GitHub: architecture, sanitized logic, limitations, and Issues
- Google Drive: real master and operational files
- Gmail: report delivery and replies
- ChatGPT: coordination, validation, reporting, and escalation

### Fixed schema and validation checklist

A report template and pre-send checklist reduce formatting drift and missing fields.

### Explicit work results

Generation, validation, sending, and reply handling are recorded separately.

## 10. Significant Failure Case

A key failure case occurred when the workflow could not obtain complete decision data for the full monitoring population.

The correct final result was not “zero matches.” It was:

- Final classification unavailable
- Full-population verification incomplete
- Missing regular/alternative-session data
- No reuse of prior results
- No guessed count

This failure became an architectural requirement: **an incomplete lookup must remain visible as incomplete**.

The project treats this as a successful design lesson even though the individual report run was NG.

## 11. Advantages

- Converts a personal analytical workflow into explicit reusable rules
- Reduces omissions across a large master list
- Prevents missing data from becoming a false negative
- Preserves the same rules across sessions
- Separates public architecture from private data
- Supports scheduled reporting without direct OpenAI API integration
- Provides traceable success and failure results
- Allows incremental refinement through Development and Live versions
- Creates a reusable pattern for other monitoring and reporting domains

## 12. Disadvantages and Trade-offs

- Complete data acquisition is time-consuming.
- Multiple market sessions increase timing complexity.
- Connected-file synchronization may be delayed.
- ChatGPT product limits may interrupt intensive runs.
- A no-OpenAI-API design provides less deterministic control than a dedicated backend.
- Email delivery verification may be incomplete.
- Detailed History and population results increase storage.
- Some market-status decisions require external authoritative sources.
- High-confidence automation requires extensive failure testing.

## 13. Current Status

### Completed or substantially defined

- Overall report purpose
- Private master concept
- Full-population requirement
- Proximity classification
- Stable field set
- Held-position exclusion
- Multi-stage decision-source model
- Missing-data failure rule
- Fixed-format report concept
- Google Drive private-source role
- Gmail delivery role
- Rule-management integration
- Work-result integration
- Public/private separation

### Implemented or tested in parts

- Scheduled report triggering
- Report title and body formatting
- Attachment and reply tests
- Master-file access experiments
- Rule changes and History design
- Full-population failure detection
- “No qualifying items” notification behavior
- Cross-session continuation design

### Not yet fully complete

- Reliable automated acquisition of every required market value
- End-to-end full-population production verification
- Final alternative-session data handling for every eligible item
- Guaranteed latest-master detection in every session
- Delivery-state confirmation for every email
- Automatic recovery after interrupted runs
- Finalized database schema and transaction tests
- Complete regression test suite

## 14. Current Final Stage

The project is in an **advanced architecture and integration-validation stage**, not yet a fully completed production system.

The rule model, report format, data-completeness policy, private storage role, Gmail role, and failure behavior are largely defined.

The remaining critical path is:

1. Guarantee the latest private master is loaded.
2. Acquire all required regular and alternative-session values.
3. Validate every eligible item reaches a terminal state.
4. Generate the fixed report.
5. Send through Gmail.
6. Verify and record each workflow stage.
7. Recover safely when any stage is interrupted.
8. Repeat the complete test across multiple market days.

Until those tests pass, the system must not claim fully autonomous production readiness.

## 15. Deferred Related Work

A more advanced derivative yearly-report concept has been deliberately moved to the bottom of the work queue because it requires additional review and validation.

This is an important project-management decision:

- Do not expand scope before the primary report is reliable.
- Complete higher-priority foundational tasks first.
- Keep the deferred concept recorded rather than deleting it.
- Mark it HOLD or deferred with a reason and prerequisite list.

## 16. Next Steps

1. Finalize the report rule schema.
2. Finalize market-source adapters and fallback policy.
3. Add per-item terminal-state records.
4. Add source timestamp and session metadata.
5. Implement latest-master verification.
6. Implement atomic result finalization.
7. Add fixed report-schema validation.
8. Separate generate, validate, send, delivery, and reply states.
9. Create recovery tests for partial population completion.
10. Run multi-day end-to-end regression tests.
11. Publish sanitized sample schemas.
12. Open technical Issues for unresolved data-completeness and session-timing problems.

## 17. Questions for Other Developers

- How should a no-OpenAI-API ChatGPT workflow guarantee full-population data completeness?
- What is the safest method for combining multiple market-session final values?
- How should source timestamps and finality be modeled?
- How should a scheduled task recover when only part of the population was processed?
- What is the best way to distinguish source failure, market closure, ineligibility, and true negative results?
- How should Gmail send acceptance and actual delivery be modeled separately?
- What checksum or version strategy is suitable for verifying the latest private master file?
- Which parts should eventually move to a deterministic local program while ChatGPT remains the coordinator?

## 18. Summary

The yearly-candle report is a concrete example of ChatGPT Applied Technology.

It shows how ChatGPT Plus can coordinate persistent rules, private connected files, multi-source verification, fixed reporting, email delivery, failure handling, and cross-session continuation without directly calling the OpenAI API.

The most important engineering lesson is simple:

> Incomplete data must remain incomplete. It must never be converted into a confident negative result.

The system is close to a complete design, but production readiness depends on reliable end-to-end full-population validation across repeated real runs.
