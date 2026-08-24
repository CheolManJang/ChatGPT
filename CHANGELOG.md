# Changelog

> [!NOTE]
> **Document baseline:** 2026-08-24. Reference environment: individual ChatGPT Plus account using ChatGPT web/Work without direct OpenAI API calls. Architectural principles are general; product behavior and observed limitations are specific to the tested plan, context, permissions, connected apps, and rollout state. Revalidate after material product changes.

All notable public technical changes to this repository will be documented in this file.

The project uses semantic-style documentation versions:

- **Major** — incompatible architecture or lifecycle change
- **Minor** — new documented subsystem, capability, or substantial design decision
- **Patch** — clarification, correction, formatting, or non-structural improvement

Private operational data is never included in public version history.

## [0.8.7] - 2026-08-24

### Changed

- Removed the duplicated standalone non-excludable-liability sentence from the public disclaimer and publication policy.
- Retained the standard “to the maximum extent permitted by applicable law” qualification used in major platform terms.
- Added OpenAI's official accuracy, user-verification, warranty, and liability terms as a drafting reference.
- Updated the current public documentation version from 0.8.6 to 0.8.7.

## [0.8.6] - 2026-08-24

### Added

- Consolidated the common Google, Microsoft, and open-source responsibility structure into repository-specific terms.
- Added explicit “as is” and “as available” language.
- Added no-product, no-paid-service, no-support-obligation, no-service-level, no-warranty, no-indemnity, and no-guaranteed-result terms.
- Added explicit user duties for testing, security, backups, verification, automation review, and legal compliance.
- Expanded covered loss categories while preserving liabilities that cannot legally be excluded.

### Changed

- Updated the current public documentation version from 0.8.5 to 0.8.6.

## [0.8.5] - 2026-08-24

### Added

- Added Microsoft software, Microsoft 365/service, and public-document responsibility references.
- Cited Microsoft's “as is” and user-risk language, its no-error-free/no-content-loss guarantee, and its warning that technical documents can contain inaccuracies.
- Clarified that Microsoft terms are drafting references and do not govern this repository.

### Changed

- Updated the current public documentation version from 0.8.4 to 0.8.5.

## [0.8.4] - 2026-08-24

### Added

- Added short quotations and links from Google's current Terms of Service and Google Drive-specific terms as disclaimer-drafting references.
- Clarified that Google's terms do not govern this repository and do not guarantee equivalent enforceability.

### Changed

- Reduced the README liability notice to a concise non-commercial, use-at-your-own-risk statement.
- Kept detailed warranty, responsibility, and liability language in the separate DISCLAIMER.
- Updated the current public documentation version from 0.8.3 to 0.8.4.

## [0.8.3] - 2026-08-24

### Clarified

- Defined the repository as a non-commercial technical knowledge-sharing and public discussion project.
- Clarified that it does not sell a product, paid service, consulting engagement, support contract, or guaranteed result.
- Clarified that public access does not create a seller–buyer, vendor–customer, consultant–client, or service-provider relationship.

## [0.8.2] - 2026-08-24

### Added

- Added a prominent use-at-your-own-risk notice to README and every detailed document.
- Added a standalone DISCLAIMER covering user responsibility, no warranties, professional-advice exclusions, validation duties, and limitation of liability.
- Added legal-safety language to the publication policy, Issue notices, and AI-authored reply rules.

### Changed

- Updated the current public documentation version from 0.8.1 to 0.8.2.

## [0.8.1] - 2026-08-24

### Added

- Added a visible 2026-08-24 document baseline to every Markdown document.
- Added an explicit per-system source map.

### Corrected

- Corrected the over-broad statement that Google Drive is backup-only.
- Clarified that the Yearly-Candle Monitoring Report uses its registered Library master.
- Clarified that the G-Yearly Report uses its Google Drive original.
- Documented Google Drive's additional roles for approved raw-data sharing, backups, and system-specific operational sources.
- Prohibited silent provider substitution without an approved migration.

