# 1. Web과 HTTP

## 1-1. Web

- 웹페이지는 하나의 파일만으로 이뤄지는 경우보다, **HTML 파일 + 여러 참조 객체들**로 이뤄지는 경우가 많음
    - 참조 객체엔 이미지(JPEG), 오디오, CSS, 비디오 클립 같은 것들이 들어갈 수 있음
- 각 객체는 **URL**로 식별 가능하며, URL은 보통 **호스트 이름 + 경로 이름**으로 구성됨
    
    ex ) `http://www.someSchool.edu/someDepartment/picture.gif` 에서 `www.someSchool.edu` 는 호스트 이름이고 `/someDepartment/picture.gif` 는 경로 이름
    
- 웹 브라우저는 이 객체들을 서버에 요청해서 받아와 화면에 조합해 보여줌

## 1-2. HTTP

: **HyperText Transfer Protocol** 의 약자이며, 웹에서 사용하는 애플리케이션 계층 프로토콜

- 동작 구조 : 전형적인 **클라이언트[브라우저] - 서버 모델[웹 서버]**
- 브라우저가 요청(request)을 보내면 서버가 응답(response)으로 객체를 돌려줌
- 이 때 브라우저는 받은 객체를 해석하고 사용자에게 보여줌
    - 웹 서버 예시 - Apache, Microsoft IIS 등

---

# 2. HTTP와 TCP 관계와 Stateless 특성

## 2-1. HTTP와 TCP 관계

- HTTP는 UDP 위가 아닌 **TCP 위에서 동작**. 즉, 브라우저가 서버에 HTTP 메시지를 보내기 전에 먼저 TCP 연결부터 맺음
- 기본적으로 HTTP 서버는 **포트 80** 사용
    - 클라이언트가 TCP 연결 → 서버가 이를 accept → 양쪽이 HTTP 요청/응답 메시지를 주고받음 → 통신이 끝나면 TCP 연결이 닫힘
        
        ⇒ TCP가 **신뢰성 있는 데이터 전송을 제공**하므로, HTTP 자체는 **손실 복구나 순서 보장을 직접 처리하지 않아도됨 (계층화의 장점)**
        

## 2-2. Stateless

- HTTP의 매우 중요한 특징은 **stateless**라는 것임. 즉, 서버는 과거에 특정 클라이언트가 **어떤 요청을 했는지 기본적으로 기억하지 않음**
- **같은 클라이언트가 몇 초 전에 같은 객체를 요청**했더라도, 서버는 방금 보냈다고 하지 않고 **다시 보내줌**

⇒ 이 무상태성 덕분에 서버 구현이 단순해지고, 수많은 동시 요청을 처리하는 데 유리함

⇒ 대신 사용자를 식별하거나 **세션 상태를 유지해야 하는 경우에는 별도 수단**이 필요하며, 그 대표가 **쿠키**임

---

# 3. Non-persistent HTTP와 Persistent HTTP

- HTTP 연결 방식은 크게 **비지속 연결(non-persistent connection)**과 **지속 연결(persistent connection)**로 나뉨
- **비지속 연결**에선 **TCP 연결 하나당 최대 객체 하나**만 전송하고 바로 연결을 닫음
    
    ∴ 웹 페이지처럼 HTML 하나와 여러 이미지가 함께 있는 경우, 객체 수만큼 TCP 연결이 여러 번 필요해짐
    
- **지속 연결**에선 TCP 연결을 한 번 열어 둔 뒤, **같은 서버와의 여러 HTTP 메시지**를 그 연결 하나로 계속 주고받을 수 있음 ⇒ 이 방식이 HTTP/1.1의 기본 방향

## 3-1. Non-persistent HTTP 동작 순서

1. 사용자가 URL 입력
2. 클라이언트가 서버 80번 포트로 TCP 연결 시작
3. 서버가 연결 수락
4. 클라이언트가 HTTP 요청 메시지 보냄
5. 서버가 요청 객체를 포함한 HTTP 응답 메시지 보냄
6. 서버가 TCP 연결 닫음
7. 클라이언트가 HTML을 해석해 참고 객체를 찾고,각 객체마다 위 과정 반복

