https://jyun627.notion.site/cn2-ch2?source=copy_link

https://jyun627.notion.site/cn2-ch3?source=copy_link

# Chapter 3. Transport Layer 정리

## 1. Transport Layer 개요

### 목표

Transport Layer에서는 다음 개념을 이해하는 것이 핵심이다.

- Multiplexing / Demultiplexing
- Reliable Data Transfer
- Flow Control
- Congestion Control
- UDP
- TCP
- TCP Congestion Control

## 2. Transport Layer Services

### 2.1 Transport Layer의 역할

Transport Layer는 서로 다른 host에서 실행 중인 **application process 간 logical communication**을 제공한다.

즉, 실제 물리적으로 직접 연결된 것은 아니지만, application 입장에서는 마치 두 process가 직접 통신하는 것처럼 보이게 한다.

### Sender 측 동작

Application message를 받으면 Transport Layer는 다음 작업을 한다.

1. Application message를 segment로 나눈다.
   1. segment는 transport layer에서 다루는 데이터 단위
   2. `application이 만든 message + Transport Layer header = segment`
2. Transport header를 붙인다.
3. Segment를 Network Layer, 즉 IP에게 넘긴다.

- segment로 나눈다
  Application message가 너무 크면 한 번에 보내기 어렵다.
  그래서 Transport Layer는 데이터를 적당한 크기로 자른다.
  예:
  ```
  큰 HTTP response
  = [part 1] [part 2] [part 3]
  ```
  각 조각에 TCP header를 붙이면 각각 TCP segment가 된다.
  ```
  TCP header + part 1 = segment 1
  TCP header + part 2 = segment 2
  TCP header + part 3 = segment 3
  ```
  단, UDP는 보통 application이 넘긴 datagram 단위를 유지하려고 한다. TCP는 byte stream이기 때문에 내부적으로 MSS 기준으로 segment를 나눈다.

### Receiver 측 동작

Network Layer로부터 segment를 받으면 Transport Layer는 다음 작업을 한다.

1. Segment header를 확인한다.
2. Application data를 추출한다.
3. 적절한 socket으로 demultiplexing한다.
4. Application process에게 전달한다.

## 3. Transport Layer vs Network Layer

### Network Layer

Network Layer는 **host-to-host communication**을 담당한다.

예를 들어 A 컴퓨터에서 B 컴퓨터까지 packet을 전달하는 역할이다.

### Transport Layer

Transport Layer는 **process-to-process communication**을 담당한다.

즉, 같은 host 안에서도 여러 application process가 있을 수 있으므로, 도착한 data를 어느 process에 전달할지 결정해야 한다.

### 비유

PPT의 집 편지 비유는 다음과 같다.

| 네트워크 개념          | 비유                                    |
| ---------------------- | --------------------------------------- |
| Host                   | 집                                      |
| Process                | 집 안의 아이                            |
| Application message    | 편지                                    |
| Transport protocol     | 집 안에서 편지를 아이에게 나눠주는 사람 |
| Network-layer protocol | 우체국                                  |

즉, 우체국은 집까지 편지를 가져다주지만, 집 안의 누구에게 줄지는 Transport Layer가 처리한다.

## 4. Internet Transport Protocols

인터넷에서 대표적인 Transport Layer protocol은 두 가지다.

## 4.1 TCP

TCP는 다음 특징을 가진다.

- Reliable delivery
- In-order delivery
- Congestion control
- Flow control
- Connection setup

즉, TCP는 data가 손실되거나 순서가 바뀌는 문제를 protocol 차원에서 처리한다.

## 4.2 UDP

UDP는 다음 특징을 가진다.

- Unreliable delivery
- Unordered delivery
- Connectionless
- Best-effort IP를 거의 그대로 사용

UDP는 신뢰성을 직접 보장하지 않는다.

## 4.3 Transport Layer에서 제공하지 않는 것

TCP와 UDP 모두 기본적으로 다음은 보장하지 않는다.

