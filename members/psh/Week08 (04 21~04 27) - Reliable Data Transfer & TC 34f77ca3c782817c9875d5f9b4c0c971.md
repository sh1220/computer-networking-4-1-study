# Week08 (04/21~04/27) - Reliable Data Transfer & TCP

# 3.4 Principles of Reliable Data Transfer (신뢰적 데이터 전송의 원리)

### 개요

- **신뢰적 채널(reliable channel)**: 상위 계층에 제공되는 서비스 추상화. 전송된 데이터 비트가 손상되거나 손실되지 않고, 전송 순서대로 모두 전달됨
- **핵심 인터페이스 함수:**
- **rdt_send()**: 송신 측 상위 계층이 호출 → 데이터 전송 요청
- **udt_send()**: 하위의 신뢰할 수 없는 채널로 패킷 전송
- **rdt_rcv()**: 채널에서 패킷 도착 시 수신 측 호출
- **deliver_data()**: 수신 측이 상위 계층으로 데이터 전달
- 가정: 패킷은 전송 순서대로 전달되되, 일부는 손실될 수 있음. 채널은 패킷 순서를 재배열(reorder)하지 않음

## 3.4.1 Building a Reliable Data Transfer Protocol (신뢰적 데이터 전송 프로토콜 구축)

### rdt1.0 — 완벽히 신뢰할 수 있는 채널 위의 RDT

- **가정**: 기저 채널(underlying channel)이 완전히 신뢰할 수 있음 → 비트 손상도, 패킷 손실도 없음
- **Sender FSM**: 상태 1개 (Wait for call from above) → rdt_send(data) → make_pkt(data) → udt_send(packet)
- **Receiver FSM**: 상태 1개 (Wait for call from below) → rdt_rcv(packet) → extract(data) → deliver_data(data)
- 피드백(ACK/NAK) 불필요. 흐름 제어 불필요. 매우 단순

### rdt2.0 — 비트 오류가 있는 채널 위의 RDT

- **가정**: 비트 오류(bit errors) 발생 가능. 패킷 손실은 없음
- **ARQ(Automatic Repeat reQuest) 프로토콜**: 비트 오류를 처리하는 세 가지 필수 기능 도입

**ARQ의 3가지 필수 기능**

- **1. 오류 검출(Error detection)**: 체크섬(checksum) 필드로 비트 오류 탐지
- **2. 수신자 피드백(Receiver feedback)**: ACK(긍정 확인 응답) = 올바르게 수신됨 / NAK(부정 확인 응답) = 오류 발생, 재전송 요청
- **3. 재전송(Retransmission)**: 오류 수신 시 송신자가 패킷 재전송

**rdt2.0 FSM (Figure 3.10)**

- **Sender 상태 2개**: 'Wait for call from above' → 패킷 생성/전송 → 'Wait for ACK or NAK'
- NAK 수신 시 재전송, ACK 수신 시 다음 데이터 대기로 복귀
- **Stop-and-wait 프로토콜**: 송신자는 ACK를 받을 때까지 새 데이터 전송 불가

**rdt2.0의 치명적 결함**

- **ACK/NAK 패킷 자체가 손상(corrupted)될 수 있음!** → 송신자가 수신자의 상태를 알 수 없게 됨

### rdt2.1 — 손상된 ACK/NAK 처리 추가

- **해결책**: 데이터 패킷에 순서 번호(sequence number) 추가 → 수신자가 재전송인지 새 패킷인지 판별
- **1비트 순서 번호(0 또는 1)** 사용: modulo-2 연산으로 0↔1 교대
- 같은 번호 = 재전송(duplicate), 다른 번호 = 새 패킷

**rdt2.1 FSM (Figures 3.11, 3.12)**

- **Sender 상태 4개**: Wait for call 0 / Wait for ACK or NAK 0 / Wait for call 1 / Wait for ACK or NAK 1
- 손상된 ACK 또는 NAK 수신 시 → 재전송
- **Receiver 상태 2개**: 0 기다림 / 1 기다림
- 올바른 순서 패킷: ACK + deliver / 손상 패킷: NAK / 이미 받은 번호(중복): ACK 전송

