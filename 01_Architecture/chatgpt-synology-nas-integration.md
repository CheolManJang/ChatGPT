# ChatGPT ↔ Synology NAS 연동 구축 기술 공유

> **Note**
>
> - 기준일: 2026-08-26
> - 참조 환경: 개인 ChatGPT Plus, ChatGPT 웹/Work, Synology DSM 6 계열, MariaDB 5, PHP 7.4
> - 직접 OpenAI API 호출을 중심으로 한 구성이 아니라 ChatGPT의 MCP/앱 연결과 NAS 측 HTTPS API를 조합해 검증한 사례다.
> - ChatGPT의 제품·플랜·권한·앱·MCP·Secure MCP Tunnel 정책은 변경될 수 있으므로 실제 도입 시 최신 공식 문서를 다시 확인해야 한다.
> - 이 문서는 교육 및 일반 기술정보 공유 목적의 구축 사례이며 정확성·완전성·안전성을 보증하지 않는다.
> - 실제 환경 적용 전 별도 평가, 백업, 보안 검토, 접근권한 검토, 비용 검토, 법적·조직 정책 검토가 필요하다.
> - 비밀번호, 토큰, API Key, 인증서, Private Key, 실제 서버 주소, 실제 DB 이름, 실제 계정명 등 인증·운영 정보는 예시값으로 치환했다.
>
> **Caution**
>
> MCP 또는 외부 API를 사용한다고 해서 연결이 자동으로 안전해지는 것은 아니다. OpenAI도 신뢰할 수 없는 MCP 서버 연결 시 프롬프트 인젝션을 포함한 보안 위험에 더 노출될 수 있다고 안내한다. DB 쓰기·수정 권한을 제공하는 경우 권한 최소화, 인증, 감사로그, 백업, DEV/LIVE 분리 등 별도 통제가 필요하다.
>
> 저장소 공통 면책 및 공개 정책:
> - [DISCLAIMER.md](../DISCLAIMER.md)
> - [PUBLICATION_POLICY.md](../PUBLICATION_POLICY.md)

---

## 1. 구축 목적

목표는 ChatGPT가 개인 Synology NAS의 MariaDB 데이터를 MCP/HTTPS API를 통해 조회하고, 허용된 범위에서 등록·수정 작업까지 수행할 수 있는 연결 환경을 만드는 것이었다.

단순 연결 성공만 목표로 하지 않았다. 실제 운용을 고려해 다음 항목을 함께 확인했다.

- ChatGPT에서 NAS DB를 직접 조회할 수 있는가
- DB를 외부에 직접 노출하지 않는 다른 연결 방식은 무엇인가
- 읽기뿐 아니라 INSERT/UPDATE 등 쓰기 작업까지 가능한가
- 인증정보가 유출되었을 때 피해 범위를 줄일 수 있는가
- 동일 요청 재전송(Replay)을 구분할 수 있는가
- 터널이 끊기거나 PC가 재부팅되면 어떻게 되는가
- Custom MCP 사용 시 ChatGPT의 다른 연결 앱과 충돌하지 않는가
- NAS 관리용 phpMyAdmin도 HTTPS로 운영할 수 있는가
- Docker를 사용할 수 없는 구형 Synology에서도 구현 가능한가

최종적으로 검증한 기본 흐름은 다음과 같다.

```text
ChatGPT
   ↓
MCP / HTTPS
   ↓
DSM Reverse Proxy
   ↓
NAS PHP API
   ↓
localhost
   ↓
MariaDB
```

---

## 2. 참조 환경

| 구분 | 환경 |
|---|---|
| NAS | Synology DS218j |
| DSM | DSM 6.2 계열 |
| Container | Docker 미지원 모델 |
| Web | Web Station / Nginx |
| PHP | PHP 7.4 |
| DB | MariaDB 5 계열 |
| 외부접속 | DDNS + HTTPS Reverse Proxy |
| ChatGPT | 개인 ChatGPT 환경에서 MCP/앱 연결 검증 |

공개 문서에서는 실제 도메인, DB 이름, 사용자명은 사용하지 않고 아래와 같이 표기한다.

```text
<NAS_DOMAIN>
<DEV_DB>
<API_DB_USER>
<API_KEY>
<PRIVATE_KEY_PATH>
```

---

## 3. 1차 시도 — MariaDB 직접 접속

### 왜 시도했는가

가장 단순한 구조는 ChatGPT가 MariaDB 포트에 직접 접속하는 방식이다.

```text
ChatGPT
   ↓ TCP
NAS MariaDB :3306
```

NAS/MariaDB 측에서는 외부 접근을 위한 계정·권한·네트워크 설정을 준비하고 외부망에서 접근 가능한지 확인했다.

### 실제 문제

NAS 쪽 DB 접속 준비와 별개로, 당시 ChatGPT 실행 환경에서는 외부 MariaDB의 TCP/Socket 연결을 직접 사용할 수 없어 이 방식으로 더 진행할 수 없었다.

