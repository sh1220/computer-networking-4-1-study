| 주차 | 기간 | Primary(v8) 교재 범위 | v9 PPT 범위 |
|---|---|---|---|
| 5주차 | 03-31 ~ 04-06 | Ch2 2.2~2.4 (p.125~165): HTTP / Email / DNS core | Ch2 슬라이드 18~83 |

# Chapter 2 Application Layer (계속)

# 2.2 The Web and HTTP

## 웹(World Wide Web)이란?

- 1990년대 초 등장한 **월드 와이드 웹**은 일반 대중에게 인터넷을 처음으로 알린 **킬러 애플리케이션**
- 웹의 핵심 특징: **주문형(on demand)** — 원하는 것을, 원하는 때에 받을 수 있음 (방송 라디오/TV와 달리 콘텐츠 제공자의 일정에 맞출 필요 없음)
- 웹은 오늘날 YouTube, Gmail, Instagram, Google Maps 등 대부분의 인터넷 서비스의 플랫폼이자 기반

### 웹의 기본 개념: 객체, URL, 웹 페이지

- **웹 페이지(Web Page)**: 여러 **객체(object)**로 구성됨
- **객체**: 단 하나의 **URL**로 주소 지정 가능한 파일
	- 종류: HTML 파일, JPEG 이미지, JavaScript 파일, CSS 스타일시트, 비디오 클립 등
- 예시: HTML 텍스트 + 5개의 JPEG 이미지로 구성된 웹 페이지 = 총 **6개 객체**
- **URL 구조**: `http://www.someSchool.edu/someDepartment/picture.gif`
	- **호스트명(hostname)**: `www.someSchool.edu`
	- **경로명(path name)**: `/someDepartment/picture.gif`
- **웹 브라우저**: HTTP 클라이언트를 구현하는 프로그램 (Chrome, Internet Explorer 등)
- **웹 서버**: HTTP 서버 측을 구현, URL로 주소 지정 가능한 웹 객체를 저장 (Apache, Microsoft IIS 등)

## 2.2.1 HTTP 개요 (Overview of HTTP)

- **HTTP(HyperText Transfer Protocol)**: 웹의 **애플리케이션 계층 프로토콜**로 웹의 핵심
	- 정의: RFC 1945 (HTTP/1.0), RFC 7230 (HTTP/1.1), RFC 7540 (HTTP/2)
- HTTP는 **클라이언트 프로그램**과 **서버 프로그램** 두 개로 구현됨
	- 서로 다른 종단 시스템에서 실행되며 **HTTP 메시지**를 교환하여 통신
- **TCP를 기본 전송 프로토콜**로 사용 (UDP 아님)
	- HTTP 클라이언트가 먼저 서버와 **TCP 연결** 초기화
	- 연결 성립 후 브라우저와 서버는 **소켓 인터페이스**를 통해 TCP에 접근
	- TCP는 HTTP에 **신뢰적 데이터 전송 서비스** 제공 → HTTP는 데이터 손실·순서 재정렬 걱정 불필요
- **HTTP는 무상태(stateless) 프로토콜**: 서버가 이전 클라이언트 요청에 대한 정보를 유지하지 않음
	- 같은 클라이언트가 몇 초 내에 같은 객체를 두 번 요청해도 서버는 그냥 다시 전송함
- **HTTP 버전 히스토리**:
	- HTTP/1.0: 1990년대 초 (비영구적 연결)
	- HTTP/1.1: 2020년 기준 대부분의 트랜잭션이 여기서 (영구적 연결 기본)
	- HTTP/2: RFC 7540, 점점 더 많은 브라우저·서버가 지원 (다중화, 프레이밍)

## 2.2.2 비영구적 연결과 영구적 연결 (Non-Persistent and Persistent Connections)

- 중요한 설계 결정: 각 요청/응답 쌍을 **별도의 TCP 연결**로 할 것인가, **같은 TCP 연결**로 보낼 것인가?
	- **비영구적 연결(Non-persistent connections)**: 각 요청마다 새 TCP 연결 → HTTP/1.0 방식
	- **영구적 연결(Persistent connections)**: 같은 TCP 연결로 여러 요청/응답 → HTTP/1.1 기본 방식

### 비영구적 연결 (HTTP with Non-Persistent Connections)

- 예시: 기본 HTML 1개 + JPEG 이미지 10개를 포함하는 웹 페이지 전송
- **동작 단계**:
	1. HTTP 클라이언트가 포트 80으로 서버와 TCP 연결 초기화
	2. HTTP 요청 메시지 전송 (경로명 포함)
	3. 서버가 요청 수신 → 객체를 HTTP 응답 메시지에 담아 전송
	4. 서버가 TCP 연결 종료 지시
	5. 클라이언트가 응답 수신, TCP 연결 종료. HTML 파일 파싱 → 10개의 JPEG 참조 발견
	6. 각 JPEG 객체에 대해 위 단계 반복
