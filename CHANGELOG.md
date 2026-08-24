# Changelog

All notable public technical changes to this repository will be documented in this file.

The project uses semantic-style documentation versions:

- **Major** — incompatible architecture or lifecycle change
- **Minor** — new documented subsystem, capability, or substantial design decision
- **Patch** — clarification, correction, formatting, or non-structural improvement

Private operational data is never included in public version history.

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
