링크: https://jyun627.notion.site/computer-network-week05?source=copy_link

[2.2 웹과 HTTP.pdf](attachment:3fdcaadc-f704-4313-a776-b2ed42e11326:2.2_웹과_HTTP.pdf)

## 2.2 웹과 HTTP

> HTTP는 웹 클라이언트가 웹 서버에게 웹 페이지를 어떻게 요청하는지와 서버가 클라이언트로 어떻게 웹 페이지를 전송하는지를 정의한다.

- HTTP는 TCP 프로토콜 사용
- 클라이언트: HTTP 요청 → 소켓 인터페이스로 보냄
  - 반대측과 서버도 마찬가지
- 비상태 프로토콜 (stateless protocol)

### 2.2.2 비지속 연결과 지속 연결

비지속 연결 HTTP

서버 → 클라이언트 전송 단계

```java
http://www.someschool.edu/someDepartment/home.index
```

1. HTTP 클라이언트는, 80포트를 통해 `www.someschool.edu` 서버로 TCP 연결을 시도
   1. TCP 연결과 관련하여 클-서에 각각 소켓이 있음
2. HTTP 클라이언트는 1단계에서 설정된 TCP 소켓을 통해 서버로 HTTP 요청 메세지를 보냄
   1. 이 요청 메세지에는 `/someDepartment/home.index` 경로 이름 포함
3. HTTP 서버는 1단계에서 설정된 연결 소켓을 통해 요청 메시지를 받음
   1. 저장장치로부터 `/someDepartment/home.index` 객체 추출
   2. HTTP 응답 메시지에 그 객체를 캡슐화
   3. 응답 메시지를 소켓을 통해 클라이언트로 보냄
4. HTTP 서버는 TCP에게 TCP 연결을 끊으라고 함
   1. 실제로 TCP 클라이언트가 응답 메시지를 올바로 받을 때까지 연결 끊지 않음
5. HTTP 클라이언트가 응답 메시지를 받으면, TCP 연결이 종료됨
   1. 메시지는 캡슐화된 객체가 HTML 파일인 것을 나타냄
   2. 클라이언트는 응답 메시지로부터 파일을 추출 → HTML 파일 조사 → 10개의 JPEG 객체에 대한 참조를 갖는다
6. 그 이후 참조되는 각 JPEG 객체에 대해 처음 네 단계 반복

![image.png](attachment:a0afdbd5-4d03-41b7-9716-8fe71e82f09b:image.png)

# 2.2 Web and HTTP 정리

## 1. Web 개요

- Web은 **on-demand 서비스**
  - 사용자가 원하는 시점에 콘텐츠 요청
- Web 페이지 = 여러 **객체(object)**의 집합
  - 객체: URL로 식별 가능한 파일
  - 예: HTML, 이미지(JPEG), CSS, JS 등

### 구성 구조

- 기본 HTML 파일 + 여러 참조 객체
- HTML이 다른 객체들의 URL을 포함

### URL 구조

- `http://host/path`
  - host: 서버 주소
  - path: 객체 위치 경로

## 2. HTTP 개요

### 정의

- Web의 **Application Layer Protocol**
- 클라이언트 ↔ 서버 간 메시지 교환 규칙 정의

### 구성

- Client: 브라우저
- Server: 웹 서버 (Apache 등)

### 동작 방식

1. 클라이언트가 요청(Request)
2. 서버가 응답(Response)

## 3. HTTP 특징

### 1) TCP 기반

- HTTP는 **TCP 위에서 동작**
- 신뢰성 보장 (loss, 순서 문제 해결은 TCP 담당)

### 2) Stateless

- 서버는 클라이언트 상태를 저장하지 않음
- 같은 요청 반복 시 동일하게 처리

## 4. HTTP 연결 방식

## 4.1 Non-Persistent Connection

### 특징

- 요청마다 TCP 연결 새로 생성

### 동작 과정

1. TCP 연결 생성
2. HTTP 요청 전송
3. HTTP 응답 수신
4. TCP 연결 종료