- **총 TCP 연결 수**: 11개 (HTML 1개 + JPEG 10개)
- **RTT(Round Trip Time)**: 작은 패킷이 클라이언트 → 서버 → 클라이언트로 돌아오는 데 걸리는 시간
	- 각 객체 당 **2 RTT** 소요: 1 RTT(TCP 3방향 핸드셰이크) + 1 RTT(HTTP 요청/응답) + 파일 전송 시간
- **비영구적 연결의 단점**:
	- 각 연결마다 TCP 버퍼·변수 할당 필요 → 서버 부담 큼
	- 각 객체마다 2 RTT 지연 발생 → 성능 저하

### 영구적 연결 (HTTP with Persistent Connections)

- HTTP/1.1에서는 응답 전송 후에도 **TCP 연결을 유지**
- 같은 클라이언트와 서버 간 후속 요청/응답이 같은 연결로 전송 가능
- 전체 웹 페이지(HTML + 10개 이미지)를 **단일 영구 TCP 연결**로 전송 가능
- **파이프라이닝(Pipelining)**: 이전 요청의 응답을 기다리지 않고 연달아 요청 전송 가능
- 서버는 일정 시간(타임아웃) 동안 연결이 미사용되면 연결 종료
- **HTTP 기본 모드**: 파이프라이닝이 적용된 영구적 연결

## 2.2.3 HTTP 메시지 형식 (HTTP Message Format)

HTTP 메시지는 두 가지: **요청 메시지(Request Message)**와 **응답 메시지(Response Message)**

### HTTP 요청 메시지 (HTTP Request Message)

```
GET /somedir/page.html HTTP/1.1
Host: www.someschool.edu
Connection: close
User-agent: Mozilla/5.0
Accept-language: fr
```

- **요청 줄(Request Line)**: 메시지 첫 번째 줄
	- **메소드 필드**: `GET`, `POST`, `HEAD`, `PUT`, `DELETE` 등
	- **URL 필드**: 요청하는 객체의 경로
	- **버전 필드**: HTTP 버전 (예: HTTP/1.1)
- **헤더 줄(Header Lines)**: 요청 줄 이후의 줄들
	- `Host`: 객체가 위치한 호스트 (웹 프록시 캐시에서 필요)
	- `Connection: close`: 서버에게 영구 연결 대신 객체 전송 후 연결 종료 요청
	- `User-agent`: 요청하는 브라우저 종류/버전 (서버가 브라우저별 다른 버전 전송 가능)
	- `Accept-language`: 선호 언어 (콘텐츠 협상)
- **빈 줄(Blank Line)**: CRLF로 헤더와 엔터티 바디 구분
- **엔터티 바디(Entity Body)**: GET에서는 비어 있음, POST에서는 사용자 입력값 포함

### HTTP 메소드 정리

- **GET**: 가장 많이 사용. 브라우저가 객체를 요청할 때 사용. 엔터티 바디 없음
- **POST**: 사용자가 폼을 작성할 때 자주 사용. 엔터티 바디에 사용자 입력값 포함
	- 참고: HTML 폼은 GET 방식도 사용 가능 → URL에 입력값 포함 (예: `?monkeys&bananas`)
- **HEAD**: GET과 유사하지만 서버가 응답 메시지에서 요청 객체를 **제외**하고 전송. 디버깅용
- **PUT**: 특정 웹 서버의 특정 경로에 객체 업로드
- **DELETE**: 웹 서버의 객체 삭제

### HTTP 응답 메시지 (HTTP Response Message)

```
HTTP/1.1 200 OK
Connection: close
Date: Tue, 18 Aug 2015 15:44:04 GMT
Server: Apache/2.2.3 (CentOS)
Last-Modified: Tue, 18 Aug 2015 15:11:03 GMT
Content-Length: 6821
Content-Type: text/html
(data data data data data ...)
```

- **상태 줄(Status Line)**: 첫 번째 줄
	- 프로토콜 버전, **상태 코드(status code)**, 상태 메시지로 구성
- **헤더 줄**:
	- `Connection: close`: 메시지 전송 후 TCP 연결 종료
	- `Date`: HTTP 응답이 생성·전송된 시간 (객체 생성 시간 아님)
	- `Server`: 응답을 생성한 서버 종류 (User-agent와 유사)
	- `Last-Modified`: 객체가 생성되거나 마지막으로 수정된 시간 → **캐싱에 매우 중요**
	- `Content-Length`: 전송되는 객체의 바이트 수
	- `Content-Type`: 엔터티 바디의 객체 유형 (파일 확장자가 아닌 공식 표시 방법)
- **엔터티 바디**: 요청된 객체 자체

### 주요 HTTP 상태 코드

