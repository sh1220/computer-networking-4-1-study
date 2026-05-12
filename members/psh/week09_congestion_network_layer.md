# Week09 — Congestion Control & Network Layer Overview

# 3.6 Principles of Congestion Control (혼잡 제어의 원리)

## 혼잡(Congestion)이란?

- **혼잡(congestion)**: 네트워크 내 너무 많은 소스가 너무 많은 데이터를 너무 빠르게 보내서, 네트워크가 처리할 수 없는 상태
- 혼잡 제어는 흐름 제어와 다름:
  - **흐름 제어(flow control)**: 수신자 버퍼 오버플로 방지 (1:1, 수신자 보호)
  - **혼잡 제어(congestion control)**: 네트워크 자체의 과부하 방지 (다수 송신자 ↔ 네트워크 전체)
- 혼잡의 증상: 패킷 손실(라우터 버퍼 오버플로), 긴 큐잉 지연(queueing delay)

## 3.6.1 The Causes and the Costs of Congestion (혼잡의 원인과 비용)

### 시나리오 1: 두 송신자, 무한 버퍼를 가진 라우터 하나

**설정**

- 호스트 A, B가 각각 λ_in 바이트/초로 데이터를 전송
- 공유 링크 용량 = R
- 라우터 버퍼 = 무한대 (패킷 손실 없음)
- 재전송 없음

**결과**

- 각 연결의 per-connection throughput: min(λ_in, R/2)
- λ_in < R/2일 때: 처리율 = λ_in (정상)
- λ_in > R/2일 때: 처리율 = R/2로 제한 (포화)
- **지연(delay)**: λ_in이 R/2에 접근할수록 큐잉 지연이 무한대로 증가

> **비용 1**: 패킷 도착률이 링크 용량에 가까워지면, 큐잉 지연이 급격히 증가한다

### 시나리오 2: 두 송신자, 유한 버퍼를 가진 라우터 하나

**설정**

- 라우터 버퍼 = 유한 → 버퍼 오버플로 시 패킷 드롭(drop)
- 패킷 손실 → 재전송 발생
- **λ_in** = 원래(original) 데이터 전송률, **λ'_in** = 원래 데이터 + 재전송 데이터 = offered load

**경우 (a): 송신자가 버퍼 빈 공간을 정확히 알 때 (이상적, 비현실적)**

- 빈 공간이 있을 때만 전송 → 패킷 손실 없음, 재전송 없음
- λ'_in = λ_in, 처리율 = λ_in (R/2까지)

**경우 (b): 송신자가 패킷 손실을 확실히 알고 재전송할 때**

- 평균적으로 R/2 처리율을 얻으려면 λ'_in = R/3 이상이어야
- 재전송된 패킷이 링크 용량의 일부를 차지 → 유용한 처리율(goodput) 감소

> **비용 2**: 패킷 드롭으로 인한 재전송은 라우터가 드롭된 패킷을 전달하는 데 사용한 링크 대역폭을 낭비한다

**경우 (c): 조기 타임아웃으로 불필요한 재전송이 발생할 때**

- 원본 패킷이 아직 네트워크에 있는데 타임아웃 → 중복 패킷 전송
- 수신자에게 두 개 사본 도착 → 하나는 불필요한 복제
- R/2 처리율을 위해 더 많은 offered load 필요

> **비용 3**: 불필요한 재전송(타임아웃에 의한)은 라우터가 동일 패킷의 불필요한 복사본을 전달하게 만들어 링크 대역폭을 추가로 낭비한다

### 시나리오 3: 네 송신자, 다중 홉 경로를 가진 유한 버퍼 라우터들

**설정**

- 4개 호스트가 겹치는 2-홉 경로를 사용
- 각 링크 용량 = R, 유한 버퍼

**핵심 관찰**

