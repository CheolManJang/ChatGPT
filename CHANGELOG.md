# Changelog
> [!NOTE]
> **Document baseline:** 2026-08-24. Reference environment: individual ChatGPT Plus account using ChatGPT web/Work without direct OpenAI API calls. Architectural principles are general; observed behavior depends on the tested plan, context, permissions, connected apps, and rollout. Revalidate after material product changes.
> **기준:** 2026년 8월 24일. 참조 환경: ChatGPT 웹/Work를 사용하며 직접 OpenAI API를 호출하지 않는 개인 ChatGPT Plus 계정. 아키텍처 원칙은 일반적이지만, 관찰된 동작은 테스트한 플랜, 맥락, 권한, 연결된 앱 및 배포 상태에 따라 달라집니다. 중요한 제품 변경 후 다시 검증하십시오.

> [!CAUTION]
> **Use at your own risk.** This material is provided for technical education and general information only, without warranties. Evaluate, test, secure, back up, and legally review any implementation. See the [Disclaimer](DISCLAIMER.md).
> **사용 시 주의.** 본 자료는 교육 및 일반 정보 제공 목적이며, 어떠한 보증도 제공하지 않습니다. 모든 구현을 직접 평가·테스트하고, 보안과 백업을 확인하며, 필요한 법적 검토를 수행하십시오. [면책 조항](DISCLAIMER.md)을 참조하십시오.


> [!NOTE]
> **Document baseline:** 2026-08-24. Reference environment: individual ChatGPT Plus account using ChatGPT web/Work without direct OpenAI API calls. Architectural principles are general; product behavior and observed limitations are specific to the tested plan, context, permissions, connected apps, and rollout state. Revalidate after material product changes.

All notable public technical changes to this repository will be documented in this file.

The project uses semantic-style documentation versions:

- **Major** — incompatible architecture or lifecycle change
- **Minor** — new documented subsystem, capability, or substantial design decision
- **Patch** — clarification, correction, formatting, or non-structural improvement

Private operational data is never included in public version history.

## [0.14.9] - 2026-08-24

### Added

- Added sanitized samples for the implemented TASK lifecycle and Rule DB promotion flow.
- Registered `PUBLIC-DOC-NOTICE-001`, requiring the approved bilingual baseline and caution notice in every public document.

### Changed

- Reclassified Task Management and Rule Management from architecture-only/incomplete descriptions to bounded private implementations with explicitly separated hardening work.
- Added version, baseline, environment, evidence class, fictional-data boundary, and retest information to the Report TAG sample.
- Applied the bilingual NOTE and CAUTION block to every Markdown document and the public HTML report sample.
- Removed README topic candidates that came only from technical questions and were not verified as shared implementation experience.

## [0.14.8] - 2026-08-24

### Added

- Added an explicit explanation that the Report TAG document shares a real development experience with other developers and users; it does not prescribe a universal stopping rule.
- Summarized the complete case: Report slowdown, Logic-level TAG introduction, successful optimization, desire for continued use, AI stability failure after small changes, unsuccessful rule reinforcement, preservation of optimized Logic, and removal of the unstable TAG layer.
- Added the transferable lesson that other teams should test and choose among retention, redesign, deterministic implementation, pause, or removal for their own environments.

### Clarified

- Clarified that the lesson is not to stop after a fixed number of failures, but to document benefits and instability honestly and not claim that additional written rules solved an observed AI limitation.
- Updated the current public documentation version from 0.14.7 to 0.14.8.

## [0.14.7] - 2026-08-24

### Corrected

- Corrected the TAG lifecycle history: the project did not introduce TAGs with a predetermined plan to remove them. The feature worked well, and continued use was initially desired for ongoing Report visibility and optimization.
- Documented that TAG retirement was a later decision caused by observed AI limitations: high interpretation burden, behavior reset or reconstruction after small changes, and failure of increasingly precise written rules to keep the mechanism stable.
- Clarified that removal became reasonable only because the underlying Report Logic had already been supplemented, corrected, optimized, and regression-checked.

### Changed

- Reframed TAG removal as a practical fallback after desired continued use failed, rather than as optional cleanup or an original temporary-use assumption.
- Clarified that the current temporary-use-only policy was adopted from this failure lesson and was not the feature's original intention.
- Updated the current public documentation version from 0.14.6 to 0.14.7.