즉 이 단계에서의 중단 사유는 SQL 문법이나 MariaDB 설정만의 문제가 아니라 **ChatGPT에서 임의의 외부 DB TCP 연결을 직접 수행하는 방식의 제약**이었다.

### 판단

직접 DB 접속 방식은 중단하고 HTTPS로 호출 가능한 중간 API를 두기로 했다.

```text
ChatGPT
   ↓ HTTPS
NAS API
   ↓ localhost
MariaDB
```

### 이 단계에서 얻은 점

- DB 3306을 ChatGPT 연결 목적으로 직접 공개할 필요가 없어졌다.
- 인증·권한·로그를 API 계층에서 추가할 수 있게 됐다.
- MariaDB 계정은 NAS 내부 localhost 접근으로 제한할 수 있게 됐다.

---

## 4. NAS API 방식으로 전환

### 작업 위치

Synology DSM의 다음 기능을 사용했다.

```text
DSM
├─ Web Station
├─ Reverse Proxy
└─ MariaDB / phpMyAdmin
```

NAS 웹 영역에 API 전용 디렉터리를 구성했다.

예시:

```text
web/
└─ nas-api/
   ├─ public/
   │  └─ index.php
   ├─ app/
   │  ├─ db.php
   │  └─ auth.php
   └─ config/
      └─ config.php
```

실제 공개 저장소에서는 운영 서버 경로와 인증파일 위치를 그대로 공개하지 않는다.

### PHP DB 연결 샘플

```php
<?php

$host = 'localhost';
$db   = '<DEV_DB>';
$user = '<API_DB_USER>';
$pass = getenv('NAS_DB_PASSWORD');

$pdo = new PDO(
    "mysql:host={$host};dbname={$db};charset=utf8mb4",
    $user,
    $pass,
    [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    ]
);
```

핵심은 API가 외부 MariaDB 주소로 다시 접속하는 것이 아니라 NAS 내부의 `localhost`를 통해 DB를 사용하는 구조다.

---

## 5. API Health Check

API와 DB 문제를 구분할 수 있도록 Health Check를 먼저 만들었다.

```php
<?php

header('Content-Type: application/json; charset=utf-8');

try {
    $stmt = $pdo->query('SELECT 1 AS ok');
    $row = $stmt->fetch();

    echo json_encode([
        'status' => 'OK',
        'db' => ((int)$row['ok'] === 1)
    ]);
} catch (Throwable $e) {
    http_response_code(500);

    echo json_encode([
        'status' => 'ERROR'
    ]);
}
```

확인 순서:

```text
HTTPS 연결
→ API 실행
→ 인증
→ DB 연결
→ SQL 실행
```

---

## 6. DB 계정 권한 분리

### 작업 위치

MariaDB 사용자/권한 설정은 NAS의 MariaDB 관리 환경 또는 phpMyAdmin에서 수행할 수 있다.

관리 URL 예시:

```text
https://<NAS_DOMAIN>:<ADMIN_PORT>/phpMyAdmin/
```

### 왜 root를 사용하지 않았는가

ChatGPT/MCP에서 SQL 쓰기 기능까지 허용하면 계정 권한 범위가 중요해진다. API에서 MariaDB `root`를 사용하면 잘못된 쿼리 또는 인증정보 유출 시 영향 범위가 지나치게 커진다.

그래서 API 전용 사용자를 만들고 특정 DEV DB에만 권한을 부여했다.

```sql
CREATE USER '<API_DB_USER>'@'localhost'
IDENTIFIED BY '<STRONG_PASSWORD>';

GRANT SELECT, INSERT, UPDATE, DELETE
ON <DEV_DB>.*
TO '<API_DB_USER>'@'localhost';

FLUSH PRIVILEGES;
```

실제 필요한 작업에 따라 DDL 권한을 추가할 수 있지만, 운영에서는 필요한 권한만 부여하는 것이 기본 원칙이다.

---

## 7. 단순 API Key만으로 충분한가

초기에는 API Key를 전달해 요청자를 식별하는 구조를 검토했다.

```text
Authorization: Bearer <API_KEY>
```

구현은 간단하지만 Key 하나가 복사되면 정상 Client와 동일한 형태의 요청을 만들 수 있다는 문제가 있다.

그래서 실제 구축에서는 요청 내용 자체에 대한 서명을 추가하는 방향으로 보완했다.

---

## 8. RSA/SHA256 요청 서명

서명 입력값을 다음처럼 구성했다.

```text
METHOD
PATH
TIMESTAMP
NONCE
SHA256(BODY)
```

개념:

```text
Client
 Private Key
     ↓
요청 데이터 서명
     ↓
HTTPS
     ↓
NAS API
 Public Key
     ↓
서명 검증
```

PHP 검증 예시:

```php
<?php

$timestamp = $_SERVER['HTTP_X_TIMESTAMP'] ?? '';
$nonce     = $_SERVER['HTTP_X_NONCE'] ?? '';
$signature = $_SERVER['HTTP_X_SIGNATURE'] ?? '';

$body = file_get_contents('php://input');

$canonical =
    $_SERVER['REQUEST_METHOD'] . "\n" .
    parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH) . "\n" .
    $timestamp . "\n" .
    $nonce . "\n" .
    hash('sha256', $body);

$result = openssl_verify(
    $canonical,
    base64_decode($signature, true),
    $publicKey,
    OPENSSL_ALGO_SHA256
);

if ($result !== 1) {
    http_response_code(401);
    exit;
}
```

> 이 코드는 구조 설명용 샘플이다. 실제 구현에서는 Header 형식, canonicalization, base64 오류 처리, key loading, 로그, 예외 처리 등을 별도로 검증해야 한다.

---

## 9. Timestamp와 Nonce — Replay Protection

서명이 정상이어도 동일한 서명 요청을 그대로 복사해 다시 보내는 공격을 고려해야 했다.

### Timestamp

```text
Server Time ± 5분
```

허용 범위를 벗어난 요청은 거부한다.

### Nonce

각 요청에 일회성 값을 넣고 서버는 이미 처리한 Nonce인지 확인한다.

```text
정상 요청
→ Nonce 저장
→ 동일 Nonce 재사용
→ Reject
```

이 역시 보안을 보장한다는 의미가 아니라 Replay 위험을 줄이기 위한 한 가지 보완 조치다.

---

## 10. MCP 연결

ChatGPT의 Developer Mode / MCP 앱을 통해 NAS 측 원격 MCP/API endpoint를 연결하는 방식을 검토했다.

공식 참고:
- https://help.openai.com/en/articles/12584461-developer-mode-apps-and-full-mcp-connectors-in-chatgpt-beta
- https://help.openai.com/ko-kr/articles/12584461-developer-mode-and-full-mcp-connectors-in-chatgpt-beta

OpenAI 공식 문서상 ChatGPT는 로컬 MCP 서버에 직접 연결하지 않는다. 원격 MCP 서버를 사용하거나 비공개 네트워크의 서버라면 Secure MCP Tunnel을 사용할 수 있다.

Full MCP의 플랜/쓰기 권한/Developer Mode 지원 범위는 변경될 수 있으므로 도입 시 현재 플랜 기준으로 다시 확인해야 한다.

---

## 11. Secure MCP Tunnel을 실제로 설정한 이유

NAS를 공용 인터넷에 바로 공개하는 구조 외에 Secure MCP Tunnel도 실제로 설정해 보았다.

```text
ChatGPT
   ↓
Secure MCP Tunnel
   ↓
Tunnel Client
   ↓
NAS MCP/API
```

Windows Client 쪽에서는 Tunnel Profile과 인증에 필요한 값을 설정하고 Client 프로세스를 실행했다.

실행 편의를 위해 환경변수 설정과 Tunnel Client 실행 명령을 BAT 파일로 묶는 방식도 검토했다.

```bat
@echo off
set CONTROL_PLANE_API_KEY=<DO_NOT_COMMIT_REAL_KEY>
tunnel-client.exe run --profile <PROFILE_NAME>
```

실제 API Key는 BAT 파일에 평문으로 저장하지 않는 방식이 더 바람직하며, 공개 GitHub에는 절대 실제 값을 넣지 않는다.

---

## 12. Tunnel에서 실제로 확인한 운영 문제

Tunnel 연결 직후에는 NAS MCP 작업이 가능했다.

그러나 다음 날 오전 확인했을 때 Tunnel이 끊겨 있었고, 그 상태에서는 ChatGPT가 NAS 작업을 수행하지 못했다.

```text
Tunnel Client 중단/연결 끊김
        ↓
ChatGPT → NAS MCP 접근 실패
```

상시 운영한다면 다음을 별도로 설계해야 한다.

- OS 재부팅 후 자동 시작
- Client 비정상 종료 감지
- 자동 재실행
- Tunnel Health Check
- 연결 실패 알림
- 인증정보 Rotation
- 장애 시 직접 HTTPS 연결 또는 다른 관리 경로

### Tunnel 비용

설정 당시에는 Tunnel 사용 자체에 대해 별도 사용료가 추가로 청구되는 구조로 확인되지 않았다.

하지만 별도 Tunnel 가격표가 항상 동일하게 유지된다고 보장할 수 없으며, OpenAI의 플랜·크레딧·MCP 정책이 변경될 수 있다.

따라서 실제 도입 시 반드시 최신 공식 문서와 가격정책을 다시 확인해야 한다.

참고:
- https://help.openai.com/en/articles/12584461-developer-mode-apps-and-full-mcp-connectors-in-chatgpt-beta
- https://openai.com/business/chatgpt-pricing/

---

## 13. Tunnel 방식과 공개 HTTPS 방식 비교