ex ) HTML 안에 JPEG 10개가 있으면, 기본 HTML 1개 + 이미지 10개까지 총 11개의 객체를 위해 여러 번 연결이 반복될 수 있음

- 브라우저는 이 과정을 순차적으로 하기도 하고, 병렬 TCP 연결을 열어서 동시에 가져오기도 함
- HTTP는 브라우저가 페이지를 어떻게 렌더링하는지는 정의하지 않고, 오직 **메시지 교환 방식**만 정의함

## 3-2. Non-persistent HTTP 응답시간

> 비지속 연결에서 객체 하나를 받는 데 걸리는 시간은 대략 **2 RTT + 파일 전송 시간**임
> 
- 첫 번째 RTT : TCP 연결 설정
- 두 번째 RTT : HTTP 요청 보내고 응답 첫 바이트 받기
- 그 뒤 : 실제 파일 전송 시간

⇒ 즉, 객체가 많아질수록 RTT가 계속 중복돼 응답시간이 길어짐

## 3-3. Persistent HTTP

- 비지속 연결의 단점
1. 객체마다 TCP 연결을 새로 열고 닫아야 해서 OS 오버헤드 큼
2. 객체마다 2 RTT가 필요해서 지연이 커짐

∴ HTTP/1.1의 지속 연결에선 **서버가 응답 후에도 연결을 유지함**

- 같은 클라이언트와  서버 사이에서 후속 요청을 계속 같은 연결로 보내므로, 여러 참조 객체를 훨씬 효율적으로 가져올 수 있음
- 클라이언트는 참조 객체를 발견하자마자 요청을 연달아 보낼 수 있고, **pipelining**을 통해 이전 응답을 다 기다리지 않고 다음 요청을 보내는 것도 가능

⇒ 결과적으로 **전체 응답시간을 많이 줄일 수 있음**

---

# 4. HTTP 메시지 형식

> HTTP 메시지는 크게 **요청 메시지(request message)**와 **응답 메시지(response message)**로 나뉨. 기본적으로 사람이 읽을 수 있는 **ASCII 텍스트** 형식임
> 

## 4-1. HTTP Request Message

요청 메시지는 일반적으로 다음 구조를 가짐

- **request line**
    - method
    - URL
    - version
- 여러 개의 **header lines**
- 빈 줄(blank line)
- 필요하면 **entity body**

ex ) `GET /somedir/page.html HTTP/1.1`

### 헤더 예시

- **Host** : 대상 호스트 지정
- **Connection** : close 또는 keep-alive
- **User-Agent** : 브라우저 종류
- **Accept-Language** : 선호 언어
- **Accept-Encoding** : gzip, deflate 등 압축 선호

**GET** 방식에서는 **보통 body가 비어있고**, **POST** 방식에서는 body에 **사용자 입력 데이터**가 들어감

## 4-2. HTTP 메서드들

- **GET** : 객체 요청 시 사용
- **POST**: 폼 입력 데이터 등을 body에 담아 서버로 전송함
- **GET으로 데이터 전송**: URL 뒤에 `?` 붙여 파라미터 붙일 수도 있음
- **HEAD**: GET과 비슷하지만 body 없이 헤더만 받음
- **PUT**: 서버 특정 경로에 파일 업로드하거나 기존 파일 완전히 대체함
- **DELETE**: 서버 객체 삭제할 수 있게 함

⇒ 즉, 메서드는 **무엇을 할것인가**를 나타내는 핵심 필드임

## 4-3. HTTP Response Message

응답 메시지는 일반적으로 다음 구조임

- **status line**
    - protocol version
    - status code
    - status phrase
- 여러 개의 **header lines**
- 빈 줄
- **entity body → 실제 요청한 데이터**가 들어감

ex ) `HTTP/1.1 200 OK`

### 헤더 예시

- **Connection** : close
- **Date** : 응답 생성 시각
- **Server** : 서버 소프트웨어 정보
- **Last-Modified** : 객체 마지막 수정 시각
- **Content-Length** : 전송되는 객체 크기
- **Content-Type** : 객체 타임(ex. text/html)

