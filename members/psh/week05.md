# week05 - psh

## 1) 범위
- Primary (8th): Ch2 2.2~2.4 p.125~165
- v9 PPT: Ch2 슬라이드 18~83
- Keyword: HTTP / Email / DNS core
- Deliverable: HTTP-Email-DNS 흐름 비교표

---

## 2) 핵심 요약

### 2.2 The Web and HTTP

#### HTTP 개요 (2.2.1)
- HTTP = 웹의 **애플리케이션 계층 프로토콜** (RFC 1945 / 7230 / 7540)
- 기반: **TCP** — HTTP는 데이터 손실·재정렬 걱정 불필요 (TCP가 처리)
- **무상태(stateless)**: 서버는 이전 요청 정보를 저장하지 않음
- 버전: HTTP/1.0(비영구) → HTTP/1.1(영구, 기본) → HTTP/2(프레이밍) → HTTP/3(QUIC)

#### 비영구적 vs 영구적 연결 (2.2.2)

| 구분 | 비영구적(HTTP/1.0) | 영구적(HTTP/1.1 기본) |
|---|---|---|
| TCP 연결 | 객체마다 새 연결 | 하나의 연결로 여러 객체 |
| 비용 | 객체당 2 RTT + 서버 부담 큼 | 파이프라이닝으로 효율적 |
| TCP 연결 수(HTML+10 JPEG) | 11개 | 1개 |

- **RTT(Round Trip Time)**: 작은 패킷이 클라이언트→서버→클라이언트 왕복하는 시간
- 비영구적: TCP 핸드셰이크(1 RTT) + HTTP 요청/응답(1 RTT) + 전송 = 총 **2 RTT + 전송 시간**

#### HTTP 메시지 형식 (2.2.3)

**요청 메시지 (Request)**:
```
GET /somedir/page.html HTTP/1.1
Host: www.someschool.edu
Connection: close
User-agent: Mozilla/5.0
Accept-language: fr
```
- 구조: 요청 줄(메소드+URL+버전) + 헤더 줄 + 빈 줄 + 엔터티 바디
- 메소드: GET(객체 요청) / POST(폼 전송) / HEAD(바디 없이 응답) / PUT(업로드) / DELETE

**응답 메시지 (Response)**:
```
HTTP/1.1 200 OK
Connection: close
Date: Tue, 18 Aug 2015 15:44:04 GMT
Last-Modified: Tue, 18 Aug 2015 15:11:03 GMT
Content-Length: 6821
Content-Type: text/html
(data...)
```
- 주요 상태 코드: 200 OK / 301 Moved Permanently / 400 Bad Request / 404 Not Found / 505

#### 쿠키 (2.2.4)
- HTTP 무상태의 한계 보완 → **쿠키**로 사용자 식별
- 4가지 구성요소: 응답의 `Set-cookie:` / 요청의 `Cookie:` / 브라우저 쿠키 파일 / 서버 백엔드 DB
- 프라이버시 우려: 사용자 정보와 결합 → 제3자 판매 가능성

#### 웹 캐싱 (2.2.5)
- **프록시 서버**: 오리진 서버 대신 요청 처리 → 응답 시간 단축 + 트래픽 감소
- 동작: ①로컬 캐시 확인 → ②있으면 즉시 반환 / 없으면 오리진 서버에서 가져와 저장
- **조건부 GET**: `If-Modified-Since:` 헤더 → 수정 안 됐으면 `304 Not Modified` (바디 없음 → 대역폭 절약)
- **CDN** (Akamai, Limelight 등)도 이 원리 활용

#### HTTP/2 (2.2.6)
- **HOL 블로킹 문제**: HTTP/1.1 단일 TCP에서 큰 객체가 작은 객체를 막음
  - 브라우저들이 우회책으로 최대 6개 병렬 TCP 연결 사용 → TCP 혼잡 제어 우회 (불공정)
- **해결: 프레이밍(Framing)**
  - 메시지를 독립적인 프레임으로 분할 → 동일 TCP 연결에서 인터리브
  - 예: 비디오(1000프레임) + 소형 8개(각 2프레임) → **18프레임 후** 소형 전부 완료 (인터리빙 없이는 1016프레임 후)
  - 프레임을 **이진(binary)** 인코딩 → 파싱 효율적