### rdt2.2 — NAK 없는 프로토콜

- **개선점**: NAK를 없애고, 마지막 올바르게 수신된 패킷의 ACK를 다시 전송하는 방식으로 대체
- **중복 ACK(duplicate ACK)**: 송신자가 같은 패킷에 대한 ACK를 두 번 받으면, 수신자가 그 다음 패킷을 올바르게 받지 못했음을 인식
- ACK 메시지에 확인하는 패킷의 순서 번호 포함 (예: ACK0, ACK1)

### rdt3.0 — 패킷 손실까지 처리 (alternating-bit protocol)

- **가정**: 비트 손상 + 패킷 손실(packet loss) 모두 발생 가능
- **새로운 문제**: 패킷 손실을 어떻게 감지할 것인가?

**해결책 — 카운트다운 타이머(Countdown Timer)**

- 적절한 타임아웃 값(timeout value) 설정 후, ACK 미수신 시 재전송
- **데이터 패킷 손실 시** → 타임아웃 후 재전송
- **ACK 손실 시** → 타임아웃 후 재전송 (중복 패킷 발생 가능, 순서번호로 처리)
- **패킷이 단순 지연(delay)된 경우** → 조기 타임아웃(premature timeout), 중복 전송 발생 → 순서번호로 처리

**송신자 타이머 동작 3가지**

- 1) 패킷 전송 시마다 타이머 시작(start)
- 2) 타임아웃 인터럽트 발생 시 재전송 수행
- 3) ACK 수신 시 타이머 정지(stop)

**동작 시나리오 (Figure 3.16)**

- (a) 손실 없는 경우: 정상 동작
- (b) 데이터 패킷 손실: 타임아웃 후 재전송
- (c) ACK 손실: 타임아웃 후 재전송, 수신자는 중복 패킷 감지 후 ACK 재전송
- (d) 조기 타임아웃: 중복 재전송, 수신자는 중복 감지 → 정상 처리

- **명칭**: 순서 번호가 0과 1을 교대(alternate)하므로 alternating-bit protocol이라고도 함

**핵심 메커니즘 요약**

- **체크섬(Checksum) + 순서 번호(Sequence number) + 타이머(Timer) + ACK/NAK** → 신뢰적 데이터 전송 가능

## 3.4.2 Pipelined Reliable Data Transfer Protocols (파이프라인 신뢰적 데이터 전송)

### rdt3.0의 성능 문제

- **Stop-and-wait 방식의 비효율**: RTT 동안 송신자가 대기 → 링크 이용률 극히 낮음
- **계산 예시**: RTT = 30ms, R = 1Gbps, L = 8,000 bits
- 전송 시간(d_trans) = L/R = 8,000 / 10^9 = 8 μs
- **송신자 이용률(U_sender) = (L/R) / (RTT + L/R) = 0.008 / 30.008 ≈ 0.00027 = 0.027%**
- 실효 처리율 = 약 267 kbps (1 Gbps 링크에서!)

### 파이프라이닝(Pipelining) 해결책

- **ACK를 기다리지 않고 여러 패킷을 연속 전송** → 이용률이 패킷 수에 비례하여 향상
- 파이프라이닝 도입 시 필요한 변화:
- **1. 순서 번호 범위 확대**: 동시에 여러 미확인 패킷이 있으므로 각 패킷에 고유 번호 필요
- **2. 버퍼링(Buffering)**: 송신자는 미확인 패킷을, 수신자는 올바르게 받은 패킷을 버퍼에 저장
- 3. 오류 복구 방식에 따라 요구 사항 달라짐 → Go-Back-N 또는 Selective Repeat

## 3.4.3 Go-Back-N (GBN)

### 기본 개념

- **송신자는 ACK를 기다리지 않고 최대 N개의 미확인 패킷을 파이프라인에 유지 가능**
- **N = 윈도우 크기(window size)** → 슬라이딩 윈도우 프로토콜(sliding-window protocol)이라고도 함

### 순서 번호 공간 (Figure 3.19)

