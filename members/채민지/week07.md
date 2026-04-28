# chapter 1 내용 정리

## 1. Internet의 두 가지 관점

### 1-1. Nuts and bolts view

Internet은 물리적인 장치와 통신 규칙들의 집합으로 볼 수 있다.

Internet을 구성하는 요소는 다음과 같다.

- Host 또는 end system
    - PC, smartphone, server 등
    - application이 실행되는 장치
- Packet switch
    - router, switch 등
    - packet을 목적지 방향으로 전달
- Communication link
    - fiber, copper, radio, satellite 등
    - 데이터가 이동하는 물리적/논리적 경로
- Protocol
    - TCP, IP, HTTP, WiFi, Ethernet 등
    - message를 어떻게 보내고 받을지 정하는 규칙

Internet은 하나의 단일 네트워크가 아니라 여러 network가 서로 연결된 구조이므로 **network of networks**라고 부른다.

### 1-2. Service view

Internet은 application에게 통신 서비스를 제공하는 infrastructure로도 볼 수 있다.

예를 들어 다음 application들이 Internet 위에서 동작한다.

- Web
- streaming video
- email
- online game
- social media
- e-commerce
- video conferencing

Application은 Internet이 제공하는 programming interface를 통해 데이터를 보내고 받는다.

즉, Internet은 distributed application이 서로 통신할 수 있게 해주는 service platform이다.

---

## 2. Protocol

Protocol은 통신 규칙이다.

Protocol은 다음을 정의한다.

- 어떤 message를 보낼지
- message를 어떤 순서로 보낼지
- message를 받으면 어떤 action을 수행할지

사람 사이의 대화에도 규칙이 있듯이, computer network에서도 통신은 protocol에 의해 이루어진다.

예시:

- HTTP
- TCP
- IP
- WiFi
- Ethernet

시험 답안식으로 쓰면 다음과 같다.

> Protocol은 통신 주체들이 주고받는 message의 format, order, 그리고 message를 받았을 때 수행해야 하는 action을 정의한 규칙이다.
> 

---

## 3. Network edge / Access network / Network core

### 3-1. Network edge

Network edge는 사용자와 가까운 부분이다.

여기에는 host, client, server가 존재한다.

예시:

- laptop
- smartphone
- web server
- email server

Server는 보통 data center에 위치한다.

### 3-2. Access network

Access network는 end system을 첫 번째 router, 즉 edge router에 연결해주는 네트워크이다.

종류:

- home access network
- institutional access network
- mobile access network
- WiFi
- Ethernet
- DSL
- cable network
- 4G/5G

### 3-3. Network core

Network core는 interconnected routers의 집합이다.

Packet을 source에서 destination까지 전달하는 중심부이다.

Network core의 핵심 동작은 packet switching이다.

---

## 4. Access networks

### 4-1. Cable-based access

Cable network는 cable modem과 cable headend를 통해 Internet에 연결된다.

특징:

- HFC, Hybrid Fiber Coax 구조
- downstream과 upstream 속도가 비대칭적일 수 있음
- 여러 가정이 cable headend까지의 access network를 공유함
- FDM을 사용하여 여러 channel을 다른 frequency band에 실어 보낼 수 있음

### 4-2. DSL

DSL은 기존 전화선을 사용하여 Internet에 연결하는 방식이다.

구성:

- DSL modem
- splitter
- DSLAM
- central office

특징:

- data는 Internet으로 전달
- voice는 telephone network로 전달
- 각 가정이 central office까지 dedicated line을 가짐

### 4-3. Home network

Home network는 modem, router, firewall, NAT, WiFi, Ethernet 등으로 구성된다.

일반적인 구조:

- cable/DSL modem
- router/firewall/NAT
- wired Ethernet
- wireless WiFi
- connected devices

### 4-4. Wireless access network

Wireless access network는 base station 또는 access point를 통해 end system을 router에 연결한다.

예시:

- WiFi
- 4G/5G cellular network

### 4-5. Enterprise network

회사나 학교에서 사용하는 네트워크이다.

특징:

- wired Ethernet과 wireless WiFi가 혼합됨
- switch와 router를 사용
- 내부 mail server, web server 등이 존재할 수 있음

### 4-6. Data center network

Data center network는 수백~수천 대의 server를 고속 link로 연결한다.

특징:

- 10Gbps~100Gbps 수준의 high-bandwidth link 사용
- server들을 Internet과 연결
- cloud service, content service에 사용됨

---

## 5. Host가 packet을 보내는 과정

Host는 application message를 작은 chunk로 나누어 packet을 만든다.

Packet의 길이를 L bits, link transmission rate를 R bps라고 하면 packet 하나를 link에 밀어 넣는 시간은 다음과 같다.

```
d_trans = L / R
```

여기서:

- L = packet length
- R = link transmission rate

즉, host는 application message를 packet으로 나누고, 각 packet을 link에 전송한다.

---

## 6. Physical media

Physical media는 bit가 이동하는 실제 매체이다.

두 종류가 있다.

### 6-1. Guided media

신호가 물리적 매체 안에서 이동한다.

예시:

- copper wire
- fiber optic cable
- coaxial cable

### 6-2. Unguided media

신호가 자유 공간을 통해 이동한다.

예시:

- radio
- wireless LAN
- cellular network
- satellite

---

## 7. Network core

Network core는 router들이 연결된 구조이다.

Host가 만든 packet은 여러 router를 거쳐 destination까지 전달된다.

Network core의 두 가지 핵심 기능은 다음과 같다.

### 7-1. Forwarding

Forwarding은 router 내부의 local action이다.

Router가 packet을 받으면 header를 보고 어느 output link로 보낼지 결정한다.

### 7-2. Routing

Routing은 source에서 destination까지의 전체 path를 결정하는 global action이다.

Routing algorithm이 path를 결정하고, forwarding table을 구성하는 데 사용된다.

정리:

| 구분 | 의미 |
| --- | --- |
| Forwarding | router 내부에서 input link의 packet을 적절한 output link로 보내는 동작 |
| Routing | source에서 destination까지의 path를 결정하는 동작 |

---

## 8. Packet switching

Packet switching은 application message를 packet으로 나누어 전송하는 방식이다.

특징:

- 자원을 미리 예약하지 않음
- 여러 사용자가 link와 router resource를 공유함
- bursty data에 효율적
- packet은 router를 거쳐 destination까지 전달됨
- congestion이 발생하면 queueing delay와 packet loss가 생길 수 있음