- **추가 기능**: 요청 우선순위 지정(1~256 가중치) / 서버 푸시(클라이언트 요청 전 객체 미리 전송)
- **HTTP/3**: QUIC (UDP 기반 전송 프로토콜) 위에서 동작, 메시지 다중화 포함

---

### 2.3 인터넷 전자 메일 (Electronic Mail)

#### 구성 요소 3가지
1. **사용자 에이전트(UA)**: Outlook, Gmail, Apple Mail — 메시지 작성·읽기
2. **메일 서버**: 메일함(mailbox) + 발신 대기열(message queue) 관리
3. **SMTP**: 메일 서버 간 전송 프로토콜 (TCP 포트 25), 인터넷 초기부터 존재한 레거시

#### SMTP 동작 과정 (2.3.1)

**전체 흐름**:
```
[Alice UA] →SMTP/HTTP→ [Alice 메일 서버] →SMTP→ [Bob 메일 서버] →HTTP/IMAP→ [Bob UA]
```

1. Alice가 메시지 작성 → UA가 Alice 메일 서버에 전송 → 대기열 배치
2. Alice 메일 서버(SMTP 클라이언트) → Bob 메일 서버에 **TCP 연결(포트 25)** 수립
3. SMTP 핸드셰이크 + 메시지 전송
4. Bob 메일 서버가 수신 → Bob의 메일함에 저장
5. Bob이 편한 시간에 UA로 메일 확인

**SMTP 대화 예시**:
```
S: 220 hamburger.edu
C: HELO crepes.fr
S: 250 Hello crepes.fr, pleased to meet you
C: MAIL FROM: <alice@crepes.fr>
S: 250 Sender ok
C: RCPT TO: <bob@hamburger.edu>
S: 250 Recipient ok
C: DATA
S: 354 Enter mail, end with "." on a line by itself
C: Do you like ketchup?
C: How about pickles?
C: .
S: 250 Message accepted for delivery
C: QUIT
S: 221 hamburger.edu closing connection
```

- **7비트 ASCII 제한**: 바이너리·멀티미디어 전송 시 인코딩 필요 (레거시 제약)
- **지속 연결**: 같은 수신 서버로 여러 메일 전송 시 동일 TCP 연결 재사용
- **중간 서버 없음**: 두 서버가 지구 반대편이어도 직접 TCP 연결

#### 메일 메시지 형식 (2.3.2)
- RFC 5322: 헤더 + 빈 줄(CRLF) + 본문
- 필수 헤더: `From:`, `To:` / 선택: `Subject:` 등
- 주의: SMTP 명령어(`MAIL FROM`, `RCPT TO`)와 메시지 헤더(`From:`, `To:`)는 별개

#### 메일 액세스 프로토콜 (2.3.3)
- SMTP는 **푸시(push)** 프로토콜 → 메일 서버에서 가져오기(pull)에는 불적합
- 수신 방법:
  - **HTTP**: 웹메일(Gmail), 스마트폰 앱 (Bob의 메일 서버에 HTTP + SMTP 인터페이스 모두 필요)
  - **IMAP** (RFC 3501): Outlook 등 메일 클라이언트, 폴더 관리·이동·삭제 가능

---

### 2.4 DNS — 인터넷의 디렉터리 서비스

#### DNS 개요 (2.4.1)
- **DNS(Domain Name System)**: 호스트명 → IP 주소 변환
  1. **분산 데이터베이스** (DNS 서버 계층 구조)
  2. **애플리케이션 계층 프로토콜** (UDP, 포트 53)
- DNS는 HTTP, SMTP 등 다른 앱이 사용하는 인프라 — 사용자가 직접 쓰지 않음
- BIND 소프트웨어 (UNIX)가 가장 널리 사용

#### DNS가 제공하는 서비스 (2.4.1)

1. **호스트명 → IP 변환**: 주된 역할
   - 브라우저가 `www.someschool.edu` 요청 → DNS 조회 → IP 획득 → TCP 연결
