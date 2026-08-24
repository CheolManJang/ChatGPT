# Storage, Context, and Workspace Boundaries

> [!NOTE]
> **Document baseline:** 2026-08-24. Reference environment: individual ChatGPT Plus account using ChatGPT web/Work without direct OpenAI API calls. Architectural principles are general; product behavior and observed limitations are specific to the tested plan, context, permissions, connected apps, and rollout state. Revalidate after material product changes.

> [!CAUTION]
> **Use at your own risk.** This material is provided for technical education and general information only, without warranties. You are responsible for evaluating, testing, securing, and legally complying with any implementation. See the [Disclaimer](../DISCLAIMER.md).


## Adoption Decision Summary

| Field | Project record |
|---|---|
| Introduced because | Library, Drive, Gmail, Memory, Chat, Projects, and Work were being treated as interchangeable when they have different boundaries. |
| Intended improvement | Assign an authoritative source and responsibility to each system. |
| Main difficulty | Access, identity, permissions, latency, synchronization, and behavior vary by context. |
| Main advantage | Clearer source selection and fewer silent substitutions. |
| Main disadvantage | More explicit routing, validation, and migration work. |
| Observed result | Source-boundary matrix and confirmed non-guarantees are documented. |
| Current status | Architecture approved; behavior remains context-dependent. |
| Retest trigger | Plan, product, connector, permission, account, or workspace changes. |

## 1. Purpose

A ChatGPT-centered operating system uses several different storage and execution surfaces. They are not interchangeable.

This document compares:

- ChatGPT Library
- Google Drive
- Gmail attachments
- Saved memory
- Chat
- Chat inside a ChatGPT Project
- ChatGPT Work
- ChatGPT Work inside a ChatGPT Project

The comparison reflects the project's current Plus reference environment, observed behavior, and current official product documentation. Availability and behavior can vary by account, plan, permissions, file type, and product updates.

## 2. Core Principle

> Fast access, shared context, authoritative state, and complete backup are different requirements.

A source may be fast but not synchronized. Another may be durable but slower to index. A Project may share files but not merge every chat's decisions. A backup may contain the files but not restore the active operational environment.

The architecture must assign one purpose to each layer.

## 3. ChatGPT Library

Library is the ChatGPT-native persistent artifact layer for user-facing files.

### Strengths

- Files remain available beyond a single transient work session.
- Native Library identity can be preserved across updates.
- A file can have versions and may be restored to an earlier version.
- Files can be found and brought into later ChatGPT work.
- It is suitable for documents, spreadsheets, reports, exports, and reusable artifacts created with ChatGPT.
- Once the correct Library item is resolved, it can be a direct source for ChatGPT file work.

### Weaknesses and limits

- A Library file is not automatically the same as the operational Live rule database.
- Restoring one Library file version does not restore Chat, memory, scheduled tasks, connected-app permissions, or external side effects.
- A file must still be identified correctly; similar names can cause ambiguity.
- Local processing may require materializing a remote file copy.
- Search or indexing may not prove that the newest intended logical source was selected.
- Concurrent edits require version checks to avoid overwriting a newer version.
- Large or binary files may take longer to transfer and validate.
- A Library artifact can preserve file history but not the complete reasoning and execution state that produced it.

### Best use in this project

- Final user-facing reports
- Public-safe documents
- Reusable templates
- Exported snapshots
- Recovery packages
- Reviewed artifacts that need identity and version history

### Not suitable as the only source for

- Real-time operational state
- Active session locks
- Heartbeats
- In-progress work ownership
- Gmail delivery state
- The only copy of critical rules

## 4. Google Drive

Google Drive has three approved roles in the current project: system-specific execution source, backup storage, and raw-data sharing.

### Strengths

- Appropriate for user-managed operational originals such as the G-Yearly Report.
- Accessible outside ChatGPT and usable by external programs.
- Supports folders, sharing, desktop synchronization, and Google-native files.
- Suitable for raw-data exchange and verified backup packages.
- Separates private operational material from public GitHub documentation.

### Weaknesses and limits

- Connector authorization and folder permissions are required.
- File indexing, recognition, retrieval, and conversion can be delayed.
- A new or changed file may not appear immediately.
- File name alone does not prove identity or latest-version status.
- Google-native and downloaded/exported formats can behave differently.
- Conflicting edits can occur outside ChatGPT.
- Access is often slower than an already-resolved Library item because of connector and network stages.
- Immediate event detection is not guaranteed.

### Best use in this project

- G-Yearly Report operational original
- Approved raw-data sharing
- User-maintained private source documents
- Verified backup and recovery packages
- Files that must be accessible outside ChatGPT

### Not suitable as the only source for

