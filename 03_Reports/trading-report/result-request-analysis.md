# E. 결과 요청 및 분석

> [!NOTE]
> **Document baseline:** 2026-08-24. Reference environment: individual ChatGPT Plus account using ChatGPT web/Work without direct OpenAI API calls. Architectural principles are general; observed behavior depends on the tested plan, context, permissions, connected apps, and rollout. Revalidate after material product changes.
>
> **기준:** 2026년 8월 24일. 참조 환경: ChatGPT 웹/Work를 사용하며 직접 OpenAI API를 호출하지 않는 개인 ChatGPT Plus 계정. 아키텍처 원칙은 일반적이지만, 관찰된 동작은 테스트한 플랜, 맥락, 권한, 연결된 앱 및 배포 상태에 따라 달라집니다. 중요한 제품 변경 후 다시 검증하십시오.

> [!CAUTION]
> **Use at your own risk.** This material is provided for technical education and general information only, without warranties. Evaluate, test, secure, back up, and legally review any implementation. See the [Disclaimer](../../DISCLAIMER.md).
>
> **사용 시 주의.** 본 자료는 교육 및 일반 정보 제공 목적이며 어떠한 보증도 제공하지 않습니다. 모든 구현을 직접 평가·테스트하고 보안·백업·필요한 법적 검토를 수행하십시오. [면책 조항](../../DISCLAIMER.md)을 참조하십시오.


## 1. 도입 이유

종목 수가 늘면서 단순 대화형 분석은 처리 속도가 느려지고, 일부 자료 누락을 미도달로 오판하거나 이전 결과를 재사용할 위험이 생겼습니다. 이 기능군은 사용자의 한 번의 결과 요청을 추적 가능한 TASK로 만들고, 전체 대상·원본·판정·완전성을 단계별로 검증하기 위해 도입했습니다.

## 2. 기능 목록

| 번호 | 기능 | 의미 |
|---|---|---|
| E-01 | 결과 생성 요청 | 사용자가 전달된 최신 자료를 기준으로 매매 레포트 결과 요청 |
| E-02 | 실행 TASK 생성 | 중복 실행을 막고 기준일·원본·Session·진행 STEP 기록 |
| E-03 | 전체 대상 구성 | 최신 마스터에서 활성 대상 구성 후 보유·제외 종목 처리 |
| E-04 | 전수검사 | 전체 대상의 필수 데이터 확보 여부를 확인하고 일부 실패 시 확정 중단 |
| E-05 | 감시가 도달 판정 | 단계별 감시 기준에 대한 당일 도달 여부 계산 |
| E-06 | 30% 근접 판정 | 미도달 대상 중 감시가 근접 항목 계산·정렬 |
| E-07 | 등급·시총·배당 반영 | 등록 등급과 공개 확인정보를 결과에 결합 |
| E-08 | 거래정지·위험 표시 | 거래 불가 또는 위험 상태를 정상 후보와 구분 |
| E-09 | 정밀판정 | 일반 판정으로 확정하기 어려운 항목을 추가 자료로 재평가 |
| E-10 | 결과 완전성 검사 | 건수·행·정렬·중복·보유 제외·필수 필드 확인 |

## 3. 사용자와 AI의 역할

| 단계 | 사용자 | AI |
|---|---|---|
| 요청 전 | 사용자 프로그램으로 시장 마감 자료를 수집·전달 | 전달 원본의 파일·버전·기준일 확인 |
| 실행 | 결과 생성 요청 | TASK·Session 생성, 대상 구성, 전수검사 |
| 판단 | 필요 시 정밀판정 자료와 승인 제공 | 도달·근접·등급·위험·완전성 판정 |
| 예외 | 재수집·보완 또는 HOLD 해제 결정 | 누락을 미도달로 바꾸지 않고 NG/HOLD 기록 |
| 완료 | 최종 결과와 이메일 수신 확인 | 검증 패키지 생성 후 G 기능군에 전달 |

## 4. 처리 STEP

1. 승인된 사용자와 최신 입력 원본 확인
2. 동일 기준일·원본의 중복 실행 여부 확인
3. REPORT_RUN과 WORK_TASK 생성
4. 활성 마스터에서 기준 대상 구성
5. 보유·비활성·제외 대상을 규칙에 따라 분리
6. 대상 수와 가격·상태·분석자료를 전수검사
7. 하나라도 필수 자료가 없으면 결과 확정을 중단
8. 감시가 단계별 도달 여부 판정
9. 미도달 종목의 30% 근접률 계산·정렬
10. 등급·시총·배당·거래정지·위험 상태 결합
11. 애매한 항목을 정밀판정 또는 HOLD로 분리
12. 건수·행·정렬·중복·보유 제외·필수 필드 재검사
13. 불변 결과 패키지를 만들고 레포트 생성 단계로 전달
14. TASK STEP과 결과·NG/HOLD·이어가기 위치 기록