- **200 OK**: 요청 성공, 응답 메시지에 객체 포함
- **301 Moved Permanently**: 요청 객체가 영구 이동, `Location:` 헤더에 새 URL 명시
- **400 Bad Request**: 서버가 요청을 이해할 수 없음 (일반적 오류 코드)
- **404 Not Found**: 요청된 문서가 서버에 없음
- **505 HTTP Version Not Supported**: 요청 HTTP 버전을 서버가 지원하지 않음

## 2.2.4 사용자-서버 상호작용: 쿠키 (User-Server Interaction: Cookies)

- HTTP는 **무상태** 프로토콜이지만 웹 사이트가 사용자를 추적하고 맞춤 콘텐츠를 제공하려면 사용자 식별 필요
- 이를 위해 HTTP는 **쿠키(Cookie)** 사용 (RFC 6265)

### 쿠키의 4가지 구성 요소

1. HTTP **응답** 메시지의 쿠키 헤더 줄 (`Set-cookie: 1678`)
2. HTTP **요청** 메시지의 쿠키 헤더 줄 (`Cookie: 1678`)
3. 사용자 종단 시스템에 저장되어 **브라우저가 관리**하는 쿠키 파일
4. 웹 사이트의 **백엔드 데이터베이스**

### 쿠키 동작 예시

- Susan이 Amazon.com에 처음 접속 시:
	1. Amazon 서버가 **고유 식별 번호(예: 1678)** 생성 → 데이터베이스에 항목 생성
	2. 서버가 응답에 `Set-cookie: 1678` 헤더 삽입
	3. Susan의 브라우저가 `Set-cookie:` 헤더 확인 → 쿠키 파일에 `amazon.com, 1678` 저장
	4. 이후 Amazon 방문 때마다 요청 메시지에 `Cookie: 1678` 헤더 자동 포함
	5. Amazon 서버는 사용자 1678의 활동 이력 추적 가능
- **쿠키의 용도**: 쇼핑 카트, 사용자 인증 유지, 개인화 추천 등
- **프라이버시 문제**: 쿠키와 사용자 제공 정보를 결합하면 웹 사이트가 많은 정보 수집 가능 → 제3자에 판매 가능성 → 논란

## 2.2.5 웹 캐싱 (Web Caching)

- **웹 캐시(Web Cache) 또는 프록시 서버(Proxy Server)**: 오리진 웹 서버를 대신하여 HTTP 요청을 처리하는 네트워크 엔티티
- 자체 디스크 저장소 보유 → 최근 요청 객체의 사본 저장

### 웹 캐시 동작 단계

1. 브라우저가 웹 캐시와 **TCP 연결** 수립 → 캐시에 HTTP 요청 전송
2. 웹 캐시가 **로컬에 객체 사본**이 있는지 확인
	- 있으면: 캐시가 브라우저에 직접 HTTP 응답 반환
	- 없으면: 오리진 서버에 TCP 연결 → HTTP 요청 → 객체 수신 → 로컬에 저장 + 브라우저에 전달
- **웹 캐시는 서버이자 클라이언트**: 오리진 서버에는 클라이언트, 브라우저에는 서버

### 웹 캐시의 이점

- **인터넷 접속 링크의 트래픽 감소** → 대역폭 비용 절감
- **사용자 응답 시간 단축**: 인터넷을 거치지 않고 로컬 캐시에서 직접 제공
- 예시 계산: 기관의 인터넷 접속 링크 대역폭 증설(비용 많이 듦) 대신 웹 캐시 설치로 훨씬 낮은 평균 응답 시간 달성 가능
- **CDN(Content Distribution Network)**: 웹 캐시를 전 세계에 분산 배치하여 트래픽을 지역화

### 조건부 GET (Conditional GET)

- **문제**: 캐시에 저장된 객체가 오래되어 웹 서버에서 수정되었을 수 있음
- **해결**: HTTP의 조건부 GET 메커니즘
- 조건: (1) GET 메소드 사용 + (2) `If-Modified-Since:` 헤더 줄 포함
- **동작 예시**:
	1. 캐시가 웹 서버에 요청 → 서버가 `Last-Modified:` 헤더 포함 응답 + 캐시가 객체·수정 날짜 저장
	2. 일주일 후 동일 객체 요청 → 캐시가 `GET` + `If-Modified-since: [이전 Last-Modified 날짜]` 전송
	3. 객체가 수정되지 않았다면 서버가 `304 Not Modified` 응답 (객체 바디 없음) → 캐시가 기존 사본 전달
	4. 객체가 수정되었다면 `200 OK` + 새 객체 전송
- 장점: 수정되지 않은 객체의 중복 전송 방지 → 대역폭 절약, 응답 시간 단축

## 2.2.6 HTTP/2

- **HTTP/2** (RFC 7540): 2015년 표준화, 2020년 기준 상위 1,000만 개 웹사이트 중 40% 이상 지원
- HTTP/2의 핵심 목표:
	- 단일 TCP 연결에서 요청·응답 **다중화(multiplexing)** → 인지 지연 감소
	- 요청 **우선순위 지정** 및 **서버 푸시(Server Push)** 기능
	- HTTP **헤더 필드의 효율적 압축**
