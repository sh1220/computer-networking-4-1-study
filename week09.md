## TCP
### **Overview**

- **Point-to-point**: sender 1 ↔ receiver 1 (1:1 통신)
- **Reliable, in-order byte stream**
    - 신뢰성 O & 순서 보장
    - 데이터를 하나의 연속된 byte stream으로 취급
    - Message Boundary가 없음 (UDP와 다른 점)
        - 어떤 단위로 send() 했는지 구분 정보를 보존 X
        - ***하나의 send() 데이터가 여러 segment로 쪼개질 수 있음***
- **Full duplex**
    - 양방향 데이터 흐름이 같은 연결 위에서 동시에
        - MSS (Maximum Segment Size): 최대 세그먼트 크기
- **Cumulative ACKs**: 누적 확인응답
- **Pipelining**
    - TCP flow / congestion control이 window size 결정
- **Connection-oriented**
    - 데이터 교환 전 handshake로 상태 초기화
- **Flow controlled**
    - sender가 receiver를 압도하지 않도록 제어

### **Segment Structure**

<img width="1464" height="698" alt="Image" src="https://github.com/user-attachments/assets/243a893d-9f65-4307-b7c2-e8f7b545916e" />


| 필드 | 어떤 서비스를 위해 존재하는가 |
| --- | --- |
| **source/dest port #** | **demultiplexing** (T4핵심 기능) |
| **sequence number** | **reliable, in-order delivery:** byte stream에서 이 segment의 첫 byte 번호 |
| **acknowledgement number** | **reliable delivery:** 다음에 받을 것으로 기대하는 byte 번호 **(cumulative ACK)** |
| **receive window** | **flow control:** receiver가 받을 수 있는 byte 수 |
| **checksum** | **error detection** |
| **C, E (ECN bits)** | **congestion control** (네트워크 계층과 협력) |
| **RST, SYN, FIN** | **connection management** (3-way handshake, 연결 종료) |
| **head length**  | options 길이가 가변이라 헤더 길이 명시 필요 |

#### Sequence Number / ACK Number

```bash
Host A의 바이트 스트림: 0 1 2 3 4 5 6 7 8 9 10 ...

A가 보내는 세그먼트 1: data = bytes [0~99]   → seq = 0
A가 보내는 세그먼트 2: data = bytes [100~199] → seq = 100
A가 보내는 세그먼트 3: data = bytes [200~299] → seq = 200

B가 세그먼트 1, 2를 정상 수신했다면:
B의 ACK = 200  (다음에 받을 byte 번호 = "expected next byte")
```

- seq #는 **segment 번호가 아니라 byte 번호**
- ACK #는 **누적(cumulative)**

<img width="856" height="986" alt="Image" src="https://github.com/user-attachments/assets/7d25db0f-d8ca-4808-b618-f4301f33834e" />

- 그림은 A sequence 공간
- sender A → receiver B의 흐름 공간에서
    - sender가 보내는 segment
        - 파랑 영역에서 segment 만들어서 보낼 차례임
        - → 파랑 첫번째 byte가 sequence number
    - receiver가 보내는 segment
        - 노랑 영역에 대한 ACK 보낼 차례임
        - → 노랑 마지막 byte+1 이 ACK number
            - [ACK] = 다음으로 기대하는 byte
- B → A 흐름 공간은 다른 차원
    - sender가 보내는 segment의 ACK
        - A가 다음에 받기를 기대하는 byte 번호
        - B의 sequence 공간에서의 번호임
    - receiver가 보내는 segment의 sequence number
        - B 자신이 보내는 데이터의 첫 byte 번호
        - 이 또한 B의 sequence 공간에서의 번호임
        - A sequence 공간과 무관