## 5. 논리 Table 구조

공개용 논리 모델이며 실제 운영 Table과 이름·필드는 다를 수 있습니다.

### REPORT_RUN

| 필드 | 의미 |
|---|---|
| REPORT_RUN_ID | 실행 식별자 |
| USER_ID | 요청 사용자 |
| BASE_DATE | 시장 기준일 |
| SOURCE_FILE_ID / SOURCE_VERSION | 검증된 입력 원본 |
| MASTER_VERSION / RULE_VERSION | 사용한 마스터·Rule 버전 |
| SESSION_ID | 실행 소유 세션 |
| STATUS | REQUESTED/RUNNING/HOLD/NG/VALIDATED/COMPLETED |
| EXPECTED_COUNT / VALID_COUNT | 대상 수와 검증 완료 수 |
| STARTED_AT / COMPLETED_AT | 실행 시각 |
| CONTINUE_STEP | 중단 후 이어갈 안전 지점 |
| RESULT_SUMMARY | 공개 가능한 처리 결과 |

### REPORT_RESULT

| 필드 | 의미 |
|---|---|
| REPORT_RUN_ID | 실행 연결 |
| ITEM_KEY | 공개용 가상 종목 식별자 |
| WATCH_LEVEL | 판정 대상 감시 단계 |
| REACHED_YN | 도달 여부 |
| NEAR_RATE | 미도달 시 근접률 |
| GRADE | 승인된 등급 |
| MARKET_CAP_STATUS | 적합/부적합/정밀 |
| DIVIDEND_STATUS | 확인된 공개 배당정보 |
| TRADING_STATUS | 정상/정지/위험/확인필요 |
| DECISION_STATUS | CONFIRMED/PRECISION/HOLD/NG |
| SOURCE_TRACE | 판정에 사용한 원본 추적값 |

### WORK_TASK_STEP

| 필드 | 의미 |
|---|---|
| TASK_ID / STEP_NO | 작업과 순서 |
| STEP_CODE | E-01~E-10 |
| STATUS | WAIT/RUNNING/COMPLETED/HOLD/NG |
| INPUT_VERSION | 단계 입력 버전 |
| RESULT_TEXT | 수행 결과 |
| ERROR_CODE / ERROR_DETAIL | 실패 원인 |
| RETRY_SAFE_YN | 재실행 안전 여부 |
| STARTED_AT / COMPLETED_AT | 단계 실행 시각 |

## 6. 핵심 알고리즘

```text
request := validate_user_request()
source := resolve_registered_source(request.source_id, request.version)
task := create_idempotent_task(request.user, source, request.base_date)

targets := active_master(source.master_version)
targets := apply_holding_and_exclusion_rules(targets)

coverage := validate_all_required_data(targets, source)
if coverage.valid_count != coverage.expected_count:
    save_hold_or_ng(task, missing_items, continue_step="E-04")
    stop_without_confirmed_report()

for each target in targets:
    reached := evaluate_watch_levels(target)
    if not reached:
        near_rate := calculate_near_rate(target)
    merge_approved_grade_and_public_status(target)
    classify_trading_risk(target)
    route_uncertain_items_to_precision_or_hold(target)

result := validate_result_integrity(
    expected_count,
    rows,
    sort_order,
    duplicates,
    holding_exclusions,
    required_fields
)

freeze_validated_package(result, source.version, rule.version)
handoff_to_report_generation(result)
```

공개 문서에서는 실제 매매 타점을 재구성할 수 있는 핵심 계산식과 실제 임계값 조합을 공개하지 않습니다.

## 7. 검증·완료 기준

- 요청 사용자와 승인 이메일의 연결이 유효함
- 기준일과 시장 마감 조건이 확정됨
- 입력 파일 ID·버전·해시가 등록 원본과 일치함
- 예상 대상 수와 검증 대상 수가 일치함
- 보유·비활성·제외 종목 처리가 중복 없이 적용됨
- 모든 대상에 필수 가격·상태·분석자료가 존재함
- 도달과 근접이 동시에 확정되지 않음
- 근접 목록이 승인된 계산값과 정렬 순서를 따름
- 거래정지·위험·정밀 항목이 정상 후보와 구분됨
- 결과 행 수·요약 건수·HTML 전달 건수가 일치함
- 사용한 마스터·Rule·원본 버전을 결과에서 추적할 수 있음

## 8. 어려웠던 점과 실제 문제

