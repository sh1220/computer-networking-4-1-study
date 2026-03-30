# Chapter 1.4 Delay, Loss, and Throughput in Packet-Switched Networks

## 1. Packet Loss

- 실제 네트워크에서는 **큐(queue)의 크기가 제한적**
- 트래픽 증가 시 큐가 가득 차면 패킷이 저장되지 못하고 **버려짐**
- 이를 **Packet Loss**라고 함

### 핵심 개념

- 트래픽 강도 증가 → 지연 증가 + 손실 증가
- 노드 성능 평가 기준
  - Delay
  - Packet Loss Probability

### 특징

- 손실된 패킷은 종단 간 재전송 가능
- 네트워크 코어에서 사라진 것처럼 보임

## 2. End-to-End Delay

종단 간 지연은 각 노드에서 발생하는 지연의 합

### 공식

[
d_{end-end} = N(d_{proc} + d_{trans} + d_{prop})
]

### 구성 요소

| 요소               | 설명                         |
| ------------------ | ---------------------------- |
| Processing Delay   | 패킷 처리 시간               |
| Transmission Delay | 패킷을 링크에 밀어 넣는 시간 |
| Propagation Delay  | 신호가 이동하는 시간         |
| Queuing Delay      | 대기 시간                    |

### 특징

- 네트워크가 혼잡하지 않으면 queuing delay 무시 가능
- 각 노드의 지연이 누적됨

## 3. Traceroute

네트워크 경로와 지연을 측정하는 도구

### 동작 방식

- 목적지로 여러 개의 특수 패킷 전송
- 각 라우터는 응답 메시지 반환
- 출발지에서 왕복 시간(RTT) 측정

### 특징

- 각 홉(router)의 주소와 이름 확인 가능
- 패킷 손실 시 `*` 표시

### 분석 포인트

- RTT 값은 다음을 모두 포함
  - processing
  - transmission
  - propagation
  - queuing

### 예시 해석

- 특정 구간에서 RTT 급증 → 장거리 링크 존재 가능
- 페이지 3 예시에서 대서양 광케이블 구간에서 지연 증가

## 4. Additional Delays

### 1) Medium Access Delay

- WiFi 등 공유 매체에서 전송 대기

### 2) Packetization Delay

- VoIP에서 음성을 패킷으로 채우는 시간

## 5. Throughput

### 정의

- 단위 시간당 전달되는 데이터량

### 종류

- Instantaneous Throughput
- Average Throughput = F / T

### 5.1 Two-Link Network

페이지 4 그림 기준

- 서버 → 라우터 → 클라이언트

[
Throughput = \min(R_s, R_c)
]

### 핵심 개념

- 가장 느린 링크 = Bottleneck

### 5.2 N-Link Network

[
Throughput = \min(R_1, R_2, ..., R_N)
]

### 5.3 Internet 구조에서의 Throughput

- 코어 네트워크는 고속
- 실제 병목은 대부분 **Access Network**

### 5.4 Shared Link 영향

- 여러 사용자 공유 시 throughput 감소

예시

- R = 5 Mbps
- 10명 공유 → 500 kbps

## 핵심 정리

- Delay = 누적 개념
- Loss = 큐 오버플로우
- Throughput = 병목 링크에 의해 결정

# Chapter 1.5 Protocol Layers and Their Service Models

## 1. Layered Architecture

복잡한 네트워크를 계층으로 나눔

### 비유

- 항공 시스템
  - Ticket → Baggage → Gate → Takeoff → Routing

페이지 7~8 그림 참고

### 계층의 역할

각 계층은

1. 특정 기능 수행
2. 하위 계층 서비스 사용
3. 상위 계층에 서비스 제공

### 장점

- 모듈화
- 유지보수 용이
- 독립적 개발 가능

### 단점

- 기능 중복 가능
- 계층 간 정보 부족 문제

## 2. Protocol Layering

### 구현 방식

- Application / Transport → Software
- Link / Physical → Hardware
- Network → 혼합

### 분산 구조

- 프로토콜은 네트워크 전체에 분산됨

## 3. Internet Protocol Stack

총 5계층

| 계층        | 역할             |
| ----------- | ---------------- |
| Application | 사용자 서비스    |
| Transport   | 프로세스 간 통신 |
| Network     | 라우팅           |
| Link        | 노드 간 전달     |
| Physical    | 비트 전송        |