- **Sequence 공간**
    
    ```bash
    A → B 방향 시퀀스 공간:
    바이트 번호: 1001  1002  1003  1004  1005  1006  ...
    실제 데이터: 'H'   'e'   'l'   'l'   'o'   ' '   ...
                ↑
                A가 첫 byte를 1001부터 시작하기로 정함 (ISN)
    ```
    
    - 연결 시작부터 끝까지 흐를 모든 바이트에 미리 번호를 매긴 것
    - 각 바이트마다 고유 번호가 매겨진 무한한 줄
    - Buffer Index ↔ Sequence 공간
        - 시작점 0 ↔ ISN (임의의 값)
        - buffer는 유한 ↔ sequence 공간은 (사실상) 무한
            - 2^32개 byte번호가 펼쳐진 거대한 공간
                
                ```bash
                실제 송신 버퍼 (예: 8 슬롯짜리 circular buffer):
                [H][e][l][l][o][ ][W][o]   ← 가득 차면 ACK된 부분만 비우고 재사용
                 ↑
                 ACK 받으면 이 공간 재활용
                
                시퀀스 공간 (개념적):
                1001 1002 1003 1004 ... 9999 10000 10001 ... → 계속 증가
                'H'  'e'  'l'  'l'      앞으로 보낼 모든 바이트의 번호가 미리 정해져 있음
                ```
                

### RTT (Round Trip Time)

- **Overview**
    - TCP는 reliable transport를 위해 **타이머 기반 재전송**
        - segment를 보내고 일정 시간 안에 ACK가 안 오면 loss로 판단하고 재전송
    - **일정 시간이 곧 timeout**
        - 그 값을 어떻게 정할지가 RTT 추정의 목적
        
        ```bash
        RTT = (A→B 전파시간) + (B의 ACK 처리시간) + (B→A 전파시간)
            = A가 ACK 받은 시각 − A가 세그먼트 보낸 시각
            = t2 − t1   (둘 다 A의 시계 기준)
        ```
        
    - **Trade - Off**
        
        
        | Timeout | 문제 |
        | --- | --- |
        | **너무 짧으면** | premature timeout → 불필요한 재전송 (네트워크 자원 낭비) |
        | **너무 길면** | 손실 감지 지연 → 처리량 저하 |
    - **RTT 값 자체가 변동**
        - 네트워크 혼잡도 변화 → Queueing Delay 변동
        - Routing 경로 변경 가능성
        - 송수신 Host의 처리 부하 변화
        
        ⇒ **RTT를 추정(estimate)** 해야 함 
        
- **RTT 계산**
    - **SampleRTT**
        - 세그먼트 전송 시점부터 ACK 수신까지의 측정 시간
            - 재전송된 세그먼트는 SampleRTT 측정에서 제외
        - SampleRTT는 매번 측정마다 들쭉날쭉함
            
             → **여러 최근 샘플을 평균**해서 부드러운 추정치를 만들어야 함
            
    - **EstimatedRTT**
        
        ```bash
        EstimatedRTT = (1 - α) × EstimatedRTT + α × SampleRTT
        ```
        
        - α (보통 **0.125** = 1/8): 새 샘플의 가중치
        - (1−α) = 0.875: 과거 추정치의 가중치
        - 각 과거 샘플의 가중치
            
            
            | 샘플 | 가중치 (α=0.125일 때) |
            | --- | --- |
            | 가장 최근 | 0.125 |
            | 1단계 전 | 0.109 |
            | 2단계 전 | 0.096 |
            | 3단계 전 | 0.084 |
            | ... | (지수적으로 감소) |
    - **TimeoutInterval**
        
        ```bash
        TimeoutInterval = EstimatedRTT + 4 × DevRTT
                          ↑                ↑
                          추정치          안전 마진(safety margin)
        ```
        
        - EstimatedRTT를 그대로 timeout으로 쓰는 경우
            - **약간의 RTT 변동**에도 **premature timeout** 발생 가능
            - 그래서 여유분(safety margin)을 더해야 함
        - **DevRTT**
            - 안전 마진을 얼마나 줄까
            - 고정값이 아니라 RTT 변동성에 따라 **적응적으로** 정해짐
                - RTT가 안정적 → 작은 마진으로 충분
                - RTT 변동 큼 → 큰 마진 필요