- Delay guarantee
- Bandwidth guarantee

즉, “몇 ms 안에 도착한다”거나 “항상 몇 Mbps를 보장한다”는 기능은 Transport Layer 기본 서비스가 아니다.

# 5. Multiplexing / Demultiplexing

커넥션 당 1소켓임
하나의 프로세스가 여러 소켓을 가진다는 뜻

## 5.1 Multiplexing

Multiplexing은 sender 측 동작이다.

여러 socket에서 내려온 data를 Transport Layer가 받아서 각각 TCP/UDP header를 붙이고 network layer로 내려보낸다.

즉,

> 여러 application process의 data를 하나의 network path로 모아 보내는 것

이다.

### 동작 과정

각 프로세스는 자기 socket을 통해 데이터를 보낸다.

```python
Chrome → socket 1 → Transport Layer
Skype → socket 2 → Transport Layer
Netflix → socket 3 → Transport Layer
```

Transport Layer는 각 데이터에 header를 붙인다.

```python
TCP/UDP header + application data = segment
```

header에는 최소한 다음 정보가 들어간다.

```python
source port
destination port
```

TCP라면 추가로 sequence number, ACK number 등도 들어간다.

그 다음 IP layer로 내려보낸다.

```python
segment → IP datagram → frame
```

→ multiplexing인 이유: 여러 application의 데이터가 Transpory layer를 통해 하나의 network layer inferface로 모여서 내려가기 때문

## 5.2 Demultiplexing

Demultiplexing은 receiver 측 동작이다.

도착한 segment의 header 정보를 보고 올바른 socket으로 전달한다.

즉,

> network에서 올라온 segment를 어느 application process에게 줄지 결정하는 것

이다.

## 5.3 Socket의 역할

Socket은 application process와 transport layer 사이의 통신 endpoint다.

Transport Layer는 segment를 받은 뒤, header의 port number 등을 보고 적절한 socket에 전달한다.

## 5.4 Demultiplexing이 필요한 이유

한 host 안에는 여러 application이 동시에 실행될 수 있다.

예를 들어 같은 컴퓨터에서 다음 process가 동시에 실행될 수 있다.

- Firefox
- Netflix
- Skype

서버에서 HTTP message가 도착했을 때, Transport Layer는 이 message를 Firefox에게 줄지, Netflix에게 줄지, Skype에게 줄지 구분해야 한다.

이때 사용하는 정보가 header에 들어 있는 port number와 IP address다.

# 6. Demultiplexing 동작 방식

receiver host에서 일어난다.

Network Layer에서 올라온 segment를 보고, Transport Layer가 어느 socket으로 전달할 지 결정하는 작업이다.

```python
IP Layer → Transport Layer ┬→ socket A → Chrome
                           ├→ socket B → Skype
                           └→ socket C → Netflix
```

도착한 segment를 header 정보에 따라 올바른 socket으로 분배한다.

## 6.1 IP Datagram이 가진 정보

Host가 IP datagram을 받으면, datagram에는 다음 정보가 있다.

- Source IP address
- Destination IP address
- Transport-layer segment
- Source port number
- Destination port number

Transport Layer는 이 정보를 이용해서 segment를 적절한 socket에 전달한다.

## 6.2 TCP/UDP Segment 기본 구조

Segment에는 기본적으로 다음 정보가 포함된다.

```
source port #
destination port #
other header fields
application data
```

Port number는 demultiplexing의 핵심 정보다.

- receiver 측 동작 과정
  Host가 IP datagram을 받는다.
  IP datagram 안에는 Transport Layer segment가 들어있다.
  ```python
  IP datagram
  = IP header + TCP/UDP segment
  ```
  IP header에는 다음 정보가 있다.
  ```python
  source IP address
  destination IP address
  ```
  TCP/UDP segment header에는 다음 정보가 있다.
  ```python
  source port number
  destination port number
  ```
  Transport Layer는 이 정보를 보고 적절한 socket을 찾는다.
  IP address = 어느 host인가?
  port number = 그 host 안의 어느 process/socket인가?