## 4-4. 주요 상태 코드

- **`200 OK`** : 요청 성공, 객체 포함됨
- **`301 Moved Permanently`** : 객체가 영구적으로 다른 위치로 이동했음. 새 URL은 Location 헤더에 들어감
- **`400 Bad Request`**  : 서버가 요청 이해 못함
- **`404 Not Found`**: 요청한 문서가 서버에 없음
- **`505 HTTP Version Not Supported`** : 요청 HTTP 버전을 서버가 지원하지 않음

상태 코드는 **응답 메시지 첫 줄**에서 확인 가능

---

# 5. Cookies

> 
> 
> - HTTP는 stateless라 **사용자 상태를 기본적으로 기억하지 못함**
> - 하지만 **실제 웹 서비스**는 로그인 유지, 장바구니, 사용자 추천 같은 기능을 위해 **상태 저장이 필요**
> 
> ⇒ 이 문제를 해결하는 대표 수단이 **쿠키(cookie)**임
> 

## 5-1. 쿠키의 4가지 구성요소

1. HTTP 응답 메시지의 cookie 헤더 줄
2. 다음 HTTP 요청 메시지의 cookie 헤더 줄
3. 사용자 호스트에 저장된 cookie file
4. 웹 사이트 백엔드 데이터베이스

⇒ 즉, **브라우저 쪽 저장소와 서버 쪽 DB가 함께 동작**하면서 사용자를 식별함

## 5-2. 쿠키 동작 예시

1. 사용자가 처음 전자상거래 사이트에 접속하면, 서버가16 사용자에게 **고유 ID**를 생성함
    
    ex. 서버가 `Set-cookie: 1678` 같은 헤더를 응답에 넣어 보내면, 브라우저는 이를 cookie file에 저장
    
2. 같은 사이트에 다시 요청 보낼 때, 브라우저는 `Cookie: 1678` 헤더를 함께 보냄
    
    → 서버는 이를 보고 **이 사용자가 예전에 왔던 그 사용자**라고 식별 가능
    
    - 이 ID는 사용자 행동과 연결된 식별값이면 충분함(이름 그 자체일 필요 X)

## 5-3. 쿠키의 활용

쿠키는 다음과 같은 곳에 사용됨

- authorization
- shopping cart
- recommendations
- web e-mail 같은 user session state

ex)

 Amazon 같은 사이트 → 쿠키 식별자를 바탕으로 사용자가 봤던 페이지, 장바구니 항목, 추천 상품 등을 관리할 수 있음

웹메일에서도 로그인 세션 유지에 자주 쓰임 

⇒ 즉, 쿠키는 무상태 HTTP 위에 **세션 계층**을 올려주는 역할을 함

## 5-4. 쿠키와 프라이버시

- 쿠키는 편리하지만, 프라이버시 문제도 큼
    - 사이트는 쿠키를 통해 사용자가 자기 사이트 안에서 무엇을 했는지 알 수 있음
    - **third-party persistent cookie** → 사용자가 직접 방문한 사이트가 아닌, 광고 네트워크 같은 **외부 사이트가 심어둔 쿠키**를 통해 여러 사이트를 가로질러 사용자 추적 가능

## 5-5. First-party / Third-party cookie

- **first-party cookie** : 사용자가 **직접 방문한 사이트가 심는 쿠키**
- **third-party cookie** : 사용자가 직접 방문하지 않은 **외부 사이트가 심는 쿠키**
    - 사용자가 tracker 사이트에 직접 들어가지 않아도, **보이지 않는 광고/링크/리소스를 통해 추적 가능**하다는 점이 핵심
    - 브라우저들은 이런 추적을 줄이기 위해 third-party tracking을 제한하는 방향으로 발전해 왔음

## 5-6. GDPR과 쿠키

- EU GDPR은 쿠키 식별자도 개인을 식별할 수 있으면 **개인정보(personal data)**로 봄. 즉, 단순한 브라우저 식별값처럼 보여도, **다른 정보와 결합해 개인 프로파일**을 만들 수 있다면 규제 대상이 됨
    
    ∴ 사용자는 **쿠키 허용 여부를 명시적으로 통제**할 수 있어야 하고, 실제로 많은 사이트에서 쿠키 등의 팝업을 띄우는 이유가 여기에 있음
    