- HTTP 메소드, 상태 코드, URL, 헤더 필드는 변경 없음 → **전송 방식**만 변경

### HTTP/1.1의 문제: HOL(Head-of-Line) 블로킹

- HTTP/1.1 영구 연결에서 단일 TCP 연결로 웹 페이지 전송 시 **HOL 블로킹** 발생
	- 예: 대용량 비디오 클립 뒤에 소형 객체 여러 개 → 비디오가 병목 링크를 오래 점유 → 소형 객체 지연
	- HTTP/1.1 브라우저들이 HOL 우회를 위해 최대 **6개의 병렬 TCP 연결** 사용
- 병렬 연결의 문제: **TCP 혼잡 제어**를 우회하여 링크 대역폭을 더 많이 점유(불공정)

### HTTP/2 해결책: 프레이밍(Framing)

- 각 HTTP 메시지를 작은 **프레임(frame)**으로 분할
- 동일한 TCP 연결에서 요청·응답 메시지를 **인터리브(interleave)** → HOL 블로킹 해소
- 예시: 비디오(1000 프레임) + 소형 객체 8개(각 2 프레임) 시:
	- 프레이밍 없이: 소형 객체가 **1016프레임** 후에야 전송
	- 프레이밍 적용: **18프레임** 후 모든 소형 객체 전송 완료 → 인지 지연 대폭 감소
- 프레이밍 서브레이어가 프레임들을 **이진(binary)** 인코딩 → 파싱 효율적, 오류 감소

### HTTP/2 추가 기능

- **메시지 우선순위 지정**: 클라이언트가 요청에 1~256 사이 가중치 할당 → 서버가 높은 우선순위 응답 먼저 전송
- **서버 푸시(Server Push)**: 클라이언트가 별도로 요청하지 않아도 HTML 페이지에 필요한 객체를 서버가 미리 전송 → 추가 요청 대기 시간 제거

### HTTP/3

- **QUIC**: UDP 위 애플리케이션 계층에 구현된 새로운 "전송" 프로토콜
- QUIC의 특징: 메시지 다중화(인터리빙), 스트림별 흐름 제어, 낮은 지연의 연결 수립
- **HTTP/3**: QUIC 위에서 동작하도록 설계 (2020년 기준 인터넷 초안 단계)

---

# 2.3 인터넷 전자 메일 (Electronic Mail in the Internet)

- 전자 메일: 인터넷이 시작된 이래 존재. 초기 가장 인기 있는 애플리케이션
- 특성: **비동기 통신** 매체 — 다른 사람의 일정과 무관하게 편리할 때 보내고 읽음
- 현대 전자 메일: 첨부 파일, 하이퍼링크, HTML 형식 텍스트, 내장 사진 등 다양한 기능

## 인터넷 메일 시스템의 3가지 주요 구성 요소

1. **사용자 에이전트(User Agent)**: 사용자가 메시지를 읽고, 작성하고, 전송·수신하는 인터페이스
	- 예: Microsoft Outlook, Apple Mail, Gmail(웹/앱)
2. **메일 서버(Mail Server)**: 전자 메일 인프라의 핵심
	- 각 수신자는 하나의 메일 서버에 **메일함(mailbox)** 보유
	- 발신 메시지 대기열(Message Queue) 관리
3. **SMTP(Simple Mail Transfer Protocol)**: 인터넷 전자 메일을 위한 주요 애플리케이션 계층 프로토콜

### 메일 전송 경로

- Alice가 Bob에게 이메일 보내기:
	1. Alice의 사용자 에이전트 → Alice의 메일 서버 발신 대기열
	2. Alice의 메일 서버(SMTP 클라이언트) → Bob의 메일 서버(SMTP 서버) TCP 연결
	3. Bob의 메일 서버가 수신 → Bob의 메일함에 저장
	4. Bob이 사용자 에이전트로 메일함에서 메시지 가져옴

- **중간 메일 서버 없음**: SMTP는 일반적으로 두 서버가 지구 반대편에 있어도 **직접 연결**
- Bob의 서버 다운 시: Alice 서버가 메시지 대기열 보관 → 보통 **30분마다 재시도** → 여러 날 실패 시 삭제 + Alice에게 알림

## 2.3.1 SMTP

- **SMTP**: RFC 5321(원래 RFC는 1982년)에 정의. HTTP보다 훨씬 오래된 레거시 기술
- **SMTP의 중요한 제약**: 모든 메일 메시지 본문(헤더 포함)을 **7비트 ASCII**로 제한
	- 1980년대 초에는 적절했으나, 오늘날 멀티미디어 전송 시 바이너리→ASCII 인코딩 필요 → 불편

### SMTP 동작 과정

