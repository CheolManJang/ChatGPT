# A. 사용자 및 기본 설정

> [!NOTE]
> **기준:** 2026-08-24. 개인 ChatGPT Plus 웹/Work, 직접 OpenAI API 호출 없음. 제품·플랜·권한·연결 앱 변경 후 재검증해야 합니다.
>
> [!CAUTION]
> 교육·기술 공유용 자료이며 보증하지 않습니다. 실제 적용 전 평가·테스트·보안·백업·법적 검토를 수행하십시오. [면책 조항](../../DISCLAIMER.md)을 참조하십시오.


## 도입 이유

여러 사용자·채팅·작업에서 요청자를 잘못 식별하거나 이전 이메일·잘못된 원본을 사용하는 문제를 막기 위해 도입했습니다. 사용자 정보는 편의 기능이 아니라 모든 TASK, 원본, 결과, 이메일 전달의 소유권 기준입니다.

## 기능

| 번호 | 기능 | 사용 시점 | 결과 |
|---|---|---|---|
| A-01 | 사용자 등록 | 최초 사용 | 내부 USER_ID와 표시 이름 생성 |
| A-02 | 이메일 등록·변경 | 결과 수신 주소 설정·변경 | 검증 상태가 있는 USER_EMAIL |
| A-03 | 권한·공개범위 | 운영자료 또는 GitHub 자료 접근 | PRIVATE/PUBLIC 경계 |
| A-04 | 공식 원본 등록 | 마스터·원시자료 신규 등록·교체 | FILE_REGISTRY 버전 |

## 사용자와 AI 작업

사용자는 이름·별명·결과 수신 이메일과 공개 가능 범위를 제공하고 변경을 승인합니다. AI는 중복 사용자를 검사하고 이메일을 임의 추정하지 않으며, 공식 원본의 파일 ID·버전·역할을 기록합니다. 민감정보를 GitHub 문서나 공개 샘플에 복사하지 않습니다.

## 처리 STEP

1. 사용자 등록 요청 접수
2. 기존 USER_ID·별명 중복 검사
3. 내부 식별자 생성
4. 이메일 등록 요청과 소유 확인 상태 기록
5. 공개/비공개 기본값 적용
6. 공식 원본 파일의 역할·버전·해시 등록
7. 변경 전 값을 History에 보존
8. 후속 TASK에서 USER_ID와 FILE_ID를 참조

## 논리 Table

### USER

| 필드 | 의미 |
|---|---|
| USER_ID | 외부에 노출하지 않는 사용자 키 |
| DISPLAY_NAME | 화면 표시명 |
| ALIAS | 명령·검색용 별명 |
| STATUS | ACTIVE/HOLD/INACTIVE |
| PRIVACY_LEVEL | 공개 제한 수준 |
| CREATED_AT/UPDATED_AT | 생성·변경 시각 |

### USER_EMAIL

| 필드 | 의미 |
|---|---|
| USER_EMAIL_ID | 이메일 레코드 키 |
| USER_ID | 사용자 연결 |
| EMAIL_ENCRYPTED | 비공개 저장 주소 |
| VERIFIED_YN | 확인 상태 |
| PRIMARY_YN | 기본 수신 주소 |
| VALID_FROM/VALID_TO | 유효 기간 |

### FILE_REGISTRY

| 필드 | 의미 |
|---|---|
| FILE_ID | 파일의 안정 식별자 |
| ROLE_CODE | MASTER/RAW/REPORT/BACKUP |
| VERSION | 원본 버전 |
| CONTENT_HASH | 내용 동일성 확인 |
| STORAGE_PROVIDER | 승인된 저장 위치 |
| STATUS | ACTIVE/REPLACED/INVALID |
| REGISTERED_AT | 등록 시각 |

## 알고리즘

```text
user = find_by_stable_identity(request)
if duplicate_or_ambiguous(user): HOLD_FOR_USER_REVIEW
else user_id = create_or_update_user_with_history()

email = normalize(request.email)
if owner_not_verified(email): keep UNVERIFIED and block_delivery
else set_primary_email(user_id, email)

file = inspect_registered_file(request.file)
if role/version/hash missing: NG
else register_version_without_overwriting_history(file)
```

## 어려움·문제·개선

- 대화에서 이름만 기억하면 동명이인·별명 변경에 취약했습니다. 내부 USER_ID를 도입해 해결했습니다.
- 이메일을 대화 기억만으로 선택하면 오발송 위험이 있습니다. 등록·검증·기본 주소를 분리했습니다.
- 파일명이 같아도 내용이 달라질 수 있습니다. FILE_ID·버전·해시를 함께 확인합니다.
- ChatGPT Memory는 편리하지만 운영 DB처럼 원자적·결정적으로 동기화되지 않습니다. 공식 원본은 FILE_REGISTRY가 가리키는 저장소로 제한합니다.

## 장점·단점

장점은 잘못된 사용자·수신자·원본 사용을 줄이고 모든 결과를 추적할 수 있다는 점입니다. 단점은 최초 등록과 변경 승인 절차가 늘고 이메일·파일 메타데이터를 안전하게 관리해야 한다는 점입니다.

## 완료 기준과 한계

USER_ID, 검증된 기본 이메일, 활성 원본 버전이 모두 확정되어야 완료입니다. 불명확한 수신자나 원본은 AI가 추정하지 않고 HOLD로 전환합니다. 사용자용 알림·스케줄 설정은 현재 구현 기능으로 주장하지 않으며 필요 시 별도 승인 작업으로 다룹니다.