### 특징 정리

- 객체 1개당 TCP 연결 1개
- 웹페이지에 객체 11개 → TCP 연결 11개

### 단점

- 연결 오버헤드 큼
- RTT 지연 증가

### 시간 분석

- 총 시간 ≈
  - 2 RTT + 전송 시간

## 4.2 Persistent Connection

### 특징

- 하나의 TCP 연결을 계속 재사용

### 장점

- RTT 감소
- 연결 생성 비용 감소
- 성능 향상

### 추가 개념

- **Pipelining**
  - 응답 기다리지 않고 요청 연속 전송 가능

## 5. HTTP 메시지 구조

## 5.1 Request Message

### 예시

```
GET /somedir/page.html HTTP/1.1
Host: www.someschool.edu
Connection: close
User-agent: Mozilla/5.0
Accept-language: fr
```

### 구조

1. Request Line
   - Method + URL + Version
2. Header Lines
3. Blank Line
4. Entity Body (선택)

### 주요 Method

- GET: 객체 요청
- POST: 데이터 전송
- HEAD: 헤더만 요청
- PUT: 업로드
- DELETE: 삭제

## 5.2 Response Message

### 예시

```
HTTP/1.1 200 OK
Connection: close
Date: ...
Server: Apache
Content-Length: ...
Content-Type: text/html
```

### 구조

1. Status Line
   - Version + Status Code + Message
2. Header Lines
3. Blank Line
4. Entity Body (실제 데이터)

## 6. HTTP 상태 코드

| 코드                      | 의미        |
| ------------------------- | ----------- |
| 200 OK                    | 요청 성공   |
| 301 Moved Permanently     | URL 변경    |
| 400 Bad Request           | 요청 오류   |
| 404 Not Found             | 리소스 없음 |
| 505 Version Not Supported | 버전 미지원 |

## 7. Cookie (상태 유지)

### 등장 이유

- HTTP는 stateless → 사용자 식별 필요

### 구성 요소 (4가지)

1. Response: `Set-Cookie`
2. Request: `Cookie`
3. 클라이언트 쿠키 파일
4. 서버 DB

### 동작 흐름

1. 서버가 ID 생성
2. 응답에 Set-Cookie 포함
3. 브라우저 저장
4. 이후 요청마다 Cookie 전송

- 쿠키: 브라우저에 저장되는 정보
- 세션: 서버가 관리하는 사용자 상태 정보

1. 서버가 세션 생성
2. 세션 ID를 쿠키로 브라우저에 저장
3. 브라우저가 요청마다 세션ID 쿠키 전송
4. 서버가 세션 ID로 사용자 상태 조회

쿠키는 세션을 식별하는 수단으로 많이 사용된다.

### 역할

- 사용자 식별
- 로그인 유지
- 장바구니 기능

## 8. Web Caching (Proxy)

### 정의

- 서버 대신 요청 처리하는 캐시 서버

### 동작 과정

1. 클라이언트 → 캐시 요청
2. 캐시에 있으면 바로 응답
3. 없으면 서버에서 가져옴

### 장점

- 응답 속도 향상
- 트래픽 감소

웹 캐시 서버는 브라우저에 달린 서버가 아니다! 네트워크 중간에 존재하는 별도 서버

보통

- ISP 내부 캐시 (통신사)
- 기업 내부 프록시 서버
- CDN 등에 존재함

## 조건부 GET 발생 조건

### 1. 캐시된 데이터가 있지만 “신선도(freshness)”가 만료됨

즉:

- 캐시에 있음
- 그런데 오래돼서 확신이 없음

→ 이때 서버에 확인

## 3-1. 신선도 기준 (대표 2가지)

### (1) Cache-Control: max-age

```
Cache-Control: max-age=60
```

→ 60초 동안은 **무조건 캐시 사용 (서버 안 감)**

→ 60초 지나면:

👉 조건부 GET 발생

### (2) Expires 헤더

```
Expires: Wed, 01 Apr 2026 10:00:00 GMT
```

→ 이 시간 지나면 stale 상태