# 6. Web Caching

- 웹 캐시(Web cache) 또는 프록시 서버 → **오리진 서버를 대신해서 객체를 저장하고 전달하는 중간 서버**
- 브라우저는 **요청을 먼저 캐시로** 보냄
    - 캐시에 객체가 있으면 캐시가 바로 응답함
    - 없으면 캐시가 오리진 서버에 요청해서 객체를 받아 저장한 뒤 클라이언트에게 돌려줌
    
    ⇒ 즉, 웹 캐시는 **클라이언트이면서 동시에 서버** 역할을 함 (브라우저에겐 서버, 오리진 서버에겐 클라이언트임)
    

## 6-1. 웹 캐시를 쓰는 이유

웹 캐시를 쓰는 이유는 크게 세 가지임

1. 클라이언트 응답시간 감소
2. 기관의 access link 트래픽 감소
3. 전체 인터넷 트래픽 완화 및 저비용 콘텐츠 전달 지원
- 클라이언트와 캐시가 가까우면 **응답이 빨라지고, 오리진 서버까지 갈 필요가 줄어듦
- 기관 네트워크 입장에선 **인터넷 접속 링크를 지나는 트래픽을 줄여** 비용과 병목을 줄일 수 있음
- 실제 인터넷은 수많은 캐시로 가득 차 있고, 이 아이디어는 CDN으로까지 발전함

## 6-2. 캐시 없는 경우의 병목 해결 방안

1. 빠른 access link 구매

- 접속 링크를 더 빠르게 업그레이드하면 병목이 줄어듦
    
    ex. 15Mbps → 100Mpbs 또는 1.54Mbps → 154Mbps로 올리면 **이용률이 크게 낮아지고 지연도 크게 줄어듦**
    
- 하지만 이 방법은 **비용이 많이 듦**. 즉, 효과는 좋지만 비쌈

2. 웹 캐시 설치

- 더 빠른 access link를 사는 것보다 **더 싸고, 경우에 따라 지연도 더 낮아질 수 있음**

## 6-3. CDN과의 연결

- 웹 캐시 아이디어는 더 확장돼 **CDN(Content Distribution Network)**으로 발전함
    - **전 세계에 캐시를 분산 배치**해 사용자가 가까운 위치에서 콘텐츠를 받도록 해, 트래픽과 지연을 더 줄이는 방식

---

# 7. Conditional GET

- 캐싱은 빠르지만 새로운 문제를 만듦 → 캐시에 저장된 복사본이 **오래되어 최신 버전이 아닐 수 있음**
    
    ⇒ 이 문제를 해결하기 위한 HTTP 메커니즘이 **Conditional GET**임
    
- 브라우저나 캐시 → 이전에 받은 객체의 **Last-Modified** 값을 기억해 둠
- 그 후 다시 같은 객체를 요청할 때 `If-modified-since: <date>` 헤더를 함께 보냄.
    - 서버는 해당 날짜 이후 객체가 수정되지 않았으면 **`304 Not Modified`** 응답만 보내고, 객체 본문은 보내지 않음
    - 수정되었다면  **`200 OK + 새 데이터`** 를 보냄

## Conditional GET의 장점

- 객체가 안 바뀌었으면 불필요한 재전송 안 함
- 대역폭 절약 가능
- 사용자 체감 응답시간 줄어듦
- 네트워크 자원 낭비 줄어듦

⇒ 즉, Conditional GET은 **캐시 써도 최신성 확인 가능하게 해주는 장치**라고 보면 됨

# 8. HTTP/2

- HTTP/2의 핵심 목표 → **여러 객체를 동시에 가져올 때 지연을 줄이는 것**
- HTTP/1.1도 persistent connection과 pipelining을 도입했지만, 여전히 한 TCP 연결 위에서 **객체 응답이 요청 순서대로 처리되는 문제**가 있었음 → 그 결과 작은 객체가 큰 객체 뒤에 줄 서서 기다리는 **HOL(Head of Line)** 발생

