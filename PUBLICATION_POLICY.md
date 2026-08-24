# Public Repository Safety and Publication Policy

## 1. Purpose

This repository shares **ChatGPT applied-technology architecture**, not private operational intelligence.

Public documentation must help other users understand system design, reliability, workflow, and limitations without exposing the user's core algorithms, private data, or competitive decision rules.

## 2. Never Publish

The following content must not be committed, quoted, summarized in reconstructable detail, attached to an Issue, or included in a screenshot.

### Core financial algorithms

- Exact stock-selection logic
- Exact buy or sell rules
- Proprietary yearly-candle decision criteria
- Exact support, target, or staged-price logic
- Position-sizing and averaging algorithms
- Portfolio recovery strategy
- Risk scoring thresholds
- Complete exclusion logic
- Private ranking or grading weights
- A combination of partial rules that reconstructs the full strategy

### Operational market data

- Real monitored symbols
- Real monitoring prices
- Real target prices
- Holdings
- Account numbers
- Orders
- Transaction history
- Portfolio values
- Private report results
- Raw market master data
- Screenshots exposing the private dataset

### Credentials and personal data

- API keys
- OAuth tokens
- Passwords
- Email addresses and recipients
- Authentication screenshots
- Private Drive links
- Personal identifiers
- Internal server addresses
- Private machine paths
- Session secrets
- Database connection strings

### Private system artifacts

- Production rules database
- Complete private prompts containing operational rules
- Private scheduled-task prompts
- Raw History exports
- Real error logs containing private values
- Full backups
- Unredacted email content
- Connected-app authorization details

## 3. What May Be Published

- High-level architecture
- Generic lifecycle and state models
- Sanitized database schemas
- Fictional examples
- Reliability principles
- Development/Live promotion concepts
- Work-management patterns
- MENU/CMD parsing design
- Generic report pipeline
- Failure-handling lessons
- Public limitations
- Open technical questions
- Non-sensitive test scaffolding

## 4. Sanitization Rules

Before publication:

1. Replace symbols and names with fictional values.
2. Replace all prices and thresholds with non-operational examples.
3. Remove counts that reveal the private master population when sensitive.
4. Remove dates tied to private transactions.
5. Remove recipients and account identities.
6. Remove file IDs, private links, and local paths.
7. Remove access tokens and query-string credentials.
8. Replace real logs with reconstructed examples.
9. Check image backgrounds and cropped areas.
10. Confirm that multiple public documents cannot be combined to reconstruct the private algorithm.

Sanitization is not only deleting names. Relationships, exact thresholds, repeated examples, and field combinations can reveal the underlying method.

## 5. Publication Gate

Every proposed public change must pass:

### Gate A: Classification

Classify the material as:

- Public
- Sanitized public
- Private
- Secret
- Unclear

Unclear material is not published.

### Gate B: Algorithm reconstruction test

Ask:

- Could a reader reproduce the trading method?
- Could separate documents be combined to infer it?
- Does the example preserve real thresholds or sequence?
- Does it expose which signals matter most?

If yes, remove or generalize it.

### Gate C: Data leakage test

Search for:

- Symbol patterns
- Account patterns
- Email addresses
- URLs with tokens
- Exact prices
- Real person names
- Local and cloud paths
- Database files
- Embedded image metadata

### Gate D: User approval

Material involving a new category, ambiguous algorithm detail, or previously private workflow requires explicit user approval before publication.

### Gate E: GitHub verification

After commit:

- Open the public page.
- Verify rendering.
- Confirm no hidden file or attachment was included.
- Check commit diff.
- Record the publication result.

## 6. Screenshot Policy

Screenshots are high risk because they may reveal information outside the intended crop.

Before publication:

- Prefer recreated diagrams over operational screenshots.
- Crop only after checking the full image.
- Remove account identities and email addresses.
- Remove file IDs and URLs.
- Remove browser bookmarks and tabs.
- Remove market symbols, prices, and holdings.
- Remove connected-app authorization information.
- Do not publish screenshots of private memory summaries.

The memory-settings screenshot supplied for discussion is used only to understand the product control. It is not added to the public repository.

## 7. Public Examples

Use neutral examples such as:

```text
ITEM-A
MONITOR_LEVEL-1
TARGET_LEVEL-1
RULE-DEMO-001
CMD.100
MENU.200
```

Do not use lightly modified real values. Fictional examples should be structurally useful but operationally useless.

## 8. GitHub Issues and Comments

Public comments must follow the same policy as repository files.

When responding:

- Answer architectural questions.
- Use fictional examples.
- Do not confirm private strategy details.
- Do not paste private logs.
- Do not reveal whether a specific real item is monitored or held.
- Escalate ambiguous requests to the user.
- Refuse requests that would reconstruct the core algorithm.

## 9. If Sensitive Data Is Published Accidentally

1. Stop further sharing.
2. Remove the exposed file or comment.
3. Assume a secret was copied if it was public.
4. Rotate credentials immediately when credentials were exposed.
5. Remove sensitive data from Git history using an appropriate history-rewrite procedure.
6. Review forks, cached pages, releases, artifacts, and attachments.
7. Record the incident privately.
8. Strengthen the publication gate.

Deleting the latest file alone may not remove sensitive content from Git history.

## 10. Repository Boundary

### Public GitHub

Architecture, sanitized examples, version history, limitations, and technical discussion.

### Private Google Drive

Operational source files, real datasets, private reports, user-specific configuration, and approved backups.

### Gmail

Private delivery, replies, notifications, and approvals.

### ChatGPT

Coordination, rule application, validation, reporting, and approval requests.

## 11. Core Rule

> Publish the engineering method, not the private decision method.

The repository explains how to build a reliable ChatGPT partner system. It does not publish the user's investment edge, operational dataset, or private identity.
