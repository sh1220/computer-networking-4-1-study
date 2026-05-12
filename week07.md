### **Multiplexing / Demultiplexing**

- **Transport layer (L4)** 의 핵심 기능 중 하나
- 여러 apps ( → sockets) 들이 하나의 network 연결을 같이 쓰기 위한 mechanism
- 필요한 이유
    - computer 1대 - 동시에 여러 network programs
        - chrome: youtube
        - 카톡 채팅
        - spotify
    - **NIC (Network Interface Card) 는 1개**
    - **IP address도 host 당 1개**
        - **물리 서버** 입장에서는 자기 interface에 붙은 IP가 정체성
        - Domain Name “Service”의 여러 IP와 별개
    
    *⇒ 들어오는 packet을 **누구한테 줘야 하는지** 구분할 방법이 필요함*
    
- **Multiplexing**
    - `Receiver` Header에 ***어느 소켓에서 온*** 건지 정보를 박아넣는 것
    - 여러 socket에서 나온 데이터들을 하나의 통로로 합치는 작업
        - chrome, 카톡, spotify가 각각 자기 socket에서 데이터를 보냄
        - L4에서 각 데이터에 **Transport Header**를 붙임
            - Source Port / Dest Port …
        - 이걸 L3 (IP layer)로 내려보내서 network로 송출
- **Demultiplexing**
    - `Sender`
    - 네트워크에서 들어온 segment를 ***올바른 소켓에 배달*** 하는 작업
        - L3 (IP layer) 에서 segment가 올라옴
        - L4 (Transport layer)에서 Header의 Port 번호를 확인
        - 해당 Port에 묶인 socket으로 데이터를 전달
            - sender OS는 “packet의 `Destination Port == 443`” → chrome socket으로
- **Connectionless vs Connection-oriented Demux**
    - **UDP : 2-tuple Demux**
        
        ```c
        (Destination IP, Destination Port)
        ```
        
        - UDP socket 식별 → 목적지에 대한 정도만 존재
        - 누가 보냈는지는 신경 쓰지 않는다
        - 시나리오
            - 서버가 포트 9157에 UDP 소켓을 열어놓음
                - 클라이언트 A (IP: 1.1.1.1, 포트: 5000) → 서버:9157로 packet전송
                - 클라이언트 B (IP: 2.2.2.2, 포트: 6000) → 서버:9157로 packet 전송
            
            → 두 packet 모두 똑같은 socket으로 들어감 
            
    - **TCP : 4-tuple Demux**
        
        ```c
        (Soruce IP, Source Port, Destination IP, Destination Port)
        ```
        
        - 네 가지가 하나라도 다르면 다른 socket
        - 시나리오
            - 웹서버가 포트 80에서 listening 중이라고 하자
                
                ```bash
                - 클라이언트 A (1.1.1.1:5000) → 서버:80에 접속
                - 클라이언트 B (2.2.2.2:6000) → 서버:80에 접속
                - 클라이언트 A (1.1.1.1:5001) → 서버:80에 또 접속 (탭 2개 연 거)
                ```
                
            - 서버에는 socket이 **3개** 만들어짐
                
                ```bash
                소켓 1: (1.1.1.1, 5000, 서버IP, 80)
                소켓 2: (2.2.2.2, 6000, 서버IP, 80)
                소켓 3: (1.1.1.1, 5001, 서버IP, 80)
                ```
                
                - 같은 80번 포트를 향하지만 4-tuple이 다 다르니까 각각 별개의 socket
                - Demux는 4-tuple 전체를 보고 정확한 소켓에 배달
        - **Connection-oriented (TCP)**
            - 두 endpoint 간의 개별 연결을 추적해야 함
            - 순서 보장, 재전송, 흐름 제어 등이 “이 특정 연결”의 상태에 묶여 있음
