# G. 레포트 생성 및 이메일 전달

> [!NOTE]
> **기준:** 2026-08-24. 개인 ChatGPT Plus 웹/Work, 직접 OpenAI API 호출 없음. 제품·플랜·권한·연결 앱 변경 후 재검증해야 합니다.
>
> [!CAUTION]
> 교육·기술 공유용 자료이며 보증하지 않습니다. 실제 적용 전 평가·테스트·보안·백업·법적 검토를 수행하십시오. [면책 조항](../../DISCLAIMER.md)을 참조하십시오.


## 도입 이유

분석 화면이 정상이어도 HTML 변환, 첨부 선택, 수신자 선택, 중복 전송에서 별도 오류가 발생했습니다. “결과 생성”, “파일 생성”, “발송 요청”, “실제 발송”, “사용자 수신”을 하나의 완료 상태로 묶지 않기 위해 전달 파이프라인을 분리했습니다.

## 기능

| 번호 | 기능 | 완료 증거 |
|---|---|---|
| G-01 | 레포트 구성 | 승인 양식의 결과 모델 |
| G-02 | HTML 생성 | 동일 결과 패키지 기반 HTML |
| G-03 | 회귀검사 | 필드·색상·건수·원본 일치 |
| G-04 | 발송 전 검사 | 사용자·메일·제목·첨부·중복 확인 |
| G-05 | 이메일 전송 | 사용자 요청에 따른 실제 전송 결과 |
| G-06 | 전송 상태 기록 | 생성/요청/발송/미확인 분리 |
| G-07 | 사용자 수신 확인 | 수신·첨부 열림 확인 |
| G-08 | 재전송 제어 | 중복 없이 재시도 판단 |
| G-09 | 최종 완료 | TASK·History 완료 기록 |

## 처리 STEP

1. E 단계의 검증 완료 패키지 수신
2. 결과 패키지 ID·기준일·건수 고정
3. 승인된 최신 레포트 템플릿 적용
4. 도달·30% 근접·등급·시총·배당·위험 상태 구성
5. HTML 렌더링
6. 화면·HTML·원본의 행과 건수 비교
7. 공개/민감정보 경계 검사
8. 등록·검증된 기본 이메일 확인
9. 제목·기준일·첨부·중복 키 확인
10. 사용자의 결과 요청 범위 안에서 발송
11. 제공자가 반환한 전송 결과 기록
12. 사용자 수신·첨부 확인
13. 미수신이면 기존 발송 상태를 조사한 뒤 재전송
14. TASK 완료

## 논리 Table

### REPORT_PACKAGE

| 필드 | 의미 |
|---|---|
| PACKAGE_ID | 불변 결과 패키지 |
| REPORT_RUN_ID | 분석 실행 |
| TEMPLATE_VERSION | 승인 양식 버전 |
| ROW_COUNT/SUMMARY_COUNT | 행·요약 건수 |
| CONTENT_HASH | 결과 동일성 |
| VALIDATED_AT | 검증 완료 |
| STATUS | FROZEN/INVALIDATED |

### REPORT_DELIVERY

| 필드 | 의미 |
|---|---|
| DELIVERY_ID | 전달 키 |
| PACKAGE_ID | 전송 결과 |
| USER_EMAIL_ID | 검증 수신자 참조 |
| IDEMPOTENCY_KEY | 중복방지 키 |
| SUBJECT | 기준일 포함 제목 |
| ATTACHMENT_HASH | 첨부 동일성 |
| STATUS | PREPARED/SEND_REQUESTED/SENT/FAILED/RECEIVED/CONFIRMED |
| PROVIDER_MESSAGE_ID | 제공자 결과 |
| ATTEMPT_NO | 전송 시도 |
| ERROR_DETAIL | 실패 사유 |

## 알고리즘

```text
package = require_frozen_validated_package(report_run)
html = render(template_version, package)
assert rows_and_summary_match(package, html)
assert no_sensitive_fields(html)

recipient = resolve_verified_primary_email(user_id)
key = hash(package.id, recipient.id, subject)
if successful_delivery_exists(key): return existing_delivery

send_result = send_after_preflight(html, recipient, key)
record provider result separately from user receipt

if user_reports_missing:
    inspect existing provider state first
    resend only when policy permits and increment attempt
```

## 어려움·문제·개선

- 화면에 결과가 보인다는 사실을 이메일 발송 완료로 잘못 취급할 수 있었습니다. 상태를 분리했습니다.
- 이전 첨부나 다른 기준일 파일을 보낼 위험이 있어 패키지·첨부 해시를 비교합니다.
- 발송 응답이 성공이어도 사용자가 받았다는 뜻은 아닙니다. SENT와 RECEIVED/CONFIRMED를 분리합니다.
- 재시도할 때 같은 메일이 여러 번 갈 수 있어 idempotency key와 시도 번호를 사용합니다.
- AI가 수신자를 문맥으로 추정하면 오발송 위험이 있습니다. 검증된 USER_EMAIL_ID만 사용합니다.

## 장점·단점

장점은 분석 오류와 전달 오류를 구분하고 중복발송·잘못된 첨부를 줄이는 것입니다. 단점은 사용자 수신 확인이 필요할 수 있고 메일 제공자의 전달 상태가 완전하지 않을 수 있다는 점입니다.

## 완료 기준

검증 패키지, HTML 회귀검사, 수신자·제목·첨부 사전검사, 전송 결과 기록이 필요합니다. 최종 운영 완료는 사용자 수신 확인 정책까지 충족해야 합니다. 단순히 “보냈다”는 AI 문장만으로 완료 처리하지 않습니다.
