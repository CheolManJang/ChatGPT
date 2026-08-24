# G-Yearly Report: Google Drive–Based Case Study

> [!NOTE]
> **Document baseline:** 2026-08-24. Reference environment: individual ChatGPT Plus account using ChatGPT web/Work with a connected Google Drive and without direct OpenAI API calls. Described behavior is based on the tested account, permissions, context, connector state, and rollout. Revalidate after material product changes.

> [!CAUTION]
> **Free, non-commercial technical sharing — use at your own risk.** This document describes architecture and sanitized operational lessons. It does not publish production data, raw financial records, real symbols, prices, holdings, recipients, private prompts, or core decision algorithms. Verify all behavior before production use. See the [Disclaimer](../DISCLAIMER.md).


## Adoption Decision Summary

| Field | Project record |
|---|---|
| Introduced because | A Drive-controlled original was needed for a system-specific report, raw-data exchange, and recovery package. |
| Intended improvement | Separate source acquisition, validation, full processing, reporting, backup, and recovery. |
| Main difficulty | Drive latency, identity, permissions, stale copies, partial parsing, and cross-context differences. |
| Main advantage | User-visible original and practical connected-source workflow. |
| Main disadvantage | Variable speed, provider dependence, synchronization ambiguity, and incomplete restoration. |
| Observed result | Architecture and qualitative performance model are documented; controlled timing benchmark is incomplete. |
| Current status | Development / partially verified. |
| Retest trigger | Drive connector, permission, file format, account, context, or product changes. |

## 1. Why It Was Built

The G-Yearly Report was built to test whether ChatGPT Plus can coordinate a persistent reporting workflow around a user-controlled Google Drive original without directly calling the OpenAI API.

The practical goals are to:

- Keep the operational original in a location the user can open and manage directly
- Exchange approved raw source files through Google Drive
- Separate source acquisition, validation, report generation, review, delivery, backup, and recovery
- Reuse the same high-level workflow across conversations and work contexts when permissions allow
- Preserve a recoverable source outside the current chat
- Record the limitations of a connected-app, no-direct-API architecture

This is a technical case study, not a published investment method or financial recommendation.

## 2. Source Boundary

The G-Yearly Report uses its registered **Google Drive original** as its system-specific authoritative source.

This boundary must not be confused with the separate Yearly-Candle Monitoring Report, which uses its registered ChatGPT Library master. A system must never silently substitute Library for Drive, Drive for Library, an email attachment for an approved original, or a conversational recollection for a registered source.

Google Drive also supports:

- Approved raw-data exchange
- User-visible working originals
- Verified backup copies
- Recovery packages
- Supporting documents assigned to the Drive-based workflow

GitHub receives only architecture, sanitized examples, limitations, and lessons. It never receives the operational original or reconstructable decision logic.

## 3. High-Level Workflow

1. Identify the registered Drive original by stable metadata and expected location.
2. Confirm connector access and the active account or workspace boundary.
3. Fetch the required file rather than relying on an earlier conversational copy.
4. Validate filename, type, size range, schema or required sections, and expected version metadata.
5. Reject missing, partial, stale, duplicated, or ambiguous input.
6. Process the complete approved population.
7. Validate completeness and output structure.
8. Generate the report.
9. Perform required human review for material decisions or external delivery.
10. Record completion, NG, HOLD, failure cause, and continuation point.
11. Store or verify the approved output and recovery artifacts according to the backup policy.

A missing value is not converted into a negative result. An incomplete population is not reported as a complete population.

## 4. Speed Characteristics

No universal processing-time guarantee is made. As of the document baseline, a controlled benchmark with fixed file sizes, identical connector state, repeated runs, and percentile measurements has not been completed. Exact seconds would therefore be misleading.

Observed performance is divided into separate stages:

| Stage | Typical characteristic | Main variables |
|---|---|---|
| Drive search or listing | Often lighter than full content retrieval | folder size, indexing, query precision, permissions |
| File metadata lookup | Usually relatively small | connector availability, identity resolution |
| File acquisition | Can be noticeably slower and variable | file size, format, network, connector cold start, provider throttling |
| Parsing and validation | Depends on structure and completeness | compression, schema, row count, malformed content |
| ChatGPT reasoning and report generation | Grows with population and validation depth | context size, tool calls, retries, evidence requirements |
| Backup and verification | Adds necessary latency | copy size, version checks, checksum or manifest validation |
| Later reuse | May still require a fresh fetch | context boundary, cache state, permission and source changes |

The important metric is not only total elapsed time. A useful benchmark should record:

- Search time
- Acquisition time
- Parse time
- Validation time
- Report-generation time
- Backup time
- End-to-end time
- File size and record count
- Retry count
- Connector or permission failures
- Warm versus cold run
- Complete, NG, or HOLD result

Speed must never be improved by skipping source verification, population completeness, backup validation, or output checks.

## 5. Advantages

- **User control:** The original remains visible and directly manageable in Google Drive.
- **Practical raw-data exchange:** Approved source packages can be shared without placing operational data in public GitHub.
- **Separation from chat history:** The source is not dependent on one conversation remaining available.
- **Cross-device accessibility:** The user can inspect the original from devices supported by Google Drive.
- **Backup utility:** Drive can hold verified backup and recovery packages for systems assigned to it.
- **No direct OpenAI API requirement:** The reference workflow can be explored with ChatGPT Plus and connected applications.
- **Auditable boundary:** The registered source, report output, and public documentation can be assigned different storage and disclosure rules.
- **Human-readable operation:** Users can inspect folders, filenames, and supporting documents without a custom database administration interface.

