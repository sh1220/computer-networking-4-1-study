# Computer Networks Study Notes
### notion link
- https://mini-gambler-27b.notion.site/Week-4-333ec367b0e98096bc52e392c74f8188?source=copy_link


## Delay in Packet-Switched Networks

### Overview
- 실제 네트워크는 throughput 제한, delay 발생, packet 손실이 존재
- packet은 `End device → Access → Dist → Core → Dist → Access → End device` 경로로 이동
- 이동 과정에서 node → node 간 여러 종류의 지연 발생

### Router 계층
| 계층 | 역할 |
|------|------|
| Core Router | ISP backbone, 수백만 라우팅 테이블, 수십 Gbps 처리 |
| Distribution Router | Core ↔ Access 트래픽 집약, ACL·QoS 정책 적용 |
| Access Router | End device와 직접 연결되는 최하위 계층 |

### Types of Nodal Delay

| 종류 | 단위 | 설명 |
|------|------|------|
| Processing Delay | microseconds 이하 | 헤더 검사, 다음 목적지 결정, bit error 확인 |
| Queuing Delay | microseconds ~ milliseconds | outbound link 대기. traffic 양에 따라 가변 |
| Transmission Delay | microseconds ~ milliseconds | packet L(bit)을 link에 밀어 넣는 시간. `L/R` |
| Propagation Delay | milliseconds (WAN) | bit가 물리 매체를 통해 전파되는 시간. `d/s` |

---

## Network Security

### 개요
- 공격 방법 이해 → 방어 기법 설계 → 안전한 아키텍처 구축을 포괄하는 분야
- 인터넷은 원래 "서로 신뢰하는 사용자들의 투명한 네트워크"로 설계되어 모든 레이어에서 후속 보안 조치가 필요

### 공격 유형

**Malware (악성 소프트웨어)**
- 감염 경로
  - Virus: 사용자가 악성 객체를 직접 실행할 때 감염 및 자가 복제
  - Worm: 사용자 개입 없이 수동적으로 수신된 객체가 스스로 실행되어 전파
- 감염 후 행동
  - Spyware: 키스트로크·방문 사이트 기록 후 외부 서버로 전송
  - Botnet 편입: 좀비 PC가 되어 스팸 발송 또는 DDoS 공격에 악용

**DoS / DDoS (서비스 거부 공격)**
- 가짜 트래픽으로 서버·대역폭을 가득 채워 정상 사용자의 접근을 차단
- 공격 과정: 타깃 선정 → Botnet 구축 → 대량 bogus 패킷 전송
- DDoS는 출처를 분산시켜 추적을 어렵게 함

**Packet Sniffing (패킷 도청)**
- Shared Ethernet, Wi-Fi 등 broadcast media 환경에서 발생
- NIC를 promiscuous mode로 설정하면 자신이 목적지가 아닌 패킷까지 기록 가능
- 암호화되지 않은 패스워드, 세션 쿠키 등 민감 정보 노출

**IP Spoofing (IP 주소 위조)**
- 패킷 전송 시 출발지 IP 주소를 가짜로 설정
- DoS/DDoS 출처 은폐 또는 신뢰된 IP로 위장해 접근 권한 획득에 악용

---

## Protocol Layers

### 계층화(Layering)의 이유
- 복잡한 시스템을 명확한 구조로 쪼개 각 부분의 역할과 관계를 파악하기 쉽게 함
- 모듈화: 한 레이어의 구현이 바뀌어도 나머지 레이어에 영향을 주지 않음

### Internet Protocol Stack (5계층)

| 계층 | 역할 | 주요 프로토콜 |
|------|------|--------------|
| Application | 네트워크 애플리케이션 지원 | HTTP, SMTP, IMAP |
| Transport | 프로세스 간 데이터 전송 | TCP, UDP |
| Network | 출발지 → 목적지 datagram 라우팅 | IP, routing protocols |
| Link | 인접한 네트워크 요소 간 데이터 전송 | Ethernet, 802.11, PPP |
| Physical | 비트를 물리 매체에 올려 전송 | - |

### Encapsulation (캡슐화)
```
[Source]
  Application  →  메시지 M
  Transport    →  Ht + M          (segment)
  Network      →  Hn + Ht + M     (datagram)
  Link         →  Hl + Hn + Ht + M (frame)

[Router]  Link·Network 레이어까지만 처리 후 재포장

[Destination]  역순으로 헤더를 벗겨내며 원본 메시지 M 수신
```

---

## Application Layer

### Application Architecture

**Client-Server**
- Server: always-on, 고정 IP, 주로 데이터센터에 위치
- Client: 간헐적 연결, 동적 IP, 클라이언트끼리 직접 통신 불가
- 예: HTTP, IMAP, FTP

**Peer-to-Peer (P2P)**
- always-on 서버 없이 임의의 end system끼리 직접 통신
- Self-scalability: 새로운 피어가 추가되면 수요와 서비스 용량이 함께 증가
- 간헐적 연결과 잦은 IP 변경으로 관리가 복잡함
- 예: P2P file sharing

### Processes Communicating

**Process & Socket**
- Process: 호스트 내에서 실행 중인 프로그램
  - Client process: 통신을 먼저 시작
  - Server process: 연결 요청을 기다림
- Socket: 프로세스가 메시지를 송수신하는 인터페이스 (문에 비유)
  - 앱 개발자: application layer(소켓 위) 제어
  - OS: transport layer 이하(소켓 아래) 제어

**Addressing**
- 프로세스 식별자 = IP 주소 + Port 번호
- IP 주소만으로는 부족 (같은 호스트에서 여러 프로세스가 동시 실행 가능)
- HTTP server: port 80 / Mail server: port 25

**Application-layer Protocol이 정의하는 것**
- 메시지 타입 (request, response)
- 메시지 문법(syntax): 필드 구성 및 구분 방식
- 메시지 의미(semantics): 각 필드의 의미
- 규칙(rules): 언제 어떻게 메시지를 보내고 응답하는지
- Open protocol (RFC 공개, 예: HTTP·SMTP) vs Proprietary protocol (예: Skype)

### Internet Transport Protocols

**TCP**
- 신뢰성 있는 데이터 전송 보장
- Flow control: 수신자가 처리할 수 있는 만큼만 송신
- Congestion control: 네트워크 과부하 시 송신 속도 조절
- Connection-oriented: 통신 전 연결 설정 필요
- 미제공: timing, minimum throughput 보장, security

**UDP**
- 비신뢰성 데이터 전송, 오버헤드 없음
- flow control·congestion control·timing·security·연결 설정 모두 미제공
- 오버헤드가 없어 빠르므로 실시간성이 중요한 앱에 적합

**TLS (Transport Layer Security)**
- 기본 TCP/UDP 소켓은 평문 전송 → 민감 정보 노출 위험
- TLS: 암호화된 TCP 연결 + 데이터 무결성 + end-point authentication 제공
- Application layer에서 구현 (앱 → TLS 라이브러리 → TCP)