2. **호스트 별칭(Host Aliasing)**: 복잡한 정식명에 기억하기 쉬운 별칭 제공
3. **메일 서버 별칭**: MX 레코드로 웹 서버와 메일 서버에 같은 별칭 허용
4. **부하 분산**: IP 주소 집합을 round-robin으로 반환 → 복제 서버 간 트래픽 분산

#### DNS 동작 원리 (2.4.2)

**중앙 집중형의 문제점**: 단일 실패 지점 / 트래픽 집중 / 지리적 지연 / 유지보수 어려움 → 확장성 없음

**3(+1)계층 분산·계층 구조**:

| 계층 | 설명 | 역할 |
|---|---|---|
| 루트 DNS | 전 세계 1000+개 인스턴스, 13개 루트의 복제본 (IANA 관리) | TLD 서버 IP 제공 |
| TLD DNS | .com, .org, .edu 등 (Verisign이 .com 관리) | 권한 서버 IP 제공 |
| 권한 있는 DNS | 조직 자체 운영 or 위탁 | 실제 호스트명-IP 매핑 보관 |
| 로컬 DNS | ISP가 보유, 계층 구조로 전달하는 프록시 | 계층 외부지만 핵심 역할 |

**DNS 조회 예시** (cse.nyu.edu → gaia.cs.umass.edu):
1. cse.nyu.edu → 로컬 DNS 서버 dns.nyu.edu (**재귀 질의**)
2. dns.nyu.edu → 루트 DNS 서버 (**반복 질의**)
3. 루트 → edu TLD 서버 IP 반환
4. dns.nyu.edu → edu TLD 서버
5. edu TLD → dns.umass.edu IP 반환
6. dns.nyu.edu → dns.umass.edu
7. dns.umass.edu → gaia.cs.umass.edu IP 반환
8. dns.nyu.edu → cse.nyu.edu에 IP 전달

→ 총 **8개 DNS 메시지** (4 쿼리 + 4 응답)

**재귀 vs 반복 질의**:
- 재귀: 요청 측이 "대신 해결해줘" → 호스트→로컬DNS 구간
- 반복: 서버가 다음 서버를 알려줌 → 로컬DNS→루트/TLD/권한 구간

**DNS 캐싱**: 응답 수신 시 로컬에 저장 (TTL 일반적으로 2일) → 루트 서버 우회 가능 (실제로 거의 대부분의 쿼리가 캐싱으로 해결)

#### DNS 레코드와 메시지 (2.4.3)

**자원 레코드(RR)** 형식: `(Name, Value, Type, TTL)`

| Type | Name | Value | 예시 |
|---|---|---|---|
| A | 호스트명 | IP 주소 | (relay1.bar.foo.com, 145.37.93.126, A) |
| NS | 도메인 | 권한 서버 호스트명 | (foo.com, dns.foo.com, NS) |
| CNAME | 별칭 호스트명 | 정식 호스트명 | (foo.com, relay1.bar.foo.com, CNAME) |
| MX | 별칭 | 메일 서버 정식명 | (foo.com, mail.bar.foo.com, MX) |

**DNS 메시지 구조**: 쿼리·응답 동일한 형식
- 헤더(12바이트): 쿼리/응답 구분 플래그, 재귀 요청/지원 플래그
- 질문 섹션: 질의 이름 + 타입(A, MX 등)
- 응답 섹션: 자원 레코드 (Type, Value, TTL)
- 권한 섹션: 다른 권한 서버 레코드
- 추가 섹션: 유용한 추가 레코드 (예: MX 응답 시 메일 서버 IP도 포함)

**DNS 보안 취약점**:
- DDoS 플러딩: 2002년 루트 서버 공격 (피해 최소), 2016년 Dyn 공격 (Mirai 봇넷, IoT 10만 대)
- 중간자 공격: 쿼리 가로채 허위 응답
- DNS 포이즈닝: 서버에 허위 레코드 캐시 유도 → 사용자를 공격자 웹사이트로 리디렉션
- 대응: **DNSSEC** (DNS Security Extensions, RFC 4033)

---

## 3) HTTP-Email-DNS 흐름 비교표 (Deliverable)