- **base**: 가장 오래된 미확인 패킷의 순서 번호
- **nextseqnum**: 다음에 전송할 패킷의 순서 번호
- **[0, base-1]**: 전송 + ACK 완료
- **[base, nextseqnum-1]**: 전송했으나 아직 미확인
- **[nextseqnum, base+N-1]**: 즉시 전송 가능
- **[base+N, ...]**: 사용 불가 (윈도우 밖)
- 순서 번호 필드 k비트 → 범위 [0, 2^k - 1], modulo 2^k 연산

### GBN Sender 동작 (Figure 3.20)

- **1. 상위 계층 호출(rdt_send)**: 윈도우 미충족 시 패킷 생성·전송, nextseqnum++. 윈도우 가득 참 → refuse_data()
- **2. ACK 수신**: 누적 확인 응답(cumulative acknowledgment) — ACK n = 순서 번호 n 이하 모든 패킷 올바르게 수신 의미. base = getacknum(rcvpkt) + 1
- **3. 타임아웃**: base부터 nextseqnum-1까지 모든 미확인 패킷 재전송 (Go-Back-N 이름의 유래)
- **단 하나의 타이머만 사용** (가장 오래된 미확인 패킷용)

### GBN Receiver 동작 (Figure 3.21)

- **순서에 맞는(in-order) 패킷**: ACK 전송 + deliver_data() + expectedseqnum++
- **순서 어긋난(out-of-order) 패킷**: 폐기(discard) + 마지막 올바르게 받은 패킷의 ACK 재전송
- 수신자 버퍼링 불필요 → 구현 단순. 단, 재전송 필요량 증가 가능

### GBN 동작 예시 (Figure 3.22, window size = 4)

- pkt0~3 전송 → pkt2 손실
- pkt3, 4, 5는 순서 어긋나 수신자가 모두 폐기, ACK1 반복 전송
- pkt2 타임아웃 → pkt2, 3, 4, 5 모두 재전송

## 3.4.4 Selective Repeat (SR, 선택적 반복)

### GBN의 문제점과 SR의 필요성

- **GBN 문제**: 윈도우 크기와 bandwidth-delay product가 클 때, 단 하나의 패킷 오류로 대량의 불필요한 재전송 발생
- **SR 해결책**: 오류가 발생한 패킷만 선택적으로 재전송 → 불필요한 재전송 방지

### SR Sender 동작

- **1. 상위 계층 데이터 수신**: 윈도우 범위 내면 전송, 아니면 버퍼링 or 반환
- **2. 타임아웃**: 각 패킷마다 개별 논리 타이머(logical timer) 사용. 타임아웃 시 해당 패킷만 재전송
- **3. ACK 수신**: 해당 패킷 수신 완료 마킹. send_base와 같으면 윈도우를 가장 작은 미확인 패킷으로 이동

### SR Receiver 동작

- **[rcv_base, rcv_base+N-1] 범위 패킷 수신**: 개별 selective ACK 전송. 처음 수신이면 버퍼링. rcv_base와 같으면 → 연속 버퍼 패킷 모두 일괄 deliver + 윈도우 전진
- **[rcv_base-N, rcv_base-1] 범위 패킷 수신**: 이미 ACK된 패킷이라도 ACK 재전송 (sender의 ACK 미수신 시 윈도우가 진행 못하므로)
- **그 외 범위 패킷**: 무시

### SR 동작 예시 (Figure 3.26, window size = 4)

- pkt0~3 전송 → pkt2 손실
- pkt3, 4, 5 수신 시 버퍼링 + ACK3, ACK4, ACK5 개별 전송
- pkt2 타임아웃 → pkt2만 재전송
- pkt2 수신 → pkt2, 3, 4, 5 일괄 deliver

### SR 윈도우 크기 제약 (Figure 3.27)

- **문제**: sender 윈도우와 receiver 윈도우가 항상 일치하지 않음 → 유한 순서 번호에서 새 패킷과 재전송 패킷 혼동 가능
- **결론**: SR에서 윈도우 크기 ≤ 순서 번호 공간 크기의 절반 이하여야 함

### GBN vs SR 비교