### 8-1. Store-and-forward

Store-and-forward는 router가 packet 전체를 받은 뒤에야 다음 link로 전송하는 방식이다.

Packet 크기 L bits, link rate R bps일 때 packet 하나를 link에 밀어 넣는 데 걸리는 시간은 다음과 같다.

```
d_trans = L / R
```

즉, router는 packet 일부만 받고 바로 넘기지 않고, 전체 packet을 받은 뒤 다음 link로 보낸다.

### 8-2. Queueing

Packet이 output link로 나가야 하는데 link가 이미 다른 packet을 전송 중이면, packet은 queue에서 기다린다.

이때 발생하는 지연이 queueing delay이다.

## 9-3. Packet loss

Router buffer는 무한하지 않다.

Packet arrival rate가 output link transmission rate보다 큰 상태가 지속되면 queue가 쌓이고, buffer가 가득 차면 packet이 drop된다.

이것이 packet loss이다.

---

## 9. Circuit switching

Circuit switching은 통신 전에 end-to-end resource를 미리 예약하는 방식이다.

특징:

- 자원을 미리 예약함
- dedicated resource 사용
- call setup 필요
- 성능이 안정적임
- 사용하지 않는 동안에도 resource가 묶여 있어 낭비될 수 있음

전통적인 telephone network가 대표적인 예이다.

---

## 10. FDM과 TDM

Circuit switching에서 자원을 나누는 방식으로 FDM과 TDM이 있다.

### 10-1. FDM

FDM은 Frequency Division Multiplexing이다.

Frequency band를 여러 개로 나누고 각 call에 하나의 band를 할당한다.

### 10-2. TDM

TDM은 Time Division Multiplexing이다.

Time slot을 나누고 각 call에 주기적인 slot을 할당한다.

정리:

| 구분 | FDM | TDM |
| --- | --- | --- |
| 나누는 기준 | frequency | time |
| 방식 | 주파수 대역을 나눔 | 시간 슬롯을 나눔 |
| resource 사용 | 각 call이 특정 frequency band 사용 | 각 call이 특정 time slot 사용 |

---

## 11. Packet switching vs Circuit switching

| 구분 | Packet switching | Circuit switching |
| --- | --- | --- |
| 자원 예약 | 하지 않음 | 미리 예약 |
| resource 사용 | 공유 | dedicated |
| 장점 | bursty data에 효율적, resource sharing | 안정적 성능 |
| 단점 | congestion 시 delay/loss 발생 | idle 상태에서도 자원 낭비 |
| 예시 | Internet | telephone network |

Packet switching은 resource sharing을 통해 효율적이지만, congestion이 생기면 queueing delay와 packet loss가 발생한다.

Circuit switching은 안정적이지만, 사용하지 않는 시간에도 resource가 묶여 있어 낭비가 생긴다.

---

## 12. Internet structure

Internet은 여러 ISP와 network가 연결된 network of networks이다.

Access ISP는 end system을 Internet에 연결한다.

Access ISP들은 서로 통신하기 위해 다른 ISP와 연결되어야 한다.

모든 access ISP를 서로 직접 연결하면 O(N²)개의 연결이 필요하므로 scale이 안 된다.

따라서 global transit ISP, regional ISP, content provider network 등이 계층적/경제적 관계 속에서 연결된다.

Internet structure의 핵심:

- access ISP
- regional ISP
- global transit ISP
- content provider network
- peering
- customer-provider relationship

Content provider network는 Google, Microsoft, Akamai 같은 회사가 자신의 content를 사용자 가까이에 제공하기 위해 만든 private network이다.

---

## 13. Delay와 loss

Packet delay는 router buffer에서 packet이 기다리거나 link를 통해 전송/전파되는 과정에서 발생한다.

Nodal delay는 네 가지로 구성된다.

```
d_nodal = d_proc + d_queue + d_trans + d_prop
```

### 13-1. Processing delay

Router가 packet header를 검사하고 output link를 결정하는 데 걸리는 시간이다.

### 13-2. Queueing delay

Packet이 output link로 나가기 전에 queue에서 기다리는 시간이다.

Congestion level에 따라 크게 달라진다.

### 13-3. Transmission delay

Packet의 모든 bit를 link에 밀어 넣는 시간이다.

```
d_trans = L / R
```

- L = packet length
- R = link transmission rate

### 13-4. Propagation delay

Bit가 물리적 link를 따라 이동하는 시간이다.

```
d_prop = d / s
```

- d = link distance
- s = propagation speed

### 13-5. Transmission delay vs Propagation delay

| 구분 | Transmission delay | Propagation delay |
| --- | --- | --- |
| 의미 | packet을 link에 밀어 넣는 시간 | bit가 link를 타고 이동하는 시간 |
| 공식 | L/R | d/s |
| 영향 요소 | packet size, link rate | distance, propagation speed |
| 비유 | 톨게이트에서 차를 밀어 넣는 시간 | 차가 도로를 달려 목적지까지 가는 시간 |

---

## 14. Traffic intensity

Traffic intensity는 다음과 같다.

```
La / R
```

- L = packet length
- a = average packet arrival rate
- R = link transmission rate

의미:

> 들어오는 traffic 양 / 처리 가능한 traffic 양
> 

La/R이 0에 가까우면 queueing delay가 작다.

La/R이 1에 가까워지면 queueing delay가 급격히 증가한다.

La/R이 1보다 크면 들어오는 양이 처리량보다 많으므로 queue가 계속 증가하고 average delay가 무한대로 커진다.

---

## 15. Traceroute

Traceroute는 source에서 destination까지 가는 path상의 router들을 확인하고, 각 router까지의 delay를 측정하는 프로그램이다.

동작 개념:

- TTL 값을 조절하여 packet을 보냄
- 각 router가 reply를 보내게 함
- sender가 response time을 측정함
- 각 hop까지의 delay를 확인할 수 있음

---

## 16. Throughput

Throughput은 단위 시간당 sender에서 receiver로 전달되는 bit 수이다.

종류:

- instantaneous throughput
- average throughput

End-to-end throughput은 보통 bottleneck link에 의해 결정된다.

예를 들어 두 link의 rate가 Rs, Rc라면 평균 throughput은 다음과 같다.

```
throughput = min(Rs, Rc)
```

여러 connection이 같은 bottleneck link를 공유하면 각 connection의 throughput은 link capacity를 나누어 가진다.

---

## 17. Network security

