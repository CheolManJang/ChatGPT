# MENU and CMD Input Interface

> [!NOTE]
> **Document baseline:** 2026-08-24. Reference environment: individual ChatGPT Plus account using ChatGPT web/Work without direct OpenAI API calls. Architectural principles are general; product behavior and observed limitations are specific to the tested plan, context, permissions, connected apps, and rollout state. Revalidate after material product changes.

> [!CAUTION]
> **Use at your own risk.** This material is provided for technical education and general information only, without warranties. You are responsible for evaluating, testing, securing, and legally complying with any implementation. See the [Disclaimer](../DISCLAIMER.md).

## 1. Overview

This document describes a compact text interface for invoking commands and menus in a conversational or automation-driven system.

The interface was introduced because users should not need to repeatedly type long operational requests. A stable numeric identifier can invoke a known function, while optional attached text can request an explanation or provide additional context.

The design covers two related but distinct concepts:

- **CMD** — an executable command or operation
- **MENU** — a navigational entry that displays choices, status, or related operations

All identifiers and examples in this document are fictional. Private operational menus and production data are not included.

## 2. Why the Interface Was Added

Natural-language requests are flexible, but repeated operational work benefits from stable shortcuts.

Without a command/menu interface:

- The same request may be phrased differently each time.
- Long instructions must be repeated.
- Similar commands may be interpreted inconsistently.
- Users may not know which operations are currently available.
- A later session may not reproduce the same operation reliably.
- It is difficult to attach documentation, validation, and History to one stable operation identity.

The MENU and CMD interface provides a consistent entry point while preserving natural-language context.

## 3. CMD and MENU Responsibilities

### CMD

A CMD represents an action.

Examples of action categories include:

- Load rules
- Validate a Development rule
- Promote a rule to Live
- Display work status
- Resume an interrupted task
- Show History
- Run a verification procedure

A command should define:

- Stable command identifier
- Name and description
- Required parameters
- Validation rules
- Authorization requirements
- Expected result
- Failure and NG behavior
- Related menu or documentation
- Active mode and version

### MENU

A MENU represents navigation or a collection of related choices.

A menu should define:

- Stable menu identifier
- Display title
- Parent menu
- Child entries
- Related commands
- Display order
- Active state
- Explanation text
- Access or visibility rules

A menu may lead to another menu or invoke a command, but those relationships must be explicit.

## 4. Dot-Based Namespace

CMD and MENU identifiers use a dot separator rather than a hyphen.

Conceptual examples:

- CMD.100
- MENU.100
- CMD.190
- MENU.200

The dot was selected because it expresses a namespace-like relationship clearly:

- The left side identifies the type.
- The right side identifies the numeric entry.
- CMD and MENU can safely use the same number.
- Parsing is more predictable than treating a hyphen as punctuation, subtraction, or a range.
- Logs and History can preserve a canonical identity.

The canonical internal form should always include the namespace, even if a user-facing shortcut allows only the number in an unambiguous context.

## 5. Input Forms

### Number or canonical identifier only

A number or canonical identifier invokes or opens the related entry.

Examples:

- 190
- CMD.190
- MENU.200

If a bare number exists in both CMD and MENU, the system must not guess silently. It should use the current menu context or ask the user to select the intended type.

### Identifier with attached explanation request

An explanation keyword can be attached to an identifier to request detailed information instead of immediate execution.

Conceptual examples:

- 190 explanation
- CMD.190 explanation
- MENU.200 explanation

In Korean input, the explanation keyword may be attached without a space, such as a fictional form equivalent to “190설명”.

The parser should normalize whitespace but preserve the original input in History.

### Identifier with additional text

Additional text after the identifier can provide parameters or context.

Conceptual examples:

- CMD.190 validate only
- MENU.200 show inactive entries
- 190 explanation of failure handling

The command definition determines whether trailing text is an explanation request, a parameter, free-text context, or invalid input.

## 6. Parsing Order

A safe parser should use a deterministic order:

1. Preserve the original input.
2. Normalize leading and trailing whitespace.
3. Detect an explicit CMD or MENU namespace.
4. Extract the numeric identifier.
5. Detect a recognized explanation keyword.
6. Parse allowed parameters or trailing context.
7. Resolve the entry in the active rule version.
8. Check ambiguity, active state, authorization, and prerequisites.
9. Display a confirmation when the operation is destructive or high impact.
10. Execute or display the requested information.
11. Record the normalized identity and result.

The parser must not use a partial numeric match that could invoke the wrong command.

## 7. Explanation Behavior

An explanation request should not perform the command unless the command explicitly defines explanation as an executable operation.

A useful explanation includes:

- Command or menu name
- Purpose
- Required inputs
- Expected output
- Prerequisites
- Side effects
- Possible NG conditions
- Related commands
- Current Development or Live status
- Version and last-change information

This makes the command interface self-documenting.

## 8. Ambiguity and Error Handling

### Unknown identifier

Return an unknown-command or unknown-menu result and show nearby valid entries when safe.

### CMD/MENU number collision

If the same number exists in both namespaces and the user provides only the number:

- Resolve it from the current menu context when the context is explicit.
- Otherwise ask whether CMD or MENU was intended.
- Never silently prefer one global type.

### Inactive entry

Explain that the entry exists but is inactive. Do not treat it as nonexistent.

### Development-only entry

Show that the entry is in Development and has not been promoted to Live. It must not execute as a Live command unless Development execution is explicitly enabled.

### Invalid trailing text