## [0.14.6] - 2026-08-24

### Corrected

- Corrected the primary reason Report TAGs were removed after optimization: a TAG looked small to the user but carried large weight for the AI by influencing logic priority, rule and example retrieval, dependency scope, comparison targets, and output selection.
- Clarified that leaving TAGs active after optimization could add processing cost, interfere with unrelated logic, revive retired behavior after a small change, and create false confidence that AI behavior was permanently fixed.

### Changed

- Defined TAG removal as part of optimization completion rather than optional cleanup.
- Required the optimized Report to remain behaviorally equivalent and independently testable without visible TAGs before Live promotion.
- Updated the current public documentation version from 0.14.5 to 0.14.6.

## [0.14.5] - 2026-08-24

### Corrected

- Corrected the primary Report TAG adoption trigger: Report processing had become slow, and TAGs were introduced to isolate the many contributing logic blocks so each could be inspected and optimized independently.
- Clarified that finding logic-specific defects and weak points was an important benefit of the same mechanism, but not the complete original reason for introducing TAGs.
- Clarified that TAGs did not improve speed by themselves; they exposed logic boundaries, reduced the optimization search scope, and helped identify unnecessary or repeated work.

### Added

- Added performance-analysis and processing-scope rows to the before/during/after comparison.
- Added a sanitized performance-diagnosis example without publishing actual timings or production rules.
- Added regression requirements to verify that optimized logic does not repeat unchanged work and that complete Report behavior remains equivalent.

### Changed

- Updated the current public documentation version from 0.14.4 to 0.14.5.

## [0.14.4] - 2026-08-24

### Added

- Added the actual Report TAG adoption story: the report already contained many logic blocks, the user identified suspicious output, and the visible TAG narrowed AI analysis to the responsible logic and its dependencies.
- Added a before/during/after comparison covering defect description, search scope, change risk, evidence, and Live output.
- Added the difficulties encountered while defining logic boundaries, maintaining TAG mappings, separating root cause from presentation, preventing a hidden rule layer, preserving public-safe evidence, and removing TAGs safely.
- Added a direct advantages/disadvantages table and an explicit statement of the improvement obtained.
- Added a fictional Report TAG diagnostic sample showing focused correction, before/after output, regression checks, and TAG-free Live output.

### Changed

- Linked the Report TAG retrospective to its diagnostic evidence sample.
- Preserved TAGs as Development-only diagnostic instrumentation: optimized logic remains, while visible TAGs are removed before Live.
- Updated the current public documentation version from 0.14.3 to 0.14.4.

## [0.14.3] - 2026-08-24

### Corrected

- Corrected the documented purpose of Report TAGs: they were added to Development report output as observable identifiers for the logic responsible for each result, not primarily as stage-selection commands.
- Documented how the user and AI used visible TAGs to trace wrong, missing, duplicated, or inconsistent output to the responsible logic and expose that logic's weak point.
- Added sanitized examples of logic identity, logic version, validation state, and result correlation.
- Corrected the optimization workflow to emphasize logic inventory, tagged output, user inspection, focused correction, before/after comparison, and logic-specific regression tests.

### Changed

- Clarified that the final gain was the optimized, independently testable logic—not the TAG itself.
- Mapped visible diagnostic TAGs to permanent internal module identifiers, version metadata, correlation evidence, deterministic tests, contracts, validation gates, TASK results, and History.
- Reaffirmed removal of visible diagnostic TAGs before Live to prevent stale mappings, AI misdirection, internal-structure leakage, and regression.
- Updated the current public documentation version from 0.14.2 to 0.14.3.

## [0.14.2] - 2026-08-24

### Added

- Documented the project observation that small prompt, rule, report, stage, example, or formatting changes can reintroduce previously retired behavior.
- Added the 2026-08-24 tested ChatGPT Plus web/Work baseline and distinguished the observation from a universal product guarantee.
- Explained that precise rules reduce ambiguity but do not compile a probabilistic AI into permanently fixed behavior.
- Added mandatory post-change regression checks for retired TAGs and commands, stage ordering, stop conditions, NG/HOLD, continuation, aliases, contexts, and source versions.