## 6. Disadvantages and Limits

- **Variable latency:** Connector search, retrieval, parsing, and repeated validation can be slower than local or already-loaded content.
- **Permission dependence:** A visible file is not automatically accessible to every Chat, Project, Work context, account, or scheduled task.
- **Synchronization ambiguity:** A renamed, copied, replaced, or concurrently edited file can be mistaken for the intended original.
- **No immediate event guarantee:** A newly uploaded or changed file may not be detected instantly without an approved event mechanism.
- **Provider dependence:** Availability, limits, behavior, and permissions can change outside the project.
- **Large-file cost:** Bigger files require more transfer, parsing, validation, and context management.
- **Weak multi-file atomicity:** A set of related files can be observed at different update states unless a manifest or release package is used.
- **Incomplete restoration:** Restoring Drive files does not restore hidden model state, chat context, memory synchronization, connector authorization, or transient task state.
- **No execution guarantee:** A stored file does not prove that a scheduled or background workflow read and processed it successfully.
- **Context separation:** Chat and Work environments may not share the same loaded state even when they refer to the same Drive source.

## 7. Problems Encountered

### Slow or delayed file recognition

A file uploaded by the user can take time to become searchable or retrievable through the connected application. Repeated blind retries increase latency and can still select the wrong copy.

**Applied response:** identify the expected folder and file, use metadata checks, bound retries, and stop with NG or HOLD when identity cannot be proven.

### Visible to the user but unavailable to ChatGPT

Browser visibility does not prove connector permission in the active context.

**Applied response:** distinguish user-visible, connector-visible, and successfully fetched states. Do not claim that content was read until acquisition succeeds.

### Duplicate and stale copies

Backups, renamed files, exported copies, and attachments can resemble the operational original.

**Applied response:** maintain a registered source identity, expected location, version metadata, manifest, and validation fields. Never choose the newest-looking filename alone.

### Partial acquisition or parsing

A file can be fetched but still be incomplete, malformed, truncated, or incompatible.

**Applied response:** validate the full expected schema or section set, population count where safe, and terminal parse state before report generation.

### Long end-to-end execution

Multiple connector calls, full-population validation, retries, reasoning, formatting, delivery, and backup verification accumulate latency.

**Applied response:** measure each stage separately, reuse verified intermediate artifacts only when their identity and freshness remain valid, and preserve a continuation point rather than restarting blindly.

### Backup that is not a full recovery

Copying the original alone does not reconstruct rules, task state, History, MENU/CMD definitions, permissions, scheduled prompts, or validation results.

**Applied response:** use the [Complete Backup and Recovery Manifest](../04_Storage_Recovery/complete-backup-and-recovery-manifest.md) and validate restoration as an operational state, not merely as a successful file download.

## 8. Reliability Controls

The Drive-based workflow should use:

- A registered source identifier and expected location
- A manifest containing non-secret identity and version fields
- Explicit source freshness rules
- Schema and completeness checks
- Idempotent report-run identifiers
- Bounded retry with recorded causes
- NG and HOLD states
- Human approval for material decisions and external communication
- A result record with evidence, validation, and continuation point
- Separate operational, backup, and public-publication boundaries
- Recovery exercises rather than backup-presence assumptions

## 9. Recommended Performance Test

A safe benchmark should use sanitized or synthetic packages of several controlled sizes. For each size:

1. Run at least one cold acquisition and multiple warm repetitions.
2. Record every stage listed in the speed table.
3. Confirm that output completeness is identical across runs.
4. Introduce one missing file, one stale version, and one malformed package.
5. Confirm that failures produce NG or HOLD rather than a false successful report.
6. Repeat in each supported Chat, Project, Work, and scheduled-task context.
7. Re-run after meaningful connector, permission, or product changes.

Benchmark results must state the date, plan, context, file size, record count, permissions, and evidence source.

## 10. Current Status

As of 2026-08-24:

### Defined

- Google Drive original as the G-Yearly source boundary
- Approved raw-data exchange and backup roles
- Public/private publication separation
- Source validation, completeness, NG/HOLD, and recovery principles
- Qualitative speed model and benchmark fields

### Partially verified

- Connected Drive discovery and file acquisition in tested contexts
- User-visible file handling and observed latency variation
- Separation of source retrieval from report generation

### Not yet claimed as complete

- Controlled end-to-end speed benchmark
- Guaranteed identical access across every Chat, Project, Work, and scheduled-task context
- Immediate event-driven detection of every Drive change
- Fully automatic recovery of permissions, hidden context, memory, and transient execution state
- Production readiness without repeated full-path validation

## 11. Next Work

1. Define a sanitized benchmark dataset and measurement log.
2. Register stable non-secret source identity fields.
3. Measure cold and warm performance by stage.
4. Test duplicate, stale, missing, partial, and permission-denied cases.
5. Verify report completeness and backup restoration.
6. Record results by version without publishing operational data.
7. Promote only validated controls into the Live workflow.

## 12. Publication Safety

Never publish:

- The operational Drive original
- Raw financial or personal data
- Real symbols, prices, holdings, accounts, recipients, or file links
- Credentials, authorization details, or private folder identifiers
- Core decision algorithms or reconstructable partial rules
- Production prompts, logs, or backup packages

Public examples must be fictional, structurally useful, and operationally useless.
