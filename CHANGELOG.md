# Changelog

All notable public technical changes to this repository will be documented in this file.

The project uses semantic-style documentation versions:

- **Major** — incompatible architecture or lifecycle change
- **Minor** — new documented subsystem, capability, or substantial design decision
- **Patch** — clarification, correction, formatting, or non-structural improvement

Private operational data is never included in public version history.

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