Internet은 처음부터 강한 security를 고려해 설계된 것이 아니다.

따라서 여러 공격이 가능하고, 각 layer에서 security를 고려해야 한다.

### 17-1. Packet sniffing

Packet sniffing은 network를 지나가는 packet을 몰래 읽거나 기록하는 공격이다.

특히 shared Ethernet이나 wireless 환경에서 가능하다.

암호화되지 않은 password 등이 노출될 수 있다.

### 17-2. IP spoofing

IP spoofing은 packet의 source IP address를 위조하는 공격이다.

공격자는 다른 host가 packet을 보낸 것처럼 속일 수 있다.

### 17-3. DoS attack

DoS는 Denial of Service이다.

공격자가 target server나 network에 대량의 bogus traffic을 보내 정상 사용자가 서비스를 이용하지 못하게 만드는 공격이다.

### 17-4. Defense methods

방어 방법:

- authentication
- confidentiality via encryption
- integrity check
- access restriction
- firewall
- detecting/reacting to DoS attacks

---

## 18. Layering

Network는 매우 복잡한 시스템이므로 layer로 나누어 설계한다.

Layering의 장점:

- 복잡한 시스템을 구조화할 수 있음
- 각 layer의 역할을 명확히 구분 가능
- maintenance와 update가 쉬움
- 한 layer의 내부 구현 변화가 다른 layer에 영향을 덜 줌

### 18-1. Internet protocol stack

Internet 5계층:

| Layer | 역할 | 예시 |
| --- | --- | --- |
| Application | network application 지원 | HTTP, SMTP, IMAP, DNS |
| Transport | process-to-process data transfer | TCP, UDP |
| Network | source host에서 destination host까지 datagram 전달 | IP, routing protocol |
| Link | neighboring network element 간 data transfer | Ethernet, WiFi |
| Physical | bit를 물리 매체에 전송 | copper, fiber, radio |

---

## 19. Encapsulation

Encapsulation은 upper layer의 data에 lower layer가 header를 붙이는 과정이다.

데이터 단위 변화:

```
message -> segment -> datagram -> frame
```

| Layer | Data unit |
| --- | --- |
| Application | message |
| Transport | segment |
| Network | datagram |
| Link | frame |

동작:

- Application layer message에 transport header가 붙으면 segment
- Segment에 network header가 붙으면 datagram
- Datagram에 link header가 붙으면 frame

Router와 switch를 지나는 과정에서도 각 layer의 역할에 따라 header가 해석된다.

---

## 20. Internet history

## 20-1. 1961~1972

- packet switching 원리 등장
- ARPAnet 시작
- NCP 등장
- 첫 email program 등장

## 20-2. 1972~1980

- ALOHAnet
- Ethernet
- internetworking 개념 발전
- proprietary network 등장

## 20-3. 1980~1990

- TCP/IP deployment
- SMTP defined
- DNS defined
- FTP defined
- TCP congestion control 등장

## 20-4. 1990~2000s

- ARPAnet decommissioned
- Web 등장
- HTML, HTTP
- Mosaic, Netscape
- commercial Web 성장
- P2P, instant messaging 등 killer app 등장

## 20-5. 2005~present

- broadband home access 확산
- SDN
- 4G/5G, WiFi
- content provider network
- cloud service
- smartphone 확산

# Chapter 2 내용 정리
## 1. Network application

Network application은 서로 다른 end system에서 실행되는 program들이 network를 통해 message를 주고받는 것이다.

중요한 점:

- application은 end system에서 실행됨
- network core device에는 user application을 만들 필요 없음
- router는 application을 실행하지 않고 packet forwarding을 담당함

예시:

- Web
- email
- text messaging
- online games
- streaming video
- P2P file sharing
- voice over IP
- video conferencing
- search
- remote login

---

## 2. Client-server architecture

Client-server 구조는 항상 켜져 있는 server가 있고, client가 server에 접속하는 구조이다.

Server 특징:

- always-on host
- permanent IP address
- data center에 위치하는 경우 많음
- scaling을 위해 여러 server 사용 가능

Client 특징:

- server에 contact
- intermittently connected 가능
- dynamic IP address 가능
- client끼리 직접 통신하지 않음

예시:

- HTTP
- IMAP
- FTP

---

## 3. P2P architecture

P2P architecture는 항상 켜져 있는 server에만 의존하지 않고, peer들이 서로 직접 통신하는 구조이다.

특징:

- no always-on server
- arbitrary end system이 직접 통신
- peer는 service를 요청하기도 하고 제공하기도 함
- self-scalability
- peer가 들어오면 service demand도 늘지만 service capacity도 같이 늘어남
- peer는 자주 접속/종료하고 IP가 바뀔 수 있음
- management가 복잡함

예시:

- BitTorrent
- P2P file sharing

---

## 4. Process

Process는 host에서 실행 중인 program이다.

같은 host 안의 process들은 OS가 제공하는 inter-process communication으로 통신한다.

서로 다른 host의 process들은 network를 통해 message를 교환한다.

P2P architecture의 application도 client process와 server process를 모두 가질 수 있다.

---

## 5. Socket

Socket은 application process와 transport layer 사이의 interface이다.

비유:

> socket은 process가 network로 나가는 문이다.
> 

Process는 socket을 통해 message를 보내고 받는다.

Transport layer는 socket을 통해 받은 message를 상대 process의 socket까지 전달한다.

---

## 6. Addressing processes

Host를 식별하려면 IP address가 필요하다.

하지만 IP address만으로는 process를 식별할 수 없다.

이유:

- 한 host 안에 여러 process가 동시에 실행될 수 있기 때문

따라서 process 식별에는 다음 두 가지가 필요하다.

- IP address: host 식별
- port number: host 내부 process 식별

예시:

- HTTP server: port 80
- mail server: port 25

---

## 7. Application-layer protocol

Application-layer protocol은 다음을 정의한다.

1. Message types
    - request, response 등
2. Message syntax
    - message에 어떤 field가 있고 어떻게 구분되는지
3. Message semantics
    - field가 어떤 의미를 가지는지
4. Rules
    - 언제 message를 보내고, 받으면 어떻게 반응하는지

Open protocol은 RFC에 정의되어 누구나 사용할 수 있고, interoperability를 제공한다.

예: HTTP, SMTP

Proprietary protocol은 특정 회사/서비스가 독자적으로 사용하는 protocol이다.

예: Zoom 등

---

## 8. Transport service requirements

Application마다 transport layer에 요구하는 service가 다르다.