1. Alice가 사용자 에이전트로 메시지 작성 → 사용자 에이전트가 Alice 메일 서버로 전송 → 대기열 배치
2. Alice 메일 서버의 SMTP 클라이언트 → 대기열 확인 → Bob 메일 서버의 SMTP 서버에 **TCP 연결(포트 25)**
3. SMTP 핸드셰이크(발신자·수신자 이메일 주소 교환)
4. SMTP 클라이언트가 TCP 연결을 통해 메시지 전송 (신뢰적 데이터 전송)
5. Bob 메일 서버가 수신 → Bob 메일함에 저장
6. Bob이 편한 시간에 사용자 에이전트로 메시지 읽음

### SMTP 대화 예시 (SMTP Transcript)

```
S: 220 hamburger.edu
C: HELO crepes.fr
S: 250 Hello crepes.fr, pleased to meet you
C: MAIL FROM: <alice@crepes.fr>
S: 250 alice@crepes.fr ... Sender ok
C: RCPT TO: <bob@hamburger.edu>
S: 250 bob@hamburger.edu ... Recipient ok
C: DATA
S: 354 Enter mail, end with "." on a line by itself
C: Do you like ketchup?
C: How about pickles?
C: .
S: 250 Message accepted for delivery
C: QUIT
S: 221 hamburger.edu closing connection
```

- `HELO`: 인사 (HELLO 약자)
- `MAIL FROM`: 발신자 이메일 주소
- `RCPT TO`: 수신자 이메일 주소
- `DATA`: 메시지 본문 시작 선언
- 단일 마침표(.)로 메시지 끝 표시 (ASCII: `CRLF.CRLF`)
- `QUIT`: 연결 종료
- **지속 연결**: 같은 수신 서버로 여러 메시지 전송 시 동일 TCP 연결 재사용

### SMTP와 HTTP 비교

| 항목 | HTTP | SMTP |
|---|---|---|
| 전송 방향 | **풀(pull)** — 서버에서 가져옴 | **푸시(push)** — 서버로 보냄 |
| 인코딩 | ASCII 제한 없음 (이진 데이터 직접 전송) | **7비트 ASCII**로 제한 |
| 다수 객체 처리 | 각 객체 별도 응답 메시지 | 하나의 메시지 안에 모든 부분 포함 |

## 2.3.2 메일 메시지 형식 (Mail Message Formats)

- 메일 메시지: **헤더(Header)**와 **본문(Body)**으로 구성 (RFC 5322)
- 헤더와 본문은 **빈 줄(CRLF)**로 구분
- **필수 헤더 라인**: `From:`, `To:`
- **선택적 헤더 라인**: `Subject:` 등

```
From: alice@crepes.fr
To: bob@hamburger.edu
Subject: Searching for the meaning of life.

(메시지 본문 - ASCII)
```

- 참고: 이 헤더 라인들은 SMTP 핸드셰이크 명령어(`MAIL FROM`, `RCPT TO`)와는 다름
	- SMTP 명령어: 프로토콜 핸드셰이크의 일부
	- 헤더 라인: 메일 메시지 자체의 일부

## 2.3.3 메일 액세스 프로토콜 (Mail Access Protocols)

- SMTP는 **푸시 프로토콜** → 메일 서버에서 사용자의 로컬 호스트로 메시지를 **가져오는(pull)** 용도로는 사용 불가
- Bob이 메일 서버에서 이메일 검색하는 방법:
	1. **HTTP 방식**: 웹 기반 이메일(Gmail)이나 스마트폰 앱 → HTTP로 메일 검색
		- 이 경우 Bob의 메일 서버는 SMTP 인터페이스 + **HTTP 인터페이스** 모두 필요
	2. **IMAP 방식**: RFC 3501 정의 IMAP(Internet Mail Access Protocol)
		- Microsoft Outlook 등의 메일 클라이언트에서 주로 사용
		- 메일 서버에 폴더 관리, 메시지 이동/삭제/표시 등 가능

### 메일 전송의 전체 흐름

```
[Alice 사용자 에이전트] --SMTP 또는 HTTP--> [Alice 메일 서버] --SMTP--> [Bob 메일 서버] --HTTP 또는 IMAP--> [Bob 사용자 에이전트]
```

- Alice 사용자 에이전트가 Bob 메일 서버에 직접 통신하지 않는 이유: 
	- Alice 에이전트가 직접 Bob 서버에 도달하지 못하면 대처 방법이 없음
	- Alice 메일 서버가 대기열에 저장하고 **재시도** 가능

---

# 2.4 DNS — 인터넷의 디렉터리 서비스 (DNS — The Internet's Directory Service)

- 사람은 기억하기 쉬운 **호스트명(hostname)** 선호 (예: www.google.com)
- 라우터는 처리하기 쉬운 고정 길이 계층적 **IP 주소** 선호 (예: 121.7.106.83)
- **IP 주소**: 4바이트, 점(.)으로 구분된 0~255의 십진수 (예: 121.7.106.83)
	- 계층적: 왼쪽→오른쪽 순서로 호스트가 속한 네트워크에 대한 점점 더 구체적인 정보 제공