# 7. Connectionless Demultiplexing: UDP

## 7.1 UDP Socket 생성

UDP는 connection을 만들지 않는다. TCP처럼 handshake도 없고, connection state도 없다.

UDP socket을 만들 때는 host-local port number를 지정한다.

예:

```java
DatagramSocket mySocket1 = new DatagramSocket(12534);
```

이 socket은 local host의 12534번 port에 bind된다.

## 7.2 UDP Segment 수신 과정

Receiver host가 UDP segment를 받으면 다음 과정을 거친다.

1. Segment의 destination port number 확인
   1. 목적지 port가 같으면, 보낸 사람이 달라도 같은 UDP socket으로 전달된다.
   2. 수신 측에서 기본적으로 destination port만 보고 socket을 찾는다.
2. 해당 port number를 가진 socket 탐색
3. 그 socket으로 segment 전달

## 7.3 UDP Demultiplexing의 핵심

UDP는 기본적으로 **destination port number만 보고 socket을 결정**한다.

즉, source IP나 source port가 달라도 destination port가 같으면 같은 socket으로 전달된다.

예를 들어 다음 두 UDP segment가 있다고 하자.

| Source IP | Source Port | Destination Port |
| --------- | ----------- | ---------------- |
| A         | 9157        | 6428             |
| C         | 5775        | 6428             |

두 segment 모두 destination port가 6428이면 receiver의 UDP socket 6428로 전달된다.

# 8. Connection-oriented Demultiplexing: TCP

## 8.1 TCP Socket 식별자

TCP socket은 다음 4가지 값으로 식별된다.

```
source IP address
source port number
destination IP address
destination port number
```

이를 **4-tuple**이라고 한다.

TCP는 connection-oriented이니까 저 4가지를 다 본다.

## 8.2 TCP에서 4-tuple이 필요한 이유

TCP server는 여러 client와 동시에 연결될 수 있다.

예를 들어 여러 client가 모두 server의 port 80으로 접속한다고 해도, server는 각 client별로 다른 socket을 만들어야 한다.

따라서 destination port만으로는 부족하다.

TCP는 다음 정보를 모두 사용한다.

| 항목             | 의미                |
| ---------------- | ------------------- |
| Source IP        | client host         |
| Source port      | client process port |
| Destination IP   | server host         |
| Destination port | server process port |

## 8.3 TCP Demultiplexing 예시

세 개의 segment가 모두 다음 목적지를 가진다고 하자.

```
Destination IP = B
Destination Port = 80
```

하지만 source가 다르면 서로 다른 TCP connection이다.

| Source IP | Source Port | Destination IP | Destination Port |
| --------- | ----------- | -------------- | ---------------- |
| A         | 9157        | B              | 80               |
| C         | 5775        | B              | 80               |
| C         | 9157        | B              | 80               |

이 세 segment는 모두 server B의 port 80으로 오지만, 각각 다른 socket으로 demultiplexing된다.

## 8.4 UDP vs TCP Demultiplexing 비교

| 구분                               | UDP                       | TCP                            |
| ---------------------------------- | ------------------------- | ------------------------------ |
| 연결 방식                          | Connectionless            | Connection-oriented            |
| Socket 식별                        | Destination port 중심     | 4-tuple                        |
| 같은 destination port로 온 segment | 같은 socket으로 전달 가능 | source 정보에 따라 다른 socket |
| 예시                               | DNS, SNMP, streaming      | HTTP, HTTPS 등                 |

- 전체 흐름
  ## Sender
  ```
  1. Application process가 socket에 data 씀
  2. Transport Layer가 data를 받음
  3. source port, destination port 등을 header에 기록
  4. segment 생성
  5. IP layer로 전달
  ```
  ## Receiver
  ```
  1. IP layer가 segment를 Transport Layer로 올림
  2. Transport Layer가 header 확인
  3. destination port 확인
  4. TCP면 4-tuple 확인
  5. 적절한 socket으로 전달
  6. socket을 통해 application process가 data 읽음
  ```