### 8-1. Data integrity

File transfer, email, Web document는 100% reliable data transfer가 필요하다.

Audio/video는 약간의 loss를 tolerate할 수 있다.

### 8-2. Timing

Internet telephony, online game, real-time video conferencing은 low delay가 중요하다.

### 8-3. Throughput

Multimedia application은 일정 수준 이상의 throughput이 필요하다.

Elastic application은 가능한 throughput을 사용하는 방식이다.

### 8-4. Security

Encryption, data integrity, authentication 등이 필요할 수 있다.

---

## 9. TCP service와 UDP service

### 9-1. TCP service

TCP가 제공하는 것:

- reliable transport
- flow control
- congestion control
- connection-oriented service

TCP가 제공하지 않는 것:

- timing guarantee
- minimum throughput guarantee
- security 자체

### 9-2. UDP service

UDP는 unreliable data transfer를 제공한다.

제공하지 않는 것:

- reliability
- flow control
- congestion control
- timing guarantee
- throughput guarantee
- security
- connection setup

UDP는 단순하고 overhead가 작기 때문에 DNS, streaming, HTTP/3 기반 QUIC 등에 사용될 수 있다.

---

## 10. Internet application과 transport protocol

대표적인 mapping:

| Application | Application protocol | Transport protocol |
| --- | --- | --- |
| file transfer | FTP | TCP |
| email | SMTP | TCP |
| Web document | HTTP | TCP |
| Internet telephony | SIP/RTP/proprietary | TCP 또는 UDP |
| streaming audio/video | HTTP, DASH | TCP |
| interactive games | proprietary | UDP 또는 TCP |

---

## 11. TLS

기본 TCP/UDP socket은 encryption이 없다.

Cleartext password가 Internet을 그대로 지나갈 수 있다.

TLS는 TCP connection에 security 기능을 제공한다.

TLS가 제공하는 것:

- encrypted TCP connection
- data integrity
- endpoint authentication

중요:

> TLS는 application layer에서 구현되고, 내부적으로 TCP를 사용한다.
> 

---

## 12. Web and HTTP

### 12-1. Web page와 object

Web page는 여러 object로 구성된다.

Object 예시:

- HTML file
- JPEG image
- audio file
- video file
- script

Base HTML file은 여러 referenced object를 포함할 수 있고, 각 object는 URL로 주소 지정된다.

### 12-2. HTTP overview

HTTP는 HyperText Transfer Protocol이다.

Web의 application-layer protocol이다.

구조:

- client: browser
- server: web server

동작:

- browser가 object를 request
- server가 object를 response

HTTP는 TCP를 사용한다.

기본 흐름:

1. client가 server port 80으로 TCP connection 생성
2. HTTP request 전송
3. server가 HTTP response 전송
4. TCP connection 종료 또는 유지

HTTP는 stateless이다.

즉, server는 past client request 정보를 유지하지 않는다.

---

## 13. HTTP connection type

### 13-1. Non-persistent HTTP

Non-persistent HTTP는 TCP connection 하나당 object 하나만 전송한다.

여러 object를 받으려면 여러 TCP connection이 필요하다.

Response time:

```
2RTT + file transmission time
```

이유:

- TCP connection setup에 1 RTT
- HTTP request 전송 후 response 첫 byte 수신까지 1 RTT
- object transmission time 추가

### 13-2. Persistent HTTP

Persistent HTTP는 하나의 TCP connection을 유지한 상태에서 여러 object를 전송한다.

장점:

- TCP connection setup overhead 감소
- 여러 object를 더 빠르게 받을 수 있음
- HTTP/1.1에서 사용

---

## 14. HTTP request message

HTTP request message는 ASCII text 형식이다.

구성:

- request line
- header lines
- blank line
- optional body

대표 method:

| Method | 의미 |
| --- | --- |
| GET | object 요청 |
| POST | form input 등을 entity body에 담아 전송 |
| HEAD | GET과 유사하지만 header만 요청 |
| PUT | server의 특정 URL에 file/object 업로드 또는 대체 |

GET도 URL의 `?` 뒤에 data를 붙여 server로 data를 보낼 수 있다.

---

## 15. HTTP response message

HTTP response message 구성:

- status line
- header lines
- blank line
- data

대표 status code:

| Code | 의미 |
| --- | --- |
| 200 OK | request succeeded |
| 301 Moved Permanently | object가 이동됨 |
| 400 Bad Request | request message를 이해할 수 없음 |
| 404 Not Found | requested document 없음 |
| 505 HTTP Version Not Supported | HTTP version 미지원 |

---

## 16. Cookies

HTTP는 stateless이므로 server가 client의 이전 request를 기억하지 않는다.

하지만 login, shopping cart, recommendation, session state 같은 기능은 state 유지가 필요하다.

이를 위해 cookie를 사용한다.

Cookie 구성 요소:

1. HTTP response message의 cookie header
2. 다음 HTTP request message의 cookie header
3. user host에 저장되는 cookie file
4. web site의 backend database

Cookie 사용 예:

- authorization
- shopping carts
- recommendations
- user session state

Privacy 문제:

- first-party cookie는 특정 website 내 행동 추적
- third-party persistent cookie는 여러 website에 걸친 tracking 가능
- user가 직접 tracker site를 방문하지 않아도 invisible tracking 가능

---

## 17. Web cache

Web cache는 proxy server라고도 한다.

동작:

1. browser가 HTTP request를 cache로 보냄
2. object가 cache에 있으면 cache가 client에게 반환
3. 없으면 cache가 origin server에 request
4. origin server에서 받은 object를 cache에 저장
5. client에게 object 전달

Web cache는 client에게는 server처럼 동작하고, origin server에게는 client처럼 동작한다.

장점:

- client response time 감소
- access link traffic 감소
- origin server 부담 감소
- content provider가 더 효율적으로 content 제공 가능

---

## 18. Web cache 계산

Cache hit rate가 h이면, origin server까지 가야 하는 request 비율은 1-h이다.

예시:

- average data rate = 1.50 Mbps
- access link rate = 1.54 Mbps
- cache hit rate = 0.4

Origin server로 가는 traffic:

```
0.6 * 1.50 Mbps = 0.9 Mbps
```

Access link utilization:

```
0.9 / 1.54 = 0.58
```

Average delay:

```
0.6 * origin server delay + 0.4 * cache delay
```

Cache delay는 매우 작으므로 average delay가 감소한다.

---

## 19. Conditional GET

