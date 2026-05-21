# 1. TCP 개요

## 1-1. TCP
![TCP 이미지](https://upload.wikimedia.org/wikipedia/commons/9/98/Tcp-handshake.svg)
- **TCP** : 신뢰성 있는 연결 지향형 전송 프로토콜
- UDP와 달리 데이터를 보내기 전에 먼저 **connection**을 설정함
- 데이터 전송 중 손실, 순서 뒤바뀜, 중복 전송 같은 문제를 처리해줌
- 애플리케이션 입장에서는 TCP가 **순서 있는 안정적인 byte stream**을 제공해주는 구조

TCP의 핵심 특징은 다음과 같음

- **Connection-oriented** : 데이터 전송 전 handshaking으로 연결 설정
- **Reliable data transfer** : 손실된 데이터 재전송
- **In-order byte stream** : 데이터를 순서 있는 바이트 흐름으로 처리
- **Full-duplex** : 하나의 연결에서 양방향 데이터 전송 가능
- **Point-to-point** : 하나의 송신자와 하나의 수신자 사이 연결
- **Flow control** : 수신자 버퍼가 넘치지 않도록 송신량 조절
- **Congestion control** : 네트워크 혼잡을 줄이기 위한 송신 속도 조절
- **Cumulative ACK** : 연속적으로 받은 데이터의 다음 바이트 번호를 ACK로 전송

⇒ TCP는 단순히 데이터를 보내는 프로토콜이 아니라, **신뢰성·순서·흐름제어·혼잡제어까지 담당하는 전송계층 프로토콜**

---

## 1-2. TCP Connection

- TCP는 데이터를 보내기 전에 먼저 연결을 설정함
- 이 연결은 실제 물리 회선이 생기는 것이 아니라, 양쪽 호스트가 서로 상태 정보를 유지하는 **논리적 연결**
- 연결이 만들어지면 양쪽 TCP는 다음과 같은 정보를 관리함
    - 송신 버퍼
    - 수신 버퍼
    - sequence number
    - acknowledgement number
    - receive window
    - connection state
- TCP는 full-duplex 방식이므로 한쪽이 데이터를 보내는 동시에 받을 수도 있음

⇒ TCP connection은 **양쪽 호스트가 데이터 전송을 위해 상태를 공유하는 논리적 통신 관계**

---

# 2. TCP Segment Structure

## 2-1. TCP Segment

- **TCP segment** : TCP가 네트워크 계층으로 넘기는 데이터 단위
- TCP segment는 크게 **TCP header + application data**로 구성됨
- TCP header에는 신뢰성 있는 전송과 연결 관리를 위한 여러 필드가 들어감

주요 필드는 다음과 같음
![TCP Segment image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FlPmGU%2FbtqxfeGtVjB%2FAAAAAAAAAAAAAAAAAAAAAFaUiHKc-Dw0dZWk4OYSVzh0FfiKmxIJ4juHfMn4dIpa%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1780239599%26allow_ip%3D%26allow_referer%3D%26signature%3DjZ80mOHEGbDZAnkYXSUHbFR%252FAWg%253D)
- **Source port number** : 송신 측 포트 번호
- **Destination port number** : 수신 측 포트 번호
- **Sequence number** : 세그먼트 데이터의 첫 번째 바이트 번호
- **Acknowledgement number** : 다음에 받고 싶은 바이트 번호
- **Header length** : TCP header 길이
- **Receive window** : 수신자가 받을 수 있는 남은 버퍼 공간
- **Checksum** : 오류 검출 필드
- **Options** : TCP 확장 기능
- **Application data** : 애플리케이션에서 내려온 실제 데이터

⇒ TCP segment는 단순 데이터 덩어리가 아니라, **데이터 전달 상태를 관리하기 위한 제어 정보까지 포함한 전송 단위**

---

## 2-2. TCP Flag

TCP header에는 연결 설정, 종료, ACK 여부 등을 표시하는 flag bit가 있음

- **SYN** : 연결 설정 요청
- **ACK** : acknowledgement number가 유효하다는 표시
- **FIN** : 연결 종료 요청
- **RST** : 연결 강제 초기화
- **PSH** : 데이터를 즉시 애플리케이션으로 넘기라는 의미
- **URG** : 긴급 데이터 존재 표시

특히 TCP 연결 설정과 종료에서 중요한 flag는 다음 세 가지임

- **SYN** : 연결 시작
- **ACK** : 확인 응답
- **FIN** : 연결 종료

⇒ TCP flag는 **TCP 연결 상태를 제어하기 위한 표시값**

---

# 3. Sequence Number와 ACK

## 3-1. Sequence Number

- **Sequence number** : 세그먼트 번호가 아니라 데이터의 첫 번째 바이트 번호
- TCP는 데이터를 segment 단위가 아니라 **byte stream**으로 보기 때문에 byte 번호를 사용함
- 따라서 TCP에서 중요한 것은 “몇 번째 세그먼트인가”가 아니라 “몇 번째 바이트부터 시작하는가”임

예시

- Sequence number = 100
- Data size = 20 bytes

이 경우 데이터 범위는 다음과 같음

- byte 100 ~ byte 119

⇒ Sequence number 100은 **이 세그먼트가 100번 바이트부터 시작한다는 의미**

---

## 3-2. ACK Number

- **ACK number** : 다음에 받고 싶은 바이트 번호
- 마지막으로 받은 바이트 번호가 아니라, 그 다음 번호를 ACK로 보냄

예시

- Sequence number = 100
- Data size = 20 bytes
- 수신자가 byte 100 ~ byte 119까지 정상 수신

이 경우 ACK number는 다음과 같음

- ACK = 120

의미는 다음과 같음

- 119번 바이트까지 받았음
- 다음에는 120번 바이트부터 보내면 됨

⇒ ACK number는 **마지막으로 받은 번호가 아니라 다음에 기대하는 번호**

---

## 3-3. Cumulative ACK

- **Cumulative ACK** : 연속적으로 받은 마지막 바이트 다음 번호를 ACK로 보내는 방식
- 중간 ACK가 손실되어도 더 큰 ACK가 오면 이전 데이터까지 함께 확인 가능

예시

- ACK = 100 손실
- 이후 ACK = 120 도착

이 경우 의미는 다음과 같음

- byte 119까지 정상 수신
- ACK=100이 없어도 이전 데이터 수신 확인 가능

⇒ Cumulative ACK 덕분에 TCP는 일부 ACK 손실에 비교적 강한 구조

---

# 4. RTT와 Timeout

## 4-1. RTT

- **RTT** : Round Trip Time
- 세그먼트를 보낸 시점부터 해당 세그먼트에 대한 ACK를 받을 때까지 걸린 시간
- TCP는 RTT를 바탕으로 timeout 시간을 정함

timeout 설정이 중요한 이유는 다음과 같음

- timeout이 너무 짧으면 불필요한 재전송 발생
- timeout이 너무 길면 실제 손실 복구가 늦어짐

⇒ TCP timeout은 **너무 빠르지도, 너무 느리지도 않게 설정하는 것이 핵심**

---

## 4-2. SampleRTT

- **SampleRTT** : 실제로 측정한 RTT 값
- 하나의 세그먼트를 보낸 뒤 ACK가 돌아오기까지 걸린 시간
- 단, 재전송된 세그먼트는 SampleRTT 측정에서 제외함

재전송 세그먼트를 제외하는 이유는 다음과 같음

- ACK가 원래 보낸 세그먼트에 대한 것인지
- 재전송된 세그먼트에 대한 것인지 구분이 어려움

⇒ SampleRTT는 **재전송되지 않은 정상 전송 구간에서 측정한 RTT 값**

---

## 4-3. EstimatedRTT

- **EstimatedRTT** : 여러 SampleRTT를 반영해 부드럽게 만든 RTT 추정값
- RTT는 계속 변하므로 현재 측정값 하나만 사용하지 않음
- TCP는 EWMA 방식을 사용해 RTT를 추정함

공식은 다음과 같음

`EstimatedRTT = (1 - α) × EstimatedRTT + α × SampleRTT`

- α 값 : 일반적으로 0.125
- EWMA : 최근 값은 반영하되 과거 값의 영향은 점점 줄어드는 평균 방식

⇒ EstimatedRTT는 **RTT의 평균적인 흐름을 부드럽게 추정한 값**

---

## 4-4. DevRTT와 TimeoutInterval

- **DevRTT** : RTT 변동성을 나타내는 값
- RTT가 많이 흔들리면 timeout을 더 넉넉하게 잡아야 함

공식은 다음과 같음

`DevRTT = (1 - β) × DevRTT + β × |SampleRTT - EstimatedRTT|`

- β 값 : 일반적으로 0.25

최종 timeout 공식은 다음과 같음

`TimeoutInterval = EstimatedRTT + 4 × DevRTT`

⇒ TimeoutInterval은 **평균 RTT에 안전 여유분을 더한 값**

---

# 5. TCP Reliable Data Transfer

## 5-1. TCP Sender

TCP sender는 세 가지 주요 이벤트에 따라 동작함

1. 애플리케이션으로부터 데이터 수신
2. timeout 발생
3. ACK 수신

### 애플리케이션 데이터 수신 시

- 데이터를 TCP segment로 만듦
- segment에 sequence number 부여
- timer가 꺼져 있으면 timer 시작
- timer는 가장 오래된 unACKed segment 기준

### Timeout 발생 시

- timeout을 발생시킨 segment 재전송
- timer 재시작

### ACK 수신 시

- 새롭게 ACK된 데이터가 있으면 SendBase 갱신
- 아직 unACKed data가 남아 있으면 timer 재시작
- 모든 데이터가 ACK되면 timer 정지

⇒ TCP sender는 **ACK와 timeout을 기준으로 손실 여부를 판단하고 재전송을 수행하는 구조**

---

## 5-2. SendBase

- **SendBase** : 아직 ACK되지 않은 가장 오래된 바이트 번호
- 송신자가 현재 어디까지 ACK를 받았는지 판단하는 기준
- ACK가 도착하면 SendBase가 앞으로 이동함

예시

- SendBase = 100
- ACK = 120 도착

이 경우 의미는 다음과 같음

- byte 119까지 수신 확인
- SendBase는 120으로 이동 가능

⇒ SendBase는 **송신 윈도우에서 가장 앞쪽에 있는 미확인 데이터 위치**

---

## 5-3. TCP Receiver

TCP receiver는 도착한 segment의 순서에 따라 ACK를 다르게 보냄

- 기대한 순서의 segment 도착
- 이미 ACK pending이 있는 상태
- out-of-order segment 도착
- gap을 메우는 segment 도착

### In-order segment 도착

- 기대한 sequence number를 가진 segment
- 바로 ACK를 보내지 않고 잠시 기다릴 수 있음
- 이를 delayed ACK라고 함

### ACK pending이 있는 경우

- 이미 보낼 ACK가 대기 중인 상태
- 새 in-order segment가 오면 즉시 cumulative ACK 전송

### Out-of-order segment 도착

- 기대한 번호보다 뒤의 segment가 먼저 도착한 상황
- 중간에 빠진 segment가 있다는 의미
- receiver는 duplicate ACK 전송

### Gap을 메우는 segment 도착

- 빠져 있던 segment가 도착한 상황
- 즉시 ACK 전송

⇒ TCP receiver는 **수신 순서와 gap 존재 여부에 따라 ACK 생성 방식이 달라지는 구조**

---

# 6. TCP Retransmission

## 6-1. Lost ACK

- **Lost ACK** : 데이터는 수신자에게 정상 도착했지만 ACK가 손실된 상황
- 송신자는 ACK를 받지 못했으므로 timeout 후 같은 segment를 재전송할 수 있음
- 수신자는 이미 받은 데이터이므로 중복 데이터는 버리고 ACK를 다시 보냄

⇒ Lost ACK 상황에서는 **데이터 손실이 없어도 재전송이 발생할 수 있음**

---

## 6-2. Premature Timeout

- **Premature timeout** : 실제 손실이 아닌데 timeout이 너무 빨리 발생한 상황
- 네트워크 지연 때문에 ACK가 늦게 온 것뿐인데 송신자가 손실로 오해할 수 있음
- 결과적으로 불필요한 재전송 발생

⇒ Premature timeout은 **timeout 값을 너무 짧게 잡았을 때 생기는 비효율**

---

## 6-3. Cumulative ACK에 의한 보완

- 중간 ACK가 손실되어도 이후 더 큰 ACK가 오면 이전 데이터까지 함께 확인 가능
- TCP의 cumulative ACK 구조 덕분에 ACK 손실이 항상 큰 문제로 이어지지는 않음

예시

- ACK=100 손실
- 이후 ACK=120 수신

이 경우 의미는 다음과 같음

- byte 119까지 정상 수신
- ACK=100 손실은 ACK=120으로 보완 가능

⇒ Cumulative ACK는 **이전 ACK 손실을 뒤의 ACK가 덮어줄 수 있는 구조**

---

# 7. Fast Retransmit

## 7-1. Duplicate ACK

- **Duplicate ACK** : 같은 ACK number가 반복해서 도착하는 것
- 보통 중간에 특정 segment가 빠졌다는 신호로 해석 가능
- 수신자는 out-of-order segment를 받을 때마다 마지막으로 연속 수신한 위치를 ACK로 반복 전송함

예시

- 수신자가 byte 100부터 기다리는 중
- byte 120 이후 데이터가 먼저 도착
- receiver는 계속 ACK=100 전송

⇒ Duplicate ACK는 **수신자가 아직 특정 바이트를 기다리고 있다는 신호**

---

## 7-2. Fast Retransmit

- **Fast retransmit** : timeout을 기다리지 않고 손실 segment를 빠르게 재전송하는 기능
- 송신자가 같은 ACK를 3개 추가로 받으면 손실 가능성이 높다고 판단함
- 이때 가장 작은 sequence number의 unACKed segment를 즉시 재전송함

동작 흐름은 다음과 같음

1. 특정 segment 손실
2. 이후 segment들이 receiver에게 도착
3. receiver가 같은 ACK 반복 전송
4. sender가 3 duplicate ACK 수신
5. sender가 timeout 전 손실 segment 재전송

⇒ Fast retransmit은 **timeout 기반 재전송보다 빠른 손실 복구 방식**

---

# 8. Flow Control

## 8-1. Flow Control

- **Flow control** : 수신자 버퍼가 넘치지 않도록 송신량을 조절하는 기능
- 수신 애플리케이션이 데이터를 천천히 읽으면 TCP receive buffer에 데이터가 쌓임
- 이 상태에서 송신자가 계속 빠르게 보내면 buffer overflow 가능

⇒ Flow control의 목적은 **receiver buffer overflow 방지**

---

## 8-2. Receive Window

- **Receive window** : 수신자가 현재 받을 수 있는 남은 버퍼 공간
- 줄여서 **rwnd**
- TCP header의 receive window field에 들어감
- receiver는 자신의 남은 버퍼 공간을 rwnd로 sender에게 알림

공식처럼 이해하면 다음과 같음

`in-flight data ≤ rwnd`

- in-flight data : 보냈지만 아직 ACK를 받지 못한 데이터
- rwnd : 수신자가 더 받을 수 있는 데이터 양

⇒ Sender는 rwnd보다 많은 unACKed data를 보내지 않음

---

## 8-3. Flow Control과 Congestion Control 차이

- **Flow control** : 수신자 버퍼 보호 목적
- **Congestion control** : 네트워크 혼잡 방지 목적

둘의 차이는 다음과 같음

- Flow control의 기준 : receiver의 처리 속도와 버퍼 여유 공간
- Congestion control의 기준 : network의 혼잡 상태
- Flow control 문제 상황 : receiver가 데이터를 빨리 못 읽는 상황
- Congestion control 문제 상황 : network에 너무 많은 트래픽이 몰리는 상황

⇒ Flow control은 **수신자 문제**, congestion control은 **네트워크 문제**

---

# 9. TCP Connection Management

## 9-1. Connection Management

- **TCP connection management** : TCP 연결 설정과 종료를 관리하는 기능
- 데이터 전송 전에 양쪽은 먼저 연결 의사와 초기 상태를 맞춰야 함
- 이 과정에서 초기 sequence number, buffer 정보, 연결 상태 등이 설정됨

⇒ TCP는 연결 설정 없이 바로 데이터를 보내지 않고, 먼저 양쪽 상태를 맞춘 뒤 전송 시작

---

## 9-2. 2-way Handshake의 문제점

- **2-way handshake** : 두 번의 메시지 교환만으로 연결을 설정하는 방식
- 하지만 실제 네트워크에서는 메시지 지연, 손실, 재전송, 순서 바뀜이 발생 가능
- 오래된 연결 요청이 나중에 도착하면 서버가 이를 새로운 요청으로 오해할 수 있음

문제 상황은 다음과 같음

- client는 이미 종료됨
- 오래된 connection request가 server에 도착
- server는 새로운 연결로 착각
- server만 연결되었다고 생각

이를 **half-open connection**이라고 함

⇒ 2-way handshake는 **오래된 메시지나 지연된 재전송에 취약한 방식**

---

## 9-3. TCP 3-way Handshake

- **TCP 3-way handshake** : SYN → SYNACK → ACK 순서로 연결을 설정하는 과정
- 양쪽이 살아 있고 연결할 의사가 있음을 확인하기 위한 절차

### 1단계 : Client → Server

- SYN = 1
- Seq = x

의미는 다음과 같음

- client가 연결 요청
- client의 초기 sequence number는 x

### 2단계 : Server → Client

- SYN = 1
- ACK = 1
- Seq = y
- ACKnum = x + 1

의미는 다음과 같음

- server가 client의 SYN 수신 확인
- server도 연결 의사 있음
- server의 초기 sequence number는 y

### 3단계 : Client → Server

- ACK = 1
- ACKnum = y + 1

의미는 다음과 같음

- client가 server의 SYN 수신 확인

⇒ 3-way handshake가 끝나면 양쪽 모두 ESTABLISHED 상태

---

## 9-4. TCP 연결 종료

- TCP 연결 종료에는 **FIN**과 **ACK** 사용
- TCP는 양방향 통신이므로 각 방향의 연결을 따로 닫음
- 한쪽이 FIN을 보내면 “나는 이제 이 방향으로 데이터 안 보냄”이라는 의미
- 상대방은 FIN을 받으면 ACK로 응답함
- 상대방도 보낼 데이터가 끝나면 자신의 FIN을 보냄

정리하면 다음과 같음

- FIN : 데이터 전송 종료 요청
- ACK : 종료 요청 수신 확인
- 양방향 FIN/ACK 완료 : TCP 연결 종료

⇒ TCP 종료는 **한 번에 끊는 것이 아니라 양쪽 방향을 각각 정리하는 과정**

---

# 10. Network Layer Control Plane

## 10-1. Data Plane과 Control Plane

- 네트워크 계층은 크게 **data plane**과 **control plane**으로 나눌 수 있음

### Data Plane

- 실제 패킷을 라우터 입력 포트에서 출력 포트로 전달하는 기능
- 라우터 내부에서 빠르게 수행되는 동작
- 핵심 기능은 forwarding

### Control Plane

- 패킷이 목적지까지 어떤 경로로 갈지 결정하는 기능
- 네트워크 전체 경로 계산과 관련됨
- 핵심 기능은 routing

⇒ Data plane은 **패킷 전달**, control plane은 **경로 결정**

---

## 10-2. Forwarding과 Routing

- **Forwarding** : 라우터가 들어온 패킷을 어느 출력 포트로 내보낼지 결정하는 동작
- **Routing** : 출발지에서 목적지까지 어떤 경로를 사용할지 결정하는 과정

차이는 다음과 같음

- Forwarding : local action
- Routing : network-wide process
- Forwarding 기준 : packet header와 forwarding table
- Routing 기준 : routing algorithm과 network topology

⇒ Forwarding은 **라우터 내부의 포트 선택**, routing은 **전체 경로 계산**

---

## 10-3. Per-router Control Plane

- **Per-router control plane** : 각 라우터가 자기 안에서 routing algorithm을 실행하는 방식
- 전통적인 네트워크 제어 방식
- 라우터들이 서로 routing 정보를 교환함
- 각 라우터가 자신의 forwarding table을 계산함

특징은 다음과 같음

- 제어 기능이 각 라우터에 분산됨
- 각 라우터가 독립적으로 계산 수행
- OSPF, BGP 같은 전통적 라우팅 프로토콜과 관련

⇒ Per-router control plane은 **각 라우터가 스스로 경로 계산에 참여하는 분산형 구조**

---

## 10-4. SDN Control Plane

- **SDN control plane** : 논리적으로 중앙화된 controller가 forwarding table을 계산하는 방식
- 각 라우터가 직접 경로를 계산하기보다는, remote controller가 경로와 forwarding rule을 계산함
- 계산된 forwarding table은 라우터에 설치됨

특징은 다음과 같음

- 제어 기능이 controller에 집중됨
- 라우터는 주로 data plane 역할 수행
- 네트워크 정책 적용이 유연함
- OpenFlow, ODL, ONOS 등과 관련

⇒ SDN control plane은 **중앙 controller가 제어하고 라우터는 전달에 집중하는 구조**

---

# 11. Routing Protocols

## 11-1. Routing Protocol의 목표

- **Routing protocol** : 송신 호스트에서 수신 호스트까지 좋은 경로를 찾기 위한 프로토콜
- 여기서 좋은 경로는 상황에 따라 다르게 정의 가능

좋은 경로의 기준은 다음과 같음

- 비용이 낮은 경로
- 빠른 경로
- 혼잡이 적은 경로
- 링크 품질이 좋은 경로
- **Path** : 패킷이 출발지에서 목적지까지 지나가는 라우터들의 순서

⇒ Routing protocol의 목적은 **목적지까지 효율적인 path를 찾는 것**

---

## 11-2. Graph Abstraction

- 네트워크는 그래프로 표현 가능
- 그래프는 `G = (N, E)` 형태

각 요소는 다음과 같음

- **N** : 라우터들의 집합
- **E** : 라우터들을 연결하는 링크들의 집합
- **Link cost** : 링크를 사용하는 데 드는 비용
- **c(x, y)** : x와 y 사이 링크의 비용

직접 연결이 없는 경우는 다음과 같음

- cost = ∞

cost는 네트워크 운영자가 정할 수 있음

- 모든 링크 cost를 1로 설정
- bandwidth가 클수록 cost를 낮게 설정
- congestion이 심할수록 cost를 높게 설정

⇒ Graph abstraction은 **라우팅 알고리즘에서 경로 계산을 하기 위한 네트워크 표현 방식**

---

## 11-3. Routing Algorithm 분류

Routing algorithm은 정보 범위와 경로 변화 속도에 따라 분류 가능

### Global Routing Algorithm

- 모든 라우터가 전체 topology와 link cost 정보를 알고 있는 방식
- 대표 예시는 link-state algorithm
- 전체 지도를 알고 최단 경로를 계산하는 방식

### Decentralized Routing Algorithm

- 각 라우터가 처음에는 자신과 직접 연결된 이웃 정보만 알고 있는 방식
- 이웃과 정보를 반복적으로 교환하면서 경로를 계산함
- 대표 예시는 distance-vector algorithm

### Static Routing

- 경로가 천천히 변하거나 거의 고정된 방식
- 관리자가 직접 설정하는 경우가 많음

### Dynamic Routing

- 네트워크 상태 변화에 따라 경로가 자동으로 바뀌는 방식
- 링크 비용 변화, 장애, 혼잡 등에 반응 가능

⇒ Routing algorithm은 **전체 정보를 쓰는지, 이웃 정보만 쓰는지 / 경로가 고정인지 동적인지로 구분 가능**

---

# 12. 핵심 비교 정리

## 12-1. Sequence Number vs ACK Number

- **Sequence number** : 내가 보내는 데이터의 첫 번째 바이트 번호
- **ACK number** : 내가 다음에 받고 싶은 상대방 데이터의 바이트 번호

예시

- Seq = 100, data = 20 bytes
- 데이터 범위 = 100 ~ 119
- ACK = 120

⇒ ACK는 **받은 번호가 아니라 다음에 받고 싶은 번호**

---

## 12-2. Timeout vs Fast Retransmit

- **Timeout** : 일정 시간 안에 ACK가 오지 않으면 재전송
- **Fast retransmit** : 3 duplicate ACK를 받으면 timeout 전 재전송

차이는 다음과 같음

- Timeout : 시간 기반 손실 판단
- Fast retransmit : duplicate ACK 기반 손실 판단

⇒ Fast retransmit은 **timeout보다 빠른 손실 복구 방법**

---

## 12-3. Flow Control vs Congestion Control

- **Flow control** : 수신자 버퍼 overflow 방지
- **Congestion control** : 네트워크 혼잡 방지

차이는 다음과 같음

- Flow control 대상 : receiver
- Congestion control 대상 : network
- Flow control 기준 : rwnd
- Congestion control 기준 : 네트워크 혼잡 상태

⇒ Flow control은 **수신자 보호**, congestion control은 **네트워크 보호**

---

## 12-4. Forwarding vs Routing

- **Forwarding** : 라우터 내부에서 패킷을 어느 출력 포트로 보낼지 결정
- **Routing** : 출발지부터 목적지까지 전체 경로 결정

차이는 다음과 같음

- Forwarding : local action
- Routing : network-wide process

⇒ Forwarding은 **포트 선택**, routing은 **경로 계산**

---

## 12-5. Per-router Control Plane vs SDN Control Plane

- **Per-router control plane** : 각 라우터가 직접 routing algorithm 실행
- **SDN control plane** : 중앙 controller가 forwarding table 계산 후 라우터에 설치

차이는 다음과 같음

- Per-router : 분산형 제어
- SDN : 논리적 중앙집중형 제어
- Per-router 라우터 : 경로 계산 + 패킷 전달
- SDN 라우터 : 주로 패킷 전달

⇒ Per-router는 **각 라우터가 스스로 계산**, SDN은 **controller가 계산**