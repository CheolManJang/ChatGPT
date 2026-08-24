# D. 사용자 프로그램 데이터 수집·전달

> [!NOTE]
> **기준:** 2026-08-24. 개인 ChatGPT Plus 웹/Work, 직접 OpenAI API 호출 없음. 제품·플랜·권한·연결 앱 변경 후 재검증해야 합니다.
>
> [!CAUTION]
> 교육·기술 공유용 자료이며 보증하지 않습니다. 실제 적용 전 평가·테스트·보안·백업·법적 검토를 수행하십시오. [면책 조항](../../DISCLAIMER.md)을 참조하십시오.


## 도입 이유

AI가 한국 시장과 키움 OpenAPI의 전체 종목 원본을 직접·지속적으로 수집한다고 가정하면 권한, 세션, 시장 마감 시점, 누락과 재현성 문제가 생깁니다. 사용자의 Delphi/키움 프로그램이 원시자료를 수집하고 AI는 등록된 원본을 검증·분석하도록 역할을 나눴습니다.

## 기능

| 번호 | 기능 | 책임 |
|---|---|---|
| D-01 | 시장 마감 확인 | 정규시장·NXT·거래일 확정 |
| D-02 | 키움 OpenAPI 수집 | 사용자 프로그램이 일·월·년·상태 수집 |
| D-03 | 원시데이터 생성 | 기준일·생성시각·버전·성공/실패 포함 |
| D-04 | 자체검증 | 누락·중복·자료형·거래일 검사 |
| D-05 | AI 전달 | 사용자가 공식 위치 등록 후 결과 요청 |
| D-06 | 원본 식별 | AI가 FILE_REGISTRY의 ID·버전·역할 확인 |
| D-07 | AI 무결성 검사 | 마스터 수·수집 수·기간·상태 전수검사 |
| D-08 | 재수집 요청 | 누락을 미도달로 처리하지 않고 보완 요청 |

## 처리 STEP

1. 한국 거래일과 정규시장 종료 확인
2. NXT 대상과 최종 마감 시점 확인
3. 사용자 프로그램이 대상 목록 버전을 고정
4. 종목별 일·월·년 자료와 거래 상태 수집
5. 성공·실패·누락 목록 생성
6. 자료형·중복·날짜·종목 수 자체검증
7. 원본 패키지·Manifest·해시 생성
8. 사용자가 공식 위치에 등록
9. AI가 FILE_REGISTRY와 일치 여부 확인
10. 기대 건수와 유효 건수 전수검사
11. 부족하면 D-08 재수집 TASK로 전환
12. 완전한 원본만 분석 단계에 전달

## 논리 Table

### RAW_PACKAGE_MANIFEST

| 필드 | 의미 |
|---|---|
| PACKAGE_ID | 전달 패키지 키 |
| BASE_DATE | 시장 기준일 |
| MARKET_CLOSE_TYPE | KRX/NXT/예외 |
| MASTER_VERSION | 수집 대상 버전 |
| EXPECTED_COUNT | 기대 종목 수 |
| SUCCESS_COUNT/FAIL_COUNT | 수집 결과 |
| GENERATED_AT | 생성 시각 |
| CONTENT_HASH | 원본 해시 |
| STATUS | CREATED/VALIDATED/REJECTED |

### PRICE_DATA / STOCK_STATUS_RAW

| 필드 | 의미 |
|---|---|
| PACKAGE_ID/STOCK_ID | 원본과 종목 |
| TIMEFRAME | DAILY/MONTHLY/YEARLY |
| TRADE_DATE | 거래일 |
| OHLCV | 가격·거래량 묶음 |
| SESSION_TYPE | KRX/NXT |
| SUSPENSION_STATUS | 거래정지 상태 |
| LISTING_RISK | 관리·상폐 위험 원시값 |
| SOURCE_CODE | 수집 API/화면 식별 |
| FETCH_STATUS/ERROR_CODE | 성공·실패 |

## 알고리즘

```text
target_snapshot = freeze_master_version()
for stock in target_snapshot:
    collect daily, monthly, yearly, market_state
    record success_or_error(stock)

manifest = summarize_collection()
if duplicate/date/type/count errors:
    reject_before_transfer()
else:
    register_package_with_hash()

AI:
resolve exact package from FILE_REGISTRY
if manifest.expected != valid_unique_rows:
    create_recollection_task(missing_or_invalid)
    stop; never convert missing to NOT_REACHED
```

## 어려움·문제·개선

- KRX 종가와 NXT 최종가의 시점이 달라 조기 실행 결과가 바뀌었습니다. 시장 세션과 마감 유형을 원본에 기록합니다.
- 대용량 파일은 업로드·인식이 지연되고 일부만 읽힐 수 있습니다. Manifest·건수·해시로 완전성을 먼저 확인합니다.
- API 오류를 빈 값으로 저장하면 미도달로 오판할 수 있습니다. 값과 FETCH_STATUS를 분리합니다.
- 종목명 변경 때문에 조인이 실패할 수 있어 코드와 시장을 키로 사용합니다.
- AI가 직접 수집했다고 잘못 표현하지 않도록 사용자 프로그램 수집과 AI 검증 책임을 문서에 명확히 고정했습니다.

## 장단점과 완료 기준

장점은 수집을 재현하고 실패 종목을 정확히 재수집할 수 있다는 점입니다. 단점은 사용자 프로그램 운영과 전달 절차가 필요하고 NXT 마감까지 기다려야 한다는 점입니다. 기대 건수, 성공 건수, 유일 종목 수, 기간 자료, 상태값, 해시가 모두 맞아야 완료입니다.