| 항목 | Secure MCP Tunnel | NAS HTTPS 직접 연결 |
|---|---|---|
| NAS MCP 공개 | 직접 공개하지 않을 수 있음 | HTTPS endpoint 외부 공개 필요 |
| 중간 Client | 필요 | 불필요 |
| PC 종료 영향 | Client 위치에 따라 영향 가능 | NAS가 켜져 있으면 별도 PC 불필요 |
| 자동복구 관리 | 필요 | Reverse Proxy/NAS 서비스 관리 필요 |
| 보안 고려 | Tunnel/MCP/Client 자격증명 | NAS 공개 포트/Reverse Proxy/API 인증 |
| 장애 지점 | Tunnel 서비스 + Client + NAS | DNS/인증서/Reverse Proxy/NAS |
| 정책 의존 | OpenAI Tunnel 정책 영향 | ChatGPT MCP 정책 + NAS 운영 영향 |

Tunnel이 더 좋거나 직접 HTTPS가 더 좋다고 일률적으로 결론 내릴 수는 없다. 사용 환경의 보안정책과 가용성 요구사항에 따라 선택해야 한다.

---

## 14. DSM Reverse Proxy 적용

Tunnel 외에 NAS에서 직접 HTTPS endpoint를 제공하는 구조도 구성했다.

```text
Internet
   ↓
HTTPS
   ↓
DSM Reverse Proxy
   ↓
NAS 내부 HTTPS API Port
   ↓
PHP API
```

Reverse Proxy를 사용하면 외부 URL과 내부 Web Station Port를 분리할 수 있다.

### 주의

Reverse Proxy 설정을 잘못하면 기존 서비스의 Host/Port를 가로채거나 Proxy loop가 생길 수 있다. 기존 규칙이 있다면 변경 전에 Source/Destination을 반드시 기록해야 한다.

---

## 15. Generic SQL Endpoint 도입

초기에는 기능마다 API Endpoint를 만들 수 있었다.

```text
/add-item
/update-user
/get-report
...
```

기능이 늘수록 Endpoint도 계속 늘어나 유지보수가 커진다.

DEV 환경에서는 Schema와 SQL을 다룰 수 있도록 Generic SQL Tool을 제공하는 방식을 선택했다.

```text
ChatGPT
   ↓
MCP SQL Tool
   ↓
NAS API
   ↓
MariaDB
```

### 장점

- 새 기능마다 PHP Endpoint를 추가하지 않아도 됨
- Schema 조회 후 SQL 생성 가능
- Migration 및 Validation 수행이 편리함
- DB 구조 변경 대응이 빠름

### 단점

Generic SQL은 권한이 매우 강하다.

따라서 다음을 별도로 고려해야 한다.

- DEV/LIVE 계정 완전 분리
- DB 사용자 권한 최소화
- DROP/TRUNCATE 등 제한
- Transaction
- Backup
- 변경 History
- 사용자 확인이 필요한 명령 분류
- SQL allow/deny 정책
- 실행 로그

---

## 16. 실제 SQL/Migration 검증

연결 성공 여부만 확인하지 않고 실제 DEV DB 작업까지 수행했다.

```text
Auth Health
→ DB Health
→ Schema 확인
→ SELECT
→ INSERT / UPDATE
→ Migration
→ View 검증
→ 중복/Key 검증
```

실제 DEV Migration에서는 수백 개의 SQL statement와 수천 건의 row를 처리했다.

공개 문서에서는 업무 DB 이름이나 실제 데이터 내용은 공개하지 않고 결과 규모와 검증 방식만 공유한다.

검증 항목 예:

```text
Invalid Key = 0
Duplicate GUID = 0
Duplicate Business Key = 0
View Validation = PASS
```

---

# phpMyAdmin HTTPS 구성과 문제 해결

## 17. 문제의 시작

NAS DB 관리용 phpMyAdmin은 기존 HTTP 경로에서 동작하고 있었다.

그러나 NAS의 HTTPS 443은 이미 API Reverse Proxy 규칙이 사용하고 있었기 때문에 같은 도메인의 `/phpMyAdmin/` 요청이 API 쪽으로 들어가는 충돌이 발생했다.

그래서 phpMyAdmin을 별도 HTTPS Port로 분리하기로 했다.

---

## 18. 첫 번째 시도 — Web Station에 별도 phpMyAdmin VHost

```text
HTTPS <ADMIN_PORT>
   ↓
Web Station Virtual Host
   ↓
web/phpMyAdmin
   ↓
별도 PHP Profile
```

PHP 7.4 Profile을 만들고 필요한 Extension을 활성화했다.

```text
mysqli
pdo_mysql
curl
iconv
openssl
zip
```

그런데 phpMyAdmin에서 `mysqli extension is missing` 오류가 나타났다.

---

## 19. phpinfo로 PHP Runtime 확인

임시 `phpinfo()` 파일을 만들어 실제 Web Runtime을 확인했다.

```text
PHP 7.4
mysqli Support = enabled
mysqlnd = enabled
pdo_mysql = enabled
```

즉 PHP Profile 화면의 설정만 보는 것이 아니라 **실제 요청이 어느 PHP Runtime에서 처리되는지 확인해야 한다**는 점을 알게 됐다.