- λ'_in이 매우 작을 때: 혼잡 없음, 처리율 ≈ λ'_in
- λ'_in이 매우 클 때: 한 연결의 트래픽이 다른 연결의 트래픽을 라우터에서 밀어냄(crowd out)
- 극단적인 경우: 처리율이 0에 수렴 가능!

> **비용 4**: 패킷이 경로 중간에서 드롭되면, 해당 패킷이 드롭 지점까지 사용한 모든 상류(upstream) 링크의 전송 용량이 전부 낭비된다

### 혼잡의 비용 종합

| 비용 | 설명 |
|---|---|
| 비용 1 | 링크 용량 접근 시 큐잉 지연 급증 |
| 비용 2 | 패킷 손실에 의한 재전송 → 링크 대역폭 낭비 |
| 비용 3 | 불필요한 중복 재전송 → 추가 대역폭 낭비 |
| 비용 4 | 다중 홉 경로에서 중간 드롭 시 상류 링크 전송 용량 전부 낭비 |

## 3.6.2 Approaches to Congestion Control (혼잡 제어의 접근 방식)

### 두 가지 큰 접근법

**1. 종단 간 혼잡 제어 (End-to-end congestion control)**

- 네트워크 계층이 혼잡 정보를 명시적으로 제공하지 않음
- 종단 시스템(end system)이 관찰된 네트워크 동작(패킷 손실, 지연 증가)으로 혼잡을 추론
- **TCP의 기본 접근법**: 타임아웃 또는 3중 중복 ACK → 혼잡으로 판단 → 윈도우 크기 감소

**2. 네트워크 지원 혼잡 제어 (Network-assisted congestion control)**

- 라우터가 송신자/수신자에게 네트워크 혼잡 상태를 명시적으로 피드백
- 피드백 방식:
  - **직접 피드백(direct feedback)**: 라우터 → 송신자 (예: choke packet)
  - **수신자 경유 피드백**: 라우터가 패킷에 혼잡 비트를 마킹 → 수신자가 ACK에 포함하여 송신자에게 전달
- **ECN(Explicit Congestion Notification)**: IP 헤더와 TCP 헤더의 비트를 사용한 네트워크 지원 혼잡 제어 (현대 인터넷에서 사용)

---

# 3.7 TCP Congestion Control (TCP 혼잡 제어)

## 개요

- TCP는 **종단 간 혼잡 제어(end-to-end congestion control)** 사용
- 각 송신자가 네트워크 혼잡 인식에 따라 전송률을 제한
- 핵심 메커니즘: **혼잡 윈도우(congestion window, cwnd)** 를 조절하여 전송률을 제한

### 혼잡 윈도우 (cwnd)

- 송신자의 미확인 데이터량 제약:

LastByteSent − LastByteAcked ≤ min(cwnd, rwnd)

- cwnd: 혼잡 제어에 의해 결정, rwnd: 흐름 제어에 의해 결정
- **송신자의 실효 전송률 ≈ cwnd / RTT** (바이트/초)

### TCP의 혼잡 감지 방법

- **손실 이벤트(loss event)**: 타임아웃 또는 3중 중복 ACK 수신
- 손실 = 혼잡의 징후 → cwnd 감소
- ACK 정상 수신 = 네트워크 양호 → cwnd 증가
- **자기 조절(self-clocking)**: ACK를 네트워크 상태의 암시적 신호로 사용

### TCP 혼잡 제어의 기본 원칙

- **혼잡 없을 때**: cwnd 증가 → 전송률 상승 (가용 대역폭 활용)
- **혼잡 감지 시**: cwnd 감소 → 전송률 하락 (혼잡 완화)

## 3.7.1 TCP 혼잡 제어 알고리즘 (TCP Congestion Control Algorithm)

세 가지 핵심 구성 요소:

1. **슬로 스타트 (Slow Start)**
2. **혼잡 회피 (Congestion Avoidance)**
3. **빠른 복구 (Fast Recovery)**

### 1. 슬로 스타트 (Slow Start)

**초기 동작**