# 9. Multiplexing / Demultiplexing 요약

시험 포인트는 다음과 같다.

- Multiplexing은 sender 측에서 여러 socket data를 모아 segment로 보내는 것
- Demultiplexing은 receiver 측에서 segment를 적절한 socket으로 전달하는 것
- UDP는 destination port number를 이용해 demultiplexing
- TCP는 source IP, source port, destination IP, destination port의 4-tuple을 이용해 demultiplexing
- Multiplexing/demultiplexing은 transport layer뿐 아니라 여러 layer에서 일반적으로 나타나는 개념이다

# 10. UDP: User Datagram Protocol

## 10.1 UDP의 정의

UDP는 “no frills”, “bare bones” transport protocol이다.

즉, 최소한의 기능만 제공하는 단순한 transport protocol이다.

## 10.2 UDP의 특징

UDP는 best-effort service를 제공한다.

UDP segment는 다음 문제가 발생할 수 있다.

- Lost
- Delivered out-of-order

즉, segment가 손실될 수도 있고, application에 도착하는 순서가 바뀔 수도 있다.

## 10.3 UDP는 Connectionless

UDP는 connection establishment가 없다.

즉, TCP처럼 handshake를 하지 않는다.

따라서 connection setup으로 인한 RTT delay가 없다.

## 10.4 UDP의 장점

UDP의 장점은 다음과 같다.

- No handshaking
- Sender/receiver에 connection state 없음
- Header size가 작음
- Congestion control 없음
- 원하는 속도로 전송 가능
- Congestion 상황에서도 동작 가능
  - UDP는 그냥 계속 보냄
  - TCP는 상황 보면서 속도 줄이면서 계속 보냄. tcp도 혼잡 상황에서 통신을 안하진 않음. 속도가 줄어듦

## 10.5 UDP의 단점

UDP는 다음을 보장하지 않는다.

- Reliable delivery
- In-order delivery
- Flow control
- Congestion control

따라서 필요하면 application layer에서 직접 구현해야 한다.

# 11. UDP를 사용하는 이유

UDP가 필요한 이유는 다음과 같다.

1. 빠른 전송

Handshake가 없기 때문에 TCP보다 시작 지연이 작다.

특히 DNS처럼 짧은 request-response에서는 connection setup overhead가 부담될 수 있다.

1. 단순함

Connection state를 유지하지 않아 sender와 receiver 구현이 단순하다.

1. 작은 Header

UDP header는 TCP보다 작다.

1. Application이 직접 제어 가능

필요한 reliability나 congestion control을 application layer에서 직접 설계할 수 있다.

예를 들어 HTTP/3는 UDP 위에 QUIC을 올려 reliability와 congestion control을 application layer 쪽에서 처리한다.

# 12. UDP 사용 예시

PPT에서 제시한 UDP 사용 예시는 다음과 같다.

- Streaming multimedia apps
- DNS
- SNMP
- HTTP/3

## 12.1 Streaming Multimedia

Streaming multimedia는 어느 정도 packet loss를 허용할 수 있다.

대신 rate에 민감하다.

즉, 완벽한 신뢰성보다 일정한 속도가 더 중요하다.

## 12.2 DNS

DNS는 보통 짧은 query-response 구조다.

TCP handshake를 매번 하면 overhead가 크므로 UDP를 많이 사용한다.

## 12.3 SNMP

simple network management protocol: 네트워크 장비를 모니터링하고 관리하기 위한 프로토콜
라우터, 스위치, 서버 같은 장비의 상태를 확인하거나 설정을 바꿀 때 사용

SNMP도 UDP를 사용하는 대표적인 application이다.

## 12.4 HTTP/3

HTTP/3는 UDP 위에서 동작한다.