- **Mechanism**
    - **TCP sender → 이벤트 기반(event-driven)**
        - **event 1) app으로부터 data receive**
            
            ```bash
            앱 ──(write/send)──→ TCP 송신 버퍼
                                     ↓
                                 세그먼트 생성 후 전송
            ```
            
            - **세그먼트 생성** (with seq #)
                - seq # = 이 세그먼트 데이터의 첫 바이트의 byte-stream number
                - ex) 지금까지 0~99까지 보냈으면 다음 세그먼트의 seq # = 100
            - 타이머가 안 돌고 있으면 시작
                - 타이머는 **가장 오래된 unACKed segment** 기준으로 작동
                - 모든 세그먼트마다 타이머가 따로 있는 게 아니라 **단 하나의 타이머**만 존재
                - 만료 시간(expiration interval) = **TimeOutInterval**
                    
                    ```bash
                    EstimatedRTT + 4×DevRTT
                    ```
                    
        - **event 2) Timeout 발생**
            - 타이머 만료
            - 가장 오래된 unACKed 세그먼트의 ACK가 너무 안 옴 → Loss 의심
            - 할 일
                - 타임아웃을 일으킨 세그먼트만 재전송 **(그 하나만 ↔ GBN)**
                - 타이머 재시작
        - **event 3) ACK receive**
            - 받은 ACK이 의미 있는 ACK인 경우 = 새로운 데이터를 ACK하는 경우
                - ACK된 범위 갱신
                    - sendBase 라고 부름
                    - "여기까지는 안전하게 도착했다"는 정보 업데이트
                - 여전히 **unACKed** 세그먼트가 남았으면 **타이머 재시작**
                    - 새로운 "가장 오래된 unACKed 세그먼트" 기준으로 타이머 새로 시작
                    - 모두 ACK 됐으면 타이머 정지
            
            ```python
            A가 보낸 것: seq=0(100B), seq=100(100B), seq=200(100B)
                        (각각 100바이트)
            
            상황: B가 첫 두 개만 받았고 ACK=200을 보냄
            A가 ACK=200 수신:
              - "0~199 다 받았구나" 인지
              - sendBase = 200으로 갱신
              - seq=200 세그먼트는 아직 unACKed → 타이머 재시작
            ```
            
- **Fast Retransmit**
    - [문제]
        
        Timeout이 너무 느림
        
        ```python
        TimeoutInterval = EstimatedRTT + 4×DevRTT
        ```
        
        - 더 빨리 손실을 감지할 수 있다면? → fast retransmit
    - [핵심 아이디어]
        - **Duplicate ACK**가 손실의 신호
        - receiver(B)는 **순서가 안 맞는(out-of-order) segment**를 받으면 즉시 **내가 기대하던 그 byte 번호** 를 다시 ACK 함
        - **cumulative ACK**의 동작 방식과 결합되어 sender에게 손실 정보를 주는
        
        ```python
        A                                             B
        │                                             │
        ├─ Seq=92, 8 bytes ──────────→                │
        ├─ Seq=100, 20 bytes ────X (손실!)            │
        │                                             │
        ├─ Seq=120, ... ──────────────→               │  
        ├─ Seq=140, ... ──────────────→               │
        ├─ Seq=160, ... ──────────────→               │
        │                                             │
        │                              ← ACK=100 ──── │   ← Seq=92 잘 받음, "다음은 100 줘"
        │                                             │
        │                              ← ACK=100 ──── │  ← Seq=120 받았는데 100이 비어있음 "100 줘"
        │                              ← ACK=100 ──── │  ← Seq=140도 마찬가지
        │                              ← ACK=100 ──── │  ← Seq=160도 마찬가지
        │                                             │
        │   ↑ 동일 ACK 4개 누적 (1 + 3 duplicate)      │
        │   ↓                                         │
        ├─ Seq=100, 20 bytes 재전송 ──────→ (timeout 안 기다림)
        ```
        
        - 왜 동일 ACK가 반복될까
        - B의 입장에서
            1. Seq=92(8B) 받음 → "다음 기대 = 100" → ACK=100 전송
            2. Seq=100 손실 → 받지 못함
            3. Seq=120 받음 → 순서 어긋남
                
                여전히 100을 못 받았으니 "다음 기대 = 100" → **ACK=100 또 전송**
                
            4. Seq=140 받음 → 마찬가지로 ACK=100
    - **Triple Duplicate ACK**
        - 동일한 ACK가 3번 추가로 도착하면(= 총 4번 같은 ACK) 즉시 재전송
        - 1~2번 정도는 단순한 **out-of-order** (순서 뒤바뀜) 일 수 있음
            - 패킷이 다른 경로로 가서 늦게 도착한 경우
        - 3번이면 거의 확실히 손실
        - 너무 자주 fast retransmit하면 멀쩡한 패킷도 재전송하는 낭비