- TCP 연결 시작 시 cwnd = 1 MSS (작게 시작)
- 초기 전송률 ≈ MSS / RTT

**증가 방식**

- ACK가 수신될 때마다 cwnd를 1 MSS씩 증가
- 결과적으로 **매 RTT마다 cwnd가 2배** → 지수적 증가(exponential growth)
- 예: 1 MSS → 2 MSS → 4 MSS → 8 MSS → ...

**슬로 스타트 종료 조건 3가지**

- **조건 1 — 타임아웃에 의한 손실**:
  - ssthresh(slow start threshold) = cwnd / 2 로 설정
  - cwnd = 1 MSS로 리셋
  - 슬로 스타트 재시작

- **조건 2 — cwnd ≥ ssthresh 도달**:
  - 슬로 스타트 종료 → 혼잡 회피(Congestion Avoidance) 모드 전환
  - ssthresh 근처에서는 지수적 증가가 위험하므로 보수적 증가로 전환

- **조건 3 — 3중 중복 ACK 감지**:
  - ssthresh = cwnd / 2
  - cwnd = ssthresh + 3 MSS
  - 빠른 복구(Fast Recovery) 모드 전환

### 2. 혼잡 회피 (Congestion Avoidance)

**진입 시점**

- cwnd가 ssthresh에 도달했을 때 → 지수적 증가 대신 선형적(linear) 증가로 전환

**증가 방식**

- **매 RTT마다 cwnd를 1 MSS씩 증가** (가산적 증가, additive increase)
- 구현: 각 ACK 수신 시 cwnd += MSS × (MSS / cwnd)
  - 예: cwnd = 10 MSS이고 MSS = 1,460 바이트일 때, ACK 하나당 1/10 MSS 증가
  - 10개 ACK(= 1 RTT) 후 cwnd = 11 MSS

**혼잡 회피 중 손실 이벤트 처리**

- **타임아웃 발생 시**:
  - ssthresh = cwnd / 2
  - cwnd = 1 MSS
  - 슬로 스타트로 돌아감

- **3중 중복 ACK 발생 시**:
  - ssthresh = cwnd / 2
  - cwnd = ssthresh + 3 MSS (또는 cwnd를 절반으로, TCP 버전에 따라)
  - 빠른 복구 모드 전환

### 3. 빠른 복구 (Fast Recovery)

**진입 시점**

- 3중 중복 ACK 수신 시 (타임아웃이 아닌 경우)

**동작**

- 중복 ACK를 수신할 때마다 cwnd를 1 MSS씩 증가
  - 이유: 중복 ACK = 네트워크에서 세그먼트가 빠져나갔다는 신호 → 새 세그먼트 전송 가능
- **새 ACK(누락 세그먼트에 대한 ACK) 수신 시**:
  - cwnd = ssthresh
  - 혼잡 회피(Congestion Avoidance) 모드로 전환
- **타임아웃 발생 시**:
  - ssthresh = cwnd / 2
  - cwnd = 1 MSS
  - 슬로 스타트로 전환

**빠른 복구는 TCP Reno의 핵심 특징**

- **TCP Tahoe** (초기 버전): 3중 중복 ACK든 타임아웃이든 무조건 cwnd = 1 MSS, 슬로 스타트 재진입
- **TCP Reno**: 3중 중복 ACK 시 빠른 복구 사용 → cwnd를 절반으로만 줄임 (타임아웃 시에만 cwnd = 1)

### TCP Tahoe vs TCP Reno 비교

| 이벤트 | TCP Tahoe | TCP Reno |
|---|---|---|
| 타임아웃 | cwnd = 1 MSS, 슬로 스타트 | cwnd = 1 MSS, 슬로 스타트 |
| 3중 중복 ACK | cwnd = 1 MSS, 슬로 스타트 | cwnd = cwnd/2, 빠른 복구 |
| ssthresh 설정 | 두 경우 모두 cwnd/2 | 두 경우 모두 cwnd/2 |