단, UDP 자체가 reliability를 제공하지 않으므로 QUIC이 reliability, congestion control 등을 application layer에서 구현한다.

# 13. UDP Transport Layer Actions

## 13.1 UDP Sender Actions

UDP sender는 다음 작업을 한다.

1. Application-layer message를 받는다.
2. UDP segment header field 값을 결정한다.
3. UDP segment를 생성한다.
4. IP layer로 넘긴다.

![image.png](attachment:c74d41ee-edd0-4d94-ad6d-0f4f294b58dd:image.png)

예시 구조:

```
SNMP message
↓
UDP header + SNMP message
↓
IP layer
```

## 13.2 UDP Receiver Actions

UDP receiver는 다음 작업을 한다.

1. IP layer로부터 UDP segment를 받는다.
2. UDP checksum header 값을 검사한다.
3. Application-layer message를 추출한다.
4. Socket을 통해 application으로 demultiplexing한다.

![image.png](attachment:04593b65-f447-44b5-bfc1-f44807cfedc0:image.png)

# 14. UDP Segment Header

UDP segment header에는 다음 field가 있다.

![image.png](attachment:8b4f26f5-ba96-41a1-983b-d9cbdaf402f0:image.png)

```
source port #
destination port #
length
checksum
application data
```

## 14.1 Source Port Number

Sender process의 port number다.

Receiver가 reply를 보낼 때 사용할 수 있다.

## 14.2 Destination Port Number

Receiver host에서 어느 socket으로 전달할지 결정하는 핵심 정보다.

## 14.3 Length

UDP segment 전체 길이를 byte 단위로 나타낸다.

여기에는 header와 data가 모두 포함된다.

## 14.4 Checksum

전송 중 bit error가 발생했는지 검사하기 위한 값이다.

# 15. UDP Checksum

## 15.1 목적

UDP checksum의 목적은 transmitted segment에서 bit flip 같은 error를 detect하는 것이다.

즉, data가 전송 중 손상되었는지 확인한다.

## 15.2 기본 아이디어

Sender는 segment 내용을 바탕으로 checksum을 계산해서 checksum field에 넣는다.

Receiver는 받은 segment로 다시 checksum을 계산하고, header의 checksum 값과 비교한다.

| 비교 결과                          | 의미               |
| ---------------------------------- | ------------------ |
| computed checksum ≠ checksum field | Error detected     |
| computed checksum = checksum field | Error not detected |

단, checksum이 같다고 해서 반드시 error가 없다는 뜻은 아니다.

# 16. Internet Checksum

## 16.1 Sender 측 계산

Sender는 다음 과정을 수행한다.

1. UDP segment 내용을 16-bit integer들의 sequence로 본다.
2. 이 값들을 one’s complement sum 방식으로 더한다.
3. 계산된 checksum 값을 UDP checksum field에 넣는다.

UDP checksum 계산에는 UDP header뿐 아니라 IP address 정보도 일부 포함된다.

## 16.2 Receiver 측 계산

Receiver는 받은 segment에 대해 checksum을 다시 계산한다.

그 결과가 checksum field와 다르면 error가 detect된다.

## 16.3 Carry Wraparound

Internet checksum에서는 가장 높은 bit에서 carry가 발생하면 그 carry를 다시 아래쪽에 더한다.

이를 wraparound라고 한다.

## 16.4 Checksum의 한계

UDP checksum은 weak protection이다.

어떤 bit들이 동시에 바뀌면 전체 합이 그대로 유지될 수 있다.

즉, segment 내용이 바뀌었는데도 checksum이 같을 수 있다.

따라서 checksum은 error를 완벽히 잡는 것이 아니라, 일부 error를 detect하는 기능이다.

