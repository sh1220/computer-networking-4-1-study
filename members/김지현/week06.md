# Chapter 2

---

# 2.5 Peer-to-Peer File Distribution

## 핵심 개념

- P2P: 각 peer가 client + server 역할 수행
- 중앙 서버 의존 없음 → 높은 확장성

---

## Distribution Time

### Client-Server

- bottleneck: 서버 업로드
- 최소 시간:

```
max { N·F / u_s , F / d_min }
```

---

### P2P

- peer도 업로드 참여
- 최소 시간:

```
max { F / u_s , F / d_min , N·F / (u_s + Σu_i) }
```

---

## 결론

- peer 수 증가 → 총 업로드 증가
- P2P가 대규모에서 더 효율적

---

## BitTorrent

### 구조

- 파일을 chunk로 분할
- peer 간 교환

### 개념

- tracker: peer 목록 관리
- swarm: peer 집합

### 전략

- rarest-first: 가장 희귀한 chunk 우선
- tit-for-tat: 업로드 많이 하는 peer 우선

---

# 2.6 Video Streaming and CDN

---

## Video Streaming

- 대용량 데이터
- delay-sensitive

---

## HTTP Streaming

- video를 chunk로 나눠 HTTP로 전송

---

## DASH

- 여러 bitrate 버전 제공
- 클라이언트가 상황에 맞게 선택
- 기준:
  - 현재 bandwidth
  - buffer 상태

---

## CDN

### 정의

- 콘텐츠를 여러 서버에 분산 저장

---

## 동작 방식

- pull 방식
  - 요청 시 콘텐츠 가져옴
  - 동시에 전달 + 캐싱

---

## 배치 전략

- enter-deep: ISP 내부까지 배치
- bring-home: 대형 데이터센터 중심

---

## 효과

- latency 감소
- 트래픽 분산

---

# 2.7 Socket Programming

---

## 개념

- socket = application ↔ transport 인터페이스

---

## UDP

- connectionless
- unreliable
- 빠름
- 순서 보장 없음, 재전송 없음

---

## TCP

- connection-oriented
- reliable
- byte-stream

---

## TCP 흐름

### Server

- socket → bind → listen → accept

### Client

- socket → connect

---

## 비교

| 항목   | UDP  | TCP             |
| ------ | ---- | --------------- |
| 연결   | 없음 | 있음            |
| 신뢰성 | 없음 | 있음            |
| 속도   | 빠름 | 상대적으로 느림 |

---

# Chapter 3

---

# 3.1 Transport Layer

---

## 역할

- process-to-process communication

---

## 위치

- application과 network 사이

---

## 핵심 기능

### 1) Multiplexing / Demultiplexing

- port 번호로 프로세스 구분

---

### 2) Reliability

- 데이터 손실 방지 (TCP)

---

### 3) Flow Control

- 수신 속도 맞춤

---

### 4) Congestion Control

- 네트워크 혼잡 제어

---

## Transport vs Network

- Network layer: host-to-host
- Transport layer: process-to-process

---

## 특징

- transport layer는 end system에서만 동작
- router는 transport 정보 처리 안 함

---

## Service Model

application에 제공되는 서비스

- reliable data transfer
- throughput
- timing
- security

---

- transport 서비스는 network layer에 의해 제한됨

---

## Internet Transport Protocol

### TCP

- reliable
- congestion control
- flow control

---

### UDP

- best effort
- 최소 기능

---

# 시험 핵심 요약

- P2P vs Client-Server 시간 식
- BitTorrent 전략 2개
- DASH = adaptive bitrate
- CDN = pull + 캐싱
- UDP vs TCP 차이
- multiplexing = port
- transport vs network 차이