## [0.8.0] - 2026-08-24

### Added

- Added a mandatory reference baseline for claims labeled current, implemented, available, advantage, disadvantage, or limitation.
- Required verification date, tested plan, execution context, connector permissions, evidence class, and retest triggers.
- Added a dated Plus/no-API reference-environment notice to the architecture document.

### Corrected

- Aligned the public architecture with the approved operational boundary: ChatGPT Library is the authoritative execution, search, and recovery source; Google Drive is backup-only.
- Distinguished official product documentation from account-specific project observations.
- Updated the README documentation summary and public version.

## [0.7.0] - 2026-08-24

### Added

- Documented a developer architecture for systems combining deterministic rules with ChatGPT inference.
- Separated facts, approved rules, model inference, state, tools, validation, and human approval.
- Defined contract-first inputs, rules, outputs, and side effects.
- Added explicit state-machine, idempotency, bounded-retry, fail-closed, and post-execution verification guidance.
- Added deterministic tests, scenario evaluations, output rubrics, and release regression sets.
- Added observability fields for rule versions, sources, tool results, external evidence, NG reasons, and continuation points.
- Added prompt and tool-design guidance.
- Added privacy, least-privilege, and human-approval boundaries.
- Added a Plus/no-direct-OpenAI-API implementation strategy.
- Added an eight-phase development sequence from observation through operation and recovery.
- Added common anti-patterns and a developer release checklist.
- Added a public Issue to discuss the boundary between deterministic logic and ChatGPT inference.

### Changed

- Updated the public documentation version from 0.6.0 to 0.7.0.
- Clarified that developers should build a deterministic operating shell around probabilistic inference rather than rely on a larger prompt.

## [0.6.0] - 2026-08-24

### Added

- Compared ChatGPT Library and Google Drive by purpose, access path, observed latency, version identity, synchronization risk, and operational boundary.
- Documented Gmail attachment failure modes and a safe attachment acquisition and validation procedure.
- Documented saved-memory speed benefits and synchronization risks.
- Compared Chat, Chat inside a ChatGPT Project, ChatGPT Work, and Work inside a ChatGPT Project.
- Added a source-of-truth matrix describing what can and cannot be authoritative.
- Documented confirmed non-guarantees across connected storage, projects, memory, Work, and background execution.
- Documented why chat-only, database-only, or file-only backups cannot provide full recovery.
- Added a complete backup manifest covering rules, work, History, MENU/CMD, projects, Library, Drive, Gmail, scheduled tasks, memory review, connectors, tests, GitHub state, and recovery runbooks.
- Added a restore sequence and objective recovery-completion criteria.
- Documented system state that cannot be restored exactly.
- Added a GitHub Issue for full-system backup and recovery design.

### Changed

- Updated the public documentation version from 0.5.0 to 0.6.0.
- Clarified that the recovery target is a verified operational state, not exact replay of hidden model or transient execution state.
- Clarified that Library and Drive are complementary layers rather than interchangeable universal storage.

## [0.5.0] - 2026-08-24

### Added

- Documented ChatGPT reliability risks, context drift, rule conflict, and unexpected behavior changes.
- Documented required foundation work before scheduling or repeated automation.
- Defined an authority order for safety rules, Live rules, work state, private sources, documentation, task prompts, memory, and conversational hints.
- Documented safe use of ChatGPT memory as a recall layer rather than the only authoritative rule source.
- Added a memory-review and isolated-diagnostic procedure based on current ChatGPT settings.
- Added a step-by-step rule-conflict diagnostic and recovery process.
- Added preventive rule, work, output, and automation controls.
- Added a Public Repository Safety and Publication Policy.
- Explicitly prohibited publication of core trading algorithms, exact decision criteria, real symbols, prices, holdings, raw data, production prompts, credentials, recipients, and private system artifacts.
- Added sanitization, screenshot, publication-gate, GitHub verification, and accidental-disclosure procedures.
- Added a GitHub Issue for public technical discussion of rule drift and recovery.