### 속도 문제

전체 종목의 일·월·년 자료와 시장 상태를 한 번에 확인하면서 단계별 검증이 누적되어 처리 시간이 길어졌습니다. 속도를 줄이기 위해 검증을 생략하면 결과 신뢰성이 떨어지므로, TASK STEP과 Logic 단위로 병목을 분리하고 재사용 가능한 확정 기준정보와 매 실행 검증정보를 구분했습니다.

### 누락을 미도달로 오판하는 문제

외부 자료 조회 실패나 일부 원본 누락을 단순 미도달로 처리하면 사용자는 정상적인 전수검사 결과로 오해할 수 있습니다. 예상 건수와 유효 건수가 다르면 전체 결과 확정을 중단하고 NG/HOLD 및 누락 목록을 남기도록 바꿨습니다.

### 작은 변경 후 이전 형식이 복원되는 문제

Rule을 자세히 작성해도 AI가 문맥과 출력 구조를 매번 재구성하기 때문에 폐기된 TAG나 이전 형식이 다시 나타날 수 있었습니다. 이를 현재 AI의 한계로 인정하고, 결정적 검사와 회귀 샘플로 통제하되 같은 현상이 반복되면 무한히 규칙을 늘리지 않고 안정된 Logic까지만 유지합니다.

### 여러 세션의 충돌

다른 채팅이나 작업이 동일한 기준정보를 수정하면 오래된 결과가 최신 결과를 덮을 수 있습니다. Session ID, Lock, Heartbeat, 입력 버전 확인과 커밋 직전 재검증을 추가했습니다.

## 9. 장점·단점·개선 결과

| 구분 | 내용 |
|---|---|
| 장점 | 전체 대상 누락을 조기에 탐지하고 결과의 원본·Rule·마스터 버전을 추적 가능 |
| 장점 | 사용자 요청부터 이메일 전달까지 TASK로 이어갈 수 있음 |
| 장점 | 정밀·HOLD·NG를 정상 결과와 분리하여 거짓 확정을 줄임 |
| 단점 | 전수검사와 회귀검사 때문에 처리 시간이 증가 |
| 단점 | 외부 자료와 AI 추론은 같은 입력에서도 완전한 결정성을 보장하지 못함 |
| 단점 | 사용자가 원본 재수집이나 HOLD 판단에 참여해야 하는 경우가 있음 |
| 개선 | 일부 조회 실패를 미도달로 처리하던 흐름을 확정 중단 방식으로 변경 |
| 개선 | TAG로 Logic 병목을 진단·최적화한 뒤 Live 출력에서는 TAG 제거 |
| 개선 | 작은 변경 후 원복을 회귀검사 대상으로 등록하고 전체 재계산 원칙 적용 |

## 10. AI 한계와 정리 기준

다음 상황이 반복되면 규칙 문구를 계속 늘리는 것으로 해결하려 하지 않습니다.

- 동일한 작은 변경 후 폐기 기능이나 이전 출력이 반복 복원됨
- 규칙 간 우선순위가 문맥에 따라 달라짐
- 입력 원본·권한·연결 앱 상태를 확정할 수 없음
- 추론 결과를 결정적 검사로 재현하거나 검증할 수 없음
- 검증 비용이 기능의 실익보다 커짐

이 경우 현재 AI의 한계로 기록하고, 이미 최적화된 Logic과 검증 가능한 최소 기능만 Live에 남깁니다. 자동 확정이 위험하면 TASK를 HOLD로 전환하여 사용자와 협의합니다.

## 11. 구현 상태

| 항목 | 상태 |
|---|---|
| 기능 목록·처리 흐름·논리 모델 | 개발 경험 기반 공개 설계 |
| Rule/TASK/Session 구조 | 별도 문서에 구현 경험과 샘플 존재 |
| 전수검사·누락 시 확정 중단 원칙 | 실제 실패 경험에서 도출 |
| 매매 레포트 전체 운영 DB | 공개 저장소에 구현 완료를 주장하지 않음 |
| 키움 OpenAPI 직접 연동 | 사용자 프로그램 영역이며 AI 직접 연동으로 주장하지 않음 |
| 이메일 전송 | 별도 전달 모듈과 사용자 요청·승인 경계 적용 |
| 반복 무인 운영 | 계정·권한·연결·제품 변경에 따라 재검증 필요 |

## 12. 공개 범위

예시는 가상 식별자와 일반화된 상태만 사용합니다. 실제 종목, 가격, 보유내역, 계좌, 이메일, 원시데이터, 운영 DB, 인증정보와 핵심 매매 알고리즘은 공개하지 않습니다.