Return the supported syntax and examples. Do not discard unrecognized text and execute the base command.

### Duplicate identifier

Registration must reject an exact duplicate within the same namespace and mode. The same numeric value may exist in CMD and MENU because their canonical identities differ.

## 9. Integration with Rule Management

CMD and MENU definitions are managed as rules rather than hard-coded labels wherever practical.

They can participate in:

- Development registration
- Validation
- Live promotion
- Active-state control
- Duplicate and similarity checking
- History
- Versioning
- Grouping
- Link relationships
- Rollback

Example conceptual identities:

- Create_GUID identifies the creation lineage of a command or menu.
- Group_GUID connects a menu with its related commands.
- Link_GUID connects Development, Live, and History versions.

## 10. Integration with Work Management

Every execution can create or update a work record containing:

- Canonical command identity
- Original user input
- Normalized input
- Session ID
- Work scope
- Start and completion timestamps
- Result status
- Result description
- NG reason
- Continuation point

A successful result must describe what the command actually did. An NG result must identify where processing stopped and whether retry is safe.

## 11. Advantages

- Short, repeatable operational input
- Stable identities across conversations and sessions
- Clear separation between navigation and execution
- Self-documenting explanation requests
- Easier History and audit correlation
- Easier automated testing
- Safer duplicate detection
- Consistent Dev-to-Live lifecycle
- Natural-language context remains available

## 12. Disadvantages and Trade-offs

- Users must learn or discover identifiers.
- Bare numbers can be ambiguous.
- A flexible text suffix makes parsing more complex.
- Renumbering can break documentation and user habits.
- Command definitions require version and lifecycle management.
- Overloading one input with execution, explanation, and parameters may confuse users.
- Localization requires controlled keyword aliases.

## 13. Difficulties Encountered

### Distinguishing execution from explanation

An identifier followed by an explanation keyword must display documentation rather than accidentally execute the command.

### Choosing a separator

A hyphen can be interpreted as punctuation, a negative value, or a range. A dot provides a clearer namespace boundary for CMD and MENU.

### Supporting compact input without unsafe guessing

Attached text is convenient, but the parser must avoid treating unknown text as harmless and executing the base command.

### Number collisions

CMD and MENU may legitimately share the same number. Their canonical identities therefore require a type namespace.

### Preserving conversational flexibility

The interface must support shortcuts without eliminating the user's ability to add context or ask a natural-language question.

## 14. Current Limitations

- The final grammar for every localized explanation keyword is still being validated.
- Prefix and suffix aliases should not be expanded without ambiguity tests.
- Parameter schemas are command-specific and not yet fully standardized.
- Authorization levels for high-impact commands are not finalized.
- Confirmation requirements for destructive operations need a formal policy.
- Menu-context resolution needs cross-session tests.
- Renumbering and alias deprecation rules are not finalized.
- Automated parser test coverage has not yet been published.

## 15. Recommended Data Model

A conceptual command/menu record may include:

- ID
- Entry_Type: CMD or MENU
- Entry_Number
- Canonical_Key
- Parent_Key
- Display_Name
- Description
- Syntax
- Parameter_Schema
- Display_Order
- Mode: Development or Live
- Active
- Requires_Confirmation
- Create_GUID
- Group_GUID
- Link_GUID
- Version
- Created_At
- Updated_At

A unique constraint should include at least Entry_Type, Entry_Number, Mode, and the relevant scope.

## 16. Suggested Tests

1. Execute a valid CMD by canonical identity.
2. Open a valid MENU by canonical identity.
3. Resolve a bare number in an explicit menu context.
4. Reject an ambiguous bare number.
5. Display explanation without executing.
6. Parse an allowed trailing parameter.
7. Reject unknown trailing text.
8. Reject an inactive entry with a clear message.
9. Prevent Development-only execution in Live mode.
10. Record original and normalized input.
11. Reject duplicate registration.
12. Verify History after Dev-to-Live promotion.
13. Verify NG result and continuation information.
14. Test whitespace and localized explanation aliases.
15. Test rollback to a prior command/menu version.

## 17. Open Questions

- Should a bare number be allowed outside an active menu context?
- Which explanation aliases should be canonical?
- Should attached text require a separator or whitespace?
- How should command aliases be versioned?
- Should deprecated identifiers redirect or return an error?
- Which commands always require confirmation?
- How should menu state be restored in a new session?
- Should documentation be stored in the rule database or generated from source files?

## 18. Summary

The MENU and CMD interface turns repeated natural-language operations into stable, versioned, and auditable entries.

The key design decisions are:

- Separate navigation (MENU) from execution (CMD).
- Use a dot-based canonical namespace.
- Allow compact explanation requests without accidental execution.
- Resolve ambiguity explicitly.
- Manage entries through the same Development, Live, History, session, and work-management architecture as other rules.

## Retired Commands: OPEN and CLOSE

The project previously experimented with `OPEN` and `CLOSE`-style commands to control Library-to-Memory synchronization and conversational work-session boundaries. They were retired because a conversational command could not prove deterministic memory loading, atomic synchronization, persistence, clearing, cross-context consistency, or completion after interruption.

These names must not imply that ChatGPT Memory behaves like a transactional database. The current architecture uses explicit Rule Engine and Task Manager commands, registered source verification, Session GUID ownership, History, idempotency, and reconciliation. See [Retired Experiment: Loading Library Rules into ChatGPT Memory](../04_Storage_Recovery/retired-library-to-memory-experiment.md).