### TCP 혼잡 제어 FSM 요약 (Figure 3.51)

```
[슬로 스타트] ──cwnd ≥ ssthresh──→ [혼잡 회피]
     │                                    │
     │ 타임아웃: ssthresh=cwnd/2,          │ 타임아웃: ssthresh=cwnd/2,
     │          cwnd=1                     │          cwnd=1
     ↓                                    ↓
   [슬로 스타트]                        [슬로 스타트]
     │                                    │
     │ 3중복ACK: ssthresh=cwnd/2,         │ 3중복ACK: ssthresh=cwnd/2,
     │           cwnd=ssthresh+3          │           cwnd=ssthresh+3
     ↓                                    ↓
 [빠른 복구] ───새ACK: cwnd=ssthresh───→ [혼잡 회피]
     │
     │ 타임아웃: ssthresh=cwnd/2, cwnd=1
     ↓
  [슬로 스타트]
```

## AIMD (Additive-Increase, Multiplicative-Decrease)

- TCP 혼잡 제어의 핵심 동작 패턴:
  - **가산적 증가(Additive Increase)**: 혼잡 회피 시 매 RTT마다 cwnd를 1 MSS 증가
  - **승법적 감소(Multiplicative Decrease)**: 손실 감지 시 cwnd를 절반으로 감소
- 결과적으로 TCP의 cwnd 그래프는 **톱니 모양(sawtooth pattern)** 을 보임
  - 선형적으로 증가하다가 손실 시 절반으로 떨어지고 다시 선형 증가 반복

### TCP의 평균 처리율 (Average Throughput)

- 윈도우 크기가 W (손실 직전)에서 W/2로 감소한다고 가정
- **평균 처리율 ≈ 0.75 × W / RTT**
  - W와 W/2 사이를 왕복하므로 평균은 약 0.75W

## 3.7.2 TCP CUBIC

### TCP Reno의 한계

- AIMD의 선형 증가가 너무 느림: 고대역폭 링크에서 손실 후 복구에 매우 오래 걸림
- 예: 10 Gbps 링크, RTT = 100ms, MSS = 1,500 바이트
  - cwnd ≈ 83,333 세그먼트가 필요
  - 손실 후 절반(41,666)에서 83,333까지 도달하려면 약 41,667 RTT ≈ 약 70분!

### TCP CUBIC의 핵심 아이디어

- **손실이 발생한 시점의 cwnd 값(W_max)을 기억**
- cwnd를 시간의 **3차 함수(cubic function)** 로 증가시킴

**CUBIC의 윈도우 증가 특성**

- W_max에서 멀리 떨어져 있을 때: 빠르게 증가 (급경사)
- W_max에 가까워질수록: 천천히 증가 (조심스럽게 접근)
- W_max를 지나면: 다시 빠르게 증가 (새로운 가용 대역폭 탐색)

**CUBIC의 윈도우 함수**

W(t) = C × (t − K)³ + W_max

- t = 마지막 손실 이후 경과 시간
- K = W_max에 도달하는 데 걸리는 시간 (즉, W(K) = W_max가 되는 시점)
- C = 상수 파라미터

**Reno vs CUBIC 비교**

- Reno: 선형적으로 천천히 증가 → 고대역폭에서 비효율적
- CUBIC: W_max 근처에서 천천히, 그 외 구간에서 빠르게 → W_max를 빠르게 회복 + 안전하게 탐색
- **CUBIC은 현재 가장 널리 사용되는 TCP 혼잡 제어 알고리즘** (Linux 기본값)

## 3.7.3 네트워크 지원 명시적 혼잡 알림과 지연 기반 혼잡 제어

### ECN: 명시적 혼잡 알림 (Explicit Congestion Notification)

**ECN 개요**

- 네트워크 지원 혼잡 제어(network-assisted congestion control)의 대표적 형태
- IP 데이터그램 헤더의 **TOS(Type of Service) 필드** 내 2비트를 ECN에 사용