- **GBN**: 누적 ACK, 단일 타이머, 순서 어긋난 패킷 폐기, 타임아웃 시 전체 재전송
- **SR**: 개별 ACK, 개별 타이머, 순서 어긋난 패킷 버퍼링, 타임아웃 시 해당 패킷만 재전송

### 핵심 메커니즘 종합 (Table 3.1)

- **체크섬(Checksum)**: 전송된 패킷의 비트 오류 검출
- **타이머(Timer)**: 패킷/ACK 손실 시 타임아웃 후 재전송
- **순서 번호(Sequence number)**: 데이터 패킷에 번호 부여. 손실 및 중복 감지
- **확인 응답(ACK)**: 수신자→송신자: 패킷 올바르게 수신 알림. 개별 또는 누적
- **부정 확인 응답(NAK)**: 수신자→송신자: 패킷 오류 수신 알림
- **윈도우/파이프라이닝(Window/Pipelining)**: 여러 패킷 동시 전송으로 이용률 향상. N으로 미확인 패킷 수 제한

### 프로토콜 진화 요약

- **rdt1.0**: (완벽한 채널) → 단순 전송
- **rdt2.0**: 비트 오류 → 체크섬 + ACK/NAK
- **rdt2.1**: ACK/NAK 손상 → 1비트 순서 번호 추가
- **rdt2.2**: NAK 제거 → 중복 ACK로 대체
- **rdt3.0**: 패킷 손실 → 타이머 + 타임아웃 재전송 (alternating-bit)
- **GBN**: stop-and-wait 비효율 → N-윈도우 파이프라인, 누적 ACK, 전체 재전송
- **SR**: GBN의 불필요 재전송 → 개별 ACK, 개별 타이머, 선택적 재전송

# 3.5 Connection-Oriented Transport: TCP (연결 지향 전송: TCP)

## 3.5.1 The TCP Connection (TCP 연결)

### TCP의 핵심 특징

- **연결 지향(connection-oriented)**: 데이터 전송 전에 두 프로세스가 반드시 핸드셰이크(handshake) 수행
- **전이중(full-duplex)**: 양방향 동시 데이터 전송 가능
- **점대점(point-to-point)**: 항상 단일 송신자 ↔ 단일 수신자. 멀티캐스트 불가
- **연결 상태는 오직 종단 시스템(end systems)에만 존재**. 중간 라우터/스위치는 TCP 연결을 전혀 모름

### 버퍼와 MSS

- TCP는 데이터를 송신 버퍼(send buffer)에 저장 후 네트워크 계층으로 전달
- **MSS(Maximum Segment Size, 최대 세그먼트 크기)**: 세그먼트의 애플리케이션 데이터 필드 최대 크기 (헤더 제외!)
- **MTU(Maximum Transmission Unit, 최대 전송 단위)**: 링크 계층 프레임의 최대 크기
- Ethernet/PPP의 MTU = 1,500 바이트, TCP/IP 헤더 = 보통 40 바이트 → MSS 전형값 = 1,460 바이트

## 3.5.2 TCP Segment Structure (TCP 세그먼트 구조)

### TCP 세그먼트 헤더 필드 (Figure 3.29)