Conditional GET은 browser cache에 저장된 object가 최신인지 확인하는 방식이다.

Client request에 다음 header를 포함한다.

```
If-Modified-Since: <date>
```

Server response:

- object가 수정되지 않음: `304 Not Modified`
- object가 수정됨: `200 OK`와 object data 전송

목표:

- 최신 cache가 있으면 object를 다시 보내지 않음
- network resource 절약
- object transmission delay 감소

## 20. HTTP/2

HTTP/2의 목표는 multi-object HTTP request에서 delay를 줄이는 것이다.

HTTP/1.1의 문제:

- single TCP connection 위에서 pipelined GET 사용
- server가 request 순서대로 response하면 작은 object가 큰 object 뒤에서 기다릴 수 있음
- 이를 Head-of-Line blocking이라고 함
- TCP segment loss가 발생하면 object transmission이 stall될 수 있음

HTTP/2 개선:

- object를 frame으로 나눔
- frame transmission interleaving
- client-specified priority
- server push 가능
- application-level HOL blocking 완화

---

## 21. HTTP/2 to HTTP/3

HTTP/2는 single TCP connection 위에서 동작하기 때문에 packet loss recovery가 모든 object transmission을 stall시킬 수 있다.

HTTP/3는 UDP 위에서 동작하며, QUIC을 사용한다.

HTTP/3/QUIC의 핵심:

- UDP 위에서 동작
- security 추가
- per-object 또는 per-stream error control
- congestion control
- 더 나은 multiplexing

---

## 22. Email

Email system의 세 가지 구성 요소:

1. User agent
2. Mail server
3. SMTP

### 22-1. User agent

User agent는 mail reader이다.

기능:

- message 작성
- message 편집
- message 읽기

예:

- Outlook
- iPhone Mail
- Gmail web interface

### 22-2. Mail server

Mail server는 다음을 관리한다.

- mailbox
- outgoing message queue

Mailbox에는 user에게 도착한 incoming message가 저장된다.

---

## 23. SMTP

SMTP는 Simple Mail Transfer Protocol이다.

특징:

- TCP 사용
- port 25
- mail server 사이의 email transfer
- sending mail server가 client 역할
- receiving mail server가 server 역할
- direct transfer
- command/response interaction
- command는 ASCII text
- response는 status code와 phrase
- persistent connection 사용
- message 끝은 CRLF.CRLF로 표시

SMTP transfer 단계:

1. handshaking
2. message transfer
3. closure

---

## 24. HTTP vs SMTP

| 구분 | HTTP | SMTP |
| --- | --- | --- |
| 방식 | pull | push |
| 목적 | web object 가져오기 | email message 보내기 |
| 전송 단위 | object마다 response message | 여러 object가 multipart message로 전송 가능 |
| 공통점 | ASCII command/response, status code | ASCII command/response, status code |

SMTP는 client push 방식이다.

HTTP는 client pull 방식이다.

---

## 25. Mail message format

SMTP는 email message를 교환하기 위한 protocol이다.

Email message 자체의 syntax는 RFC 2822가 정의한다.

Mail message 구성:

- header
- blank line
- body

Header 예:

- To:
- From:
- Subject:

주의:

> email header의 To, From과 SMTP command의 MAIL FROM, RCPT TO는 다르다.
> 

---

## 26. Mail access protocol

SMTP는 receiver mail server에 email을 전달하고 저장하는 데 사용된다.

Receiver가 mail server에서 email을 읽기 위해서는 mail access protocol이 필요하다.

대표:

- IMAP
- HTTP 기반 webmail

IMAP 기능:

- server에 message 저장
- retrieval
- deletion
- folder management

Gmail, Hotmail, Yahoo Mail 등은 web-based interface를 제공하며 내부적으로 SMTP/IMAP 등을 사용한다.

---

## 27. DNS

DNS는 Domain Name System이다.

역할:

> hostname을 IP address로 변환한다.
> 

사람은 `www.example.com` 같은 name을 사용하지만, host/router는 IP address를 사용한다.

DNS는 이 둘을 mapping한다.

DNS 특징:

- distributed database
- hierarchy of name servers
- application-layer protocol
- core Internet function이지만 application layer에서 구현됨
- complexity at network edge

---

## 28. DNS services

DNS가 제공하는 서비스:

- hostname-to-IP-address translation
- host aliasing
- canonical name 관리
- mail server aliasing
- load distribution

DNS를 중앙집중형으로 만들면 안 되는 이유:

- single point of failure
- traffic volume
- distant centralized database
- maintenance problem
- scale 불가능

---

## 29. DNS hierarchy

DNS는 계층 구조이다.

주요 server:

1. Root DNS server
2. TLD DNS server
3. Authoritative DNS server
4. Local DNS server

### 29-1. Root DNS server

최상위 server이다.

직접 답을 모르면 TLD server 정보를 알려준다.

### 29-2. TLD DNS server

Top-Level Domain을 담당한다.

예:

- .com
- .org
- .net
- .edu
- .kr
- .jp

### 29-3. Authoritative DNS server

특정 organization의 hostname-to-IP mapping에 대해 authoritative answer를 제공한다.

### 29-4. Local DNS server

Host가 DNS query를 보낼 때 먼저 문의하는 server이다.

Cache를 가지고 있을 수 있다.

DNS hierarchy에 엄밀히 포함되지는 않지만 resolution 과정에서 중요하다.

---

## 30. Iterative query와 recursive query

### 30-1. Iterative query

Contacted DNS server가 답을 모르면 다음에 물어볼 server를 알려준다.

예:

> “나는 모르지만 이 server에게 물어봐.”
> 

### 30-2. Recursive query

Contacted DNS server가 client 대신 DNS hierarchy를 따라가며 최종 답을 찾아서 반환한다.

비교:

| 구분 | Iterative query | Recursive query |
| --- | --- | --- |
| 응답 | 다음 server 알려줌 | 최종 answer 찾아서 반환 |
| 부담 | requester/local DNS 부담 | contacted DNS server 부담 |
| upper-level server 부하 | 상대적으로 작음 | 커질 수 있음 |

---

## 31. DNS caching

DNS server는 한 번 알게 된 mapping을 cache에 저장한다.

장점:

- response time 감소
- DNS traffic 감소
- root/TLD server 부하 감소

단점:

- cache entry는 TTL 후 삭제됨
- host IP가 바뀌면 TTL이 만료될 때까지 outdated 정보일 수 있음
- best-effort name-to-address translation