- **NAT - Mux / Demux**
    - NAT : Network Address Translation
        - Private Network 안의 host들이 public IP 없이 Internet과 통신 가능하도록 하는 구조
        - IPv4 address - 약 43억 개 밖에 없음 → 절충안
            - `public IP` Internet 전체에서 유일한 주소라 비싸고 한정적임
            - `private IP` 내부망에서만 쓰는 주소 → 누구나 자기 network에서 자유롭게 사용
                - 대역이 정해져 있음
                    - `10.0.0.0/8`
                    - `172.16.0.0/12`
                    - `192.168.0.0/16`
    - **동작 방식**
        
        ```bash
        [노트북: 192.168.0.10]  ─┐
                                 ├─→ [공유기 / NAT GW] ─→ 인터넷
        [폰:    192.168.0.20]  ─┘    공인 IP: 14.5.6.7
        ```
        
        - 노트북이 `naver.com (223.130.200.107:80)`에 접속 시도
            1. 노트북이 보내는 패킷
                
                ```bash
                출발지: 192.168.0.10:5000
                목적지: 223.130.200.107:80
                ```
                
            2. 공유기가 받아서 출발지를 자기 공인 IP로 바꿔치기
                
                ```bash
                출발지: 14.5.6.7:5000  ← 출발지 IP 변경
                목적지: 223.130.200.107:80
                ```
                
            3. 이걸 인터넷으로 내보냄
                - 네이버 서버 입장에서는 "14.5.6.7에서 왔구나" 하고 받음
            4. 응답이 돌아오면
                - 공유기가 다시 목적지를 `192.168.0.10`으로 바꿔서 노트북에게 전달
    - **NAT의 Demux ?**
        
        > 공유기 뒤에 노트북, 폰, 태블릿이 다 있는데
        이 셋이 동시에 같은 서버 (ex naver.com:80) 에 접속하면? 
        응답 패킷이 돌아왔을 때 공유기는 누구 거인지 어떻게 알아?
        > 
        
        ```bash
        [노트북:  192.168.0.10:5000] ─┐
        [폰:     192.168.0.20:5000] ─┼─→ [공유기 14.5.6.7] ─→ naver.com:80
        [태블릿: 192.168.0.30:5000] ─┘
        ```
        
        - **Solution? NAPT (Network Address Port Translation)**
            - ***Port 까지 갈아치우기***
                
                ```bash
                [노트북]  192.168.0.10:5000  ──→  14.5.6.7:40001
                [폰]     192.168.0.20:5000  ──→  14.5.6.7:40002
                [태블릿] 192.168.0.30:5000  ──→  14.5.6.7:40003
                ```
                
                - 네이버 서버는 각각을 별개의 연결로 인식
                - 응답 시에도 각자 다른 port로 답을 보내줌

### **UDP**

- IP에 **”port 번호 & 선택적 checksum”**만 얹은 transport layer protocol
    - Multiplexing / Demultiplexing
        - Port 번호로 어느 socket것 인지 구분
    - (선택적) Checksum
        - 전송 중 bit 손상 감지
- **Best effort 서비스**
    - IP 계층의 특성을 그대로 물려받음
        - 최선을 다해 보내볼건데 도착은 보장 못함 → 그 위에 보강 X
    - 발생 가능 이슈
        - Segment 손실 가능 (lost)
        - 순서가 뒤바뀔 수 있음 (out-of-order)
        - 중복될 수도 있음 (같은 packet * 2)
- **Why UDP?**
    - ***No connection establishment → RTT Delay X***
        - TCP handshake → 1 RTT를 까먹는데 DNS 같은 짧은 요청에서 효율 에바
    - No connection State
        - TCP는 양쪽 다 연결마다 상태 유지 필요함
            - sequence 번호, window size, RTT 추정 값, Congestion window …
            - 연결 당 메모리 필요
    - Small header size
        - TCP header: 최소 20 byte (option도 포함되면 더 커짐)
        - UDP header: 8 byte → source port, dest port, length, checksum
    - No congestion Control
        - 그냥 보내고 싶은 만큼 다 보냄