> `phpinfo()`는 서버 경로, 모듈, 환경정보를 많이 노출한다. 진단 후 즉시 제거해야 한다.

---

## 20. Synology 기본 phpMyAdmin 실행 구조 확인

SSH로 Synology 패키지의 Nginx 설정을 확인했다.

```nginx
location ^~ /phpMyAdmin/ {
    root /var/services/web;

    location ~ \.php$ {
        include fastcgi.conf;
        fastcgi_pass unix:/run/php-fpm/php74-fpm.sock;
    }
}
```

Synology가 이미 phpMyAdmin을 위한 Nginx + PHP 7.4 FPM 환경을 구성하고 있었다.

그래서 별도의 Web Station phpMyAdmin 환경을 계속 만드는 대신 기존 Synology 패키지 실행환경을 그대로 재사용하는 쪽으로 방향을 바꿨다.

---

## 21. phpMyAdmin 최종 Proxy 구조

```text
Browser
https://<NAS_DOMAIN>:<ADMIN_PORT>/phpMyAdmin/
        ↓
DSM Reverse Proxy
        ↓
http://127.0.0.1:80/phpMyAdmin/
        ↓
Synology Nginx
        ↓
PHP 7.4 FPM
        ↓
MariaDB
```

Reverse Proxy 화면에서는 `web/phpMyAdmin` 같은 실제 폴더를 지정하지 않는다.

`/phpMyAdmin/` URL path를 backend Nginx에 그대로 전달하고, Synology 기본 Nginx가 실제 폴더를 처리한다.

---

## 22. 로그인 페이지는 열리는데 Session Cookie 실패

로그인 페이지는 정상적으로 열렸지만 로그인하면 다음 오류가 발생했다.

```text
Failed to set session cookie.
Maybe you are using HTTP instead of HTTPS to access phpMyAdmin.
```

외부 요청은 HTTPS였지만 Reverse Proxy backend는 HTTP였다.

```text
External: HTTPS
    ↓
Reverse Proxy
    ↓
Backend: HTTP
```

phpMyAdmin이 실제 Client 요청을 HTTPS로 인식하지 못하는 것이 핵심 원인이었다.

---

## 23. X-Forwarded Header 적용

Reverse Proxy Request Header에 다음 정보를 추가했다.

```text
X-Forwarded-Proto: https
X-Forwarded-Port: <ADMIN_PORT>
Host: <NAS_DOMAIN>:<ADMIN_PORT>
```

목적은 backend에 원래 Client 요청이 HTTPS였다는 정보를 전달하는 것이다.

---

## 24. PmaAbsoluteUri 수정

잘못된 예:

```php
$cfg['PmaAbsoluteUri'] =
    'https://<NAS_DOMAIN>:<ADMIN_PORT>/';
```

최종 예:

```php
$cfg['PmaAbsoluteUri'] =
    'https://<NAS_DOMAIN>:<ADMIN_PORT>/phpMyAdmin/';

$cfg['CheckConfigurationPermissions'] = false;
```

핵심은 외부 URL에 `/phpMyAdmin/` path까지 포함하는 것이다.

### 결과

```text
HTTPS phpMyAdmin 로그인 성공
```

---

## 25. config.inc.php 파일 권한 문제

File Station을 통해 설정 파일을 교체하는 과정에서는 phpMyAdmin이 설정 파일 권한을 문제 삼는 상황도 발생했다.

```text
Wrong permissions on configuration file,
should not be world writable!
```

SSH에서 Owner/Permission을 확인하고 다음과 같이 정리했다.

```bash
chown root:root /var/services/web/phpMyAdmin/config.inc.php
chmod 644 /var/services/web/phpMyAdmin/config.inc.php
```

실제 환경의 설치 경로는 NAS 모델/DSM/package에 따라 확인 후 사용해야 한다.

---

## 26. SSH를 임시로 사용한 이유

DSM GUI만으로 확인이 어려운 부분은 SSH를 잠시 활성화해 직접 확인했다.

```text
Nginx package config
PHP-FPM socket
실제 PHP version
config file path
Owner / Group
File permission
Process
```

작업이 끝난 뒤에는 SSH 서비스를 다시 비활성화하고, 기본 관리자 계정도 비활성 상태로 되돌렸다.

---

# ChatGPT/MCP 운영상 확인한 제약

## 27. MCP가 보안을 보장하지 않는다

OpenAI 공식 문서에도 Developer Mode/MCP는 강력한 기능이므로 별도 보안 검토가 필요하다고 안내되어 있다.

특히 신뢰할 수 없는 MCP Server에 연결하면 Prompt Injection을 포함한 보안 위험에 더 노출될 수 있다.

공식 참고:
- https://help.openai.com/en/articles/12584461-developer-mode-apps-and-full-mcp-connectors-in-chatgpt-beta
- https://help.openai.com/ko-kr/articles/12584461-developer-mode-and-full-mcp-connectors-in-chatgpt-beta