### Changed

- Classified every material change as a possible regression trigger.
- Required deterministic state machines, contracts, database constraints, validators, and tests to detect and contain AI drift.
- Added Live-promotion blocking when retired behavior reappears.
- Updated the current public documentation version from 0.14.1 to 0.14.2.

## [0.14.1] - 2026-08-24

### Added

- Defined TAGs as temporary Development, optimization, migration, testing, or diagnostic scaffolding only.
- Added required TAG owner, scope, reason, creation version, expiration condition, permanent replacement target, and removal regression test.
- Added a mandatory pre-Live removal procedure covering definitions, activations, aliases, fallbacks, prompts, memory guidance, files, schedules, and tests.
- Documented why continued TAG use drifts across logic changes, contexts, priorities, missing or extra labels, Memory, Library, schedules, and model interpretation.

### Changed

- Added a Live promotion gate that rejects business behavior still dependent on TAGs, prompt-only routing, or unversioned experimental controls.
- Defined TAG removal as a required completion step for optimization rather than optional cleanup.
- Updated the current public documentation version from 0.14.0 to 0.14.1.

## [0.14.0] - 2026-08-24

### Added

- Added a retrospective on the temporary TAG mechanism used to split Report work into source, validation, processing, formatting, delivery, and result stages.
- Documented how a visually simple TAG can materially change AI attention, retrieval scope, priority, stage order, validation focus, output selection, and context size.
- Documented the optimization benefits: bounded context, clearer stage diagnosis, explicit prerequisites, focused tests, and safer NG/HOLD continuation.
- Documented long-term risks: TAG proliferation, hidden duplicate rule layers, version drift, conflicting activation, missing dependencies, false determinism, context growth, synchronization risk, debugging ambiguity, and possible internal-logic leakage.
- Added a temporary-use and removal procedure.

### Changed

- Marked persistent TAG-based report business logic as retired.
- Mapped temporary TAG purposes to explicit replacements: report state machine, input contracts, validation gates, TASK dependencies, module commands, NG/HOLD, continuation records, versioned output schemas, Session GUIDs, and History.
- Clarified that report color badges are display labels only and do not execute logic.
- Updated the current public documentation version from 0.13.0 to 0.14.0.

## [0.13.0] - 2026-08-24

### Added

- Added a dedicated multi-session synchronization and locking retrospective for simultaneous Chat, Project, Work, and scheduled-task contexts.
- Documented split-brain scenarios involving different rule versions, duplicate TASK ownership, mid-run source changes, interrupted owners, delayed CLOSE operations, and Memory synchronization assumptions.
- Defined the critical boundary that conversations and saved Memory are not treated as lockable transactional state.
- Added Session GUID, authoritative-store lock, lock-scope, heartbeat, lease, takeover, optimistic version-check, idempotency, external-side-effect, reconciliation, and split-brain recovery designs.
- Documented coordination between separate Rule Engine and Task Manager databases without assuming cross-database atomicity.
- Added implementation acceptance requirements for concurrency, timeout, duplicate action, restore, and reconciliation tests.

### Changed

- Replaced conversational session synchronization assumptions with authoritative-record coordination.
- Updated the current public documentation version from 0.12.1 to 0.13.0.

## [0.12.1] - 2026-08-24

### Added

- Documented the retired `OPEN` and `CLOSE`-style commands that attempted to create explicit Library-to-Memory synchronization and work-session boundaries.
- Documented their intended benefits: visible synchronization points, source comparison, ownership, result recording, and continuation.
- Documented failure scenarios involving stale summaries, mid-session source changes, interrupted conversations, concurrent versions, partial close results, and false assumptions that memory was cleared or synchronized.
- Documented the replacement: deterministic Rule Engine and Task Manager operations, registered source validation, Session GUID ownership, History, idempotency, and reconciliation.

### Changed

- Marked `OPEN` and `CLOSE` as retired Memory synchronization commands in the MENU/CMD documentation.
- Clarified that conversational commands cannot make ChatGPT Memory behave like an atomic transactional database.
- Updated the current public documentation version from 0.12.0 to 0.12.1.