**ECN 동작 과정**

- **1단계**: 라우터가 혼잡을 경험하면, 해당 IP 데이터그램의 ECN 비트를 마킹 (CE: Congestion Experienced 코드포인트 설정)
- **2단계**: 수신 호스트가 마킹된 데이터그램을 수신 → TCP 헤더의 **ECE(ECN-Echo) 비트**를 설정하여 ACK에 포함
- **3단계**: 송신 호스트가 ECE 비트가 설정된 ACK를 수신 → 혼잡 감지 → cwnd 절반으로 감소
- **4단계**: 송신 호스트가 다음 세그먼트에 **CWR(Congestion Window Reduced) 비트** 설정 → 수신자에게 혼잡 반응 완료를 알림

**ECN의 장점**

- 패킷 손실 전에 혼잡을 감지 → 패킷 드롭 및 재전송 방지 → 지연 감소, 효율 향상
- 라우터가 명시적으로 혼잡을 알려주므로 손실 기반 추론보다 정확

### 지연 기반 혼잡 제어 (Delay-based Congestion Control)

**기본 아이디어**

- 패킷 손실이 발생하기 전에 혼잡을 미리 감지하는 접근법
- **측정 대상**: RTT 변화 → 경로의 큐잉 지연 증가를 관찰

**동작 원리**

- **RTT_min** = 혼잡 없을 때의 최소 RTT (경로의 전파 지연만 포함)
- **비혼잡 시 처리율 상한** = cwnd / RTT_min
- 실제 측정 처리율 = cwnd / RTT_measured

**판단 기준**

- 측정 처리율 ≈ cwnd / RTT_min → 경로 혼잡 없음 → cwnd 증가
- 측정 처리율 < cwnd / RTT_min → 경로에 큐가 쌓이고 있음 → cwnd 감소

**TCP Vegas**

- 지연 기반 혼잡 제어의 초기 구현
- RTT 기반으로 큐잉 지연을 추정하여 cwnd 조절

**TCP BBR (Bottleneck Bandwidth and Round-trip propagation time)**

- Google이 개발한 지연 기반 혼잡 제어 알고리즘
- 병목 대역폭(bottleneck bandwidth)과 최소 RTT를 동시에 추정
- 손실이 아닌 지연과 대역폭 측정을 기반으로 전송률 결정
- Google 내부 네트워크와 YouTube에서 사용

## 3.7.4 공정성 (Fairness)

### 공정성이란?

- **공정성(fairness)**: K개의 TCP 연결이 대역폭 R의 병목 링크를 공유할 때, 각 연결이 R/K의 평균 전송률을 얻는 것이 이상적

### TCP의 공정성

- **AIMD 동작이 공정한 대역폭 분배로 수렴함을 증명 가능**
- 두 연결의 예시:
  - 두 연결 모두 동일한 RTT를 가진다고 가정
  - 가산적 증가: 45도 방향으로 처리율 증가 (두 연결 동일하게 증가)
  - 승법적 감소: 원점 방향으로 처리율 감소 (비례적 감소)
  - 반복 후 점차 공정한 분배점(equal bandwidth share)으로 수렴

### 실제 공정성의 한계

**1. RTT 차이 문제**

- RTT가 작은 연결이 더 빨리 cwnd를 증가시킴 → **RTT가 작을수록 더 큰 대역폭을 차지**
- 동일한 R/K 분배가 보장되지 않음

**2. UDP와의 공정성 문제**

- UDP는 혼잡 제어를 하지 않음 → 손실이 발생해도 전송률을 줄이지 않음
- TCP 연결들이 혼잡에 반응하여 전송률을 줄이면, UDP가 해당 대역폭을 차지
- **UDP 트래픽이 TCP 트래픽을 밀어낼(crowd out) 수 있음**

**3. 병렬 TCP 연결 문제**