- **출발지 포트 번호(Source port #)** — 16비트: 다중화/역다중화용
- **목적지 포트 번호(Destination port #)** — 16비트: 다중화/역다중화용
- **순서 번호(Sequence number)** — 32비트: 신뢰적 데이터 전송용
- **확인 응답 번호(Acknowledgment number)** — 32비트: 신뢰적 데이터 전송용
- **헤더 길이(Header length)** — 4비트: 32비트 워드 단위로 헤더 길이 표시 (옵션 필드로 가변)
- **수신 윈도우(Receive window)** — 16비트: 흐름 제어용. 수신자가 받을 수 있는 바이트 수
- **인터넷 체크섬(Internet checksum)** — 16비트: 오류 검출
- **긴급 데이터 포인터(Urgent data pointer)** — 16비트: URG 비트와 함께 사용 (실제로 거의 안 씀)
- **옵션(Options)** — 가변: MSS 협상, 윈도우 스케일링, 타임스탬프 등

### 6개 플래그 비트

- **ACK**: 확인 응답 번호 필드가 유효함을 표시
- **RST**: 연결 리셋 (포트 불일치 시 리셋 세그먼트 전송)
- **SYN**: 연결 설정 시 사용 (SYN 세그먼트)
- **FIN**: 연결 종료 시 사용
- **CWR, ECE**: 명시적 혼잡 알림(ECN)에서 사용
- **PSH**: 수신자가 데이터를 즉시 상위 계층에 전달 (거의 안 씀)

### 순서 번호 (Sequence Numbers)

- TCP는 데이터를 순서 있는 바이트 스트림(ordered byte stream)으로 봄
- **순서 번호** = 세그먼트 데이터 필드의 첫 번째 바이트의 바이트 스트림 번호
- 예: 500,000바이트 파일, MSS=1,000 → 첫 세그먼트 순서번호=0, 두 번째=1,000, 세 번째=2,000...
- **초기 순서 번호(ISN, Initial Sequence Number)**: 양쪽 모두 랜덤으로 선택 (이전 연결 세그먼트와 혼동 방지)

### 확인 응답 번호 (Acknowledgment Numbers)

- **호스트 A의 ACK 번호** = A가 B로부터 기대하는 다음 바이트의 번호
- 예: A가 B로부터 0~535 수신 완료 → ACK 번호 = 536
- **누적 확인 응답(cumulative acknowledgments)**: 첫 번째 누락 바이트 이전까지만 확인 응답
- **순서 어긋난 세그먼트(out-of-order)**: RFC 규정 없음 → 실제로는 버퍼링이 일반적

### Telnet 예시 (Figure 3.31)

- 클라이언트 ISN=42, 서버 ISN=79, 문자 'C' 전송
- 1) 클라이언트→서버: Seq=42, ACK=79, data='C'
- 2) 서버→클라이언트: Seq=79, ACK=43, data='C' (에코 백 + 피기배킹 ACK)
- 3) 클라이언트→서버: Seq=43, ACK=80 (에코에 대한 ACK)
- **피기배킹(piggybacking)**: ACK를 데이터 세그먼트에 함께 실어 보내는 것

## 3.5.3 Round-Trip Time Estimation and Timeout (왕복 시간 추정과 타임아웃)

### SampleRTT

- 세그먼트가 전송된 시점부터 ACK 수신까지의 시간
- **재전송된 세그먼트에 대해서는 측정하지 않음** (Karn 1987)
- 한 번에 하나의 세그먼트에 대해서만 측정 (대략 RTT마다 새 값 하나)

### EstimatedRTT (지수 가중 이동 평균, EWMA)

EstimatedRTT = (1 − α) × EstimatedRTT + α × SampleRTT

- **권장값: α = 0.125 (= 1/8)**
- 최신 샘플에 더 큰 가중치 → 현재 네트워크 혼잡 상태 반영
- **EWMA(Exponential Weighted Moving Average)**: 오래된 샘플의 가중치가 지수적으로 감소

### DevRTT (RTT 변동성 추정)

DevRTT = (1 − β) × DevRTT + β × |SampleRTT − EstimatedRTT|

- **권장값: β = 0.25**
- SampleRTT 변동이 작으면 DevRTT 작음, 변동이 크면 DevRTT 큼

### TimeoutInterval (재전송 타임아웃 구간)

TimeoutInterval = EstimatedRTT + 4 × DevRTT

- **초기값: 1초 권장 (RFC 6298)**
- **타임아웃 발생 시: TimeoutInterval을 2배로 증가** (지수적 백오프, 혼잡 제어의 일종)
- ACK 수신 후 EstimatedRTT 업데이트 시 위 공식으로 재계산

## 3.5.4 Reliable Data Transfer (신뢰적 데이터 전송)

### 기본 원리

- IP는 비신뢰적(unreliable) 서비스 → TCP가 그 위에 신뢰적 데이터 전송 서비스 구축
- **보장 사항: 손상 없음, 갭 없음, 중복 없음, 순서 보장**
- **단일 재전송 타이머**: 여러 미확인 세그먼트가 있어도 타이머는 하나만 사용 (RFC 6298)