### Changed

- Updated the public documentation version from 0.4.0 to 0.5.0.
- Added a prominent safety boundary to README.
- Clarified that memory review is only one diagnostic layer and cannot replace Live rules, History, source verification, or regression tests.

## [0.4.0] - 2026-08-24

### Added

- Documented the yearly-candle monitoring report as a concrete ChatGPT Applied Technology use case.
- Documented why the reporting system was built and its technical goals.
- Documented the private master dataset and Google Drive source role.
- Documented full-population market verification across regular and alternative sessions.
- Documented staged decision-price selection and terminal-state classification.
- Documented fixed report validation and Gmail delivery stages.
- Documented the rule that missing data must never be converted into a negative result.
- Documented major implementation difficulties and applied solutions.
- Documented a significant incomplete-data failure case and the resulting architecture requirement.
- Documented completed, partially tested, incomplete, deferred, and next-step work.
- Added a public Issue for multi-session, full-population data completeness.

### Changed

- Updated the current public documentation version from 0.3.0 to 0.4.0.
- Added the yearly-candle system to the README as an applied case study.
- Clarified that production readiness requires repeated end-to-end population validation.

## [0.3.0] - 2026-08-24

### Added

- Reframed the project under the umbrella topic “ChatGPT Applied Technology.”
- Documented the no-OpenAI-API approach using ChatGPT Plus as the reference environment.
- Separated core patterns, Plus reference implementation, and availability-dependent extensions.
- Documented Google Drive as the private persistent source layer.
- Documented Gmail as the report, notification, reply, and approval communication layer.
- Documented generation, validation, sending, delivery, receipt, and reply as separate workflow states.
- Documented scheduled monitoring behavior and approval boundaries.
- Added a Free-plan manual adaptation model without claiming universal feature availability.
- Added official ChatGPT Work, plugins, scheduled tasks, email, and web references.
- Documented current implementation, under-construction items, and capabilities not claimed.

### Changed

- Changed the README title to “ChatGPT Applied Technology.”
- Updated the current public documentation version from 0.2.0 to 0.3.0.
- Clarified that rule management, work management, MENU/CMD, Drive, Gmail, GitHub, and automation are modules of one partner-oriented system.

## [0.2.0] - 2026-08-24

### Added

- Documented the MENU and CMD input interface.
- Defined MENU as navigation and CMD as executable operation.
- Defined dot-based canonical identifiers such as CMD.100 and MENU.100.
- Documented number-only invocation and attached explanation requests.
- Documented deterministic parsing order and ambiguity handling.
- Integrated MENU and CMD definitions with Development, Live, History, sessions, and work results.
- Documented advantages, disadvantages, difficulties, limitations, suggested schema, and parser tests.

### Changed

- Updated the current public version from 0.1.0 to 0.2.0.
- Expanded detailed documentation links in README.

## [0.1.0] - 2026-08-24

### Added

- Defined the repository as a public technical knowledge-sharing project.
- Added the first integrated English README.
- Documented the motivation for database-driven rule management.
- Documented Development and Live rule separation.
- Documented Development validation and Live promotion concepts.
- Documented Create_GUID, Group_GUID, and Link_GUID responsibilities.
- Documented History requirements, including previous values and change reasons.
- Documented work management with priority, results, NG, HOLD, and cancellation.
- Documented Session GUID, database locks, heartbeat, timeout, and recovery concerns.
- Documented advantages, disadvantages, encountered difficulties, current limitations, and next steps.
- Added open design questions as GitHub Issues for public technical feedback.

### Open design work

- Lock granularity and stale-session recovery
- Dev/Live storage model and atomic promotion
- Duplicate and semantic-similarity detection
- Work status model and History retention

[0.1.0]: https://github.com/CheolManJang/ChatGPT/commits/main
