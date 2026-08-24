# Sanitized TASK Implementation Sample
> [!NOTE]
> **Document baseline:** 2026-08-24. Reference environment: individual ChatGPT Plus account using ChatGPT web/Work without direct OpenAI API calls. Architectural principles are general; observed behavior depends on the tested plan, context, permissions, connected apps, and rollout. Revalidate after material product changes.
> **기준:** 2026년 8월 24일. 참조 환경: ChatGPT 웹/Work를 사용하며 직접 OpenAI API를 호출하지 않는 개인 ChatGPT Plus 계정. 아키텍처 원칙은 일반적이지만, 관찰된 동작은 테스트한 플랜, 맥락, 권한, 연결된 앱 및 배포 상태에 따라 달라집니다. 중요한 제품 변경 후 다시 검증하십시오.

> [!CAUTION]
> **Use at your own risk.** This material is provided for technical education and general information only, without warranties. Evaluate, test, secure, back up, and legally review any implementation. See the [Disclaimer](../DISCLAIMER.md).
> **사용 시 주의.** 본 자료는 교육 및 일반 정보 제공 목적이며, 어떠한 보증도 제공하지 않습니다. 모든 구현을 직접 평가·테스트하고, 보안과 백업을 확인하며, 필요한 법적 검토를 수행하십시오. [면책 조항](../DISCLAIMER.md)을 참조하십시오.


> [!NOTE]
> **Sample version:** `TASK-SAMPLE-1.0`
> **Document baseline:** 2026-08-24
> **Related feature status:** bounded private implementation in use; recovery hardening in Development
> **Evidence class:** reconstructed fictional example based on project implementation experience
> All identifiers, timestamps, descriptions, and results are fictional. This is not a private queue export.

## What This Sample Shows

It shows why work status, execution result, NG/HOLD reason, ownership, and continuation must be stored separately. It does not prove that every connector side effect is transactionally reversible or that every state transition is fully automated.

## Fictional TASK Record

```yaml
task_no: TASK-DEMO-0042
title: Validate sanitized report sample
work_area: DOCUMENTATION
priority: 2
priority_sequence: 10
status: HOLD
status_reason: User decision required for public wording
owner_session_guid: SESSION-DEMO-B
required_order: true
duplicate_prohibited: true
work_description: Compare the sample against the approved publication boundary
result_description: Baseline and fictional-data labels verified; one wording decision remains
partial_effects: Draft updated locally; no external publication performed
retry_safe: true
continuation_point: Resume after user selects the public wording
validation_evidence: CHECK-DEMO-07
```

## Reconstructed State History

| Step | State | Recorded result |
|---|---|---|
| 1 | Registered | Scope, priority, acceptance criteria, and duplicate check recorded |
| 2 | In Progress | Session acquired ownership and began validation |
| 3 | HOLD | Material wording decision assigned to user; partial effects recorded |
| 4 | In Progress | New session verified the prior result and resumed the same TASK |
| 5 | Completed | Change and validation evidence recorded; no remaining continuation |

## NG Contrast

```yaml
status: NG
stop_point: External publication verification
cause: Permission or response could not be confirmed
partial_effects: Local change exists; remote state unknown
retry_safe: false
next_action: Verify remote state before any retry
```

The important implementation lesson is that `NG` is not “not completed.” It must preserve the exact stop point, partial effects, retry safety, and next safe action.

## Publication Boundary

Never publish the real TASK number, owner session, private work description, recipients, repository credentials, connector IDs, local paths, or operational result data.