- checksum 계산 예시
    <aside>
    
    전체흐름
    
    1. 데이터를 16-bit 단위로 나눔
    2. 전부 더함 (one’s complement addition)
    3. carry 발생 시 wraparound 처리
    4. 최종 결과를 bit 반전 → checksum
    </aside>
    
    입력 데이터 
    
    ```
    A = 1100 1100 1100 1100
    B = 1010 1010 1010 1010
    ```
    
    1. 더하기 
    
    ```
      1100 1100 1100 1100
    + 1010 1010 1010 1010
    --------------------
    1 0111 0111 0111 0110
    ```
    
    여기서 `맨 앞의 1 = carry` 
    
    1. Wraparound (carry 처리)
        1. carry를 다시 더한다: 
        
        ```
          0111 0111 0111 0110
        + 0000 0000 0000 0001
        --------------------
          0111 0111 0111 0111
        ```
        
    2. 1의 보수 (bit 반전) 
    
    ```
    0111 0111 0111 0111
    ↓
    1000 1000 1000 1000
    ```
    
    이 값이 checksum 임 
    
    ### receiver 에서 검증
    
    receiver는 data + checksum을 다시 더함 → 정상이라면 결과 `1111 1111 1111 1111`  (all 1) → 문제 없음

# 17. UDP 요약

UDP는 다음과 같이 정리할 수 있다.

- No frills protocol
- Segment가 lost될 수 있음
- Segment가 out-of-order로 delivered될 수 있음
- Best-effort service
- Handshaking 없음
- Setup RTT 없음
- Checksum으로 error detection 가능
- Reliability가 필요하면 application layer에서 직접 구현
- HTTP/3는 UDP 위에 추가 기능을 구현한 대표 사례

# 18. Principles of Reliable Data Transfer (RDT)

## 18.1 기본 개념

Transport Layer는 **unreliable network 위에서 reliable service를 구현**해야 한다.

### 핵심 아이디어

- 실제 network는 unreliable (loss, corruption, reorder)
- 하지만 application에는 **reliable channel처럼 보이게 해야 함**

```
Application view: reliable channel
Reality: unreliable channel + protocol logic
```

---

## 18.2 구조

Reliable Data Transfer는 다음 두 부분으로 구성된다.

- sender-side protocol
- receiver-side protocol

이 둘이 협력해서 reliability를 만든다.

---

## 18.3 중요한 특징

- sender와 receiver는 서로의 상태를 모름
- 상태는 메시지를 통해서만 알 수 있음 (ACK 등)

---

# 19. RDT 인터페이스

<aside>

rdt 인터페이스

Reliable Data Transfer 프로토콜이 위/아래 계층과 통신할 때 사용하는 함수규칙

</aside>

핵심 함수 4개 (시험 매우 중요)

| 함수               | 역할                        |
| ------------------ | --------------------------- |
| rdt_send(data)     | application → sender(rdt)   |
| udt_send(packet)   | sender(rdt) → network       |
| rdt_rcv(packet)    | network → receiver(rdt)     |
| deliver_data(data) | receiver(rdt) → application |

rdt는 application과 transport 사이에있는게 ❌
transport layer 자체의 내부 동작 모델 ⭕️

```
Application
    ↑ deliver_data()
Transport Layer (TCP/UDP 구현 = rdt 개념)
    ↓ udt_send()
Network Layer (IP, unreliable, udt)
```

rdt = transport layer가 신뢰성을 구현하는 방식

# 20. FSM (Finite State Machine)

RDT는 FSM으로 모델링된다.

구성 요소:

- state
- event
- action
- state transition

![image.png](attachment:02a73faf-20a4-46d1-87ed-9fd168f5918a:image.png)

# 21. rdt1.0 (완전한 채널)

## 가정

- bit error 없음
- packet loss 없음

## 특징

- sender: 그냥 send
- receiver: 그냥 receive

→ 매우 단순 (현실에서는 의미 없음)

![image.png](attachment:d2837725-c83e-40df-853f-e7f72bc98401:image.png)

# 22. rdt2.0 (bit error 존재)

## 문제

- packet corruption 발생

## 해결 방법

- checksum → error detection
- ACK / NAK 도입

## 동작

- 정상 → ACK
- 오류 → NAK → 재전송

