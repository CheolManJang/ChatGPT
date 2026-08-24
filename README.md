# ChatGPT Applied Technology

> [!WARNING]
> **AI-assisted responses:** Straightforward questions may be answered by ChatGPT. AI-generated responses can contain errors or omissions. Verify all technical information before applying it to production systems. Material design decisions and rule changes require human review and approval.

A public technical knowledge-sharing project for building a persistent, rule-driven operational partnership with ChatGPT Plus—without directly using the OpenAI API.

The purpose of this repository is to show how ChatGPT can become the operational center of a practical system rather than only a conversational assistant.

The reference implementation combines persistent rules, work management, MENU and CMD input, Google Drive, Gmail, GitHub, scheduled monitoring, and human approval.

The repository also:

1. Shares solutions that may help users and developers facing similar problems.
2. Publishes unresolved technical questions so that others can suggest better approaches.

This is not a production-data repository. All examples are sanitized and focus on reusable technical ideas.

## Documentation Baseline

Claims labeled **current**, **implemented**, **available**, **advantage**, **disadvantage**, or **limitation** must state:

- Verification date
- Tested plan and account type
- Chat, Project, Work, or scheduled-task context
- Connected application and relevant permission boundary
- Whether the claim comes from official documentation, project observation, or inference
- Conditions that require retesting

Product behavior can change by plan, account, region, rollout, permissions, and workspace policy. Undated “current” claims are treated as incomplete documentation.

## Topics

### Rule Management and Rule Engine Design

A database-driven rule-management system using Delphi and SQLite, including:

- Development and Live rule separation
- Development-to-Live validation and promotion
- Rule grouping and relationship tracking with GUIDs
- Duplicate and similar-rule detection
- Change history and audit trails
- Active-mode management
- Shared processing through a Global Function interface

### Database Concurrency and Recovery

Practical design questions involving:

- Database locks
- Unique session IDs
- Heartbeats and timeout detection
- Ownership verification before commit
- Conflict and NG handling
- Interrupted-operation recovery
- History retention and database-size management

### Work and Task Management

Designing a work queue that records not only what must be done, but also what actually happened:

- Priority and priority sequence
- Required execution order
- Duplicate prevention
- Registered, completed, cancelled, NG, and HOLD states
- Work descriptions and result descriptions
- Registration and completion timestamps
- Resuming interrupted work
- Diagnosing historical failures

### Delphi Development

Reusable implementation notes and examples for:

- SQLite access and transaction handling
- Global functions and shared modules
- GUID and session management
- API communication
- File downloads and large-response handling
- Event-based file processing
- Defensive error handling

### Financial and Public APIs

Technical integration notes may include:

- Kiwoom OpenAPI stock-master and status information
- Trading suspension and administrative status checks
- DART Open API authentication and corporate-code downloads
- Large XML/ZIP response processing
- News and disclosure integration architecture

Only API techniques and fictional examples are shared. Real accounts, holdings, trading rules, and private financial data are excluded.

### Printing and Device Integration

Implementation notes for topics such as:

- Zebra ZPL commands
- Built-in printer fonts
- English-only label output
- Character encoding
- Delphi-to-printer communication
- PLC and 24V device integration concepts

### Automation and Reporting Architecture

General technical patterns for:

- Scheduled reports
- Data completeness validation
- Multi-stage verification
- Notification delivery checks
- Failure reporting
- Backup and synchronization workflows

## Initial Project: Rule Management Rebuild

The first documented project is a rebuild of a common rule-management system.

### Current Design

The system uses:

- An auto-increment integer primary key
- **Create_GUID** to identify rule creation
- **Group_GUID** to group related rules
- **Link_GUID** to connect Development, Live, and History records
- A unique Session ID for each active operation
- Heartbeats to detect inactive sessions
- History records containing previous values and change reasons

Development and Live rules may both remain active. When a Development rule is validated, its values are inserted into or used to update the related Live rule. The previous Live state is preserved in History.

### Questions Under Review

- What is the safest lock scope for concurrent sessions?
- How should abandoned sessions be detected and recovered?
- When should a failed operation be retried automatically?
- How should similar rules be detected without blocking valid variants?
- How long should detailed History be retained?
- How should normal completion, NG, HOLD, and cancellation be represented consistently?
- What information is required to continue work safely in a later session?

These questions will be documented as GitHub Issues so that alternative designs can be discussed openly.

## Current Public Version

**v0.8.0** — Reference-baseline and current-source alignment