## 8-1. HTTP/1.1의 HOL blocking

- 작은 객체들이 빨리 렌더링될 수 있는데도, 큰 객체 하나 때문에 지연되는 문제 발생
    
    ex. 큰 비디오 객체 O1 하나와 작은 객체 O2, O3, O4를 동시에 요청했다 가정했을 때, O1이 먼저 전송되기 시작하면, 나머지들은 그 뒤에서 기다려야 할 수 있음
    

⇒ 그채서 브라우저는 이런 문제를 피하려고 여러 병렬 TCP 연결을 여는 경향이 있었음 (실제로 많은 HTTP/1.1 브라우저는 최대 6개 정도의 병렬 연결을 열기도 함)

## 8-2. HTTP/2의 개선점

- 기본 HTTP 의미론 자체는 크게 변경 X (메서드, 상태 코드, URL, 대부분의 헤더 필드는 그대로 유지)
- 대신 전송 방식을 바꿈
    - 클라이언트가 지정한 우선순위에 따라 객체 전송 가능
    - 요청하지 않은 객체를 서버가 미리 보내는 **server push**
    - 객체를 작은 **frame** 단위로 쪼갬
    - 여러 객체의 frame을 **interleaving** 해서 번갈아 전송 가능
    - 헤더 압축 가능
    
    ⇒ 이 덕분에 O2, O3, O4 같은 작은 객체를 먼저 빨리 보내고, O1 같은 큰 객체는 조금 늦게 끝나도록 조정 가능함 (**HOL blocking 완화**)
    
    ## 8-3. HTTP/2의 한계
    
    - HTTP/2도 보통 **단일 TCP 연결** 위에서 동작하므로, 패킷 손실이 생기면 TCP 재전송/복구 때문에 전체 객체 전송이 함께 멈출 수 있음
        
        즉, HTTP 계층에선 프레임 다중화가 되더라도, 그 아래 TCP 손실 복구 특성 때문에 여전히 지연 발생 가능
        
        ∴ 브라우저가 병렬 TCP 연결을 열고 싶어 하는 유인이 완전히 사라지지 않음
        

# 9. HTTP/3와 QUIC

- QUIC : **UDP 위에서 동작하는 애플리케이션 계층 프로토콜**로, HTTP 성능 향상을 목적으로 설계됨
    - 기존 구조가 `IP - TCP - TLS - HTTP/2` 였다면, QUIC 기반 구조는 `IP - UDP - QUIC - slimmed HTTP/2` 형태로 표현 가능 (HTTP/3는 QUIC 위에서 돌아가는 HTTP라고 보면 됨)

### QUIC의 장점

- 연결 설정, 오류 제어, 혼잡 제어를 TCP와 유사하게 수행함
- 인증, 암호화, 신뢰성, 혼잡 제어 상태를 **1 RTT** 안에 설정 가능함
- TCP handshake + TLS handshake처럼 **두 번의 직렬 핸드셰이크**가 아니라, 더 짧은 절차로 가능함
- 두 번째 연결부터는 **0-RTT** 연결 설정도 가능함
    - 서버가 session ticket을 주고
    - 클라이언트가 다음 연결 때 재사용함

⇒ HTTP/3/QUIC은 **HTTP/2의 성능 한계를 줄이기 위한 방향**

# 10. E-mail

전자메일 시스템은 세 가지 핵심 구성요소로 이루어짐

- **User Agent** (메일 리더라고도 하며, 메일 작성/수정/읽기 담당함)
- **Mail Server** (사용자의 수신 메일과 송신 메일은 실제론 메일 서버 쪽에 저장됨)
- **SMTP**

## 10-1. Mail Server

메일 서버엔 두 가지 주요 저장 영역이 있음

- **mailbox**: 사용자 수신 메일 저장
- **message queue**: 발송 대기 중인 송신 메일 저장

메일 서버끼리는 SMTP를 사용해 메일을 주고받음

이때 보내는 쪽 메일 서버가 클라이언트 역할, 받는 쪽 메일 서버가 서버 역할을 함