---

## 32. DNS Resource Records

DNS database는 resource record를 저장한다.

형식:

```
(name, value, type, ttl)
```

주요 type:

| Type | 의미 |
| --- | --- |
| A | hostname -> IP address |
| NS | domain -> authoritative name server |
| CNAME | alias name -> canonical name |
| MX | domain -> mail server |

---

## 33. DNS protocol message

DNS query와 reply message는 같은 format을 사용한다.

Header field:

- identification
- flags
- number of questions
- number of answers
- number of authority RRs
- number of additional RRs

본문:

- questions
- answers
- authority records
- additional information

Identification은 query와 reply를 matching하는 데 사용된다.

---

## 34. DNS에 정보 등록하기

새로운 domain을 등록하려면 DNS registrar에 domain name을 등록한다.

예:

- `networkutopia.com` 등록
- authoritative name server 이름과 IP 제공
- registrar가 TLD server에 NS record와 A record 삽입
- local authoritative server에 A record, MX record 등을 생성

---

## 35. DNS security

DNS 공격:

### 35-1. DDoS attack

Root server나 TLD server에 대량 traffic을 보내는 공격이다.

Root server 공격은 local DNS cache와 traffic filtering 때문에 성공하기 어렵지만, TLD server 공격은 더 위험할 수 있다.

### 35-2. DNS spoofing / cache poisoning

공격자가 DNS query를 가로채거나 bogus reply를 보내 잘못된 IP address를 cache에 저장하게 만드는 공격이다.

방어:

- DNSSEC
- authentication
- message integrity

---

## 36. P2P architecture 복습

P2P는 no always-on server 구조이다.

Peer들이 직접 통신하면서 file chunk를 주고받는다.

장점:

- self-scalability
- peer가 늘어나면 demand와 capacity가 동시에 증가

단점:

- peer churn
- peer management 복잡
- IP address 변화 가능

---

## 37. Client-server file distribution

File size가 F, peer 수가 N일 때 client-server 방식에서는 server가 N개의 file copy를 upload해야 한다.

공식:

```
D_c-s >= max{NF/u_s, F/d_min}
```

의미:

- `NF/u_s`: server가 N개 copy를 upload하는 데 걸리는 시간
- `F/d_min`: 가장 느린 client가 file 하나를 download하는 시간

N이 증가하면 distribution time이 선형적으로 증가한다.

---

## 38. P2P file distribution

P2P에서는 server뿐 아니라 peer들도 file chunk를 upload한다.

공식:

```
D_P2P >= max{F/u_s, F/d_min, NF/(u_s + Σu_i)}
```

의미:

- `F/u_s`: server가 최소 한 copy를 upload해야 함
- `F/d_min`: 가장 느린 peer가 file 하나를 download해야 함
- `NF/(u_s + Σu_i)`: 전체 upload capacity 기준으로 NF bits를 분배하는 시간

P2P는 peer가 늘어날수록 aggregate upload capacity도 증가하므로 scalable하다.

---

## 39. BitTorrent

BitTorrent는 P2P file distribution protocol이다.

구성:

- file은 256KB chunk로 나뉨
- torrent: 같은 file을 교환하는 peer group
- tracker: torrent에 참여 중인 peer 목록 관리

Peer joining 과정:

1. peer는 처음에 chunk가 없음
2. tracker에 등록하고 peer list를 받음
3. 일부 peer와 connection 생성
4. chunk를 download하면서 동시에 upload
5. file을 다 받은 뒤 떠나거나 계속 남아 upload 가능

---

## 40. BitTorrent chunk 요청과 전송

### 40-1. Rarest first

Peer는 자신이 없는 chunk 중 가장 rare한 chunk를 먼저 요청한다.

목적:

- rare chunk가 사라지는 것을 방지
- chunk availability를 높임
- 전체 torrent에 chunk가 균등하게 퍼지도록 함

### 40-2. Tit-for-tat

Peer는 자신에게 가장 높은 rate로 chunk를 보내주는 4명의 peer에게 upload한다.

특징:

- top 4 peer는 10초마다 재평가
- 나머지는 choked
- 30초마다 random peer 하나를 optimistically unchoke
- 높은 upload rate를 제공하는 peer가 더 빠르게 file을 받을 수 있음

---

## 41. Video streaming

Streaming video traffic은 Internet bandwidth의 큰 부분을 차지한다.

문제:

- scale
- 많은 사용자에게 video 제공
- user마다 bandwidth와 device capability가 다름
- network congestion에 따라 bandwidth가 변함

해결 방향:

- distributed application-level infrastructure
- CDN
- adaptive streaming

---

## 42. Video coding

Video는 일정한 rate로 표시되는 image sequence이다.

Digital image는 pixel의 배열이다.

각 pixel은 bit로 표현된다.

Encoding은 redundancy를 줄여 bit 수를 감소시킨다.

종류:

- spatial coding: 한 image 내부의 redundancy 제거
- temporal coding: 연속 frame 사이의 redundancy 제거

CBR:

- Constant Bit Rate
- encoding rate가 고정

VBR:

- Variable Bit Rate
- scene complexity에 따라 encoding rate 변화

---

## 43. Streaming stored video

Stored video streaming의 문제:

- server-to-client bandwidth가 시간에 따라 변함
- congestion으로 delay 발생
- packet loss 발생 가능
- continuous playout constraint
- pause, fast-forward, rewind, jump 등 interactivity 필요

### 43-1. Client-side buffering

Network delay는 variable하므로 client는 buffer를 사용한다.

목적:

- delay jitter 흡수
- continuous playout 보장
- 끊김 감소

---

## 44. DASH

DASH는 Dynamic Adaptive Streaming over HTTP이다.

Server 동작:

- video file을 여러 chunk로 나눔
- 각 chunk를 여러 encoding rate로 저장
- manifest file 제공
- CDN node에 file 복제

Client 동작:

- server-to-client bandwidth를 주기적으로 추정
- manifest file을 보고 chunk URL 확인
- 현재 bandwidth에 맞는 encoding rate 선택
- buffer starvation/overflow가 생기지 않도록 request timing 결정
- 가까운 server 또는 bandwidth가 좋은 server에서 chunk 요청

핵심:

> DASH의 intelligence는 client에 있다.
> 

---

## 45. CDN

CDN은 Content Distribution Network이다.

Mega-server 하나로 video를 제공하면 안 되는 이유:

- single point of failure
- network congestion
- distant clients에게 긴 delay
- scale 불가능