- **SNMP (Simple Network Management Protocol)**

#### **UDP Checksum**

- 필요 이유 → Bit 손상 검출
- **Internet Checksum**
    - 데이터를 16 bits 씩 쪼개서 summation
    - 1의 보수 (carry → LSB로)
    - sum + checksum == 0xFFFF

```bash
1. UDP Pseudo-header   ← IP 헤더의 일부 정보
2. UDP Header          ← 출발지/목적지 포트, 길이, 체크섬 (=0으로 두고 계산)
3. UDP Data (payload)
```

- IP 정보를 끌어들이는 이유?
    - *IP Header가 망가져서 잘못된 Host로 배달* 되는 경우를 잡기 위해서
- **Pseudo - header**
    - 실제로 전송이 되지는 않고 checksum 계산할 때만 가상으로 만들어서 포함
        - source IP
        - destination IP
        - 0 (padding)
        - protocol 번호 (UDP = 17)
        - UDP length
- **특이사항**
    - UDP checksum은 선택사항
    - 만약 checksum 계산값이 0이면 0xFFFF로 바꿔서 보냄
    - 오류 검출 시 그냥 packet 버림

### RDT

```bash
- `LEFT` Application Layer (L7)은 Reliable Channel이라는 환상을 누리고 싶음
- `RIGHT` Network Layer (L3)은 보장 안함 - 그냥 Best Effort
```

- Transport Layer (L4) 중간에 꼽껴서 → RDT protocol
- **Unreliable channel** 위에서 **Reliable Service**를 구현한다는 목표
    - ACK? → unreliable channel에서 도착했는지 확인하려고
    - Sequence Number? → 순서 바뀌거나 중복 방지
    - Timer / 재전송? → 손실 우려
    - Checksum? → Bit 손상 우려
- Unreliable : packet loss, bit 오류, 순서 바뀜

#### RDT Protocol

- **PREV**
    - RDT 1.0 : channel 완벽 가정
    - RDT 2.0 : Bit Error만 가정 → ACK/NAK 도입
        - error 발견 시 app layer한테 그 packet의 데이터를 안줌
        - NAK를 sender에게 보냄
        - sender가 NAK 받으면 재전송
    - RDT 2.1, 2.2 : Sequence 번호 도입
    - RDT 3.0 : Packet loss 고려 → Timer 도입
- **Interfaces**
    
    ```bash
    애플리케이션 계층
               │  ▲
       rdt_send│  │deliver_data
               ▼  │
           [ RDT 프로토콜 ]
               │  ▲
       udt_send│  │rdt_rcv
               ▼  │
           네트워크 계층 (unreliable channel)
    ```
    
    - `rdt_send(data)`
        - App layer에서 호출
            - 다른 app으로 data 전송하고 싶음
        - RDT가 알아서 신뢰성 보장하겠지
    - `udt_send(packet)`
        - RDT가 호출 → Network layer로
            - 실제로 network에 packet 내보내고 싶음
        - 이 packet을 unreliable한 network로 던져라
            - UDT(Unreliable Data Transfer) → 손실/오류 가
    - `rdt_rcv(packet)`
        - RDT가 호출됨 (RDT가 packet을 받아야 함)
            - Network layer에서 호출함
        - Packet 도착함 → RDT 깨워서 처리하라고 함
            - RDT: checksum 검사, 순서 확인, 중복 검사 등
    - `deliver_data(data)`
        - RDT가 위로 호출
            - App layer한테 data 제공
        - 검증 완료 → App한테 줘도 되는 data 준비완
    
    ```bash
    앱 ←→ RDT:    "data"           (그냥 데이터)
    RDT ←→ 네트워크: "packet"       (Header + data)
    ```
    
