# F. 특수 검증 기능

> [!NOTE]
> **기준:** 2026-08-24. 개인 ChatGPT Plus 웹/Work, 직접 OpenAI API 호출 없음. 제품·플랜·권한·연결 앱 변경 후 재검증해야 합니다.
>
> [!CAUTION]
> 교육·기술 공유용 자료이며 보증하지 않습니다. 실제 적용 전 평가·테스트·보안·백업·법적 검토를 수행하십시오. [면책 조항](../../DISCLAIMER.md)을 참조하십시오.


## 도입 이유

종목은 한 번 등록했다고 계속 적합한 것이 아닙니다. 재무 악화, CB·유증, 관리·거래정지, 상장변경, 가격 구조 붕괴가 생길 수 있고 Rule·Logic의 작은 변경이 과거 오류나 폐기된 출력을 되살릴 수 있습니다. 일반 레포트 실행과 별도로 재검증 기능을 둔 이유입니다.

## 기능

| 번호 | 기능 | 발동 조건 | 결과 |
|---|---|---|---|
| F-01 | 종목 재검증 | 사용자 요청·오래된 검증 | 유지/변경/제외/HOLD |
| F-02 | 이벤트 재검증 | CB·유증·관리·정지·상장변경 | 즉시 위험 재평가 |
| F-03 | 정기 재검증 | 기준 유효기간 만료 | 재무·등급 갱신 |
| F-04 | 타점 재검증 | 구조 붕괴·가격 체계 변화 | 감시가·매도선 재평가 |
| F-05 | 등급 재분류 | 위험·재무·구조 변경 | 등급 상향/하향/X |
| F-06 | 마스터 일관성 | 원본·등록부·세션 불일치 | 충돌 복구/HOLD |
| F-07 | 누락·중복 복구 | 건수·키 이상 | 원본 기준 정합화 |
| F-08 | 작은 변경 회귀검사 | Rule·형식·Logic 수정 | 원복·부작용 탐지 |
| F-09 | 전체 재계산 | Logic·Rule·마스터 변경 | 오래된 결과 폐기 |
| F-10 | 확정 불가 | 필수 근거 미확보 | NG/HOLD 보존 |

## 처리 STEP

1. 요청·이벤트·유효기간·회귀실패로 재검증 TASK 생성
2. 기존 판정과 근거 버전 동결
3. 최신 승인 자료 수집
4. 재무·자본·상장·거래 상태 재검사
5. 가격 구조와 감시·매도 단계 재검사
6. 기존 결과와 변경 원인 비교
7. 등급·분류·기준 변경안 생성
8. 영향받는 전체 종목·레포트 범위 계산
9. 사용자 승인 또는 HOLD
10. 승인 후 History와 새 버전 반영
11. 전체 재계산·회귀검사
12. TAG 없는 Live 결과 동일성 확인

## 논리 Table

### STOCK_REVALIDATION

| 필드 | 의미 |
|---|---|
| REVALIDATION_ID | 재검증 키 |
| STOCK_ID | 대상 |
| TRIGGER_TYPE | USER/EVENT/PERIODIC/REGRESSION |
| OLD_VALIDATION_VERSION | 이전 판정 |
| NEW_SOURCE_VERSION | 새 근거 |
| CHANGE_SET | 변경 항목 |
| PROPOSED_GRADE/CLASS | 제안 |
| STATUS | RUNNING/HOLD/NG/APPROVED/APPLIED |
| APPROVAL_ID | 사용자 승인 |
| IMPACT_SCOPE | 재계산 범위 |

### REGRESSION_CASE

| 필드 | 의미 |
|---|---|
| CASE_ID | 회귀 사례 |
| LOGIC_CODE | 담당 Logic |
| INPUT_FIXTURE_VERSION | 가상 고정 입력 |
| EXPECTED_CONTRACT | 기대 필드·상태 |
| FORBIDDEN_OUTPUT | 폐기 TAG·구형 필드 |
| RESULT | PASS/FAIL |
| DIFFERENCE | 변경 차이 |

## 알고리즘

```text
trigger = detect_revalidation_trigger()
baseline = freeze_previous_approved_state()
current = collect_and_validate_current_evidence()

proposal = compare(baseline, current)
impact = find_all_dependent_rules_and_reports(proposal)

if evidence_incomplete:
    HOLD with missing evidence; do not preserve old grade as newly verified
else if material_change:
    request_user_approval(proposal, impact)

after approval:
    write_new_version_and_history()
    recalculate_full_impacted_population()
    run_regression_cases()
    reject release if retired_TAG_or_old_format_returns
```

## TAG 도입·제거 경험

레포트의 여러 Logic 중 어느 Logic이 잘못된 행을 만들었는지 찾기 위해 Development 결과에 TAG를 붙였습니다. TAG는 담당 Logic, 불러올 규칙·예제, 수정 금지 영역과 비교 결과를 빠르게 찾는 진단 정보였습니다. 덕분에 전체 레포트를 반복 수정하지 않고 문제 Logic만 보완할 수 있었습니다.

하지만 계속 사용하려 했을 때 TAG가 문맥과 출력 부담을 늘렸고, 작은 수정 뒤 제거한 TAG나 예전 형식이 다시 나타났습니다. 규칙을 더 정교하게 작성해도 AI가 컴파일된 프로그램처럼 항상 동일하게 실행되지 않아 완전한 해결이 되지 않았습니다. Logic 보완과 최적화가 끝난 뒤 TAG는 목적을 달성한 임시 계측으로 판단해 Live에서 제거했습니다.

정리 기준은 “계속 규칙을 추가하면 반드시 해결된다”가 아닙니다. 같은 회귀가 반복되고 결정적 검사로 통제할 수 없는 부분은 현재 AI의 한계로 기록합니다. 검증 가능한 최소 Logic만 Live에 남기고 자동화가 위험한 부분은 HOLD 후 사용자와 협의합니다.

## 장점·단점·개선

장점은 오래된 판단과 숨은 회귀를 발견하고 변경 영향 범위를 추적하는 것입니다. 단점은 전체 재계산과 회귀검사 비용이 크고 이벤트 원본 확보가 늦을 수 있다는 것입니다. TAG를 영구 기능으로 만들지 않고 진단 후 제거하여 Live 출력 부담을 줄였습니다.

## 완료 기준

재검증 근거, 변경 비교, 영향 범위, 사용자 승인, 새 버전·History, 전체 재계산, 회귀 PASS가 모두 필요합니다. 근거 부족이나 반복 회귀는 완료가 아니라 HOLD/NG입니다.