따라서 다음 표현은 사용하지 않는다.

```text
완벽한 보안
완전히 안전
MCP라서 안전
HTTPS라서 안전
```

대신 보안 위험을 줄이기 위한 조치를 적용했다고 표현한다.

---

## 28. ChatGPT 데이터 보관/처리 정책 확인

MCP를 NAS나 사내 DB에 연결하기 전에는 ChatGPT의 데이터 처리·보관 조건을 충분히 확인해야 한다.

검토 항목:

- 대화 데이터 저장/삭제 정책
- `Improve the model for everyone` 설정
- 연결 앱 데이터 처리 방식
- Memory 사용 여부
- Workspace/개인 플랜 차이
- 데이터 Residency 요구사항
- Compliance 요구사항
- 민감정보를 MCP Tool 결과로 전달해도 되는지

공식 참고:
- Data Controls FAQ  
  https://help.openai.com/en/articles/7730893-datacontrols-faq
- Google App for ChatGPT – Data Controls FAQ  
  https://help.openai.com/en/articles/10408842-google-connector-for-chatgpt-data-controls-faq
- Connected Apps  
  https://help.openai.com/en/collections/12923329

NAS에 접속할 수 있다는 것과 해당 데이터를 ChatGPT로 전달해도 된다는 것은 별개의 문제다.

---

## 29. API Key / 인증정보 관리

실제 공개 GitHub에는 다음 정보를 올리지 않는다.

```text
API Key
Private Key
DB Password
인증서 Private Key
실제 Connection String
실제 내부 주소
Tunnel 인증값
운영 Token
```

OpenAI도 API Key를 Repository에 Commit하지 말고 환경변수 또는 Secret 관리기능을 사용하도록 권고한다.

공식 참고:
- https://help.openai.com/en/articles/5112595
- https://help.openai.com/ko-kr/articles/5112595
- https://help.openai.com/en/articles/8304786

---

## 30. Custom MCP 사용 중 Gmail 호출 제한 경험

실제 테스트 환경에서는 Custom/Developer MCP를 사용하는 대화에서 NAS MCP Tool은 호출되지만 Gmail 등 기본 연결 앱 호출이 제한되는 상황을 경험했다.

```text
FORBIDDEN:
This conversation is restricted to developer MCPs
```

이 내용은 **모든 ChatGPT 사용자에게 항상 발생한다고 일반화할 수 없다.**

제품 배포 상태, 플랜, Developer Mode, App 권한이 변경될 수 있기 때문에 우리의 테스트 환경에서 실제로 관찰한 운영 제약으로 기록한다.

---

## 31. Gmail 제한에 대한 실제 대안 — NAS에서 Mail 발송

NAS MCP로 데이터를 처리한 뒤 같은 대화에서 Gmail Tool 호출이 제한되면 전체 Workflow가 중단될 수 있었다.

그래서 대안으로 메일 발송을 NAS 측 기능으로 분리했다.

```text
ChatGPT
   ↓
NAS MCP/API
   ↓
Report 생성/데이터 처리
   ↓
NAS Mail Sender
   ↓
SMTP / Mail Server
   ↓
수신자
```

### 장점

- 같은 대화에서 Gmail Connector가 반드시 호출될 필요가 없음
- DB 처리와 Mail 발송을 NAS 쪽 Workflow로 묶을 수 있음

### 새로 생기는 고려사항

- SMTP 인증정보 보호
- 수신자 정보 보호
- Mail 발송 로그
- 실패/재전송 처리
- SMTP Rate Limit
- 메일 서버 정책
- 스팸/보안 정책

즉 Gmail 제한을 우회했지만 새로운 운영 책임이 NAS 쪽으로 이동한다.

---

# 실제 구축 중 얻은 교훈

## 32. 시행착오와 변경한 결정

| 시도 | 문제 | 최종 판단 |
|---|---|---|
| MariaDB 직접 TCP | ChatGPT 환경에서 직접 DB 연결 진행 불가 | HTTPS API로 변경 |
| API Key만 사용 | Key 복제 시 요청 위조 위험 | RSA Request Signature 추가 |
| Signature만 사용 | Replay 가능성 | Timestamp + Nonce 추가 |
| Secure MCP Tunnel | Tunnel 중단 시 NAS 작업 불가 | 자동복구 필요, 직접 HTTPS와 비교 |
| 별도 phpMyAdmin VHost | PHP Runtime/Extension 문제 | Synology 기본 phpMyAdmin 재사용 |
| HTTPS → HTTP Proxy | Session Cookie 오류 | X-Forwarded Header 추가 |
| PmaAbsoluteUri root | `/phpMyAdmin/` Path 누락 | External URL 전체 경로 지정 |
| File Station config 교체 | 파일 권한 문제 | Owner/Mode 재확인 |
| Custom MCP + Gmail | 같은 대화에서 Gmail 제한 경험 | NAS Mail Sender로 우회 |

---