CDN 방식:

- content copy를 여러 geographically distributed site에 저장
- client와 가까운 CDN node에서 content 제공
- delay 감소
- congestion 감소
- scalability 향상

CDN 배치 방식:

- enter deep: access network 깊숙이 많은 server 배치
- bring home: POP 근처에 큰 cluster 배치

---

## 46. Socket programming

Socket programming의 목표는 socket을 이용해 client/server application을 만드는 것이다.

Socket은 application process와 transport protocol 사이의 door이다.

두 가지 socket type:

| Transport | Socket type | 특징 |
| --- | --- | --- |
| UDP | SOCK_DGRAM | unreliable datagram |
| TCP | SOCK_STREAM | reliable byte stream |

---

## 47. UDP socket programming

UDP 특징:

- connection 없음
- handshaking 없음
- sender가 destination IP address와 port number를 packet마다 명시
- receiver는 sender IP address와 port number를 received packet에서 추출
- data loss 가능
- out-of-order 가능
- application 관점에서는 unreliable datagram transfer

### 47-1. UDP client 흐름

핵심 code 흐름:

```
fromsocketimport*

serverName='hostname'
serverPort=12000

clientSocket=socket(AF_INET,SOCK_DGRAM)

message=input('Input lowercase sentence:')
clientSocket.sendto(message.encode(), (serverName,serverPort))

modifiedMessage,serverAddress=clientSocket.recvfrom(2048)

print(modifiedMessage.decode())
clientSocket.close()
```

핵심:

- `socket(AF_INET, SOCK_DGRAM)`으로 UDP socket 생성
- `sendto()` 사용
- destination address를 sendto에 함께 넣음
- `recvfrom()`으로 message와 server address를 받음
- connection setup이 없음
# Chapter 3 내용 정리
## 1. Transport layer service

Transport layer는 서로 다른 host에서 실행되는 application process 사이에 logical communication을 제공한다.

Transport protocol의 sender/receiver 동작:

Sender:

- application-layer message를 받음
- transport header field 값을 결정
- segment 생성
- segment를 network layer로 전달

Receiver:

- IP에서 segment를 받음
- header 값을 확인
- segment에서 application-layer message 추출
- socket을 통해 해당 process로 demultiplexing

---

## 2. Transport layer vs Network layer

| 구분 | Network layer | Transport layer |
| --- | --- | --- |
| 통신 범위 | host-to-host | process-to-process |
| 대표 protocol | IP | TCP, UDP |
| 데이터 단위 | datagram | segment |
| 역할 | host 사이 전달 | process 사이 logical communication |

Network layer는 host 사이의 통신을 제공한다.

Transport layer는 host 안의 특정 process까지 전달되도록 한다.

비유:

- host = 집
- process = 집 안의 아이
- network layer = 집에서 집으로 편지 배달
- transport layer = 집 안의 특정 아이에게 편지 전달

---

## 3. Transport layer actions

### 3-1. Sender

Sender transport layer는 application message를 받아 segment를 만든다.

과정:

1. application-layer message를 받음
2. segment header field 값을 결정
3. message에 header를 붙여 segment 생성
4. segment를 IP로 전달

### 3-2. Receiver

Receiver transport layer는 IP에서 segment를 받는다.

과정:

1. IP에서 segment 수신
2. header value 확인
3. application-layer message 추출
4. 적절한 socket/process로 demultiplexing

---

## 4. TCP와 UDP

Internet transport protocol의 대표 두 가지는 TCP와 UDP이다.

### 4-1. TCP

TCP는 Transmission Control Protocol이다.

제공 기능:

- reliable delivery
- in-order delivery
- congestion control
- flow control
- connection setup

### 4-2. UDP

UDP는 User Datagram Protocol이다.

특징:

- unreliable
- unordered delivery 가능
- no-frills
- best-effort IP service의 단순 확장
- delay guarantee 없음
- bandwidth guarantee 없음

비교:

| 구분 | TCP | UDP |
| --- | --- | --- |
| 연결 | connection-oriented | connectionless |
| 신뢰성 | reliable | unreliable |
| 순서 보장 | in-order | unordered 가능 |
| flow control | 있음 | 없음 |
| congestion control | 있음 | 없음 |
| setup | 필요 | 없음 |

---

## 5. Multiplexing / Demultiplexing

### 5-1. Multiplexing

Multiplexing은 sender 측에서 여러 socket에서 내려온 data를 transport layer가 segment로 만들어 network layer로 보내는 과정이다.

즉:

> 여러 process의 data를 모아서 transport segment로 만들어 IP에 넘기는 것
> 

### 5-2. Demultiplexing

Demultiplexing은 receiver 측에서 도착한 segment를 header 정보를 보고 올바른 socket/process로 전달하는 과정이다.

즉:

> 도착한 segment가 어느 process로 가야 하는지 구분하는 것
> 

---

## 6. Demultiplexing이 동작하는 방식

Host가 IP datagram을 받으면, 각 datagram 안에는 transport-layer segment가 들어 있다.

Segment에는 다음 정보가 있다.

- source port number
- destination port number

Host는 IP address와 port number 정보를 사용해 segment를 적절한 socket으로 전달한다.

---

## 7. Connectionless demultiplexing: UDP

UDP socket을 만들 때 host-local port number를 지정한다.

UDP receiver는 segment의 destination port number를 확인하고, 해당 port number에 연결된 socket으로 segment를 전달한다.

중요:

> destination port number가 같으면 source IP address나 source port number가 달라도 같은 UDP socket으로 전달될 수 있다.
> 

UDP sender는 datagram을 보낼 때 destination IP address와 destination port number를 명시해야 한다.

---

## 8. Connection-oriented demultiplexing: TCP

TCP socket은 4-tuple로 식별된다.

4-tuple:

1. source IP address
2. source port number
3. destination IP address
4. destination port number

TCP server는 동시에 여러 TCP socket을 지원할 수 있다.

예를 들어 여러 client가 같은 web server의 port 80으로 접속해도, source IP와 source port가 다르므로 서로 다른 connection socket으로 구분할 수 있다.

---

## 9. UDP demux vs TCP demux

| 구분 | UDP | TCP |
| --- | --- | --- |
| 방식 | connectionless demux | connection-oriented demux |
| socket 식별 기준 | destination port number 중심 | 4-tuple |
| source IP/port 영향 | 달라도 같은 socket 가능 | 다르면 다른 socket |
| 예시 | DNS query 처리 | web server의 multiple client connection |