| 항목 | HTTP | SMTP(이메일) | DNS |
|---|---|---|---|
| 역할 | 웹 페이지 전송 | 이메일 전송 | 호스트명→IP 변환 |
| 전송 방향 | **풀(pull)**: 서버에서 가져옴 | **푸시(push)**: 서버로 보냄 | 요청-응답 |
| 기반 전송 | TCP (포트 80) | TCP (포트 25) | **UDP** (포트 53) |
| 연결 방식 | 영구적 연결 (기본) | 지속 연결 | 비연결 |
| 상태 유지 | 무상태 — 쿠키로 보완 | 상태 있음 (대기열 관리) | 캐싱으로 성능 향상 |
| 인코딩 제약 | 없음 | **7비트 ASCII 제한** | 해당 없음 |
| 서버 구조 | 중앙집중형 (데이터센터) | 도메인별 분산 | **3계층 계층적 분산** |
| 데이터 형식 | 요청/응답 메시지 | RFC 5322 (헤더+본문) | RR (Name, Value, Type, TTL) |
| 주요 RFC | 7230 (HTTP/1.1), 7540 (HTTP/2) | RFC 5321 (SMTP), 3501 (IMAP) | RFC 1034, 1035 |
| 캐싱 | 웹 캐시/프록시 서버 | 없음 | TTL 기반 DNS 캐시 |

---

## 4) v8 vs v9 차이점 (최소 3개)
1. **HTTP/2 독립 절로 확장**: v8에서 간략 소개됐던 HTTP/2가 v9에서 2.2.6절로 독립 → HOL 블로킹·프레이밍·서버 푸시 상세 설명 추가
2. **HTTP/3 신규 추가**: v9에서 QUIC 기반 HTTP/3이 새로운 절로 추가됨 (v8에는 없음)
3. **IMAP 중심으로 단순화**: v9에서 POP3가 사실상 제외되고 IMAP + HTTP 방식으로 메일 액세스 정리
4. **DNS 보안 사례 갱신**: v9 PPT에 2016년 Dyn DDoS 공격 (Mirai 봇넷, IoT 기기) 사례 추가

---

## 5) 헷갈린 포인트 / 질문 (최소 2개)
1. **비영구적 연결의 "2 RTT"**: TCP 핸드셰이크 1 RTT + HTTP 요청/응답 1 RTT = 2 RTT인데, 파이프라이닝 없는 영구적 연결에서는 객체마다 1 RTT씩만 추가되는가? (첫 객체만 2 RTT?)
2. **DNS 조회 시 캐시 효과**: 실제 환경에서 루트 서버까지 가는 쿼리는 얼마나 되는가? TLD 서버 IP까지 캐싱되면 루트를 완전히 우회할 수 있는데, TTL 만료 이후 재조회 패턴은?
3. **MX vs CNAME 우선순위**: 같은 도메인에 웹+메일 서버를 두려면 MX를 써야 하는데, CNAME과의 차이 및 공존 가능 여부는?

---

## 6) 발표 포인트 (1분 버전)
> **"DNS의 분산·계층 구조"**
>
> DNS는 인터넷의 전화번호부입니다.
> www.google.com을 치면 브라우저가 직접 서버를 찾는 게 아니라 먼저 DNS 쿼리가 갑니다.
> DNS 서버가 하나뿐이라면 → 전 세계 모든 요청이 집중 → 단일 실패 지점, 엄청난 지연.
> 그래서 루트→TLD→권한 서버의 3계층 분산 구조 + 로컬 캐싱으로 대부분의 쿼리가 근처에서 해결됩니다.
> 실제로 루트 서버까지 가는 쿼리는 극소수입니다.

---

## 7) 참고 링크
- 스터디 공통 주차표: [`weeks/week05.md`](../../weeks/week05.md)
- 교재: Kurose & Ross, *Computer Networking: A Top-Down Approach*, 8th Edition, Ch2 p.125~165
- Kurose & Ross PPT (9th): [gaia.cs.umass.edu/kurose_ross/ppt.php](https://gaia.cs.umass.edu/kurose_ross/ppt.php)
- Notion 정리: https://www.notion.so/2cb9422c9c644e6b971472dea47b7fc5
- nslookup / Wireshark 실습 (교재 부록)
