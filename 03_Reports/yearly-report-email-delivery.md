# Yearly Report Email Delivery Module

> [!NOTE]
> **Document baseline:** 2026-08-24. Reference environment: individual ChatGPT Plus account using ChatGPT web/Work with connected Gmail and without direct OpenAI API calls. Product behavior depends on account, permissions, context, connected-app state, and rollout. Revalidate after material changes.

> [!CAUTION]
> **Use at your own risk.** This is a sanitized delivery architecture. It does not contain recipients, email addresses, operational report values, real subjects, private attachments, credentials, tokens, or core financial algorithms. See the [Disclaimer](../DISCLAIMER.md).


## Adoption Decision Summary

| Field | Project record |
|---|---|
| Introduced because | Report generation and Gmail delivery failed for different reasons but were previously easy to confuse. |
| Intended improvement | Separate immutable validated report packages from sending, delivery evidence, receipt, replies, and resend control. |
| Main difficulty | Recipient privacy, attachment identity, timeout with uncertain side effects, and provider limitations. |
| Main advantage | Safer resend and clearer distinction between generated, sent, delivered, received, and replied. |
| Main disadvantage | More states, checks, latency, and provider-specific verification. |
| Observed result | Delivery architecture is defined; guaranteed delivery and immediate reply detection are not claimed. |
| Current status | Development / partially verified. |
| Retest trigger | Gmail connector, permissions, recipient source, attachment, schedule, or provider changes. |

## 1. Functional Boundary

This module manages delivery of an already generated and validated Yearly Report.

It does not:

- Select or evaluate report items
- Calculate decision values
- Own the report's financial logic
- Modify the report master
- Decide that incomplete data is complete
- Publish recipients or operational content
- Treat “message created” as proof of delivery

The report-generation module produces an immutable delivery package. The email module accepts or rejects that package and records delivery state.

## 2. Why Delivery Is Separate

Report generation and email delivery fail for different reasons.

A report may be valid even when Gmail permission, attachment acquisition, sending, delivery confirmation, or reply processing fails. Conversely, successful email transmission does not prove that the report was generated from complete and correct source data.

Separating the modules prevents:

- Recalculating a report during a simple resend
- Sending an unvalidated draft
- Marking a report complete because an email was created
- Sending duplicates after a timeout
- Losing the distinction between generation failure and delivery failure
- Exposing private content in public delivery logs

## 3. Input Contract

The email module accepts a delivery package containing non-secret control fields such as:

- Report run GUID
- Report version
- Generation completion state
- Validation completion state
- Immutable content hash
- Sanitized display title template
- Attachment manifest without public recipient data
- Delivery priority
- Approval requirement
- Idempotency key
- Creation and expiration timestamps

The operational package and recipient resolution remain private.

The module rejects a package when generation or validation is incomplete, the content hash does not match, a required attachment is missing, the package is stale, recipient resolution is ambiguous, or the same idempotency key already reached a successful terminal state.

## 4. Delivery States

| State | Meaning |
|---|---|
| Prepared | Immutable delivery package created |
| Validated | Package, content hash, and attachments verified |
| Waiting for Approval | Human confirmation required |
| Ready to Send | All prerequisites satisfied |
| Sending | Gmail operation in progress |
| Sent | Provider accepted the send request |
| Delivery Verified | Available evidence confirms expected delivery state |
| Receipt Confirmed | User or downstream process confirmed receipt |
| Reply Received | A reply associated with the report was acquired |
| NG | Delivery failed with recorded cause and stop point |
| HOLD | Waiting for permission, recipient resolution, approval, or provider recovery |
| Cancelled | Explicitly stopped before successful delivery |
| Superseded | Replaced by a newer approved package |

“Sent,” “Delivery Verified,” and “Receipt Confirmed” are not interchangeable.

## 5. Delivery Workflow

1. Receive an immutable validated report package.
2. Verify run GUID, report version, content hash, and expiration.
3. Resolve the approved private recipient through the authorized source.
4. Validate attachment identity, filename, type, size range, and hash when available.
5. Check the idempotency key and previous delivery attempts.
6. Request human approval when required.
7. Create the email using the approved template.
8. Perform a final content and recipient review.
9. Send through the connected Gmail capability.
10. Record provider response and message identifiers privately.
11. Verify delivery evidence when supported.
12. Record receipt or reply separately when observed.
13. Finish as Completed, NG, HOLD, Cancelled, or Superseded with a continuation point.

## 6. Idempotency and Duplicate Prevention

A resend must not be based only on “I did not see the result.”

Before retrying, check:

- Report run GUID
- Package content hash
- Recipient identity reference
- Previous send attempt
- Provider message reference
- Sent-mail evidence
- Delivery or receipt evidence
- Whether a newer package superseded the original

The idempotency key should bind the report version, approved recipient reference, and delivery purpose without exposing them publicly.

If provider acceptance is unknown after a timeout, use HOLD or NG with “side effect uncertain.” Do not send again until duplication risk is checked.