## 33. 현재 최종 아키텍처

### ChatGPT ↔ NAS

```text
┌───────────────┐
│    ChatGPT    │
└───────┬───────┘
        │
      HTTPS
        │
        ▼
┌──────────────────┐
│ DSM Reverse Proxy│
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ NAS MCP / PHP API│
└────────┬─────────┘
         │ localhost
         ▼
┌──────────────────┐
│     MariaDB      │
└──────────────────┘
```

### 관리자 DB 관리

```text
Browser
  ↓ HTTPS
DSM Reverse Proxy
  ↓ HTTP localhost
Synology phpMyAdmin
  ↓ PHP 7.4 FPM
MariaDB
```

---

# 장점과 단점

## 34. 장점

- MariaDB Port를 ChatGPT 연결 목적으로 직접 공개하지 않아도 됨
- 정형 DB를 SQL로 빠르게 검색 가능
- ChatGPT가 Schema 확인/검증/Migration까지 수행 가능
- Delphi 또는 다른 프로그램이 같은 NAS DB를 사용할 수 있음
- Synology Web Station 기반으로 Docker 없는 NAS에서도 구현 가능
- DEV DB를 별도 운영하면 실험/검증 구조를 만들기 쉬움
- Tool/API layer에서 인증·로그·권한정책을 추가할 수 있음

---

## 35. 단점과 위험

- NAS가 인터넷에 노출되는 구조라면 NAS 자체 공격면이 증가함
- MCP Server 또는 Tool이 과도한 권한을 가지면 피해 범위가 커질 수 있음
- Prompt Injection 등 MCP 특유의 위험을 고려해야 함
- Generic SQL은 편리하지만 Write/Delete/DDL 권한이 강력함
- Tunnel 방식은 Client/Tunnel 가용성에 의존함
- 직접 HTTPS 방식은 DDNS/인증서/Reverse Proxy/NAS 가용성에 의존함
- ChatGPT 플랜·MCP 권한·App 조합 정책이 변경될 수 있음
- Custom MCP와 다른 App의 동시 사용이 환경에 따라 제한될 수 있음
- 외부 공개 phpMyAdmin은 추가적인 공격면이 됨
- 구형 DSM/PHP/MariaDB는 업데이트 및 보안지원 한계를 확인해야 함

---

# 도입 전 검토사항

## 36. MCP/NAS 도입 Checklist

### ChatGPT / OpenAI

- [ ] 현재 사용 플랜에서 필요한 MCP 기능을 지원하는가
- [ ] Read뿐 아니라 Write/Modify가 필요한가
- [ ] Agent mode / Deep research에서 필요한 Tool을 사용할 수 있는가
- [ ] Developer Mode 권한 정책을 확인했는가
- [ ] Secure MCP Tunnel을 사용할지 검토했는가
- [ ] ChatGPT 데이터 보관·삭제 정책을 확인했는가
- [ ] Model Improvement 설정을 확인했는가
- [ ] 연결 App 데이터 처리 정책을 확인했는가
- [ ] Gmail/Calendar/Contacts 등 다른 App과 함께 실제 테스트했는가
- [ ] 정책 변경 시 재검증 담당자를 정했는가

### 보안

- [ ] MCP Server를 신뢰할 수 있는가
- [ ] Prompt Injection 위험을 검토했는가
- [ ] API Key/Private Key를 Repository에 저장하지 않는가
- [ ] DB root 계정을 사용하지 않는가
- [ ] 최소 권한 계정을 사용하는가
- [ ] DEV와 LIVE를 분리했는가
- [ ] Replay Protection이 필요한가
- [ ] SQL 실행 로그가 남는가
- [ ] Backup/Restore를 실제 테스트했는가
- [ ] SSH를 평상시 비활성화할 수 있는가
- [ ] phpMyAdmin 외부 공개가 반드시 필요한가
- [ ] Firewall/IP Allowlist/VPN 적용 가능성을 검토했는가

### Tunnel

- [ ] Tunnel 비용/플랜 조건을 최신 문서에서 다시 확인했는가
- [ ] PC/NAS 재부팅 후 자동 연결되는가
- [ ] Tunnel Client 종료 시 자동복구되는가
- [ ] 연결 끊김을 감지하는가
- [ ] 장애 알림이 있는가
- [ ] Tunnel을 사용하지 못할 때 대체 경로가 있는가

### 비용

- [ ] ChatGPT 플랜 비용
- [ ] MCP/앱 관련 Credit/사용량 정책
- [ ] Secure MCP Tunnel 정책
- [ ] OpenAI API를 추가로 사용하는 경우 API 비용
- [ ] NAS 전력/도메인/인증서/백업 비용
- [ ] 외부 SMTP 또는 메일 서비스 비용
- [ ] 정책 변경 후 비용 증가 가능성

### 운영