## [0.12.0] - 2026-08-24

### Added

- Added a retrospective on the retired experiment that loaded or summarized Library-based rules into ChatGPT saved memory.
- Documented the original goals: faster startup, reduced repeated source retrieval, cross-conversation orientation, and less repeated explanation.
- Documented observed benefits for lightweight recall and non-critical preferences.
- Documented synchronization drift, missing version identity, summarization loss, non-deterministic recall, rule mixing, weak auditability, context differences, and incomplete backup and recovery.
- Documented why memory was rejected as an authoritative store for Live rules, TASK state, report masters, and recovery.
- Added the approved source boundary, replacement architecture, version-bound cache requirements, and a memory-drift diagnostic procedure.

### Changed

- Marked the Library-to-memory architecture as Retired rather than silently omitting the failed experiment.
- Updated the current public documentation version from 0.11.0 to 0.12.0.

## [0.11.0] - 2026-08-24

### Added

- Defined Rule Engine and Task Manager as functionally separate subsystems with separate databases, state models, services, lock namespaces, commands, transactions, History, and ownership.
- Added an explicit cross-system integration contract using stable GUID references, correlation IDs, idempotent commands, results, and domain events.
- Added a migration plan and acceptance criteria for private implementation; documentation does not claim that operational migration is already complete.
- Added a standalone Yearly Report Email Delivery Module covering immutable validated packages, private recipient resolution, attachment validation, idempotency, sent/delivered/received/replied states, NG, HOLD, resend safety, and reply handling.
- Added a publication-safe, color-coded HTML report example and SVG preview using fictional identifiers and redacted values only.

### Changed

- Split Rule Management, Task Management, and MENU/CMD into separate functional folders and documents.
- Clarified that MENU/CMD routes operations but does not own Rule or TASK business state.
- Updated the current public documentation version from 0.10.0 to 0.11.0.

## [0.10.0] - 2026-08-24

### Changed

- Reorganized technical documents into visible top-level topic folders: Architecture, Rule/Work/MENU, Reports, and Storage/Recovery.
- Added an index README to every topic folder.
- Updated all root documentation links to the new paths.
- Added a visible Feedback folder with direct links for viewing Issues and creating a new Issue.
- Added clear instructions for commenting on an existing topic or opening a new technical question.
- Updated the current public documentation version from 0.9.0 to 0.10.0.

## [0.9.0] - 2026-08-24

### Added

- Added a standalone G-Yearly Report case study using its registered Google Drive original.
- Documented why the system was built, its source boundary, high-level workflow, advantages, disadvantages, and current status.
- Added a qualitative stage-by-stage speed model and explicitly marked controlled timing benchmarks as not yet completed.
- Documented delayed file recognition, permission mismatch, duplicate or stale copies, partial parsing, accumulated end-to-end latency, and incomplete recovery risks.
- Added reliability controls, a sanitized performance-test plan, next work, and strict publication-safety boundaries.

### Changed

- Updated the current public documentation version from 0.8.9 to 0.9.0.

## [0.8.9] - 2026-08-24

### Added

- Added a third-party provider responsibility boundary for connected services, development tooling, database distributions, operating systems, repositories, libraries, and other dependencies.
- Clarified that support requests, complaints, claims, and exercises of legal rights concerning a third-party failure should be directed to the responsible provider or rights holder under its governing terms and procedures.
- Clarified that this repository and its contributors do not represent, guarantee, insure, resell, or substitute for third-party providers.
- Clarified that the clause does not predetermine legal liability, which depends on the facts, applicable law, and governing terms.

### Changed

- Updated the current public documentation version from 0.8.8 to 0.8.9.

## [0.8.8] - 2026-08-24

### Added

- Added a plain-language responsibility summary tailored to a free, non-commercial GitHub technical-sharing project.
- Combined common open-source and AI-project practices: educational purpose, “as is” delivery, no support promise, AI fallibility, independent testing, backup, security, human supervision, and use at the user's own risk.

### Changed

- Consolidated the README's separate responsibility and AI notices into one prominent warning.
- Rewrote the wording for this repository instead of copying another project's disclaimer.
- Updated the current public documentation version from 0.8.7 to 0.8.8.

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
