# Synology NAS PHP Threat Feed → MariaDB 저장 구조

> **Note**
>
> - 기준일: 2026-08-27
> - 참조 환경: Synology DSM 6 계열, MariaDB 5.5.68, PHP 7.4
> - 이 문서는 Synology NAS에서 외부 IP Threat Feed를 수집해 MariaDB DEV 데이터베이스에 저장한 검증 사례다.
> - 외부 Threat Feed의 제공 방식·주소·정책과 Synology·MariaDB·PHP 환경은 변경될 수 있으므로 실제 도입 시 최신 공식 정보와 실행 환경을 다시 확인해야 한다.
> - 이 문서는 교육 및 일반 기술정보 공유 목적이며 정확성·완전성·안전성을 보증하지 않는다.
> - 실제 환경 적용 전 별도 평가, 백업, 보안 검토, 접근권한 검토, 비용 검토, 법적·조직 정책 검토가 필요하다.
> - 비밀번호, 토큰, API Key, 실제 사용자 홈 경로 등 인증·운영 정보는 예시값으로 치환했다.
>
> **Caution**
>
> 외부 Threat Feed의 IP를 수집했다고 해서 해당 IP를 자동 차단해도 안전하다는 의미는 아니다. 오탐·공급망 변경·Feed 장애 가능성이 있으므로 자동 차단과 영구 차단은 별도 검증과 승인 절차를 적용해야 한다. DB 계정에는 최소 권한을 적용하고, DEV/LIVE 분리, 감사로그, 백업 및 복구 절차를 유지해야 한다.
>
> 저장소 공통 면책 및 공개 정책:
> - [DISCLAIMER.md](../DISCLAIMER.md)
> - [PUBLICATION_POLICY.md](../PUBLICATION_POLICY.md)

---

## 개요

Synology NAS에서 PHP 스크립트를 주기적으로 실행하여 외부 IP Threat Feed를 수집하고, MariaDB의 DEV 데이터베이스에 저장하는 구조다.

이번 문서는 **Threat Feed 수집과 MariaDB 저장 파이프라인**, 그리고 수집한 IP 정보를 기존 NAS 차단목록과 통합해 DSM 보안에 활용한 사례를 다룬다. 메일 발송과 년봉 레포트 기능은 범위에서 제외한다.

## 아키텍처

```text
Synology Task Scheduler
        |
        v
PHP 7.4 Collector
        |
        +--> blocklist.de
        |
        +--> abuse.ch Feodo Tracker
        |
        v
IP Validation / Deduplication
        |
        v
MariaDB Stored Procedure
        |
        +--> SEC_IP위협정보
        |
        +--> SEC_IP피드수집이력
```

## 실행 환경

- Synology NAS
- MariaDB 5.5.68
- DEV DB: `YearReport_DEV`
- PHP 7.4 binary:

```text
/volume1/@appstore/PHP7.4/usr/local/bin/php74
```

NAS 기본 PHP 5.6 환경에서는 `pdo_mysql`/`mysqli` 사용 문제가 있어 PHP 7.4 런타임을 사용했다.

PHP 7.4 시작 시 `mcrypt.so`, legacy `mysql.so` 관련 warning이 발생할 수 있었지만 `pdo_mysql` 기반 DB 연결에는 영향을 주지 않았다.

## PHP 배치 위치

```text
/volume1/homes/<USER>/scripts/sec_ip/
```

Collector는 외부 Feed를 다운로드하고 IP 형식을 검증한 뒤 중복을 제거한다.

## Threat Feed

### blocklist.de

```text
https://lists.blocklist.de/lists/all.txt
```

최근 공격 관측 IP를 한 줄에 하나씩 제공한다.

### abuse.ch Feodo Tracker

```text
https://feodotracker.abuse.ch/downloads/ipblocklist_recommended.txt
```

Botnet C2로 알려진 IP 목록을 제공한다.

## DB 저장 방식

PHP가 테이블에 직접 INSERT/UPDATE하지 않고 Stored Procedure를 호출한다.

### SEC_IP위협정보

IP별 현재 Threat 정보를 저장한다.

`SP_SEC_IP위협정보반영`은 IP 존재 여부를 확인하여:
- 신규 IP → INSERT
- 기존 IP → UPDATE

처리 후 `INSERT` 또는 `UPDATE` 결과를 PHP에 반환한다.

### SEC_IP피드수집이력

Feed 실행 단위 감사 이력을 저장한다.

기록 항목:
- Feed 코드/명
- 수집 시작/종료
- 정상여부
- 수집건수
- 신규 반영건수
- 갱신건수
- 제외건수
- 오류건수
- HTTP 상태
- 오류 코드/메시지
- 처리시간

Feed 수집 이력은 PHP가 전체 실행 결과를 알기 때문에 DB Trigger가 아니라 별도 Stored Procedure 호출 방식으로 기록한다.