핵심 암기:

> UDP demux는 destination port number가 핵심이고, TCP demux는 4-tuple이 핵심이다.
> 

---

## 10. UDP

UDP는 no-frills, bare-bones Internet transport protocol이다.

특징:

- best-effort service
- UDP segment가 lost될 수 있음
- out-of-order로 application에 전달될 수 있음
- connectionless
- handshaking 없음
- 각 UDP segment는 독립적으로 처리됨

UDP 사용 예:

- streaming multimedia
- DNS
- SNMP
- HTTP/3 기반 QUIC

HTTP/3처럼 UDP 위에서 reliable transfer가 필요하면 application layer에서 reliability와 congestion control을 추가한다.

---

## 11. UDP sender/receiver actions

### 11-1. UDP sender

UDP sender는 다음을 수행한다.

1. application-layer message를 받음
2. UDP header field 값을 결정
3. UDP segment 생성
4. segment를 IP로 전달

### 11-2. UDP receiver

UDP receiver는 다음을 수행한다.

1. IP에서 segment 수신
2. UDP checksum header value 확인
3. application-layer message 추출
4. socket을 통해 application으로 demultiplexing

---

## 12. UDP segment format

UDP header는 단순하다.

주요 field:

| Field | 의미 |
| --- | --- |
| source port number | sender process port |
| destination port number | receiver process port |
| length | UDP segment 전체 길이, header 포함 |
| checksum | bit error 검출 |

UDP segment 구성:

```
source port # | dest port #
length        | checksum
application data(payload)
```

UDP header는 32 bits 단위로 표시된다.

---

## 13. UDP checksum

UDP checksum의 목적은 transmitted segment에서 bit error가 발생했는지 검출하는 것이다.

예:

- 송신한 값: 5, 6, 11
- 수신한 값: 4, 6, 11
- 값이 달라졌으므로 bit error 발생 가능

Checksum은 error detection 기능이다.

Error correction은 아니다.

---

## 14. Internet checksum

Sender 동작:

1. UDP segment 내용을 16-bit integer들의 sequence로 간주
2. 1의 보수 합, one’s complement sum을 계산
3. checksum field에 checksum value 삽입

Receiver 동작:

1. 받은 segment에 대해 checksum 다시 계산
2. 계산값과 checksum field 비교
3. 다르면 error detected
4. 같으면 no error detected로 판단

주의:

> 값이 같다고 해서 오류가 절대 없다는 뜻은 아니다. 일부 오류는 검출되지 않을 수 있다.
> 

---

## 15. Internet checksum 계산에서 carry 처리

16-bit integer를 더할 때 most significant bit에서 carryout이 발생하면 그 carry를 결과에 다시 더한다.

즉, wraparound carry를 적용한다.

Checksum은 최종 sum의 1의 보수 형태로 저장된다.

---

## 16. Internet checksum의 약점

Internet checksum은 약한 error detection 방식이다.

일부 bit error 조합은 checksum 결과를 바꾸지 않을 수 있다.

따라서 receiver가 checksum이 같다고 판단해도 실제로는 error가 존재할 수 있다.

하지만 간단하고 overhead가 낮기 때문에 UDP에서 기본적인 error detection 용도로 사용된다.

---

## 17. UDP Summary

UDP는 다음과 같이 정리할 수 있다.

- no-frills protocol
- best-effort service
- segment가 lost될 수 있음
- segment가 out-of-order로 전달될 수 있음
- connection setup 없음
- handshaking 없음
- no RTT incurred for setup
- checksum으로 error detection 제공
- 필요한 reliability는 application layer에서 추가 가능

시험 답안식:

> UDP is a connectionless, no-frills transport protocol. It provides best-effort delivery, so segments may be lost or delivered out of order. UDP does not provide reliability, flow control, or congestion control, but it has low overhead and no connection setup delay.
> 

---

# 전체 범위 핵심 공식 모음

```
d_trans = L / R
```

Packet의 모든 bit를 link에 밀어 넣는 시간.

```
d_prop = d / s
```

Bit가 물리적 link를 따라 이동하는 시간.

```
d_nodal = d_proc + d_queue + d_trans + d_prop
```

한 node에서 발생하는 전체 delay.

```
traffic intensity = La / R
```

들어오는 traffic 양과 처리 가능한 양의 비율.

```
throughput = min(Rs, Rc)
```

End-to-end throughput은 bottleneck link에 의해 제한됨.

```
Non-persistent HTTP response time = 2RTT + file transmission time
```

TCP setup 1RTT + HTTP request/response 1RTT + file transmission time.

```
D_c-s >= max{NF/u_s, F/d_min}
```

Client-server file distribution time lower bound.

```
D_P2P >= max{F/u_s, F/d_min, NF/(u_s + Σu_i)}
```

P2P file distribution time lower bound.

---

# 전체 범위 핵심 비교표

## Packet switching vs Circuit switching

| 구분 | Packet switching | Circuit switching |
| --- | --- | --- |
| 자원 예약 | 없음 | 있음 |
| resource | 공유 | dedicated |
| 장점 | bursty traffic에 효율적 | 안정적 성능 |
| 단점 | delay/loss 가능 | resource waste 가능 |

## TCP vs UDP

| 구분 | TCP | UDP |
| --- | --- | --- |
| 연결 | connection-oriented | connectionless |
| 신뢰성 | reliable | unreliable |
| 순서 | in-order | unordered 가능 |
| flow control | 있음 | 없음 |
| congestion control | 있음 | 없음 |

## HTTP vs SMTP

| 구분 | HTTP | SMTP |
| --- | --- | --- |
| 방식 | pull | push |
| 목적 | web object 가져오기 | email 보내기 |
| 공통점 | ASCII command/response, status code | ASCII command/response, status code |

## Iterative vs Recursive DNS

| 구분 | Iterative | Recursive |
| --- | --- | --- |
| 응답 | 다음 server 알려줌 | 최종 답 찾아줌 |
| 부담 | requester/local DNS | contacted DNS server |
| 특징 | upper-level server 부담 적음 | upper-level server 부담 증가 가능 |

## UDP demux vs TCP demux

| 구분 | UDP | TCP |
| --- | --- | --- |
| 기준 | destination port number | 4-tuple |
| 4-tuple | 사용 안 함 | source IP, source port, dest IP, dest port |
| 같은 dest port | 같은 socket 가능 | client별 다른 socket 가능 |