## 방식

- stop-and-wait

![image.png](attachment:b0fb47e8-d5e4-4859-9cd7-e192a4aa9b17:image.png)

- state 위에 적힌 것 = 이벤트 + 조건
- 아래 = 행동

sender 읽는 법

- 상태1
  - wait for call from above
  - application이 데이터 보내라고 할 때까지 기다림
- 이벤트 발생
  - `rdt_send(data)`: application이 데이터 보냄
- 행동
  - `sndpkt = make_pkt(data, checksum)
udt_send(sndpkt)`
- 의미
  - checksum 포함해서 packet 생성
  - network로 전송
- 상태 전의
  - wait for ACK or NAK
- 상태 2
  - wait for ACK or NAK
  - 상대가 잘 받았는지 기다림
- case 1: ACK 받음
  - `rdt_rcv(rcvpkt) && isACK(rcvpkt)`
  - 정상 수신 → 다시 wait for call from above
- case 2: NAK 받음
  - `rdt_rcv(rcvpkt) && isNAK(rcvpkt)`
  - 에러 발생 → `udt_send(sndpkt)` → 재전송

receiver 읽는 법

- 상태
  - wait for call from below
  - network에서 packet 올 때까지 기다림
- case 1: packet 손상됨
  - `rdt_rcv(rcvpkt) && corrupt(rcvpkt)`
  - 행동: `udt_send(NAK)` → “다시 보내” 요청
- case2: 정상 packet
  - `rdt_rcv(rcvpkt) && notcorrupt(rcvpkt)`
  - 행동
    - `extract(rcvpkt, data)`: 데이터 꺼냄
      `deliver_data(data)`: application 에 전달
      `udt_send(ACK)`: ACK 보냄
- 전체 흐름 연결

  ```
  Sender                    Receiver
  -----------------------------------------
  rdt_send(data)
  → make_pkt
  → send packet      → rdt_rcv

                        정상 → ACK
                        오류 → NAK

  ACK → 다음 데이터
  NAK → 재전송
  ```

![image.png](attachment:5cab2363-35c9-4732-aa8a-85fbec4c60fa:image.png)

---

## 22.1 문제점

ACK/NAK 자체가 손상되면?

→ sender는 상태를 알 수 없음

---

# 23. rdt2.1 (sequence number 도입)

## 해결

- sequence number 추가 (0,1)

## 이유

- duplicate packet 구분

## 특징

- sender는 ACK/NAK corruption 고려
- receiver는 duplicate discard

sender FSM

![image.png](attachment:c0bc4a8b-8f01-459d-83d8-30e11e0cb2e7:image.png)

receiver FSM

![image.png](attachment:d00a45e7-ea2e-4314-8516-4a5481856544:image.png)

# 24. rdt2.2 (NAK 제거)

## 핵심

- NAK 없이 ACK만 사용

## 방법

- 잘못된 packet → 이전 ACK 재전송

→ duplicate ACK = implicit NAK

---

# 25. rdt3.0 (loss 포함)

## 새로운 문제

- packet loss
- ACK loss

## 해결

- timeout 도입

---

## 동작

- sender:
  - packet 전송 후 timer 시작
  - timeout 발생 시 재전송
- receiver:
  - duplicate detection 후 discard

---

## 핵심 특징

- checksum
- sequence number
- ACK
- timeout

→ TCP의 핵심 구성 요소

---

# 26. Stop-and-Wait 성능 문제

## 문제

RTT 동안 대기 → 비효율

## 예시 (ppt)

- 1Gbps link
- RTT = 30ms
- utilization ≈ 0.00027

→ 매우 비효율

---

# 27. Pipelining

## 핵심 아이디어

여러 packet을 동시에 전송 (in-flight)

## 요구사항

- larger sequence number space
- buffering 필요

---

# 28. Go-Back-N (GBN)

## 28.1 Sender

- window size = N
- cumulative ACK 사용
- timeout 시:
  → 해당 packet 이후 전부 재전송