## 7. Attachment Management

An attachment visible in Gmail or Google Drive is not automatically acquired and validated by ChatGPT.

The module distinguishes:

- Attachment referenced
- Attachment accessible
- Attachment fetched
- Attachment parsed
- Attachment validated
- Attachment attached to the outgoing message
- Attachment delivery evidence observed

Common failures include permission mismatch, delayed availability, wrong file version, similar filenames, unsupported format, partial acquisition, oversized packages, stale cached content, and an attachment belonging to a different report run.

The safe response is to verify stable identity and package metadata. A filename alone is insufficient.

## 8. Subject and Body Templates

Templates may include only approved fields. Operational examples are not published.

A safe logical subject contains:

- Report type
- Report date or run date
- Completion state
- Sanitized result counts when publication is allowed

The body should clearly distinguish:

- Data and report baseline
- Source-validation result
- Report completion result
- Delivery timestamp
- Warnings or incomplete states
- Attachment manifest
- Human-review requirement
- Non-advisory and verification notice

An incomplete report must not use a subject or color suggesting successful full completion.

## 9. Scheduled Sending

A scheduled task initiates the delivery check; it does not bypass validation.

Before sending, the task must confirm:

- The expected report run exists
- Generation and full validation succeeded
- The correct package is current
- The approved private recipient resolves unambiguously
- Gmail permission is available
- No successful delivery already exists
- Required approval is present
- The provider is able to accept the operation

If any prerequisite fails, record NG or HOLD and do not fabricate a successful report or delivery.

## 10. Failure Recording

Every failed attempt records:

- Delivery attempt GUID
- Report run GUID
- State and stop point
- Direct cause
- Provider or connector evidence
- Whether a message may already have been sent
- Partial side effects
- Retry safety
- Required user action
- Continuation point
- Time and active session

Examples of distinct causes:

- Package validation failure
- Recipient unresolved
- Gmail permission unavailable
- Attachment acquisition failure
- Provider rejection
- Timeout with uncertain send side effect
- Delivery evidence unavailable
- Receipt not confirmed
- Reply acquisition failure

## 11. Reply Handling

A received reply is a new inbound work item, not a modification of the original report.

Reply processing should:

1. Resolve the conversation or message relationship.
2. Acquire the actual message and approved attachments.
3. Verify sender identity without guessing.
4. Classify the request.
5. Separate factual response from material design or rule decisions.
6. Require user review when private, ambiguous, or high impact.
7. Record the response result and relationship to the original delivery.

A polling or scheduled check is not an immediate event guarantee.

## 12. Privacy and Publication Boundary

Never publish:

- Recipient or sender addresses
- Provider message IDs
- Private subject lines
- Email bodies containing operational results
- Real report attachments
- Drive links or file IDs
- Account or authorization details
- Delivery logs containing private values
- Replies containing personal or financial information
- Credentials, tokens, or private prompts

Public GitHub may show only sanitized state models, fictional templates, validation rules, and reconstructed failure examples.

## 13. Advantages

- Report calculation and transmission failures remain distinguishable.
- Valid reports can be safely resent without recalculation when identity is proven.
- Duplicate delivery risk is reduced.
- Attachment and recipient validation become explicit.
- Sent, delivered, received, and replied states are auditable.
- Gmail provider failures do not alter report logic or source data.

## 14. Disadvantages and Limits

- More states and verification steps increase total time.
- Gmail and connected-app availability are outside project control.
- Provider acceptance may not prove human receipt.
- Timeout can leave uncertain side effects.
- Attachment retrieval can be slower than message metadata access.
- Recipient resolution and replies require strict privacy controls.
- Exact delivery tracking may be unavailable in the tested consumer environment.
- Background and scheduled execution are not guaranteed across every context.

## 15. Current Status

### Defined

- Functional separation from report calculation
- Immutable delivery package
- State model
- Idempotency and resend checks
- Attachment validation stages
- NG, HOLD, and uncertain-side-effect handling
- Private/public boundary

### Partially verified

- Connected Gmail message creation and sending in tested contexts
- Attachment and reply workflows under specific permissions
- Scheduled initiation and result reporting

### Not claimed as complete

- Guaranteed delivery or receipt confirmation for every message
- Immediate reply detection
- Identical behavior across Chat, Project, Work, and scheduled tasks
- Automatic safe retry for unknown send outcomes
- Full restoration of Gmail authorization and provider state from backup

## 16. Next Work

1. Define a sanitized delivery-package schema.
2. Create deterministic state-transition tests.
3. Test duplicate, timeout, provider rejection, missing attachment, and wrong-version cases.
4. Verify resend without report recalculation.
5. Record provider acceptance, delivery evidence, and user receipt separately.
6. Test reply acquisition with sanitized attachments.
7. Measure preparation, attachment, send, verification, and reply-check latency separately.
8. Publish only sanitized results.