- **RDT 2.0**
    - **FSM**
        
        ![image.png](attachment:c180723f-caae-45fa-8ca7-b4bc626e6be1:image.png)
        
        - **rdt_send(data)**
            - app한테 요청 받음 data 보내라
                - sndpkt 생성하고
                - udt_send(sndpkt): network로 packet 보내기 요청
        - **rdt_rcv(rcvpkt) && isNAK(rcvpkt)**
            - network가 RDT 깨워서 packet 받음
            - 근데 NAK임 → udt_send(sndpkt): network로 packet 보내기 재요청
        - **rdt_rcv(rcvpkt) && isACK(rcvpkt)**
            - network가 RDT 깨워서 packet 받음
            - 근데 ACK임 → 아무 action 안함
                - app한테 요청 받을 준비 상태로 돌아감
                - 다음 data 받을 준비 완료
    - **Stop-and-wait**
        
        ```bash
        송신자                   수신자
          │ ──── 패킷 1 ────→     │
          │ (대기...)             │ (검사)
          │ ←──── ACK ────        │
          │                       │
          │ ──── 패킷 2 ────→     │
          │ (대기...)             │
          │ ←──── ACK ────        │
        ```
        
        - 한 번에 packet 1개만 network에 띄움
        - ACK 받기 전까지는 다음 packet을 보내지 않음
    - **ARQ (Automatic Repeat reQuest)**
        - 오류가 있으면 자동으로 다시 요청해서 받음
            - Error Detection (checksum)
            - Receiver Feedback (ACK/NAK)
            - Retransmission (NAK 받은 경우 재전송)
- **Sequence Number**
    - **Duplicated Data (RDT 2.0)**
        - ACK/NAK 자체가 bit error로 망가지면?
            - ACK/NAK도 실제로 작은 packet임
            
            ```bash
            송신자                       수신자
              │ ──── 패킷 1 ────→          │ (잘 받음, 앱에 deliver)
              │                            │
              │  ←──[ACK이지만 망가짐]──    │
              │                            │
              │ "응답 못 알아보겠다, 다시!"
              │                            │
              │ ──── 패킷 1 (재전송) ───  → │ ← 어??? 이거 같은 데이터인데?
              │                            │ ← 또 앱에 deliver??
            ```
            
            - 같은 data가 receiver의 L7에 2번 전송 가능
    - **Sequence Number (RDT 2.1)**
        1. ACK/NAK가 망가지면 receiver는 그냥 재전송
        2. sender가 각 packet에 고유한 번호를 붙여서 보냄
            - stop-and-wait & 단방향이라면 sequence 번호 0/1만 번갈아 써도 ㄱㅊ
                
                ```bash
                첫 번째 패킷: [seq=0] [data=A] [checksum]
                두 번째 패킷: [seq=1] [data=B] [checksum]
                세 번째 패킷: [seq=0] [data=C] [checksum]   
                ...            ← stop-and-wait에선 0/1 번갈아 사용
                ```
                
        3. receiver는 중복 packet을 버림 (app에 전달 X)
            - 새로운 sequence number → 새 data, app에 전달 O
            - 방금 받은 번호와 같음 → 중복, 버림
            - 중복이라 data를 app에 전달 안해도 ACK는 보냄
                - ACK 못 받으면 sender가 또 재전송
            
            ```bash
            Sender                       Receiver
              │ ── pkt(seq=0, A) ──── →    │ (deliver, ACK)
              │ ← ─[망가진 ACK]─           │
              │ "재전송!"
              │ ── pkt(seq=0, A) ──── →    │ ← 어 똑같은 seq=0이네, 중복!
              │                            │ (버림, 하지만 ACK은 또 보냄)
              │ ← ─── ACK ────             │
              │ "오케이 이제 다음"
              │ ── pkt(seq=1, B) ──── →    │ ...
            ```
            