## 2.4.1 DNS가 제공하는 서비스 (Services Provided by DNS)

- **DNS(Domain Name System)**: 인터넷의 도메인 네임 시스템
	- (1) DNS 서버들의 계층 구조에 의해 구현된 **분산 데이터베이스**
	- (2) 호스트들이 분산 데이터베이스를 조회할 수 있게 해주는 **애플리케이션 계층 프로토콜**
- DNS 프로토콜: **UDP** 위에서 동작, **포트 53** 사용
- DNS 소프트웨어: 주로 UNIX 머신에서 **BIND(Berkeley Internet Name Domain)** 사용

### DNS의 주요 역할

1. **호스트명 → IP 주소 변환**: 주된 역할
	- 예: 브라우저가 `www.someschool.edu/index.html` 요청 시:
		1. DNS 클라이언트 측 실행
		2. 브라우저가 호스트명 `www.someschool.edu`를 DNS에 전달
		3. DNS 클라이언트가 DNS 서버에 쿼리 전송
		4. DNS 응답 수신 → IP 주소 획득
		5. 브라우저가 해당 IP 주소의 포트 80으로 TCP 연결 시작
	- DNS가 추가 지연 초래 가능 → 가까운 DNS 서버에 **캐시**로 완화

2. **호스트 별칭(Host Aliasing)**: 복잡한 정식 호스트명에 기억하기 쉬운 별칭 제공
	- 예: `relay1.westcoast.enterprise.com`(정식) ← `enterprise.com`, `www.enterprise.com`(별칭)
	- DNS가 별칭에 대한 정식 호스트명과 IP 주소 조회 가능

3. **메일 서버 별칭(Mail Server Aliasing)**: 이메일 주소 단순화
	- 예: bob@yahoo.com → 실제 메일 서버 정식명: `relay1.west-coast.yahoo.com`
	- **MX 레코드**: 회사의 메일 서버와 웹 서버가 같은 별칭을 가질 수 있게 허용
		- 예: `enterprise.com`이 메일 서버와 웹 서버 모두의 별칭

4. **부하 분산(Load Distribution)**: 복제된 서버들 사이에 트래픽 분산
	- 여러 서버가 같은 도메인명을 가질 때 DNS가 IP 주소 집합을 순환(round-robin) 반환
	- 클라이언트는 보통 첫 번째 IP로 요청 → 다른 클라이언트마다 다른 서버로 유도

### DNS는 핵심 인터넷 기능을 애플리케이션 계층에서 구현하는 예

- 1.2절의 "복잡성은 네트워크 에지에" 원칙의 사례: 이름→주소 변환을 네트워크 내부가 아닌 에지의 클라이언트-서버로 구현

## 2.4.2 DNS 동작 원리 개요 (Overview of How DNS Works)

### 중앙 집중형 DNS의 문제점

- 단순한 설계: 단일 DNS 서버에 모든 매핑 → 클라이언트가 단일 서버에만 쿼리
- **문제점**:
	- **단일 실패 지점(Single Point of Failure)**: DNS 서버 다운 → 인터넷 전체 마비
	- **트래픽 볼륨**: 모든 HTTP 요청·이메일에 대한 DNS 쿼리를 단 하나가 처리
	- **먼 중앙 집중형 데이터베이스**: 뉴욕에 단일 서버 → 호주의 쿼리는 지구 반대편을 거침
	- **유지 보수 문제**: 모든 인터넷 호스트 기록 관리 + 빈번한 업데이트 필요
- 결론: 단일 DNS는 **확장성이 없음** → DNS는 처음부터 **분산 구조**로 설계

### 분산·계층적 데이터베이스 (A Distributed, Hierarchical Database)

DNS 서버의 3가지 계층:

1. **루트 DNS 서버(Root DNS Servers)**
	- 전 세계에 **1,000개 이상의 인스턴스** 분산 (2020년 기준)
	- 12개 다른 조직이 관리하는 **13개 루트 서버**의 복제본
	- IANA(Internet Assigned Numbers Authority)가 조정
	- 역할: **TLD 서버의 IP 주소** 제공

2. **최상위 도메인(TLD: Top-Level Domain) DNS 서버**
	- `com`, `org`, `net`, `edu`, `gov` 등 + 국가별(`uk`, `fr`, `jp` 등) 각각에 서버 존재
	- 예: Verisign이 `com` TLD 서버 관리, Educause가 `edu` TLD 서버 관리
	- 역할: **권한 있는 DNS 서버의 IP 주소** 제공