## 10-2. SMTP

- **Simple Mail Transfer Protocol**의 약자로, TCP를 사용하고, **포트 25**를 통해 신뢰성 있게 이메일 메시지를 전송함
- 동작은 세 단계로 요약 가능
    - handshaking
    - message transfer
    - closure
- **명령/응답 형태**로 동작하며, HTTP처럼 **ASCII 텍스트 기반**임. 명령은 **텍스트**이고, 응답은 **상태 코드 + 문구**임
- SMTP는 **push** 방식이고, HTTP는 **pull** 방식

## 10-3. 메일 메시지 형식과 메일 접근 프로토콜

SMTP는 **이메일을 전달하는 프로토콜**이고, 이메일 메시지 자체 문법은 별도로 정의됨

메일 메시지는 To, From, Subject 같은 **header**와 body로 구성됨. 이 헤더들은 SMTP의 `MAIL FROM`, `RCPT TO` 와는 다른 계층의 정보임

수신 측 사용자가 메일을 읽으려면 **mail access protocol** 이 필요

- root DNS domain 관리에는 ICANN이 관여

## 11-5. TLD server와 Authoritative server

- **TLD 서버**: `.com`, `.org`, `.edu`, 국가도메인 `.kr`, `.uk`, `.jp` 등 최상위 도메인 담당
- **Authoritative DNS server**: 특정 조직의 이름과 IP 매핑에 대한 공식 정보를 제공하는 서버

⇒ 즉 authoritative 서버가 **최종 답을 알고 있는 원천 정보원** (조직이 직접 운영할 수도 있고, 서비스 제공업체가 대신 운영할 수도 있음)

## 11-6. Local DNS server

호스트가 DNS 질의를 하면 먼저 **local DNS server** 로 보냄

- local DNS는
    - 자기 캐시에 있으면 바로 응답하고
    - 없으면 DNS 계층으로 질의를 전달함
- 각 ISP는 보통 local DNS server를 갖고 있음

strict hierarchy 안에 딱 들어가는 존재는 아니지만, 실제 질의 처리에서 매우 중요한 첫 관문임

## 11-7. Iterative query vs Recursive query

- **Iterative query** 에서는, 질의받은 서버가 **나는 모르지만 저 서버에 물어봐라** 식으로 다음 서버 정보 알려줌. 즉 local DNS가 root, TLD, authoritative를 차례로 따라감
- 부담이 분산되고, **각 서버는 자기 수준에서만 답함**
- **Recursive query** 에서는, 질의받은 서버가 다음 서버들까지 대신 물어봐서 최종 답을 가져와 줌.
    
    ⇒ 즉 부담이 상위 이름 서버 쪽으로 몰릴 수 있음.
    

(PPT에선 upper levels of hierarchy에 heavy load가 생길 수 있다는 점 지적)

## 11-8. DNS caching

- 이름 서버는 어떤 매핑을 한 번 배우면 **cache** 에 저장 가능
- 캐시된 값은 즉시 응답에 사용되어 성능을 높임
- 다만 캐시 항목은 일정 시간이 지나면 사라지는데, 이 시간이 **TTL** 임.
- 문제는 **호스트 IP가 바뀌어도 TTL이 다 만료되기 전까지는 오래된 정보가 남을 수 있다**는 점임. 즉 **DNS 이름 변환**은 엄밀히 말하면 **best-effort** 임

## 11-9. DNS Resource Record

- DNS는 **RR(Resource Record)** 형태로 정보 저장 ( 형식은 `(name, value, type, ttl)` )
- 주요 타입:
    - **A**: hostname → IP address
    - **NS**: 어떤 도메인의 authoritative name server
    - **CNAME**: alias → canonical name
    - **MX**: 메일 서버 이름

⇒ 즉 DNS는 단순 주소록이 아니라 **다양한 타입의 정보를 담는 레코드 DB**

## 11-10. DNS protocol message

DNS query와 reply는 기본적으로 같은 형식을 가짐.

헤더에는 다음 정보가 포함됨.

