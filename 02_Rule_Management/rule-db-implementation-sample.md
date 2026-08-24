# Sanitized Rule DB Implementation Sample
> [!NOTE]
> **Document baseline:** 2026-08-24. Reference environment: individual ChatGPT Plus account using ChatGPT web/Work without direct OpenAI API calls. Architectural principles are general; observed behavior depends on the tested plan, context, permissions, connected apps, and rollout. Revalidate after material product changes.
> **기준:** 2026년 8월 24일. 참조 환경: ChatGPT 웹/Work를 사용하며 직접 OpenAI API를 호출하지 않는 개인 ChatGPT Plus 계정. 아키텍처 원칙은 일반적이지만, 관찰된 동작은 테스트한 플랜, 맥락, 권한, 연결된 앱 및 배포 상태에 따라 달라집니다. 중요한 제품 변경 후 다시 검증하십시오.

> [!CAUTION]
> **Use at your own risk.** This material is provided for technical education and general information only, without warranties. Evaluate, test, secure, back up, and legally review any implementation. See the [Disclaimer](../DISCLAIMER.md).
> **사용 시 주의.** 본 자료는 교육 및 일반 정보 제공 목적이며, 어떠한 보증도 제공하지 않습니다. 모든 구현을 직접 평가·테스트하고, 보안과 백업을 확인하며, 필요한 법적 검토를 수행하십시오. [면책 조항](../DISCLAIMER.md)을 참조하십시오.


> [!NOTE]
> **Sample version:** `RULE-DB-SAMPLE-1.0`
> **Document baseline:** 2026-08-24
> **Related feature status:** bounded private implementation in use; migration and concurrency hardening in Development
> **Evidence class:** sanitized logical extract and reconstructed fictional transaction
> No real rule value, private prompt, database file, GUID, path, or operational condition is reproduced.

## What This Sample Shows

It shows the implemented separation of rule identity, environment-specific values, immutable History, and promotion evidence. It does not publish the complete private schema or prove production-scale concurrency.

## Sanitized Logical Tables

| Table | Public purpose | Example fields |
|---|---|---|
| `RULE_DEFINITION` | Stable identity and scope | Rule ID, Create GUID, Group GUID, Link GUID, scope, lifecycle status |
| `RULE_VALUE` | Development or Live value | Rule ID, environment, version, sanitized value, active flag |
| `RULE_HISTORY` | Previous/new value and reason | Rule ID, from/to version, reason, validation result, session, timestamp |
| `RULE_PROMOTION_LOG` | Development-to-Live attempt | promotion ID, Development version, expected Live version, outcome |

These field names are a public logical model. Types, constraints, indexes, and private operational columns are intentionally omitted.

## Fictional Promotion Transaction

```text
BEGIN IMMEDIATE
  verify active session ownership
  verify expected Live version = 12
  validate Development version = 13
  preserve Live version 12 in RULE_HISTORY
  update linked Live value to version 13
  record PASS in RULE_PROMOTION_LOG
COMMIT
```

If session ownership, expected version, required relationships, or validation fails, the transaction does not silently promote the rule.

## Similarity Review Example

```text
candidate_rule = RULE-DEMO-B
similar_to     = RULE-DEMO-A
similarity     = REVIEW
automatic_merge = PROHIBITED
reason         = Similar wording may still represent a different scope or exception
```

The project uses similarity to find review candidates, not as authority to merge or activate a rule.

## What Improved

- Development testing no longer required overwriting the active Live value.
- Promotion preserved the prior value and change reason.
- Sessions could detect an expected-version mismatch instead of silently overwriting newer work.
- History supported rollback analysis and cross-session diagnosis.

## Remaining Limits

- The public sample is not a runnable migration.
- Concurrent performance and stale-lock recovery are not fully measured.
- History retention and archive thresholds remain policy decisions.
- ChatGPT inference may recommend classification or similarity, but cannot silently modify Live.