3. **권한 있는(Authoritative) DNS 서버**
	- 인터넷에 공개 접근 가능한 호스트를 보유한 모든 조직이 DNS 레코드 제공
	- 각 조직은 자체 권한 있는 DNS 서버 구축 or 서비스 제공업체에 위탁
	- 대부분의 대학·대기업은 자체 주(primary) 및 보조(secondary) 서버 운영

4. **로컬 DNS 서버(Local DNS Server)** (엄밀히 계층 외부지만 중요)
	- 각 ISP가 보유 (기본 이름 서버라고도 함)
	- 호스트가 ISP에 연결 시 DHCP를 통해 로컬 DNS 서버 IP 주소 제공
	- 호스트와 가까이 위치 (같은 LAN이거나 몇 개 라우터 거리)
	- 호스트의 DNS 쿼리를 받아 **프록시** 역할로 DNS 계층 구조에 전달

### DNS 조회 과정 예시

`cse.nyu.edu`가 `gaia.cs.umass.edu`의 IP 주소를 얻는 과정:

1. `cse.nyu.edu` → 로컬 DNS 서버 `dns.nyu.edu`에 쿼리 (재귀적 질의)
2. `dns.nyu.edu` → 루트 DNS 서버에 전달
3. 루트 서버 → edu TLD 서버의 IP 주소 반환
4. `dns.nyu.edu` → edu TLD 서버에 쿼리
5. edu TLD 서버 → `dns.umass.edu`(UMass 권한 서버)의 IP 주소 반환
6. `dns.nyu.edu` → `dns.umass.edu`에 쿼리
7. `dns.umass.edu` → `gaia.cs.umass.edu`의 IP 주소 반환
8. `dns.nyu.edu` → `cse.nyu.edu`에 IP 주소 전달

→ 총 **8개의 DNS 메시지** (4개 쿼리 + 4개 응답)

### 반복적 질의 vs 재귀적 질의

- **재귀적 질의(Recursive Query)**: 요청 측이 상대방에게 "대신 해결해줘" 요청
	- 예: 호스트 → 로컬 DNS 서버 쿼리 (로컬 서버가 모든 것을 대신 해결)
- **반복적 질의(Iterative Query)**: 서버가 직접 다음 서버를 알려주고 응답을 직접 로컬 DNS로 보냄
	- 예: 로컬 DNS → 루트 → TLD → 권한 서버 순서의 나머지 쿼리들
- **실제 패턴**: 호스트→로컬DNS는 재귀적, 나머지는 반복적

### DNS 캐싱 (DNS Caching)

- DNS는 **지연 성능 개선**과 **메시지 수 감소**를 위해 캐싱을 광범위하게 활용
- **동작 원리**: DNS 서버가 DNS 응답 수신 시 → 매핑을 **로컬 메모리에 캐시**
	- 이후 동일 호스트명 쿼리 → 권한이 없어도 캐시된 IP 주소 즉시 제공
- **TTL(Time To Live)**: 일반적으로 **2일**로 설정 → 이후 캐시 정보 폐기
	- 호스트-IP 주소 매핑이 영구적이지 않기 때문
- **효과**: TLD 서버의 IP 주소도 캐시 가능 → 대부분의 DNS 쿼리에서 **루트 서버 우회** 가능
- 예: `apricot.nyu.edu`가 `cnn.com` IP 질의 → `dns.nyu.edu`가 캐시 저장 → 몇 시간 후 `kiwi.nyu.edu`가 같은 질의 → 루트/TLD 거치지 않고 즉시 응답

## 2.4.3 DNS 레코드와 메시지 (DNS Records and Messages)

### DNS 자원 레코드 (Resource Records, RRs)

- DNS 분산 데이터베이스를 구현하는 서버들이 저장하는 데이터
- 형식: **(Name, Value, Type, TTL)** (4-튜플)
	- TTL: 레코드의 수명(Time To Live), 캐시에서 제거할 시기 결정

### 4가지 주요 레코드 타입

- **Type=A** (Address): 표준 호스트명-IP 주소 매핑
	- Name: 호스트명 | Value: IP 주소
	- 예: `(relay1.bar.foo.com, 145.37.93.126, A)`

- **Type=NS** (Name Server): 도메인에 대한 권한 있는 DNS 서버 정보
	- Name: 도메인 | Value: 권한 있는 DNS 서버 호스트명
	- DNS 쿼리를 다음 단계로 라우팅하는 데 사용
	- 예: `(foo.com, dns.foo.com, NS)`

- **Type=CNAME** (Canonical Name): 별칭 호스트명에 대한 정식 이름 제공
	- Name: 별칭 호스트명 | Value: 정식 호스트명
	- 예: `(foo.com, relay1.bar.foo.com, CNAME)`

- **Type=MX** (Mail Exchange): 메일 서버 별칭에 대한 정식 이름
	- Name: 별칭 | Value: 메일 서버 정식 이름
	- 예: `(foo.com, mail.bar.foo.com, MX)`
	- 장점: 회사의 메일 서버와 웹 서버에 동일한 별칭 사용 가능