- **RDT 2.1**
    
    
    | 패킷 | 망가짐? | seq | 액션 | 데이터 deliver? | 상태 변경? |
    | --- | --- | --- | --- | --- | --- |
    |  | 망가짐 | - | NAK 보냄 | 안 함 | 그대로 |
    |  | 멀쩡 | 기대한 거 | ACK 보냄 | **함** | **변경** |
    |  | 멀쩡 | 다른 거 (중복) | ACK 보냄 | 안 함 | 그대로 |
    - 기대한 sequence 번호
    - 망가지지 않고 멀쩡한 packet
        
        → 둘 다 충족한 경우에만 data를 app layer에 올리고 상태 변경 
        
    - Receiver는 자기가 보낸 ACK/NAK가 Sender에게 잘 도착했는지 영원히 알 수 없음
    - 그냥 다음 packet이 올 때까지 기다림
- **RDT 2.2**
    - NAK free
    - ACK + sequence number
        - 중복 ACK == NAK
        - RDT는 ACK sequence number를 FSM 상태가 기억함
            - TCP는 TCB (Transmission Control Block, OS kernel) 에 “연결” 별로
- **RDT 3.0**
    - error가 아닌 **packet loss**
        - error는 적어도 응답이 오기는 한다는 전제
        - loss는 sender가 영원히 멍하니 기다림
        
        ```bash
        Sender                      Receiver
          │ ── pkt ────[손실]──      │ ← 도착 안 함
          │                          │ → 응답도 안 보냄
          │                          │
          │  (영원히 ACK 대기...)    
          │  (영원히 ACK 대기...)
          │
          │  =>>> deadlock
        ```
        
    - **Timer 도입**
        - 정해진 시간까지 sender에게 ACK가 오지 않는다면?
        - → Timeout & 재전송
            
            ```bash
            Sender                     Receiver
              │ ── pkt ─[손실]──         │ ← 안 옴
              │                          │
              │ (타이머 시작)             │
              │ (째깍째깍...)             │
              │                          │
              │ ⏰ 타임아웃!             │
              │ "ACK 안 왔네, 다시!"      │
              │ ── pkt (재전송) ──→       │ ← 이번엔 도착!
              │ ←─── ACK ────            │
            ```
            
        - **Delayed vs Loss ?**
            - 중복 packet 발생 가능
                
                ```bash
                Sender                     Receiver
                  │ ── pkt ────────→         │ (네트워크 어딘가에서 지연 중...)
                  │                          │
                  │ ⏰ 타임아웃!             │
                  │ "ACK 안 왔네, 다시!"      │
                  │ ── pkt (재전송) ──→       │ ← 이게 먼저 도착
                  │                          │ ← (앗, 곧이어 첫 번째 패킷도 도착!)
                  │ ←─── ACK ──              │
                  │                          │ ← 두 번째(원래) 패킷도 옴 = 중복!
                ```
                
                - packet이 살아있지만 receiver에 delay
                - sender는 loss인줄 알고 재전송함
                - 원래 packet, 재전송한 packet 둘 다 도착 → 중복 발생
            - sequence number mechanism
        - **Timer**
            - 동작
                - Packet을 보냄 (sender → receiver)
                - Timer 시작
                - ACK 도착 시 → timer 정지
                - timer 만료 → interrupt → 재전송 trigger
            - Reasonable amount
                - RTT (round-trip time) 이상적이나 변동이 큼

### Pipelining

- **Go-Back-N (GBN)**
    - Receiver가 순서대로만 받음
        - 순서 어긋난 packet은 버림
    - Sender는 ACK가 안 온 packet부터 싹 다 재전송
    - 단순함, 손실 하나에 여러 packet 다시 보내야 해서 비효율적
- **Selective Repeat (SR)**
    - Receiver가 순서 어긋난 packet도 일단 buffer에 저장함
    - Sender는 손실된 packet만 집어서 재전송
    - 효율적, 양측에서 더 많은 상태 관리가 필요해서 복잡함
