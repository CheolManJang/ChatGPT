# B. 종목 마스터 관리

> [!NOTE]
> **기준:** 2026-08-24. 개인 ChatGPT Plus 웹/Work, 직접 OpenAI API 호출 없음. 제품·플랜·권한·연결 앱 변경 후 재검증해야 합니다.
>
> [!CAUTION]
> 교육·기술 공유용 자료이며 보증하지 않습니다. 실제 적용 전 평가·테스트·보안·백업·법적 검토를 수행하십시오. [면책 조항](../../DISCLAIMER.md)을 참조하십시오.


## 도입 이유

사용자가 관심을 가진 종목을 바로 마스터에 넣으면 재무 악화, 거래정지·상장폐지 위험, 반복 CB·유증, 장기 가격 구조 부재 때문에 실제 매매 관리가 어려워질 수 있습니다. 따라서 사용자 1차 선택과 AI 사전검증, 사용자 최종승인을 분리했습니다.

## 기능

| 번호 | 기능 | 핵심 처리 |
|---|---|---|
| B-01 | 종목 추가 요청 | 사용자 후보 접수 |
| B-02 | 기본 식별 | 코드·시장·상장상태·중복 확인 |
| B-03 | 적합성 검증 | 매출·영업이익·순이익·적자/흑자·재무·CB/유증·정지/상폐 위험 |
| B-04 | 타점 형성 검증 | 년·월·주·일 자료에서 관리 가능한 가격 구조 확인 |
| B-05 | 등급 분류 | A++~C, X, 정밀, HOLD 분류 |
| B-06 | 사용자 승인 | 등록·보류·제외 결정 |
| B-07 | 마스터 등록 | MASTER/WATCH/PRE-WATCH 반영 |
| B-08 | 감시가 | 단계별 기준 등록·변경 |
| B-09 | 매도선 | 단계별 대응 구간 등록·변경 |
| B-10 | 종목 변경 | 승인 변경과 History |
| B-11 | 제외·비활성 | 삭제 대신 추적 상태 전환 |
| B-12 | 중복·누락 | 마스터·관찰·보유·제외 일관성 검사 |

## 처리 STEP

1. 사용자가 후보를 1차 확인하고 AI에 추가 요청
2. 종목명·코드·시장·상장 상태 확인
3. 기존 마스터·관찰·보유·제외 목록 중복검사
4. 최신 재무와 자본조달·거래 위험 자료 확인
5. 년·월·주·일 구조에서 타점이 반복·검증 가능한지 판단
6. 적합·부적합·정밀·HOLD 및 등급 제안
7. 부적합 사유와 불확실성을 사용자에게 설명
8. 사용자 승인 후 마스터 분류와 감시가·매도선 등록
9. History와 검증 근거 버전 기록
10. 레포트 대상 일관성 검사

## 논리 Table

### STOCK_MASTER

| 필드 | 의미 |
|---|---|
| STOCK_ID | 내부 종목 키 |
| MARKET/CODE | 시장과 코드 |
| DISPLAY_NAME | 공개 시 가상화 가능한 표시명 |
| CLASS_CODE | MASTER/WATCH/PRE_WATCH/EXCLUDED |
| GRADE | 승인 등급 |
| ELIGIBILITY | SUITABLE/UNSUITABLE/PRECISION/HOLD |
| ACTIVE_YN | 레포트 대상 여부 |
| VALIDATION_VERSION | 마지막 검증 버전 |
| APPROVED_BY/APPROVED_AT | 사용자 승인 추적 |

### STOCK_VALIDATION

| 필드 | 의미 |
|---|---|
| VALIDATION_ID | 검증 키 |
| STOCK_ID | 대상 |
| FINANCIAL_STATUS | 재무 판정 |
| CAPITAL_EVENT_STATUS | CB·유증 등 |
| LISTING_RISK | 거래정지·관리·상폐 위험 |
| ENTRY_STRUCTURE_STATUS | 타점 구조 판정 |
| PROPOSED_GRADE | AI 제안 등급 |
| DECISION/REASON | 결과와 사유 |
| SOURCE_DATE | 근거 기준일 |

### STOCK_WATCH_PRICE / STOCK_SELL_LINE

| 필드 | 의미 |
|---|---|
| STOCK_ID | 대상 |
| LEVEL_NO | 1·2·3 단계 |
| VALUE_TOKEN | 비공개 실제값 참조 |
| BASIS | 구조의 근거 |
| STATUS | DEV/LIVE/RETIRED |
| VALID_FROM/VALID_TO | 유효 기간 |
| APPROVAL_ID | 사용자 승인 연결 |

## 알고리즘

```text
candidate = resolve_stock_identity(user_request)
if duplicate(candidate): return existing_record_for_review

risk = validate_financial_and_listing_sources(candidate)
structure = evaluate_manageable_price_structure(candidate)

if risk.has_critical_exclusion:
    proposal = X / UNSUITABLE
else if source_missing or structure_uncertain:
    proposal = PRECISION or HOLD
else:
    proposal = grade(candidate, risk, structure)

present_reasoned_proposal_to_user(proposal)
if user_approved:
    write_master_and_levels_in_one_versioned_change()
    append_history()
else:
    preserve_as_rejected_or_hold()
```

## 어려움·문제·개선

- 종목명이 바뀌거나 합병·시장 이전이 있으면 이름만으로 중복을 놓쳤습니다. 코드·시장·변경 이력을 함께 봅니다.
- 흑자 여부 하나만으로 안전성을 판단할 수 없었습니다. 영업이익·순이익·현금흐름·자본조달·상장 상태를 분리했습니다.
- 가격이 하락했다는 이유만으로 타점이 생기는 것은 아닙니다. 장기 우하향·저점 갱신·구조 미형성은 제외 또는 정밀판정합니다.
- AI 판단은 자료와 문맥에 따라 흔들릴 수 있습니다. 사용자의 최종승인과 근거 버전, 회귀검사를 필수화했습니다.
- 변경 시 기존 값을 덮어쓰면 과거 레포트가 설명되지 않았습니다. 유효기간과 History를 추가했습니다.

## 장점·단점·개선 결과

장점은 위험 종목의 무분별한 등록을 줄이고 등급·감시가·매도선의 근거를 추적할 수 있다는 점입니다. 단점은 자료 수집과 정밀검증 시간이 들고 공개정보가 늦거나 불완전할 수 있다는 점입니다. 자동 등록보다 사용자 승인형 구조가 느리지만 잘못된 마스터 반영 위험을 낮췄습니다.

## 완료 기준

종목 식별, 필수 재무·위험, 타점 구조, 등급 제안, 사용자 승인, 마스터/감시가/매도선 반영, History 기록이 모두 끝나야 완료입니다. 하나라도 불확실하면 등록 완료로 표시하지 않습니다.