- identification
- flags  → query/reply 여부, recursion desired/available, authoritative reply 여부 등이 들어감
- number of questions
- number of answers
- number of authority RRs
- number of additional RRs

그 뒤에 질문, 답변, 권한 정보, 추가 정보가 variable length로 붙음

## 11-11. 새 도메인 DNS 등록

- 새로운 스타트업이 도메인을 등록할 때는 **registrar에 도메인 이름을 등록**하고, **authoritative name server의 이름과 IP 주소를 제공**해야 함.
    
    → 그러면 registrar가 TLD 서버에 NS, A 레코드를 추가함
    
- 그 뒤 조직은 자기 authoritative 서버 안에 `www` 용 A 레코드, 메일용 MX 레코드 등을 직접 구성함. 즉 DNS는 등록기관과 authoritative 서버 구성이 함께 맞물려 동작

## 11-12. DNS 보안

- **DDoS attacks**
    - root server 폭격
    - TLD server 폭격
- **Spoofing attacks**
    - 질의 가로채서 가짜 응답 보내기
- **DNS cache poisoning**
    - 캐시에 잘못된 정보 주입하기

⇒ 이런 문제에 대응하기 위해 **DNSSEC** 이 인증 서비스를 제공

대표적으로:

- **IMAP**: 서버에 저장된 메시지를 조회, 삭제, 폴더 관리 가능
- **HTTP 기반 웹메일**: Gmail, Hotmail, Yahoo Mail 등
- POP도 존재

즉 SMTP는 **보내는 역할**, IMAP/HTTP는 **읽어오는 역할**

---

# 11. DNS

- 인터넷 호스트와 라우터는 두 종류의 식별자를 가짐
    - 사람이 읽는 이름(ex. `cs.umass.edu`)
    - 네트워크가 쓰는 IP 주소
    
    ⇒ DNS는 이 둘을 연결하는 **Domain Name System**
    
- 즉, 이름을 IP 주소로 바꾸고, 경우에 따라 반대로도 바꿔줌
- DNS는 단순한 표 하나가 아닌, **계층적으로 분산된 거대한 데이터베이스**이며, 동시에 호스트와 DNS 서버가 통신하는 **애플리케이션 계층 프로토콜**이기도 함
- 인터넷의 핵심 기능이지만 네트워크 고어가 아닌 **엣지 쪽 복잡성**으로 구현됨

## 11-1. DNS를 중앙집중식으로 두지 않는 이유

- single point of failure
- 엄청난 traffic volume
- 중앙 DB가 멀리 있으면 지연 커짐
- 유지보수 어려움

→ 실제로 질의 수가 엄청나서, 특정 ISP나 CDN 업체 DNS만 봐도 하루 수천억~수조 건 수준의 질의 발생 ∴ DNS는 반드시 **분산 구조**여야 하고, 그래야 **확장성과 성능을 확보**할 수 있음

## 11-2. DNS가 제공하는 서비스

DNS는 단순 이름→주소 변환만 하는 게 아님

- hostname to IP address translation
- host aliasing
- mail server aliasing
- load distribution

특히 **하나의 이름에 여러 IP를 연결해 replicated Web server들로 부하 분산**하는 데도 사용

## 11-3. DNS의 계층 구조

DNS는 **계층형 분산 DB**

- **Root DNS servers**
- **TLD DNS servers**
- **Authoritative DNS servers**
- 그리고 사용자 가까이에 있는 **Local DNS server**

### 예시 )

 `www.amazon.com` IP를 알고 싶으면 대략

1. root에 `.com` 담당 물어봄
2. `.com` TLD 서버에 `amazon.com` 담당 물어봄
3. `amazon.com` authoritative 서버에 `www.amazon.com` 물어봄

이런 식으로 내려감

## 11-4. Root name sever

: 어떤 이름 서버도 해결 못할 때 찾아가는 **최후의 공식 연락처**같은 존재

- 인터넷 전체가 DNS에 의존하므로 매우 중요함
- 전 세계적으로 **13개의 논리적 root server**가 있고, 각 서버는 **여러 위치에 복제**되어 있음
- 또 DNSSEC을 통해 **인증과 무결성 보안 제공 가능**