### TCP 송신자의 3가지 주요 이벤트 (Figure 3.33)

**1. 애플리케이션으로부터 데이터 수신**

- TCP 세그먼트 생성 (순서 번호 = NextSeqNum)
- 타이머가 미실행 중이면 타이머 시작 → IP로 세그먼트 전달 → NextSeqNum += length(data)

**2. 타이머 타임아웃**

- 가장 작은 순서 번호의 미확인 세그먼트 재전송 + 타이머 재시작

**3. ACK 수신 (ACK 값 = y)**

- y > SendBase이면: SendBase = y (누적 확인 응답)
- 아직 미확인 세그먼트 있으면 타이머 재시작

### 타임아웃 구간 두 배 증가 (Doubling the Timeout Interval)

- 타임아웃 발생 시마다 TimeoutInterval을 2배로 설정 (예: 0.75초 → 1.5초 → 3.0초)
- 다른 이벤트(데이터/ACK 수신)로 타이머 시작 시에는 공식으로 재계산

### 빠른 재전송 (Fast Retransmit)

- **문제**: 타임아웃까지 기다리면 지연이 큼
- **해결**: 중복 ACK(duplicate ACK)으로 세그먼트 손실 조기 감지
- 수신자가 순서 번호가 예상보다 큰 세그먼트 수신 시 → 갭 감지 → 마지막 정상 수신 바이트에 대한 중복 ACK 즉시 전송
- **3개의 중복 ACK 수신 시**: 빠른 재전송(fast retransmit) 수행 → 타이머 만료 전에 누락 세그먼트 재전송 (RFC 5681)

### TCP ACK 생성 규칙 (Table 3.2, RFC 5681)

- **예상 순서 번호 도착 + 이미 모두 ACK됨** → 지연 ACK: 최대 500ms 대기 후 전송
- **예상 순서 번호 도착 + 대기 중인 ACK 있음** → 즉시 단일 누적 ACK 전송
- **예상보다 높은 순서 번호 도착 (갭 감지)** → 즉시 중복 ACK 전송
- **갭을 채우는 세그먼트 도착** → 즉시 ACK 전송

### GBN vs SR: TCP의 위치

- TCP ACK는 누적(cumulative) → GBN과 유사
- 그러나 TCP는 순서 어긋난 세그먼트를 버퍼링 → GBN과 다름
- **SACK(Selective Acknowledgment, RFC 2018)**: 순서 어긋난 세그먼트를 선택적으로 ACK 가능한 TCP 확장
- **결론: TCP 오류 복구 메커니즘은 GBN과 SR의 혼합(hybrid)**

## 3.5.5 Flow Control (흐름 제어)

### 목적

- **송신자가 수신자의 버퍼를 넘치게(overflow) 하는 것을 방지** → 속도 매칭(speed-matching) 서비스
- **흐름 제어(flow control) vs 혼잡 제어(congestion control)**: 전자는 수신자 버퍼 보호, 후자는 네트워크 내 혼잡 방지. 원인이 다름

### 수신 윈도우 (rwnd, receive window)

- **RcvBuffer**: 수신 버퍼 총 크기
- **LastByteRead**: 애플리케이션이 버퍼에서 읽은 마지막 바이트 번호
- **LastByteRcvd**: 네트워크에서 도착하여 버퍼에 저장된 마지막 바이트 번호

**수신 측 제약 조건**

LastByteRcvd − LastByteRead ≤ RcvBuffer

**rwnd 공식**

rwnd = RcvBuffer − [LastByteRcvd − LastByteRead]

- rwnd는 동적으로 변하며, Host B는 매 세그먼트의 수신 윈도우 필드에 현재 rwnd 값을 포함하여 전송

**송신 측 제약 (Host A)**

LastByteSent − LastByteAcked ≤ rwnd

### rwnd = 0 문제 및 해결

- **문제**: 수신 버퍼가 꽉 차면 rwnd = 0 → Host A가 전송 중단 → 버퍼가 비어도 Host A가 알 방법 없음 (교착 상태)
- **해결**: rwnd = 0이어도 Host A는 1바이트 데이터 세그먼트를 계속 전송 → 수신자가 ACK로 응답 → 버퍼 공간 생기면 비영 rwnd 값 통보