- [ ] NAS 장애 시 대응방법
- [ ] 인증서 만료 대응
- [ ] DDNS 장애 대응
- [ ] DB Lock/Transaction 정책
- [ ] SQL 오류 시 Rollback
- [ ] Mail 실패 시 재전송
- [ ] 관리자 계정/SSH 활성화 절차
- [ ] 장애 해결 후 임시 진단 파일 삭제

---

# 결론

## 37. 도입 판단 시 함께 봐야 할 것

ChatGPT와 NAS를 MCP/HTTPS API로 연결하면 개인 또는 조직의 DB를 ChatGPT의 Tool로 조회하고, 허용된 범위에서 변경하는 환경을 만들 수 있다.

하지만 **연결에 성공했다는 사실만으로 도입 여부를 결정하면 안 된다.**

도입 전에 최소한 다음 다섯 가지를 같이 판단해야 한다.

### 1. 보안 위험

MCP, 외부 API, Generic SQL, phpMyAdmin 공개는 모두 새로운 공격면을 만든다.

RSA Signature, HTTPS, Timestamp, Nonce, DB 권한 제한은 위험을 줄이기 위한 조치이지 완전한 보안을 의미하지 않는다.

### 2. ChatGPT의 제품 제한

MCP Read/Write 지원 플랜, Developer Mode, Agent Mode, Deep Research, 다른 App과의 조합 등은 제품 정책에 영향을 받는다.

이번 구축 과정에서도 Custom MCP 사용 대화에서 Gmail 호출이 제한되는 상황을 실제로 경험했다.

### 3. 데이터 보관과 내부 정책

NAS에서 데이터를 읽을 수 있는 것과 그 데이터를 ChatGPT에 전달해도 되는 것은 별개의 문제다.

MCP 도입 전 OpenAI의 Data Controls, App 데이터 처리, Memory, 데이터 보관정책과 조직 내부 보안정책을 충분히 확인해야 한다.

### 4. 비용과 정책 변경

Tunnel은 테스트 당시 별도 Tunnel 사용료 없이 사용했지만 향후 정책은 변경될 수 있다.

ChatGPT 플랜, Credits, API 사용, Tunnel, 메일, NAS 운영비용을 함께 검토해야 한다.

### 5. 가용성과 우회 경로

실제 테스트에서는 Tunnel이 끊긴 상태에서 ChatGPT의 NAS 작업이 중단됐다.

Custom MCP와 Gmail을 함께 사용할 때도 제한을 경험했으며, 이 경우 NAS가 직접 Mail을 발송하는 구조를 대안으로 사용했다.

즉 실제 운영에는 반드시 장애 시 우회 경로가 필요하다.

---

## 38. 이번 구축에서 최종적으로 확인한 것

```text
ChatGPT → NAS HTTPS/MCP 연결
API 인증
MariaDB 조회
MariaDB 등록/수정
Generic SQL
Migration / Validation
RSA/SHA256 Request Signature
Timestamp / Nonce
Replay Protection
Secure MCP Tunnel 실제 연결
Tunnel 중단 시 가용성 문제
DSM Reverse Proxy
Synology phpMyAdmin HTTPS Proxy
phpMyAdmin Session Cookie 문제 해결
SSH 기반 Runtime/권한 진단
작업 종료 후 SSH/admin 비활성화
Custom MCP + Gmail 제한 경험
NAS Mail Sender 대안
```

이 문서의 목적은 특정 구성을 정답으로 제시하는 것이 아니라, 실제 구축 과정에서 성공한 방법뿐 아니라 **실패한 접근, 변경 이유, 운영 중 발견한 제약과 도입 전 확인해야 할 조건까지 함께 공유하는 것**이다.

---

# 공식 참고자료

OpenAI 관련 기능은 변경될 수 있으므로 아래 문서는 도입 시점에 다시 확인한다.

- Developer mode and MCP apps in ChatGPT  
  https://help.openai.com/en/articles/12584461-developer-mode-apps-and-full-mcp-connectors-in-chatgpt-beta

- ChatGPT 개발자 모드 및 MCP 앱  
  https://help.openai.com/ko-kr/articles/12584461-developer-mode-and-full-mcp-connectors-in-chatgpt-beta

- API Key Safety  
  https://help.openai.com/en/articles/5112595

- API Key 안전 모범 사례  
  https://help.openai.com/ko-kr/articles/5112595

- OpenAI Account Security  
  https://help.openai.com/en/articles/8304786

- Data Controls FAQ  
  https://help.openai.com/en/articles/7730893-datacontrols-faq

- Google App for ChatGPT – Data Controls FAQ  
  https://help.openai.com/en/articles/10408842-google-connector-for-chatgpt-data-controls-faq

- Connected Apps  
  https://help.openai.com/en/collections/12923329

- ChatGPT Pricing  
  https://openai.com/business/chatgpt-pricing/

---

## 공개 전 마지막 확인

```text
실제 NAS Domain/IP
실제 DB 이름
실제 사용자명/이메일
DB Password
API Key
Private Key
Token
실제 인증서
Connection String
실제 SMTP Password
실제 수신자 정보
민감한 Screen Capture
```