- [Changelog](CHANGELOG.md)
- [Open design discussions](https://github.com/CheolManJang/ChatGPT/issues)

## Detailed Documentation

- [Developer Approach for Rule-and-Inference Systems](docs/developer-approach-rule-and-inference-systems.md) — deterministic vs. inference boundaries, contracts, states, idempotency, verification, evals, observability, human approval, implementation sequence, anti-patterns, and release checklist.
- [Storage, Context, and Workspace Boundaries](docs/storage-context-and-workspace-boundaries.md) — Library vs. Google Drive, Gmail attachments, memory synchronization, Chat, Projects, Work, authoritative sources, and confirmed non-guarantees.
- [Complete Backup and Recovery Manifest](docs/complete-backup-and-recovery-manifest.md) — why ordinary backups are incomplete, required backup contents, restore order, validation, completion criteria, and non-restorable state.
- [ChatGPT Reliability and Recovery](docs/chatgpt-reliability-and-recovery.md) — why stable workflows can suddenly drift, required foundation, memory review, conflict diagnosis, recovery steps, preventive controls, and limitations.
- [Public Repository Safety Policy](PUBLICATION_POLICY.md) — prohibited algorithms and data, sanitization, publication gates, screenshot safety, incident response, and public/private boundaries.
- [ChatGPT Plus No-API Architecture](docs/chatgpt-plus-no-api-architecture.md) — baseline: 2026-08-24, individual Plus account, ChatGPT web/Work, no direct OpenAI API; covers Library as the authoritative source, Drive as backup-only, Gmail, scheduled tasks, advantages, limitations, and verified construction status.
- [Yearly-Candle Monitoring Report System](docs/yearly-candle-monitoring-report.md) — motivation, goals, private master, multi-session market verification, report and Gmail pipeline, difficulties, solutions, failure lessons, current status, limitations, and next steps.
- [Rule Management and Work Tracking System](docs/rule-management-and-work-tracking.md) — why the system was built, architecture, rule lifecycle, History, work management, concurrency, advantages, disadvantages, difficulties, limitations, open questions, and next steps.
- [MENU and CMD Input Interface](docs/menu-and-command-interface.md) — command and menu namespaces, dot-based identifiers, explanation requests, parsing order, ambiguity handling, rule lifecycle integration, limitations, and tests.

## Planned Repository Structure

```text
.
├── README.md
├── CHANGELOG.md
├── docs/
│   ├── rule-management/
│   ├── database/
│   ├── delphi/
│   ├── api-integration/
│   ├── printing/
│   └── automation/
├── examples/
│   ├── delphi/
│   ├── sql/
│   └── sample-data/
└── .github/
    ├── ISSUE_TEMPLATE/
    └── pull_request_template.md
```

The structure will grow gradually. Documentation will be added only after each topic is organized and sanitized.

## Safety Boundary

This repository publishes the engineering architecture, not private decision intelligence.

Never publish core trading algorithms, exact decision rules, real symbols, monitoring or target prices, holdings, raw master data, production prompts, credentials, recipients, or private connected files. See the [Public Repository Safety Policy](PUBLICATION_POLICY.md).

## Data Safety

Never commit:

- Passwords, access tokens, API keys, or private certificates
- Account numbers or personal identifiers
- Email authentication information
- Production databases or database backups
- Real holdings, orders, prices, or private investment strategies
- Internal server addresses
- Private machine paths
- Proprietary customer or company data

Examples must use fictional names, values, identifiers, and logs.

## How to Participate

Feedback and contributions are welcome.

You can:

- Open an Issue for a technical question or bug
- Comment on an unresolved design problem
- Suggest improvements to a database schema
- Propose safer concurrency and recovery patterns
- Share an alternative Delphi implementation
- Submit a Pull Request containing documentation or sanitized sample code

A useful technical Issue should include:

- Problem statement
- Expected behavior
- Actual behavior
- Current design or attempted solution
- Reproduction steps
- Sanitized logs, schema, or code
- Specific questions for reviewers

## Project Status

This repository is at an early documentation and architecture stage. The README defines the overall scope; detailed documents, Issues, sample schemas, and example code will be added incrementally.

## Language

The main documentation is written in English to reach a wider developer audience. Korean notes or bilingual explanations may be added when they improve clarity.

## License

A license has not yet been selected. Until a license is added, the repository is publicly viewable, but reuse and redistribution rights are not automatically granted.