## 3.5.6 TCP Connection Management (TCP 연결 관리)

### 연결 설정: 3-방향 핸드셰이크 (Three-Way Handshake, Figure 3.39)

**Step 1 — SYN 세그먼트 (클라이언트 → 서버)**

- SYN 비트 = 1, 애플리케이션 데이터 없음
- 클라이언트가 초기 순서 번호(client_isn) 랜덤 선택 → 순서 번호 필드에 삽입

**Step 2 — SYNACK 세그먼트 (서버 → 클라이언트)**

- SYN 비트 = 1, 확인 응답 번호 = client_isn + 1
- 서버가 초기 순서 번호(server_isn) 랜덤 선택 → 순서 번호 필드에 삽입
- 서버는 이 시점에 TCP 버퍼와 변수를 할당

**Step 3 — ACK 세그먼트 (클라이언트 → 서버)**

- SYN 비트 = 0 (연결 수립됨), 확인 응답 번호 = server_isn + 1
- 클라이언트도 버퍼와 변수 할당. 이 세그먼트부터 데이터 페이로드 포함 가능

### 연결 종료 (Connection Teardown, Figure 3.40)

- 1) 클라이언트: FIN 비트 = 1 세그먼트 전송
- 2) 서버: ACK 전송
- 3) 서버: 자신의 FIN 비트 = 1 세그먼트 전송
- 4) 클라이언트: ACK 전송 → 양측 자원 해제

### 클라이언트 TCP 상태 전이 (Figure 3.41)

- **CLOSED** → (연결 시작) SYN 전송 → **SYN_SENT**
- **SYN_SENT** → (SYNACK 수신) ACK 전송 → **ESTABLISHED**
- **ESTABLISHED** → (종료 시작) FIN 전송 → **FIN_WAIT_1**
- **FIN_WAIT_1** → (ACK 수신) → **FIN_WAIT_2**
- **FIN_WAIT_2** → (서버 FIN 수신) ACK 전송 → **TIME_WAIT**
- **TIME_WAIT** → (30초~2분 대기) → **CLOSED**

### 서버 TCP 상태 전이 (Figure 3.42)

- **CLOSED** → (listen 소켓 생성) → **LISTEN**
- **LISTEN** → (SYN 수신) SYN+ACK 전송 → **SYN_RCVD**
- **SYN_RCVD** → (ACK 수신) → **ESTABLISHED**
- **ESTABLISHED** → (클라이언트 FIN 수신) ACK 전송 → **CLOSE_WAIT**
- **CLOSE_WAIT** → (FIN 전송) → **LAST_ACK**
- **LAST_ACK** → (ACK 수신) → **CLOSED**

### TIME_WAIT 상태

- **목적**: 최종 ACK가 손실된 경우 재전송 가능하도록 대기
- 전형적 대기 시간: 30초, 1분, 또는 2분 (구현에 따라 다름)
- TIME_WAIT 종료 후 포트 번호를 포함한 모든 자원 해제

### SYN 플러드 공격 (SYN Flood Attack)

- **공격 방식**: 공격자가 3-방향 핸드셰이크 3단계를 완료하지 않고 대량의 SYN 세그먼트 전송 → 서버의 연결 자원이 반열린 연결(half-open connection)로 고갈 → DoS 공격
- **방어: SYN 쿠키(SYN cookies, RFC 4987)**
- 서버가 SYN 수신 시 버퍼/변수를 즉시 할당하지 않음
- 해시 함수로 초기 순서 번호(쿠키) 생성하여 SYNACK 전송
- 정상 클라이언트의 ACK가 오면 쿠키 검증 후 연결 완성

### 포트 불일치 시 동작

- **TCP**: 목적지 포트에 해당하는 소켓이 없으면 RST 비트 = 1 세그먼트(리셋 세그먼트) 전송
- **UDP**: 포트 불일치 시 ICMP 데이터그램 전송 (TCP와 다름)