페이지 10 그림

## 4. 각 계층 상세

### 4.1 Application Layer

- HTTP, SMTP, FTP, DNS
- 데이터 단위: Message

### 4.2 Transport Layer

#### TCP

- 신뢰성
- 흐름 제어
- 혼잡 제어

#### UDP

- 비연결형
- 최소 기능

데이터 단위: Segment

### 4.3 Network Layer

- IP 기반 데이터 전달
- 라우팅 수행

데이터 단위: Datagram

### 4.4 Link Layer

- 인접 노드 간 전달
- Ethernet, WiFi

데이터 단위: Frame

### 4.5 Physical Layer

- 비트 단위 전송
- 전송 매체 의존

## 5. Encapsulation

데이터가 계층을 내려가며 헤더가 붙음

### 흐름

1. Message
2. Segment
3. Datagram
4. Frame

### 구조

- Header + Payload
- Payload = 상위 계층 데이터

### 비유 (우편 시스템)

- Message → 편지
- Segment → 내부 봉투
- Datagram → 외부 봉투

### 특징

- 계층마다 헤더 추가
- 수신 시 역순으로 제거

### 추가 사항

- 큰 메시지는 분할됨
- 수신 측에서 재조립 필요

# Chapter 1.6 Networks Under Attack

## 1. Network Security 개요

- 인터넷은 필수 인프라
- 동시에 공격 대상

## 2. Malware

### 특징

- 사용자 기기 감염
- 정보 탈취
- 파일 삭제

### Botnet

- 감염된 기기 집합
- 공격에 활용됨

### 확산 특징

- 자기 복제
- 빠른 전파

## 3. DoS (Denial of Service)

서비스를 마비시키는 공격

### 유형

#### 1) Vulnerability Attack

- 취약점 이용

#### 2) Bandwidth Flooding

- 트래픽 과부하

---

# Chapter 2 Application Layer

## 2.1 Principles of Network Applications

## 1. Application Layer 개요

- 애플리케이션 계층은 **네트워크 애플리케이션이 동작하는 계층**
- 여러 종단 시스템에서 실행되는 **소프트웨어로 구성됨**

### 핵심 특징

- 애플리케이션은 네트워크 코어가 아니라 **end system에서 구현**
- 새로운 애플리케이션 개발 시
  - 네트워크 장비 수정 필요 없음
  - 프로그램만 작성하면 됨

## 2. Network Application Architecture

### 1) Client-Server Architecture

- 항상 켜져 있는 서버 존재
- 클라이언트가 요청

#### 특징

- 서버: 항상 실행
- 클라이언트: 요청 시 연결
- 예시: Web, Email

### 2) P2P Architecture

- 중앙 서버 없음
- 각 노드가 client + server 역할

#### 특징

- 확장성 높음
- 비용 효율적
- 관리 어려움

## 3. Process Communication

네트워크에서 통신 주체는 **host가 아니라 process**

### 통신 구조

- process ↔ process communication

### 3.1 Addressing

프로세스를 식별하기 위한 요소

1. IP Address → 호스트 식별
2. Port Number → 프로세스 식별

예시

- HTTP → 80
- SMTP → 25

## 4. Transport Layer Interface (Socket)

- 애플리케이션은 **socket을 통해 통신**
- socket = API

### 구조

```
Application
   ↓
Socket
   ↓
Transport Layer (TCP / UDP)
```

## 5. Transport Services

애플리케이션이 요구하는 서비스 4가지

### 1) Reliability

- 데이터 손실 없이 전달

### 2) Throughput

- 최소 전송률 요구 가능

### 3) Timing

- 지연 시간 요구

### 4) Security

- 암호화 및 인증

## 6. TCP vs UDP

### TCP

- Reliable
- Flow Control
- Congestion Control
- 연결 지향

### UDP

- Best effort
- 비연결형
- 빠름

### 중요한 포인트

- 인터넷은 **throughput / timing 보장 없음**

## 7. TLS (보안)

- TCP 위에서 동작
- 암호화 제공

### 특징

- 별도 프로토콜이 아니라 TCP 확장
- 애플리케이션에서 구현됨

참고링크: https://jyun627.notion.site/computer-network-weeo04?source=copy_link