- Subsystems explicitly assigned to a Library master
- Session lock ownership
- Chat-specific progress
- Scheduled-task configuration
- Memory state
- Git commit history

## 5. Library vs. Google Drive

| Dimension | ChatGPT Library | Google Drive |
| --- | --- | --- |
| Current role | Authoritative source for assigned subsystems; native artifacts | G-Yearly original; raw-data sharing; backups; other assigned Drive sources |
| Typical access | Directly resolved within ChatGPT | Connected-app lookup and retrieval |
| Observed speed | Often faster after exact item resolution | May be slower because of connector and indexing stages |
| Speed guarantee | None | None |
| Version identity | Library item identity and versions | Drive file identity and Drive version behavior |
| External program access | Limited to supported ChatGPT workflows | Stronger for desktop and external-program sharing |
| Sync risk | Resolution and materialization delays | Connector indexing, permissions, conversion, and external edits |
| Full-system backup alone | No | No |
| Public-data boundary | Private unless deliberately published | Private operational layer |

### Current per-system source map

- Yearly-Candle Monitoring Report: Library master
- G-Yearly Report: Google Drive original
- Raw-data exchange: Google Drive
- Backups and recovery packages: Google Drive
- Public documentation and Issues: GitHub
- Rules and work state: the registered authoritative database for that subsystem

The system must resolve the subsystem first and then use its registered source. Provider substitution is a migration, not an automatic fallback.

## 6. Gmail Attachments

Finding an email and reading its attachment are separate operations.

### Common failure modes

- The email body is available but attachment bytes are not yet accessible.
- Attachment metadata is visible but the file must be downloaded separately.
- Large attachments take longer to retrieve or process.
- A file type may require conversion or a specialized reader.
- Scanned PDFs require OCR and may produce recognition errors.
- ZIP files require extraction and validation.
- Password-protected or encrypted files cannot be processed without authorized access.
- Multiple attachments with similar names can be confused.
- Inline images can be mistaken for meaningful attachments.
- A temporary attachment link may expire.
- The email connector may have permission to read messages but not perform every attachment action.
- A scheduled task may find the message but lack the same file-processing context as an interactive Work session.

### Safe attachment procedure

1. Resolve the exact email using sender, subject, date, and message identity.
2. List attachment names, types, and sizes.
3. Select the intended attachment explicitly.
4. Retrieve the attachment bytes through an authorized route.
5. Save the attachment to a persistent approved location when continued work is required.
6. Verify file size and, where practical, checksum.
7. Open with the correct file tool.
8. Validate expected pages, rows, sheets, or records.
9. Record whether the attachment was found, downloaded, opened, and validated.
10. Never report “attachment processed” when only the email body was read.

### Recommended project rule

For important workflows, Gmail is the communication entrance. The attachment should be moved or saved to Library or private Google Drive before it becomes an authoritative processing source.

## 7. Saved Memory

Saved memory can make responses feel faster and more continuous because ChatGPT may recall stable preferences and project purpose without requiring the user to restate them.

### Strengths

- Fast recall of stable preferences
- Useful cross-chat continuity
- Good for terminology and high-level project intent
- Reduces repeated explanations

### Synchronization problems

- Memory may not update immediately.
- A new correction may coexist with an older summary.
- Memory can omit detailed exceptions.
- Memory may generalize a temporary decision.
- Different surfaces may not use identical memory stores or controls.
- Memory does not automatically synchronize with the current Live database.
- Deleting or changing a source chat does not guarantee every derived memory state is reconstructed.
- Memory does not carry complete work-result, lock, or source-version details.

### Required rule

Memory is a cache and recall layer, not the source of truth.

Store only:

- Stable preferences
- Project purpose
- General vocabulary
- Pointers to authoritative sources

Do not store as memory alone:

- Exact operational rules
- Core financial algorithms
- Current thresholds
- Current master population
- Task ownership
- Credentials
- Destructive-action authorization

### Sync control

Every important workflow should compare:

- Memory summary
- Current Live rule version
- Current work state
- Current source-file version

A mismatch must be visible and resolved rather than silently merged.

## 8. Chat

Chat is appropriate for quick questions, clarification, and one focused outcome.

### Current strengths

- Fast interaction
- Easy refinement
- Good for exploration and small tasks
- Keeps one conversation's messages and attachments together

### Current limits

- Its context belongs to that chat.
- A new chat does not automatically contain every decision from another chat.
- Long conversations can be compacted or summarized.
- Attachments may be transient or chat-scoped unless saved elsewhere.
- Chat is not an authoritative database.
- It does not automatically expose a complete project source inventory.
- Restoring text does not restore tool state or external actions.

### Can be authoritative for

- The user's current instruction during the active task
- A reviewed decision before it is promoted

### Cannot be authoritative for