- 한 애플리케이션이 여러 병렬 TCP 연결을 열면, 단일 연결보다 더 큰 대역폭을 가져감
- 예: 9개 기존 연결 + 새 앱이 11개 연결 → 새 앱이 R/2 이상 차지 가능
- 웹 브라우저가 실제로 이를 사용 (동일 서버에 다중 연결)

---

# Ch4 네트워크 계층: 데이터 평면 (Network Layer: Data Plane)

# 4.1 Overview of Network Layer (네트워크 계층 개요)

## 4.1.1 네트워크 계층의 역할

### 핵심 기능

- **네트워크 계층의 주된 역할**: 송신 호스트에서 수신 호스트로 세그먼트를 전달(transport)하는 것
- **송신 측**: 전송 계층 세그먼트를 받아 → IP 데이터그램(datagram)으로 캡슐화(encapsulate) → 네트워크로 전송
- **수신 측**: 데이터그램에서 세그먼트를 추출(extract) → 전송 계층으로 전달
- **모든 호스트와 라우터에 네트워크 계층 프로토콜 존재** (전송 계층은 종단 호스트에만 존재)

### 네트워크 계층의 두 가지 핵심 기능

**1. 포워딩 (Forwarding) — 데이터 평면(Data Plane)**

- **로컬 동작**: 라우터의 입력 포트에 도착한 패킷을 적절한 출력 포트로 이동시키는 것
- **매우 빠르게 수행**: 하드웨어 수준, 나노초 단위
- **포워딩 테이블(forwarding table)** 참조: 패킷 헤더의 값 → 출력 링크 인터페이스 결정

**2. 라우팅 (Routing) — 제어 평면(Control Plane)**

- **네트워크 전체 동작**: 패킷이 출발지에서 목적지까지 이동하는 경로(route/path)를 결정하는 과정
- **라우팅 알고리즘(routing algorithm)** 사용: 경로 계산
- **상대적으로 느림**: 소프트웨어 수준, 초 단위

### 포워딩과 라우팅의 비유

- **라우팅 = 여행 계획 세우기**: 출발지에서 목적지까지의 전체 경로를 지도에서 선택
- **포워딩 = 교차로에서 방향 전환**: 각 교차로에서 어느 도로로 갈지 결정

### 포워딩 테이블 (Forwarding Table)

- 라우터는 도착 패킷의 **헤더 필드 값(header field value)** 을 검사
- 이 값을 포워딩 테이블에서 인덱스로 사용하여 출력 링크 인터페이스를 결정
- **핵심 질문**: 포워딩 테이블은 어떻게 구성되고 유지되는가? → 이것이 제어 평면의 역할

## 4.1.2 Control Plane: The Traditional Approach (전통적 접근법)

### 전통적 라우팅 (Per-router Control Plane)

- **각 라우터에 라우팅 알고리즘 구성 요소가 실행**
- 라우터의 라우팅 프로세서가 다른 라우터의 라우팅 프로세서와 통신 (라우팅 메시지 교환)
- 각 라우터가 자체적으로 포워딩 테이블을 계산
- 라우팅 알고리즘이 포워딩 테이블의 내용을 결정

**특징**

- 포워딩 기능과 라우팅 기능이 각 라우터 내에 함께 존재
- 분산 방식(distributed): 각 라우터가 독립적으로 계산, 이웃 라우터와 정보 교환
- 예: OSPF, BGP 등 전통적인 라우팅 프로토콜

## 4.1.3 Control Plane: The SDN Approach (SDN 접근법)

### SDN (Software-Defined Networking)

- **원격의 컨트롤러(remote controller)가 포워딩 테이블을 계산하고 각 라우터에 배포**
- 라우터 자체에는 라우팅 알고리즘이 없음 → 포워딩 기능만 수행

**SDN의 구조**

- **원격 컨트롤러(Remote Controller)**:
  - 네트워크 전체 토폴로지 정보를 수집
  - 중앙 집중식으로 포워딩 테이블 계산
  - 각 라우터에 포워딩 테이블을 설치(install)
