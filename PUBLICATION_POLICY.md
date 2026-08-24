# Public Repository Safety and Publication Policy

> [!NOTE]
> **Document baseline:** 2026-08-24. Reference environment: individual ChatGPT Plus account using ChatGPT web/Work without direct OpenAI API calls. Architectural principles are general; product behavior and observed limitations are specific to the tested plan, context, permissions, connected apps, and rollout state. Revalidate after material product changes.

> [!CAUTION]
> **Use at your own risk.** This repository provides technical education and general information only, without warranties. You are responsible for evaluating, testing, securing, and legally complying with any implementation. To the maximum extent permitted by applicable law, the project owner and contributors disclaim liability for loss or damage arising from use or inability to use this material. See the [Disclaimer](DISCLAIMER.md).

## 1. Purpose

This repository is a **non-commercial public technical knowledge-sharing and discussion project**. It shares ChatGPT applied-technology architecture; it does not sell a product, paid service, consulting engagement, support contract, or guaranteed result.

Public documentation must help other users understand system design, reliability, workflow, and limitations without exposing the user's core algorithms, private data, or competitive decision rules.

## 2. Currency and Reference Baseline

Any public statement described as current, supported, implemented, available, advantageous, disadvantageous, or limited must identify:

1. Verification date
2. Tested plan and account type
3. Execution context: Chat, Project, Work, or scheduled task
4. Connected application and relevant permission boundary
5. Evidence class: official documentation, project observation, or inference
6. Retest trigger, including material product, permission, or rollout changes

Official product documentation and project-specific observations must not be presented as the same evidence. Undated current-state claims are incomplete and must be corrected before publication.

## 3. Never Publish

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

## 4. What May Be Published

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

## 5. Sanitization Rules

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

## 6. Publication Gate

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

## 7. Screenshot Policy

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

## 8. Public Examples

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

## 9. GitHub Issues and Comments

Public comments must follow the same policy as repository files.

Straightforward questions may be answered by ChatGPT. AI-generated responses can contain errors or omissions and must be verified before production use. Material design decisions and rule changes require human review and approval. AI-assisted replies should clearly carry this warning.

When responding:

- Answer architectural questions.
- Use fictional examples.
- Do not confirm private strategy details.
- Do not paste private logs.
- Do not reveal whether a specific real item is monitored or held.
- Escalate ambiguous requests to the user.
- Refuse requests that would reconstruct the core algorithm.

## 10. If Sensitive Data Is Published Accidentally

1. Stop further sharing.
2. Remove the exposed file or comment.
3. Assume a secret was copied if it was public.
4. Rotate credentials immediately when credentials were exposed.
5. Remove sensitive data from Git history using an appropriate history-rewrite procedure.
6. Review forks, cached pages, releases, artifacts, and attachments.
7. Record the incident privately.
8. Strengthen the publication gate.

Deleting the latest file alone may not remove sensitive content from Git history.

## 11. Repository Boundary

### Public GitHub

Architecture, sanitized examples, version history, limitations, and technical discussion.

### Private ChatGPT Library

Authoritative operational source files, registered masters, approved supporting files, and private outputs.

### Private Google Drive

Operational originals for systems assigned to Drive, including G-Yearly Report; approved raw-data sharing; verified backup copies and recovery packages.

### Gmail

Private delivery, replies, notifications, and approvals.

### ChatGPT

Coordination, rule application, validation, reporting, and approval requests.

## 12. User Responsibility and Legal Disclaimer

- Repository content is provided for technical education and general information.
- It is not legal, financial, investment, accounting, tax, security, or other professional advice.
- Examples and AI-assisted responses may contain errors, omissions, insecure assumptions, or outdated behavior.
- Users must independently review, test, validate, secure, back up, and confirm legal and regulatory compliance before use.
- Users assume the risks arising from implementation, configuration, automation, external communication, data processing, financial decisions, and production use.
- No warranty is provided regarding accuracy, completeness, fitness for purpose, availability, non-infringement, or error-free operation.
- To the maximum extent permitted by applicable law, the project owner and contributors disclaim liability for direct, indirect, incidental, special, consequential, or other losses or damages arising from use or inability to use the material.
- Liability that applicable law does not permit to be excluded or limited remains unaffected.

The repository's consolidated terms are:

- Materials are provided “as is” and “as available.”
- No product, paid service, support obligation, professional advice, service level, warranty, indemnity, or guaranteed result is offered.
- Users must independently test, secure, back up, verify, and confirm compliance.
- Responsibility is limited to the maximum extent permitted by applicable law.
- Liability that applicable law does not allow to be excluded or limited remains unaffected.

See the full [Disclaimer](DISCLAIMER.md).

## 13. Core Rule

> Publish the engineering method, not the private decision method.

The repository explains how to build a reliable ChatGPT partner system. It does not publish the user's investment edge, operational dataset, or private identity.