- Long-term Live rules by itself
- Complete backups
- Cross-chat task state
- Connected-source versioning

## 9. Chat Inside a ChatGPT Project

A ChatGPT Project provides shared project files, project instructions, and connected context to its chats.

### Current strengths

- Related chats stay grouped.
- Project instructions apply across project chats.
- Uploaded project files are shared.
- Connected project sources can be reused.
- Separate chats can focus on separate outcomes.

### Current limits

- Each chat still has its own messages, context, results, and goal.
- Decisions in one chat are not automatically merged into every other chat as a new authoritative rule.
- Two chats can work from different points in time.
- Shared files can be updated while another chat still holds older context.
- Concurrent writes to the same connected source can conflict.
- A Project does not directly provide access to an arbitrary local computer folder on the web.
- Project membership does not guarantee that every source is loaded on every turn.

### Recommended use

- Keep one project for one long-running operating system.
- Use separate chats for separate outcomes.
- Store approved shared instructions and source indexes at the project level.
- Promote final decisions to the authoritative rules source.
- Do not treat chat grouping as database synchronization.

## 10. ChatGPT Work

ChatGPT Work is designed for larger multi-step tasks using files, plugins, and approved tools to produce reviewable results.

### Current strengths

- Multi-step planning and execution
- Connected tool use
- File creation and editing
- Progress updates and user steering
- Reviewable deliverables
- Better fit for reports, analysis, and repository work

### Current limits

- A standalone Work chat still has its own context.
- Work is not a permanent database.
- A cloud task does not automatically have access to local computer folders.
- Available tools depend on permissions and surface.
- Long work can be interrupted by limits, errors, or missing input.
- A finished artifact does not automatically preserve the full execution environment.
- External send or write actions require separate verification.
- Work can use memory, but memory is not authoritative state.

### Recommended use

- Complex implementation
- Multi-source validation
- Creating reports and documents
- GitHub updates
- Controlled connected-app workflows
- Recovery and diagnostic procedures

## 11. ChatGPT Work Inside a ChatGPT Project

This means starting a Work task from within a ChatGPT Project.

### Current strengths

- Combines Work's multi-step execution with shared project files and instructions.
- Better for repeated workflows that depend on the same sources.
- Related Work chats remain organized under the project.

### Current limits

- Each Work chat still has its own execution context and result history.
- Shared project context does not merge all chat decisions automatically.
- Parallel tasks should not write to the same source.
- Project files may change while an existing Work chat continues with older context.
- Scheduled tasks may use saved prompts that differ from the latest project decision.
- Completion of one Work chat does not automatically update the rule database unless the workflow explicitly performs that write.

### Recommended use

Use it for substantial tasks, but finalize every approved change through:

1. Rule validation
2. Authoritative-source update
3. History entry
4. Work result
5. Backup checkpoint

## 12. Current Source-of-Truth Matrix

| Information | Authoritative source | Not authoritative alone |
| --- | --- | --- |
| Safety and publication policy | Versioned policy and user approval | Memory or a single chat |
| Live rules | Rules database or signed/versioned export | README summary |
| Development proposals | Development rule store | Conversational suggestion |
| Work state | Work database and result log | Chat completion wording |
| Private master data | Verified Drive file or approved database | Email attachment or stale download |
| Finished artifact | Versioned Library file or approved Drive output | Chat preview alone |
| Public technical documentation | GitHub main branch and version tag | Memory summary |
| Scheduled behavior | Saved task definition plus Live-rule reference | Original chat request alone |
| Gmail delivery state | Recorded message/send/delivery evidence | Report-generation success |
| User preferences | Memory plus explicit user confirmation when material | Old memory alone |

## 13. Confirmed Non-Guarantees

The current architecture does not assume:

- Immediate Drive synchronization
- Immediate memory synchronization
- Automatic sharing of every decision across project chats
- Complete attachment access from Gmail
- Perfect restoration from one exported chat
- Identical behavior across Chat and Work
- Identical tool access across surfaces
- Complete recovery from a single database file
- Real-time background execution
- Deterministic output without validation

## 14. Official Product References

- [Projects and chats](https://learn.chatgpt.com/docs/projects)
- [Use ChatGPT](https://learn.chatgpt.com/docs/use-chatgpt)
- [Get started with ChatGPT Work](https://learn.chatgpt.com/docs/get-started-with-work)
- [Long-running work](https://learn.chatgpt.com/docs/long-running-work)
- [Work with files](https://learn.chatgpt.com/docs/artifacts-viewer)
- [Memories](https://learn.chatgpt.com/docs/customization/memories)

## 15. Summary

Library, Drive, Gmail, memory, Chat, Projects, and Work solve different problems.

The system is reliable only when each layer has a defined role and important state is copied into a versioned authoritative source.