## 최소권한 DB 계정

수집 전용 DB 계정을 별도로 두고 테이블 직접 DML 대신 필요한 Procedure 실행권한만 준다.

```sql
GRANT EXECUTE ON PROCEDURE YearReport_DEV.SP_SEC_IP위협정보반영
TO 'sec_ip_collector'@'localhost';

GRANT EXECUTE ON PROCEDURE YearReport_DEV.SP_SEC_IP피드수집이력등록
TO 'sec_ip_collector'@'localhost';
```

DB 비밀번호와 API Key 등 비밀정보는 소스나 공개 저장소에 기록하지 않는다.

## Synology Task Scheduler 예시

```bash
cd /volume1/homes/<USER>/scripts/sec_ip

DB_HOST=127.0.0.1 \
DB_PORT=3306 \
DB_NAME=YearReport_DEV \
DB_USER=sec_ip_collector \
DB_PASS='<SECRET>' \
/volume1/@appstore/PHP7.4/usr/local/bin/php74 sec_ip_threat_feed_collector_v2.php --db \
> run74_v2.log 2>&1
```

운영 스케줄 예:
- 매일
- 00:00 시작
- 30분 간격
- 23:30 마지막 실행

## 실제 DEV 검증

2026-08-27 기준 실제 DEV DB에서 다음 파이프라인을 검증했다.

```text
Scheduler
→ PHP Collector
→ External Threat Feed
→ Stored Procedure
→ SEC_IP위협정보
→ SEC_IP피드수집이력
```

검증 시 `SEC_IP위협정보`는 24,674건이 저장되어 있었다.

Feed 실행 예:

### BLOCKLIST_DE_ALL
- HTTP: 200
- 수집: 24,582
- 신규: 150
- 갱신: 24,432
- 오류: 0

### FEODO_RECOMMENDED
- HTTP: 200
- 수집: 1
- 신규: 0
- 갱신: 1
- 오류: 0

중복 IP에 대해서는 UPDATE가 정상 수행되어 동일 IP가 무한히 신규 INSERT되지 않는 것도 확인했다.

## NAS 보안 보강에 활용한 사례

MariaDB에 축적한 Threat Feed IP는 저장과 조회에만 사용하지 않고 NAS 보안 보강에도 활용했다.

당시 NAS에서 사용하던 기존 deny 목록 약 47,950건과 새로 수집한 Threat Feed IP를 통합하고, 중복을 정리한 차단목록을 DSM에 수동 업로드했다. 이를 통해 기존에 관리하던 차단대상과 외부 Feed에서 새로 확보한 위협 IP를 하나의 NAS 차단목록으로 운용할 수 있었다.

```text
기존 NAS deny 목록
        +
PHP로 수집한 Threat Feed IP
        ↓
형식 정리 / 중복 제거
        ↓
DSM 차단목록 수동 업로드
        ↓
NAS 접근 차단 범위 보강
```

이 사례에서는 DB 수집 기능과 DSM 반영 기능을 분리했다. 자동 차단과 자동 영구차단 정책은 활성화하지 않고, 생성된 목록을 확인한 뒤 수동으로 DSM에 반영하는 방식으로 운용했다.

### 적용 과정에서 확인한 주의점

- 외부 Feed에는 오탐이나 오래된 IP가 포함될 수 있다.
- 기존 deny 목록과 신규 Feed 사이의 중복을 정리할 필요가 있다.
- 관리자 접속 IP나 정상 서비스 IP가 포함되면 NAS 접속에 영향을 줄 수 있다.
- DSM 버전과 가져오기 형식에 따라 목록 형식을 맞춰야 한다.
- 기존 차단목록은 병합 전에 별도로 보관해 두는 것이 복구에 도움이 된다.
- 자동 차단 또는 영구차단 적용 여부와 운영 기준은 각 환경의 관리자가 판단할 사항이다.
## 보안 정책

Threat Feed DB 저장 기능과 실제 자동 차단 기능은 분리한다.

검증 시 자동 차단 관련 정책은 OFF 상태를 유지했다.

```text
AUTO_BLOCK_ENABLED = OFF
AUTO_PERM_ENABLED  = OFF
```

즉 이 단계는 위험 IP 정보를 수집하고 DB에 축적하는 역할만 수행한다.

## 설계 포인트

- NAS 스케줄러는 실행 트리거 역할만 담당
- PHP는 외부 HTTP 통신과 실행 전체 결과 집계 담당
- MariaDB Stored Procedure는 데이터 변경 규칙 담당
- Feed 이력은 실행 단위로 별도 저장
- 수집 DB 계정은 최소 권한 적용
- 비밀정보는 공개 소스에 포함하지 않음

이 구조로 외부 Threat Feed 공급자를 추가하더라도 PHP의 Feed 정의와 저장 호출을 확장하는 방식으로 대응할 수 있다.