# H. 운영 통제·복구

> [!NOTE]
> **기준:** 2026-08-24. 개인 ChatGPT Plus 웹/Work, 직접 OpenAI API 호출 없음. 제품·플랜·권한·연결 앱 변경 후 재검증해야 합니다.
>
> [!CAUTION]
> 교육·기술 공유용 자료이며 보증하지 않습니다. 실제 적용 전 평가·테스트·보안·백업·법적 검토를 수행하십시오. [면책 조항](../../DISCLAIMER.md)을 참조하십시오.


## 도입 이유

여러 채팅·Work·작업이 동시에 Rule, 마스터, 레포트에 접근하면 중복 실행, 오래된 값 덮어쓰기, 작업 중단 후 이어가기 실패가 발생할 수 있습니다. 대화 기억을 운영 상태로 보지 않고 DB의 Rule·TASK·Session·History·Backup을 기준으로 통제하기 위해 도입했습니다.

## 기능

| 번호 | 기능 | 의미 |
|---|---|---|
| H-01 | Rule 관리 | Development 검증 후 Live 승격·History |
| H-02 | TASK 관리 | 정상·NG·HOLD·부분효과·이어가기 |
| H-03 | Session·Lock·Heartbeat | 중복 작업·오래된 덮어쓰기 방지 |
| H-04 | 백업 | 공식 원본·등록부·Manifest 보존 |
| H-05 | 장애 복구 | 마지막 검증 상태로 복원 |
| H-06 | 공개자료 생성 | 운영정보 제거한 MD·HTML·SVG 샘플 |
| H-07 | GitHub 게시 검사 | 주의문·근거·민감정보·구현 주장 검사 |

## 상태 모델

| 상태 | 의미 | 다음 처리 |
|---|---|---|
| REGISTERED | 작업 등록 | 우선순위에 따라 실행 |
| RUNNING | Session 소유 실행 중 | Heartbeat 갱신 |
| HOLD | 사용자 결정·자료 대기 | 사유와 이어가기 STEP |
| NG | 검증 실패 | 부작용·재시도 안전성 확인 |
| COMPLETED | 검증 완료 | 결과·History 고정 |
| CANCELLED | 승인 취소 | 실행 부작용 분리 기록 |

## 논리 Table

### RULE 계열

| Table | 핵심 필드 |
|---|---|
| RULE_DEFINITION | RULE_ID, GROUP_GUID, SCOPE, OWNER |
| RULE_VALUE | LINK_GUID, MODE(DEV/LIVE), VERSION, VALUE |
| RULE_HISTORY | 변경 전후, 변경 사유, 사용자 승인 |
| RULE_PROMOTION_LOG | DEV→LIVE 검증·승격 결과 |

### WORK_TASK 계열

| Table | 핵심 필드 |
|---|---|
| WORK_TASK | PRIORITY, PRIORITY_SEQ, STATUS, SCOPE, SESSION_ID |
| WORK_TASK_STEP | STEP_NO, STATUS, INPUT_VERSION, CONTINUE_POINT |
| WORK_TASK_RESULT | RESULT, NG_REASON, PARTIAL_EFFECT, RETRY_SAFE |

### SESSION_LOCK / BACKUP_MANIFEST

| 필드 | 의미 |
|---|---|
| RESOURCE_KEY | 잠금 대상 |
| SESSION_ID | 소유 세션 |
| ACQUIRED_AT/HEARTBEAT_AT | 소유·생존 시각 |
| LEASE_UNTIL | 잠금 만료 |
| EXPECTED_VERSION | 커밋 전 버전 |
| BACKUP_ID/SOURCE_VERSION | 백업과 원본 연결 |
| HASH/RESTORE_TEST_STATUS | 내용·복구 검증 |

## 처리 알고리즘

```text
task = register_task(priority, scope, idempotency_key)
session = create_session_guid()
lock = acquire(resource, session, expected_version)

while task.running:
    verify_lock_owner(session)
    heartbeat(session)
    execute_one_versioned_step()
    record result and continuation point

before_commit:
    assert lock_owner_and_version_unchanged()
    commit data + history
    mark step completed

if uncertain_user_decision:
    save HOLD(reason, continue_step, partial_effect)
if validation_failure:
    save NG(reason, retry_safe, partial_effect)

release lock
backup approved state
verify restore manifest
```

## 답변·Issue·TASK 처리 규칙

AI가 근거로 답할 수 있는 일반 기술 질문은 직접 답변합니다. 사용자 결정, 민감정보 공개, 운영 DB 변경, 핵심 알고리즘 공개, 불명확한 외부 동작처럼 임의 답변이 위험한 내용은 TASK에 HOLD로 등록하고 사용자와 협의합니다. GitHub Issue는 공개 토론이 필요한 독립 주제에만 사용하며 내부 작업 큐를 대신하지 않습니다.

## 어려움·문제·개선

- 대화가 길어지거나 새 창으로 이동하면 현재 STEP과 결과가 유실됐습니다. TASK STEP과 CONTINUE_POINT를 저장합니다.
- NG인지 정상 완료인지 구분하지 않으면 나중에 실패 원인을 찾을 수 없습니다. 처리 결과·NG 사유·부분효과를 별도 저장합니다.
- 두 세션이 같은 값을 수정할 수 있어 Session GUID, Lock, Heartbeat, 커밋 전 버전 검사를 도입했습니다.
- 규칙을 메모리에만 두면 작은 변경 후 원복되거나 오래된 규칙과 섞일 수 있습니다. Development·Live·History DB를 권위 원본으로 둡니다.
- 백업 파일이 있다는 것만으로 복구 가능하다고 할 수 없습니다. Manifest와 복원 순서·복구 테스트를 기록합니다.
- GitHub에 실제 구현처럼 과장된 설계가 섞일 수 있습니다. 구현 완료·테스트 중·설계만 존재를 구분하고 전 문서 공통 주의문과 공개 검사를 적용합니다.

## 장점·단점

장점은 중단된 작업을 이어가고, 누가 어떤 버전을 변경했는지 추적하며, NG와 부분 부작용을 복구할 수 있다는 점입니다. 단점은 상태·잠금·History·백업 관리 비용이 커지고 완전한 분산 트랜잭션을 ChatGPT 대화만으로 보장할 수 없다는 점입니다.

## 완료 기준과 현재 한계

Rule 승격은 회귀검사와 사용자 승인 후에만 완료합니다. TASK는 결과와 부작용, 이어가기 지점까지 기록해야 완료입니다. Backup은 실제 복원 검증 전에는 완료로 주장하지 않습니다. ChatGPT Memory·대화·Project는 편의 계층이며 잠금 가능한 운영 DB가 아닙니다.