- **라우터(스위치)**: 컨트롤러가 보낸 포워딩 테이블에 따라 패킷 포워딩만 수행

**전통적 방식 vs SDN 비교**

| 항목 | 전통적 방식 (Per-router) | SDN |
|---|---|---|
| 라우팅 계산 위치 | 각 라우터 내부 (분산) | 원격 컨트롤러 (중앙 집중) |
| 포워딩 테이블 생성 | 라우터 자체 계산 | 컨트롤러가 계산 후 배포 |
| 라우터 역할 | 포워딩 + 라우팅 | 포워딩만 |
| 유연성 | 프로토콜에 종속 | 소프트웨어로 자유롭게 제어 가능 |

**SDN의 장점**

- 네트워크 전체의 글로벌 뷰(global view) 확보 → 최적 경로 계산 용이
- 소프트웨어 기반 → 새로운 라우팅 정책 적용이 빠르고 유연
- 트래픽 엔지니어링(traffic engineering) 용이

## 4.1.4 Network Service Model (네트워크 서비스 모델)

### 네트워크 서비스 모델이란?

- **네트워크 계층이 전송 계층에게 제공하는 서비스의 정의**
- 송신 호스트에서 수신 호스트까지 패킷을 전달할 때 어떤 수준의 서비스를 보장하는가?

### 가능한 서비스들 (개별 데이터그램 수준)

- **보장된 전달(guaranteed delivery)**: 패킷이 반드시 목적지에 도착
- **제한된 지연의 보장 전달(guaranteed delivery with bounded delay)**: 특정 지연 한계 내에 반드시 도착

### 가능한 서비스들 (데이터그램 흐름 수준)

- **순서 보장 전달(in-order delivery)**: 전송 순서대로 목적지에 도착
- **보장된 최소 대역폭(guaranteed minimum bandwidth)**: 특정 비트 전송률 이상 보장
- **패킷 간 간격 보장(guaranteed maximum jitter)**: 연속 패킷 간 송/수신 시간 차이 제한

### 인터넷의 네트워크 서비스 모델: Best-Effort Service

- **인터넷의 네트워크 계층은 best-effort 서비스 제공**
  - 보장된 전달: **없음** (패킷 손실 가능)
  - 보장된 지연: **없음** (지연 변동 가능)
  - 순서 보장: **없음** (패킷 순서 바뀔 수 있음)
  - 최소 대역폭 보장: **없음**
  - 보안 보장: **없음**
- **"best-effort = no effort"** 라는 비판도 있지만, 이 단순한 모델이 인터넷의 성공 비결:
  - 구현이 간단 → 네트워크 확장 용이
  - 충분한 대역폭 제공(over-provisioning)으로 실질적으로 "충분히 좋은(good enough)" 성능 달성
  - 부족한 부분은 애플리케이션 계층에서 보완 (예: TCP의 신뢰적 전송, CDN의 지연 최소화)

### 다른 네트워크 아키텍처의 서비스 모델

- **ATM(Asynchronous Transfer Mode)** 네트워크는 다양한 서비스 모델 제공:
  - **CBR(Constant Bit Rate)**: 일정 전송률, 지연 보장
  - **ABR(Available Bit Rate)**: 최소 전송률 보장
- 그러나 인터넷의 best-effort가 사실상 승리 → 복잡한 QoS 메커니즘보다 대역폭 확충이 더 효과적이었음

### 핵심 정리

- 인터넷의 IP 서비스 모델은 **best-effort**: 아무것도 보장하지 않음
- 그럼에도 불구하고 충분한 대역폭 + 종단 프로토콜(TCP 등)의 보완으로 동영상 스트리밍, VoIP 등 다양한 실시간 서비스가 가능해짐
- 네트워크를 단순하게 유지하고, 복잡성은 종단(end)에 두는 **종단 간 원칙(end-to-end principle)** 의 대표적 사례