---

## 28.2 Receiver

- in-order packet만 처리
- out-of-order packet → discard
- 항상 마지막 정상 packet ACK

---

## 특징

- 구현 단순
- 비효율적 (많이 재전송)

---

# 29. Selective Repeat (SR)

## 29.1 특징

- 각 packet individually ACK
- out-of-order packet도 buffer

---

## 29.2 Sender

- 각 packet별 timer
- timeout → 해당 packet만 재전송

---

## 29.3 Receiver

- out-of-order도 저장
- in-order 가능 시 전달

---

## 29.4 문제

sequence number overlap 문제

→ 조건:

```
window size ≤ sequence space / 2
```

---

# 30. TCP Overview

## 특징

- reliable
- in-order
- connection-oriented
- full duplex
- flow control
- congestion control

---

## 중요한 포인트

- byte stream 기반 (segment 기반 아님)
- cumulative ACK

---

# 31. TCP Segment Structure

## 주요 필드

| 필드            | 설명            |
| --------------- | --------------- |
| source port     | 송신 포트       |
| dest port       | 수신 포트       |
| sequence number | byte 단위 번호  |
| ACK number      | 다음 기대 byte  |
| rwnd            | receiver buffer |
| checksum        | 오류 검출       |
| SYN/ACK/FIN     | connection 관리 |

# 32. TCP Sequence Number & ACK

## Sequence Number

- segment의 첫 byte 번호

## ACK Number

- 다음에 받을 byte 번호

→ cumulative ACK

# 33. TCP RTT & Timeout

## 문제

RTT는 변함 → 고정 timeout 불가능

---

## 해결

### EstimatedRTT

EstimatedRTT = (1-\alpha)EstimatedRTT + \alpha SampleRTT

- α = 0.125

---

### DevRTT

DevRTT = (1-\beta)DevRTT + \beta |SampleRTT - EstimatedRTT|

- β = 0.25

---

### TimeoutInterval

TimeoutInterval = EstimatedRTT + 4 \cdot DevRTT

---

# 34. TCP Reliable Data Transfer

## Sender

- data → segment 생성
- timer 시작
- timeout → 재전송

---

## Receiver

ACK 정책 (중요)

| 상황         | 행동          |
| ------------ | ------------- |
| in-order     | delayed ACK   |
| 두 개 도착   | 즉시 ACK      |
| out-of-order | duplicate ACK |
| gap 채움     | 즉시 ACK      |

---

# 35. Fast Retransmit

## 조건

- duplicate ACK 3개 수신

## 동작

- timeout 기다리지 않고 즉시 재전송

---

# 36. TCP Flow Control

## 문제

receiver buffer overflow 방지

---

## 해결

- receiver가 rwnd 광고
- sender는 rwnd 만큼만 전송

```
LastByteSent - LastByteAcked ≤ rwnd
```

---

# 37. TCP Connection Management

## 목적

- connection 설정
- 초기 seq 번호 교환

---

# 38. 2-way handshake 문제

문제:

- delayed packet
- retransmission
- half-open connection

→ 안정성 부족

---

# 39. 3-way handshake (매우 중요)

## 과정

1. SYN (client → server)
2. SYN + ACK (server → client)
3. ACK (client → server)

---

## 의미

- 양쪽 모두 alive 확인
- seq 번호 동기화

---

# 40. TCP Connection 종료

## 과정

- FIN → ACK
- 양쪽 각각 종료

---

## 특징

- half-close 가능
- simultaneous close 가능

---

# 41. 정리 (시험 핵심 포인트)

## 반드시 외워야 하는 것

### RDT

- rdt1.0 ~ rdt3.0 evolution
- stop-and-wait 문제
- pipelining 필요성

---

### ARQ 프로토콜

- GBN vs SR 차이

---

### TCP

- 3-way handshake
- sequence number vs ACK
- timeout 계산식
- fast retransmit
- flow control (rwnd)