## 3-2. 흐름 정리

```
[1] 캐시에 있음 + 아직 fresh
→ 그냥 사용 (서버 요청 없음)

[2] 캐시에 있음 + stale
→ 조건부 GET (서버 확인)

[3] 캐시에 없음
→ 일반 GET (전체 다운로드)
```

# 4. 조건부 GET은 “주기적으로” 보내나?

→ ❌ 아니다

조건부 GET은:

- “매 N초마다 자동 전송” ❌
- “요청이 들어왔을 때 필요하면” ⭕

# 2.3 Electronic Mail (Email)

## 1. 구성 요소 (시험 핵심)

이메일 시스템은 **3개 컴포넌트**로 구성됨

- **User Agent (UA)**
  - 사용자 인터페이스 (Gmail, Outlook 등)
- **Mail Server**
  - 메일 저장, 전달 담당
- **SMTP (Simple Mail Transfer Protocol)**
  - 메일 전송 프로토콜

---

## 2. 전체 동작 흐름 (중요)

1. 사용자 A → UA에서 메일 작성
2. A의 Mail Server로 전달
3. SMTP로 **상대 Mail Server로 전송**
4. 상대 서버에 저장
5. 사용자 B가 접근하여 읽음

👉 핵심:

- **메일 전송 = SMTP**
- **메일 읽기 = 다른 프로토콜 (POP3/IMAP)**

---

## 3. SMTP 특징 (시험 포인트)

- **Application layer protocol**
- **TCP 사용**
- **Push protocol**
  - 보내는 쪽이 밀어 넣음
- **3단계 과정**
  1. Handshake
  2. Message transfer
  3. Connection 종료

---

## 4. Mail Message Format

- Header + Body 구조
- Header 예시:
  - From:
  - To:
  - Subject:

---

## 5. Mail Access Protocol (중요 비교)

| 프로토콜 | 특징                                  |
| -------- | ------------------------------------- |
| **POP3** | 서버에서 메일 다운로드 후 삭제 (단순) |
| **IMAP** | 서버에 메일 유지 (동기화 가능)        |

👉 시험 포인트

- SMTP = 보내기
- POP3/IMAP = 읽기

---

# 2.4 DNS (Domain Name System)

## 1. DNS 역할 (핵심)

👉 **hostname → IP address 변환**

- 예:`www.google.com → 142.250.xxx.xxx`

👉 인터넷의 **Directory Service**

---

## 2. DNS가 제공하는 서비스

(시험에 자주 나옴)

1. Hostname → IP 변환
2. Host aliasing (별칭)
3. Mail server aliasing
4. Load distribution (여러 IP 반환)

---

## 3. DNS 구조 (핵심 그림 개념)

👉 **분산 계층 구조 (hierarchical)**

- Root DNS
- TLD DNS (.com, .org 등)
- Authoritative DNS

👉 이유:

- 중앙 서버로 하면 확장 불가능
- 분산 구조로 scalability 확보

---

## 4. DNS 동작 방식 (시험 핵심)

### (1) Iterative Query

- 클라이언트가 직접 계속 질의

### (2) Recursive Query

- DNS 서버가 대신 찾아줌

📌 실제:

- Local DNS가 recursive 수행
- 이후 iterative 방식으로 내려감

---

## 5. DNS Query 흐름 (중요)

1. Host → Local DNS
2. Local DNS → Root DNS
3. Root → TLD DNS
4. TLD → Authoritative DNS
5. IP 주소 반환

---

## 6. DNS Cache (자주 출제)

- Local DNS가 결과 저장
- 이후 빠른 응답 가능

👉 대부분 요청은 Root까지 안 감

---

## 7. DNS Resource Record (RR)

형식:

```
(Name, Value, Type, TTL)
```

대표 타입:

- **A**: hostname → IP
- **NS**: DNS 서버 정보
- **MX**: 메일 서버

👉 DNS 응답에는 RR 포함됨

---

## 8. DNS 메시지 구조

- Query / Response
- 여러 RR 포함 가능