### Flow Control

- **Overview**
    
    ```python
    [송신자 A] → 네트워크 → [B의 IP] → [B의 TCP] → [B의 수신 버퍼]   →    [B의 앱]
                                                       ↑                  ↑
                                               데이터가 쌓이는 곳   데이터를 꺼내가는 곳
    ```
    
    - **데이터가 들어오는 속도 > 앱이 꺼내가는 속도**
    - 수신 버퍼가 가득 차서 **overflow** 발생
    - 수신 측 앱이 데이터를 못 꺼내가는 상황
        - 앱이 다른 작업 중이라 `recv()` 호출이 늦음
        - 앱 처리가 느린데 네트워크가 빠름 (ex 고속 다운로드 중 디스크 I/O 병목)
        - CPU가 다른 일로 바쁨
    - TCP 수신 버퍼는 한정된 메모리(ex 4096 bytes 기본)
        - 가득 차면 들어오는 segment를 **버려야** 함
- **동작 원리**
    
    [목표] 수신자가 송신자의 전송 속도를 제어해서 수신 버퍼가 overflow되지 않도록 한다
    
    - rwnd로 여유 공간 알리기
        
        ```bash
        rwnd = RcvBuffer − (도착했지만 아직 앱이 안 꺼낸 데이터 양)
             = RcvBuffer − [LastByteRcvd − LastByteRead]
        ```
        
        - 앞으로 받을 수 있는 여유 공간(byte 수)
        - receiver(B)는 **자신의 수신 버퍼에 남은 공간(free buffer space)**을 매 ACK마다 송신자에게 알려줌
        - 이 값이 헤더의 **rwnd (receive window)** 필드
            
            ```bash
            [B의 수신 버퍼 구조]
            
            ┌─────────────────┐  ← RcvBuffer 전체 크기 (예: 4096 bytes)
            │                 │
            │  buffered data  │  ← 도착했지만 앱이 아직 안 꺼낸 데이터
            │                 │
            ├─────────────────┤
            │                 │
            │  free buffer    │  ← 남은 공간
            │    space        │  ← 이게 rwnd
            │                 │
            └─────────────────┘
            ```
            
    - Sender의 행동
        - sender(A)는 받은 rwnd 값에 따라 자신의 전송을 제한
            
            ```python
            LastByteSent − LastByteAcked ≤ rwnd
               ↑
               "지금 in-flight인 데이터 양"
            ```
            
            - 아직 ACK 안 받은 데이터(in-flight) 양이 rwnd를 넘지 않도록
            - B의 버퍼 여유 공간만큼만 in-flight 데이터를 둠
            - → **overflow 방지 보장**