### DNS 메시지 형식

- DNS는 **쿼리 메시지**와 **응답 메시지** 두 가지만 존재 (같은 형식)
- **헤더 섹션(Header Section)** (12바이트):
	- 쿼리 식별 16비트 번호 (응답 메시지에 복사되어 매칭)
	- 플래그들: 쿼리/응답 구분(1비트), 권한 있는 서버 여부(1비트), 재귀 요청(1비트), 재귀 지원(1비트)
	- 이후 4가지 데이터 섹션의 개수 필드
- **질문 섹션(Question Section)**: 질의 이름 + 질문 타입(A, MX 등)
- **응답 섹션(Answer Section)**: 원래 질의에 대한 자원 레코드들 (Type, Value, TTL 포함)
	- 하나의 이름에 여러 IP 주소 가능 → 여러 RR 반환 가능
- **권한 섹션(Authority Section)**: 다른 권한 있는 서버들의 레코드
- **추가 섹션(Additional Section)**: 유용한 추가 레코드들
	- 예: MX 응답 시 응답 섹션에 정식 호스트명, 추가 섹션에 해당 호스트의 IP 주소(Type A)

### DNS 데이터베이스에 레코드 삽입

- 새 도메인 등록 예시: `networkutopia.com` 스타트업 설립
- **등록기관(Registrar)**에 도메인 이름 등록
	- 도메인 이름의 고유성 확인, DNS 데이터베이스에 입력, 수수료 부과
	- ICANN(Internet Corporation for Assigned Names and Numbers)이 다양한 등록기관 인증
- 등록기관에 권한 있는 DNS 서버의 이름과 IP 주소 제공 → com TLD 서버에 NS 및 A 레코드 삽입
	- 예: `(networkutopia.com, dns1.networkutopia.com, NS)` + `(dns1.networkutopia.com, 212.212.212.1, A)`
- 자신의 권한 있는 서버에는 Type A 레코드(웹 서버)와 Type MX 레코드(메일 서버) 추가

### DNS 보안 취약점 (Focus on Security: DNS Vulnerabilities)

- **DDoS 대역폭 플러딩 공격**: DNS 루트 서버에 엄청난 양의 패킷 전송 → 정상 쿼리 응답 방해
	- 2002년 10월 21일 실제 공격: 봇넷으로 13개 루트 IP에 ICMP 핑 메시지 수십 개 전송
	- 피해 최소화: 루트 서버들이 패킷 필터로 보호, 대부분의 로컬 DNS가 TLD 서버 주소 캐시
	- 더 효과적인 공격: `.com` TLD 서버에 DNS 쿼리 폭주 (2016년 10월 Dyn 공격 사례 — Amazon, Twitter, Netflix 등 서비스 장애)
- **중간자 공격(Man-in-the-Middle)**: 공격자가 쿼리 가로채고 허위 응답 전송
- **DNS 포이즈닝(DNS Poisoning)**: DNS 서버에 허위 응답 보내 잘못된 레코드 캐시 유도
	- 사용자를 공격자의 웹사이트로 리디렉션 가능
- **대응**: **DNSSEC(DNS Security Extensions)** (RFC 4033) — DNS 보안 버전으로 이러한 공격 대응

---

# HTTP-Email-DNS 흐름 비교표 (Week 5 Deliverable)

| 항목 | HTTP | SMTP(이메일) | DNS |
|---|---|---|---|
| 역할 | 웹 페이지 전송 | 이메일 전송 | 호스트명→IP 주소 변환 |
| 전송 방향 | 풀(pull): 서버에서 가져옴 | 푸시(push): 서버로 보냄 | 요청-응답(양방향) |
| 기반 전송 | TCP (포트 80) | TCP (포트 25) | UDP (포트 53) |
| 연결 방식 | 영구적 연결 (기본), 비영구적 가능 | 지속 연결 (여러 메시지 전송 시) | 비연결 (UDP 데이터그램) |
| 상태 유지 | 무상태(stateless) — 쿠키로 보완 | 상태 있음 (핸드셰이크, 대기열) | 캐싱으로 성능 향상 |
| 인코딩 제약 | 없음 (이진 데이터 직접 가능) | 7비트 ASCII 제한 | 해당 없음 |
| 서버 구조 | 중앙집중형 (데이터센터) | 메일 서버 분산 (각 도메인별) | 계층적 분산 (루트→TLD→권한) |
| 데이터 형식 | 요청/응답 메시지 (ASCII 헤더 + 바디) | 헤더 + 본문 (RFC 5322) | 자원 레코드 (Name, Value, Type, TTL) |
| 주요 버전/RFC | HTTP/1.0, 1.1, 2, 3 | RFC 5321 (SMTP), 3501 (IMAP) | RFC 1034, 1035 |
| 캐싱 | 웹 캐시/프록시 서버 | 해당 없음 | DNS 캐싱 (TTL 기반) |
